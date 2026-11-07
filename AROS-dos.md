# AROS DOS library: source-only bug candidates

Reviewed at source commit `282a0454356dc2b9554ab023997d7f7ca24fc03e`. This is a static review, not a runtime reproduction. No project code, build output, devices, or media were changed or used. Line numbers refer to that commit. Severity describes the plausible consequence, and confidence describes how directly the code establishes it. Handler-specific outcomes remain conditional on handler behavior.

## Coverage and boundaries

The active library manifests are `rom/dos/mmakefile.src` (`FUNCTIONS`, `FILES`, `LOADSEG_FILES`) and `rom/dos64/mmakefile.src` (`FUNCS`, `FILES`). The review covered packet transport and wrappers; open, close, seek, buffered and raw I/O; DOS64 native and 32-bit delegation; path, lock, assign, DOS-list and device resolution; metadata, notifications, record locks and enumeration; process/CLI streams, variables and command execution; and the loader/build selections. Narrow adjacent handler behavior was inspected only to establish a DOS-side contract. Bugs inside filesystem implementations belong in [AROS-filesystems.md](AROS-filesystems.md); disk-driver bugs belong in [AROS-disk.md](AROS-disk.md). The candidates below are deduplicated by root cause; related call sites are grouped.

## Findings

### D01 — Buffered write failure is reported as a successful `Close` [high impact, high confidence]

`rom/dos/close.c:68-76`: `Flush(file)` can fail, but its result is immediately overwritten by the `ACTION_END` result. Trigger: a buffered write fails during final close and the handler accepts `ACTION_END`. Consequence: `Close` returns success even though data was not written; `InternalFlush` has already cleared `FHF_WRITE` on error (`rom/dos/internalflush.c:47-51`). Applies to buffered writable handles. The comment in `Close` explicitly promises that final-write failure can be reported.

### D02 — `InternalFlush` discards pending bytes after a negative write and spins on zero progress [high impact, high confidence]

`rom/dos/internalflush.c:39-56`: on `Write` returning `-1`, the routine clears `FHF_WRITE`, but leaves the buffered bytes and position intact. A subsequent `Flush` sees no write flag and cannot retry them. If `Write` returns `0` for a nonzero request, the loop never advances. Trigger: error or zero-byte result from a handler during buffered output. Consequence: data loss or permanent hang. `rom/dos/fread.c:99-116` has the analogous no-progress loop when switching a handle from write to read; after a partial write followed by error, a retry restarts at offset zero and can duplicate the already written prefix.

### D03 — Seek ignores failed buffer flush [high impact, high confidence]

`rom/dos/seek.c:71-80` calls `InternalFlush` without checking its boolean result, then seeks anyway. `rom/dos64/dos64_support.c:85-101` likewise ignores `Flush(file)`, and `rom/dos64/getfilesize64.c:60-63` ignores it before querying size. Trigger: pending buffered writes followed by seek or 64-bit size query, with the write failing. Consequence: operation appears successful while unwritten data is lost or a stale size is returned. Applies to writable buffered handles, including DOS64 wrappers.

### D04 — `SetVBuf` drops buffered data when changing size [high impact, high confidence]

`rom/dos/setvbuf.c:79-96,103-116`: a size change calls `vbuf_free` before flushing pending output or rewinding unread input. `vbuf_free` clears positions and frees the old buffer. Trigger: `SetVBuf(handle, ..., size >= 0)` after `FWrite`/`FPutC` or after prefetched reads. Consequence: pending output disappears, or unread bytes are silently skipped. This affects otherwise valid buffer-mode changes.

### D05 — `FGets` reads when the destination has no room [medium impact, high confidence]

`rom/dos/fgets.c:52-78`: `buflen--` is unconditional and the `do` loop performs a `FGetC` before checking its length limit. At `buflen == 1`, it consumes and writes one byte plus NUL into a one-byte buffer; at `buflen == 0`, unsigned underflow allows an unbounded read. Trigger: zero- or one-byte destination. Consequence: overwrite and/or unintended stream consumption. The function's own contract says it reads at most `buflen-1` bytes.

### D06 — `FRead` block-count multiplication can wrap [medium impact, high confidence]

`rom/dos/fread.c:60-72`: `number * blocklen` is stored in a 32-bit `ULONG` without an overflow check. Trigger: large but individually valid count and block size whose product exceeds `ULONG_MAX`. Consequence: fewer bytes are read than requested, potentially zero with a success-looking return. Applies to buffered block reads.

### D07 — `Rename` copies an unbounded pathname into a 256-byte stack buffer [high impact, high confidence]

`rom/dos/rename.c:118-127`: when either name contains a parent reference and the new name has a volume prefix, `strcpy(buf2, newName)` runs before any length check. Trigger: a destination pathname longer than 255 bytes in this branch. Consequence: stack corruption before the handler sees the request. Applies independently of the filesystem.

### D08 — Assign creation reads one byte past the source string [low to medium impact, high confidence]

`rom/dos/assignpath.c:67-85` and `rom/dos/assignlate.c:66-84`: after `while (*s2++)`, `s2-path` already includes the terminator; `+1` makes `CopyMem` read an extra byte. Trigger: `AssignPath` or `AssignLate` with a normal NUL-terminated path at the end of accessible memory. Consequence: out-of-bounds read or fault. The destination is overallocated, so the primary issue is the source read.

### D09 — Nonbinding assign target failure dereferences a null lock [high impact, high confidence]

`rom/dos/getdeviceproc.c:268-279`: a `DLT_NONBINDING` target is resolved with `Lock`, then `BADDR(lock)->fl_Task` is read without checking for `BNULL`. Trigger: target path missing, unmounted, or lock allocation failure. Consequence: null dereference in `GetDeviceProc`, so callers cannot receive an ordinary lookup failure.

### D10 — Packet-name conversion failure is not handled [medium impact, high confidence]

`rom/dos/packethelper.c:15-49`: `getpacketinfo` returns true even when `C2BSTR(name)` fails; the colon branch also leaks the allocated BSTR when `GetDeviceProc` fails. Callers including `CreateDir`, `DeleteFile`, and `SetProtection` then send packets with a null name, or leak on failed device lookup. Trigger: allocation failure or unknown device name on builds without `AROS_FAST_BSTR` (notably m68k, where `C2BSTR` allocates). Consequence: incorrect packet request and/or leak. Applies to the shared metadata-operation helper. Fast-BSTR targets have no conversion allocation here.

### D11 — Failed `GetVar` size query leaks an open file [medium impact, high confidence]

`rom/dos/getvar.c:209-233`: after `Open` succeeds, `ExamineFH` failure returns `-1` without `Close(file)`. Trigger: a global variable file whose examine packet fails. Consequence: leaked handle and handler resources on each attempt.

### D12 — `GetVar` scans beyond valid data and can turn a failed read into success [medium to high impact, high confidence]

`rom/dos/getvar.c:134-145,245-263`: both newline scans evaluate `buffer[j]` before `j < i`. When `i` is zero or the valid data has no newline, this reads past the returned bytes; the global branch also reaches `buffer[0]='\0'` after `Read` returns `-1` and then returns a nonnegative length. A selected caller is `netinfo.device` startup: it reads the global `SYS/userdb` and `SYS/groupdb` values into a 128-byte stack buffer with default text flags (`workbench/devs/netinfo/server.c:31-55`). If either file supplies at least 128 bytes with no newline in that prefix, `getvar_from` reads `buffer[128]` before its length test, one byte beyond that caller's array; it subsequently truncates and NUL-terminates the result at byte 127, so the later `StrDup` is not itself the overread. Trigger: an empty local value, a full no-newline variable buffer, or a failed global read. Consequence: an out-of-bounds read and, on the failed-read branch, masked I/O failure. No variable file or startup was run; source-only.

### D13 — `SetVar` local allocation failures are unchecked [high impact, high confidence]

`rom/dos/setvar.c:119-194`: a failed `AllocVec` for a new local variable leaves `lv == NULL`, yet code writes `lv->lv_Len`. For an existing variable, the old value is freed before the replacement `AllocMem`, which is unchecked before `CopyMem`. Trigger: low memory while setting a local variable. Consequence: crash, use-after-free state, or loss of the previous value. Applies to local and combined local/global setting.

### D14 — `SetVar` claims success after failed or short global writes [medium impact, high confidence]

`rom/dos/setvar.c:211-263`: both ENV: and ENVARC: branches ignore `Write` and `Close` results, and failure to open ENVARC: is also ignored. Trigger: full or read-only environment volume, short write, or archival-volume failure. Consequence: `DOSTRUE` despite truncated or missing variable data. Applies when global persistence is requested.

### D15 — `StartNotify` copies caller/device names into fixed stack buffers without bounds [high impact, high confidence]

`rom/dos/startnotify.c:58-117`: `name` and `buf` are `MAXFILENAMELENGTH+1` bytes; the loops copying the volume prefix and `fib_FileName`, plus `':'` and terminator, do not check remaining capacity. The first loop also has no NUL stop if a name without `':'` reaches this branch. Trigger: long notification path/device or maximal-length reported root name. Consequence: stack overwrite or out-of-bounds scan.

### D16 — `StartNotify` leaks a lock when root examination fails [low to medium impact, high confidence]

`rom/dos/startnotify.c:95-107`: `Lock(name, SHARED_LOCK)` succeeds; an `Examine` failure frees the FIB and DevProc but does not `UnLock(lock)`. Trigger: handler error during root examine. Consequence: one leaked lock per failed notification request.

### D17 — 32-bit DOS64 packet helper does not install reply error in `IoErr` [medium impact, high confidence]

`rom/dos64/dos64_packet.c:71-110`: `dos64_SendPkt64OS4` returns the packet result and optionally copies `dp_Res2` to `res2`, but never calls `SetIoErr` after a completed reply. Trigger: a 32-bit build using OS4-style 64-bit seek, size, or probe packets that fail. Consequence: callers receive `-1`/false with stale `IoErr`, contrary to their documented error contract. The allocation/error paths do set `IoErr`; this finding concerns completed packets.

### D18 — DOS64 seek may misidentify a failed OS4-style position change [medium impact, medium confidence]

`rom/dos64/dos64_support.c:29-47`: after querying the old position, `ACTION_CHANGE_FILE_POSITION64` is treated as successful on any nonzero result. The protocol's boolean success is `DOSTRUE`/`DOSFALSE`; a handler returning `-1` on failure makes `Seek64` report the old position as success. This candidate depends on a handler using `-1` for failure; the code does not test `ret == DOSTRUE` or the packet error. The selected NTFS handler does not take this `-1` path: it returns `DOSFALSE` with `dp_Res2 == 0`; that separate unsupported-packet reporting mismatch is D34 below.

### D19 — `LockRecords` rollback includes the failed record [medium impact, high confidence]

`rom/dos/lockrecords.c:55-67`: on failure it calls `UnLockRecords(rLock)` before replacing the failed entry's `rec_FH` with `BNULL`; it restores the same value afterward without ever setting it to `BNULL`. `UnLockRecords` thus walks through the failed record and all later entries until the original terminator. Trigger: failure of any record but the last. Consequence: unsolicited unlock packets and potential release of unrelated locks. `rom/dos64/lockrecords64.c` sets the terminator before rollback and avoids this specific error.

### D20 — Process argument injection can fail silently [medium impact, high confidence]

`rom/dos/setvbuf.c:169-203`: for an existing buffered handle whose buffer pointer no longer matches its original buffer, `vbuf_inject` rejects the operation with `FALSE`; after replacing the old buffer, its allocation-failure path nevertheless falls through and returns `TRUE`. `rom/dos/createnewproc.c:519-526` ignores the helper result in either case, so a CLI child can start without its requested arguments. `SystemTagList` checks the result (`systemtaglist.c:270-274`), but that cannot detect the allocation-failure case because the helper reports success. Trigger: pass `NP_Arguments` with a non-null `NP_Input` using a modified existing buffer, or exhaust memory while the helper replaces/allocates that buffer. Consequence: the child starts with absent or truncated command input while process creation reports success. No buffer alteration or allocation failure was induced; source-only.

### D21 — `SystemTagList` dereferences a failed command-copy allocation [high impact, high confidence]

`rom/dos/systemtaglist.c:257-268`: when a command lacks a trailing newline, `AllocVec(commandlen + 2)` is used by `CopyMem` without a null check. The selected print-to-tool backend passes the chosen tool path directly as that command (`workbench/devs/printtotool/printtotool.c:19-34`), so an ordinary path without a trailing newline reaches this allocation before the tool launches. Trigger: low memory or an allocation too large. Consequence: null write/crash instead of a failed `SystemTagList` call. No allocation failure was induced; source-only.

### D22 — `ExAll` fallback writes a link even if the output buffer holds no entry [medium impact, high confidence]

`rom/dos/exall.c:158-173,211-219,289-304`: the emulation initializes `last = buffer` and unconditionally writes `last->ed_Next = NULL` at `end`, including error exits before any entry fits. Trigger: handler without `ACTION_EXAMINE_ALL`, caller supplies a buffer smaller than `offsetof(struct ExAllData, ed_Next)+sizeof(pointer)` or zero. Consequence: write outside the provided buffer, rather than a clean overflow error.

### D23 — CLI string getters underflow on zero buffer length [medium impact, high confidence]

`rom/dos/getprompt.c:74-81`, `rom/dos/getprogramname.c:75-82`, and `rom/dos/getcurrentdirname.c:73-80` compare `clen >= len-1` and then set `clen = len-1` without first checking `len > 0`. With `len == 0`, this becomes `-1` (then a huge `ULONG` in `clen`) and `CopyMem` overruns the caller buffer. `GetProgramName` also writes through `buf` on the no-CLI path without a length check. Trigger: zero-length destination.

### D24 — `internal_CopyPath` silently truncates a search path on allocation/lock failure [medium impact, high confidence]

`rom/dos/createnewproc.c:692-713`: on failed `AllocVec` or `DupLock`, the loop breaks, terminates the copied path, and returns the surviving prefix as if complete. On `DupLock` failure, the just-allocated node is also lost. Trigger: low memory or lock failure during CLI child creation. Consequence: commands resolve against an incomplete path with no failure indication, plus a leak.

### D25 — `ReadArgs` help-line allocation and terminator handling can overrun [high impact, high confidence]

`rom/dos/readargs.c:342-390`: on the interactive `?` help path, `AllocVec(ibuf)` is not checked before `CopyMem`, and the line reader writes `iline[isize] = '\0'` immediately after seeing a newline. If the last available byte holds that newline, `isize == ibuf` and the terminator is one byte out of bounds; the capacity check is bypassed because it occurs after the newline test. Trigger: low memory or a help response filling the allocation exactly. Consequence: crash or heap corruption.

### D26 — ELF loader trusts unbounded section indexes in an executable file [high impact, high confidence]

`rom/dos/internalloadseg_elf.c:1780-1787`: each relocation section uses `sh[sh[i].info]` without verifying `info < int_shnum`. `load_header` checks ELF identity/class/type/machine (`:195-223`) but not this index. Trigger: malformed relocatable ELF supplied to `LoadSeg`/`NewLoadSeg`. Consequence: out-of-bounds section-header read and possible fault while loading untrusted program content.

### D27 — ELF section-table byte count can wrap before allocation [high impact, high confidence]

`rom/dos/internalloadseg_elf.c:1595-1602,1839`: `int_shnum * eh.shentsize` is computed without checking multiplication overflow; `read_shnum` can accept the extended count from the file (`:161-188`). Trigger: crafted section count and entry size in an ELF file. Consequence: too-small allocation/read followed by iteration through `int_shnum` headers beyond the buffer. Applies to ELF load paths; whether the overflow is reachable depends on ELF structure widths and target word size.

### D28 — Handler startup dereferences a failed process creation [high impact, high confidence]

`rom/dos/runhandler.c:131-164`: `CreateNewProcTags` may return `NULL`, but `RunHandler` continues to allocate the reply objects and calls `PutMsg(&(process->pr_MsgPort), ...)`. Trigger: low memory or another process-creation failure while mounting/starting a handler. Consequence: null dereference rather than a clean device-not-mounted error. Failures creating the reply port or packet after a successful process start also leave the child waiting for a startup packet.

### D29 — Handler fallback retains the temporary `L:` directory lock [low to medium impact, high confidence]

`rom/dos/runhandler.c:62-74`: the fallback gets `dir = Lock("L:")`, switches to it with `CurrentDir(dir)`, then restores the old current directory but never releases `dir`. Trigger: a handler load that falls back to `L:`. Consequence: one lock leak per fallback attempt; on removable volumes this can also keep a volume busy.

### D30 — `CreateNewProc` dereferences a null process during allocation-failure cleanup [high impact, high confidence]

`CreateNewProc` jumps to `enomem` when both attempts to allocate its `Process` structure fail (`rom/dos/createnewproc.c:226-238`). That label calls `freeLocalVars(process, DOSBase)` without checking `process` (`:594-604`), and the helper immediately traverses `process->pr_LocalVars` (`:674-680`). Trigger: the initial process-structure allocation fails, including the fallback on 64-bit builds. Consequence: the intended null return and `ERROR_NO_FREE_STORE` path faults while dereferencing a null process pointer. Only call the local-variable cleanup after a `Process` was allocated and its list initialized. Source-only; no allocation failure was induced.

### D31 — CON: handler uses failed signal allocations as message-port bit numbers [medium impact, high source confidence for signal exhaustion; selected CON:/RAW:/AUX: handler]

`rom/filesys/console_handler/con_handler.c:565-581` allocates two reply ports and assigns each `AllocSignal(-1)` result to `mp_SigBit` without checking it. Exec returns `-1` when no signal bit is available (`rom/exec/allocsignal.c:45-48,77-97`). The read port's bit is shifted into the handler's `Wait` mask (`con_handler.c:759-777`); the write port is used as the reply port for console writes (`:312,371`). Trigger: the handler process has too few free signal bits during open. Consequence: a negative bit reaches a shift or reply-port signal operation, so reads or writes may wait on the wrong mask or fail to wake. Check both allocations, unwind any bit already allocated, and fail the open cleanly. The ordinary handler closes its process at `:1259-1271`, so the absence of explicit `FreeSignal` for these two bits is not claimed as a persistent leak. Source-only; signal exhaustion was not reproduced.

### D32 — `SystemTagList` dereferences a failed startup-extension allocation [high impact, high source confidence under low memory; selected DOS and Workbench CLI]

After `CreateNewProc(proctags)` succeeds, `SystemTagList` allocates a `struct ExtArg` with `AllocMem` and immediately writes `ea->ea_CES` and `ea->ea_Flags` without checking whether allocation returned null (`rom/dos/systemtaglist.c:402-414`). The selected Workbench CLI launcher reaches this path through its asynchronous `SystemTagList("", tags)` call (`workbench/system/CLI.c:173-187`); other callers use the same public DOS function. Trigger: enough memory remains to create the shell process but the subsequent `ExtArg` allocation fails. Consequence: the caller faults on a null write after process creation, instead of receiving a failure result, and the newly created child is left without the startup packet prepared below this allocation. Check `ea` and unwind the created process and packet on failure. This is distinct from D21's earlier command-copy allocation. Source-only; allocation failure was not induced.

### D33 — ExecSMP late-assign promotion can leak a target lock [medium impact, high source confidence under concurrent first use]

`GetDeviceProc()` holds `LockDosList(LDF_ALL | LDF_READ)` while resolving a `DLT_LATE` entry (`rom/dos/getdeviceproc.c:191-218`). Its promotion block uses `Forbid()` as the serialization point and, if the entry still appears late with `dol_Lock == BNULL`, stores the newly acquired target lock in `dl->dol_Lock` (`:246-259`). `LockDosList` implements `LDF_READ` with `ObtainSemaphoreShared()`, so multiple readers can execute this branch concurrently (`rom/dos/lockdoslist.c:60-65`). Exec documents that on ExecSMP, `Forbid()` blocks task switches only on the calling processor; shared data still needs cross-CPU synchronization (`rom/exec/forbid.c:58-60`). `Lock()` permits multiple shared locks and returns a lock handle, while `UnLock()` sends `ACTION_FREE_LOCK` for that handle (`rom/dos/lock.c:35-60`; `rom/dos/unlock.c:35-66`). If two CPUs both pass the `DLT_LATE`/`BNULL` checks before either store is visible, each can assign its own successful `Lock()` result to `dol_Lock`; the last store overwrites the other pointer. Neither branch unlocks the local lock when it observed `dol_Lock == BNULL`, so the overwritten FileLock is leaked. Trigger: concurrent first `GetDeviceProc()` calls for the same late assign on an ExecSMP build, with both target `Lock()` calls succeeding. Consequence: an untracked lock reference remains held and may keep the target/volume busy. The single-CPU serialization remains effective. No concurrent lookup was run; source-only.

## Review limitations

These are candidates inferred from code and API comments; none were executed. A handler may mask or amplify a failure (for example by always making progress on writes). The `D18` result convention is especially dependent on the handler implementation and should be confirmed against concrete handlers before prioritizing it as a fix. This report does not attempt to duplicate findings in the disk or filesystem reports.


<a id="d34"></a>
### D34 — DOS64 position operations fail when NTFS declines 64-bit packets without an error [medium impact, high confidence]

`rom/dos64/dos64_support.c:27-53` falls back to 32-bit `ACTION_SEEK` only when `dos64_UnsupportedPkt()` recognizes an unsupported response. That helper requires a false result and a nonzero error (`dos64_intern.h:66-68`). NTFS initializes each packet's `err` to zero (`workbench/fs/ntfs/packet.c:45`) and its `ACTION_GET_FILE_POSITION64` and `ACTION_CHANGE_FILE_POSITION64` cases set `res = DOSFALSE` without setting `err` (`:369-381`); the packet loop returns that zero as `dp_Res2` (`:893-899`). On 32-bit AROS, `dos64_Seek()` therefore accepts the zero GET result as a valid old position, then returns `-1` when the zero CHANGE result fails its Boolean check, without trying legacy `ACTION_SEEK`. On 64-bit AROS, NTFS first rejects `ACTION_SEEK64` through its unknown-action path, then the same false-with-zero-error position-packet pair prevents the 32-bit fallback. `GetFilePosition64()` also calls this helper with `OFFSET_CURRENT` (`rom/dos64/getfileposition64.c:60-61`).

**Trigger and consequence:** call `Seek64()` or `GetFilePosition64()` on an NTFS filehandle, including a position representable by legacy `Seek()`. DOS64 reports failure even though NTFS implements ordinary `ACTION_SEEK`. On 32-bit builds the completed packet also leaves global `IoErr()` unchanged, covered separately by D17. **Affected configuration:** AROS builds containing `dos64.library` (including Raspberry Pi ARM and AArch64 package manifests) when the explicitly buildable optional `workbench-fs-ntfs` handler is built and installed; its parent filesystem aggregate edge is commented (`arch/arm-raspi/boot/mmakefile.src:180`, `arch/aarch64-raspi/boot/mmakefile.src:213`; `workbench/fs/ntfs/mmakefile.src:3,13`). **Confidence:** high for the packet result/error values and DOS64 fallback decision; no seek or mounted NTFS volume was exercised. Return an unsupported-action error for unimplemented 64-bit packets, or implement the position operations. Source-only.

<a id="d35"></a>
### D35 — DOS64 reports successful file-size changes on read-only NTFS [medium impact, high confidence]

`rom/dos64/dos64_support.c:156-182` treats any `ACTION_CHANGE_FILE_SIZE64` result other than `DOSFALSE` as success, unless the response is recognized as unsupported. The selected NTFS handler defines `NTFS_READONLY` in `ntfs_fs.h:34`; its `ACTION_CHANGE_FILE_SIZE64` case consequently puts `ERROR_DISK_WRITE_PROTECTED` (214) in `dp_Res1` but leaves `dp_Res2` at its packet-loop default of zero (`workbench/fs/ntfs/packet.c:45,417-437,893-899`). This result is neither `DOSFALSE` nor the unsupported response expected by DOS64, so `dos64_SetFileSize()` accepts it as success. `ChangeFileSize64()` calls the helper with `wantsize == FALSE` and returns `DOSTRUE`; `SetFileSize64()` also returns a non-error value, but its follow-up query is affected separately by D36 (`rom/dos64/changefilesize64.c:60-63`; `setfilesize64.c:64-66`).

**Trigger and consequence:** call either DOS64 size-change API on an NTFS filehandle. A read-only rejection is reported as success, and the file remains unchanged. `ChangeFileSize64()` returns true; `SetFileSize64()` also returns a non-error value, but its follow-up size query is separately affected by D36, so this finding does not claim that the returned value is the unchanged size. On 64-bit builds the helper first tries `ACTION_SET_FILE_SIZE64`; NTFS rejects that unknown action, then the `CHANGE_FILE_SIZE64` response is misclassified. On 32-bit builds it sends the change packet directly. **Affected configuration:** any build containing dos64.library with the optional NTFS handler; NTFS's read-only macro is defined in its checked-in header, and the handler target is explicitly buildable (`workbench/fs/ntfs/mmakefile.src:5-15`). **Confidence:** high for the result values and helper/API decisions; no file resize or NTFS volume was used. Have the handler return a DOS failure result with a nonzero write-protected error, and make DOS64 accept only the documented success value. Source-only.

<a id="d36"></a>
### D36 — NTFS returns the file-size field address as the DOS64 size result [medium impact, high confidence]

`workbench/fs/ntfs/packet.c:455-466` handles `ACTION_GET_FILE_SIZE64` by setting `res = (IPTR)&fl->gl->size`. The packet loop publishes this as `dp_Res1` (`:893-899`). DOS64's `dos64_GetFileSize()` treats the packet result itself as the file size and returns it directly on both 64-bit and 32-bit paths (`rom/dos64/dos64_support.c:111-123`); the public `GetFileSize64()` returns that helper result (`rom/dos64/getfilesize64.c:56-61`). The field is a `UQUAD` file size (`workbench/fs/ntfs/ntfs_fs.h:165`). The selected PFS3 OS4 handler confirms the packet's expected result shape: `dd_GetFileSize64()` assigns `GetDEFileSize(...)` directly to the 64-bit result field (`rom/filesys/pfs3/fs/dd_funcs.c:2296-2303`).

**Trigger and consequence:** call `GetFileSize64()` on an NTFS filehandle. DOS64 returns the address of NTFS's size field as if it were the file's byte count; on 32-bit builds the ordinary packet result also truncates that address rather than returning the overlay's 64-bit size. `SetFileSize64()` uses the same query after its change packet and therefore also returns a non-size value on this path (in addition to D35's false-success result). **Affected configuration:** dos64.library with the optional NTFS handler; the handler target is explicitly buildable (`workbench/fs/ntfs/mmakefile.src:5-15`). **Confidence:** high for the packet result, helper handling, and contrasting PFS3 implementation; no file query or NTFS volume was used. Return the file-size value in the packet result according to the 64-bit packet contract. Source-only.
