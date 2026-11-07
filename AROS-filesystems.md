# AROS filesystem subsystem source review

Source-only review of commit `282a0454356dc2b9554ab023997d7f7ca24fc03e`. This report is a candidate list: source paths and control-flow traces substantiate each entry, but none was reproduced at runtime. Project code and media were not changed, built, or run.

## Scope and method

The repository-wide inventory located 186 C files (about 116,600 source lines) in filesystem and handler trees across ROM, Workbench, SMBFS, and hosted platforms, plus the `rom/filesystem` facade and Sam440 ext2 boot reader. The review traced build manifests, packet and block-I/O contracts, mount geometry, cache state, allocation/free-space logic, malformed-media parsing, read/write lengths, metadata ordering, lifecycle and optional variants. It used source reading and targeted searches, not runtime tests or a formal proof of completeness. The disk/block report covers the underlying drivers and partition/boot plumbing; this report follows their filesystem-facing contracts.

| Area | Build path and coverage | Findings |
|---|---:|---:|
| FAT | ROM handler and FAT BPB helper; cache, cluster chain, directories, metadata packets | 16 |
| NTFS | Workbench Storage handler (`NTFS_READONLY`); MFT, runlists, directory index, packet/cache paths | 13 |
| AFS/FFS | ROM handler; volume mount, validator, cache, files and enumeration | 6 |
| PFS3 | Three ROM handler variants plus PFSDoctor; LRU, allocation, metadata, repair I/O | 13 |
| SFS | ROM handler plus SFScheck/SFSDefrag and formatter interface; transactions, bitmap, device I/O | 6 |
| Optical | Native CDVDFS ISO/Joliet/HFS/Rock Ridge and separately defined CDFS ISO handler | 9 |
| RAM and DOS handlers | RAM, CON, PIPE, AUX, debug, port, speak, and `rom/filesystem` facade | 9 |
| Hosted filesystem | Shared emul handler and Unix/Windows/Android backends | 4 |
| SMBFS | Optional network share handler and SMB transport/metadata | 4 |
| Sam440 ext2 boot reader | Parthenope filesystem-facing callback; RDB integration in disk report | 1 |

Counts are the number of findings primarily attributed to each area; a finding can trace supporting code in another area. CDVDFS is selected by native boot manifests. The independent `kernel-fs-cdfs` target has no dependency in the checked source manifests, so CDFS findings are scoped to builds that include it. NTFS `file.c` is explicitly disabled by its build file; AFS Unix support and PFS3 legacy mount helpers are not part of the active handler object lists. SMBFS, PFSDoctor and SFSDefrag are separate optional program targets. Hosted backends apply only to their platform families.

## Findings

### FS-001 — FAT cache constructor dereferences a failed allocation

- **Source:** `rom/filesys/fat/cache.c:74–145, 148–160`; `rom/filesys/fat/volume.c:189–195`.
- **Trigger/trace:** `Cache_CreateCache` writes `b->use_count`, `b->state` and `b->data` at lines 122–125 before checking whether `AllocVec` returned NULL at line 127. If the initial cache-structure `AllocVec` fails, the common failure path calls `Cache_DestroyCache(c)` with `c == NULL`, and that function immediately calls `Cache_Flush(c)`.
- **Consequence:** A memory-pressure allocation failure can fault the FAT handler during volume mount instead of returning `ERROR_NO_FREE_STORE`. The later `sb->cache == NULL` check cannot contain either crash.
- **Confidence/limits:** High source confidence. Requires allocation failure; no runtime reproduction. Both null paths share the same constructor failure-handling root cause.

### FS-002 — FAT cluster accounting changes even when the FAT entry write fails

- **Source:** `rom/filesys/fat/fat.c:245–285, 350–370`; `rom/filesys/fat/file.c:283–345`; `rom/filesys/fat/cache.c:199–250`.
- **Trigger/trace:** `AllocCluster` and `FreeCluster` discard the Boolean result of `SET_NEXT_CLUSTER`, then decrement/increment `free_clusters` and dirty FSInfo. `WriteFileChunk` also ignores the result when linking the previous cluster to an allocated successor. `GetFatEntryPtr` can fail when `Cache_GetBlock` cannot supply a block, and every `SetFat*Entry` reports that failure.
- **Consequence:** On cache/I/O failure, in-memory free-space accounting and potentially persisted FSInfo can diverge from the FAT. Extending a file may report successful bytes in the newly allocated cluster without a valid predecessor link, orphaning content after remount.
- **Confidence/limits:** High source confidence; requires a cache or underlying I/O failure. Flush behavior can surface a separate later error, but does not roll back the accounting or chain mutation.

### FS-003 — FAT block access accepts successful but short device transfers

- **Source:** `rom/filesys/fat/disk.c:214–266`; `rom/filesys/fat/cache.c:218–244, 298–336`.
- **Trigger/trace:** `AccessDisk` checks only the `DoIO` return error, not `io_Actual` against `chunk * block_size`, before advancing `num` and `data` by the full requested amount. Cache loads and flushes treat its zero return as a complete range.
- **Consequence:** A driver returning zero error with a short transfer can feed incomplete sector data into filesystem parsing or cause a partially written range to be marked clean. The exact effect depends on the driver and which sectors were transferred.
- **Confidence/limits:** High source confidence, conditional on that device response. No assertion that conforming block drivers routinely return short successful I/O.

### FS-004 — FAT volume rename can use an uninitialized directory entry on allocation failure

- **Source:** `rom/filesys/fat/volume.c:760–834`; `rom/filesys/fat/direntry.c` (`AllocDirEntry`, `UpdateDirEntry`).
- **Trigger/trace:** `SetVolumeName` attempts `AllocDirEntry` when no existing volume-ID entry is found. If allocation returns an error, `err != 0` skips the name initialization, but lines 811–817 unconditionally copy `de.e.entry.name` into the boot block and line 819 writes it to disk. A failure from `UpdateDirEntry` returns at line 807 without freeing the boot buffer or releasing the directory handle.
- **Consequence:** Directory-allocation failure can persist uninitialized stack bytes as the boot-sector volume label. A metadata update failure leaks resources.
- **Confidence/limits:** High source confidence for the unchecked path; requires a root-directory allocation/update error. The exact boot write may itself fail and is ignored separately.

### FS-005 — FAT mount can publish a volume without its required volume info

- **Source:** `rom/filesys/fat/volume.c:919–1075, 1087–1128`.
- **Trigger/trace:** On `CreatePool` or `AllocVecPooled` failure, `vol_info` or `newvol` can remain NULL. The branch nevertheless sets `sb->info = vol_info`, calls `SendVolumePacket(newvol, ACTION_VOLUME_ADD, glob)`, and returns mount success. `AttemptDestroyVolume` and `FillDiskInfo` subsequently dereference `sb->info`.
- **Consequence:** Memory pressure during mount can publish an unusable volume and lead to a handler fault on ordinary status or removal paths. If the DOS-list allocation succeeds while the info allocation fails, its lock-list pointer is also null.
- **Confidence/limits:** High source confidence; requires allocation failure. The recipient of `ACTION_VOLUME_ADD` was not yet used to claim a particular additional failure.

### FS-006 — FAT cache can accept a BPB sector size different from the device block size

- **Source:** `rom/filesys/fat/volume.c:77–135, 189–233`; `compiler/fatbpb/bpb.c:44–69, 72–154`; `rom/filesys/fat/disk.c:217–231`.
- **Trigger/trace:** The boot sector is read with the DosEnvec `de_SizeBlock * 4`, while `FAT_ParseBPB` accepts any plausible 512/1024/2048/4096-byte BPB sector size and never compares it with that device size. The handler then creates a cache and computes all block-device byte offsets using the BPB size.
- **Consequence:** A plausible malformed or mismatched BPB can make filesystem logical sectors address the wrong device byte positions, including writes. Media-size divisibility and BPB tolerance reject some combinations, but not all equal-total-byte layouts.
- **Confidence/limits:** High source confidence for the absent equality check and offset dependency. Requires a mount with mismatched BPB/device sector sizes; the exact driver rejection behavior is provider dependent.


### FS-007 — NTFS cache constructor has copied failure checks and an additional wrong pointer test

- **Source:** `workbench/fs/ntfs/cache.c:70–136, 138–151`; `rom/filesys/fat/cache.c:74–145`.
- **Trigger/trace:** The NTFS cache constructor also dereferences each block allocation before the null check and calls its destructor with `c == NULL` after the initial allocation fails. Additionally, after allocating `c->blocks`, it checks `if (c == NULL)` instead of `if (c->blocks == NULL)` and enters the block loop with a null pointer table.
- **Consequence:** Memory pressure can fault NTFS mount at any of these allocations instead of reporting an error. The `c->blocks` typo creates a distinct failure manifestation, grouped here with the copied constructor root cause.
- **Confidence/limits:** High source confidence. Requires allocation failure; NTFS is a Workbench `Storage` program, not a default ROM handler.

### FS-008 — NTFS update-sequence fixups trust an unchecked media offset

- **Source:** `workbench/fs/ntfs/ntfs.c:68–126, 1152–1183`; `workbench/fs/ntfs/ntfs_struct.h` (record header).
- **Trigger/trace:** `PostProcessMFTRecord` derives `seqarray` from on-disk `usa_offset` and reads its first UWORD, then advances and reads one UWORD for each sector. It checks `usa_count - 1 == len`, but does not verify `usa_offset + 2 * usa_count` lies within the allocated MFT record, or that the record has enough bytes for the struct header before reading it. It is called on the first MFT record before `ValidateMFTRecord` is applied to later records.
- **Consequence:** An NTFS volume with a forged update-sequence offset can make mount/read processing access past the MFT record allocation or apply bytes from unrelated memory.
- **Confidence/limits:** High source confidence; requires malformed on-disk metadata. The exact fault or disclosure depends on memory layout and architecture.

### FS-009 — NTFS resident attribute read checks value length without checking its enclosing attribute

- **Source:** `workbench/fs/ntfs/ntfs.c:255–300`.
- **Trigger/trace:** `ReadMFTAttribData` rejects `value_offset > attrentry->length` and `ofs + len > value_length`, but it does not check `value_offset + value_length <= attrentry->length`, nor that `attrentry->length` itself remains inside the validated MFT record. It then copies from `attrentry + value_offset + ofs`.
- **Consequence:** A malformed resident attribute can direct reads beyond its own record or into another attribute and return unrelated memory as file/metadata content.
- **Confidence/limits:** High source confidence for the local check gap. Reachability depends on the caller-provided record and attribute parser; separate attribute-iterator bounds are reviewed below.

### FS-010 — NTFS runlist parser has no attribute-end bound

- **Source:** `workbench/fs/ntfs/ntfs.c:138–233, 303–309, 603–759`.
- **Trigger/trace:** `ReadNTFSRunList` reads the run header, then consumes up to 15 length and 15 offset bytes via `GetMappingPairPos` without an end pointer or a check that the mapping pair fits in its enclosing attribute. `FindMFTAttrib` likewise advances by on-disk attribute-list lengths without validating each step, and a zero length can prevent progress.
- **Consequence:** Malformed NTFS metadata can make file reads walk beyond an attribute/record allocation, or hang while searching an attribute list.
- **Confidence/limits:** High source confidence for missing bounds and progress checks; effects depend on malformed media. Grouped as one missing bounded-parser contract, though the iterators have separate entry points.

### FS-011 — NTFS nonresident reads omit a partial final sector

- **Source:** `workbench/fs/ntfs/ntfs.c:390–506`.
- **Trigger/trace:** In the uncompressed path, `lastblock = ((blockend + skipfirst) >> SECTORSIZE_SHIFT)` rounds down. For a read starting at a sector boundary with `len = 513`, `lastblock` becomes 1; the loop visits only its final-sector branch and sets the copy count to `blockend & 511`, or one byte. It then returns success with the rest of the requested output untouched. Other requests spanning a partial final sector can similarly leave a caller buffer tail untouched.
- **Consequence:** File reads can return success with incomplete or stale output bytes. A zeroed destination may hide the omission.
- **Confidence/limits:** High source confidence for the arithmetic example, conditional on a nonresident uncompressed attribute reaching this path. No executable test was run.

### FS-012 — AFS flush discards failed writes and reports success

- **Source:** `rom/filesys/afs/cache.c:85–98, 354–378`; `rom/filesys/afs/os_aros_support.c:581–627`.
- **Trigger/trace:** `flushCache` calls `writeDisk` but clears `BCF_WRITE` regardless of its return. `writeBlock` does the same and always returns `DOSTRUE`. `flush` ignores both the cache-write result and `CMD_UPDATE` result, clears its cache, then returns `DOSTRUE`.
- **Consequence:** After an underlying write/update failure, dirty metadata or file data may be forgotten while the caller receives a successful flush/update. The lack of rollback is particularly relevant for bitmap/root-block ordering.
- **Confidence/limits:** High source confidence; requires a device write/update error. Driver behavior and actual persistence are not inferred.

### FS-013 — AFS root-block disk name is copied past 32-byte buffers

- **Source:** `rom/filesys/afs/os_aros_support.c:71–108, 162–199`; `rom/filesys/afs/afsblocks.h:34–35`.
- **Trigger/trace:** `initDeviceList` obtains an on-disk length byte from the root-block name and copies that many bytes into a 32-byte `AllocVec` buffer without a cap. `createDosVolume` then copies the same BSTR length into `char string[32]` and writes a terminating zero at `string[length]`. Neither routine checks the media length against 31.
- **Consequence:** A malformed root-block name longer than 31 bytes can corrupt the heap and stack during mount.
- **Confidence/limits:** High source confidence; malformed media required. A standard formatter may write only valid names, but that does not constrain externally supplied volumes.

### FS-014 — AFS cache allocation underflows its loop count for zero buffers

- **Source:** `rom/filesys/afs/cache.c:27–62`; `rom/filesys/afs/volumes.c:186–203`.
- **Trigger/trace:** `initVolume` passes `de_NumBuffers` directly to `initCache`. If it is zero, the allocation size is zero, while the loop condition `i < numBuffers - 1` underflows the unsigned `ULONG` count and traverses beyond the allocation whenever a non-NULL pointer is returned. There is no local positive-minimum check.
- **Consequence:** A mount configuration with zero buffers can corrupt memory during handler startup.
- **Confidence/limits:** High source confidence for the arithmetic, conditional on a zero-buffer DosEnvec and non-NULL zero-byte allocation. External mount validation, if any, could reject that configuration before the handler.

### FS-015 — PFS3 clears dirty data-cache slots before their write succeeds

- **Source:** `rom/filesys/pfs3/fs/disk.c:1215–1265, 1279–1310`; `rom/filesys/pfs3/fs/update.c:180–204`.
- **Trigger/trace:** `UpdateSlot` clears every adjacent slot's `dirty` bit in its gathering loop, then calls `RawWrite` without checking its result. The caller also discards any result because the function returns `void`. `CachedRead` and `CachedWrite` can evict through this path; `UpdateDisk` invokes it before metadata commit.
- **Consequence:** A block write failure can cause dirty file data to be forgotten and a later metadata/root update to proceed as if the data write completed.
- **Confidence/limits:** High source confidence; requires a failed raw write. The disk layer may ask for retry, but cancellation/error still reaches this unchecked path.

### FS-016 — PFS3 caches a failed block read and later treats it as valid

- **Source:** `rom/filesys/pfs3/fs/disk.c:1215–1245, 1356–1377`.
- **Trigger/trace:** On a cache miss, `CachedRead` puts `RawRead`'s error in `*error` but unconditionally assigns the requested block number to the slot. `DiskRead` copies its contents regardless of that error. A later `CheckDataCache` hit on the same block sets `*error = 0` and returns the slot without retrying I/O.
- **Consequence:** After a read failure, subsequent reads of the same block can return stale or partially initialized cache contents as successful file data.
- **Confidence/limits:** High source confidence. Requires a read error and a subsequent access before slot eviction; the first read itself still returns the error.

### FS-017 — PFS3 can report a successful root commit after its writes fail

- **Source:** `rom/filesys/pfs3/fs/update.c:259–292`; `rom/filesys/pfs3/fs/disk.c:1748` (`WriteRootBlocks`), `2238` (`UpdateAndMotorOff`).
- **Trigger/trace:** `UpdateDisk` discards `WriteRootBlocks`' nonzero error result, increments the root datestamp, clears `rootblockchangeflag`, calls the void update/motor function, then sets `success = TRUE`. It clears the global dirty flag after both success and failure branches.
- **Consequence:** A failed root-block write or update can be acknowledged as a committed filesystem state and lose the retry signal, leaving metadata durability uncertain.
- **Confidence/limits:** High source confidence for unchecked result paths. Requires a write or update failure; exact on-media state depends on earlier successful writes and device behavior.

### FS-018 — PFS3 trackdisk loop can stall on sub-block MaxTransfer

- **Source:** `rom/filesys/pfs3/fs/disk.c:1579–1642` and `rom/filesys/pfs3/fs/mmakefile.src` (trackdisk variant enabled).
- **Trigger/trace:** `RawReadWrite_TD` sets `io_transfer` to the lesser of remaining length, maximum transfer and DosEnvec `de_MaxTransfer`, then rounds it down to a filesystem block size. If the effective maximum is smaller than one block, `io_transfer` becomes zero. The loop issues a zero-length request and never reduces `io_length`.
- **Consequence:** A mount configuration advertising a smaller-than-block transfer maximum can hang a read or write request.
- **Confidence/limits:** High source confidence for the loop. Requires that configuration; normal mount settings generally provide larger limits.

### FS-019 — SFS treats successful but short block transfers as complete

- **Source:** `rom/filesys/SFS/FS/deviceio.c:665–703, 789–841`; `rom/filesys/SFS/FS/cachedio.c:515–530`.
- **Trigger/trace:** Both buffered and direct `transfer` loops test only `DoIO`'s error code. They never compare `io_Actual` or `scsi_Actual` with the requested bytes before advancing `blockoffset` and `buffer`. The cache copyback path clears dirty bits after `transfer` returns zero.
- **Consequence:** If a block provider returns zero error but fewer bytes than requested, SFS can accept incomplete read data or mark an incompletely persisted cache range clean.
- **Confidence/limits:** High source confidence, conditional on a short successful device response. Normal full-count devices do not trigger it.

### FS-020 — SFS update failures are invisible to transaction and flush callers

- **Source:** `rom/filesys/SFS/FS/deviceio.c:54–60`; `rom/filesys/SFS/FS/cachedio.c:535–565`; `rom/filesys/SFS/FS/transactions.c:868–950`.
- **Trigger/trace:** `update` sends `CMD_UPDATE` via `DoIO` but returns `void` and discards the result. `flushiocache` calls it only after data-cache copyback succeeds, then returns the earlier zero error. The transaction path uses that zero to proceed through commit stages.
- **Consequence:** A provider-side update/barrier failure cannot be reported as an SFS transaction or flush failure, weakening the intended ordering/durability guarantee.
- **Confidence/limits:** High source confidence; requires a device that reports an update failure. Actual persistence depends on provider semantics; this does not claim every CMD_UPDATE implementation flushes hardware caches.

### FS-021 — CDFS indexes a 2048-byte sector with an absolute directory offset

- **Source:** `rom/filesys/cdfs/iso9660.c:316–349, 352–406`.
- **Trigger/trace:** `isoReadDir` requests sector `extent + offset / 2048`, then casts `&buff[offset]` to a directory record rather than using `offset % 2048`. The caller accumulates `offset` across the directory and calls this routine again for entries beyond the first sector.
- **Consequence:** Enumerating or locating entries after the first 2048 bytes reads outside the cached sector and can misparse adjacent memory or fault.
- **Confidence/limits:** High source confidence. Requires a directory spanning more than one sector; the first sector alone can appear functional.

### FS-022 — CDFS path-table parser trusts entry lengths and parent indexes

- **Source:** `rom/filesys/cdfs/iso9660.c:118–124, 450–533, 352–377, 709–712`.
- **Trigger/trace:** The path-table count loop reads `dp[0]` and advances by its padded length while only checking `size > 0`; it does not verify a full header or that the next entry fits the remaining allocation. A malformed length can step beyond `data`, and the second pass then stores out-of-allocation pointers. Path-table parent indexes are later used as `Entry[ParentDirectory-1]` without range checks.
- **Consequence:** A malformed ISO path table can cause out-of-bounds reads and invalid pointer dereferences during mount or navigation.
- **Confidence/limits:** High source confidence; requires malformed optical media/image. The other CD handler has a separate parser.

### FS-023 — CDFS unmount leaks its path-table allocations and reports an error

- **Source:** `rom/filesys/cdfs/iso9660.c:450–533, 648–654`.
- **Trigger/trace:** Successful path-table loading allocates `iv_PathTable.Data` and `.Entry` separately. `ISO9660_Unmount` frees only `vol->cv_Private` and returns `ERROR_NO_FREE_STORE` unconditionally.
- **Consequence:** Mount/unmount cycles leak both buffers, and a successful unmount is reported as an allocation error. Discs without a path table avoid the allocation part.
- **Confidence/limits:** High source confidence; active `cdfs` handler build.

### FS-024 — CDVDFS Rock Ridge field walker can loop forever or read beyond a record

- **Source:** `rom/filesys/CDVDFS/src/rock.c:95–159, 168–202`.
- **Trigger/trace:** `Get_System_Use_Field` advances by on-disc `slen` without checking `slen >= 4` or `system_use_pos + slen <= length`. A zero length leaves the scan position unchanged. A matching entry copies up to the caller-buffer size, not up to bytes remaining in the directory record. `Get_RR_File_Name` subtracts five from an unchecked field length, making a too-short NM field yield a negative `len` passed to `CopyMem` after integer conversion.
- **Consequence:** Crafted ISO Rock Ridge extensions can hang directory lookup or cause out-of-bounds reads/large invalid copies.
- **Confidence/limits:** High source confidence; malformed Rock Ridge media/image required. The exact copy/fault depends on the entry and caller buffer length.

### FS-025 — RAM filesystem ExNext dereferences a failed examination allocation

- **Source:** `rom/filesys/ram/commands.c:1357–1394`; `rom/filesys/ram/handler.h:135–139`.
- **Trigger/trace:** On the first `CmdExamineNext` call, `AllocPooled` returns an `Examination` pointer. The code immediately passes it to `AddTail` and stores `examination->next_object` without checking NULL. The adjacent `CmdExamineAll` path does check its equivalent allocation.
- **Consequence:** Memory pressure during a directory enumeration can fault the RAM handler rather than returning an allocation error.
- **Confidence/limits:** High source confidence; allocation failure required. This is an active RAM handler path.

### FS-026 — PIPE handler can leave zero-length I/O requests pending

- **Source:** `workbench/fs/pipe/pipesched.c:75–121, 141–199`; `workbench/fs/pipe/pipebuf.c:81–121, 131–171`.
- **Trigger/trace:** `StartPipeIO` places a zero-length read or write request on a waiting list. `CheckWaiting` only enters the reader loop while the pipe buffer is nonempty, and the writer loop while it is not full; it therefore does not reach the `len == 0` completion test for an empty-pipe read or a full-pipe write. Buffer movers themselves return zero for `amt <= 0`.
- **Consequence:** A zero-byte read on an empty pipe, or zero-byte write on a full pipe, can block until another client changes pipe state, although no bytes need to move.
- **Confidence/limits:** High source confidence. Requires the stated pipe state; nonempty reads and nonfull writes hit the completion branch.

### FS-027 — PIPE handler leaks an I/O waiter when the requested direction is invalid

- **Source:** `workbench/fs/pipe/pipesched.c:90–121`.
- **Trigger/trace:** `StartPipeIO` allocates `WAITINGDATA` before checking that the opened `PIPEKEY` permits the requested read/write direction. On a mismatch, `goto SPIOEXIT` replies to the packet but never frees `wd`.
- **Consequence:** Repeated invalid-direction I/O consumes handler memory until allocation pressure affects ordinary pipe requests.
- **Confidence/limits:** High source confidence; requires a client sending read to a write-only pipe or write to a read-only pipe. The DOS library may reject some such calls before they reach the handler, so raw packet clients are the certain trigger.

### FS-028 — debug-handler prints an unterminated translated write buffer

- **Source:** `workbench/fs/debug/debug-handler.c:116–140` (ACTION_WRITE branch).
- **Trigger/trace:** For non-ASCII input, the handler copies every second input byte into `pa->outbuf[0..len/2-1]`, but does not append a NUL afterward. It initialized only `outbuf[0]` before the loop and then passes the buffer to `kprintf("%s", ...)`.
- **Consequence:** A binary write whose translated half contains no zero byte can make debug output read stale bytes beyond the translated content, potentially past the fixed output buffer.
- **Confidence/limits:** High source confidence. Requires the non-ASCII branch and a nonzero translated length; exact over-read depends on prior buffer contents.

### FS-029 — port handler remains in its packet loop after startup failure

- **Source:** `workbench/fs/port/port-handler.c:370–405, 490–499`.
- **Trigger/trace:** If `OpenLibrary("utility.library")` fails or `decodeStartup` returns an error, the handler replies to the startup packet with failure but still enters the main `do` loop and waits for another packet. It does not cleanly return or close the library on that failure path; the eventual common exit calls `CloseLibrary(pb.pb_UtilityBase)` even when it is NULL.
- **Consequence:** A failed mount can leave a dormant handler process/resource behind; a later forced `ACTION_DIE` may reach invalid library-close input.
- **Confidence/limits:** High source confidence for the missing exit; conditional on startup failure. Whether DOS terminates the process externally after the failed reply needs a caller contract check before asserting a persistent leak.

### FS-030 — CON: open path dereferences a failed filename allocation

- **Source:** `rom/filesys/console_handler/con_handler.c:140–151, 549–558`.
- **Trigger/trace:** `BSTR2C` returns NULL on `AllocVec` failure. `open_con` assigns the result to `fn` and immediately calls `isdosdevicec(fn)`, whose loop reads `s[b]` without a NULL guard.
- **Consequence:** Low-memory console open faults the handler instead of returning `ERROR_NO_FREE_STORE`.
- **Confidence/limits:** High source confidence; allocation failure required.

### FS-031 — CON: uses timer request and port without validating their creation

- **Source:** `rom/filesys/console_handler/con_handler.c:549–551, 764, 1153–1156`.
- **Trigger/trace:** `open_con` passes an unchecked `CreateMsgPort()` result to `CreateIORequest()`, then passes its unchecked result to `OpenDevice()`. Later `ACTION_WAIT_CHAR` dereferences `fh->timerreq` and the event loop dereferences `fh->timermp`. The `OpenDevice` return value is also ignored.
- **Consequence:** An allocation/open failure can crash during open or leave the handler running with unusable timeout handling.
- **Confidence/limits:** High source confidence for missing checks; exact fault point depends on Exec API behavior for NULL requests/ports. The path needs memory pressure or timer-device failure.

### FS-032 — Unix hosted filesystem reads one byte before checking the requested count

- **Source:** `arch/all-unix/filesys/emul_handler/emul_host.c:551–612`; `arch/all-hosted/filesys/emul_handler/emul_handler.c:855–868`.
- **Trigger/trace:** In the `FHD_STDIO` branch, `DoRead` calls host `read(..., buff++, 1)` before testing `if (res++ == len)`. A zero-length read therefore reads and writes one byte beyond a zero-length caller buffer if input is available; a length of N can consume N+1 bytes if `poll` keeps reporting readiness. The loop can also report N+1 bytes to `ACTION_READ`.
- **Consequence:** Buffer overrun and loss of a byte from the hosted standard-input stream; the subsequent consumer sees an incorrect read count.
- **Confidence/limits:** High source confidence for Unix hosted builds on the standard-input file handle. Reaching N+1 requires input to remain ready between iterations; zero-length input still attempts its first read.

### FS-033 — Unix hosted filesystem mishandles seek validation and success status

- **Source:** `arch/all-unix/filesys/emul_handler/emul_host.c:637–687`; `arch/all-hosted/filesys/emul_handler/emul_handler.c:884–896`.
- **Trigger/trace:** `DoSeek` stores the current host position in a 32-bit `LONG oldpos`. It then reuses `res` for `fstat`, so the `OFFSET_CURRENT` precheck computes `newpos = offset + res` (normally offset + 0) rather than offset + old position. It sets `*err = ERROR_SEEK_ERROR` unconditionally, including when `LSeek` succeeds.
- **Consequence:** Invalid seek bounds can pass or valid ones fail depending on the current position; successful `ACTION_SEEK` replies carry a spurious error code. On hosts with positions beyond 32-bit LONG range the old position also truncates.
- **Confidence/limits:** High source confidence; applies to Unix hosted filesystem. The exact impact of the nonzero secondary result on callers depends on DOS seek handling; the bounds precheck error is direct.

### FS-034 — Windows hosted stdin read scans beyond the bytes returned

- **Source:** `arch/all-mingw32/filesys/emul_handler/emul_host.c:328–391`; `arch/all-mingw32/filesys/emul_handler/emul_handler_native.c:50–56`.
- **Trigger/trace:** The asynchronous reader uses `ReadFile` into the caller's buffer and sets `reader->actual` to its byte count. After success, `DoRead` removes CRLF pairs with `while (*c)`, ignoring that byte count and never requiring a terminator. It also accesses `c[1]` at the last returned byte.
- **Consequence:** A successful read containing no NUL can scan and write past the supplied buffer, or process stale data beyond the returned count, while still reporting a length based on `reader->actual`.
- **Confidence/limits:** High source confidence for Windows hosted builds with `FHD_STDIO`. Requires a successful nonempty asynchronous read whose data lacks an early NUL.

### FS-035 — SMBFS copies a server-selected challenge length into an eight-byte field

- **Source:** `workbench/network/smbfs/source_code/proc.c:2314–2330`; `workbench/network/smbfs/source_code/include/smb/smb_fs_sb.h:41`.
- **Trigger/trace:** During NT1-or-newer negotiation, `smb_proc_connect` takes `server->crypt_key_length` directly from the server packet byte at `smb_vwv16 + 1`, then calls `memcpy(server->crypt_key, SMB_BUF(packet), server->crypt_key_length)`. The destination `crypt_key` array has eight bytes, and this branch contains no length cap.
- **Consequence:** A server response claiming more than eight challenge bytes overwrites adjacent `smb_server` fields, potentially corrupting the mounted filesystem process.
- **Confidence/limits:** High source confidence; requires a malformed or malicious SMB negotiation response. This is in the active SMBFS build.

### FS-036 — SMBFS can spin forever after a peer closes mid-packet

- **Source:** `workbench/network/smbfs/source_code/sock.c:136–151, 164–180, 537–545`.
- **Trigger/trace:** `smb_receive_raw` reads the advertised payload in a loop that advances `already_read` by the `recvfrom` result. It handles negative results but not zero. If the server closes the TCP connection after the four-byte NetBIOS header and before the full payload arrives, `recvfrom` can return zero repeatedly while `already_read < len` remains true.
- **Consequence:** The SMBFS process can consume CPU indefinitely and cease servicing filesystem packets.
- **Confidence/limits:** High source confidence. Requires an orderly peer close during a nonempty response body; applies to regular and raw SMB receives.

### FS-037 — SMBFS treats a short request send as a complete request

- **Source:** `workbench/network/smbfs/source_code/sock.c:393–450`.
- **Trigger/trace:** `smb_request` calls `send(sock_fd, buffer, len, 0)` once. Any nonnegative return, including fewer than `len` bytes, immediately proceeds to `smb_receive` without sending the remaining bytes or reporting the partial write.
- **Consequence:** Under a short socket send, the server receives an incomplete SMB frame while the client waits for a reply; the mount can stall or lose connection state.
- **Confidence/limits:** High source confidence for the unchecked result; a short successful send is the prerequisite.

### FS-038 — SFS soft-link creation copies an unbounded path into one metadata block

- **Source:** `rom/filesys/SFS/FS/filesystemmain.c:1415–1435`; `rom/filesys/SFS/FS/objects.c:859–879`; `rom/filesys/SFS/FS/objects.h:177–184`.
- **Trigger/trace:** The `ACTION_MAKE_LINK` soft-link target comes from packet `dp_Arg3` without a target-length check. `createobjecttags` allocates one admin/cache block for `fsSoftLink`, then copies bytes through the target NUL into its zero-length trailing `string` field without checking the remaining block size.
- **Consequence:** A link target longer than `globals->bytes_block - sizeof(struct fsSoftLink) - 1` overwrites the cache buffer/adjacent memory, potentially damaging filesystem metadata or the handler.
- **Confidence/limits:** High source confidence; requires an oversized soft-link target through the packet API. DOS callers may impose their own path limits, but the handler accepts raw packets with no local bound.

### FS-039 — AFS directory enumeration trusts on-disk string lengths over destination sizes

- **Source:** `rom/filesys/afs/filehandles3.c:128–147, 270–299`; `compiler/include/dos/dos.h:57, 81`.
- **Trigger/trace:** `getNextExamineBlock` copies the on-disk length-prefixed filename into `UBYTE cstr[34]` without checking the length. `examineNext` copies filename and comment byte counts from disk into fixed `FileInfoBlock` fields without bounding them. A damaged entry with a length byte above the destination capacity reaches these copies after the block is loaded.
- **Consequence:** Enumerating a malformed directory entry can overwrite stack memory or adjacent fields in the caller's `FileInfoBlock`, potentially crashing the handler.
- **Confidence/limits:** High source confidence for the missing bounds; requires malformed on-disk metadata or a path that admits an oversized entry. The block validator/checksum may reject some corrupted blocks, but a length-byte change with a valid checksum is sufficient to reach the copy.

### FS-040 — Sam440 ext2 boot reader dereferences a failed device allocation

- **Source:** `arch/ppc-sam440/boot/parthenope/src/ext2.c:67–96`.
- **Trigger/trace:** After validating the partition with `ext2fs_mount`, `ext2_create` assigns `malloc(sizeof(ext2_boot_dev_t))` to `boot` and immediately writes function pointers and geometry through `boot` without checking NULL.
- **Consequence:** Low memory during boot-device creation causes a firmware loader fault instead of a clean ext2 boot failure.
- **Confidence/limits:** High source confidence; applies only to the Sam440 Parthenope firmware boot reader, with a malloc failure required. The associated RDB and partition reader was already reviewed in the disk report; this entry covers only its ext2-facing creation path.

### FS-041 — CDFS converts ISO date digits by subtracting `'9'`

- **Source:** `rom/filesys/cdfs/iso9660.c:43–62`.
- **Trigger/trace:** `ascdec`, used by `ascdate2stamp` for the volume descriptor's ASCII year/month/day, adds `*val - '9'` for each digit instead of its value from `'0'`. For example, the two-digit string `"09"` becomes `-90` in unsigned arithmetic rather than 9.
- **Consequence:** ISO descriptor timestamps can be converted to incorrect `DateStamp` values; dates exposed for affected entries/volumes are wrong.
- **Confidence/limits:** High source confidence; applies when the ASCII date conversion path is used. This does not affect the binary seven-byte recording dates handled separately.

### FS-042 — CDFS Rock Ridge names and comments are copied only when too long

- **Source:** `rom/filesys/cdfs/iso9660.c:211–225, 252–260`.
- **Trigger/trace:** The `AS` comment and `NM` alternate-name branches place their `CopyMem` calls inside `if (len > remaining_capacity)`. Ordinary fields that fit never copy; oversized fields are truncated and copied. The `NM` capacity expression also uses `MAXCOMMENTLENGTH` for the filename field, although `fib_FileName` has its own capacity.
- **Consequence:** Normal Rock Ridge/Amiga extension names and comments are silently omitted, so directory listings show ISO names and empty comments even when extension metadata exists.
- **Confidence/limits:** High source confidence; requires an image with these extension fields. Oversized malformed fields may additionally overrun or read beyond their record, but this entry claims the directly demonstrated omission.

### FS-043 — CDVDFS HFS catalog parser copies unvalidated record lengths from disc nodes

- **Source:** `rom/filesys/CDVDFS/src/hfs.c:211–240, 247–299, 363–373`.
- **Trigger/trace:** `HFS_Load_Catalog_Record` indexes 512-byte node offset entries using `p_record` without checking the node's record count or offset bounds. It computes `len = sp[254-p_record] - start` from on-disc shorts and passes it to `CopyMem` into fixed `t_catalog_record`, with no nonnegative or destination-size check. Leaf readers likewise copy `node[pos]+1` bytes into fixed `leaf_rec` without checking the offset or length.
- **Consequence:** A malformed HFS catalog node can cause out-of-bounds reads and writes, including a huge unsigned copy when the computed signed length is negative.
- **Confidence/limits:** High source confidence; requires malformed HFS metadata on a disc. This is CDVDFS's HFS path, separate from its ISO/Joliet parser.

### FS-044 — CDVDFS ISO directory record can be copied past the sector boundary

- **Source:** `rom/filesys/CDVDFS/src/iso9660.c:421–460`; `rom/filesys/CDVDFS/src/generic.h:24–42`.
- **Trigger/trace:** `Get_Directory_Record` reads one 2048-byte sector, takes the record length byte at `p_offset & 2047`, and copies that many bytes from the sector buffer into a 256-byte record buffer. It never checks that the length fits before the end of the sector. A malformed length near byte 2047 can make `CopyMem` read beyond the loaded sector.
- **Consequence:** Corrupt ISO directory metadata can expose adjacent memory as a directory record, with subsequent parsing based on bytes that were not read from disc.
- **Confidence/limits:** High source confidence; requires a malformed record that straddles the logical sector. The length byte itself caps the destination write at 255, so this entry claims an out-of-bounds source read, not a direct destination overflow.

### FS-045 — NTFS directory enumeration dereferences a failed index-key allocation

- **Source:** `workbench/fs/ntfs/direntry.c:168–182`.
- **Trigger/trace:** When `de->key` is NULL, `GetDirEntry` calls `AllocMem(sizeof(struct Index_Key), ...)` and immediately writes `de->key->indx` and `de->key->pos` without checking the result.
- **Consequence:** Under memory pressure, listing a directory can crash the NTFS handler rather than returning `ERROR_NO_FREE_STORE`.
- **Confidence/limits:** High source confidence; active NTFS path, allocation failure required.

### FS-046 — NTFS nonresident directory bitmap length is read from a scaled pointer

- **Source:** `workbench/fs/ntfs/direntry.c:86–116`.
- **Trigger/trace:** After reading a nonresident `$BITMAP` attribute, the code sets `dh->ioh.bitmap_len = AROS_LE2LONG(*((ULONG *)(curattr + 0x30)))`. `curattr` is a `struct MFTAttr *`, so adding `0x30` advances by 48 structures, not 48 bytes. The resident branch correctly uses the attribute's value length and the nonresident branch already computed `bitmap_len` from `data_size`.
- **Consequence:** The handler reads an unrelated or out-of-bounds length, then `GetDirEntry` may scan beyond the allocated bitmap or enumerate an incorrect set of index blocks.
- **Confidence/limits:** High source confidence; requires a directory whose `$BITMAP` is nonresident. Exact neighboring memory contents determine the observed length.

### FS-047 — PFS3 can enter its packet loop without a notification port

- **Source:** `rom/filesys/pfs3/fs/init.c:188–198`; `rom/filesys/pfs3/fs/boot.c:300–305, 445–451`; `rom/filesys/pfs3/fs/dd_support.c:190–199`.
- **Trigger/trace:** `Initialize` assigns `g->notifyport = CreateMsgPort()` and does not check for NULL before continuing. The main packet loop derives a signal mask from `g->notifyport->mp_SigBit`, and notification sending sets it as a reply port.
- **Consequence:** Memory pressure during handler startup can crash PFS3 rather than failing the mount cleanly.
- **Confidence/limits:** High source confidence; applies to active PFS3 variants, requiring `CreateMsgPort` failure.

### FS-048 — FAT filename handling reads before a short name while testing `.info`

- **Source:** `rom/filesys/fat/names.c:264–307, 458–563, 568–571`.
- **Trigger/trace:** `SetDirEntryName` strips leading/trailing spaces and periods, then always computes at least one long-name slot for any nonempty name. At the end it calls `strcmp(name + len - 5, ".info")` without requiring `len >= 5`. Names of one to four characters make the pointer refer before the supplied name buffer.
- **Consequence:** Creating or renaming a short-named FAT entry can read unrelated memory, crash under guard-page placement, or set the hidden flag incorrectly.
- **Confidence/limits:** High source confidence; active FAT path. Exact visible effect depends on what lies before the name pointer.

### FS-049 — FAT directory-space reservation ignores entry-write failures

- **Source:** `rom/filesys/fat/direntry.c:429–515`.
- **Trigger/trace:** After finding an end-of-directory marker, `AllocDirEntry` advances to `last`, setting each intermediate entry's name byte to zero. It calls `UpdateDirEntry(de, glob)` for every entry but discards each result, including writes that extend the directory through a new cluster. Its later clearing loop and final `GetDirEntry(dh, last - 1, ...)` also ignore results before the function returns success.
- **Consequence:** An I/O or allocation failure while extending a FAT directory can be reported as a successful reservation, leaving missing/misordered end markers or causing the caller to write a name into an entry that was never obtained successfully.
- **Confidence/limits:** High source confidence for lost errors; a failure in a directory extension/update is required. Cluster-link failure also interacts with FS-002, but this is a separate caller-level failure to abort reservation.

### FS-050 — NTFS accepts boot-record sizing bytes that produce invalid shifts and buffers

- **Source:** `workbench/fs/ntfs/ntfs.c:1065–1085, 1105–1135`.
- **Trigger/trace:** `ReadBootSector` checks only that sector and cluster sizes are nonzero powers of two. It derives MFT and index record sizes from signed boot-sector bytes with `1 << (-clusters_per_*_record - SECTORSIZE_SHIFT)` for zero/negative values, without constraining the shift to a valid range or ensuring the resulting record size is nonzero and within the cluster/sector geometry. Bytes such as zero or -1 yield a negative shift; very negative bytes yield an oversized shift. The resulting `mft_size` controls allocation and subsequent sector reads.
- **Consequence:** A malformed NTFS boot sector can invoke undefined shift behavior, allocate an incorrect size, and drive out-of-bounds metadata reads during mount.
- **Confidence/limits:** High source confidence for missing bounds and the negative-shift examples; requires malformed boot metadata.

### FS-051 — NTFS mount skips the declared volume-extent check

- **Source:** `workbench/fs/ntfs/ntfs.c:1092–1103`; `workbench/fs/ntfs/disk.c:402–440`.
- **Trigger/trace:** `ReadBootSector` reads `number_of_sectors` from the NTFS boot sector, but the comparison against the configured device end is enclosed in `#if (0)`. The data path uses `first_device_sector` and the boot-declared total when translating requests.
- **Consequence:** A boot sector claiming more sectors than the mounted partition can make NTFS follow metadata or file extents outside that partition, potentially reading adjacent media regions or surfacing device errors.
- **Confidence/limits:** High source confidence for the disabled check; actual out-of-partition access requires subsequent on-disk references beyond the partition and a device that accepts them.

### FS-052 — SMBFS volume names can overflow `FileInfoBlock.fib_FileName`

- **Source:** `workbench/network/smbfs/source_code/main.c:1940–2087, 3723–3748, 3774–3799`; `compiler/include/dos/dos.h:57, 81`.
- **Trigger/trace:** `AddVolume` permits a configured volume name up to 255 bytes. In root `Action_ExamineObject` and the corresponding root examine path, it copies the full counted volume name into `fib->fib_FileName + 1` with no capacity check. `FileInfoBlock.fib_FileName` has 108 bytes total.
- **Consequence:** Mounting a share with a volume name longer than 106 characters and examining its root overwrites following `FileInfoBlock` fields or caller memory.
- **Confidence/limits:** High source confidence; requires an explicitly long configured volume name. Active SMBFS build.

### FS-053 — PFS3 examination writes terminators past fixed DOS name/comment fields

- **Source:** `rom/filesys/pfs3/fs/directory.c:1019–1040`; `rom/filesys/pfs3/fs/blocks.h:503–507`; `compiler/include/dos/dos.h:57, 81`.
- **Trigger/trace:** `FillFib` copies the counted on-disk name/comment into `FileInfoBlock`, then writes `fib_FileName[nlength + 1] = 0` and `fib_Comment[*comment + 1] = 0` without constraining either index. The comment copy is capped at `CMSIZE` bytes, but its terminator uses the original on-disk length. Even a maximum 79-character comment occupies all 80 bytes of `fib_Comment`, so the extra terminator lands at index 80.
- **Consequence:** Examining a long valid or malformed PFS3 entry overwrites adjacent `FileInfoBlock` fields or memory beyond it.
- **Confidence/limits:** High source confidence; active PFS3 variants. The final impact depends on the name/comment lengths encountered; a valid maximum-length comment suffices for the one-byte overflow.

### FS-054 — PFS3 soft links accept targets larger than their one-block storage

- **Source:** `rom/filesys/pfs3/fs/directory.c:2378–2420, 2486–2570`.
- **Trigger/trace:** `CreateSoftLink` copies the caller's target with `strcpy` into a fixed 1024-byte `softblock` without checking length, writes only one disk block without checking `DiskWrite`'s result, and records the full `strlen(softblock)` as file size. Targets over 1023 bytes overflow memory; targets longer than the written block can be truncated on disk while their metadata retains the larger size. `ReadSoftLink` reads one block into a 1024-byte uninitialized buffer and uses `strchr` before checking a stored length or NUL.
- **Consequence:** Long targets can corrupt handler memory or create broken links whose later reads consume uninitialized bytes; a failed block write can also leave a directory entry pointing at unwritten link data.
- **Confidence/limits:** High source confidence for the unbounded copy and one-block mismatch; exact disk threshold depends on configured PFS3 block size. Requires a long soft-link target or malformed link metadata.

### FS-055 — PFSDoctor SCSI write passes the remaining count instead of the chunk size

- **Source:** `rom/filesys/pfs3/pfsdoctor/device.c:379–405`; `rom/filesys/pfs3/pfsdoctor/mmakefile.src:1–23`.
- **Trigger/trace:** `dev_WriteBlocksDS` splits a multi-block write into `transfer = min(blocks, maxtransfer)` chunks and encodes `transfer` in the SCSI WRITE(10) command, but passes `blocks << BLOCKSHIFT` as the data-buffer length to `DoSCSICommand`. For any chunk after a max-transfer split, the declared data length exceeds the command's transfer count and the available bytes at the advancing `buffer` pointer.
- **Consequence:** Repair writes can read past the supplied buffer or issue an inconsistent SCSI request; depending on the device wrapper, this can fail the repair or write unintended data.
- **Confidence/limits:** High source confidence; applies to the optional built `PFSDoctor` utility when a write exceeds its configured max-transfer chunk.

### FS-056 — PFSDoctor device loops can make no progress when MaxTransfer is too small

- **Source:** `rom/filesys/pfs3/pfsdoctor/device.c:294–324, 344–368, 383–405, 425–457`.
- **Trigger/trace:** The standard paths round `min(io_length, de_MaxTransfer)` down to a multiple of `volume.blocksize`; the direct-SCSI paths shift `de_MaxTransfer` by `volume.blockshift`. If the configured maximum is below one filesystem block, the computed chunk is zero. Each `while` loop then issues a zero-length transfer or command and leaves its remaining length/count unchanged when that operation reports success.
- **Consequence:** A scan or repair operation can hang indefinitely on successful zero-length I/O instead of rejecting the invalid mount geometry; a device that rejects it instead triggers the retry/error path.
- **Confidence/limits:** High source confidence for the zero-progress loops; requires a device environment with `de_MaxTransfer < blocksize` and reaching PFSDoctor. This is separate from the active PFS3 handler's equivalent configuration issue in FS-018.

### FS-057 — PFSDoctor block search uses a failed scratch allocation as an I/O buffer

- **Source:** `rom/filesys/pfs3/pfsdoctor/fullscan.c:373–414`; `rom/filesys/pfs3/pfsdoctor/standardscan.c:499–517`.
- **Trigger/trace:** `SearchBlock` assigns `calloc(1, SIZEOF_RESBLOCK)` to `blk.data` and does not check for NULL. It passes the block to `volume.getblock`, whose `vol_GetBlock` passes `blok->data` to the metadata read path.
- **Consequence:** Memory pressure during a PFSDoctor full scan can fault the utility during a raw metadata read rather than reporting an allocation error.
- **Confidence/limits:** High source confidence; optional PFSDoctor utility and allocation failure required.

### FS-058 — AFS reads a file data block using an unchecked on-disk byte count

- **Source:** `rom/filesys/afs/filehandles1.c:742–788`.
- **Trigger/trace:** For the original FFS data-block format (`dosflags == 0`), `readData` takes `BLK_DATA_SIZE` from the loaded block, subtracts `ah->current.byte`, caps only against the requested length, and copies from the data area. It does not require the stored count to fit in `BLOCK_SIZE - BLK_DATA_START * 4`, nor prevent subtraction underflow if it is below the current byte offset.
- **Consequence:** A corrupted data block with a valid checksum can make a read copy beyond that block's buffer or return bytes outside the declared payload.
- **Confidence/limits:** High source confidence for missing bounds; requires malformed original-FFS data metadata and a read spanning the affected block.

### FS-059 — FAT metadata operations acknowledge failed directory-entry updates

- **Source:** `rom/filesys/fat/ops.c:1080–1143, 1148–1197`; `rom/filesys/fat/direntry.c:405–428`.
- **Trigger/trace:** `OpSetProtect` and `OpSetDate` obtain an entry and call `UpdateDirEntry`, but discard its error, send a change notification, and return success. The directory `.` update in `OpSetProtect` also ignores both its read and write results. `UpdateDirEntry` can fail while loading or writing the entry through `WriteFileChunk`.
- **Consequence:** Under cache allocation or I/O failure, callers are told permissions or dates changed when they did not; watchers may receive notifications for absent changes. A directory can also have mismatched protection attributes between its parent entry and `.` entry.
- **Confidence/limits:** High source confidence; active FAT path, failure while updating metadata required. Other operations show the same pattern, but these examples establish the root cause without listing every call site.

### FS-060 — FAT `SetFileSize` counts chain links instead of clusters

- **Source:** `rom/filesys/fat/ops.c:959–1060`.
- **Trigger/trace:** `OpSetFileSize` initializes `count = 0` for a nonempty chain and increments it only while a next cluster exists. A one-cluster file is therefore counted as zero clusters; a two-cluster file as one. When resizing a one-cluster file to another size still within one cluster, `count < want` allocates an unnecessary second cluster. Truncating a longer file similarly stops one cluster late.
- **Consequence:** Routine resizes consume extra FAT clusters and fail to free the intended tail cluster, gradually wasting space. The function can report disk full despite enough space for the requested file size.
- **Confidence/limits:** High source confidence; active FAT path. Requires a size change on a nonempty file, so the early equal-size return does not apply.

### FS-061 — FAT `SetFileSize` leaves open handles with stale file size

- **Source:** `rom/filesys/fat/ops.c:903–1075`; `rom/filesys/fat/packet.c:350–373`.
- **Trigger/trace:** On success, `OpSetFileSize` changes `de.e.entry.file_size` and `*newsize` but never updates the shared `lock->gl->size` used by file reads, writes, and subsequent `OFFSET_END` size calculations. The packet dispatcher returns the new size directly and performs no follow-up update.
- **Consequence:** An open file can continue to report/use its pre-resize length until it is reopened; a subsequent write or resize may compute positions from stale metadata.
- **Confidence/limits:** High source confidence; active FAT path, requires a successful size-changing call on an open file.

### FS-062 — FAT file extension exposes stale contents of newly allocated clusters

- **Source:** `rom/filesys/fat/ops.c:1003–1075`; `rom/filesys/fat/fat.c:350–362`; `rom/filesys/fat/file.c:75–251`.
- **Trigger/trace:** `OpSetFileSize` extends a file by selecting free clusters, setting FAT links, and raising the directory entry's file size. `AllocCluster` updates only FAT and FSInfo accounting; no path zeros the newly allocated data sectors or the old last cluster's formerly out-of-file tail. After reopening, `ReadFileChunk` reads those sectors as ordinary file contents.
- **Consequence:** Bytes from previously deleted files or old slack space become visible in the extended file instead of zeros, and unwritten parts of the new logical file contain nondeterministic old data.
- **Confidence/limits:** High source confidence; active FAT path. Requires extending a file with `ACTION_SET_FILE_SIZE` and then reading the new range, potentially after reopening because of FS-061.

### FS-063 — FAT rename continues after destination creation fails

- **Source:** `rom/filesys/fat/ops.c:594–648`.
- **Trigger/trace:** `OpRename` calls `CreateDirEntry` for the destination. On nonzero error it releases both directory handles but does not return or branch; it immediately writes through the destination `dde`, updates open lock locations, calls `DeleteDirEntry` on the source, sends notification, and returns success. The results of the destination metadata update and source deletion are also ignored.
- **Consequence:** A full directory, allocation failure, or I/O error during rename can lead to use of an uninitialized/partial destination entry, deletion of the original name, or a falsely successful rename with lost/inconsistent metadata.
- **Confidence/limits:** High source confidence; active FAT path, destination creation failure required. Exact corruption depends on how far `CreateDirEntry` progressed before failing.

### FS-064 — FAT delete treats a directory read error as emptiness and frees clusters after failed unlink

- **Source:** `rom/filesys/fat/ops.c:388–506`.
- **Trigger/trace:** `OpDeleteFile` scans a directory with `while (GetDirEntry(...) == 0)` and, on any exit, proceeds as if it reached an end marker; it does not distinguish an I/O/cache error from an empty directory. It also ignores errors from the containing-directory lookup and `DeleteDirEntry`, then unconditionally frees the file's cluster chain and returns success.
- **Consequence:** A read failure can allow removal of a nonempty directory. A failed directory-entry deletion followed by successful cluster freeing can leave a live entry pointing at now-free sectors, enabling later allocation to overwrite its data.
- **Confidence/limits:** High source confidence; active FAT path, contingent on a directory read or entry update failure.

### FS-065 — FAT directory creation dereferences a failed new lock and ignores initialization failures

- **Source:** `rom/filesys/fat/ops.c:711–778`.
- **Trigger/trace:** After creating a new directory entry and cluster, `OpCreateDir` initializes `.` and `..` entries and clears the rest of the cluster, ignoring `GetDirEntry` and `UpdateDirEntry` failures. (`InitDirHandle` currently always succeeds.) It then calls `LockFile` and, regardless of its result, dereferences `(*newdirlock)->ioh` to send a notification.
- **Consequence:** A cache/I/O failure can leave a directory visible with missing `.`/`..` or stale entries; a lock allocation failure can crash the handler immediately after the directory was created.
- **Confidence/limits:** High source confidence; active FAT path, failure during initialization or lock creation required.

### FS-066 — FAT resize leaves allocated clusters behind when growth runs out of space

- **Source:** `rom/filesys/fat/ops.c:1028–1060`.
- **Trigger/trace:** `OpSetFileSize` grows the chain one cluster at a time. If `FindFreeCluster` fails after one or more successful allocations, it immediately returns the error without freeing newly allocated clusters or updating the directory entry; the source comment explicitly notes the missing cleanup. For an initially empty file, the new first cluster is not published, so all allocated clusters are orphaned. For an existing file, its on-disk chain grows despite the reported resize failure.
- **Consequence:** Failed resizes consume free space and can leave the chain longer than the file size, making later accounting and recovery harder.
- **Confidence/limits:** High source confidence; active FAT path, requires disk-full or allocation failure after partial growth.

### FS-067 — Unix hosted filesystem returns an uninitialized secondary error after successful reads

- **Source:** `arch/all-unix/filesys/emul_handler/emul_host.c:551–614`; `arch/all-hosted/filesys/emul_handler/emul_handler.c:855–868`.
- **Trigger/trace:** `DoRead` declares `SIPTR error` without initializing it. It assigns the variable only if `Hidd_UnixIO_Wait` fails or the host `read` returns -1. On ordinary successful reads it nevertheless executes `*err = error`, and the packet dispatcher uses that as `dp_Res2`.
- **Consequence:** Successful `ACTION_READ` replies contain an unpredictable secondary error code, potentially confusing clients that inspect `IoErr()` even after a positive byte count.
- **Confidence/limits:** High source confidence; Unix hosted builds, ordinary successful file reads. The primary byte count remains the host `read` result, apart from FS-032's stdio loop issue.

### FS-068 — CDVDFS accepts a short optical-sector read as a full cache fill

- **Source:** `rom/filesys/CDVDFS/src/cdrom.c:265–293, 298–355`.
- **Trigger/trace:** `Read_From_Drive` requests `2048 * p_number_of_sectors` bytes and returns success whenever `DoIO` returns zero, without comparing `scsireq->io_Actual` to the requested byte count. `Read_Chunk` then exposes the cache buffer to ISO/HFS parsers as though all requested sectors were filled.
- **Consequence:** A device that reports a successful short transfer can make filesystem parsers consume stale bytes from a prior cache fill or uninitialized buffer content.
- **Confidence/limits:** High source confidence; native CDVDFS path, contingent on successful short I/O from the underlying device.

### FS-069 — AFS validator can mark a rebuilt bitmap valid before its writes succeed

- **Source:** `rom/filesys/afs/validator.c:219–238, 593–611, 737–788`.
- **Trigger/trace:** During validation, the root block's bitmap-valid flag is set and its cache buffer marked dirty while traversing the root. After traversal, `record_bitmap` rewrites bitmap blocks but has a `void` return and ignores `writeDisk` results. Its low-memory fallback dereferences each `getBlock` result without checking NULL. `start_superblock` still returns `vr_OK` when bitmap writes fail.
- **Consequence:** An I/O failure can leave the root advertising a valid bitmap that was not fully rebuilt, risking later allocation based on stale free-space data; low-memory/read failure in the fallback can crash validation.
- **Confidence/limits:** High source confidence; active AFS validator, contingent on write/read failure during bitmap repair. General cache failure propagation also appears in FS-012, but this entry covers validation's commit ordering.

### FS-070 — SFS free-space guard underflows when transaction reserve exceeds free blocks

- **Source:** `rom/filesys/SFS/FS/bitmap.c:65–75, 506–522`; `rom/filesys/SFS/FS/transactions.c:1148–1161`.
- **Trigger/trace:** `enoughspace` evaluates `blocksfree - transactionspace() < blocks + ALWAYSFREE` with unsigned `ULONG` values. When the buffered transaction's required storage blocks exceed currently free blocks, the subtraction wraps to a large number and the guard can return TRUE. `findandmarkspace` may then allocate from the remaining free bitmap area despite not having room to persist the transaction.
- **Consequence:** On a near-full SFS volume with a large pending transaction, further allocations can be admitted until transaction flush lacks reserve space, producing disk-full failures after metadata has been staged.
- **Confidence/limits:** High source confidence for the unsigned underflow; reaching the state requires transactionspace() > blocksfree and enough individual free bits for the requested allocation. The transaction rollback path may contain some downstream effects, so the claimed consequence is loss of the intended reserve guarantee.

### FS-071 — SFS accepts invalid mount block geometry before division and shift-based I/O

- **Source:** `rom/filesys/SFS/FS/deviceio.c:261–327, 356–381, 620–631`.
- **Trigger/trace:** `initdeviceio` calls `changegeometry` with the DosEnvec after opening the device, without validating `de_SectorPerBlock` or `de_SizeBlock`. `changegeometry` divides `sectors_total` by `sectors_block` and derives `shifts_block` by floor-log2 of `bytes_block`; later I/O forms byte offsets and lengths with that shift. A zero sectors-per-block divides by zero, and a non-power-of-two block size produces byte offsets/lengths for the wrong size.
- **Consequence:** An invalid mountlist can crash SFS during mount or make it access the wrong sectors, risking filesystem corruption on writes.
- **Confidence/limits:** High source confidence for the unchecked arithmetic; requires malformed or unsupported mount geometry, not a normal SFS volume with a valid DosEnvec.

### FS-072 — SFSDefrag keeps polling after a defragment-step packet fails

- **Source:** `rom/filesys/SFS/SFSdefrag/main.c:202–251`.
- **Trigger/trace:** After a successful `ACTION_SFS_DEFRAGMENT_INIT`, the GUI loops until `stop_me` or a `DONE` step. If `ACTION_SFS_DEFRAGMENT_STEP` returns `DOSFALSE`, it skips step parsing but still calls `updateBitmap`, processes one UI input cycle, and repeats with no error branch or termination.
- **Consequence:** A persistent handler/device error can leave the defragmenter repeatedly sending failing packets while appearing to remain in progress until the user manually stops it.
- **Confidence/limits:** High source confidence; optional SFSDefrag GUI utility, requires step failure after initialization. The loop still permits user cancellation.

### FS-073 — PFS3 LRU setup frees uninitialized pointers after a partial allocation

- **Source:** `rom/filesys/pfs3/fs/lru.c:115–162`.
- **Trigger/trace:** `InitLRU` allocates `LRUarray` without `MEMF_CLEAR`, then fills its pointer slots one by one. If allocation of a cached block fails at slot `j`, it calls `DeallocLRU`, which iterates over the full `poolsize` and calls `FreeVec` on every slot, including never-initialized entries after `j`.
- **Consequence:** Memory pressure during PFS3 mount can pass arbitrary pointer values to `FreeVec`, corrupting the allocator or crashing the handler.
- **Confidence/limits:** High source confidence; applies to active PFS3 variants when a per-block allocation fails after the pointer array allocation.

### FS-074 — NTFS read packets can return an uninitialized byte count on failure or zero-length reads

- **Source:** `workbench/fs/ntfs/ops.c:299–329`; `workbench/fs/ntfs/packet.c:251–283`.
- **Trigger/trace:** The packet dispatcher declares `UQUAD read` without initializing it. `OpRead` returns success immediately for `want == 0` without setting `*read`. It also initializes `err = 0` and never changes it when `MapMFTAttrib` finds no data attribute or `ReadMFTAttrib` fails; in those cases it does not set `*read` either. The dispatcher then converts the uninitialized `read` to its success reply count.
- **Consequence:** A zero-byte read, missing data attribute, or metadata/I/O read failure can produce an arbitrary positive count or other false success, with caller buffers left untouched.
- **Confidence/limits:** High source confidence; active NTFS handler. The zero-length trigger is deterministic at the source level; failure cases depend on damaged metadata or I/O errors.

### FS-075 — Read-only NTFS accepts output open and truncates only its in-memory file state

- **Source:** `workbench/fs/ntfs/ntfs_fs.h:34`; `workbench/fs/ntfs/ops.c:144–229`; `workbench/fs/ntfs/direntry.c:519–542`; `workbench/fs/ntfs/packet.c:215–245`.
- **Trigger/trace:** `NTFS_READONLY` is defined for the built handler. Nevertheless, `ACTION_FINDOUTPUT` calls `OpOpenFile`, which obtains an exclusive lock on an existing file, calls `UpdateDirEntry` (returning `ERROR_DISK_WRITE_PROTECTED` in this build) but ignores that result, then sets the shared first cluster and size to empty values and returns success.
- **Consequence:** Opening an existing NTFS file for output appears to truncate it within the handler even though no on-disk change was made; subsequent reads through that shared lock view can show an empty file until the volume state is rebuilt. The caller is not told that the operation is write-protected.
- **Confidence/limits:** High source confidence; active Workbench NTFS build, requires `ACTION_FINDOUTPUT` for an existing file. The on-disk file remains unchanged because this build is read-only.

### FS-076 — Windows hosted filesystem startup leaks its allocated IRQ on worker creation failure

- **Source:** `arch/all-mingw32/filesys/emul_handler/emul_handler_native.c:64–86`; `arch/all-mingw32/kernel/host_intr.c:379–413`.
- **Trigger/trace:** `Emul_Init_Native` allocates a system IRQ, then creates a command event and a worker thread. If either host allocation fails, it closes a created event but returns without calling `KrnFreeSystemIRQ` for the successful IRQ allocation.
- **Consequence:** Each failed startup retains one of the finite Windows-hosted IRQ slots; repeated startup failures can exhaust slots needed by this or other virtual devices.
- **Confidence/limits:** High source confidence for the Windows hosted emul.handler under event or thread creation failure. No host allocation failure was run.

### FS-077 — PFSDoctor retries forever when mount requester allocation fails

- **Source:** `rom/filesys/pfs3/pfsdoctor/standardscan.c:602–609, 652–675, 716–739`; `workbench/libs/asl/allocaslrequest.c:78–80, 142–144`; `workbench/libs/asl/aslrequest.c:71–75`.
- **Trigger/trace:** In both root-block repair branches, PFSDoctor stores `AllocAslRequestTags`'s result in `freq` without checking it, then enters a `do/while (!ok)` loop. `AllocAslRequest` returns NULL after its internal allocations fail. `AslRequest(NULL, ...)` explicitly returns FALSE, while `ok` starts false and is assigned only from a successful `MakeMountFile` call inside the true requester branch.
- **Consequence:** Under memory pressure after repair, PFSDoctor repeatedly prints “no mountlist written” and cannot reach `FreeAslRequest` or return from `GetRootBlock`; the repair operation becomes stuck instead of reporting the allocation failure.
- **Confidence/limits:** High source confidence; optional PFSDoctor utility, requires requester allocation failure on either relocated/redefined partition path. No allocation failure was induced.

### FS-078 — PFSDoctor copies the selected mountfile path into a 108-byte buffer without bounds

- **Source:** `rom/filesys/pfs3/pfsdoctor/standardscan.c:607–608, 652–662, 716–726`; `rom/filesys/pfs3/fs/blocks.h:499–505`; `workbench/libs/asl/filereqhooks.h:36–39`; `workbench/libs/asl/filereqhooks.c:878–880, 1772–1790`.
- **Trigger/trace:** `GetRootBlock` declares `mfname[FNSIZE]`, with `FNSIZE == 108`, then concatenates the selected `fr_Drawer` and `fr_File` using two unbounded `stpcpy` calls in both repair branches. The selected FileRequester accepts a drawer string up to `MAX_PATH_LEN` (1025) characters and a file string up to `MAX_FILE_LEN` (257), then clones those strings into the returned requester fields.
- **Consequence:** A valid longer drawer or combined drawer/file path overwrites the stack beyond `mfname` before `MakeMountFile` opens the destination, potentially corrupting nearby state or stopping PFSDoctor.
- **Confidence/limits:** High source confidence for the undersized destination and unbounded copies; optional PFSDoctor utility, requires a selected path whose combined length exceeds 107 characters. No requester or path was exercised.

### FS-079 — RAM handler reports 16-byte block counts as 8-byte blocks on 64-bit builds

- **Source:** `rom/filesys/ram/handler.h:55–60`; `compiler/include/exec/memory.h:68–69`; `rom/filesys/ram/filesystem.c:1301–1318,1350–1368`; `rom/filesys/ram/commands.c:1679–1694`; `compiler/include/dos/dos.h:175–205`.
- **Trigger/trace:** On a 64-bit target, `handler.h` sets `MEM_BLOCKSHIFT` to 4, so `MEMBLOCKS()` and the file/object `block_count` accounting use 16-byte units (`filesystem.c:1318,1366`). `CmdInfo()` returns those counts as `id_NumBlocks`/`id_NumBlocksUsed` but sets `id_BytesPerBlock` to `MEM_BLOCKSIZE`, which is 8 bytes. Its free-memory contribution is also divided into 16-byte units by the same macro.
- **Consequence:** `Info()` advertises half the byte capacity and half the used/free byte totals represented by its block counts on 64-bit builds. Use one block-size definition consistently for rounding, accounting and `id_BytesPerBlock`.
- **Confidence/limits:** High source confidence for the 16-versus-8 unit mismatch. Applies to the selected `kernel-fs-ram` handler on 64-bit targets; no `Info()` request was run.

### FS-080 — RAM handler's Info block count underflows when no memory is free

- **Source:** `rom/filesys/ram/commands.c:1679–1694`; `rom/filesys/ram/handler.h:55–60`; `rom/exec/availmem.c:64–75`.
- **Trigger/trace:** When `AvailMem(MEMF_ANY)` returns zero, `CmdInfo()` still evaluates `MEMBLOCKS(AvailMem(...))`. The macro computes `(((A) - 1) >> MEM_BLOCKSHIFT) + 1`; because `AvailMem()` returns unsigned `IPTR`, subtracting one from zero wraps before the shift. The resulting huge count is added to the used block count and stored in `id_NumBlocks`.
- **Consequence:** A RAM disk `Info()`/`DiskInfo()` request reports a wildly oversized or wrapped total capacity exactly when system free memory is exhausted, instead of reporting only the blocks already used. Guard zero before rounding or use a zero-safe ceiling division.
- **Confidence/limits:** High source confidence for the unsigned underflow and invalid result on both pointer widths. Requires an `Info()` request while `AvailMem(MEMF_ANY) == 0`; no memory exhaustion or packet was induced.

### FS-081 — NTFS publishes partially initialized volume state after per-volume allocation failure

- **Source:** `workbench/fs/ntfs/disk.c:183–245`; `workbench/fs/ntfs/packet.c:611–614`; `workbench/fs/ntfs/disk.c:47–71`.
- **Trigger/trace:** After a successful boot-sector read and no matching existing volume, `DoDiskInsert()` creates a separate per-volume pool and allocates `VolumeInfo` and `DosList` from it. Failures are not treated as mount failure: if the pool creation fails, execution skips the allocation block but still assigns `fs_data->info = NULL`, publishes `glob->data = fs_data`, and queues `ACTION_VOLUME_ADD` with `newvol == NULL`. The packet handler dereferences `vol` to obtain its lock-list at `packet.c:612–614` before checking the packet type or volume pointer. If the pool exists but `VolumeInfo` allocation fails while `DosList` allocation succeeds, `disk.c:215` dereferences `vol_info->locks`. If `DosList` allocation fails after `VolumeInfo` succeeds, the code deletes the pool but retains `vol_info`, publishes it through `fs_data->info`, and queues the null volume packet; later `FillDiskInfo()` reads the freed lock-list state at `disk.c:68–70`.
- **Consequence:** Memory pressure during NTFS volume insertion can fault the handler immediately or publish dangling/incomplete volume state that faults on a later disk-info request. The handler does not report insertion failure and can leave `glob->data` set to the invalid state.
- **Confidence/limits:** High source confidence; requires a per-volume pool or child allocation failure after the separate handler pool has already allocated `fs_data` and the boot-sector read succeeds. No allocation failure, mount, or packet was run.

<a id="fs-082"></a>

### FS-082 — SFS buffered block I/O uses `CopyMemQuick` with an odd caller buffer

- **Source:** `rom/filesys/SFS/FS/deviceio.c:635–696,789–806`; `rom/filesys/SFS/FS/cachedio.c:755–804,886–897`; `rom/filesys/SFS/FS/filesystemmain.c:2041–2110`; `workbench/c/Mount.c:634–666`; `arch/m68k-all/exec/copymemquick.S:18–41`.
- **Trigger/trace:** `Mount`'s default DOS environment sets `de_Mask` to `-2` (`0xfffffffe`). `initdeviceio()` installs that mask, and `transfer()` routes any buffer with bits outside it—including an odd address—through `transfer_buffered()`. SFS `read()` sends large block-aligned reads directly to `transfer()`; writes also reach it after cache write-through. The bounce buffer is aligned, but both `DIO_WRITE` and `DIO_READ` call `CopyMemQuick()` with the original caller buffer as one endpoint, without checking ULONG alignment. A byte-aligned odd DOS packet buffer is valid for the byte-counted file I/O path; the cache-hit path itself explicitly falls back to `CopyMem()` for unaligned buffers.
- **Consequence:** On selected m68k builds, the quick-copy assembly performs longword loads/stores through that odd address, which raises an address error on 68000-class CPUs instead of completing the buffered file transfer. On generic C implementations the same call violates `CopyMemQuick()`'s ULONG-alignment precondition.
- **Confidence/limits:** High source confidence for the missing alignment check and m68k address-error path. Requires an SFS read/write that reaches block transfer, an odd caller buffer, and a DMA mask that sends it through the bounce path; the default `Mount` environment supplies that mask. Other custom DOS environments may avoid the trigger. No file operation, fault, or build was run.

<a id="fs-083"></a>

### FS-083 — SFS cache exhaustion is dereferenced before callers can handle it

- **Source:** `rom/filesys/SFS/FS/cachebuffers.c:278–294, 299–330, 636–685`; `rom/filesys/SFS/FS/adminspaces.c:17–52`; `rom/filesys/SFS/FS/transactions.c:1098–1130`.
- **Trigger/trace:** `getcachebuffer()` scans the fixed cache and explicitly returns NULL after every buffer is found locked or is an original block that cannot be evicted while its latest version is cached (`cachebuffers.c:656–684`). `createnewcachebuffer()` immediately stores `cb->blckno` and `cb->bits` through that result (`:287–292`); `newcachebuffer()` then also assumes the constructor returned a buffer (`:323–327`). This defeats the explicit NULL/error handling at `allocadminspace()` (`adminspaces.c:46–48`) and the error-return contract of `applyoperation()` (`transactions.c:1105–1129`).
- **Consequence:** If an operation needs a new cache entry while all configured cache buffers are pinned, the handler faults in the constructor instead of returning `ERROR_NO_FREE_STORE`/a recoverable cache exhaustion result. The request path is explicitly recognized by `getcachebuffer()`, which displays the exhaustion requester and returns NULL.
- **Confidence/limits:** High source confidence for the unchecked NULL path. Trigger requires cache exhaustion during a call that needs a new buffer; buffer count and pinning depend on mount configuration and operation state. No exhaustion, requester, or filesystem operation was induced.

<a id="fs-084"></a>

### FS-084 — NTFS dereferences a failed notification-port allocation during startup

- **Source:** `workbench/fs/ntfs/main.c:66,68–84`; `rom/exec/createmsgport.c:50–74`.
- **Trigger/trace:** `handler()` stores `CreateMsgPort()` directly in `glob->notifyport` without checking the result. `CreateMsgPort()` can return NULL when either the port allocation or signal allocation fails. If the subsequent library, pool, timer and disk-handler initialization succeeds, `handler()` dereferences `glob->notifyport->mp_SigBit` at line 83 while constructing its `Wait()` mask.
- **Consequence:** NTFS handler startup faults instead of returning an initialization error and replying failure to the mount request.
- **Scope:** The Workbench NTFS handler target (`workbench-fs-ntfs`), when initialization reaches the event-loop setup after notification-port creation failed. The later initialization successes make this a conditional resource-failure path; exhaustion that also prevents a later required allocation may instead take an existing error path.
- **Confidence/limits:** High for the unchecked NULL result and dereference. No allocation failure or mount was induced; source-only.

<a id="fs-085"></a>

### FS-085 — NTFS reports successful startup after initialization fails

- **Source:** `workbench/fs/ntfs/main.c:49,68–80,145–150`; `rom/dos/runhandler.c:163–181`.
- **Trigger/trace:** NTFS initializes `error` to `ERROR_NO_FREE_STORE` and only enters its service loop after the required libraries, pool, timer and `InitDiskHandler()` succeed (`main.c:49,68–80`). If any setup step fails, the startup packet remains non-NULL and reaches the common cleanup block. That block unconditionally sets `dp_Res1 = DOSTRUE` and `dp_Res2 = 0` (`:145–150`), ignoring `error`. DOS `RunHandler()` treats only `DOSFALSE` as failed startup and returns the new process's message port on any other result (`runhandler.c:171–181`).
- **Consequence:** A failed NTFS handler is accepted as started. The handler never assigned `devnode->dol_Task` or entered its packet loop, yet DOS can return its dead process's message port as the device port; subsequent filesystem requests can be sent to a process that has already exited and remain unanswered.
- **Scope:** Workbench NTFS handler target (`workbench-fs-ntfs`), when a required startup operation fails after the startup packet is received. This is independent of FS-084's notification-port NULL dereference.
- **Confidence/limits:** High source confidence for the unconditional success reply and DOS caller interpretation. The exact failure depends on a required startup operation failing; no failure or mount was induced.

## Coverage boundaries

All filesystem implementation families found in the repository-wide inventory are represented above, including handlers with no substantiated candidate in this pass (AUX and speak). The reported conditions are deliberately narrower than a blanket claim that every configuration is affected. Malformed media, allocation failure, short I/O, invalid mount geometry, and raw DOS packets are stated where needed. This is a source review at one commit, not a runtime reproduction or guarantee that no other bugs remain.

Related report: [AROS-disk.md](AROS-disk.md). The broader DOS library, Exec/kernel, network stack, graphics/UI, applications, compiler/runtime and other device families remain outside this filesystem review.
