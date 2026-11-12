# AROS disk/block subsystem source review

## Status and baseline

The source review covers the inventoried scope and dependency boundaries below; final candidate reconciliation continues as caller paths are rechecked. This source-derived report does not claim hardware reproduction or prove that the subsystem contains no other bugs.

Baseline: `282a0454356dc2b9554ab023997d7f7ca24fc03e`. Reviewed the current master worktree at that fixed commit. Project files were not modified; no builds, tests, executables, device commands or media writes were run.

There are **324 retained numbered candidate entries**, plus five historical aliases and two withdrawn entries. This is not a count of independent confirmed defects: some entries group several copied implementations, and overlapping manifestations retain distinct triggers/fixes. DISK-022 → DISK-006, DISK-305 → DISK-244, DISK-310 → DISK-233, DISK-313 → DISK-245 and DISK-314 → DISK-246 are aliases; DISK-283 is withdrawn because its arithmetic example was wrong, and DISK-317 because its source is unselected by the current ATA build.

## Scope and evidence limits

The review covers all inventoried local block-device implementations, storage abstractions, ATA/AHCI/NVMe/VirtIO/SCSI/SD transports, both USB mass-storage implementations, floppy/RAM/file/host-backed devices, disk-image backends, platform variants, partition parsers/publication, automount/boot integration, and direct partition/format/install/configuration consumers. Source inventories were compared with modern and relevant standalone/legacy build definitions. Optional and inactive source paths retain their actual build conditions.

Dependencies were followed where needed to establish request, allocation, mapping, list, callback, semaphore, DOS and boot contracts. General filesystem implementations, unrelated USB/PCI functionality, firmware/GUI presentation, externally supplied plugins and callback implementations, and generic third-party compression/XML/crypto algorithms are outside this disk-driver audit. In particular, the diskimage-local standalone codec/XML directories contain upstream algorithm sources: their wrappers, allocators, ABI and build selections were reviewed, but their full generic algorithm bodies were not exhaustively audited. Modern externally fetched library sources were not downloaded. Disk-image PNG GUI rendering is presentation code, not a block-image backend. The generated external compatibility declaration forest was inventoried and used call boundaries were checked; unrelated exported vendor APIs are not claimed fully read.

Confidence describes the source trace separately from impact. Hardware ordering/coherency, actual DMA mappings, strict-alignment faults, allocation failures, short transfers, malformed media and uncommon build configurations remain prerequisites where stated. A high-impact conditional candidate does not assert routine data loss. Entries without a standalone severity label describe their impact directly; severity labels are triage judgments, not measured runtime outcomes. No runtime reproduction is claimed.

## Coverage outcomes

“Read” below refers to the recorded implementation, local header, configuration and build passes. The later review log provides their consecutive-read and caller-contract evidence; historical progress statements there are superseded by this matrix and the final reconciliation.

| Component | Completed source/build outcome | Material limits and result |
|---|---|---|
| Shared storage | Five C units, local/exported headers, storage.conf and build | ID naming/allocation/concurrency findings; noexpunge startup resource and ReleaseID limitations recorded |
| ATA/ATAPI and PCI | Thirteen ATA C units, six PCI units, private/public interfaces and builds | Bounds, SCSI emulation, PIO/DMA/completion findings; larger native logical sectors explicitly conditional |
| ATA platforms | Five active Gayle/FastATA units; Amiga legacy override, Efika and Sam440 sources/builds | FastATA default probe disabled; legacy/inactive runtime observations keep those conditions |
| SATA/AHCI | Fifteen C units, nine local/exported headers, public hardware header and build | DMA, recovery, SCSI/accounting, lifecycle findings; identity/coherent PC mappings can mask portable-shim faults |
| NVMe | Twelve C units, nine headers, hardware/configuration/build declarations | Queues, PRPs, capacity, completion/timeouts, cleanup; unsupported page sizes/coherency and actual big-endian deployment bounded |
| VirtIO block | All eight C units, headers/configuration/build and PCI protocol contracts | Bounds, rings, mapping, reset, configuration and modern discovery findings; stable/conforming hosts can mask defensive omissions |
| Generic SCSI / WD33C93 | Ten generic units/six headers; four WD units plus intern/config/build | Generic translation/lifecycle findings; optional WD backend is skeletal and not represented as working hardware I/O |
| SD/MMC | Eight generic and five Broadcom units, public/private headers and all three platform selections | Addressing, timeout/DMA/listener, hotplug/geometry findings; SDHOST and SDHCI paths distinguished; completion-bit fallback limits corrected |
| ROM USB mass storage | Seven class/device/transport/partition C units, five headers/configuration/build | Bulk/CBI/UAS, queues, reconnect, geometry and mount findings; UAS primary protocol comparison and suspend timing bounded |
| Workbench USB mass storage | Six C units, headers/configuration and both build definitions | Chunking, cancellation/disconnect, mount and ownership findings; expunge/noexpunge applicability retained |
| PC and Amiga floppy | Both two-unit modules, headers/configurations/builds, public trackdisk API | DMA, cache/media, parser and arbitration findings; extended-command caller limits explicit |
| disk.resource | Local initialization/build, public resource API and arbitration callers | Allocation/arbitration atomicity findings; no generic hot-unload claim |
| RAM/file-backed devices | Both C devices, both gcc headers, configurations and Workbench build | Allocation/bounds/accounting/teardown outcomes; no extra undiscovered block backend in nvdisk |
| Hosted disks | Eight C units across shared/UNIX/Windows/Linux/Darwin plus headers/build overrides | Host offsets, short I/O, flush, worker lifetime and automount; dynamic host ABI behavior remains unexecuted |
| Disk-image device/plugins/audio | Twelve device units, all 32 plugin C files, seven audio adapters, ten DMS units, local/public headers and selectors | Every local block-image backend, optional parser/decoder and control path reviewed; codec provider implementations remain the dependency boundary |
| Disk-image support/preferences | Twenty-seven support and three preference C files; platform adapters, used GUI callbacks and builds | Allocation/path/XML/persistence/ABI outcomes; full general GUI/PNG rendering excluded explicitly |
| Disk-image compatibility libraries | z/bz2/Expat init/malloc/entry stubs, generated APIs and standalone/modern build choices | Wrapper/allocation/lifecycle findings; upstream generic algorithms and unrelated vendor declarations not exhaustively reviewed |
| Partition library | Thirty-two modern C units plus legacy initializer/platform override, headers/configuration/build | GPT/MBR/EBR/RDB bounds/width/checksum/stream/publication findings; parser features match selected builds |
| Boot/automount | Core partition scanner, bootstrap/cleanup/init and disk-presence menu slice; driver-specific registration | Name/geometry/offset/bootblock findings; general screen/menu animation and undocumented partition-boot ABI limited |
| Partition and HDTool commands | Both Partition units and all five HDTool units, local interfaces/builds | Creation/persistence/CLI/accounting findings; HDTool install/setup stubs do not perform media writes |
| Formatters | All four BHFormat units and SFSformat, local headers/builds and DOS/packet contracts | Confirmation, bounds, transfer negotiation, lifetime and result propagation findings; no formatting performed |
| PC/m68k/Parthenope installers | Complete GRUB/GRUB2/C:Install/m68k-host sources/builds; Parthenope installer, firmware RDB reader and callback support | Copied helper causes grouped; standalone/firmware selections distinguished; external GRUB and firmware callbacks excluded |
| Trackdisk preferences/IPrefs | Preference producer, local headers/modern and legacy builds, complete startup consumer | Native/serialized layouts distinguished; zero retry value reaches generic PC driver's decrementing error loops (GD2339) |
| QuickPart | All fourteen active units, local headers/build | GUI capacity/layout/ownership findings; Apply/Save are stubs and staged edits are not called media writes |
| HDToolBox | All ten modern units plus legacy debug, local headers/preferences/modern and legacy builds | Resize, ownership, identifier and export findings; failed saves retain changed state; invalid drag-shortening candidate withdrawn |
| InstallAROS | All eleven active units, every local header/build and option/GUI/DOS contracts | Partition command, unit selection, backup/copy/format/status/bootloader/path findings; post-install script is external and was not executed |

## Correctness dimensions and remaining assumptions

| Dimension | Coverage / conclusion |
|---|---|
| Addressing and bounds | Checked byte/LBA conversions, native sector sizes, capacity extents, pointer/native-width arithmetic, parser lengths, partition offsets, command negotiation and exported mount geometry. Exact hardware response to invalid commands is not assumed. |
| Transfer accounting | Checked io_Actual/scsi_Actual/residuals, host/DOS short counts, decoder output lengths, partial block handling and publication return values. Small used.len on a VirtIO write is not a media-byte count. |
| DMA and ownership | Checked descriptor/PRP/ring construction, address translation, bounce buffers, cache maintenance, setup failure, queue/task/IRQ lifetimes and timeout cleanup. Nonidentity/noncoherent backend effects remain conditional where the local provider does not prove them. |
| Concurrency/completion/cancellation | Traced queues, reply timing, unlocked walkers, signal failures, callback ownership, semaphores, explicit suspend and disconnect. No ordinary misuse claim is based only on closing a device with outstanding caller-owned I/O. |
| Timeout/reset/recovery | Checked polling budgets, lost IRQ paths, reset state transitions, busy recovery and post-timeout DMA ownership. SDHCI's bad completion bit has a compensating task recovery path; its separate unbounded Wait remains distinct. |
| Media/write protection | Checked change counters, geometry/cache refresh, removable task state, startup/open outcomes and writable flags. Device-side enforcement can prevent a particular invalid write; no unsupported media is presumed attached. |
| Flush/durability/publication | Checked UPDATE/flush barriers, cached host/image writes, filesystem flush attempts, Close outcomes and child/parent table publication. Final write/flush guarantees remain provider dependent; failed status propagation is documented separately from physical success. |
| Architecture/configuration | Checked public widths/endian packing, strict alignment, PPC asm constraints, platform selectors, optional parsers and legacy-only builds. A declared conditional big-endian path is not claimed deployed or hardware-tested. |

Additional bounded resource/support/helper observations are retained in the review log and relevant finding notes, rather than inflated into independent ordinary-path failures. All previous pending implementation/build gates have completed records or a stated scope/provider/configuration limit. The original pass verified its then-current IDs, anchors, alias/withdrawal accounting, source inventory and references; subsequent source cross-checks added findings. Abbreviated paths are relative to their stated component. The baseline commit and clean repository status were verified again. No project execution was required or performed.

## Findings by component

This index groups the current candidates for navigation. Cross-component root causes appear in more than one group. IDs are stable references, not a count of independent bugs; DISK-022, DISK-305, DISK-310, DISK-313 and DISK-314 are merged historical aliases; DISK-283 is withdrawn following arithmetic correction. Detailed evidence and review limitations remain in the sections below. This index does not assert completion of coverage or validation.

### Shared storage and cross-driver request contracts

- [DISK-001](#disk-001) — Shared storage ID formatting produces duplicate names
- [DISK-002](#disk-002) — Storage name allocations are dereferenced without checking failure
- [DISK-005](#disk-005) — Failed asynchronous submission removes the same request twice
- [DISK-006](#disk-006) — Asynchronous reply can precede BeginIO's final request access
- [DISK-009](#disk-009) — NVMe and AHCI DEVICEQUERY returns a pointer into its expired stack
- [DISK-010](#disk-010) — Disk drivers acknowledge CMD_UPDATE without flushing volatile writes
- [DISK-089](#disk-089) — NVMe and ATA boot-node open flags are uninitialized
- [DISK-102](#disk-102) — ATA device-query response ignores caller buffer bounds
- [DISK-106](#disk-106) — Shared storage ID allocation is not an atomic transaction

### ATA, IDE and platform backends

- [DISK-017](#disk-017) — ATA SCSI emulation ignores supplied data-buffer length
- [DISK-018](#disk-018) — ATA emulated READ CAPACITY reports sector count as last LBA
- [DISK-092](#disk-092) — ATA selection and diagnostic probing have unbounded busy waits
- [DISK-093](#disk-093) — ATA timeout emulation can loop forever with interrupts disabled
- [DISK-094](#disk-094) — Failed ATA first-write handshake retains the caller buffer in its IRQ handler
- [DISK-095](#disk-095) — PCI ATA passes reversed DMA directions to cache helpers
- [DISK-096](#disk-096) — PCI ATA interrupt acknowledgement clears DMA errors before checking them
- [DISK-097](#disk-097) — ATA bus-task creation failure leaves discovery semaphore locked
- [DISK-098](#disk-098) — Gayle PIO assembly mishandles short transfers and modified operands
- [DISK-099](#disk-099) — Gayle initializes the storage-controller method base from the wrong interface
- [DISK-100](#disk-100) — ATA probes read ownership markers after failure cleanup freed them
- [DISK-101](#disk-101) — PCI ATA frees a reference embedded inside its controller object
- [DISK-102](#disk-102) — ATA device-query response ignores caller buffer bounds
- [DISK-103](#disk-103) — ATA bus worker uses an unchecked timer allocation
- [DISK-104](#disk-104) — ATA fallback volume name is not terminated
- [DISK-105](#disk-105) — ATA expunge reuses and frees controllers already disposed by HW removal
- [DISK-107](#disk-107) — ATA bounce allocation failure submits DMA at address zero
- [DISK-108](#disk-108) — ATA polling treats BSY clear as complete before moving data
- [DISK-316](#disk-316) — PCI ATA interrupt-registration failure leaks its DMA buffer
- [DISK-320](#disk-320) — PCI ATA init releases bases against the wrong IDs after method-base failure
- [DISK-321](#disk-321) — AHCI init leaks partial attribute bases when initial acquisition fails
- [DISK-322](#disk-322) — PCI ATA never releases runtime PCI attribute-base references
- [DISK-325](#disk-325) — ATA reports TRIM success without running the operation
- [DISK-326](#disk-326) — ATA advertises SMART but rejects the command

### AHCI

- [DISK-009](#disk-009) — NVMe and AHCI DEVICEQUERY returns a pointer into its expired stack
- [DISK-010](#disk-010) — Disk drivers acknowledge CMD_UPDATE without flushing volatile writes
- [DISK-012](#disk-012) — AHCI emulated inquiry/capacity uses the command buffer as response storage
- [DISK-013](#disk-013) — AHCI disk path dereferences a NULL transfer when slots are exhausted
- [DISK-014](#disk-014) — AHCI DMA shim publishes virtual buffers and syncs NULL maps
- [DISK-015](#disk-015) — AHCI DMA shim emits an oversized single PRD
- [DISK-016](#disk-016) — AHCI VPD response padding advances a word pointer by a byte count
- [DISK-109](#disk-109) — AHCI recovery releases requests without establishing DMA has stopped
- [DISK-110](#disk-110) — AHCI allocation wrappers do not provide the assumed infallible allocations
- [DISK-111](#disk-111) — AHCI callout cancellation does not synchronize callback lifetime
- [DISK-112](#disk-112) — AHCI ignores helper and timeout-task creation failures
- [DISK-113](#disk-113) — AHCI media-monitor creation failure hangs registration
- [DISK-114](#disk-114) — AHCI failed attachment leaves registrations pointing at freed controller data
- [DISK-115](#disk-115) — AHCI attachment can leave initialized ports with interrupts masked
- [DISK-116](#disk-116) — AHCI native logical-sector size is assembled with reversed words
- [DISK-117](#disk-117) — Port-multiplier reset failure leaves the error CCB reserved
- [DISK-118](#disk-118) — AHCI boot-volume ID fallback dereferences NULL
- [DISK-119](#disk-119) — AHCI repeated detach accesses the freed CAM unit after probe failure
- [DISK-120](#disk-120) — AHCI autosense writes 32 bytes after validating only 18
- [DISK-121](#disk-121) — AHCI SCSI translation silently narrows LBAs and transfer counts
- [DISK-122](#disk-122) — AHCI ignores its computed residual transfer length
- [DISK-123](#disk-123) — AHCI disk writes ignore SCSI force-unit-access
- [DISK-124](#disk-124) — AHCI FORMAT validates against one track instead of disk capacity
- [DISK-125](#disk-125) — AHCI never runs its state machine after initial startup
- [DISK-126](#disk-126) — AHCI port-multiplier targets have no addressable public units
- [DISK-127](#disk-127) — AHCI standard-command batching can issue work after an exclusive command
- [DISK-128](#disk-128) — AHCI NCQ error-log tag can index beyond allocated CCBs

### NVMe

- [DISK-318](#disk-318) — NVMe failed submission-queue creation retains an unusable I/O queue
- [DISK-250](#disk-250) — NVMe namespace sector exponent is used without a supported-range check
- [DISK-005](#disk-005) — Failed asynchronous submission removes the same request twice
- [DISK-006](#disk-006) — Asynchronous reply can precede BeginIO's final request access
- [DISK-009](#disk-009) — NVMe and AHCI DEVICEQUERY returns a pointer into its expired stack
- [DISK-010](#disk-010) — Disk drivers acknowledge CMD_UPDATE without flushing volatile writes
- [DISK-011](#disk-011) — NVMe completion walkers retain a stale local head across unlock
- [DISK-082](#disk-082) — NVMe controller identify timeout leaves a live stack completion handler
- [DISK-083](#disk-083) — NVMe queue teardown frees memory while its interrupt handler remains registered
- [DISK-084](#disk-084) — NVMe identity fields are treated as unbounded C strings
- [DISK-085](#disk-085) — NVMe capacity scales native LBA counts twice
- [DISK-086](#disk-086) — NVMe completion readiness is published before completion work finishes
- [DISK-087](#disk-087) — NVMe failed queue setup frees a queue still used by its task
- [DISK-088](#disk-088) — NVMe startup does not handle essential allocation/task failures
- [DISK-089](#disk-089) — NVMe and ATA boot-node open flags are uninitialized
- [DISK-090](#disk-090) — NVMe PRP-list alignment mask truncates 64-bit pointers
- [DISK-091](#disk-091) — NVMe discovery consumes multibyte Identify fields without endian conversion

### VirtIO

- [DISK-251](#disk-251) — VirtIO configuration is read once without generation validation

- [DISK-003](#disk-003) — VirtIO silently rounds down misaligned byte offsets
- [DISK-004](#disk-004) — VirtIO checks starting sector but not full transfer extent
- [DISK-005](#disk-005) — Failed asynchronous submission removes the same request twice
- [DISK-006](#disk-006) — Asynchronous reply can precede BeginIO's final request access
- [DISK-007](#disk-007) — VirtIO DMA completion never performs CachePostDMA
- [DISK-008](#disk-008) — VirtIO copies fixed-width strings from short allocated buffers
- [DISK-129](#disk-129) — VirtIO ignores negotiated per-segment transfer limits
- [DISK-130](#disk-130) — VirtIO startup dereferences failed unit-array and storage-ID allocations
- [DISK-131](#disk-131) — VirtIO queue teardown does not revoke asynchronous users
- [DISK-132](#disk-132) — VirtIO accesses PCI BAR bus addresses as CPU memory without mapping
- [DISK-133](#disk-133) — VirtIO reset polling counter underflows and permits failed reset
- [DISK-134](#disk-134) — VirtIO capability offsets and lengths are never validated

### Hosted disks

- [DISK-006](#disk-006) — Asynchronous reply can precede BeginIO's final request access
- [DISK-021](#disk-021) — Hostdisk replies to a change-interrupt request while retaining its list node
- [DISK-023](#disk-023) — Hostdisk UPDATE overtakes queued writes and never flushes host storage
- [DISK-024](#disk-024) — Hostdisk write loop never progresses on a zero-byte successful host write
- [DISK-025](#disk-025) — UNIX hostdisk truncates 64-bit seek results before checking failure
- [DISK-026](#disk-026) — UNIX Host_Open erases the allocated-name ownership flag
- [DISK-135](#disk-135) — Hostdisk worker port failure leaves the open result unset
- [DISK-136](#disk-136) — Hostdisk does not join unit workers before library cleanup

### Floppy, disk.resource and RAM/file-backed devices

- [DISK-044](#disk-044) — PC floppy result collection never advances its destination
- [DISK-045](#disk-045) — PC floppy cache survives medium replacement
- [DISK-046](#disk-046) — PC floppy DMA replacement does not ensure boundary safety
- [DISK-047](#disk-047) — Floppy addressing truncates unchecked offsets
- [DISK-048](#disk-048) — Amiga floppy sub-sector requests make no progress
- [DISK-049](#disk-049) — Amiga shared buffer switch discards another unit's dirty writes
- [DISK-050](#disk-050) — Amiga HD buffer allocation failure leaves dangling pointers
- [DISK-051](#disk-051) — Amiga track-cache partial allocation failure causes null copy
- [DISK-052](#disk-052) — Amiga raw-track parsers dereference before checking the end
- [DISK-053](#disk-053) — Amiga read cache is not invalidated on insertion or CMD_CLEAR
- [DISK-054](#disk-054) — RAM disk bounds check wraps before validation
- [DISK-080](#disk-080) — Amiga floppy arbitration leaves wakeup messages on its reply port
- [DISK-081](#disk-081) — disk.resource unit allocation is not atomic

### Partition command and HDTool integration

- [DISK-254](#disk-254) — HDTool passes uninitialized autosense fields to ATA emulation
- [DISK-255](#disk-255) — Partition command discards table-write failures
- [DISK-256](#disk-256) — Partition command reads pointer-width attributes written as LONG
- [DISK-257](#disk-257) — Partition command dereferences a missing extended container on a full MBR
- [DISK-258](#disk-258) — Pure-RDB MAXWORK can omit the requested work partition

- [DISK-269](#disk-269) — Partition WIPE submits an unchecked buffer and ignores erasure failures
- [DISK-270](#disk-270) — HDTool queries active status for bootable and automount displays

### Format and configuration integration

- [DISK-264](#disk-264) — BHFormat device listing retains a pointer to an expired InfoData local
- [DISK-265](#disk-265) — BHFormat CLI proceeds after failed or exhausted confirmation input

- [DISK-266](#disk-266) — BHFormat uses 64-bit commands at an unqueried exactly-4-GiB boundary
- [DISK-267](#disk-267) — BHFormat GUI reads an uninitialized volume-name string
- [DISK-268](#disk-268) — BHFormat GUI exits success after a failed format callback

### Boot installer integration

- [DISK-259](#disk-259) — Installer FFS collectors probe a node past the boot-sector buffer
- [DISK-260](#disk-260) — GRUB boot-volume helpers dereference failed volume creation
- [DISK-261](#disk-261) — Boot installers return success for failed installation paths
- [DISK-262](#disk-262) — C:Install replaces bootblock after failed preservation write
- [DISK-263](#disk-263) — Partition and installer I/O retains ordinary commands after required 64-bit negotiation fails

### Partition library

- [DISK-055](#disk-055) — GPT backup table placement mixes bytes and sectors
- [DISK-056](#disk-056) — GPT header CRC reads an unchecked header size
- [DISK-057](#disk-057) — GPT entry size/count are used without structural or overflow validation
- [DISK-058](#disk-058) — GPT stops advancing at unused partition entries
- [DISK-059](#disk-059) — GPT 64-bit partition addresses narrow to 32-bit geometry
- [DISK-060](#disk-060) — NSD capability selection can wrap capacity arithmetic on 32-bit builds
- [DISK-061](#disk-061) — Destroying GPT writes its header over sector zero
- [DISK-062](#disk-062) — Cyclic EBR chains have no traversal bound or cycle detection
- [DISK-063](#disk-063) — EBR addition dereferences an allocation failure
- [DISK-064](#disk-064) — MBR partition position is used as an unchecked array index
- [DISK-065](#disk-065) — Adding an MBR partition to an occupied explicit slot aliases handles
- [DISK-066](#disk-066) — RDB checksum length is trusted before structural validation
- [DISK-067](#disk-067) — RDB partition name lengths overrun fixed buffers
- [DISK-068](#disk-068) — RDB environment-table length overruns the partition handle
- [DISK-069](#disk-069) — RDB linked-block traversal has no cycle detection
- [DISK-070](#disk-070) — RDB filesystem reads use sector size with 512-byte allocations
- [DISK-071](#disk-071) — RDB writer returns success after failed metadata transfers
- [DISK-072](#disk-072) — RDB bad-block-list head is written in host byte order
- [DISK-073](#disk-073) — New RDB partition geometry is never initialized
- [DISK-074](#disk-074) — RDB filesystem stream seek addresses only one LSEG payload
- [DISK-075](#disk-075) — RDB filesystem stream reads past end on repeated reads
- [DISK-076](#disk-076) — RDB load path accepts missing or partially read filesystem data
- [DISK-077](#disk-077) — RDB filesystem ID lookup omits big-endian conversion
- [DISK-078](#disk-078) — Filesystem registration leaks its entry when loading fails
- [DISK-079](#disk-079) — RDB metadata writer has no reserved-area capacity check

### SCSI framework

- [DISK-243](#disk-243) — WD33C93 build lists a C file containing a raw diff header
- [DISK-137](#disk-137) — Empty SCSI subsystem expunge leaves its daemon running
- [DISK-138](#disk-138) — SCSI build includes a nonexistent case-sensitive header name

### Disk-image device, parsers, audio and preferences

- [DISK-245](#disk-245) — Disk-image name query checks the output pointer instead of allocation result
- [DISK-246](#disk-246) — Disk-image mount persists an uninitialized path after name-query failure
- [DISK-244](#disk-244) — PPC disk-image endian stores declare written memory as input only
- [DISK-037](#disk-037) — AROS disk-image seeks silently truncate 64-bit offsets
- [DISK-038](#disk-038) — CISO index allocation arithmetic overflows
- [DISK-039](#disk-039) — CISO compressed extent can exceed its staging buffer
- [DISK-040](#disk-040) — DAX frame lengths exceed the allocated input buffer
- [DISK-041](#disk-041) — DAX checks only the first uncompressed area
- [DISK-042](#disk-042) — Disk-image CD audio reads bypass SCSI buffer bounds
- [DISK-043](#disk-043) — Generic image writes do not enforce advertised medium size
- [DISK-330](#disk-330) — ISO high-offset reads wrap the raw-sector seek and return earlier data
- [DISK-139](#disk-139) — MDS accepts unsupported track modes with unsafe sector buffers
- [DISK-140](#disk-140) — MDS accepts images with no tracks and later dereferences NULL
- [DISK-141](#disk-141) — NRG DAO track count exceeds allocated metadata
- [DISK-142](#disk-142) — GI reads from stale file positions
- [DISK-143](#disk-143) — CPC multi-track transfers overrun the track-offset array
- [DISK-144](#disk-144) — CPC accepts zero sizes and divides by zero
- [DISK-145](#disk-145) — Crypt bounds clipping underflows beyond the end of the image
- [DISK-146](#disk-146) — UIF allocates hash entries as eight bytes on pointer-wide builds
- [DISK-147](#disk-147) — UIF index size multiplication can underallocate metadata
- [DISK-148](#disk-148) — UIF capacity arithmetic wraps before hash allocation
- [DISK-149](#disk-149) — UIF accepts incomplete decompression as full blocks
- [DISK-150](#disk-150) — XAD missing-library failure frees uninitialized pointers
- [DISK-151](#disk-151) — Temporary extraction overwrites a predictable existing file
- [DISK-152](#disk-152) — IPF initialization failure uses an uninitialized device pointer
- [DISK-153](#disk-153) — IPF MFM parser's end guard omits lookahead
- [DISK-154](#disk-154) — DAA spanning compressed blocks seek to the wrong volume offset
- [DISK-155](#disk-155) — DAA exposes unwritten decompression bytes and excess final capacity
- [DISK-156](#disk-156) — ADC back-reference decoding reads beyond compressed input and loses cursor position
- [DISK-157](#disk-157) — ADC emits repeated bytes instead of copying back-references
- [DISK-158](#disk-158) — Empty cleaned DMG base64 data reads before its buffer
- [DISK-159](#disk-159) — DMG resource run count is trusted beyond the resource buffer
- [DISK-160](#disk-160) — DMG zero-sized runs can stall transfers
- [DISK-161](#disk-161) — DMS RLE decoder has no compressed-input bound
- [DISK-162](#disk-162) — DMS ignores track numbers when placing extracted data
- [DISK-163](#disk-163) — DMS dictionary matches exceed the output endpoint
- [DISK-164](#disk-164) — DMS Heavy tree metadata overflows fixed arrays
- [DISK-165](#disk-165) — FDI header constructor publishes unchecked work-buffer allocations
- [DISK-166](#disk-166) — FDI Huffman root is uninitialized before tree expansion
- [DISK-167](#disk-167) — FDI maximum track count writes past its offset table
- [DISK-168](#disk-168) — FDI checks token input bounds only after consuming operands
- [DISK-169](#disk-169) — FDI pulse decoder has no output or timing capacity checks
- [DISK-170](#disk-170) — FDI raw-track length overflows the internal destination
- [DISK-171](#disk-171) — FDI normal-track dispatch omits its final array entry
- [DISK-172](#disk-172) — FDI low pulse timings can hang decoding
- [DISK-253](#disk-253) — Legacy Expat clib prototypes omit parameter types
- [DISK-173](#disk-173) — Disk-image compatibility allocators free the original allocation when realloc fails
- [DISK-174](#disk-174) — WAV/AIFF end-of-stream clipping can overwrite the caller buffer
- [DISK-175](#disk-175) — AIFF compression identifier is compared without endian conversion
- [DISK-176](#disk-176) — FLAC stream cleanup leaks PCM buffers
- [DISK-177](#disk-177) — mpg123 writes size_t output count into a LONG local
- [DISK-178](#disk-178) — WavPack malformed binary cuesheet starts parsing past allocation
- [DISK-179](#disk-179) — CUE repeats global codec initialization for each plugin descriptor
- [DISK-180](#disk-180) — CUE first-track nonzero index is ignored
- [DISK-181](#disk-181) — Optional CUE build references nonexistent audio source paths
- [DISK-182](#disk-182) — Empty disk-image SCSI command is dereferenced before validation
- [DISK-183](#disk-183) — Filtered disk-image TOC changes the reported last track
- [DISK-184](#disk-184) — Unit control retains a caller error pointer beyond the request
- [DISK-185](#disk-185) — Change-notification allocation failure never completes its request
- [DISK-186](#disk-186) — Reused error-string capacity can overwrite a later smaller buffer
- [DISK-187](#disk-187) — MountImage caller register definition disagrees with its implementation
- [DISK-188](#disk-188) — Disk-image preferences emit unescaped filenames into XML
- [DISK-240](#disk-240) — Disk-image preference saves discard write/close failures and overwrite the previous file

### SD/MMC and Broadcom backends

- [DISK-249](#disk-249) — SD 32-bit reads compare against a truncated sector capacity
- [DISK-019](#disk-019) — SD reports every card as write-protected despite implemented writes
- [DISK-020](#disk-020) — MMC high-speed switch treats success as failure
- [DISK-189](#disk-189) — SD change-notification list is never initialized
- [DISK-190](#disk-190) — SD reinsertion writes past the single-slot unit allocation
- [DISK-191](#disk-191) — SD Open uses global unit number as a per-bus array index
- [DISK-192](#disk-192) — SDHCI interrupt-mode completion wait has no timeout
- [DISK-193](#disk-193) — SDHCI software timeout relinquishes DMA storage without stopping DMA
- [DISK-194](#disk-194) — SDHCI listener claiming does not wait for an in-progress IRQ callback
- [DISK-195](#disk-195) — SDHOST narrows caller data pointers to 32 bits on aarch64
- [DISK-196](#disk-196) — SDHOST ignores data status errors and command-idle timeout
- [DISK-197](#disk-197) — SD platform mailbox allocation failure is dereferenced
- [DISK-198](#disk-198) — Combined aarch64 SD build defines GetClockDiv twice
- [DISK-199](#disk-199) — Broadcom SD delay arithmetic fails across microsecond timer wrap
- [DISK-200](#disk-200) — SDHOST selection failure calls a NULL SDHCI accessor
- [DISK-201](#disk-201) — SD unit initialization can overlap another controller transaction
- [DISK-202](#disk-202) — SDHCI PIO partial transfers can be reported at full length
- [DISK-203](#disk-203) — Failed SD signal allocation produces invalid signal masks
- [DISK-204](#disk-204) — SD media changes do not invoke registered notification handling
- [DISK-233](#disk-233) — SDHCI transfer completion uses the block-gap interrupt bit

### USB ROM transports, block translation and lifecycle

- [DISK-248](#disk-248) — USB change-notification traversal races with handler removal
- [DISK-247](#disk-247) — USB CDB construction stores ULONG through unaligned byte-array pointers
- [DISK-241](#disk-241) — USB transfer-limit autodetection divides by an unavailable block size
- [DISK-242](#disk-242) — USB transfer-limit autodetection retains the failing probe limit
- [DISK-033](#disk-033) — Poseidon CBI reads a second status after already consuming it
- [DISK-034](#disk-034) — Poseidon autosense inspects bytes beyond the supplied sense buffer
- [DISK-252](#disk-252) — UAS command and status structures disagree with the implemented wire protocol
- [DISK-035](#disk-035) — Poseidon UAS accepts empty/truncated status as successful completion
- [DISK-036](#disk-036) — Poseidon bulk NAK-timeout retry budget renews indefinitely
- [DISK-205](#disk-205) — ROM USB cancellation/flush ignores the actual request queue
- [DISK-206](#disk-206) — UAS reconnect retains endpoint pointers from the previous device
- [DISK-207](#disk-207) — USB reported block size is trusted as a shift without validation
- [DISK-208](#disk-208) — USB geometry combines zero mode-page heads with division
- [DISK-209](#disk-209) — USB write alignment expression compares inside the shift
- [DISK-210](#disk-210) — USB seek passes an uninitialized command pointer
- [DISK-211](#disk-211) — USB large-block emulation drops high LBA bits
- [DISK-212](#disk-212) — USB emulation counts a partial native-block tail as transferred
- [DISK-213](#disk-213) — USB packet subclass copies arbitrary CDB length into twelve-byte stack buffer
- [DISK-214](#disk-214) — USB READ6/WRITE6 zero count translates to zero transfer
- [DISK-215](#disk-215) — Default SimpleSCSI rejects the driver's own large-disk CDB16
- [DISK-216](#disk-216) — Mode translation allocation failure submits incompatible original buffer
- [DISK-217](#disk-217) — SimpleSCSI all-pages response writes into a zero-length buffer
- [DISK-218](#disk-218) — USB media replacement preserves the previous medium's block size
- [DISK-219](#disk-219) — Removable task leaks its IO reply port on every restart
- [DISK-220](#disk-220) — USB automount dereferences unvalidated DOS startup values
- [DISK-221](#disk-221) — CBI command-stall retry can return success before transferring data or status
- [DISK-222](#disk-222) — USB block wrappers credit requested bytes after short successful transport transfers
- [DISK-223](#disk-223) — USB mode-page parser accepts incomplete pages and miscounts block descriptors
- [DISK-231](#disk-231) — USB persisted transfer limit is used as an unchecked shift count
- [DISK-232](#disk-232) — USB raw configuration strings lack termination validation before mount conversion
- [DISK-234](#disk-234) — ROM USB publishes units whose worker startup failed
- [DISK-235](#disk-235) — ROM USB fails to reject exhaustion of the unit-port signal
- [DISK-236](#disk-236) — USB emulation scratch allocation failure is not returned in the IO request
- [DISK-237](#disk-237) — USB READ/WRITE(16) count construction shifts a 32-bit value beyond its width
- [DISK-238](#disk-238) — Synthesized USB inquiry reports zero returned bytes
- [DISK-239](#disk-239) — ROM USB class expunge retains its utility-library reference

### USB Workbench implementation

- [DISK-027](#disk-027) — Workbench USB writes use the original buffer/length for every chunk
- [DISK-028](#disk-028) — Workbench USB passthrough can overflow its 16-byte stack CDB
- [DISK-029](#disk-029) — Workbench USB transport ignores data-phase failure/residue
- [DISK-030](#disk-030) — Workbench USB disconnect skips queued requests indefinitely
- [DISK-031](#disk-031) — Workbench USB cache compares NULL optional device strings
- [DISK-032](#disk-032) — Workbench USB geometry reports last LBA as total sector count
- [DISK-224](#disk-224) — Workbench USB whole-device mount expresses sectors as cylinders
- [DISK-225](#disk-225) — Workbench USB discovery dereferences unchecked boot-node allocation
- [DISK-226](#disk-226) — USB partition automounters divide by unvalidated cylinder geometry
- [DISK-227](#disk-227) — USB mounting leaks copied parameter packets on success and skipped geometry
- [DISK-229](#disk-229) — Automatic partition mounting ignores explicit no-mount flags
- [DISK-323](#disk-323) — USB MassStorage reports non-CD SCSI media writable when write-protected
- [DISK-324](#disk-324) — SCSI device never deletes its memory pool

### Boot and mount integration

- [DISK-220](#disk-220) — USB automount dereferences unvalidated DOS startup values
- [DISK-224](#disk-224) — Workbench USB whole-device mount expresses sectors as cylinders
- [DISK-225](#disk-225) — Workbench USB discovery dereferences unchecked boot-node allocation
- [DISK-226](#disk-226) — USB partition automounters divide by unvalidated cylinder geometry
- [DISK-227](#disk-227) — USB mounting leaks copied parameter packets on success and skipped geometry
- [DISK-228](#disk-228) — Boot scanner duplicate-name suffix overflows a full partition-name buffer
- [DISK-229](#disk-229) — Automatic partition mounting ignores explicit no-mount flags
- [DISK-230](#disk-230) — Bootblock checksum loop wraps its 16-bit index for large bootblocks

### Additional formatter and partition-management consumers

- [DISK-271](#disk-271) — BHFormat leaks the device-list read lock when a Workbench volume has no matching device
- [DISK-272](#disk-272) — QuickPart divides by zero when every enumerated drive lacks usable capacity
- [DISK-273](#disk-273) — QuickPart disk objects omit device and root-partition ownership cleanup
- [DISK-274](#disk-274) — QuickPart loses trailing free-space markers when inserting a partition
- [DISK-275](#disk-275) — QuickPart subtracts a sector from every leaf partition capacity
- [DISK-276](#disk-276) — QuickPart dereferences failed partition-display node allocations
- [DISK-277](#disk-277) — HDToolBox leaves a freed node linked after partition creation fails
- [DISK-278](#disk-278) — HDToolBox endpoint-only resize validation allows overlapping partitions
- [DISK-279](#disk-279) — HDToolBox cleans up nested partition tables after their handles have been freed
- [DISK-280](#disk-280) — HDToolBox writes a mountlist icon using a freed pathname
- [DISK-281](#disk-281) — HDToolBox exports nested partitions with an out-of-bounds unit lookup
- [DISK-282](#disk-282) — HDToolBox copies unbounded type identifiers into fixed 32-byte arrays
- [DISK-283](#disk-283) — Withdrawn: drag-rounding example was arithmetically incorrect

### InstallAROS, additional formatters and platform boot installers

- [DISK-284](#disk-284) — InstallAROS accepts a backup that was never opened before overwriting the original
- [DISK-285](#disk-285) — InstallAROS overflows its partition command buffer with ordinary option combinations
- [DISK-286](#disk-286) — InstallAROS continues installation after a requested format fails
- [DISK-287](#disk-287) — InstallAROS ignores Cancel in the GRUB1 filesystem compatibility warning
- [DISK-288](#disk-288) — InstallAROS marks bootloader progress complete without checking execution failure
- [DISK-289](#disk-289) — InstallAROS truncates the selected disk unit to eight bits
- [DISK-290](#disk-290) — InstallAROS looks for GRUB1 stage files in the wrong source directory
- [DISK-291](#disk-291) — InstallAROS advances to its completion page after recorded installation failure
- [DISK-292](#disk-292) — Installers copy unbounded device prefixes into 32-byte stack buffers
- [DISK-293](#disk-293) — InstallAROS and GRUB helpers traverse the DOS device list without its lock
- [DISK-294](#disk-294) — InstallAROS environment helpers overflow their fixed pathname buffer
- [DISK-295](#disk-295) — The m68k floppy host installer mishandles short reads and writes
- [DISK-296](#disk-296) — Parthenope RDB checksums treat a byte limit as a checksum word count
- [DISK-297](#disk-297) — Parthenope's standalone installer reads a full payload past the final boot-file fragment
- [DISK-298](#disk-298) — Parthenope's standalone installer allocates boot blocks outside the declared reserved range
- [DISK-299](#disk-299) — Parthenope's standalone installer commits a new boot-chain head after allocation or I/O failure
- [DISK-300](#disk-300) — SFSformat returns success after reported formatting failure
- [DISK-301](#disk-301) — Parthenope repeatedly reads the same partition block after a bad checksum
- [DISK-302](#disk-302) — Parthenope copies an RDB counted partition name as an unbounded C string

### Additional normalized driver and consumer findings

- [DISK-303](#disk-303) — VirtIO identifies modern block devices by subsystem ID instead of product ID
- [DISK-304](#disk-304) — HDTool dereferences an omitted optional argument before reporting usage
- [DISK-305](#disk-305) — Merged into DISK-244
- [DISK-306](#disk-306) — ATA PCI probe error exits unmap the physical BAR instead of its mapping
- [DISK-307](#disk-307) — ROM USB mass storage allows explicit suspend without draining active I/O
- [DISK-308](#disk-308) — Disk-image Reaction GUI checks the wrong hook after allocation
- [DISK-309](#disk-309) — ATA assumes 512-byte logical sectors for every non-ATAPI disk
- [DISK-310](#disk-310) — Merged into DISK-233
- [DISK-311](#disk-311) — InstallAROS counts incomplete file transfers as successfully copied files
- [DISK-312](#disk-312) — HDToolBox mountlist export rounds away a parent partition's sector offset
- [DISK-313](#disk-313) — Merged into DISK-245
- [DISK-314](#disk-314) — Merged into DISK-246

## Retained candidate details

<a id="disk-001"></a>

### DISK-001 — Shared storage ID formatting produces duplicate names

- **Severity:** High (device naming ambiguity). **Confidence:** High.
- **Source:** `rom/hidds/base/storage/storage_class.c`, `StorageHW__Hidd_Storage__AllocateID`, decimal formatting after `startnum`.
- **Trigger/trace:** Allocate a common stem through index 100. The hundreds branch emits `1`, reduces the remainder to zero, skips the tens branch, then emits `0`: index 100 becomes `HD10`, already assigned at index 10. Similarly 101 becomes `HD11`; values above 999 emit a nondecimal first character.
- **Consequence:** Duplicate boot-device identifiers can misidentify disks. Applies across drivers using the shared allocator.
- **Limits:** Exact DOS handling of duplicate boot names remains to be traced. The formatter defect itself does not depend on it.

<a id="disk-002"></a>

### DISK-002 — Storage name allocations are dereferenced without checking failure

- **Severity:** High (kernel crash under memory pressure). **Confidence:** High.
- **Source:** `rom/hidds/base/storage/storage_class.c`, both name-buffer `AllocVec` calls in `AllocateID`.
- **Trigger/trace:** Stem/node allocation succeeds but its subsequent name allocation returns NULL. `CopyMem` immediately writes through the NULL destination.
- **Consequence:** Crash during disk discovery; all consumers of this allocator are affected.

<a id="disk-003"></a>

### DISK-003 — VirtIO silently rounds down misaligned byte offsets

- **Severity:** High (wrong-sector writes). **Confidence:** High.
- **Source:** `rom/devs/virtio/virtio_io.c:48`, `:74`.
- **Trigger/trace:** Write 512 bytes at byte offset 1 on a 512-byte-sector unit. Bounds and length checks pass; header sector is `off64 >> 9`, hence zero. No offset-alignment check rejects the request.
- **Consequence:** Writes target the preceding sector; reads return the wrong bytes. Larger advertised logical sectors also lack alignment enforcement.
- **Contracts checked:** NVMe explicitly rejects misaligned byte offsets; the shared commands represent byte offsets.

<a id="disk-004"></a>

### DISK-004 — VirtIO checks starting sector but not full transfer extent

- **Severity:** Medium. **Confidence:** High.
- **Source:** `rom/devs/virtio/virtio_io.c:48`.
- **Trigger/trace:** A two-sector transfer beginning at the last sector passes validation because only the starting LBA is compared with `au_High`.
- **Consequence:** Invalid I/O is submitted beyond device capacity instead of rejected locally. Backend behavior determines whether any partial operation occurs; no out-of-capacity successful write is assumed.

<a id="disk-005"></a>

### DISK-005 — Failed asynchronous submission removes the same request twice

- **Severity:** High (request-list corruption). **Confidence:** High.
- **Source:** VirtIO `virtio_io.c:78`, `:81`, `:114`, `:117`, `:269`; NVMe `nvme_io.c:135`, submission-failure return, `:415`; `rom/exec/remove.c`.
- **Trigger/trace:** The helper removes the request before submission. A full queue or failed mapping/submission returns synchronous completion; `BeginIO` then removes it again. `Remove` retains stale links. Another task can insert a request between the two removals, and the second removal rewrites the old neighbors, unlinking the new request.
- **Consequence:** Corrupted outstanding-request tracking under concurrent I/O and submission failure. Without an intervening list mutation, the repeated removal happens to rewrite the same links.

<a id="disk-006"></a>

### DISK-006 — Asynchronous reply can precede BeginIO's final request access

- **Severity:** High (use after reply/request reuse). **Confidence:** High.
- **Source:** VirtIO `virtio_queue.c:285`, `virtio_io.c:272`; NVMe `nvme_io.c:187`, `:418`, completion task to be reviewed further.
- **Trigger/trace:** An I/O completion task runs after queue publication but before `BeginIO` clears `IOF_QUICK`. VirtIO replies unconditionally, returning ownership to the caller; `BeginIO` later modifies `io_Flags`. A caller on another CPU can already reuse/free the request. VirtIO's completion task priority is 21, so this is not limited to slow devices or hypothetical source ordering.
- **Consequence:** Access after ownership transfer and possible corruption of a reused/freed request. NVMe shares the publication-before-flag-clear pattern, and `nvme_busclass.c:98` replies from the completion task. AHCI clears the flag in its submission helper but still writes it again in BeginIO after asynchronous completion is possible.

- **Additional affected source (former DISK-022):** `arch/all-hosted/devs/hostdisk/hostdisk_device.c`, asynchronous BeginIO cases. PutMsg can wake the unit task before the caller clears IOF_QUICK. This shares the publication/completion ownership race above and is counted once.

<a id="disk-007"></a>

### DISK-007 — VirtIO DMA completion never performs CachePostDMA

- **Severity:** High on noncoherent targets. **Confidence:** High for contract violation; target applicability pending build mapping.
- **Source:** `rom/devs/virtio/virtio_queue.c:182`, `:188`, `:199`, `:267`; `rom/exec/cachepredma.c`, `cachepostdma.c` contract comments.
- **Trigger/trace:** Header/data/status buffers are mapped through `CachePreDMA`. Completion reads status and replies without any corresponding `CachePostDMA`; failure paths also leave pre-DMA operations unpaired.
- **Consequence:** On targets requiring cache maintenance, stale status/data or unreleased mappings. Generic x86 implementations are no-ops, so this finding does not claim an x86 cache failure.

<a id="disk-008"></a>

### DISK-008 — VirtIO copies fixed-width strings from short allocated buffers

- **Severity:** Medium (out-of-bounds reads). **Confidence:** High.
- **Source:** `rom/devs/virtio/virtio_busclass.c:83`, `:87`, `:91`; `virtio_unitclass.c:23`, `:53`–`:55`.
- **Trigger/trace:** Bus strings are allocated with `strlen+1`, including an empty serial string and firmware `1.0`. Unit construction passes these buffers to a routine reading exactly 40/20/8 bytes backwards, ignoring the first terminator.
- **Consequence:** Reads beyond heap allocations during normal attach and can fault if an allocation ends near inaccessible memory. The copied strings still terminate at their original in-bounds NUL, so this finding does not claim adjacent bytes appear in exported metadata.

<a id="disk-009"></a>

### DISK-009 — NVMe and AHCI DEVICEQUERY returns a pointer into its expired stack

- **Severity:** High (invalid command metadata). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_io.c:206`, `:321`; `rom/devs/ahci/ahci_io.c`, local `NSDSupported` and DEVICEQUERY assignment.
- **Trigger/trace:** `NSDSupported` is an automatic array inside `BeginIO`. DEVICEQUERY stores its address in the caller's output. After `BeginIO` returns, another call overwrites that storage before the caller walks the list.
- **Consequence:** Invalid/garbled supported commands and possible unbounded command-list reads. VirtIO uses a static array and avoids this defect.

<a id="disk-010"></a>

### DISK-010 — Disk drivers acknowledge CMD_UPDATE without flushing volatile writes

- **Severity:** High (durability/data loss). **Confidence:** High, conditional on controller volatile cache.
- **Source:** `rom/devs/nvme/nvme_io.c:388`, `CMD_UPDATE`; `rom/devs/ata/ata.c`, `cmd_Update`; `rom/devs/ahci/ahci_io.c`, `CMD_UPDATE`; ATA `scsiemu.c` also silently succeeds SYNCHRONIZE CACHE; `workbench/devs/USB/classes/MassStorage/storage_device.c:58-61,474-481` and `storageclass.c:376-396`.
- **Trigger/trace:** A write completes into a device/controller volatile cache and a caller requests CMD_UPDATE. NVMe/ATA/AHCI and the USB Bulk-Only MassStorage device acknowledge without issuing a media-cache flush. In the Workbench USB implementation, CMD_UPDATE maps to `cmd_Reset()`, which sends the Bulk-Only Mass Storage Reset class request through `HIDD_USBStorage_Reset()`; that method returns TRUE without issuing SCSI SYNCHRONIZE CACHE. Selected filesystem callers use CMD_UPDATE as their sync step: AFS `flush()` does DoIO then clears its cache (`rom/filesys/afs/os_aros_support.c:575-587`), while FAT, SFS, PFS3 and NTFS also submit the command. The Sam440/Efika HIDD manifests select this Workbench USB MassStorage implementation (`workbench/devs/USB/classes/MassStorage/mmakefile.src:5-10`; boot manifests cited in DISK-323).
- **Consequence:** A caller relying on update completion for durable storage can lose acknowledged writes on power loss while the device retains them in volatile cache. No claim is made for devices without volatile write caches. Implement a device-supported synchronization command where available and report unsupported/failed synchronization rather than returning success.

<a id="disk-011"></a>

### DISK-011 — NVMe completion walkers retain a stale local head across unlock

- **Severity:** High. **Confidence:** High on SMP.
- **Source:** `rom/devs/nvme/nvme_hw.c`, `nvme_process_cq`, unlock/relock around `nvme_complete_event`.
- **Trigger/trace:** Walker A consumes entry i and publishes head i+1, drops the queue lock for its hook. Walker B consumes i+1 and advances the shared head/phase. A reacquires the lock but continues from its old local head/phase, consuming i+1 again and potentially publishing an older head. A second walker can also ring the completion-head doorbell while A still references its entry, permitting device reuse despite the comment claiming that cannot happen.
- **Consequence:** Duplicate completions, incorrect CQ accounting, and potential slot/DMA cleanup races. Applies to SMP interrupt/poll concurrency; no reproduction claimed.

<a id="disk-012"></a>

### DISK-012 — AHCI emulated inquiry/capacity uses the command buffer as response storage

- **Severity:** High. **Confidence:** High.
- **Source:** `rom/devs/ahci/ahci_scsi.c`, `ahci_scsi_disk_io`, `rdata`/`rdata_len` initialization and INQUIRY/READ_CAPACITY cases; `compiler/include/devices/scsidisk.h`.
- **Trigger/trace:** A normal six-byte INQUIRY CDB and separate 36-byte data buffer is rejected because the code compares CDB length with SHORT_INQUIRY_LENGTH. READ CAPACITY uses its ten-byte CDB as output, clears/overwrites it, and leaves the actual data buffer untouched.
- **Consequence:** Standard discovery commands fail or return incorrect data and destroy the caller's command. Output must use `scsi_Data`/`scsi_Length`.

<a id="disk-013"></a>

### DISK-013 — AHCI disk path dereferences a NULL transfer when slots are exhausted

- **Severity:** High (kernel crash). **Confidence:** High.
- **Source:** `rom/devs/ahci/ahci_scsi.c`, `ahci_scsi_disk_io`; `rom/devs/ahci/ahci.c:3871`, `ahci_ata_get_xfer`.
- **Trigger/trace:** All CCBs are busy, so get_xfer returns NULL. Unlike the ATAPI path, the disk path does not check it and uses `xa->flags`/`xa->fis`, or passes NULL to put_xfer for an immediate command.
- **Consequence:** Concurrent disk I/O can crash instead of reporting busy/queueing.

<a id="disk-014"></a>

### DISK-014 — AHCI DMA shim publishes virtual buffers and syncs NULL maps

- **Severity:** High on nonidentity/noncoherent targets. **Confidence:** High for source defect; supported-target mapping pending.
- **Source:** `rom/devs/ahci/ahci_aros_bus.c`, `bus_dmamap_create/load/sync`; `ahci.c:2124` and PRDT callback.
- **Trigger/trace:** Map creation sets the map to NULL. Load reports one segment at `(bus_addr_t)data`, ignores physical contiguity and tag limits, and never associates data with the map. Sync consequently calls CachePreDMA/CachePostDMA on NULL for user data instead of the transfer buffer. Returned DMA addresses are ignored.
- **Consequence:** Incorrect DMA on targets where CPU virtual addresses differ from bus addresses or data requires cache maintenance. Identity/coherent x86 may mask these faults.

<a id="disk-015"></a>

### DISK-015 — AHCI DMA shim emits an oversized single PRD

- **Severity:** High (incorrect transfer/possible partial writes). **Confidence:** High.
- **Source:** `rom/devs/ahci/ahci_aros_bus.c`, `bus_dmamap_load`; `ahci.c:2167`, `ahci_load_prdt_callback`; `ahci.h`, AHCI_MAXPHYS.
- **Trigger/trace:** Issue an aligned transfer larger than 4 MiB directly. BeginIO does not enforce advertised max transfer; the shim does not enforce tag maxsize/maxsegsz or split data. Callback writes `len-1` into one PRD's flags, whose byte count field is only 22 bits.
- **Consequence:** Hardware sees a shorter transfer than the software supplied. Transfer-size contract/protocol constants must be checked during final validation; normal filesystem requests respecting 2 MiB DE_MAXTRANSFER avoid this trigger.

<a id="disk-016"></a>

### DISK-016 — AHCI VPD response padding advances a word pointer by a byte count

- **Severity:** High (buffer overwrite). **Confidence:** High.
- **Source:** `rom/devs/ahci/ahci_scsi.c:313`, `ahci_scsi_page_inquiry`; `compiler/include/devices/scsidisk.h`, UWORD pointer type of scsi_Data.
- **Trigger/trace:** Request a VPD page with output buffer larger than the generated response. `memset(scsi_Data + len, ..., scsi_Length - len)` begins at byte offset 2*len and writes past the declared output end by len bytes whenever padding reaches that end.
- **Consequence:** Memory corruption and uncleared bytes in the intended response buffer. Exact-size responses skip padding and avoid this trigger.

<a id="disk-017"></a>

### DISK-017 — ATA SCSI emulation ignores supplied data-buffer length

- **Severity:** High (memory overwrite/DMA overrun). **Confidence:** High.
- **Source:** `rom/devs/ata/scsiemu.c`, `scsi_inquiry`, `scsi_modesense`, `scsi_readcapacity`, read/write dispatch.
- **Trigger/trace:** INQUIRY with a short allocation/data buffer still writes fields through byte 35. READ CAPACITY always writes eight bytes; MODE SENSE writes full pages. Read/write CDB counts are passed to low-level transfers without comparing required bytes with scsi_Length. CDB lengths are also unchecked.
- **Consequence:** Short but valid inquiry allocations or malformed passthrough requests can corrupt adjacent memory. This source path bypasses the sector-command validation in ata.c.

<a id="disk-018"></a>

### DISK-018 — ATA emulated READ CAPACITY reports sector count as last LBA

- **Severity:** Medium. **Confidence:** High.
- **Source:** `rom/devs/ata/scsiemu.c`, `scsi_readcapacity`, `wl(out, blocks)`; ATA capacity initialization to be cross-checked.
- **Trigger/trace:** For N sectors, output reports N rather than N-1. The read/write command paths treat au_Capacity as exclusive sector count.
- **Consequence:** Clients infer one nonexistent extra sector. Large-device saturation/Capacity48 handling also needs review.

<a id="disk-019"></a>

### DISK-019 — SD reports every card as write-protected despite implemented writes

- **Severity:** Medium. **Confidence:** High.
- **Source:** `rom/devs/sdcard/sdcard_device.c`, `cmd_ProtStatus`, write handlers; `sdcard_ioops.c`, WriteSector32/64.
- **Trigger/trace:** Query TD_PROTSTATUS on an unlocked writable card. It unconditionally returns -1; the code checking actual flags is disabled, with an obsolete comment that writes are unsupported.
- **Consequence:** Clients honoring protection status refuse to write even though write commands work. Applies to the shared SD driver.

<a id="disk-020"></a>

### DISK-020 — MMC high-speed switch treats success as failure

- **Severity:** Medium. **Confidence:** High.
- **Source:** `rom/devs/sdcard/sdcard_mmcunit.c`, MMCChangeFrequency/MMCSwitch.
- **Trigger/trace:** MMCSwitch completes successfully and returns a value other than -1. MMCChangeFrequency tests `!= -1` and returns -1 with a failure diagnostic; actual failure falls through to the subsequent check.
- **Consequence:** High-speed setup fails after a successful switch, potentially leaving card and host configuration inconsistent. Caller's recovery response still needs tracing. MMCSwitch also discards WaitStatus failure.

<a id="disk-021"></a>

### DISK-021 — Hostdisk replies to a change-interrupt request while retaining its list node

- **Severity:** High (list corruption/use after release). **Confidence:** High.
- **Source:** `arch/all-hosted/devs/hostdisk/hostdisk_device.c`, addchangeint/unitentry TD_ADDCHANGEINT and unconditional ReplyMsg.
- **Trigger/trace:** Unit task adds the request node to changeints, then replies to the same message. ReplyMsg inserts that node in the caller's reply port, overwriting its links while changeints still references it.
- **Consequence:** Change notification traversal or later removal follows the wrong list or freed request. Unlike ATA's bus task, hostdisk does not suppress the reply for TD_ADDCHANGEINT.

<a id="disk-023"></a>

### DISK-023 — Hostdisk UPDATE overtakes queued writes and never flushes host storage

- **Severity:** High (durability). **Confidence:** High.
- **Source:** `arch/all-hosted/devs/hostdisk/hostdisk_device.c`, beginio CMD_UPDATE; UNIX/Windows Host_Write implementations.
- **Trigger/trace:** Enqueue a write, then request update. Writes go to the unit task while update immediately succeeds in the caller. No host fsync/FlushFileBuffers operation exists in the loaded host interface.
- **Consequence:** Update can complete before the write even runs and provides no host-storage durability guarantee afterward. Related to DISK-010, with an additional ordering failure.

<a id="disk-024"></a>

### DISK-024 — Hostdisk write loop never progresses on a zero-byte successful host write

- **Severity:** Medium (unit task hang). **Confidence:** High for control flow; host trigger conditional.
- **Source:** `arch/all-hosted/devs/hostdisk/hostdisk_device.c`, write.
- **Trigger/trace:** Host_Write returns zero for a nonempty request. Only -1 is handled; size and buffer remain unchanged and the loop repeats forever. The read loop explicitly handles zero.
- **Consequence:** All I/O on that unit stalls. Actual zero-write availability depends on backing device/host behavior; this is a conditional candidate.

<a id="disk-025"></a>

### DISK-025 — UNIX hostdisk truncates 64-bit seek results before checking failure

- **Severity:** Medium. **Confidence:** High.
- **Source:** `arch/all-unix/devs/hostdisk/hostdisk_host.c`, Host_Seek64; `hostdisk_host.h`, LSeek and lseek signature.
- **Trigger/trace:** Successfully seek to a positive 64-bit offset whose low 32 bits equal 0xffffffff. Store the returned off_t in int; it becomes -1, so a successful seek is reported as failure.
- **Consequence:** Reads/writes at those offsets fail spuriously. Odd byte offsets are permitted by hostdisk; ordinary sector-aligned requests avoid this exact value.

<a id="disk-026"></a>

### DISK-026 — UNIX Host_Open erases the allocated-name ownership flag

- **Severity:** Low (leak). **Confidence:** High.
- **Source:** `arch/all-unix/devs/hostdisk/hostdisk_host.c`, Host_Open `Unit->flags=0`; shared hostdisk Open/freeUnit.
- **Trigger/trace:** Open a numeric unit. Shared code allocates unitname and sets UNIT_FREENAME; Host_Open clears the entire flags field. freeUnit no longer frees that allocation. Reopening an already-open numeric unit independently also leaks its newly allocated name.
- **Consequence:** Repeated open/close or duplicate opens leak memory.

<a id="disk-027"></a>

### DISK-027 — Workbench USB writes use the original buffer/length for every chunk

- **Severity:** High (wrong data/transport failure). **Confidence:** High.
- **Source:** `workbench/devs/USB/classes/MassStorage/storageclass.c`, Hidd_USBStorage Write.
- **Trigger/trace:** A normal Trackdisk `CMD_WRITE`/`TD_WRITE64` or format request reaches `cmd_Write32`/`cmd_Write64`, which passes the sector count and caller buffer to `HIDD_USBStorage_Write` (`storage_device.c:244-280,300-337,472-505`). For a request of 1,025 512-byte sectors, `Write` builds a first WRITE(10) CDB for 1,024 sectors and a second for one sector (`storageclass.c:588-599`), but each call to DirectSCSI still advertises 1,025 sectors' bytes and uses `msg->buffer` rather than the advanced `buf` and chunk length. Thus each Bulk-Only Command Block Wrapper advertises more data than its CDB requests, and the second chunk also repeats the first chunk's source bytes while targeting the next LBA. The selected caller reports the full IO length after the HIDD method returns success (`storage_device.c:280-294,337-351`). DirectSCSI separately ignores the data-stage BulkTransfer result and CSW residue (`storageclass.c:450-476`), tracked as DISK-029; that can obscure the malformed write if a matching successful CSW is still received.
- **Consequence:** Multi-chunk writes are sent with inconsistent command/data lengths and repeated source data, so they can fail or write incorrect contents on the target; depending on the device's CSW response, the caller can report the full request as successful. Read uses the correct advanced buffer and per-chunk length.

<a id="disk-028"></a>

### DISK-028 — Workbench USB passthrough can overflow its 16-byte stack CDB

- **Severity:** High (stack overwrite). **Confidence:** High.
- **Source:** `workbench/devs/USB/classes/MassStorage/storageclass.c`, DirectSCSI copy loop; `storage_device.c`, cmd_DirectSCSI forwarding scsi_CmdLength; `storage.h`, cbw_t.
- **Trigger/trace:** HD_SCSICMD carries a declared CDB length from 17 through 255 bytes. `scsi_CmdLength` is a UWORD, then `storage_device.c` forwards it through the HIDD method’s 8-bit `cmdLen`; values in this range reach the unchecked loop into `cbw.CBWCB`, a 16-byte array. Larger UWORD values narrow modulo 256 and are not included in this overrun trigger.
- **Consequence:** Stack corruption in a disk I/O task. Inputs should be rejected or represented safely; silently truncating also needs a defined contract.

<a id="disk-029"></a>

### DISK-029 — Workbench USB transport ignores data-phase failure and CSW residue

- **Severity:** High (false successful or incomplete transfer). **Confidence:** High for the ignored status/residue and full-length result packaging; manifestation requires a short/failed data phase with a successful CSW.
- **Source:** workbench/devs/USB/classes/MassStorage/storageclass.c, DirectSCSI and data-command wrappers; storage_device.c, success accounting.
- **Trigger/trace:** DirectSCSI advertises msg->dataLen in the CBW, calls the data-stage HIDD_USBDevice_BulkTransfer() without checking its return, and after a successful CSW checks signature, tag and status but never checks dCSWDataResidue (storageclass.c:417-481). The USB-IF BOT specification requires the host to derive relevant data from requested length minus CSW residue for status 00h/01h and treats residue greater than the requested length as a non-meaningful CSW (USB Mass Storage Bulk-Only Transport 1.0, sections 6.3.2 and 6.7.2: https://www.usb.org/sites/default/files/usbmassbulk_10.pdf). Selected higher-level Read() and Write() wrappers treat a true DirectSCSI() result as full success (storageclass.c:530-611), and Trackdisk handlers set the full requested io_Actual (storage_device.c:131-185,244-299). The raw HD_SCSICMD handler likewise sets scsi_Actual and io_Actual to requested lengths (storage_device.c:357-465). Startup INQUIRY ignores its result and volume creation uses msu_inquiry[0] to select cdrom.handler versus fat.handler (storage_task.c:82-97; storage_volumes.c:472-474). READ CAPACITY decodes its eight-byte response whenever DirectSCSI() returns true (storageclass.c:614-646).
- **Consequence:** short/failed data can be treated as complete, leaving read buffers or response structures incomplete while callers receive success/full-length accounting; capacity or device classification can use incomplete response bytes. The existing multi-chunk write mismatch remains DISK-027 and may be obscured by this transport behavior.
- **Affected configuration:** the selected Workbench USB MassStorage class and a selected BOT host-controller implementation. The concrete effect requires the data phase to be short/failed while a matching successful CSW is accepted; whether an individual controller signals or recovers each short/stall combination remains hardware-path dependent. No device, transfer, or build was exercised; source-only.

<a id="disk-030"></a>

### DISK-030 — Workbench USB disconnect skips queued requests indefinitely

- **Severity:** High (hung callers). **Confidence:** High.
- **Source:** `workbench/devs/USB/classes/MassStorage/storage_task.c`, SIGF_SINGLE removal branch and request processing guarded by !MSF_DeviceRemoved; storageclass Dispose.
- **Trigger/trace:** Disconnect while requests remain queued. Removal sets MSF_DeviceRemoved; the task continues waiting but never drains/replies to the queue while removed. Its message signal may be consumed in that state, so even reconnection need not process older requests until another signal arrives.
- **Consequence:** WaitIO callers can hang indefinitely instead of receiving media/device errors.

<a id="disk-031"></a>

### DISK-031 — Workbench USB cache compares NULL optional device strings

- **Severity:** High (attach crash). **Confidence:** High for NULL cache entries.
- **Source:** `workbench/devs/USB/classes/MassStorage/storageclass.c`, GetUnitFromCache.
- **Trigger/trace:** Cache creation explicitly permits absent product/manufacturer/serial pointers and allocation failures, storing NULL. A later matching VID/PID invokes strcmp on those pointers without checks.
- **Consequence:** Attaching/reconnecting a device with missing strings, or after partial allocation failure, crashes. Device attribute implementation may normalize missing hardware strings; allocation-failure trigger remains regardless.

<a id="disk-032"></a>

### DISK-032 — Workbench USB geometry reports last LBA as total sector count

- **Severity:** Medium. **Confidence:** High.
- **Source:** `workbench/devs/USB/classes/MassStorage/storageclass.c`, ReadCapacity; `storage_device.c`, cmd_GetGeometry.
- **Trigger/trace:** ReadCapacity stores the first READ CAPACITY response field (last LBA) in msu_blockCount. Bounds correctly compare last requested LBA with it, but geometry returns it directly as dg_TotalSectors and derives cylinders from it without adding one.
- **Consequence:** Geometry omits the final sector and can understate usable cylinders at boundaries.

<a id="disk-033"></a>

### DISK-033 — Poseidon CBI reads a second status after already consuming it

- **Severity:** High (command timeout/desynchronization). **Confidence:** High.
- **Source:** `rom/usb/classes/massstorage/massstorage_cbi.c`, nScsiDirectCBI asynchronous data/status handling.
- **Trigger/trace:** A CBI command with data posts its interrupt status pipe, waits until statusdone, and consumes that message. Afterward the common success path calls psdDoPipe on the interrupt pipe again to obtain status for the same command, ignoring the already received statusdone flag.
- **Consequence:** Waits for an extra status that does not belong to the command; timeout or next-command desynchronization. Applies to MS_PROTO_CBI data commands, not CB or no-data commands.

<a id="disk-034"></a>

### DISK-034 — Poseidon autosense inspects bytes beyond the supplied sense buffer

- **Severity:** Medium (out-of-bounds read). **Confidence:** High.
- **Source:** `rom/usb/classes/massstorage/massstorage_cbi.c`, nScsiDirectCBI autosense; `massstorage_bulk.c`, autosense key/ASC/debug reads.
- **Trigger/trace:** Caller requests autosense with fewer than 14 bytes of sense storage. CBI safely fetches into an 18-byte temporary and copies only the declared length, but then reads caller scsi_SenseData[12], [2], [13] unconditionally. Bulk likewise reads fixed sense fields without checking scsi_SenseActual or allocated length.
- **Consequence:** Invalid reads and decisions based on unrelated memory during error recovery. Four-byte old autosense makes this particularly relevant.

<a id="disk-035"></a>

### DISK-035 — Poseidon UAS accepts empty/truncated status as successful completion

- **Severity:** High (false success). **Confidence:** High.
- **Source:** `rom/usb/classes/massstorage/massstorage_uas.c`, nUasDoCommand and nScsiDirectUAS.
- **Trigger/trace:** Status transfer returns RUNTPACKET with zero to two bytes. status starts as SCSI_GOOD; parsing does not change it unless at least three bytes arrived, yet reads statusbuf[0] even for zero bytes and returns success. It also does not validate returned tag or expected IU type before treating status as command completion.
- **Consequence:** Missing/mismatched/ready status may be accepted as completed successful I/O. UAS wire-layout and ready-handshake validation remain a separate pending review.

<a id="disk-036"></a>

### DISK-036 — Poseidon bulk NAK-timeout retry budget renews indefinitely

- **Severity:** Medium (unbounded I/O hang). **Confidence:** High.
- **Source:** `rom/usb/classes/massstorage/massstorage_bulk.c`, all UHIOERR_NAKTIMEOUT branches and trailing do/while(retrycnt--).
- **Trigger/trace:** Repeated NAK timeouts set retrycnt to 1 whenever zero. Loop decrement consumes it, then the next timeout renews it. A permanently busy connected device never exhausts retries, even when caller did not request autoretry.
- **Consequence:** Holds transfer lock and disk request indefinitely until denial/removal; prevents bounded timeout behavior.

<a id="disk-037"></a>

### DISK-037 — AROS disk-image seeks silently truncate 64-bit offsets

- **Severity:** High (wrong-sector writes). **Confidence:** High.
- **Source:** `workbench/devs/diskimage/support/dos64.c:45,79`; `plugins/generic.c:189`.
- **Trigger/trace:** On AROS the 64-bit seek implementation always returns ACTION_NOT_KNOWN; its fallback casts the requested QUAD position to LONG. Generic_Read/Write explicitly reconstruct the 64-bit TD64 offset and pass it to this helper. Offset 0x100000000 becomes zero and can succeed.
- **Consequence:** A high-offset write can overwrite the beginning of the backing image instead of failing. The support/mmakefile.src includes dos64 in diskimagesupport. Existing file-size limits do not prevent an out-of-range request from reaching this unchecked path.

<a id="disk-038"></a>

### DISK-038 — CISO index allocation arithmetic overflows

- **Severity:** High (heap corruption on image open). **Confidence:** High.
- **Source:** `workbench/devs/diskimage/plugins/ciso.c:155,183,201`.
- **Trigger/trace:** A header declaring 0x40000000 blocks produces index_size=4 in ULONG arithmetic. Four index bytes satisfy the read, after which the endian conversion iterates through total_blocks+1 entries in a four-byte allocation. The doubled block-buffer allocation also has unchecked overflow.
- **Consequence:** Crafted image metadata causes out-of-bounds writes; counts must be checked before narrowing/multiplication and against the actual file.

<a id="disk-039"></a>

### DISK-039 — CISO compressed extent can exceed its staging buffer

- **Severity:** High (heap corruption). **Confidence:** High.
- **Source:** `workbench/devs/diskimage/plugins/ciso.c:184,296,300`.
- **Trigger/trace:** block_buf holds twice block_size, but the index difference shifted by unchecked align is passed directly as Read length. An extent exceeding twice block_size overwrites the allocation. Non-monotonic indexes and oversized shifts are also unchecked.
- **Consequence:** Malformed image indexes corrupt memory during disk reads.

<a id="disk-040"></a>

### DISK-040 — DAX frame lengths exceed the allocated input buffer

- **Severity:** High (heap corruption). **Confidence:** High.
- **Source:** `workbench/devs/diskimage/plugins/dax.c:259,348`.
- **Trigger/trace:** The image supplies a 16-bit frame size up to 65535; the staging allocation is 8192+1024 bytes. Read uses frame->size without checking that capacity.
- **Consequence:** A malformed compressed frame overwrites heap memory before decompression.

<a id="disk-041"></a>

### DISK-041 — DAX checks only the first uncompressed area

- **Severity:** Medium (valid image read failure). **Confidence:** High.
- **Source:** `workbench/devs/diskimage/plugins/dax.c:141`.
- **Trigger/trace:** IsNCArea iterates count times but never advances ncareas or indexes by i. A frame belonging to a second or later uncompressed area is marked compressed.
- **Consequence:** Raw data is sent to Uncompress, causing errors or incorrect interpretation of valid DAX images.

<a id="disk-042"></a>

### DISK-042 — Disk-image CD audio reads bypass SCSI buffer bounds

- **Severity:** High (caller-buffer corruption). **Confidence:** High.
- **Source:** `workbench/devs/diskimage/device/scsicmd.c:274,310`; `plugins/cue/cue.c:806`.
- **Trigger/trace:** READ_CD and READ_CD_MSF derive frame counts from the CDB and pass the caller's data pointer directly to Plugin_ReadCDDA without limiting frames to scsi_Length/2352. CUE_ReadCDDA reads frames*2352 bytes into that pointer; the BytePack bounds initialized earlier are bypassed.
- **Consequence:** A one-frame request with a shorter output buffer overwrites caller memory. CUE is optional in the default build; other CDDA implementations still need tracing.

<a id="disk-043"></a>

### DISK-043 — Generic image writes do not enforce advertised medium size

- **Severity:** Medium (out-of-range writes accepted). **Confidence:** High.
- **Source:** `workbench/devs/diskimage/plugins/generic.c:189`.
- **Trigger/trace:** Generic_Write seeks and writes without checking offset/length against image->total_blocks, unlike bounded plugins. A request beyond the last sector extends the backing file when the host filesystem allows it.
- **Consequence:** Reports successful writes outside the mounted medium's advertised geometry. The same loop also treats a zero-byte `Write()` as progress: `buffer` and `size` remain unchanged, so the DiskImage unit worker never reaches request completion (`generic.c:201-210`; worker dispatch/reply in `device/unit.c:312-424`). This has a concrete selected backend trigger: back a writable Generic image with a file on `RAM:`, then write beyond its current allocated end while RAMFS cannot allocate another data block. `WriteData()` then gets NULL from `AddDataBlock()`, records `ERROR_DISK_FULL`, and returns the bytes written so far, which is zero if failure occurs before any byte is copied (`rom/filesys/ram/filesystem.c:876-944,1280-1322`). `CmdWrite()` sends that count as the `ACTION_WRITE` result (`rom/filesys/ram/commands.c:637-665`; `rom/filesys/ram/handler.c:181-184`), and DOS `Write()` returns it unchanged (`rom/dos/write.c:24-45,57-75`). A partial write followed by allocation failure also leaves bytes remaining and reaches the same stuck loop. This is the same progress-check omission tracked by DISK-024, not a separate finding.

<a id="disk-044"></a>

### DISK-044 — PC floppy result collection never advances its destination

- **Severity:** High (false successful I/O; possible hang). **Confidence:** High.
- **Source:** `rom/devs/trackdisk/trackdisk_hw.c:156`.
- **Trigger/trace:** td_waitint initializes i=0 but never increments it while collecting the seven result bytes. Each byte replaces td_result[0]; td_result[1..6] remain stale. Read/write/format interpret slot zero as ST0 although it now contains the last returned byte, commonly sector-size code 2.
- **Consequence:** Hardware errors can appear successful and ST1/ST2 error decoding uses unrelated bytes. If command-busy remains set and byte retrieval fails, the loop has no effective bound and ignores the retrieval error.

<a id="disk-045"></a>

### DISK-045 — PC floppy cache survives medium replacement

- **Severity:** High (stale reads and wrong-medium writes). **Confidence:** High.
- **Source:** `rom/devs/trackdisk/trackdisk_device.c:392,766`; `trackdisk_hw.c`, td_readtrack/td_update.
- **Trigger/trace:** Removal and insertion update presence/count/protection but never invalidate tdu_lastcyl/lasthd or clear TDUF_WRITE. A same-track read after insertion skips hardware and returns the previous medium's buffer. If dirty, a later update writes that old buffer once the replacement's change latch is cleared by stepping/recalibration.
- **Consequence:** Silent stale data or corruption of a replacement floppy. Applications issuing CMD_CLEAR on every change mitigate but do not repair the driver's missing invalidation.

<a id="disk-046"></a>

### DISK-046 — PC floppy DMA replacement does not ensure boundary safety

- **Severity:** High (DMA memory corruption). **Confidence:** High.
- **Source:** `rom/devs/trackdisk/trackdisk_device.c:86`; `trackdisk_hw.h`, set_dma_addr/count.
- **Trigger/trace:** A track allocation crossing a 64 KiB DMA page is replaced once with another ordinary allocation, which is never checked. The DMA address counter is 16 bits; page selection is programmed separately.
- **Consequence:** If the replacement also crosses a boundary, track DMA wraps within the page and accesses memory outside the allocation.

<a id="disk-047"></a>

### DISK-047 — Floppy addressing truncates unchecked offsets

- **Severity:** High (wrong-track writes). **Confidence:** High.
- **Source:** `rom/devs/trackdisk/trackdisk_hw.c:513,560,707`; `arch/m68k-amiga/devs/trackdisk/trackdisk_hw.c`, td_read/td_write2/td_format2.
- **Trigger/trace:** Neither dispatch nor hardware entry validates the complete request against disk capacity/alignment. PC code discards offset low nine bits and narrows cylinders to UBYTE. Amiga code narrows calculated track to UBYTE. A PC offset of 256 cylinders (256*36*512) aliases cylinder zero; an Amiga offset of 256 ADOS tracks aliases track zero.
- **Consequence:** Out-of-range writes can be accepted into cached data for the beginning of the disk and later flushed there. Misaligned offsets also access a different byte position than requested.

<a id="disk-048"></a>

### DISK-048 — Amiga floppy sub-sector requests make no progress

- **Severity:** Medium (device task hangs). **Confidence:** High.
- **Source:** `arch/m68k-amiga/devs/trackdisk/trackdisk_hw.c:844,964`.
- **Trigger/trace:** Read/write loop computes sectors from len/512 without rejecting a nonmultiple length. With len=1, sectorsdone/totalsectorsneeded remains zero, len remains one, and no retry budget is consumed in a successful read/cache path.
- **Consequence:** A malformed request blocks all requests served by the device task indefinitely.

<a id="disk-049"></a>

### DISK-049 — Amiga shared buffer switch discards another unit's dirty writes

- **Severity:** High (data loss). **Confidence:** High.
- **Source:** `arch/m68k-amiga/devs/trackdisk/trackdisk_hw.c:808,1021`.
- **Trigger/trace:** With dirty data belonging to unit A, an uncached request for unit B calls maybe_flush(B). td_flush(B) returns zero because td_buffer_unit is A; maybe_flush then calls td_clear and drops A's dirty flag and sector data.
- **Consequence:** A successful buffered write to one floppy is lost when another floppy is accessed before the periodic flush. Different-unit format detection can also replace the shared data without flushing its owner.

<a id="disk-050"></a>

### DISK-050 — Amiga HD buffer allocation failure leaves dangling pointers

- **Severity:** High (use-after-free/double-free under memory pressure). **Confidence:** High.
- **Source:** `arch/m68k-amiga/devs/trackdisk/trackdisk_hw.c:225`.
- **Trigger/trace:** checkbuffer frees the DD buffers, allocates HD buffers and frees both new allocations if either fails, but does not reset the pointers. A later call sees td_supportHD still false and frees these dangling pointers using DD sizes, or a DD request uses the freed successful allocation.
- **Consequence:** Memory-manager corruption or access to freed buffers after a recoverable allocation failure.

<a id="disk-051"></a>

### DISK-051 — Amiga track-cache partial allocation failure causes null copy

- **Severity:** High (crash under memory pressure). **Confidence:** High.
- **Source:** `arch/m68k-amiga/devs/trackdisk/trackdisk_hw.c:732,778`.
- **Trigger/trace:** tc_ensure allocates each slot independently but accepts the cache when only slot zero exists. tc_store checks only slot zero and picks the first empty slot by track marker, even if that slot's allocation failed.
- **Consequence:** After filling earlier slots, CopyMem writes a track to NULL rather than disabling/skipping failed cache slots.

<a id="disk-052"></a>

### DISK-052 — Amiga raw-track parsers dereference before checking the end

- **Severity:** Medium (out-of-bounds read). **Confidence:** High.
- **Source:** `arch/m68k-amiga/devs/trackdisk/trackdisk_hw.c:366,375,505,514`.
- **Trigger/trace:** Both decoders test *raw before raw>=rawend in the sync scan and before raw<rawend in the repeated-sync loop. A track without a remaining sync word reaches the one-past-end address and reads it before exiting.
- **Consequence:** Malformed/unreadable floppy tracks cause reads beyond the DMA allocation. The PCDOS payload bound also reserves only 512 words while consuming 512 data words plus two CRC words.

<a id="disk-053"></a>

### DISK-053 — Amiga read cache is not invalidated on insertion or CMD_CLEAR

- **Severity:** High (stale reads). **Confidence:** High.
- **Source:** `arch/m68k-amiga/devs/trackdisk/trackdisk_device.c:92,124,437`; `trackdisk_hw.c`, tc_find/maybe_detect/td_detectformat/td_clear.
- **Trigger/trace:** Removal calls td_clear, which clears only the single active buffer. Insertion calls td_detectformat directly and sets an identified format; it never invalidates the multitrack cache. maybe_detect's cache invalidation runs only for DT_UNDETECTED, which insertion already replaces. A same-format replacement therefore matches cached unit/track/sector masks. CMD_CLEAR likewise retains these slots.
- **Consequence:** Reads of a replacement disk or reads after an explicit clear can return the previous disk's decoded tracks.

<a id="disk-054"></a>

### DISK-054 — RAM disk bounds check wraps before validation

- **Severity:** High (out-of-bounds memory access). **Confidence:** High.
- **Source:** `workbench/devs/ramdrive_device.c:443,477`.
- **Trigger/trace:** offset and size are ULONG; validation uses offset+size > DISKSIZE. offset=0xffffff00,size=512 wraps the sum to 256 and passes, then CopyMem accesses unit->mem+offset for 512 bytes.
- **Consequence:** Malformed requests read/write outside the RAM disk allocation instead of reporting a seek/bounds error. Both read and write are affected.

<a id="disk-055"></a>

### DISK-055 — GPT backup table placement mixes bytes and sectors

- **Severity:** High (wrong-location metadata writes). **Confidence:** High.
- **Source:** `rom/partition/partitiongpt.c:574`.
- **Trigger/trace:** tablesize is a byte count, but GPTWriteTable receives backup-tablesize as the table's starting block. For 128 entries of 128 bytes on 512-byte sectors, it subtracts 16384 sectors instead of 32.
- **Consequence:** Rewriting a GPT stores its backup entry array far before its reserved location, potentially overwriting user partition data. Header StartBlock is changed to the same erroneous location. On small media the subtraction can underflow before the request.

<a id="disk-056"></a>

### DISK-056 — GPT header CRC reads an unchecked header size

- **Severity:** High (out-of-bounds read during disk discovery). **Confidence:** High.
- **Source:** `rom/partition/partitiongpt.c:264,274`.
- **Trigger/trace:** A header is read into one sector, but HeaderSize is checked only for a minimum of 92. Crc32_ComputeBuf then reads HeaderSize bytes, even when that value exceeds the allocation.
- **Consequence:** A malformed disk header reads arbitrary adjacent memory or faults before CRC rejection. GPT_MAX_HEADER_SIZE exists but is not enforced.

<a id="disk-057"></a>

### DISK-057 — GPT entry size/count are used without structural or overflow validation

- **Severity:** High (out-of-bounds access). **Confidence:** High.
- **Source:** `rom/partition/partitiongpt.c:344` and GPTReadPartitionTable entry loop.
- **Trigger/trace:** entrysize*cnt and its sector roundup use ULONG without overflow checks. Neither minimum entry size nor enough bytes for GPTPartition fields is enforced. A CRC-valid table with entrysize=1,cnt=1 supplies a one-byte logical entry, while code reads UUID/start/end/name fields and advances p into undersized entries. Large counts can wrap allocation arithmetic yet retain a huge iteration bound.
- **Consequence:** Crafted partition metadata causes out-of-bounds reads and invalid partition handles; a header CRC is not a structural validation substitute.

<a id="disk-058"></a>

### DISK-058 — GPT stops advancing at unused partition entries

- **Severity:** Medium (partitions disappear; metadata loss on rewrite). **Confidence:** High.
- **Source:** `rom/partition/partitiongpt.c:388`.
- **Trigger/trace:** When TypeID is zero the loop continues before advancing p by entrysize. All remaining iterations examine the same unused entry. Used entries following a gap are never loaded.
- **Consequence:** Valid GPT partitions after a deleted/unused slot disappear. Writing the resulting in-memory list rebuilds the array without those omitted entries, deleting their metadata.

<a id="disk-059"></a>

### DISK-059 — GPT 64-bit partition addresses narrow to 32-bit geometry

- **Severity:** High (wrong-partition addressing on large disks). **Confidence:** High.
- **Source:** `rom/partition/partitiongpt.c:399`; `partition_support.c`, initPartitionHandle/getStartBlock; `readpartitiondata.c`/`writepartitiondata.c`.
- **Trigger/trace:** GPT start/end are UQUAD, but initPartitionHandle accepts ULONG start and count. A partition beginning at 0x100000000 sectors is initialized as starting at zero. Its original GPT entry still exposes the full PT_STARTBLOCK, while raw I/O constructs offset from the truncated DosEnvec and ULONG getStartBlock.
- **Consequence:** Reads/writes through the partition handle address another disk region, potentially including the disk beginning. Single-block helper arguments are also ULONG and truncate backup-header addresses above 2^32 sectors.
- **Integration extension:** Boot `rom/dosboot/bootscan.c:41–57`, ROM USB `partitions.c:118–134`, and Workbench USB `storage_volumes.c:117–133` also accumulate ancestor start sectors into ULONG GetOffset results. Their mount callers convert that narrowed offset into cylinder additions. On pointer-wide geometry builds an ancestor offset above2^32 sectors is lost even if intermediate DosEnvec fields can represent it. This is grouped with the existing partition-address narrowing root cause, not counted as three more independent bugs.

<a id="disk-060"></a>

### DISK-060 — NSD capability selection can wrap capacity arithmetic on 32-bit builds

- **Severity:** High (high-offset I/O aliases low offsets). **Confidence:** High for the arithmetic; conditional on driver behavior.
- **Source:** `rom/partition/partition_support.c`, PartitionNsdCheck; `openrootpartition.c`.
- **Trigger/trace:** The capacity threshold multiplies IPTR geometry fields without widening. On 32-bit builds large capacity products can wrap below the threshold, preventing NSCMD_DEVICEQUERY. cmdread/cmdwrite remain CMD_READ/CMD_WRITE, although raw I/O places high offset bits in io_Actual.
- **Consequence:** A driver that consumes high offset bits only for 64-bit commands receives an ordinary 32-bit transfer and may access the wrong disk region. The test also uses HighCyl rather than cylinder count, excluding one full cylinder from the decision.
- **Installer manifestation:** C:Install nsdCheck, Install-grub.c nsdCheck and Install-grub2.c nsdCheck also evaluate ULONG start/count arithmetic before the threshold test. Their overflow-induced skipped negotiation is grouped here. Required negotiation actually attempted but unsuccessful is separately DISK-263; widening the threshold alone does not fix that fallback.

<a id="disk-061"></a>

### DISK-061 — Destroying GPT writes its header over sector zero

- **Severity:** High (boot-sector corruption). **Confidence:** High.
- **Source:** `rom/partition/partitiongpt.c`, PartitionGPTDestroyPartitionTable; `partitionmbr.c`, PartitionMBRDestroyPartitionTable.
- **Trigger/trace:** GPT destroy delegates to MBR destroy, which assumes root->table->data contains the MBR. For GPT it contains the GPT header read from sector one. The routine clears MBR entry/signature offsets in that header buffer and writes the entire buffer to sector zero.
- **Consequence:** Destroying GPT overwrites MBR boot code with a GPT header instead of preserving boot code while invalidating partition metadata. Primary/backup GPT signatures remain on disk.

<a id="disk-062"></a>

### DISK-062 — Cyclic EBR chains have no traversal bound or cycle detection

- **Severity:** High (memory exhaustion during discovery). **Confidence:** High.
- **Source:** `rom/partition/partitionebr.c:115`.
- **Trigger/trace:** An EBR with a nonzero next link pointing to itself, or a pair forming a cycle, is repeatedly read and creates another logical-partition handle on every visit. The UBYTE i increments but does not govern termination; neither visited blocks nor container bounds are checked.
- **Consequence:** Disk discovery loops until allocation failure (or indefinitely for zero-count entries), consuming memory. On error the partial handle list is not freed before OpenPartitionTable frees its containing table object, leaking the allocations.

<a id="disk-063"></a>

### DISK-063 — EBR addition dereferences an allocation failure

- **Severity:** High (crash under memory pressure). **Confidence:** High.
- **Source:** `rom/partition/partitionebr.c:337,339`.
- **Trigger/trace:** PartitionEBRNewHandle returns NULL if either handle/data allocation fails; caller unconditionally passes &new_ph->ln to Insert.
- **Consequence:** AddPartition crashes instead of returning failure; the public wrapper performs no additional validation.

<a id="disk-064"></a>

### DISK-064 — MBR partition position is used as an unchecked array index

- **Severity:** High (memory corruption). **Confidence:** High.
- **Source:** `rom/partition/partitionmbr.c:274,288` and PartitionMBRSetPartitionAttrs PT_POSITION.
- **Trigger/trace:** AddPartition accepts any explicit position except -1 and indexes pcpt[4] with it; SetPartitionAttrs narrows the value to UBYTE then likewise indexes without a 0..3 check. For example position 4 addresses past the four-entry array and writes its status/type/start/count fields.
- **Consequence:** Caller-supplied invalid positions corrupt the sector buffer and potentially heap memory, and may persist this corruption when the table is written.

<a id="disk-065"></a>

### DISK-065 — Adding an MBR partition to an occupied explicit slot aliases handles

- **Severity:** High (partition metadata loss). **Confidence:** High.
- **Source:** `rom/partition/partitionmbr.c`, PartitionMBRAddPartition.
- **Trigger/trace:** An explicit PT_POSITION bypasses the free-slot search and overwrites an existing entry without rejecting/removing the existing handle. A second handle referencing the same entry is then appended.
- **Consequence:** Two handles describe one mutated slot with different cached geometries. Deleting either clears the other's entry; subsequent table writes can silently remove the remaining partition.

<a id="disk-066"></a>

### DISK-066 — RDB checksum length is trusted before structural validation

- **Severity:** High (out-of-bounds read). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c:165,213,254,279,325,372`.
- **Trigger/trace:** Every RDB/PART/BADB/FSHD/LSEG validator passes the on-disk SummedLongs directly to calcChkSum. A value larger than the available block makes the checksum walker read beyond the sector buffer. No maximum/minimum length check precedes it.
- **Consequence:** Malformed disk metadata can fault discovery; checksum rejection occurs after the invalid access.

<a id="disk-067"></a>

### DISK-067 — RDB partition name lengths overrun fixed buffers

- **Severity:** High (heap corruption). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c:295,296,834,835`.
- **Trigger/trace:** A PART block supplies an unchecked UBYTE name length copied into a 32-byte allocation, followed by a terminator at that index. The setter also uses strlen without enforcing capacity. Even a documented 31-character name writes data->pb_DriveName[32]=0 beyond the on-disk 32-byte BSTR field.
- **Consequence:** Crafted names corrupt heap memory; a maximum valid API name also overwrites the following field.

<a id="disk-068"></a>

### DISK-068 — RDB environment-table length overruns the partition handle

- **Severity:** High (heap corruption). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c:297 and PartitionRDBSetPartitionAttrs PT_DOSENVEC`.
- **Trigger/trace:** The on-disk environment table count controls CopyBE2HostDosEnvec into the fixed DosEnvec without checking its capacity or the 20-long input array. The setter similarly copies de_TableSize+1 elements into both fixed structures.
- **Consequence:** Malformed disk or caller metadata overwrites the partition handle or partition block; checksums do not constrain field lengths.

<a id="disk-069"></a>

### DISK-069 — RDB linked-block traversal has no cycle detection

- **Severity:** High (hang or memory exhaustion). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c, PartitionRDBCalcFSSize and PartitionRDBOpenPartitionTable`.
- **Trigger/trace:** BADB/PART/FSHD/LSEG next pointers are followed until -1, with no visited-block set or traversal bound. A CRC-valid self-referential LSEG loops forever in the size-counting pass; other chains repeatedly allocate duplicate nodes until memory is exhausted.
- **Consequence:** A malformed medium can hang discovery/boot or exhaust memory. This shares the general missing traversal guard in DISK-062, but affects separate RDB readers.

<a id="disk-070"></a>

### DISK-070 — RDB filesystem reads use sector size with 512-byte allocations

- **Severity:** High (heap corruption on non-512-byte media). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c:395 and PartitionRDBReadFileSys`.
- **Trigger/trace:** The filesystem array allocates size*512 and indexes fixed 512-byte LoadSegBlock objects, while readBlock reads root->de.de_SizeBlock*4 bytes. On a 4096-byte-sector device each read overruns its slot and the last read overruns the allocation. WriteFileSys likewise reads a whole sector from a 512-byte object.
- **Consequence:** Non-512-byte block devices expose out-of-bounds transfer buffers for RDB filesystem data.

<a id="disk-071"></a>

### DISK-071 — RDB writer returns success after failed metadata transfers

- **Severity:** High (silent partition/filesystem corruption). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c:628,651,681,692`.
- **Trigger/trace:** BADB, PART, FSHD and final RDSK writes discard writeBlock results. The LSEG helper returns the next block number on failure, and the caller treats that as normal progress and publishes a header for incomplete filesystem data. The public writer always returns zero.
- **Consequence:** Write-protected/erroring media produce false successful edits; partial metadata can be linked and persisted.

<a id="disk-072"></a>

### DISK-072 — RDB bad-block-list head is written in host byte order

- **Severity:** Medium (invalid bad-block chain). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c:618`.
- **Trigger/trace:** For a nonempty list rdb_BadBlockList is assigned block directly. Adjacent list-head and next fields use AROS_LONG2BE. On little-endian machines a head at block 2 is recorded as 0x02000000 in the RDB format.
- **Consequence:** Reopening the table follows an incorrect bad-block address and loses the bad-block list.

<a id="disk-073"></a>

### DISK-073 — New RDB partition geometry is never initialized

- **Severity:** Medium (incorrect public geometry). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c:858,898`.
- **Trigger/trace:** AddPartition requires PT_DOSENVEC to enter, then initializes ph->dg only if FindTagItem(PT_DOSENVEC)==NULL. That branch is unreachable; the MEMF_CLEAR geometry remains zero. The setter updates DosEnvec without updating geometry.
- **Consequence:** PT_GEOMETRY on a newly created partition reports zero sector size/counts instead of the supplied geometry.

<a id="disk-074"></a>

### DISK-074 — RDB filesystem stream seek addresses only one LSEG payload

- **Severity:** Medium (valid ELF filesystem fails to load). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c, SeekFunc; rom/dos/internalloadseg_elf.c:84`.
- **Trigger/trace:** SeekFunc compares absolute pos to fakefile.size=492 and changes only offset, never count. ELF loader seeks absolute file offsets; offsets beyond 491 are rejected, and seeks below 492 after advancing blocks still use the old block index.
- **Consequence:** Normal multiblock ELF filesystem binaries cannot be loaded reliably from RDB storage.

<a id="disk-075"></a>

### DISK-075 — RDB filesystem stream reads past end on repeated reads

- **Severity:** High (out-of-bounds read). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c, ReadFunc`.
- **Trigger/trace:** At the last payload boundary ReadFunc increments count to fsblocks and breaks. A subsequent read starts with that one-past-end count and immediately accesses filesystem[count], because there is no entry EOF check.
- **Consequence:** A loader request after EOF can consume unrelated memory rather than returning zero.

<a id="disk-076"></a>

### DISK-076 — RDB load path accepts missing or partially read filesystem data

- **Severity:** High (null access or invalid executable input). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c, PartitionRDBReadFileSys/LoadFS`.
- **Trigger/trace:** ReadFileSys returns void and leaves a queued FileSysNode even when sizing/allocation fails, or when the second read pass fails partway. LoadFS on non-m68k unconditionally examines node->filesystem[0], and all builds expose the unvalidated array to ReadFunc.
- **Consequence:** A malformed/read-error medium or memory pressure can crash filesystem loading or feed uninitialized payloads into the loader.

<a id="disk-077"></a>

### DISK-077 — RDB filesystem ID lookup omits big-endian conversion

- **Severity:** Medium (filesystem not found). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c:979`.
- **Trigger/trace:** FindFileSystem compares fhb_DosType directly to the host-order FST_ID tag. The getter converts with AROS_BE2LONG, but lookup does not.
- **Consequence:** On little-endian builds querying by the normal DOSType fails despite a matching filesystem header.

<a id="disk-078"></a>

### DISK-078 — Filesystem registration leaks its entry when loading fails

- **Severity:** Low (memory leak). **Confidence:** High.
- **Source:** `rom/partition/fsloader.c`, AddFS final error branch.
- **Trigger/trace:** AddFS allocates fsrnode, invokes LoadFileSystem, and on a null segment list returns IoErr without FreeVec(fsrnode).
- **Consequence:** Each unsuccessful filesystem load leaks a FileSysEntry allocation.

<a id="disk-079"></a>

### DISK-079 — RDB metadata writer has no reserved-area capacity check

- **Severity:** High (user-data overwrite). **Confidence:** High.
- **Source:** `rom/partition/partitionrdb.c, PartitionRDBWritePartitionTable`.
- **Trigger/trace:** Writer lays out BADB/PART/FSHD/LSEG blocks consecutively after rdbblock without checking rdb_RDBBlocksHi, first usable cylinder, or medium capacity. AddPartition also imposes no metadata-area capacity limit.
- **Consequence:** Adding enough entries/filesystem blocks can make a table rewrite overwrite filesystem data beyond the RDB reserved region.

<a id="disk-080"></a>

### DISK-080 — Amiga floppy arbitration reuses a queued wakeup message

- **Severity:** High (resource wait or reply-port list corruption/hang under contention). **Confidence:** High for the source-level queue and signal paths; manifestation requires contention.
- **Source:** `arch/m68k-amiga/devs/trackdisk/trackdisk_device.c:52-57,104-114,274-285,345-349`; `rom/disk/getunit.c:17-31`; `rom/disk/giveunit.c:19-27`; `rom/exec/replymsg.c:52-64`; `rom/exec/putmsg.c:81-105`; `rom/exec/waitport.c:67-95`.
- **Trigger/trace:** When `GetUnit()` finds the disk resource busy, it links the embedded `dru_Message` into `dr_Waiting`. `GiveUnit()` removes and replies to that message, placing it on `td_druport`; the trackdisk loop calls `WaitPort()` and retries without `GetMsg()`, so the message remains linked on the reply port. A later contended `GetUnit()` can link that same node into `dr_Waiting` while it is still on `td_druport`, corrupting list links or leaving the waiter spinning on the stale reply. A second caller path makes the wrong signal owner observable: `BeginIO()` synchronously calls `TD_PerformIO()` for `IOF_QUICK` commands, which calls `getunit()`, but `td_druport` signals `td_task`; a contending quick-I/O caller waiting in another task may therefore remain blocked after the resource is released.
- **Consequence:** Under disk-resource contention, a quick `BeginIO()` caller can wait indefinitely; repeated reuse can also corrupt the resource waiting list or reply-port queue.
- **Affected configuration:** m68k Amiga images that select `trackdisk` in `KDEVS` (`arch/m68k-amiga/boot/mmakefile.src:156,373`).
- **Mitigation direction:** Dequeue the reply before retrying and ensure the wait is signaled to the task actually waiting. No contention, request, or build was run; source-only.

<a id="disk-081"></a>

### DISK-081 — disk.resource unit allocation is not atomic

- **Severity:** Medium (double allocation/lost flags). **Confidence:** High.
- **Source:** `rom/disk/allocunit.c`/`freeunit.c`.
- **Trigger/trace:** AllocUnit checks and updates dr_Flags without Disable/Forbid or a lock, unlike GetUnit's protected flag updates. A task switch between check and set allows two callers to succeed for the same unit; read-modify-write updates can also lose another flag change.
- **Consequence:** Resource ownership is inconsistent even on a preemptive single-CPU Amiga build. Floppy startup calls AllocUnit for persistent ownership.

<a id="disk-082"></a>

### DISK-082 — NVMe controller identify timeout leaves a live stack completion handler

- **Severity:** High (late DMA and use-after-return). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_controllerclass.c`, identify submission/one-second wait/error exit; `nvme_queue_admin.c`.
- **Trigger/trace:** Identify uses stack cehandle and a temporary PCI buffer. When the timer expires after a successful submit, no command cancellation, handler detachment, or controller quiescence occurs. The buffer is freed and constructor returns, while the admin queue retains &cehandle. A delayed completion writes ceh_Result/status, releases DMA state and signals through this expired pointer.
- **Consequence:** A slow or stalled controller can DMA into freed memory and later corrupt reused constructor-stack memory. Admin command submission only unregisters handlers on immediate submission failure, not timeout.

<a id="disk-083"></a>

### DISK-083 — NVMe queue teardown frees memory while its interrupt handler remains registered

- **Severity:** High (interrupt use-after-free). **Confidence:** High; disposal reachability is conditional.
- **Source:** `rom/devs/nvme/nvme_controllerclass.c`, temporary-identify-buffer allocation failure and Root__Dispose; `nvme_queue.c`, nvme_free_queue.
- **Trigger/trace:** After AddInterrupt succeeds, failure to allocate the identify buffer frees admin queue memory but never removes the interrupt handler. Controller disposal likewise frees all queues without removing registered handlers or waiting for controller DMA shutdown. The IRQ handler is embedded in the freed queue and its data pointer points there.
- **Consequence:** Later shared-line interrupt dispatch can follow freed handler memory; active controller DMA may also access freed queue buffers. Hot removal is not advertised as supported; the constructor allocation-failure path does not require hot removal.

<a id="disk-084"></a>

### DISK-084 — NVMe identity fields are treated as unbounded C strings

- **Severity:** Medium (out-of-bounds read/incorrect identity). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_busclass.c:140,144,148; nvme_controllerclass.c identity tags`.
- **Trigger/trace:** The Identify structure declares fixed-width sn[20], mn[40], fr[8] arrays without terminators. Bus construction calls strlen on each and copies until an unrelated zero byte is found. A completely nonzero returned Identify page can make strlen leave the supplied allocation. Early zero bytes conversely create short allocations later read at fixed width by nvme_strcpy, sharing DISK-008.
- **Consequence:** Identity strings can include adjacent fields or cause invalid reads. Allocation failures in these copies are also unchecked.

<a id="disk-085"></a>

### DISK-085 — NVMe capacity scales native LBA counts twice

- **Severity:** High (incorrect geometry/bounds). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_busclass.c:550,553,573; nvme_io.c:348`.
- **Trigger/trace:** For 4 KiB LBAs the code multiplies NSZE by eight to set au_SecCnt and au_High, while sector size remains 4096 and nvme_sector_rw compares native 4 KiB LBAs against au_High.
- **Consequence:** Reported capacity is eight times the namespace size and the driver accepts out-of-range requests before controller rejection. This does not establish that a conforming controller writes beyond its namespace.

<a id="disk-086"></a>

### DISK-086 — NVMe completion readiness is published before completion work finishes

- **Severity:** High (premature replies/races on SMP). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_queue_io.c:36; nvme_busclass.c:93`.
- **Trigger/trace:** Completion sets ceh_Reply first, before releasing DMA mappings, freeing descriptors, and setting io_Error/io_Actual. The reply task scans all slots on every wakeup without a lock. On another CPU a wakeup from a different completion can observe this flag, reply the request and release its command ID while the hook still accesses the slot and request.
- **Consequence:** Caller reuse and slot reuse race with unfinished completion, risking invalid memory accesses, stale status and duplicate signalling. Separate from DISK-011 CQ walker state corruption.

<a id="disk-087"></a>

### DISK-087 — NVMe failed queue setup frees a queue still used by its task

- **Severity:** High (task use-after-free). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_busclass.c:370,454,461; nvme_queue.c`.
- **Trigger/trace:** An infinite nvme_iotask is created before CQ/SQ registration. CQ registration failure or vector-attribute failure frees the queue without stopping the task. A task not yet scheduled starts later and dereferences the freed queue; an already waiting task retains it for future wakes.
- **Consequence:** Recoverable setup failure leaves a live task with freed state. Successful controller disposal also does not stop queue tasks.

<a id="disk-088"></a>

### DISK-088 — NVMe startup does not handle essential allocation/task failures

- **Severity:** High (crash or permanently outstanding I/O under memory pressure). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_busclass.c:140,153,370; nvme_controllerclass.c bus-name allocation`.
- **Trigger/trace:** Identity CopyMem and bus-name sprintf use unchecked AllocVec results; the namespace-pointer array can be NULL yet startup indexes it. NewCreateTask failure is unchecked, so completions signal a NULL task and no reply worker exists.
- **Consequence:** Low memory can crash discovery or create an apparently present disk whose I/O never completes.

<a id="disk-089"></a>

### DISK-089 — NVMe and ATA boot-node open flags are uninitialized

- **Severity:** Medium (nondeterministic device-open behavior). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_busclass.c:624`.
- **Trigger/trace:** The automatic pp array is not initialized. Entry pp[3], the device open flags consumed by MakeDosNode, is commented out rather than assigned. Other entries are filled explicitly.
- **Consequence:** Boot-node startup flags depend on stack contents; device-open behavior or downstream assumptions can vary.

<a id="disk-090"></a>

### DISK-090 — NVMe PRP-list alignment mask truncates 64-bit pointers

- **Severity:** High (wrong-address writes/crash). **Confidence:** High.
- **Source:** `rom/devs/nvme/nvme_prp.c, prp_storage alignment assignment`.
- **Trigger/trace:** page_mask is ULONG (32 bits). Complementing it before AND with an IPTR pointer produces a zero-extended 32-bit mask on 64-bit builds, clearing all upper pointer bits. A PRP-list allocation above 4 GiB is therefore replaced with its low-address alias before list_page writes.
- **Consequence:** Transfers requiring a PRP list can corrupt unrelated low-address memory or fault on 64-bit machines. SGL fallback availability may avoid this path for some controllers.

<a id="disk-091"></a>

### DISK-091 — NVMe discovery consumes multibyte Identify fields without endian conversion

- **Severity:** High (incorrect discovery/geometry on big-endian builds). **Confidence:** High; architecture applicability conditional.
- **Source:** `rom/devs/nvme/nvme_controllerclass.c`, ctrl->nn tag; `nvme_busclass.c`, id_ns->nsze/ncap and rt->slba/nlb.
- **Trigger/trace:** Other command/result fields explicitly use AROS_LONG2LE/LE2LONG/LE2WORD, but namespace count, capacity and range fields are read directly from DMA structures. On a big-endian build the values are byte-swapped relative to the controller data representation.
- **Consequence:** Incorrect unit counts, huge allocations, absent units or wrong geometry/range boundaries. NVMe mmake definition has no endian restriction; actual hardware/backend availability must be considered before reproducing.

<a id="disk-092"></a>

### DISK-092 — ATA selection and diagnostic probing have unbounded busy waits

- **Severity:** High (boot or bus-task hang). **Confidence:** High.
- **Source:** rom/devs/ata/lowlevel.c, ata_SelectUnit and ata_ReadSignature.
- **Trigger/trace:** Selecting a different unit loops while BSY remains set without a timer or retry limit, before ata_exec_cmd reaches its bounded command wait. Signature probing issues EXECUTE DIAGNOSTIC and waits indefinitely for BSY despite the earlier bounded reset waits.
- **Consequence:** A stuck drive can hang discovery or permanently stop requests on its bus. Discovery holds DetectionSem, also blocking other bus discovery.

<a id="disk-093"></a>

### DISK-093 — ATA timeout emulation can loop forever with interrupts disabled

- **Severity:** High (system hang). **Confidence:** High.
- **Source:** rom/devs/ata/lowlevel.c:499, ata_WaitBusyTO; ata_IRQPIORead and ata_IRQNoData.
- **Trigger/trace:** IDENTIFY enables fake_irq. After its one-second timeout, code disables interrupts, samples status once, and repeatedly invokes ab_HandleIRQ with that unchanged status until the handler clears itself. With BSY set, ata_IRQPIORead falls through to ata_IRQNoData, which returns without clearing the handler. No further status read or timeout occurs.
- **Consequence:** Recovery spins indefinitely before Enable(), turning a failed IDENTIFY into a machine hang on affected builds.

<a id="disk-094"></a>

### DISK-094 — Failed ATA first-write handshake retains the caller buffer in its IRQ handler

- **Severity:** High (stale-buffer reads / unintended transfer). **Confidence:** High.
- **Source:** rom/devs/ata/lowlevel.c, ata_exec_cmd CM_PIOWrite early returns; rom/devs/ata/ata_busclass.c, Hidd_ATABus_HandleIRQ; rom/devs/ata/ata.c, bus request loop.
- **Trigger/trace:** PIO write stores the caller buffer before sending the command. Initial busy timeout or missing DRQ returns without clearing ab_HandleIRQ or resetting the command. The bus task replies the failed request. A later DRQ invokes ata_IRQPIOWrite and reads the retained buffer after caller reuse/free. Subsequent commands can also replace the handler while the earlier command remains active.
- **Consequence:** Late activity can consume freed/reused memory or interfere with the next request. Ordinary final IRQ timeout clears its handler; these earlier polling returns bypass cleanup.

<a id="disk-095"></a>

### DISK-095 — PCI ATA passes reversed DMA directions to cache helpers

- **Severity:** High (stale read data on noncoherent architectures). **Confidence:** High; architecture applicability conditional.
- **Source:** rom/hidds/ata_pci/interface_dma.c, dma_Setup and dma_Cleanup; rom/devs/ata/lowlevel.c, CM_DMARead/CM_DMAWrite; arch/ppc-chrp/exec/cachepostdma.c.
- **Trigger/trace:** ATA read calls DMA_Setup with read=TRUE, meaning device-to-memory (also confirmed by its inverse-logic DMA_WRITE register assignment). Both cache preparation and cleanup instead set DMA_ReadFromRAM when read is TRUE. Exec documents this flag as memory-to-device. PPC-CHRP CachePostDMA invalidates the data cache only when the flag is absent, so disk reads skip the required invalidation.
- **Consequence:** CPU accesses may return stale cached data after successful DMA reads. Cache-coherent x86 implementations are no-ops and avoid this manifestation.
- **Additional contract issue:** PRD-table CachePreDMA is never paired with CachePostDMA, and setup failure after data preparation returns without unwinding prepared mappings. Mapping/cache consequences depend on the platform implementation.

<a id="disk-096"></a>

### DISK-096 — PCI ATA interrupt acknowledgement clears DMA errors before checking them

- **Severity:** High (failed transfer reported successful). **Confidence:** High.
- **Source:** rom/hidds/ata_pci/ata_pci_busclass.c, ata_PCI_Interrupt; rom/hidds/ata_pci/interface_dma.c, dma_CheckErr; rom/devs/ata/lowlevel.c, ata_IRQDMAReadWrite.
- **Trigger/trace:** The IRQ handler reads DMA status, then acknowledges with dmastatus | DMAF_Error | DMAF_Interrupt before invoking the transport completion handler. dma_CheckErr subsequently rereads the hardware register rather than receiving the saved status. The acknowledgement clears the latched DMA error bit, so a bus-master failure without a separate ATA ERROR bit is lost.
- **Consequence:** The request can complete successfully and advertise its entire byte count despite failed DMA, returning invalid read data or concealing incomplete writes. The source explicitly describes these writes as clearing the bits; external hardware testing was not performed.

<a id="disk-097"></a>

### DISK-097 — ATA bus-task creation failure leaves discovery semaphore locked

- **Severity:** High (boot hang under resource exhaustion). **Confidence:** High.
- **Source:** rom/devs/ata/ata_busclass.c:1023, Hidd_ATABus_Start; rom/devs/ata/bootwait.c:93; rom/devs/ata/ata.c:1134.
- **Trigger/trace:** Start obtains DetectionSem shared before NewCreateTask. Only the created bus task releases this acquisition. If task creation fails, Start returns FALSE without releasing it. The boot-wait resident obtains the semaphore exclusively to wait for discovery completion.
- **Consequence:** Boot waits forever after a failed task allocation, even though startup reported failure and no task exists to release the semaphore.

<a id="disk-098"></a>

### DISK-098 — Gayle PIO assembly mishandles short transfers and modified operands

- **Severity:** High (memory corruption / runaway transfer). **Confidence:** High.
- **Source:** arch/m68k-amiga/hidd/gayle_ata/interface_pio.c, ata_insw/ata_outsw/ata_insl/ata_outsl; rom/devs/ata/lowlevel.c, ATAPI data handlers.
- **Trigger/trace:** Word routines unconditionally execute two word accesses before decrementing count >> 2; long routines execute two longs with count >> 3. A two-byte ATAPI pad/drain transfer therefore starts with zero iterations, performs four/eight bytes and decrements to 0xffffffff, looping through caller memory. Other lengths not divisible by four/eight lose their remainder. ATAPI handlers explicitly call Unit_InS/Unit_OutS with two bytes for odd data and draining.
- **Consequence:** Short or odd ATAPI phases can overwrite/read far beyond the buffer and hang the system. Separately, assembly changes count/address registers declared input-only and omits a memory clobber, violating compiler operand contracts and permitting miscompilation even for sector-sized transfers.
- **Configuration:** Active Amiga ata_gayle PIO vectors; 32-bit mode changes the grouping but does not fix the short-transfer problem.

<a id="disk-099"></a>

### DISK-099 — Gayle initializes the storage-controller method base from the wrong interface

- **Severity:** High (disk discovery failure / incorrect method dispatch). **Confidence:** High.
- **Source:** arch/m68k-amiga/hidd/gayle_ata/class_init.c, GayleATA_Init; probe.c, HIDD_StorageController_AddBus; rom/hidds/base/storage/storage.conf.
- **Trigger/trace:** HiddStorageControllerBase is assigned OOP_GetMethodID(IID_HW, 0), identical to HWBase, instead of the hidd.storage.controller interface base. The probe invokes HIDD_StorageController_AddBus using this base. storage.conf declares AddBus under hidd.storage.controller.
- **Consequence:** The call dispatches an unrelated HW method ID or fails to dispatch, preventing creation/setup of the Gayle disk bus and potentially interpreting incompatible arguments. Actual generated stub wiring should be inspected before runtime reproduction; no generated build or test performed.

<a id="disk-100"></a>

### DISK-100 — ATA probes read ownership markers after failure cleanup freed them

- **Severity:** Medium (use-after-free / possible double free). **Confidence:** High for the direct PCI path; Gayle remains conditional on DISK-099 dispatch being corrected.
- **Source:** `rom/hidds/ata_pci/probe.c:648-665`; `rom/hidds/ata_pci/ata_pci_busclass.c:217-239`; `arch/m68k-amiga/hidd/gayle_ata/probe.c:237-249`; `arch/m68k-amiga/hidd/gayle_ata/gayleata_busclass.c:150-162`; `rom/hidds/base/storage/storage_controllerclass.c:91-125`.
- **Trigger/trace:** The shared `AddBus()` constructs the bus and calls the controller's `SetUpBus()`; when setup fails, it disposes the constructed bus before returning NULL. Both PCI ATA and Gayle bus destructors free their `data->bus` (`probedbus`). Their probe callers then read fields from `probedbus` after the failed AddBus return to log the failure and inspect `atapb_Node.ln_Succ` (`ata_pci/probe.c:654-665`; `gayle_ata/probe.c:239-249`). PCI setup failure is reachable after successful bus construction, including mandatory PIO-interface setup failure, an empty legacy channel with `KeepEmpty=FALSE`, or ATA bus-task creation failure. The same Gayle setup-failure path exists, but `DISK-099`'s wrong storage-controller method base can prevent its call from reaching this AddBus implementation on the current build.
- **Consequence:** Both callers access freed probe-record memory during failure handling. If allocator reuse changes the ownership marker before the final check, the caller may also free the record again. PCI ATA is selected by i386-PC, x86-64-PC, and Sam440 module targets; Gayle is included by the Amiga boot manifest, subject to the DISK-099 dispatch limitation. **Related finding:** PCI's constructor-failure DMA-buffer leak is DISK-2661; the embedded controller-reference release is DISK-101. These are separate ownership defects.

<a id="disk-101"></a>

### DISK-101 — PCI ATA frees a reference embedded inside its controller object

- **Severity:** High (invalid free / allocator corruption on cleanup). **Confidence:** High.
- **Source:** rom/hidds/ata_pci/probe.c, atapb_Device assignment; ata_pci_busclass.c, DeviceUnref/DeviceFree; ata_pci_controllerclass.c.
- **Trigger/trace:** Each probed PCI bus points at &ataPCIData->ctrllrDevice, an embedded field in the controller's OOP instance. DeviceUnref decrements its count and, when the last reference is dropped, DeviceFree calls FreeMem(ref, sizeof(struct PCIDeviceRef)). No standalone allocation backs this pointer.
- **Consequence:** Last-bus startup failure or disposal frees an interior portion of a live controller allocation using the wrong allocation size, corrupting allocator state. PCI ownership release precedes this invalid free.
- **Limitations:** Ordinary bus removal is unsupported, but startup failure cleanup is reachable. Interaction with DISK-100 may add further invalid accesses; root causes remain distinct.

<a id="disk-102"></a>

### DISK-102 — ATA device-query response ignores caller buffer bounds

- **Severity:** Medium (caller memory overwrite). **Confidence:** High.
- **Source:** rom/devs/ata/ata.c, HandleIO NSCMD_DEVICEQUERY.
- **Trigger/trace:** The immediate query casts io_Data and writes every NSDeviceQueryResult field without checking io_Data or io_Length. A short response buffer receives the full structure, and NULL faults.
- **Consequence:** Malformed or undersized queries overwrite adjacent memory or crash instead of returning a length/address error. This is independent of the dangling supported-command array in DISK-009; ATA's array itself is static.

<a id="disk-103"></a>

### DISK-103 — ATA bus worker uses an unchecked timer allocation

- **Severity:** High (startup crash / boot discovery hang). **Confidence:** High.
- **Source:** rom/devs/ata/ata.c, BusTaskCode; rom/devs/ata/waitto.c, ata_WaitTO.
- **Trigger/trace:** The worker assigns ata_OpenTimer to ab_Timer without checking failure and proceeds to unit setup/IDENTIFY. ata_WaitBusyTO invokes ata_WaitTO, which immediately dereferences the timer's reply port. A successful earlier scan timer allocation does not guarantee this separate worker allocation succeeds.
- **Consequence:** Resource exhaustion can fault the worker while it holds the shared discovery semaphore, preventing boot-wait completion. Bounce-buffer pool creation in the same worker is also unchecked and needs separate allocation-path accounting.

<a id="disk-104"></a>

### DISK-104 — ATA fallback volume name is not terminated

- **Severity:** Medium (out-of-bounds string read / malformed boot name). **Confidence:** High.
- **Source:** rom/devs/ata/ata_init.c, ata_RegisterVolume fallback dosdevname.
- **Trigger/trace:** If shared storage ID allocation returns NULL, the code fills only dosdevname[0..2] in an uninitialized four-byte automatic array and passes it as pp[0] to MakeDosNode. It never assigns the final zero terminator.
- **Consequence:** DOS-node name construction reads beyond the intended three characters until an incidental zero, yielding stack-derived names or invalid reads. Successful ID allocation avoids this path.

<a id="disk-105"></a>

### DISK-105 — ATA expunge reuses and frees controllers already disposed by HW removal

- **Severity:** High (use-after-free / double disposal / allocator corruption). **Confidence:** High; expunge invocation conditional.
- **Source:** rom/devs/ata/ata_init.c, ata_expunge; rom/hidds/hidd/hwclass.c, HW_RemoveDriver; rom/devs/ata/ata_controllerclass.c, Root.Dispose.
- **Trigger/trace:** HW_RemoveDriver calls OOP_DisposeObject before returning TRUE. ATA controller disposal already removes ac_Node and calls its superclass to free the object. ata_expunge then calls Remove on that freed embedded node, reads ac_Class/ac_Object, invokes superclass disposal again and FreeMem on the embedded controller instance.
- **Consequence:** Expunging an initialized ATA device with a registered controller accesses freed state and performs invalid repeated frees. ATA config has no noexpunge option; normal device open counts and caller choice govern whether expunge runs.
- **Related limitation:** Generic storage does not override HW removal to reject controllers with live buses. Controller disposal also leaves bus tasks/units active; teardown can therefore leave workers with a freed controller even before the additional expunge errors.

<a id="disk-106"></a>

### DISK-106 — Shared storage ID allocation is not an atomic transaction

- **Severity:** Medium (duplicate boot names / list corruption). **Confidence:** High.
- **Source:** rom/hidds/base/storage/storage_class.c, AllocateID; rom/devs/ata/ata.c and ata_init.c, concurrent worker volume registration.
- **Trigger/trace:** Stem lookup/create, ID-list scan, number selection and insertion execute without a semaphore, Forbid or spinlock. Two workers can both find a missing stem and insert separate same-name stem nodes, or scan an existing stem before either inserts and choose the same next number. List insertion itself is also unprotected.
- **Consequence:** Concurrent disk discovery can assign duplicate HD/CD names and race list pointers. MEMF_SEM_PROTECTED applies only to a memory pool, not these AllocVec allocations or namespace transactions. Separate from DISK-001 formatting collisions.
- **Configuration:** Multiple bus workers can run concurrently; ordinary task preemption suffices, with additional exposure on SMP.

<a id="disk-107"></a>

### DISK-107 — ATA bounce allocation failure submits DMA at address zero

- **Severity:** High (wrong-memory DMA / disk corruption). **Confidence:** High for x86 PCI path.
- **Source:** rom/devs/ata/lowlevel.c, ata_exec_blk; rom/hidds/ata_pci/interface_dma.c, dma_Setup; rom/exec/cachepredma.c.
- **Trigger/trace:** A DMA request whose caller buffer lies above 4 GiB allocates a low-memory bounce buffer. Allocation failure is unchecked and replaces blk->buffer with NULL. Since bounce_buffer is NULL, write copying is skipped and ata_exec_cmd proceeds with this pointer. Generic x86 CachePreDMA returns its input unchanged, and PCI PRD construction accepts physical zero and starts DMA.
- **Consequence:** Disk reads overwrite physical memory starting at zero; disk writes consume unrelated physical memory instead of the caller data. Allocation failure can be reported as a successful transfer. Controller/platform protection can alter the resulting failure, but the wrong descriptor address is directly source-supported.
- **Additional path:** BusTaskCode also fails to check CreatePool; attempting AllocPooled through a missing pool can fault before descriptor submission.

<a id="disk-108"></a>

### DISK-108 — ATA polling treats BSY clear as complete before moving data

- **Severity:** High (false success / unfinished DMA / stale-buffer access). **Confidence:** High; timing-dependent.
- **Source:** rom/devs/ata/lowlevel.c, ata_WaitBusyTO and ata_exec_cmd; rom/devs/ata/ata_init.c, ATA=poll.
- **Trigger/trace:** ATA=poll forces the final command wait into its polling branch. That branch stops once BSY clears, without servicing DRQ, checking remaining PIO bytes, invoking the handler or waiting for its removal. Readiness for a data phase can therefore satisfy the completion wait before IRQ-driven transfer occurs. It also reads primary status, acknowledging the pending interrupt. ata_exec_cmd returns the initially zero command error and DMA_End may stop an unfinished engine.
- **Consequence:** The driver can report the full request successful with untouched/partial data and leave a PIO handler retaining the caller buffer after reply. Real IRQs arriving before the poll observes readiness may mask this defect; the option supplies no reliable completion invariant.

<a id="disk-109"></a>

### DISK-109 — AHCI recovery releases requests without establishing DMA has stopped

- Source: rom/devs/ahci/ahci.c:1180, :3310, :3337, :3970, :4104, :4233.
- Trigger: command timeout or error recovery when the controller cannot stop its command engine (CR remains set), or fails to clear active slots after the retry.
- Evidence: ahci_port_stop explicitly returns failure when CR does not clear. Interrupt recovery and both timeout implementations ignore that return. The interrupt path warns if activity remains but still computes completions from its edited ci_saved and calls ccb_done. The regular timeout path calls ccb_done and xa.complete before its later attempt to clear remaining activity. ahci_ata_cmd_done unloads the data mapping and calls the request completion callback. Quick timeout similarly declares TIMEOUT and removes software activity after merely logging persistent hardware activity, allowing callers to unload/reuse its scratch mapping.
- Impact: caller can reuse or free a buffer, and a slot can be reused, while an unquiesced controller may still access the old buffer/command. Memory corruption and incorrect I/O are possible; recovery needs a successful stop/reset or isolation before releasing ownership.
- Confidence: high for failure being ignored and premature ownership release; actual continuing DMA depends on the controller failure mode. This is a hardware-error-path candidate, not a claim about healthy-controller transfers.

<a id="disk-110"></a>

### DISK-110 — AHCI allocation wrappers do not provide the assumed infallible allocations

- Source: rom/devs/ahci/ahci_aros.h:160; rom/devs/ahci/ahci.c:214, :270, :297, :3749.
- Trigger: memory allocation failure during controller/port/DMA setup.
- Evidence: kmalloc simply returns AllocVec, which can return NULL. Temporary port and persistent port pointers are dereferenced without checks; target allocation is dereferenced during initialization; ahci_dmamem_alloc forms &adm->adm_kva and &adm->adm_map without checking adm.
- Impact: NULL dereference during driver initialization rather than clean resource failure.
- Confidence: high; M_WAITOK/M_INTWAIT names do not implement waiting or failure handling in this shim.

Follow-up candidate: the NCQ error-log tag is used as an `ap_ccbs` array index without checking against `sc_ncmds` (promoted as DISK-128 below). The `ahci_set_feature` NULL-transfer concern is not promoted for the selected call graph: its only callers are in `ahci_port_link_pwr_mgmt`, which has no caller elsewhere in the selected tree. Previously noted exclusive-command and reset-status candidates require caller/platform checks.

<a id="disk-111"></a>

### DISK-111 — AHCI callout cancellation does not synchronize callback lifetime

- Source: rom/devs/ahci/ahci_aros.c:35–110; rom/devs/ahci/ahci.c:3970 and :4030.
- Trigger: completion/cancellation races with timer expiry, particularly concurrent execution on different CPUs.
- Evidence: callout_stop only signals SIGF_ABORT and immediately clears co_Task. It neither joins the task nor waits for an already-started func(arg). The handler tests its previously returned signals, clears co_Task and invokes func with no cancellation-generation check. ahci_ata_cmd_done assumes cancellation has stopped the callback before clearing flags, unloading DMA and permitting CCB reuse. An old handler can also clear co_Task belonging to a replacement callout.
- Impact: stale timeout can modify a reused CCB or signal a freed port; replacement timeout can become uncancellable. Port teardown does not establish that all callout tasks have exited.
- Confidence: high for missing synchronization; interleaving depends on scheduling/SMP. Serial-number check after cancellation does not prevent a callback that starts afterwards.

<a id="disk-112"></a>

### DISK-112 — AHCI ignores helper and timeout-task creation failures

- Source: rom/devs/ahci/ahci_aros.c:222–232; ahci_aros.h:182; ahci_attach.c:715; ahci_busclass.c:231; ahci.c:2326.
- Trigger: NewCreateTask returns NULL during port startup or command timeout scheduling.
- Evidence: start_port sets INIT/THREAD_SYNC then ignores kthread_create's ENOMEM result. Only the missing helper clears those bits, while attachment and bus startup wait on them. ahci_start_timeout marks TIMEOUT_RUNNING and ignores callout_reset's ENOMEM result, leaving no task to expire an outstanding command.
- Impact: indefinite controller discovery wait, or outstanding command without its intended timeout recovery.
- Confidence: high; allocation-failure path.

Additional observations: nonblocking port-lock helper always returns failure, routing every IRQ through the port worker (performance/design limitation, not yet an independent correctness finding). Timer-open failure causes sleep helpers to return without delay; callout-handler timer-open failure invokes timeout immediately. Calibration return is ignored and scale-doubling can wrap in an extreme timing case; these need platform/trigger assessment before promotion.

<a id="disk-113"></a>

### DISK-113 — AHCI media-monitor creation failure hangs registration

- Source: rom/devs/ahci/ahci_cam_aros.c:161–184; rom/exec/newcreatetaska.c:190–272.
- Trigger: NewCreateTask fails while registering a port.
- Evidence: registration inserts the unit into the device list, assigns sim_Monitor from NewCreateTask and unconditionally Waits for SIGBREAKF_CTRL_C. NewCreateTaskA returns NULL when its allocation or NewAddTask path fails (:190–197,257–272), and no monitor can then issue the intended parent Signal (:69–70). NULL is never tested before Wait.
- Impact: discovery stalls indefinitely with a partially registered unit.
- Confidence: high. A pre-existing CTRL_C signal can instead spuriously satisfy the handshake; there is no task-specific handshake object.

<a id="disk-114"></a>

### DISK-114 — AHCI failed attachment leaves registrations pointing at freed controller data

- Source: rom/devs/ahci/ahci_attach.c:397–408, :789–860; bus_pci.c:199–208; ahci_controllerclass.c:25–62.
- Trigger: attachment fails after reset callback/controller/bus registration, including allocation or interrupt setup failure.
- Evidence: MSI attachment installs dev_ResetInt, but ahci_pci_detach never removes it or releases the obtained vectors. PCI discovery calls ahci_release on attach failure, freeing dev and its embedded softc, while the controller object retains ac_dev and remains registered. Successfully created bus objects likewise retain their port data, which detach frees. Controller Dispose merely removes its list entry and does not detach or call the superclass.
- Impact: later reset/attribute access can follow freed pointers; failed initialization leaks object/vector registrations. The reset callback itself is embedded in freed dev memory.
- Confidence: high for stale registrations on failure; subsequent use depends on reset or object access. Routine successful unload is explicitly unsupported, so this finding centers on the implemented failure cleanup path.

<a id="disk-115"></a>

### DISK-115 — AHCI attachment can leave initialized ports with interrupts masked

- Source: rom/devs/ahci/ahci_attach.c:709–753; ahci.c:574–584, :2677–2701; ahci_aros.c:188–208.
- Trigger: port helper completes its initial worker iteration and goes to sleep before attachment masks PxIE and clears pending PxIS.
- Evidence: attach waits only until THREAD_SYNC clears, then writes PxIE=0 for every port before enabling global interrupts. It does not restore ap_intmask. Initialization/worker restore PxIE, but they can already have run; clearing status also removes the event that could wake the sleeping helper. The explicit ahci_intr call only handles set global status bits.
- Impact: subsequent asynchronous commands receive no completion IRQ until another helper event (e.g. their timeout) happens to restore interrupts; media/hotplug notifications can be missed meanwhile.
- Confidence: medium-high; scheduling-dependent. A helper iteration after the masking can repair it, so this is not claimed to occur on every boot.

Additional substantiated observations awaiting grouping with allocation/addressing findings:
- OD2556 promotes the MapPCI NULL/resource-handle path: bus_alloc_resource_any returns a non-NULL resource after a failed map, and AHCI attach then dereferences the null bus handle. OD2557 records the separate unchecked IRQ-handle AllocMem when the pooled resource allocation succeeds.
- bus_pci.c reads AHCI VS directly through the raw 32-bit BAR5 value before MapPCI, rather than the mapped CPU address. This assumes identity MMIO mapping and no BAR flag bits requiring masking; architecture-dependent.
- Revision/hardware-name AllocVec calls are unchecked before CopyMem/sprintf/RawDoFmt; group with DISK-110 allocation-failure handling.
- CAM unregister frees ap_sim without clearing the pointer, but caller tracing did not establish a selected second-detach path: ahci_cam_attach unregisters itself on its own failure, ahci_pci_detach is only called on attach failures before the synchronous CAM scan, and the other bus-start caller is compiled out. Retain as a latent ownership concern, not a current project finding, unless a reachable reattach/detach caller is found.
- A previously considered error-CCB issue on controllers with fewer than two slots is rejected: attachment requires at least four command slots.
Port-multiplier source read through line 270; remainder and full CAM probing remain pending.

<a id="disk-116"></a>

### DISK-116 — AHCI native logical-sector size is assembled with reversed words

- Source: rom/devs/ahci/ahci_cam_aros.c:676–689; include/devices/atascsi.h:149; ahci_io.c:95–126.
- Trigger: ATA IDENTIFY word 106 marks an extended logical sector size (e.g. native 4096-byte sectors).
- Evidence: words_lsec begins at word 117. Like the adjacent ATA multword capacity fields, the first word is the low word, but probe builds (word117 << 16) + word118. For 4096 bytes, the 2048-word count becomes 0x08000000 words, and sector_size becomes 0x10000000 bytes. The driver uses this value for alignment, converting byte offsets/lengths to LBAs, geometry and boot environment.
- Impact: ordinary native-sector requests are rejected and reported geometry/addressing is wrong; requests accepted under the bogus size have a byte-length/ATA-sector-count mismatch.
- Confidence: high on little-endian systems. Numeric identify fields here also lack le16toh, so big-endian correctness needs a separate consolidated architecture assessment. The physical-size debug calculation uses division rather than the documented larger-physical-sector scaling, but its value is not used for I/O.

<a id="disk-117"></a>

### DISK-117 — Port-multiplier reset failure leaves the error CCB reserved

- Source: rom/devs/ahci/ahci_pm.c:773–890; ahci.c:3540–3630.
- Trigger: final retry of the second soft-reset FIS fails.
- Evidence: get_err_ccb sets ERR_CCB_RESERVED, saves software active masks/count and clears them. First-FIS failure puts the CCB back; second-FIS failure only puts it back if a retry remains. On the final failure it jumps directly to err, which sends further PM register commands and returns without ahci_put_err_ccb.
- Impact: reserved state persists and saved command accounting is not restored; later recovery can overwrite that saved state, lose outstanding commands or trip assertions.
- Confidence: high for missing release. Impact on other requests depends on the recovery context.

<a id="disk-118"></a>

### DISK-118 — AHCI boot-volume ID fallback dereferences NULL

- Source: rom/devs/ahci/ahci_cam_aros.c:283–303.
- Trigger: HIDD_Storage_AllocateID returns NULL while boot volumes are registered.
- Evidence: fallback selects a local name, then the unconditional pp[0] assignment immediately dereferences ap_IDNode->ln_Name anyway.
- Impact: boot-time NULL dereference under allocation failure or an unsupported ID-allocation interface.
- Confidence: high. Fallback name is also unterminated, but that is masked by the immediate NULL dereference.

<a id="disk-119"></a>

### DISK-119 — AHCI repeated detach accesses the freed CAM unit after probe failure

- Source: rom/devs/ahci/ahci_cam_aros.c:195–228, :336–389, :810–839; ahci_attach.c:764–774, :815–821.
- Trigger: IDENTIFY/probe fails during cam_attach, and controller cleanup subsequently detaches the port.
- Evidence: cam_attach registers ap_sim, then calls probe; on failure it calls cam_detach. Unregister removes/frees unit but never clears ap_sim. Attachment treats the failed cam_attach as a skipped port, leaving it installed. A later controller detach calls cam_detach again, whose non-NULL ap_sim test enters Unregister on freed memory.
- Impact: use-after-free, invalid monitor signalling, or double list removal/free.
- Confidence: high for the reachable sequence; controller detach is primarily an attachment-failure path in this revision.

Grouped updates:
- DISK-089 also applies to AHCI boot-node pp: pp[3] (OpenDevice flags) and SECORG/PREALLOC/INTERLEAVE environment fields remain uninitialized.
- Successful AHCI RegisterVolume returns without closing its opened expansion.library reference, as in ATA's corresponding path.
- PM read failure explicitly zeroes its output through ahci_pm_read; the apparently uninitialized feature-register locals in pm_identify are therefore not a finding.
- Both direct and PM softreset detect signature mismatch, set EBUSY, then unconditionally overwrite error with zero; OD2560 in the [Other Drivers report](AROS-other-drivers.md) now traces the resulting stale-type IDENTIFY calls. DISK-117 remains the separate PM final-retry error-CCB release defect.
- PM register helpers dereference `ahci_ata_get_xfer` without a NULL check, but the selected PM probe/reset call chain issues these synchronous helpers serially during startup before external storage I/O is available; no reachable exhaustion path was established there. CAM feature/IDENTIFY helpers similarly were not promoted as duplicates of DISK-013 without a same-state exhaustion trace. `ahci_set_feature` is only called by the otherwise unreferenced `ahci_port_link_pwr_mgmt` helper.
Remaining AHCI work: finish public/private headers and configuration cross-check; re-read SCSI/IO code without output truncation; resolve NCQ log-index, exclusivity, media/hotplug and architecture candidates.

<a id="disk-120"></a>

### DISK-120 — AHCI autosense writes 32 bytes after validating only 18

- Source: rom/devs/ahci/ahci_scsi.c:183–232; ahci_scsi.h:211–252.
- Trigger: failed disk/ATAPI request with AUTOSENSE or OLDAUTOSENSE and an 18–31-byte sense buffer.
- Evidence: completion checks SenseLength against offsetof(extra_bytes), 18. ahci_ata_sense then memset clears sizeof(scsi_sense_data), 32, through the caller pointer. The reported SenseActual is only 18.
- Impact: writes up to 14 bytes past a normal minimum-size sense buffer.
- Confidence: high; the accepted minimum and actual write extent are explicit.

<a id="disk-121"></a>

### DISK-121 — AHCI SCSI translation silently narrows LBAs and transfer counts

- Source: rom/devs/ahci/ahci_scsi.c:436–543; ahci_io.c:125–174.
- Trigger: READ/WRITE(16) LBA >= 2^48; READ/WRITE(12/16) count > 65536; READ/WRITE(10/12/16) zero count.
- Evidence: SCSI LBA/count are decoded into 64/32-bit values but ATA FIS receives only 48/16 bits without range checks or splitting. Count zero is sent directly, although zero means no blocks for these SCSI commands and 256/65536 blocks in the selected ATA command. No check relates CDB count to scsi_Length or at_capacity. Ordinary trackdisk requests can also exceed 65536 sectors since BeginIO does not enforce the advertised maximum.
- Impact: high LBAs alias lower disk locations; oversized counts can perform partial/wrong transfers; zero-block SCSI requests can instead initiate real disk access. DMA length/count mismatch can yield hardware errors or falsely reported transfer length.
- Confidence: high for narrowing/zero semantic mismatch. Hardware behavior when PRDT length disagrees with command count varies; no claim that DMA must exceed the supplied PRDT.
- Related: DISK-015 is the separate PRDT 4 MiB limit. The LBA48 command selection also does not verify device LBA48 support.

<a id="disk-122"></a>

### DISK-122 — AHCI ignores its computed residual transfer length

- Source: rom/devs/ahci/ahci.c:2203–2214; ahci_scsi.c:203–217.
- Trigger: successfully completed non-NCQ operation transfers fewer bytes than requested, including shortened ATAPI INQUIRY/REQUEST SENSE.
- Evidence: unload calculates xa.resid from PRDBC. Completion always reports xa.datalen for scsi_Actual or io_Actual. ATAPI translation explicitly shortens some CDB allocation lengths without reducing datalen.
- Impact: callers treat untouched trailing buffer bytes as returned data, and short transfers are reported as full.
- Confidence: high; NCQ has no PRDBC-based residual here and is not the claimed short-transfer trigger.

<a id="disk-123"></a>

### DISK-123 — AHCI disk writes ignore SCSI force-unit-access

- Source: rom/devs/ahci/ahci_scsi.c:472–543; ahci_scsi.h:271–310; ahci_cam_aros.c:604–624.
- Trigger: WRITE(10/12/16) has FUA set on a disk with volatile write caching enabled.
- Evidence: translation reads only opcode/address/count, never byte2 FUA. It issues ordinary cached ATA writes with neither supported ATA FUA encoding nor a subsequent flush, while probe can enable the device write cache.
- Impact: a successful FUA write does not establish durable data, permitting acknowledged data loss after power failure.
- Confidence: high, conditional on volatile caching. Distinct API trigger from CMD_UPDATE no-op in DISK-010.

<a id="disk-124"></a>

### DISK-124 — AHCI FORMAT validates against one track instead of disk capacity

- Source: rom/devs/ahci/ahci_io.c:326–344.
- Trigger: TD_FORMAT/FORMAT64 writing at or beyond one track, or formatting a whole first track.
- Evidence: both length and byte offset are compared with nsectors*sector_size - 1, where nsectors is sectors per track; the actual capacity and heads/cylinders are ignored. For conventional 63*512 geometry a 32256-byte request is rejected, as is any request at byte offset 32256 or above. Accepted formatting is then treated as a normal write.
- Impact: standard formatting of later disk regions fails despite advertised FORMAT64 support.
- Confidence: high; native-sector size corruption in DISK-116 is an additional independent problem.

Additional reviewed outcomes:
- ATAPI validates packet CDB length 6–16 and checks transfer availability, unlike disk translation. Disk CDB length, response buffer NULL and opcode-specific minimal lengths remain unchecked (group with the disk SCSI bounds family).
- ATAPI READ/WRITE(6) conversion retains the upper three LUN bits in the address instead of applying SRW_TOPADDR; malformed/nonzero-LUN input can change the translated LBA.
- Immediate disk emulation does not consistently initialize scsi_Status/Actual/SenseActual. READ CAPACITY's separate command-buffer defect (DISK-012) masks its wrong saturation at capacities above 2^32 and hardcoded 512-byte sectors.
- AbortIO is a no-op returning zero; change-interrupt requests stay pending permanently unless explicitly removed through another request. Exact REMCHANGEINT request reuse/list ownership and caller expectations remain to be checked before assigning an independent finding.
- Media readiness for disks is always synthesized as success; physical removal OffLine publication and stale capacity/identity on replacement still require consolidated hotplug analysis.
- Existing DMA allocation review needs one further check: slab segment allocation uses dt_maxsegsz whereas load size is dt_maxsize, relevant to command-table storage. This is now an explicit next action.

<a id="disk-125"></a>

### DISK-125 — AHCI never runs its state machine after initial startup

- Source: rom/devs/ahci/ahci_aros.c:188–208; ahci.c:828–1010, :2677–2701, :3440–3479.
- Trigger: disk insertion after boot, or fatal error selecting NEED_REINIT.
- Evidence: the only call to ahci_port_state_machine is the initial port-thread setup. Worker iterations call ahci_port_thread_core, which processes timeouts/interrupts but never runs the state machine. NEED_REINIT/INSERT merely sets NEED_INIT and calls cam_changed; the AROS cam_changed implementation toggles OffLine and marks SCAN_COMPLETED without scheduling or executing reset/IDENTIFY. NEED_REINIT additionally leaves FATAL_ERROR and IN_RESET set.
- Impact: newly inserted media is not reset/identified; the unit may be marked online prematurely, with stale identity/capacity/type. Fatal recovery cannot restore normal operation through the intended state progression.
- Confidence: high; exhaustive symbol search within the built AHCI sources establishes the missing later call.

<a id="disk-126"></a>

### DISK-126 — AHCI port-multiplier targets have no addressable public units

- Source: rom/devs/ahci/ahci_cam_aros.c:158–193, :397–412, :983–1018; ahci_unitclass.c:62–70; ahci_io.c:78–80, :239–240; ahci_scsi.c:339–341, :587–589.
- Trigger: detected SATA port multiplier with one or more attached disks.
- Evidence: registration creates one cam_sim per physical port, numbered host*32+port. All public I/O selects ap_ata[0] and dispatches only when ap_type is DISK/ATAPI; a multiplier has ap_type PM, so dispatch rejects it. Probe creates unit objects using only the parent bus (constructor again selects target zero), and RegisterVolume rejects parent type PM. Each successful target probe still calls OOP_NewObject, whose constructor overwrites the bus's sole ab_Unit pointer; earlier unit objects are not disposed (`ahci_cam_aros.c:1012–1017`; `ahci_unitclass.c:63–70,81–90`). cam_changed ignores target identity when changing the sole sim's online state.
- Impact: disks behind a detected multiplier cannot be selected or used through the exported device interface; per-target probe also leaks the superseded unit objects, and one target removal can mark the shared physical-port sim offline.
- Confidence: high; applies to AROS glue, despite low-level PM probing/reset implementation.

<a id="disk-127"></a>

### DISK-127 — AHCI standard-command batching can issue work after an exclusive command

- Source: rom/devs/ahci/ahci.c:2574–2601; ahci_scsi.c:613–616; ahci_pm.c:770–773.
- Trigger: pending non-NCQ queue contains an AUTOSENSE/EXCLUSIVE command followed by an ordinary command, with a batching limit greater than one.
- Evidence: after issuing a command, loop breaks when run_flags contain neither exclusive flag, but continues when an exclusive flag is present. The next-command check only stops if the next command itself is exclusive. Thus an exclusive first command and ordinary second command can both be made active, and run_flags is overwritten by the ordinary command. Entry guards protect against a previously active exclusive command but not this same-loop transition.
- Impact: violates the serialization needed for reliable RFIS/autosense attribution and reset-sensitive sequences.
- Confidence: high for queue/state condition; normal public ATAPI concurrency often queues AUTOSENSE work and can supply ordinary following requests. Non-FBSS PM limit one masks that configuration.

<a id="disk-128"></a>

### DISK-128 — AHCI NCQ error-log tag can index beyond allocated CCBs

- Source: rom/devs/ahci/ahci.c:390–391, :3703–3722; ahci_attach.c:439–445.
- Trigger: invalid/device-corrupted NCQ error log reports tag >= sc_ncmds on a controller exposing fewer than 32 slots.
- Evidence: CCB array is allocated for sc_ncmds. READ LOG extracts five-bit tag and immediately accesses ap_ccbs[tag] without comparing it with allocated/negotiated depth. Minimum four-slot check does not impose 32 slots.
- Impact: out-of-bounds read/write when copying error register FIS or handling diagnostic target pointers.
- Confidence: high for missing validation; conditional on erroneous device response, not normal NCQ tags.

Rejected/qualified pending observations:
- Command-table slab undersizing rejected: its maxsegsz is sc_ncmds*sizeof(table), matching maxsize and the per-port allocation. Header and RFIS sizes likewise match their accesses.
- NULL timeout callback candidate rejected for current code: rechecked every selected `ahci_poll` call site in `ahci.c` and `ahci_pm.c`; each passes `ahci_quick_timeout` or `ahci_ata_cmd_timeout`.
- Early port-free NULL ap_sc path via bus_space_subregion failure is unreachable with the current shim, which always returns success.
- Slab bitmap width is smaller than the tag's allowed segment count, but current bus_dmamem_alloc calls use one-segment tags. Data tag's 1016 segments do not enter this slab allocator; no reachable independent finding assigned.
- Exclusive-access wait cannot run timeout recovery itself, but current state-machine use is initial startup with no external commands. The missing post-start state-machine call masks the suspected later recovery deadlock; do not claim current runtime reachability without another caller.
- Initial probe can finish before HW bus-object publication: cam_probe then gets no bus data, fails OOP unit creation, still marks GOOD; later cam_attach only retries probe in NEED_IDENT. This is a scheduling-dependent missing unit/boot-node candidate pending final startup ordering assessment.
- Raw MMIO accesses are host-endian and bus_space_barrier is empty. Together with unconverted identify fields and ignored DMA constraints, retain an architecture-dependent correctness limitation; target build applicability still needs global architecture inventory.

<a id="disk-129"></a>

### DISK-129 — VirtIO ignores negotiated per-segment transfer limits

- Source: rom/devs/virtio/virtio_controllerclass.c:91–100; virtio_hw.c:249–286; virtio_queue.c request descriptor construction.
- Trigger: device advertises nonzero size_max smaller than the requested data buffer, or seg_max below the submitted data-descriptor count.
- Evidence: SIZE_MAX and SEG_MAX are negotiated but neither configuration field is read/enforced. Submission places the entire data buffer in a single descriptor, without splitting to size_max or rejecting it.
- Impact: driver sends requests outside accepted device limits, causing errors or incomplete transfers; published filesystem maximum does not substitute for device-specific limits.
- Confidence: high for missing enforcement. Single data descriptor satisfies ordinary seg_max >=1; size_max is the main normal trigger.

<a id="disk-130"></a>

### DISK-130 — VirtIO startup dereferences failed unit-array and storage-ID allocations

- Source: rom/devs/virtio/virtio_busclass.c:99–100, :225–254.
- Trigger: AllocMem for ab_Units or HIDD_Storage_AllocateID fails.
- Evidence: bus construction returns success with NULL ab_Units; startup writes ab_Units[0]. Storage-ID allocation is immediately followed by ab_IDNode->ln_Name without a check.
- Impact: kernel NULL dereference during disk discovery.
- Confidence: high. String allocation failures additionally reach unit fixed-length copying (DISK-008); group there rather than add duplicates.

<a id="disk-131"></a>

### DISK-131 — VirtIO queue teardown does not revoke asynchronous users

- Source: rom/devs/virtio/virtio_controllerclass.c:161–183; virtio_busclass.c:43–63, :165–194; virtio_queue.c:96–112.
- Trigger: controller object disposal after queue/task/IRQ activation, including recovery from partially initialized objects.
- Evidence: free_queue frees ring, request array and queue without device reset, draining requests, removing its embedded IRQ handler, or terminating/joining q_IOTask. The completion task loops indefinitely and the IRQ keeps the queue pointer. Controller disposal also leaves its reset callback installed and frees the object containing that callback through superclass disposal.
- Impact: DMA/IRQ/task use-after-free and stale reset callback if this disposal path is invoked.
- Confidence: high for unsafe disposal implementation; normal runtime unloading is disabled and RemoveBus returns FALSE, so do not present as an unconditional successful-boot fault.
- Related startup failures after task creation return FALSE without stopping that task/IRQ or reversing DRIVER_OK. Bus disposal frees strings but not units/ab_Units, leaving leaked partial resources.

Other reviewed outcomes:
- Timer construction checks and unwinds port/request/device failures; WaitTO aborts and waits for its timer when another signal wins. No new timer-specific finding assigned.
- Init error paths fail to consistently close utility.library/release method bases, storage root or pool; failed HW_AddDriver also leaves claimed PCI devices allocated/owned. In particular, VIRTIO_Init opens Utility and only its missing-kernel.resource branch closes it; later CreatePool, base-array and storage-root failures return without a matching Utility close (virtio_init.c:73–85,113–140). If both storage-root constructors fail, acquired pool and OOP bases also remain. Generated failed-init rollback has no local VirtIO expunge hook to release those manually held resources (compiler/autoinit/functions.c:42–84; virtio.conf noexpunge). Consolidate as initialization resource-leak observations.
- The VirtIO enumerator accepts legacy block product 0x1001 and obtains PCI ownership (virtio_init.c:210–250), but its controller constructor requires modern vendor capabilities and returns a nonnull object on failed mapping (virtio_controllerclass.c:44–75; virtio_hw.c:45–118). On a legacy-only device, hardware-root registration can succeed without creating a bus, and the selected controller disposer has no PCI release. This is a separate legacy support/claim observation from modern product filtering in DISK-303; no legacy device was probed.
- Boot-node packet has uninitialized pp[3] and omitted environment fields (also BAUD here), extending DISK-089's shared packet-initialization family.
- Modern device filtering relies on subsystem ID instead of product ID minus modern base; source comments claim subsystem identifies class. Need repository-defined PCI contract comparison before promoting.
- Device capacity/config reads do not retry on config_generation changes and ISR handler ignores configuration-change status. This observation is resolved as DISK-251, with contract and build applicability documented there.
Subsequent VirtIO header/capability and final build reconciliation records close the source coverage items identified here. Final candidate normalization remains separate.

<a id="disk-132"></a>

### DISK-132 — VirtIO accesses PCI BAR bus addresses as CPU memory without mapping

- Source: rom/devs/virtio/virtio_hw.c:24–43, :77–108; rom/hidds/pci/pcideviceclass.c:815–829.
- Trigger: platform where PCI memory addresses differ from mapped CPU virtual addresses.
- Evidence: PCI Base getter returns masked BaseReg.addr for memory BARs, without MapPCI. VirtIO directly adds capability offsets and dereferences the resulting pointer for all configuration/notification/ISR accesses. No HIDD_PCIDriver_MapPCI is called in this transport.
- Impact: invalid CPU memory accesses or access to unrelated address space during startup/I/O.
- Confidence: high for mapping omission; conditional on nonidentity mapping. Identity-mapped PC targets can mask it; RISC-V boot inventory makes architecture tracing material.

<a id="disk-133"></a>

### DISK-133 — VirtIO reset polling counter underflows and permits failed reset

- Source: rom/devs/virtio/virtio_hw.c:121–138.
- Trigger: device_status never becomes zero throughout reset polling.
- Evidence: while(status != 0 && spin--) evaluates the post-decrement once with spin==0, wrapping unsigned spin to ULONG_MAX before loop exits. The subsequent if(!spin) does not detect exhaustion. It then writes ACKNOWLEDGE/DRIVER despite unconfirmed reset.
- Impact: proceeds to feature/queue initialization while previous device activity may persist; intended reset failure is suppressed.
- Confidence: high. A status transition exactly at the counter boundary can also produce false failure, but persistent nonzero status establishes the principal bug.

<a id="disk-134"></a>

### DISK-134 — VirtIO capability offsets and lengths are never validated

- Source: rom/devs/virtio/virtio_hw.c:55–106, :220–227.
- Trigger: malformed capability advertises short common/device window, offset outside BAR, misalignment, or notify offset/multiplier beyond its notify window.
- Evidence: cap_len and length are read but unused; BAR size/type is not verified. Code reads notify multiplier at cur+16 without requiring extended capability length, casts arbitrary region to register structs, and computes notification address without bounds/overflow checks.
- Impact: out-of-window MMIO reads/writes, potentially to unrelated hardware, or alignment faults.
- Confidence: high for absent checks; conditional on erroneous device configuration.
- Limitation: capability traversal is bounded to 64 iterations, so a cyclic linked list does not cause an unbounded walk.

Additional outcomes/limitations:
- Header itself defines modern block product 0x1042, but enumeration instead accepts all modern IDs whose subsystem ID is 2. This can reject modern block devices with different subsystem IDs or claim a different modern device class; retain as a source-supported identification candidate pending final protocol comparison.
- Hardware structs/registers/ring fields are accessed without little-endian conversion. PC targets mask it; no big-endian boot inclusion established by this search, so retain architectural limitation rather than unconditional fault.
- Queue completed used.len is discarded and full requested data length reported. This mirrors AHCI DISK-122's transfer-accounting family; device status OK with a short written length can expose untouched caller-buffer bytes as valid data.
- Device-config capacity reads lack config_generation sampling and config-change handling; dynamic resize consistency remains an explicit limitation.
- Queue allocator rejects unavailable/small hardware queues cleanly but hardcodes desired size 128 instead of adapting to supported smaller queues, a compatibility limitation rather than corruption candidate.

<a id="disk-135"></a>

### DISK-135 — Hostdisk worker port failure leaves the open result unset

- Source: arch/all-hosted/devs/hostdisk/hostdisk_device.c:142–165, :484–499.
- Trigger: CreateMsgPort returns NULL in unitentry.
- Evidence: worker signals the parent and returns without setting the open request's io_Error. Parent decides registration solely from that field. If it is zero, it publishes a unit whose port is NULL and whose worker has exited.
- Impact: subsequent queued I/O or CloseDevice dereferences the missing port, or requests cannot complete. If entry error is already nonzero, the failure is detected but lacks the intended NoMem result.
- Confidence: high for missing assignment; false success depends on the incoming/generated open-error initialization contract. Keep the conditional explicit rather than assume all opens initialize zero.

<a id="disk-136"></a>

### DISK-136 — Hostdisk does not join unit workers before library cleanup

- Source: arch/all-hosted/devs/hostdisk/hostdisk_device.c:44–58, :179–199, :643–657; hostdisk.conf.
- Trigger: final CloseDevice followed promptly by permitted library expunge, while unit worker has not processed its quit signal.
- Evidence: Close removes the unit and signals the worker but does not wait. Cleanup drops the host function interface and closes its host library without stopping/joining workers. Worker subsequently calls Host_Close through hdskBase->iface and executes driver code before freeing its unit. Configuration does not disable expunge.
- Impact: potential use of freed host interface/library base or unloaded driver code during asynchronous close.
- Confidence: medium-high; library lifetime depends on generated close/expunge scheduling, which remains to be checked in the shared module-generation contract.

Additional concrete outcomes:
- Worker Host_Open failure never deletes its successfully created MsgPort; normal worker exit likewise omits DeleteMsgPort. Repeated open/close or failed opens leak ports/signal resources.
- Numeric Open allocates a formatted filename before FindName; finding an existing unit returns without freeing this new name. Unit-structure allocation failure also leaks that name. UNIX's separate UNIT_FREENAME clearing is already DISK-026.
- Automount packet pp[3], SECORG/PREALLOC/INTERLEAVE/BAUD are uninitialized; extend DISK-089's shared boot-packet family.
- Automount name buffer uses strlen(format)+5 with no explicit formatted-output bound. Current Darwin numeric range 0–29 and UNIX letter templates fit; Windows numeric Open allows 0–1023 and likewise fits its supplied template. Do not claim overflow for ordinary built-in templates.
- UNIX Host_Open branches on errno without first checking open failed. A successful open can leave stale EBUSY/EROFS, cause an unnecessary second open and leak the first descriptor; retain as a conditional resource/permission-semantics observation.
- Linux geometry build file's MM dependency names Darwin, unlike its arch-specific linux selection. Record build-integration inconsistency for final inventory audit, not a proven runtime backend omission.

<a id="disk-137"></a>

### DISK-137 — Empty SCSI subsystem expunge leaves its daemon running

- Source: rom/devs/scsi/scsi_init.c:248–281, :284–333; scsi.c:875–970; scsi.conf:1–12.
- Trigger: successful SCSI library initialization with no registered controllers, followed by expunge.
- Evidence: init starts daemon unconditionally. Expunge stops daemon only inside its controller traversal. With an empty controller list it skips shutdown and returns TRUE after closing utility.library/releasing bases. Daemon retains the base pointer and loops through list/semaphore/timer operations and driver code. Configuration permits expunge.
- Impact: live task accesses unloaded driver code/freed library state.
- Confidence: high for empty-list teardown omission; conditional on module being loadable/built (see DISK-138).

<a id="disk-138"></a>

### DISK-138 — SCSI build includes a nonexistent case-sensitive header name

- Source: rom/devs/scsi/scsi_busclass.c:24; rom/devs/scsi/mmakefile.src:27–35.
- Trigger: building kernel-scsi on a case-sensitive filesystem from this source tree.
- Evidence: source includes scsi_Bus.h, while tracked local file is scsi_bus.h. File inventory contains no scsi_Bus.h. Build includes scsi_busclass in module sources, and no header-generation rule in its mmakefile supplies that spelling.
- Impact: preprocessing fails, blocking this subsystem build.
- Confidence: high from source inventory; no build run, in accordance with audit constraints.

Grouped/qualified findings:
- SCSI expunge after successful HW_RemoveDriver repeats the ATA DISK-105 root cause: disposal already removes/frees controller instance, then expunge removes/accesses it again, calls superclass disposal and FreeMem. Requires an actual controller and successful removal.
- No-controller daemon shutdown omission is separate from double-disposal because it requires no controller and does not enter that branch.
- Daemon init failure is checked through scsi_Daemon after handshake, unlike ATA's controller-daemon case. Do not duplicate that ATA claim here.
- Failed init/expunge omit DeletePool and method-base release; successful boot-node registration leaks expansion.library reference. Shared boot-packet uninitialized fields extend DISK-089, with skeleton reachability limitation.
- Timer allocation unwinds cleanly within OpenTimer; unchecked caller in scsi_InitBus remains a conditional startup NULL-dereference candidate.
- Calibration scale can wrap to zero if its largest trial is still under 10ms; inherited extreme-case issue, not a normal-speed claim.
- Bus disposer correctly removes its reset callback before freeing interfaces. Controller RemoveBus explicitly refuses unloading.

<a id="disk-139"></a>

### DISK-139 — MDS accepts unsupported track modes with unsafe sector buffers

- Source: workbench/devs/diskimage/plugins/mds.c:278–324, :428–462.
- Trigger: MDS track mode other than UNKNOWN/AUDIO/MODE1, including the defined MODE2 variants, with sector_size below 2048.
- Evidence: non-UNKNOWN modes allocate a track with MEMF_ANY. Switch only initializes audio and validates size for AUDIO/MODE1; there is no default rejection. Other modes leave audio uninitialized and accept arbitrary sector_size. Data read allocates sector_size bytes and copies 2048 bytes from that allocation.
- Impact: if uninitialized audio is false, out-of-bounds heap read copied to caller; valid MODE2 images also have nondeterministic classification and missing sync-offset handling.
- Confidence: high for initialization/validation omission; exact path depends on heap audio byte. Malformed metadata supplies the small-sector trigger.

<a id="disk-140"></a>

### DISK-140 — MDS accepts images with no tracks and later dereferences NULL

- Source: workbench/devs/diskimage/plugins/mds.c:262–270, :325–327, :406–414, :504–512.
- Trigger: descriptor advertises positive session_end but zero track blocks, or every block has UNKNOWN mode.
- Evidence: opener never checks image.tracks before done=TRUE. Read and ReadCDDA check only offset against total_blocks, then immediately use track->sectors.
- Impact: kernel/plugin NULL dereference on first read.
- Confidence: high. CCD explicitly rejects an empty track list and avoids this particular defect.

Additional outcomes:
- CCD fixed 2352-byte sectors and validated mode-specific sync offsets leave >=2048 bytes available for data extraction; no analogous sector-buffer overread found there.
- Both plugins stop at linked-list end on transfers crossing available tracks, returning error/short CDDA length; no unbounded track walk found.
- CCD index values and declared track extents are not checked against backing file size. MDS offsets/sector extents likewise lack backing-size checks; seeks/reads generally catch absent bytes, but short-read IoErr-to-device conversion must be traced before concluding failure is reliably reported.
- MDS num_tracks is taken from descriptor and narrowed to UBYTE independently of the actual allocated list. Track-list consumers/TOC must be traced for count/list disagreement.
- CCD also stores num_tracks in UBYTE, permitting metadata with >255 track sections to wrap the exported count; malformed-input metadata issue.
- CCD parser retains current track across non-TRACK sections, so subsequent MODE/INDEX keys in another section can alter the preceding track. No ordinary CloneCD layout demonstrating this trigger established yet.
- MDS reads only first session and ignores footer/multiple-file metadata, a format support limitation; do not claim full multisession support.

<a id="disk-141"></a>

### DISK-141 — NRG DAO track count exceeds allocated metadata

- Source: workbench/devs/diskimage/plugins/nrg.c:262–300, :308–352.
- Trigger: DAOI/DAOX chunk contains one track record but declares two or more tracks via first_track/last_track.
- Evidence: minimum-size check requires only header plus one record. Loop indexes tracks[i] for the independently declared count without checking chunk.size against header + count * record size.
- Impact: out-of-bounds heap reads while decoding sector sizes/offsets; adjacent heap bytes can become track metadata.
- Confidence: high; malformed image trigger. Both DAO variants affected.

<a id="disk-142"></a>

### DISK-142 — GI reads from stale file positions

- Source: workbench/devs/diskimage/plugins/gi.c:246–267.
- Trigger: read offset zero after an earlier read, or read across a split-file boundary whose next file handle was previously used.
- Evidence: first handle is sought only if in_offs is nonzero; subsequent handles are never sought when advancing fn. Persistent handles retain their prior positions.
- Impact: returns unrelated later bytes as the requested sectors, or incorrectly reports EOF on otherwise valid reads. Repeated reads of block zero demonstrate the first case without malformed input.
- Confidence: high; applies to normal GI image use, including a single file.

Additional outcomes:
- NRG accepts index2 below index1, causing unsigned length underflow and bogus capacity/track mapping. Physical reads still use checked seek/read calls; no independent memory write established from this arithmetic alone.
- NRG total_blocks narrows the computed UQUAD byte count to ULONG sectors; very large images truncate advertised addressable capacity.
- Invalid NRG first/last track ranges can wrap num_tracks to zero and accept an empty image, but total_blocks also stays zero; the read offset check prevents the MDS positive-capacity NULL-track defect in this case.
- NRG supported sector sizes initialize audio and have enough data bytes; no MDS-style uninitialized mode defect found.
- GI filename extension pointer subtraction depends on the loader selecting it through the .gi check; no ordinary loader path bypassing that precondition established.
- Short reads in NRG, CCD, MDS and other plugins using DOS2IOErr(IoErr()) do not become success when IoErr is zero: shared converter's default is TDERR_NotSpecified. That earlier suspicion is resolved, although the reported error may be nonspecific.
- GI split-name formatting has an eight-byte destination; sufficiently large part numbers truncate, a format/scale limit requiring further evidence before claiming a normal-image bug.

Review remains in progress: remaining disk-image formats/support, SD/MMC, USB transport/class integration, partition callers and final inventory/deduplication are not yet complete.

<a id="disk-143"></a>

### DISK-143 — CPC multi-track transfers overrun the track-offset array

- Source: workbench/devs/diskimage/plugins/cpc.c:243–278, :294–327.
- Trigger: read/write spanning beyond remaining tracks, or a transfer starting partway into a track and crossing its boundary.
- Evidence: ptr advances per iteration, but track never increments, so the bounds check continually tests the original valid track. to_skip also never resets, skipping that many bytes in every subsequent track.
- Impact: wrong sector data/overwritten sectors for ordinary multi-track requests; longer requests read past track_offsets and use heap bytes as file seek positions, potentially corrupting arbitrary image offsets on writes.
- Confidence: high; both callbacks affected.

<a id="disk-144"></a>

### DISK-144 — CPC accepts zero sizes and divides by zero

- Source: workbench/devs/diskimage/plugins/cpc.c:136–143, :172–175, :218, :243, :294.
- Trigger: header track size exactly 256, or zero-sector tracks / size exponent yielding a zero UWORD sector_size.
- Evidence: opener allows track_size >=256 then subtracts 256 without rejecting zero. sector_size remains zero if all sector counts are zero; 128 << buf[20] is unvalidated and narrows to UWORD (exponent 9 gives 65536 -> zero; larger shifts can be undefined).
- Impact: division by zero in read/write or geometry, crashing the plugin/device.
- Confidence: high for concrete zero cases; malformed metadata trigger.

<a id="disk-145"></a>

### DISK-145 — Crypt bounds clipping underflows beyond the end of the image

- Source: workbench/devs/diskimage/plugins/crypt.c:277–284, :318–346.
- Trigger: aligned request starting beyond image end and a successful seek beyond EOF; writable handler permits extending files.
- Evidence: read assigns total_bytes-offset to ULONG size without first rejecting offset>total_bytes. Write assigns total_blocks-block to unsigned size under the same condition; subtraction underflows and loops encrypting 512-byte chunks far beyond the caller's stated length.
- Impact: read can issue an oversized write into caller memory if bytes become available; write reads beyond caller buffer and may extend/corrupt the image for huge numbers of sectors. Write sector-address narrowing and block+size wrap introduce further capacity-check bypasses.
- Confidence: high for arithmetic and write loop; actual path depends on handler allowing seek beyond EOF. Build applicability conditional: Crypt is not in the inspected AROS plugin build definitions.

Additional outcomes:
- CPC cylinder*head count is narrowed to UBYTE tracks independently of full geometry, so geometries exceeding 255 tracks advertise more than the stored offset table represents.
- CPC ignores high io_Actual address bits in its read/write callbacks, aliasing 64-bit addresses to the low 32-bit offset when invoked through the device's 64-bit dispatch.
- Crypt CBC IV uses native ULONG byte order and only low 32 bits of byte offset, so ciphertext portability across endian architectures is not ensured and offsets repeat IVs every 4 GiB. Record as format/security limitations, not proven disk-buffer corruption.
- Crypt key and password disposal uses ordinary memset; compiler elimination of dead stores is possible but not verified from source-only review. Weak ad-hoc password derivation is outside this correctness audit's primary focus.
- Crypt uses fixed per-image 512-byte encryption buffer and write does not modify caller data; correct for in-range sector-sized requests.

<a id="disk-146"></a>

### DISK-146 — UIF allocates hash entries as eight bytes on pointer-wide builds

- Source: workbench/devs/diskimage/plugins/uif.c:150–153, :284–301.
- Trigger: UIF plugin built for a 64-bit pointer ABI and image spanning enough hash intervals (e.g. four MiB).
- Evidence: UIFHash contains a pointer plus ULONG and padding, but allocation unconditionally multiplies entry count by eight. Hash population increments by sizeof(UIFHash), which exceeds eight on this ABI.
- Impact: heap overwrite during image opening, followed by out-of-bounds hash reads during I/O.
- Confidence: high; architecture conditional.

<a id="disk-147"></a>

### DISK-147 — UIF index size multiplication can underallocate metadata

- Source: workbench/devs/diskimage/plugins/uif.c:228–230, :257–277.
- Trigger: blhr.num chosen such that sizeof(blhr_data_t)*num wraps the stored ULONG out_size; compressed payload finishes successfully within the small allocation.
- Evidence: unchecked size multiplication drives allocation and inflate output limit, but the subsequent endian-conversion loop still processes the original num records.
- Impact: heap out-of-bounds reads and writes in the decoding loop.
- Confidence: high; malformed image trigger. Distinct from CISO index representation, same unchecked-count pattern.

<a id="disk-148"></a>

### DISK-148 — UIF capacity arithmetic wraps before hash allocation

- Source: workbench/devs/diskimage/plugins/uif.c:268–284, :293–302.
- Trigger: total sectors times sector size exceeds ULONG range (ordinary images above four GiB when ULONG is 32 bits), while individual decoded block sizes fit ULONG.
- Evidence: total_bytes = total_blocks * block_size evaluates two ULONG operands before assignment to UQUAD. Hash allocation uses this wrapped total, but population accumulates individual byte extents in UQUAD.
- Impact: too-small hash allocation and heap overwrite while opening large images. The read path also sees truncated total capacity.
- Confidence: high for 32-bit ULONG configurations. Independently, per-block size multiplication and total_blocks accumulation lack overflow checks.

<a id="disk-149"></a>

### DISK-149 — UIF accepts incomplete decompression as full blocks

- Source: workbench/devs/diskimage/plugins/uif.c:248–257, :429–439.
- Trigger: valid compressed stream ends after fewer bytes than the declared metadata or data output length.
- Evidence: both inflations check only Z_STREAM_END, never actual output length / remaining avail_out. Index allocation is MEMF_ANY and decoded as num complete records; data buffer is copied as though all declared bytes were produced.
- Impact: uninitialized heap bytes become metadata or are returned as disk contents; reused data buffers expose stale contents.
- Confidence: high; malformed image trigger, related to the earlier CISO/DAX short-output observation.

Additional outcomes:
- UIF ignores decoded sector positions and reconstructs all block locations by cumulative out_size; correctness assumes a contiguous ordered index. No independent valid sparse/reordered UIF sample established in this source-only pass.
- Zero sector_size and zero output extents are not rejected; zero capacity normally stops reads, but malformed arithmetic can bypass ordinary index assumptions. Keep grouped with metadata validation rather than inventing a proven zero-divide path.
- Read advances records without an explicit data-array end check and stops only at exact cumulative total_bytes equality. Trusted metadata arithmetic is therefore essential; wrapping/incomplete metadata findings above violate that premise.
- Source also uses a module-global Z1Base set on each open, while closing per-image library references. Cross-image library lifecycle contract requires later support-library review.

<a id="disk-150"></a>

### DISK-150 — XAD missing-library failure frees uninitialized pointers

- Source: workbench/devs/diskimage/plugins/xad.c:130–146, :245–247.
- Trigger: XAD_OpenImage called with no previously opened xadmaster.library, and OpenLibrary fails (e.g. explicit plugin selection).
- Evidence: xai and patbuf are uninitialized locals; failed library open jumps to error before assigning either. Cleanup tests/frees xai and unconditionally FreeVec(patbuf).
- Impact: invalid frees or crash on a recoverable missing dependency.
- Confidence: high for the failure path; automatic CheckImage rejects the plugin on absent library, so ordinary autodetection masks it. Explicit selection exposes the opener.

<a id="disk-151"></a>

### DISK-151 — Temporary extraction overwrites a predictable existing file

- Source: workbench/devs/diskimage/device/tempfile.c:65–68, :80–85; plugins/xad.c:185–190; plugins/xpk.c:148–153.
- Trigger: configured temporary directory already contains unit_<unitnumber>.<extension>, or another device instance uses the same directory and unit number.
- Evidence: CreateTempFile chooses solely unit number and extension; OpenTempFile(MODE_NEWFILE) opens that exact name without exclusive creation, uniqueness or existence check. RemoveTempFile later deletes the same path.
- Impact: existing file is truncated and subsequently deleted; concurrent instances can replace each other's extracted backing image.
- Confidence: high for existing-file loss. Concurrent instance applicability depends on whether multiple device instances are enabled; not necessary for the primary trigger.

Additional outcomes:
- Nested archive wrappers cannot allocate another temporary file while unit.TempDir remains set: CreateTempFile returns ERROR_OBJECT_IN_USE. Explicit support limitation rather than claimed successful nested extraction.
- XAD/XPK close original and output handles before reopening the temporary image; RemoveDisk closes the final plugin before removing its temporary backing file, matching ownership on ordinary success/failure.
- Password retries do not explicitly rewind/truncate input/output handles. Whether external archive libraries do so is unresolved; do not claim corrupted retries without that contract.
- XAD frees archive object after successful GetInfo without explicit xadFreeInfo (unlike CheckImage). Potential archive metadata leak remains conditional on external library object-free semantics.
- TempDir lock can remain set when TempName allocation fails, but InsertDisk failure calls RemoveDisk and releases it; no normal insertion leak claimed.

<a id="disk-152"></a>

### DISK-152 — IPF initialization failure uses an uninitialized device pointer

- Source: workbench/devs/diskimage/plugins/ipf.c:116–137, :186–202.
- Trigger: image allocation succeeds, then CreateIORequest or OpenDevice fails.
- Evidence: image uses MEMF_ANY and only id is initialized before allocating the request. capsimagebase remains uninitialized until successful OpenDevice; error cleanup tests it and may call CAPS/device cleanup through arbitrary state.
- Impact: invalid library/device calls, CloseDevice on a failed/unopened request, or crash during recoverable initialization failure.
- Confidence: high; runtime conditional on separately building IPF.

<a id="disk-153"></a>

### DISK-153 — IPF MFM parser's end guard omits lookahead

- Source: workbench/devs/diskimage/plugins/ipf.c:286–303, :329–337, :356–361.
- Trigger: matching sync/header/sector near the last permitted candidate position of the CAPS-provided raw track, with shifted MFM decoding.
- Evidence: end reserves 540 words for the sector; decoded header/data consume 540 words, while getmfmlong reads three words for a two-word field. Last even-data read therefore looks one word beyond the reserved extent. Sync matching advances data before parsing without revalidating the full sector extent; odd byte track lengths are rounded up too.
- Impact: raw-track buffer overread on boundary input; checksum behavior can depend on adjacent bytes.
- Confidence: high for missing logical bounds. Physical overread depends on CAPS allocation padding, an external contract not established here; do not imply guaranteed crash.

Additional outcomes:
- Geometry leaves dg_SectorSize unset, but TDGeometry initializes it to 512 before calling the plugin, matching IPF's fixed sector size. Rejected as an independent bug.
- IPF advertises fixed 80 cylinders/two heads/11 sectors despite obtaining image_info; other geometries are a support limitation.
- Read balances successful CAPSLockTrack with CAPSUnlockTrack on both decoding error and success. No retained track lock found on those paths.
- Read ignores the high 64-bit offset bits like CPC; conditional address alias for explicit 64-bit out-of-range requests.
- CreateMsgPort failure is not separately checked before CreateIORequest. Exact NULL-port behavior needs Exec constructor contract; group with initialization robustness rather than assert an additional independent crash.

<a id="disk-154"></a>

### DISK-154 — DAA spanning compressed blocks seek to the wrong volume offset

- Source: workbench/devs/diskimage/plugins/daa.c:329–352, 470–493.
- Trigger: one compressed chunk crosses a split-volume boundary.
- Evidence: each continuation volume stores its compressed-data start in fn->offset. The reader advances fn and in_buf but never updates read_offs for the new volume; every seek uses data->offset from the original volume instead of the new volume's payload offset. Remaining-size calculation likewise reuses the original offset. The loop also does not confirm bytes_left reached zero before passing the original data->size to Inflate.
- Impact: valid split images fail decompression or consume incorrect compressed bytes; subtraction can underflow for shorter subsequent volumes.
- Confidence: high; conditional on separately enabling DAA and a spanning chunk.

<a id="disk-155"></a>

### DISK-155 — DAA exposes unwritten decompression bytes and excess final capacity

- Source: workbench/devs/diskimage/plugins/daa.c:126–138, 296–301, 305, 419–427, 446–499.
- Trigger: a chunk's stream reaches Z_STREAM_END before producing the output span later copied, or the final ISO chunk is shorter than chunksize.
- Evidence: capacity is computed as chunk count * chunksize, ignoring header isosize. Inflate checks only Z_STREAM_END and copies the requested span without validating actual decompressed length. The output buffer uses MEMF_ANY, so an unproduced tail can contain stale or uninitialized bytes.
- Impact: stale/uninitialized bytes can be returned as disk data, and final image size can be advertised beyond the declared ISO. Non-final full chunks with short streams and a partial final chunk are distinct triggers sharing the unchecked output-length/capacity handling.
- Confidence: high for short-output acceptance and rounded-up capacity; final-chunk manifestation depends on the format's declared isosize and output padding. Related short-output roots in DISK-149/CISO/DAX remain separately scoped.

Additional outcomes reconciled:
- The 32-bit total_bytes multiplication is promoted to DISK-332; it is separate from DISK-155's rounded-up final capacity.
- The unchecked chunk-table allocation product is promoted to DISK-333; malformed extreme size tables can underallocate the record array before its population loop.
- Optional header type/len are read without little-endian decoding. On big-endian systems multi-volume/encryption classification and skipping are incorrect; zero/undersized len is unvalidated, though the DOS seek wrapper's behavior determines whether this can loop.
- Reader never checks bytes_left after exhausting the volume chain, so inflate may consume uninitialized tail input on missing final payload.
- Signature strcmp comparisons with fixed literals stop at the literal terminator or mismatch; no unbounded header-string scan claimed.
- Failure to allocate first file node after allocating image leaks original file: close only walks image.files, which is NULL.
- Zero chunksize yields zero capacity and read's initial offset check rejects requests; no reachable divide-by-zero claim on that simple case.

<a id="disk-156"></a>

### DISK-156 — ADC back-reference decoding reads beyond compressed input and loses cursor position

- Source: workbench/devs/diskimage/plugins/dmg/adc.c:53–90; dmg/dmg.c:788–797.
- Trigger: ADC back-reference token, especially one ending exactly at compressed-input boundary.
- Evidence: short token checks one remaining byte but reads in[1] as well; long token checks two but reads in[2]. Neither branch advances in after consuming offset bytes, though src_len is decremented. Repeated parsing therefore treats offset bytes as tokens and remaining-length accounting no longer matches actual cursor.
- Impact: out-of-bounds compressed-input reads and incorrect decompressed disk contents. DMG ignores decoder return and caches output as successful.
- Confidence: high; supported DMG ADC path.

<a id="disk-157"></a>

### DISK-157 — ADC emits repeated bytes instead of copying back-references

- Source: workbench/devs/diskimage/plugins/dmg/adc.c:61–67, :79–85.
- Trigger: back-reference refers to a prior sequence containing distinct bytes.
- Evidence: implementation uses memset(out, out[-off], len), repeating only its first source byte instead of copying the referenced sequence with overlap handling.
- Impact: silently corrupted decompressed sectors for nonuniform back-references; overlaps require sequential expansion.
- Confidence: high for algorithmic distinction; retain separately from token cursor/bounds defect because either fix alone leaves wrong output.

<a id="disk-158"></a>

### DISK-158 — Empty cleaned DMG base64 data reads before its buffer

- Source: workbench/devs/diskimage/plugins/dmg/base64.c:70–87; dmg/dmg.c:631–639.
- Trigger: nonempty XML data text consisting only of whitespace or other characters removed by cleanup_base64.
- Evidence: XML handler checks original data.len>0; cleaned string length is zero. decode_base64 performs no groups then evaluates *--src before checking dlen, dereferencing one byte before source.
- Impact: heap buffer underread during image metadata parsing; potentially crashes on malformed XML.
- Confidence: high. For nonempty decoding, in-place destination fits the original encoded allocation; no output-allocation overrun claimed.

Additional outcomes:
- ADC fills missing output with zeros and returns a short decoded length; caller discards it and returns those bytes as successful sectors. Truncated streams therefore silently produce zero-filled data.
- Long back-reference permits off==0 and reads out[0] before writing it, producing uninitialized output; group with ADC decoding validation.
- Base64 ignores incomplete final groups and treats arbitrary '=' placement as zero characters. Main DMG parser's decoded-length validation must be completed before assessing downstream metadata bounds.

<a id="disk-159"></a>

### DISK-159 — DMG resource run count is trusted beyond the resource buffer

- Source: workbench/devs/diskimage/plugins/dmg/dmg.c:389–407, :506–557.
- Trigger: resource-fork mish header fits, but blocks_run_count advertises more 40-byte records than remain in the resource.
- Evidence: outer parser checks only len>=0xcc, passes 0x28*num_parts as the available length to add_to_plist without comparing against actual len. The helper decodes every advertised record.
- Impact: out-of-bounds heap reads become block extents and compression metadata; opening malformed images may crash.
- Confidence: high. Count multiplication and allocation arithmetic also lack overflow validation.

<a id="disk-160"></a>

### DISK-160 — DMG zero-sized runs can stall transfers

- Source: workbench/devs/diskimage/plugins/dmg/dmg.c:770–884.
- Trigger: request takes only part of a block because min is cast to LONG and capped at zero for sizes above LONG_MAX, or malformed zero-size COPY/compressed records.
- Evidence: normal nonzero to_read consumes min(request, remaining block) and advances part; this is correct when request finishes. However zero-size compressed/COPY branches break without advancing part and to_read remains zero; size never changes. Similarly a >LONG_MAX to_read becomes zero while part advances, skipping data.
- Impact: zero-sized records encountered during cross-block reads can hang the device worker; oversized block transfers can skip bytes and return errors/wrong data.
- Confidence: high for zero-size record nonprogress; oversized transfer conditional on huge accepted block/request sizes. Title retained for transfer progression family; ordinary smaller partial reads are correct.

Additional outcomes:
- Run in_offs and in_size decode only low 32 bits of 64-bit fields; out_size similarly uses low sector count shifted into ULONG. Backing offsets above four GiB alias, and runs >=four GiB truncate. first_sector/data_start and koly data_fork_offset are ignored; valid nonzero-base layouts require applicability verification.
- XML parser_data.data allocation is never freed after XML_ParserFree, on either success or failure; repeated opens leak decoded metadata scratch space.
- Non-AROS callbacks fetch parser_data.expatbase, but opener does not initialize it; error callbacks calling XML_StopParser may use NULL base. AROS uses module-global base; conditional legacy-library issue.
- Zlib and bzip2 return actual out_len but DMG never validates it before copying declared output. Same short-output root cause as DISK-149/DISK-155; consolidate in final report.
- Hash uses sizeof correctly (unlike UIF), but hash_size narrows to ULONG; extreme total_bytes can underallocate. No ordinary small-image hash-size defect found.
- Short decoded XML data causes add_to_plist num_parts<=0 to return without reading records; no direct metadata overread proved from a short base64 blob beyond DISK-158.
- Node cleanup and final file ownership are balanced; no persistent part-list leak found on failure after AddTail.

<a id="disk-161"></a>

### DISK-161 — DMS RLE decoder has no compressed-input bound

- Source: workbench/devs/diskimage/dms/u_rle.c:12–38; plugins/dms.c:319–343.
- Trigger: malformed RLE stream expands insufficient output before reaching the end of initialized or allocated input, with matching archive CRC.
- Evidence: decoder accepts only origsize (output size), repeatedly dereferences in with no input-end argument/check; wrapper supplies pklen1 bytes but cannot communicate that length. RLE extended run markers consume additional bytes unchecked.
- Impact: stale input bytes become decompressed sectors, and sufficiently long nonproductive streams read beyond the 32-KiB input allocation.
- Confidence: high for absent bounds; CRC is not protection against deliberately constructed malformed data. Output run bounds are checked.

<a id="disk-162"></a>

### DISK-162 — DMS ignores track numbers when placing extracted data

- Source: workbench/devs/diskimage/plugins/dms.c:254, :287–309.
- Trigger: archive begins at a nonzero track or contains missing, reordered or repeated track numbers.
- Evidence: number is used only to filter non-disk tracks and decrypt; output is always appended, without seeking number*track-size or validating a contiguous sequence.
- Impact: extracted sectors shifted/reordered relative to disk track addresses; partial archives are exposed as an incorrectly based disk.
- Confidence: high for placement behavior; applicability to accepted partial/reordered archives should remain explicit.

Additional outcomes:
- cmode 0 copies unpklen bytes regardless of pklen1; inconsistent lengths expose uninitialized tail of input allocation as decoded data. Heavy mode without RLE similarly assumes pklen2==unpklen without validating it.
- done is set before reopening the extracted file; reopening failure suppresses SetDiskImageError, returning NULL without the correct error report.
- Short output Write returns IoErr directly; if a handler short-writes with zero IoErr, extraction continues as success. Unlike plugin data reads this path does not use the nonzero default DOS2IOErr conversion.
- Reuses predictable temporary image naming: DISK-151 applies.
- Header/track CRC validation and 32-KiB declared buffer limits are present; these do not bound decoder input consumption.
- Non-TR bytes after a track header read are silently treated as normal EOF; truncated/corrupt archive completeness is not independently checked against header track range.

<a id="disk-163"></a>

### DISK-163 — DMS dictionary matches exceed the output endpoint

- Source: workbench/devs/diskimage/dms/u_quick.c:27–32; u_medium.c:29–39; u_heavy.c:55–62.
- Trigger: final decoded match length exceeds remaining requested output bytes; wrapper permits output size equal to the 32-KiB allocation.
- Evidence: outer loop checks out<outend, but match loops write all j bytes without checking remaining space.
- Impact: heap overwrite beyond the output buffer for a full-sized track; smaller tracks overwrite beyond the logical output extent.
- Confidence: high; malicious compressed stream can supply the final match. Dictionary indexing itself is masked to its allocated ranges.

<a id="disk-164"></a>

### DISK-164 — DMS Heavy tree metadata overflows fixed arrays

- Source: workbench/devs/diskimage/dms/u_heavy.c:135–179; xdms.h:41–46.
- Trigger: character tree count 511 (nine-bit field, c_len capacity 510), or position tree count >20 (five-bit field).
- Evidence: count is copied directly into a loop writing c_len[i]/pt_len[i] without validating against array capacity or active alphabet size.
- Impact: overwrites adjacent decoder state before table construction can reject the stream.
- Confidence: high. Constant-tree branches also accept invalid symbols: character 510/511 enters internal-node decoding; position >=np enters unvalidated tree walking.

Additional outcomes:
- DROPBITS in xdms.h advances input indefinitely without an input-end check; input bound omission from DISK-161 applies across QUICK/MEDIUM/HEAVY and the pending DEEP decoder.
- Heavy's zero-count constant trees accept symbols outside their alphabets; uninitialized/zero internal links can create nonterminating decode traversal, particularly with no newly supplied valid trees. Keep grouped with DISK-164 pending final root-cause consolidation.
- make_table checks node capacity, recursive depth and table codeword bounds; no separate unconditional table-construction overwrite found beyond unvalidated input bit-length arrays.
- Init_Decrunchers clears only initial 0x3fc8 dictionary bytes. MEDIUM indexes up to 0x3fff, leaving 56 bytes potentially stale on reset; malicious references to not-yet-written tail can expose previous dictionary contents. Normal compressor initialization contract not established.

<a id="disk-165"></a>

### DISK-165 — FDI header constructor publishes unchecked work-buffer allocations

- Source: workbench/devs/diskimage/plugins/fdi/fdi2raw.c:2031–2034, :2070; :2081.
- Trigger: any mfmsync/track-source/track-destination/timing allocation fails after FDI object allocation succeeds.
- Evidence: four allocations are never tested; constructor still returns the object. Track load paths use the buffers (e.g. memset track_dst_buffer) without establishing successful allocation.
- Impact: NULL dereference during first read instead of clean image-open failure.
- Confidence: high; conditional on optional FDI build and allocation failure.

Additional outcomes:
- FDI read_sector duplicates IPF's 540-word reserve/lookahead defect from DISK-153; same decoder logic, group together in final consolidation.
- Sector getter always returns 11 or 22, so wrapper's division by sector count is not a reachable zero-divide.
- last_track getter actually returns total track count, despite its name; wrapper uses it consistently, so no off-by-one finding.
- Header parser ignores Read/Seek return values; zero-initialized trailing header bytes can let short files pass initial signature/version checks and produce invented geometry. Full track reads still need review.
- Caller supplies fixed mfmbuf/tracktiming sizes but fdi2raw_loadtrack signature has no destination capacity. Revolution copy uses decoder-produced len; overall bounds require remaining decoder pass.
- Wrapper closes FDI state, both track buffers and input file on normal/error cleanup. No wrapper-owned buffer leak found.
- Wrapper only decodes Amiga MFM sectors while FDI header can describe other disk types; explicit format-support limitation.

<a id="disk-166"></a>

### DISK-166 — FDI Huffman root is uninitialized before tree expansion

- Source: workbench/devs/diskimage/plugins/fdi/fdi2raw.c:173–207, :265–286.
- Trigger: fdi_decode receives a stream whose root is encoded as a leaf.
- Evidence: automatic NODE root is never initialized; expand_tree's leaf branch frees node->left and node->right before setting them NULL. For root these are indeterminate pointers.
- Impact: invalid heap frees/crash during compressed pulse decoding.
- Confidence: high for the local path; full fdi_decode call applicability is still being traced.

Additional outcomes:
- expand_tree allocates each child then immediately memset without testing allocation, adding tree allocations to DISK-165's failure robustness family.
- Tree expansion and values/data decoding have no stream-end parameter and no recursion limit; malformed streams can overread input or exhaust stack/memory. Need trace compressed-pulse caller lengths before final grouping.
- decode_raw_track trusts a four-byte size to memcpy into track_dst with no allocation/input bounds. Its call sites and type handling remain pending.
- bit_add has a post-write out limit and resets out=1 on overflow; subsequent calls stay inside the allocation if out starts valid. Do not assert an unconditional bit_add overflow without tracing initial out.
- add_mfm_sync_bit checks its counter after increment and resets at capacity; first write at final valid index is permitted. No direct off-by-one finding there.
- FDI header constructor writes track_offsets[i] after its loop with i==last_track, while array is MAX_TRACKS and count clamps to MAX_TRACKS. This writes one ULONG beyond the array at maximum geometry; adjacent file field can be corrupted. Promote after checking layout/build type definitions.

<a id="disk-167"></a>

### DISK-167 — FDI maximum track count writes past its offset table

- Source: workbench/devs/diskimage/plugins/fdi/fdi2raw.c:109, :139, :2038–2039, :2057–2068.
- Trigger: header geometry yields >=166 tracks, clamped to MAX_TRACKS==166.
- Evidence: array track_offsets[MAX_TRACKS] holds 166 entries, but constructor always stores terminal offset at index last_track after filling indices 0..last_track-1. Index 166 is outside allocation's array and precedes file field.
- Impact: adjacent state corruption (on layouts without intervening padding, overwrites file handle), then later track loading uses invalid state and reads past offset array for the last track.
- Confidence: high for array overrun; exact adjacent field effect depends on ABI padding.

Additional outcomes:
- s08/s09 and s0a..s0d consume token operands and encoded byte spans without local source-end checks. Main dispatch has a source limit check at line 1324; its ordering and operand handling remain pending.
- Amiga integrity helper uses potentially unsafe doubled track length/rotation bounds, but its only located call is commented out. Do not classify that helper as an active runtime finding.
- Amiga sector encoder stack arrays fit their fixed loops (512-byte data and 16-byte labels); no independent stack overwrite found in those fixed-size emitters.
- FDI's low-level bit output continues to use centralized bit_add limits; further direct-copy/pulse paths still need review.

<a id="disk-168"></a>

### DISK-168 — FDI checks token input bounds only after consuming operands

- Source: workbench/devs/diskimage/plugins/fdi/fdi2raw.c:872–934, :1093–1140, :1303–1328; s08/s09/s0a..s0d earlier.
- Trigger: sector-described track ends inside a fixed-size sector token, or carries a length exponent exceeding available track input.
- Evidence: dispatch reads token, invokes decoder and only afterward compares cursor with track_src_len. Operand routines copy/read 512-byte sectors and 128<<shift spans without checking input extent; exponent is unvalidated.
- Impact: stale/out-of-bounds input reads before error reporting; excessive exponent can request huge reads/loops or invoke undefined shifts. Central output bit limits do not bound input reads.
- Confidence: high; malformed optional FDI image trigger.

Additional outcomes:
- Sector dispatch rejects cursor==track_src_len even after processing the end marker, so a marker exactly ending the declared track is treated as overrun. Padding requirements need format confirmation before labeling a normal-image rejection.
- Normal PC/ST generators pass current_track as sector number for every sector instead of i+1; generated headers repeat the wrong number. FDI wrapper only reads Amiga sectors, so these paths are format limitations within this device, though decoder output is incorrect for a general PC/ST consumer.
- fdi_decompress mode 0 tests pulses*2 against len but consumes pulses*4 bytes; raw pulse stream byte bounds remain unvalidated.
- fdi_decompress mode 1 allocation is unchecked before fdi_decode memset; related to DISK-165. Allocation size pulses*4 can wrap and requires caller pulse-count tracing.
- Module globals temp/temp2 and IBM CRC are shared across per-unit readers. Concurrent FDI decoder use can race on Huffman bit state; serialized plugin caller contract remains to be checked.

<a id="disk-169"></a>

### DISK-169 — FDI pulse decoder has no output or timing capacity checks

- Source: workbench/devs/diskimage/plugins/fdi/fdi2raw.c:1377–1383, :1754–1764.
- Trigger: decoded pulse widths produce more than 40000 raw bytes or 200000 timing entries (the internal allocations), before completing a revolution.
- Evidence: addbit increments module-global bitoffset and writes p[bitoffset/8] without a bound; active fdi2_decode also increments timing pointer per produced bit without checking MAX_TIMING_BUFFER.
- Impact: heap overwrite from malformed/extreme pulse metadata. Unlike sector-described bit_add, this separate pulse writer has no overflow guard.
- Confidence: high for unbounded writes; exact accepted metadata path still requires remaining pulse loader review.

Additional outcomes:
- Active decoder can enter a nonprogress loop when avg_size is zero (totalavg too small) or one (avg_size/2 truncates zero). While adjusted_pulse>=avg_size then subtracts zero. Caller timing validation remains pending.
- Pulse helper state bitoffset, psarray, total, totaldiv and array_index is module-global; concurrent readers can corrupt calculations and output positions. Per-unit worker serialization does not itself serialize across units; final caller review will determine applicability.
- Disabled older implementation has similar risks but is not counted as a separate active finding.

<a id="disk-170"></a>

### DISK-170 — FDI raw-track length overflows the internal destination

- Source: workbench/devs/diskimage/plugins/fdi/fdi2raw.c:313–319, :2146–2148.
- Trigger: raw-track type 0xf0..0xff with declared bit count requiring >40000 output bytes.
- Evidence: decode_raw_track reads size from input and memcpy copies (size+7)>>3 bytes to fixed MAX_DST_BUFFER allocation without validating size or compressed input extent.
- Impact: heap overwrite / input overread during optional FDI read.
- Confidence: high. Raw path also leaves fdi.out zero so published tracklength is zero, rejecting otherwise decoded raw tracks; copy includes size prefix rather than explicitly skipping it.

<a id="disk-171"></a>

### DISK-171 — FDI normal-track dispatch omits its final array entry

- Source: workbench/devs/diskimage/plugins/fdi/fdi2raw.c:1259–1268, :2160–2164.
- Trigger: track type 0x0f.
- Evidence: decode_normal_track has entries 0..9 plus five zxx entries (10..14), but dispatch permits every type<0x10 and indexes 15.
- Impact: out-of-bounds function pointer read and call.
- Confidence: high; malformed optional FDI image trigger.

<a id="disk-172"></a>

### DISK-172 — FDI low pulse timings can hang decoding

- Source: workbench/devs/diskimage/plugins/fdi/fdi2raw.c:1819–1821, :1912–1932, active fdi2_decode at :1559 onward.
- Trigger: positive pulse count and stable pulses with totalavg below 12500, giving standard_MFM_8_bit_cell_size zero.
- Evidence: loader only rejects zero pulse count; it performs no lower bound on cumulative timings. Decoder can clamp avg_size to zero, then its adjusted_pulse>=avg_size loop subtracts avg_size/2==0 forever.
- Impact: device worker hangs on malformed pulse timing.
- Confidence: high; caller tracing confirms no timing validation masks the path.

Additional outcomes:
- DISK-166 is reachable through low-level compressed streams: fdi_decompress mode 1 calls fdi_decode directly. No root initialization in intervening caller.
- DISK-169 remains applicable: pulse count and cumulative output have no capacity bounds. Final revolution copy likewise lacks caller mfmbuf capacity and celltiming writes one UWORD per output byte plus two; caller tracktiming allocation is only mfmbuf_size bytes, so even internally bounded large tracks can overflow caller timing buffer.
- Pulse counts drive unchecked pulses*4/pulses*2 allocations; mode0 reads/writes pulses*4 bytes without comparing stream length, and mode1 Huffman stream has no end/depth bound. Group with input-size/allocation findings during final consolidation.
- minp/maxp/idxp failure/unsupported compression modes are dereferenced without checks; DISK-165 extends to these pulse arrays.
- Input Read/Seek results remain unchecked before decoding, permitting stale track-source contents after short reads.
- Global decoder state is shared across units; device workers are per-unit (existing unit review). No decoder-specific lock found in wrapper, making concurrent optional FDI readers a source-level race candidate.
- The track offset terminal-array defect DISK-167 also affects track+1 lookup at maximum geometry.
- Failed intermediate pulse decompressions return without releasing earlier allocated arrays, which have not yet been installed in cache; leak on malformed input.

<a id="disk-173"></a>

### DISK-173 — Disk-image compatibility allocators free the original allocation when realloc fails

- Source: workbench/devs/diskimage/plugins/cue/malloc.c:67–83.
- Trigger: audio codec asks realloc to grow an existing allocation and new allocation fails.
- Evidence: failure branch calls free(old) before returning NULL, violating realloc's preservation of the original allocation on allocation failure.
- Impact: callers retaining/freeing the original pointer on failure can use freed memory or double-free; external codec caller applicability still needs contract tracing.
- Confidence: high for adapter contract violation; practical codec failure behavior conditional on its caller.
- Additional affected implementation: `workbench/devs/diskimage/expat_library/malloc.c` contains the same free(old)-on-failure branch. The bundled parser installs this realloc as its default callback (`xmlparse.c:726`). On leftover-buffer growth (`xmlparse.c:1512–1521`), a failed realloc returns XML_ERROR_NO_MEMORY while preserving the now-freed parser buffer pointer; XML_ParserFree later calls FREE(buffer) (`xmlparse.c:1137`). This establishes a concrete double-free path for standalone bundled Expat under allocation failure, stronger than the still-conditional external CUE codec trace. Modern workbench-libs-expat uses separately fetched sources and is not claimed affected by this local wrapper.

Additional outcomes:
- malloc adds sizeof(size_t) without overflow check; calloc multiplies count*size unchecked. Extreme codec allocation requests can underallocate; codec size constraints need tracing.
- malloc returns allocation plus one size_t, potentially reducing alignment below max_align_t requirements; ABI-specific codec alignment applicability pending.
- Pool operations are semaphore-protected, including free/realloc's separate allocation/free operations; no independent concurrent pool list corruption found.
- File/stdio/select adapters are stubs returning failure. Audio paths must use callback interfaces; callback decoder support is still being reviewed.
- Plugin build CFILES reference audio/aiff etc under plugins directory, but actual files reside ../audio. No plugins/audio directory was found. Verify build macro relative path handling before promoting a build integration candidate.
- String duplication handles NULL and allocation failure; case comparisons delegate to Utility library initialized by CUE.

<a id="disk-174"></a>

### DISK-174 — WAV/AIFF end-of-stream clipping can overwrite the caller buffer

- Source: workbench/devs/diskimage/audio/wave.c:188–203; audio/aiff.c:277–293.
- Trigger: read starts inside the declared stream but requested length crosses its end, with declared stream.length greater than caller length and additional file bytes available.
- Evidence: clipping assigns read_length=stream.length rather than stream.length-offset. FRead may then write more than requested length. If bytes_read>length, subsequent length-bytes_read underflows and memset writes a huge range.
- Impact: caller-buffer corruption and potentially unbounded zero fill; both audio wrappers affected.
- Confidence: high. Trailing chunks or inconsistent metadata can supply bytes beyond the logical stream.

<a id="disk-175"></a>

### DISK-175 — AIFF compression identifier is compared without endian conversion

- Source: workbench/devs/diskimage/audio/aiff.c:211.
- Trigger: 22-byte COMM chunk specifies NONE compression on little-endian system.
- Evidence: comm.compression is read directly as host ULONG and compared to MAKE_ID; surrounding fields use rbe16/rbe32.
- Impact: rejects otherwise supported uncompressed AIFF-C input.
- Confidence: high; endian conditional, plus this parser's restricted COMM lengths limit broader AIFF-C support.

Additional outcomes:
- WAV accepts data before any fmt chunk; AIFF accepts SSND before any COMM chunk. Neither requires validated format before returning stream; non-CD-format data may therefore be treated as stereo 44.1-kHz 16-bit audio.
- AIFF fallback length includes SSND's eight-byte offset/blocksize header instead of subtracting it; dataoffset is not checked against chunk size, permitting unsigned underflow. Explicit malformed/COMM-missing path.
- Both silently zero-fill short file reads and report requested length, masking truncated backing audio as successful silent data.
- File-position addition and offset+length arithmetic use ULONG without overflow checks.
- Known chunks are bounded to their local stack structures; no stack overwrite from advertised fmt/COMM size found.

<a id="disk-176"></a>

### DISK-176 — FLAC stream cleanup leaks PCM buffers

- Source: workbench/devs/diskimage/audio/flac.c:132–148, :313–332.
- Trigger: any decoded frame allocates pcm[0]/pcm[1], followed by closing the stream.
- Evidence: write callback allocates both planes with AllocVec; close frees decoder, cuesheet, file and stream, but never these PCM allocations.
- Impact: persistent heap leak after every played/read FLAC image.
- Confidence: high; optional CUE audio path.

Additional outcomes:
- Cached sample copying uses stereo output and wle16 correctly for the accepted 44.1-kHz/16-bit/stereo metadata.
- Allocation failure frees old PCM planes and leaves samples_in_pcm unchanged; later cache checks can regard NULL planes as populated if a decoder error is retried. Caller error/retry behavior needs tracing before promoting.
- Decoder errors are ignored by empty error_callback; process_single may resynchronize and output later samples. Accuracy/completeness contract needs codec API/caller tracing.
- PCM byte counts return pointer casts to LONG instead of pointer subtraction. Architecture-dependent truncation/wrap may produce incorrect counts across address boundaries; consolidate with wider-address adapter review.
- FLAC embedded cuesheet adds index-1 offset to the potentially already modified index0, instead of immutable track base. When index0 is nonzero-relative, index1 can be shifted twice; valid FLAC cuesheet constraints must be checked before claiming a supported-image trigger.
- FLAC stream close owns cuesheet clone; embedded wrapper routes final close through CUE, so no independent cloned-cuesheet leak established.

<a id="disk-177"></a>

### DISK-177 — mpg123 writes size_t output count into a LONG local

- Source: workbench/devs/diskimage/audio/mp3_mpg123.c:113, :138, :161; include/mpg123.h mpg123_read declaration.
- Trigger: build with size_t wider than AROS LONG (64-bit pointer ABI).
- Evidence: bytes_read is LONG, but mpg123_read's final argument is size_t*. Each decoder call writes the wider output count through &bytes_read.
- Impact: overwrites adjacent stack state during MP3 reading or seek-discard.
- Confidence: high; architecture conditional. Build integration applicability remains to verify for optional CUE plugin.

Additional outcomes:
- Vorbis installs DOS-backed callbacks with no close callback and explicitly closes file after ov_clear; avoids double file close on normal success.
- Vorbis initial format check applies to initial logical bitstream only. Chained streams with differing rate/channels are not revalidated on stream.bs changes; output length and CD-audio interpretation can be incorrect.
- Vorbis/MP3 total sample counts narrow to LONG and byte lengths to ULONG; large audio files truncate geometry.
- MP3 callback rejects positions above signed 32-bit max, an explicit large-file limit despite wider helper position.
- MP3 seek-discard/read loops assume codec progress on OK; no concrete codec return sequence proving a hang established.
- Both codec wrappers zero-fill EOF intentionally; decode errors produce short reads. Caller handling remains part of main CUE review.
- MP3 uses IPTR pointer subtraction for result (unlike FLAC/Vorbis LONG casts); no analogous pointer-width result defect.

<a id="disk-178"></a>

### DISK-178 — WavPack malformed binary cuesheet starts parsing past allocation

- Source: workbench/devs/diskimage/plugins/cue/wavpack.c:135–145.
- Trigger: binary cuesheet tag contains no NUL filename separator within cue_len bytes.
- Evidence: buffer allocates cue_len+1 and appends NUL at cue_len; strlen then returns cue_len, so cuesheet_start is allocation end plus zero (one-past pointer), which strtok_r dereferences.
- Impact: out-of-bounds heap read while parsing malformed embedded metadata.
- Confidence: high; external library tag validation could constrain payload but no local separator validation exists.

Additional outcomes:
- Text tag retrieval supplies cue_len rather than cue_len+1 to an API that may reserve NUL space; possible last-character truncation remains conditional on external WavpackGetTagItem contract.
- Correction filename is derived from ExamineFH basename and opened relative to current directory, rather than the source file's parent. Absolute/subdirectory input can miss its .wvc or pick unrelated same-name correction file; cue file-directory handling must be traced for normal-path applicability.
- PCM allocation matches 588 stereo int32 samples, and unpack request is capped to that size; no local PCM overrun found under codec count contract.
- PCM/file/context cleanup balances allocations and handles, unlike FLAC's leaked planes.
- Failed seek drops context; reopen does not explicitly rewind backing handles or revalidate format. External reader initialization contract determines correctness.
- Read always zero-fills short sample output and returns full requested length, masking decode truncation as silence.
- Shared strtok_r advances state correctly for empty strings/normal separators; binary separator defect occurs before its valid-string precondition.

<a id="disk-179"></a>

### DISK-179 — CUE repeats global codec initialization for each plugin descriptor

- Source: workbench/devs/diskimage/plugins/cue/cue.c:74–87, :134–157; cue/flac.c and cue/wavpack.c plugin_Init entries; device/plugins.c:174–195.
- Trigger: optional CUE build enables FLAC/WavPack (as its build CPPFLAGS do).
- Evidence: table exports three descriptors, each with CUE_Init. Loader invokes Init per descriptor; each malloc_init overwrites global pool/semaphore pointers without freeing prior instances. Only CUE descriptor has CUE_Exit.
- Impact: leaks two pools/semaphores and repeats mpg123 global initialization; failure on a later initialization can leave earlier descriptors registered against freed/invalid allocator state.
- Confidence: high for repeated allocator ownership leak; codec global failure consequences depend on init return sequence.

Additional outcomes:
- cue_fix_track_indexes starts each new backing file at offset zero and only adds index1-index0. A first track with INDEX 01 nonzero and no INDEX 00 therefore reads from byte zero instead of indexed start; partial-file layout semantics need explicit valid CUE trigger before promotion.
- Bounds validation occurs after subtracting offsets from backing length. Offset beyond file length can underflow derived sector count; declared track spans are not checked against backing extent.
- CUE parent-directory handling makes relative referenced audio files work; absolute/subdirectory references can still trigger WavPack correction basename lookup issue.
- cue_parse_args respects MAX_ARGS and detects unmatched quotes; no argv overwrite found. isspace/isdigit use signed char without cast for non-ASCII input, a libc precondition issue.
- INDEX time parser validates seconds<60 and frame<75; fixed two-digit minute format is explicit limitation.
- Successful CUE close frees tracks/files/audio streams and scratch output; audio-specific PCM leak remains DISK-176.

<a id="disk-180"></a>

### DISK-180 — CUE first-track nonzero index is ignored

- Source: workbench/devs/diskimage/plugins/cue/cue.c:576–609.
- Trigger: backing file's first track has INDEX 01 at a nonzero file-relative time and no INDEX 00.
- Evidence: index0 is defaulted to index1, file offset starts zero, and only index1-index0 is added. Thus track starts at byte zero regardless of INDEX 01 value.
- Impact: exposes wrong sectors/audio from the beginning of the backing file instead of declared track start; derived final track capacity includes discarded prefix.
- Confidence: high for mapping; applies to nonzero-first-index layouts, not usual INDEX 01 00:00:00 images.

Additional outcomes:
- CUE audio callbacks take ULONG offsets while tracks/read_pos use UQUAD; backing audio positions above four GiB narrow at AUDIO_read. Shared audio length limit also truncates capacity.
- DISK-174 is reachable at rounded final audio sector: cue_fix_track_indexes rounds stream length up to sector size and CDDA requests complete 2352-byte frames, so final padded frame can invoke WAV/AIFF clipping path.
- Validated MODE1/MODE2 raw sectors leave sufficient bytes after sync_size for 2048-byte CopyMem. No local raw extraction overread found.
- Cross-track reads advance list only while request remains; linked-list end returns short/error. No arbitrary count-based list dereference found in these callbacks.
- num_tracks is UBYTE and wraps above 255 tracks, while linked list retains all entries. TOC consumers still pending.
- DATA reads on MOTOROLA files do not byte-swap, while CDDA does; unusual data-track MOTOROLA support semantics remain uncertain.
- Missing/zero-length tracks are skipped in initial sector lookup; no CUE positive-capacity empty-list NULL dereference because opener rejects no tracks.
- Optional CUE build/source path issue remains to verify, along with alternative MPEGA backend.

<a id="disk-181"></a>

### DISK-181 — Optional CUE build references nonexistent audio source paths

- Source: workbench/devs/diskimage/plugins/mmakefile.src:82–88; config/make.tmpl:635–674.
- Trigger: explicitly building workbench-devs-diskimage-cue from this baseline.
- Evidence: CFILES names audio/aiff, audio/flac, audio/mp3_mpg123, audio/vorbis, audio/wave, audio/wavpack relative to plugins. Actual files are sibling ../audio. Compile macro constructs source search directories SRCDIR/CURDIR/<supplied dir>; no sibling correction is provided.
- Impact: optional CUE target cannot compile its audio sources using the checked build rule.
- Confidence: high from build definitions, no build executed. Default dependencies exclude CUE; runtime CUE/audio findings are conditional on correcting/building this target separately.

Support initial pass:
- support/reallocbuf.c read fully: deliberately discards old scratch buffer on growth and resets size to zero on failure. Unlike standard realloc adapter, caller contract is scratch replacement; do not group its intended discard with DISK-173.
- support/istext.c, checklib.c and checkbptr.c bodies inspected. IsText expects nonnegative length; plugin loader supplies read lengths. CheckLib requires non-NULL library and reviewed plugin callers check it before use.
- swab2.c combined output was truncated; reread needed before full coverage claim.
- Remaining support helpers and audio header completeness check remain pending.

<a id="disk-182"></a>

### DISK-182 — Empty disk-image SCSI command is dereferenced before validation

- Source: workbench/devs/diskimage/device/scsicmd.c, DoSCSICmd entry and switch(cmd[0]).
- Trigger: HD_SCSICMD request containing a SCSICmd-sized structure but scsi_CmdLength=0 and an inaccessible/NULL command pointer.
- Evidence: outer io_Length is checked; command pointer and length are copied, then cmd[0] is read before any command-specific length check.
- Impact: invalid request can read outside its command buffer or fault the device worker rather than returning a length error.
- Confidence: high from source. Caller pointers generally require valid memory, but a zero-length command should not require a byte to exist. This extends the command-validation audit independently of DISK-042's CDDA output overrun.

<a id="disk-183"></a>

### DISK-183 — Filtered disk-image TOC changes the reported last track

- Source: workbench/devs/diskimage/device/scsicmd.c, SCSICMD_READ_TOC selection loop and header writes.
- Trigger: multi-track image and READ TOC requesting a starting track after the first, or lead-out track 0xaa.
- Evidence: skipping tracks decrements num_tracks; header's last-track byte is then written from this remaining count. For three tracks starting at track 2 it reports last track 2 while still emitting track 3. Lead-out selection reports last track zero.
- Impact: internally inconsistent TOC can confuse consumers and hide later tracks. Linked-list traversal itself remains NULL-checked; this is a metadata error, not an unchecked count-based dereference.
- Confidence: high for the internal inconsistency; expected protocol interpretation is source-review inference, not exercised against clients.

Additional loader/dispatcher outcomes:
- BytePack scalar/text writes clip against caller data length; no ordinary INQUIRY/TOC overwrite established. READ CD bypasses this clipping, covered by DISK-042.
- INQUIRY and READ TOC ignore CDB allocation lengths and use only scsi_Length; EVPD INQUIRY returns a two-byte response for every page. Protocol compatibility candidates need final classification; no separate memory overrun because BytePack clips to host buffer size.
- READ CAPACITY subtracts one from geometry total sectors; zero-capacity geometry wraps to 0xffffffff, conditional on a plugin exposing a zero-sized image.
- Plugin loader's .dbg continue is inside a do/while whose condition advances ead; it does not hang on debug files.
- Loader tracks successful plugin references and unloads a shared segment only after last plugin exits. Per-plugin repeated global initialization (CUE) remains DISK-179.
- FindPlugin bounds header/footer read allocations by file length and rewinds before callbacks. It does not rewind between candidate CheckImage callbacks; reviewed format checks normally inspect supplied headers/names, so no independent currently reachable position corruption established here.
- Untrusted dynamically executable plugins are outside malformed-image-only assumptions; TypeOfMem is not a complete plugin-table bounds validator, but no new image-triggered finding claimed.

<a id="disk-184"></a>

### DISK-184 — Unit control retains a caller error pointer beyond the request

- Source: workbench/devs/diskimage/device/unit.c, DICMD_TAGLIST and SetDiskImageErrorA; device/main_vectors.c, MountImage.
- Trigger: call MountImage (which supplies a stack-local error address), return, then issue UnitControlA with a failing Filename/Plugin operation and no DITAG_Error.
- Evidence: ErrorPtr is assigned only when its tag is present and never reset at taglist entry or exit. SetDiskImageErrorA writes through it on the later error. MountImage's stack variable has already ceased to exist.
- Impact: stale-pointer write corrupts an earlier caller's reused stack or other freed error storage.
- Confidence: high. Sequential calls suffice; no concurrency required.

<a id="disk-185"></a>

### DISK-185 — Change-notification allocation failure never completes its request

- Source: workbench/devs/diskimage/device/unit.c, TD_ADDCHANGEINT; device/io.c, LibAbortIO.
- Trigger: AllocMem for ChangeInt fails while processing TD_ADDCHANGEINT.
- Evidence: error is set to TDERR_NoMem but iotd is set NULL in both allocation-success and failure branches, suppressing ReplyMsg. Request has already been removed from IOPort; AbortIO searches only that queue and cannot find it.
- Impact: caller waiting for failed registration can wait forever. Successful registration also retains no request identity for eventual completion; notification-removal contract needs final classification separately.
- Confidence: high for allocation-failure hang.

<a id="disk-186"></a>

### DISK-186 — Reused error-string capacity can overwrite a later smaller buffer

- Source: workbench/devs/diskimage/device/unit.c, DITAG_ErrorString and DITAG_ErrorStringLength.
- Trigger: one UnitControlA supplies a large ErrorStringLength, then a later call supplies a smaller ErrorString buffer before its new length tag, with an intervening failing operation (or omits length).
- Evidence: only ErrorString pointer is reset at entry; ErrorStringLength persists. New pointer is installed using the old capacity, and SetDiskImageErrorA formats using that capacity before encountering the replacement length tag.
- Impact: later caller buffer can be overwritten beyond its actual capacity.
- Confidence: high conditional on tag ordering/omitted length; final API documentation check should establish whether length-first ordering is explicitly required.

Additional lifecycle outcomes:
- InitUnit short-circuit allocation cleanup passes addresses of leading Message members to DeleteMsg/FreeVec. NULL containing pointers have a member at offset zero; no independent actual nonzero-address dereference established despite undesirable C expression.
- FreeUnit frees the ChangeInts list allocation only, never its individually allocated handlers. Registered handlers leak on last unit close. Request-completion semantics remain an integration check.
- GetImageName checks ti_Data after ASPrintf instead of checking the allocated result; allocation failure returns NULL without setting the error output. This is an observable error-reporting defect; promote during final dedup if not covered elsewhere.
- InsertDisk ignores NameFromFH failure and may persist uninitialized/nonterminated fullpath stack storage when a path cannot fit the 512-byte buffer. Exact DOS failure-output behavior remains to trace before claiming an overread.
- Worker acknowledges startup before loading plugins/preferences. Later control messages queue until initialization completes; this alone is not a startup-port publication race.
- Last close waits for DeathMessage before freeing unit resources; avoids the unjoined hostdisk worker pattern. DeathMessage reply occurs immediately before returning from worker under Forbid; SMP/lifetime assumptions need architecture review.
- Cleanup drops ports without draining requests, conditional on closing with outstanding requests. Ordinary caller obligation to complete IO before CloseDevice prevents claiming that misuse as a standalone defect.
- ReloadPlugins unmounts via synchronous unit message before exclusive plugin lock. Disk-change/reload callbacks execute under semaphores; reentrant callbacks that call control APIs can deadlock, but actual callback inventory remains pending.
- CMD_UPDATE remains immediate no-op (diskimage addition to cached-write/ordering observations); no disk-image plugin flush interface is offered.

<a id="disk-187"></a>

### DISK-187 — MountImage caller register definition disagrees with its implementation

- Source: workbench/devs/diskimage/device/diskimage.conf functionlist; device/main_vectors.c MountImage; tools/genmodule/config.c:1987–2059, writeincdefines.c register emission.
- Trigger: generated MountImage client call on an ABI where A0 and D0 are physically meaningful (not native ABIs that ignore register labels).
- Evidence: configuration pairs unit_num with A0 and filename with D0, while implementation pairs unit_num with D0 and filename with A0. Generator associates each argument with the corresponding listed register in order.
- Impact: callee sees filename address as unit number and unit number as filename pointer; mounting fails or dereferences an invalid pointer if an unintended unit matches.
- Confidence: high for mismatch; architecture/build applicability conditional. Current AROS device rule uses this configuration. Native x86 calling conventions may mask the error.

<a id="disk-188"></a>

### DISK-188 — Disk-image preferences emit unescaped filenames into XML

- Source: workbench/devs/diskimage/prefs/writeprefs.c WritePrefsObject; device/unit.c DITAG_Filename preference saving; prefs/readprefs.c ReadPrefs.
- Trigger: mount an image whose full path contains an ampersand or less-than character, then reopen the unit after preference persistence.
- Evidence: string/key values are interpolated directly into XML text with FPrintf; no entity escaping. Reader uses Expat and rejects malformed XML. Both ENV and ENVARC persistence use the same writer.
- Impact: valid image paths produce unreadable preference files and automatic remount fails; XML-like names can alter preference structure instead of round-tripping literally.
- Confidence: high. Quotes alone in element text are harmless; ampersand/less-than establish the defect.

Additional outcomes:
- Preference writer reports TRUE after opening even if FPrintf/Close fail, and truncates previous preference file before successful replacement. Persistence error handling is deficient; classify alongside DISK-188 during final report consolidation.
- Reader bounds character accumulation to fixed 64-byte key/512-byte value arrays but silently truncates. A well-formed long path can remount a different/truncated filename. Dictionary allocation/ownership cleanup is locally balanced for constructor failures.
- Preference dictionaries recursively free children; XML nesting has no application-level depth limit, making deeply nested preferences a potential recursion/stack exhaustion trigger. Device defaults use shallow dictionaries.
- Password helper returns a newly allocated empty string when no supplied password exists on AROS. Progress UI is stubbed out on AROS; cancellation via that UI is unavailable by design. Optional archive retry behavior still needs final cross-check.
- Locale lookup walks catalog records until matching ID without an end bound. Device/plugin error IDs need caller inventory before promoting any missing-ID trigger.
- ErrorStringLength ordering is not documented in the inspected public header. In-tree clients normally place pointer then length consecutively before operations, which avoids DISK-186's intervening-operation trigger. Candidate remains conditional on custom tag ordering/omission, and should be clearly distinguished from DISK-184's ordinary stale stack pointer.
- Legacy m68k SNPrintf helper writes a terminator even for supplied capacity zero; current disk callers use nonzero fixed buffers. AllocVecPooled adds four bytes without overflow check, with no reachable enormous local allocation established.
- Legacy x86 device stub uses movl -1,%eax (absolute memory load) instead of an immediate return value. Current AROS device CFILES excludes these entry stubs, so record as inactive-source defect rather than active driver failure.

<a id="disk-189"></a>

### DISK-189 — SD change-notification list is never initialized

- Source: rom/devs/sdcard/sdcard_bus.c RegisterUnit; sdcard_unit.h sdcu_SoftList; sdcard_device.c cmd_AddChangeInt/cmd_TestChanged.
- Trigger: register a TD_ADDCHANGEINT notification on a detected SD card.
- Evidence: unit comes from the MEMF_CLEAR pool; no NEWLIST/InitList initializes sdcu_SoftList anywhere in SD sources. cmd_AddChangeInt calls AddHead on the all-zero list, whose head/tail sentinel pointers are invalid.
- Impact: NULL-pointer list manipulation/crash during notification registration. cmd_TestChanged also traverses the same uninitialized list when invoked.
- Confidence: high from full constructor/request trace.

<a id="disk-190"></a>

### DISK-190 — SD reinsertion writes past the single-slot unit allocation

- Source: rom/devs/sdcard/sdcard_bus.c RegisterUnit and empty ReleaseUnit; arch/arm-native/soc/broadcom/2708/sdcard/sdcard_sdhost_init.c:292–297 (UnitMax=1 and sizeof(BusUnits) allocation).
- Trigger: successful initial card detection followed by a removal/reinsertion detected by the bus.
- Evidence: registration increments UnitCnt and indexes the embedded first unit pointer. It rejects index only when index>UnitMax, accepting index1 for a one-pointer allocation. ReleaseUnit does not free/reset the old unit or decrement the count.
- Impact: second registration writes beyond BusUnits allocation, leaks old state and publishes invalid unit counts; further insertions advance count even on rejection.
- Confidence: high conditional on controller delivering reinsertion events; SDHCI media interrupt path and platform event applicability remain full-review work. Single-card initial registration uses index0 correctly.

<a id="disk-191"></a>

### DISK-191 — SD Open uses global unit number as a per-bus array index

- Source: rom/devs/sdcard/sdcard_init.c Open; sdcard_bus.c RegisterUnit; platform init UnitBase assignment.
- Trigger: open unit on a bus with nonzero UnitBase, or attempt a lower-numbered unit while iterating a later bus.
- Evidence: registration stores at local index and then adds UnitBase to public unit number. Open checks only unitnum<base+count, omits lower bound and indexes the per-bus pointer array using unadjusted unitnum.
- Impact: reads outside per-bus allocation or opens wrong/corrupt pointer on configurations with multiple registered buses.
- Confidence: high for indexing defect; currently selected Raspberry Pi controllers may provide only one active bus, so multi-bus applicability must remain conditional.

Additional outcomes:
- SD NSCMD_DEVICEQUERY writes the result without NULL/length checks; add as another manifestation of DISK-102's request-length validation root cause during final consolidation.
- Read32 compares against low 32 bits of capacity whereas Read64 uses full value. At exactly 2^32 sectors, Read32 rejects every positive transfer; CSD v2 path can represent this capacity because multiplication into UQUAD occurs after cylinder calculation. Extreme-capacity candidate pending complete capacity/platform audit.
- 32-bit byte offsets/lengths are shifted before block+count; for ordinary 512-byte sectors that sum cannot overflow ULONG. Do not claim the usual unchecked-byte-addition wrap here.
- Read/write chunk loops advance by at most 128 sectors and credit completed chunks only after acknowledged stop. Non-high-capacity commands switch arguments to byte addressing; normal SDSC bounds keep them within 32-bit command address range.
- ReadSector64/WriteSector64 put a UQUAD LBA into command tag, eventually limited by 32-bit SD command argument; ordinary supported CSD v2 capacity does not exceed 2^32 sectors.
- Open acknowledges success even when StartUnit fails; records an open count on an unconfigured card. StartUnit also ignores WaitStatus, capability and block-length failures. Promote/group after complete transport trace.
- StartUnit tests AB_Card_HighSpeed52 (bit number 27) instead of AF_Card_HighSpeed52 (mask), selecting the wrong clock. This is separate from DISK-020's inverted switch-result test but frequency path reachability must be noted.
- Common Open can issue controller commands directly from caller task while bus task handles IO; no bus transaction semaphore is taken. Listener overwrites/concurrent transfer risk pending full command implementation trace.
- Boot wait decrements pending counter before media registration in task loop; five-second wait therefore does not prove boot node discovery completed. Post-IRQ init/event sequencing remains to inspect.
- Media change callback dispatch is only cmd_TestChanged (private HD_SCSICMD+1 or failed Eject), not visibly called from media task branch. IRQ and platform review needed before final notification-loss finding.
- AbortIO is a no-op; CMD_FLUSH drains waiting requests. Flush durability/cache behavior remains distinct from queue cancellation.

<a id="disk-192"></a>

### DISK-192 — SDHCI interrupt-mode completion wait has no timeout

- Source: rom/devs/sdcard/sdcard_bus.c WaitCmd.
- Trigger: bus task has previously received any controller IRQ (AF_Bus_IRQSeen), then a later command produces no completion/error signal, for example lost controller interrupt.
- Evidence: Wait(1L<<CommandSig) executes before the timed poll loop, with no timer request or bounded wait. The supplied timeout is checked only after that Wait returns.
- Impact: bus task and every queued request can hang indefinitely despite configured command timeout.
- Configuration/confidence: SDHCI backend (EMMC2/legacy), high. SDHOST uses a separate synchronous WaitCmd stub and is not affected by this path.

<a id="disk-193"></a>

### DISK-193 — SDHCI software timeout relinquishes DMA storage without stopping DMA

- Source: rom/devs/sdcard/sdcard_bus.c WaitCmd timeout branch and subsequent DataListener recovery; FinishData ADMA branch.
- Trigger: ADMA transfer remains inhibited until software timeout, without hardware error IRQ/reset.
- Evidence: timedOut only sets a flag and exits polling; recovery calls FinishData, which clears DMAActive, invalidates/copies the full bounce result and returns without resetting/stopping controller. WaitCmd then returns an error and caller can release/reuse its data buffer and descriptor state.
- Impact: controller may keep writing to caller/bounce storage after ownership returns, corrupting reused memory or a later transfer; premature bounce copy can expose incomplete data.
- Configuration/confidence: SDHCI ADMA-enabled targets, high conditional on hardware remaining active. Hardware-error IRQ path separately attempts reset, so ordinary reset-success errors are not this trigger.

<a id="disk-194"></a>

### DISK-194 — SDHCI listener claiming does not wait for an in-progress IRQ callback

- Source: rom/devs/sdcard/sdcard_bus.c BusIRQ listener claim/unlock, FinishCmd/FinishData and WaitCmd recovery.
- Trigger: SMP IRQ handler clears a listener under lock, releases lock, and is delayed before processing it while caller task sees controller inhibit cleared.
- Evidence: listener NULL is treated as callback already completed. WaitCmd has no in-progress callback count/join and can return while IRQ handler still holds pointer to caller's stack tags/data. FinishData can also still be copying data after controller transfer completion.
- Impact: callback can access expired stack command tags or released IO buffer after caller returns/replies.
- Configuration/confidence: SMP SDHCI, medium-high from ownership trace; scheduling/interrupt routing determines race window. Lock prevents double claim but does not synchronize completion.

<a id="disk-195"></a>

### DISK-195 — SDHOST narrows caller data pointers to 32 bits on aarch64

- Source: arch/arm-native/soc/broadcom/2708/sdcard/sdcard_sdhost_bus.c SendCmd; sdcard_sdhost_intern.h dma_data_addr and MMIO accessors.
- Trigger: raspi-aarch64 running SDHOST on earlier SoC with caller buffer above 4GiB in virtual address space.
- Evidence: priv->dma_data_addr=(ULONG)sdData; read bounce copy casts this truncated field back to APTR. This remains used even though DMA itself always targets bounce buffer.
- Impact: successful read copies bytes to wrong low address rather than caller storage, potentially corrupting memory.
- Configuration/confidence: high conditional on high virtual caller addresses. Platform build explicitly includes SDHOST in aarch64. MMIO accessor also narrows IOBase; earlier SoC peripheral base normally fits 32 bits, so ordinary MMIO failure is not separately claimed.

<a id="disk-196"></a>

### DISK-196 — SDHOST ignores data status errors and command-idle timeout

- Source: arch/arm-native/soc/broadcom/2708/sdcard/sdcard_sdhost_bus.c SendCmd and BusIRQ.
- Trigger: DMA reaches END but controller reports SDHSTS data CRC/FIFO error, or command engine remains NEW past its idle timeout without SDCMD_FAIL.
- Evidence: SendCmd tests DMA completion and SDCMD_FAIL only; idle wait failure is merely debug logged, and SDHSTS is cleared unconditionally at done. IRQ handler also clears status without latching errors. Read bytes are copied before these final checks.
- Impact: damaged/incomplete card data or unsettled command can be reported as full successful IO.
- Configuration/confidence: SDHOST, high for ignored local error fields/timeout; exact hardware relation between SDHSTS errors and SDCMD_FAIL remains a hardware-dependent limitation.

<a id="disk-197"></a>

### DISK-197 — SD platform mailbox allocation failure is dereferenced

- Source: sdcard_sdhost_init.c SDHostInit and sdcard_bcm2708init.c BCM2708Init.
- Trigger: AllocMem of MBoxMessage_ fails on the backend selected for the SoC, while mailbox resource opens successfully.
- Evidence: aligned MBoxMessage is computed from NULL and used to fill message words without allocation check. Failure label only guards eventual FreeMem.
- Impact: NULL/low-address writes during device initialization under memory pressure.
- Configuration/confidence: both Raspberry Pi SD backends, high. Skipped-backend early returns do not dereference the message.

<a id="disk-198"></a>

### DISK-198 — Combined aarch64 SD build defines GetClockDiv twice

- Source: arch/arm-native/soc/broadcom/2708/sdcard/mmakefile.src BCM2708BUILDFILES64; sdcard_sdhost_bus.c and sdcard_bcm2708bus.c global FNAME_SDCBUS(GetClockDiv).
- Trigger: build current raspi-aarch64 sdcard.device target.
- Evidence: target includes both controller bus files; each defines the same non-static, non-weak SDCARD__SDBus__GetClockDiv symbol, one a stub and one implementation. No conditional guard separates definitions.
- Impact: ordinary linker multiple-definition failure prevents this configured target from building.
- Configuration/confidence: high from build/source definitions, no build executed. 32-bit raspi-arm/armeb selects SDHOST only and avoids duplicate definition.

<a id="disk-199"></a>

### DISK-199 — Broadcom SD delay arithmetic fails across microsecond timer wrap

- Source: sdcard_bcm2708time.c sdcard_Udelay; sdcard_bcm2708bus.c BCM283xWriteLong.
- Trigger: operation near 32-bit microsecond timer wrap (about every 71.6 minutes).
- Evidence: Udelay compares absolute CurrentTime() with now+usec instead of elapsed subtraction; overflow makes requested delay return immediately. Legacy write helper stores last time and compares against last+6 (IPTR-sized on aarch64); after wrap a stored pre-wrap timestamp can require waiting almost a full timer cycle, or beyond representable CurrentTime for the top few values on 64-bit.
- Impact: delays violate required register pacing or driver stalls around timer wrap; legacy backend can remain stuck in its write wait.
- Configuration/confidence: high for arithmetic; legacy delayed-write backend runtime selection is limited to BCM2711 fallback in current combined configuration. Udelay is shared by both backends.

Further outcomes and corrections:
- Current platform selection is mutually exclusive at runtime: SDHOST skips BCM2711/2712; SDHCI skips earlier SoCs. DISK-191 remains conditional for multiple-bus/future configuration, not ordinary current single-controller boot.
- DISK-190 reinsertion trigger is applicable to SDHCI media IRQ path. SDHOST assumes always-present card and does not implement hotplug events.
- Startup pending counter is decremented after BusPostIRQInit returns; both platform post-init functions register the initial unit synchronously. Earlier observation that it precedes initial card registration is rejected. It still does not prove asynchronous later insertion completion.
- SDHOST has no generic MMIO accessors. StartUnit's CMD7 failure diagnostic calls sdcb_IOReadLong unconditionally, causing a NULL function call on that backend after selection failure. This strengthens unconfigured-open/error-path candidate; final consolidation must assign an independent ID or group startup error handling.
- Open StartUnit runs in caller task without controller serialization while bus task can process IO. SDHCI listeners are overwritten by SendCmd; SDHOST shares a single DMA control block/bounce buffer. Concurrent initialization/reopen and IO therefore lacks transaction ownership. Needs final finding promotion with direct sequence.
- SDHCI PIO partial accounting and ignored recovery callback result are classified as DISK-202.
- WaitCmd's final data recovery ignores FinishData return entirely. If FinishData sees an error/timeout without IRQ latching CmdError, WaitCmd can return success. Lost-IRQ recovery similarly does not latch its failure. Related to transfer-accounting/error propagation above.
- ADMA descriptor builder bounds descriptor count, splits mapped pages and uses little-endian fields. Descriptor table physical address itself is cast to ULONG without physical-range check; MEMF_31BIT is a virtual allocation constraint, mapping applicability conditional.
- Hardware reset routines log timeout but return void; error IRQ path may release DMA even after reset failure. This extends reset ownership findings with conditional hardware failure.
- SDHOST DMA control-block address uses virtual cb directly through bus alias, while data uses KrnVirtualToPhysical. Allocations lack low-physical constraints/contiguity checks, and bounce address setup assumes one contiguous physical extent. Trace dma.resource/platform allocator before promoting mapping failure.
- SDHOST init allocation-failure cleanup leaks bus/private/control-block/channel/bounce allocations on several goto paths. Noexpunge limits ordinary lifecycle teardown; initialization failure remains a resource leak.
- Media/command signal allocation failure leaves -1 shifted into masks instead of fallback; timer creation failure is currently unused by transport waits, so no timer NULL dereference established.
- Byte order: PIO data reads convert little-endian MMIO ULONG to host integer then store native ULONG; big-endian target would byte-swap media word order. Current armeb selects SDHOST DMA byte copies, avoiding that specific PIO path. Command response endian handling separately reviewed.

<a id="disk-200"></a>

### DISK-200 — SDHOST selection failure calls a NULL SDHCI accessor

- Source: rom/devs/sdcard/sdcard_bus.c StartUnit failure diagnostic; sdcard_sdhost_init.c generic accessor assignments.
- Trigger: CMD7/WaitCmd selection fails while opening an SDHOST-backed unit.
- Evidence: failure branch unconditionally evaluates sdcb_IOReadLong(SDHCI_PRESENT_STATE,...); SDHOST sets sdcb_IOReadLong=NULL because it has a different register layout. Diagnostic uses bug(), not compiled-out debug wrapper.
- Impact: ordinary card selection error turns into NULL function call/crash instead of failed configuration.
- Confidence/configuration: high, SDHOST only.

<a id="disk-201"></a>

### DISK-201 — SD unit initialization can overlap another controller transaction

- Source: rom/devs/sdcard/sdcard_init.c Open; sdcard_bus.c StartUnit/SendCmd/BusTask; sdcard_sdhost_bus.c SendCmd.
- Trigger: two opens configure an inactive unit concurrently, or an existing opener submits IO while another Open retries StartUnit after a prior startup failure.
- Evidence: Open calls StartUnit directly and takes no bus transaction lock; bus task handles queued reads/writes independently. StartUnit leaves AF_Card_Active unset until it returns. SDHCI SendCmd replaces shared listener pointers/descriptors; SDHOST commands share DMA CB/bounce/private fields. Inhibit/idle polling is not a lock and can be passed by both contenders.
- Impact: command/listener overwrite, wrong data-buffer copy, lost completion or command failure; disk buffers may be accessed by the wrong transaction.
- Confidence/configuration: medium-high from local synchronization absence, both backends. Exec OpenDevice serialization may serialize two opens on some targets, but it does not serialize an Open with the independent bus worker; retry-after-failed-StartUnit path remains explicit because Open reports success despite failure.

<a id="disk-202"></a>

### DISK-202 — SDHCI PIO partial transfers can be reported at full length

- Source: rom/devs/sdcard/sdcard_bus.c FinishData and WaitCmd; sdcard_ioops.c ReadSector32/64 and WriteSector32/64.
- Trigger: DATA_END occurs while FinishData still has requested bytes remaining, or recovery-context FinishData encounters an error not latched by IRQ.
- Evidence: FinishData exits on DATA_END, subtracts remaining bytes from DATALEN, returns zero for early end. IO wrapper ignores resulting DATALEN and credits original chunk. WaitCmd final recovery also ignores FinishData's error return.
- Impact: truncated read can expose untouched caller-buffer bytes as valid disk data; short write can report full success.
- Confidence/configuration: high for accounting/error propagation, SDHCI PIO fallback. Whether ordinary hardware can signal premature end without an error bit is conditional; malformed/failing controller behavior is relevant to recovery.

<a id="disk-203"></a>

### DISK-203 — Failed SD signal allocation produces invalid signal masks

- Source: rom/devs/sdcard/sdcard_bus.c BusTask, SendCmd, WaitCmd and BusIRQ.
- Trigger: AllocSignal(-1) fails for MediaSig or CommandSig.
- Evidence: code logs failure and continues with field=-1; expressions 1L<<MediaSig and 1L<<CommandSig remain used for wait/signal/clear masks. Only TaskSig gets a fallback.
- Impact: undefined shift and wrong wait/signalling, potentially permanent disk-request hangs under resource exhaustion.
- Confidence/configuration: high from allocation failure path; generic bus initialization used by both backends, command-mask consequence primarily SDHCI.

<a id="disk-204"></a>

### DISK-204 — SD media changes do not invoke registered notification handling

- Source: rom/devs/sdcard/sdcard_bus.c media event branch; sdcard_device.c cmd_TestChanged and command table.
- Trigger: SDHCI card insertion/removal IRQ after unit discovery, with client querying change number or expecting its notification interrupt.
- Evidence: IRQ sets MediaChanged and signals bus task; task invokes RegisterUnit/empty ReleaseUnit only. ChangeNum increment/Cause callbacks are confined to cmd_TestChanged, reached by private HD_SCSICMD+1 or cmd_Eject, never by media branch. No other call sites found in all SD sources.
- Impact: ordinary physical changes leave change count and client notifications stale; old unit state may be used after replacement, alongside DISK-190's reinsertion allocation corruption.
- Confidence/configuration: high, SDHCI media detection. DISK-189 independently prevents safe add-notification registration until list initialization is fixed; stale ChangeNum requires no registration.

Corrections/limitations established:
- SDHOST DMAAllocChannel(0) does not install dma.resource completion IRQ (opt-in DMACHF_IRQ), so a second resource handler cannot clear its END bit. Reject that hypothesized polling race.
- ARM-native KrnVirtualToPhysical leaves low virtual memory unchanged and translates high kernel range; SDHOST's direct CB virtual address is therefore conditional on allocation region. Do not claim universal identity-mapping failure. Its data pointer narrowing remains independent DISK-195.
- Platform runtime selection currently leaves one bus; startup unit arrays are exactly one pointer, confirming DISK-190.
- SD initial detection builds boot node using hardcoded unit0/nameMMC0, uninitialized pp[3] and some environment slots (DE_INTERLEAVE/PREALLOC etc.). pp[3] overlaps DISK-089's boot parameter defect; missing slots/geometry semantics remain final partition integration work.
- Supported-command list includes DirectSCSI even though handler always returns BADADDRESS; this is capability misreporting, not a successful transfer defect.
- No tests/build/hardware execution performed. These outcomes remain source-derived candidates with stated conditional hardware/ABI assumptions.

<a id="disk-205"></a>

### DISK-205 — ROM USB cancellation/flush ignores the actual request queue

- Source: rom/usb/classes/massstorage/dev.c devBeginIO/devAbortIO; massstorage.class.c nMSTask CMD_FLUSH/CMD_RESET.
- Trigger: enqueue disk IO then abort it while it remains in unit_MsgPort, or send CMD_FLUSH intending to cancel queued requests.
- Evidence: BeginIO queues only into unit_MsgPort. AbortIO and CMD_FLUSH inspect XFerPending/XFerQueue, which source-wide inventory shows are never populated (only initialized/cleared). CMD_FLUSH consequently processes behind earlier FIFO requests and does not drain later unit messages.
- Impact: pending requests cannot be canceled through AbortIO and CMD_FLUSH does not implement advertised queue cancellation; writes may execute despite a flush/cancel request.
- Confidence: high from complete producer/consumer search. No premature active-request reply claimed: the stale XFerPending branch has no current assignment making it active.

<a id="disk-206"></a>

### DISK-206 — UAS reconnect retains endpoint pointers from the previous device

- Source: massstorage.class.c usbForceInterfaceBinding reuse, nAllocMS, nUasCollectEndpoints and nFreeMS.
- Trigger: detach then reconnect the same device/interface ID so the preserved NepClassMS unit is reused with newly created Poseidon endpoint objects.
- Evidence: nFreeMS releases pipes/streams but never clears EPCmd/EPStatus/EPIn/EPOut endpoint fields. UAS collector sets a field only if currently NULL, so old pointers prevent installation of new endpoints. Rebind replaces Device/Interface fields but not endpoints.
- Impact: psdAllocPipe/endpoint attribute operations can use endpoints belonging to the detached device, causing stale-pointer access or failed/wrong binding.
- Confidence: high conditional on Poseidon endpoint object lifetime across detach; trace core release to finalize UAF classification. Non-UAS allocation replaces EPIn/EPOut and avoids that particular stale-selection path.

<a id="disk-207"></a>

### DISK-207 — USB reported block size is trusted as a shift without validation

- Source: massstorage.class.c nGetBlockSize and nGetGeometry.
- Trigger: device returns zero, non-power-of-two or extremely large sector size in READ CAPACITY response.
- Evidence: BlockSize copied directly; BlockShift increments using signed 1<<shift until reaching size, without rejecting unsupported size or limiting shift. A non-power-of-two value rounds up shift, inconsistent with actual byte length; sizes beyond signed shift range invoke undefined shifts/possible nontermination.
- Impact: wrong byte/LBA mapping, malformed transfer sizes or hanging discovery on invalid device metadata.
- Confidence: high for missing validation and shift arithmetic; final read/write body trace will specify concrete transfer consequences. Read capacity short-length handling remains separate pending transport trace.

<a id="disk-208"></a>

### DISK-208 — USB geometry combines zero mode-page heads with division

- Source: massstorage.class.c nGetGeometry mode page 0x04 and missing-track calculation.
- Trigger: device returns nonzero cylinders but zero heads in accepted page 0x04, no usable track-sector page 0x03/0x05, and valid READ CAPACITY.
- Evidence: gotheads and gotcyl are set TRUE based solely on nonzero cylinders; Heads=mpdata[5] may be zero. Missing TrackSectors calculation divides TotalSectors by Cylinders*Heads without zero check.
- Impact: divide-by-zero fault during geometry discovery.
- Confidence: high conditional on malformed/inconsistent mode data and SIMPLE_SCSI disabled (default config enables it).

Other reviewed outcomes:
- QueryPartitionAttrs/GetPartitionAttrs callers do not validate success/attributes and compute offset modulo blockspercyl without zero guard. Existing malformed RDB environments make zero-cylinder geometry plausible; final partition contract trace required before independent promotion.
- partitions.c name[32] has worst-case 26-character device prefix +2 unit digits +P +2 partition digits +terminator =32, fitting ordinary two-digit values. Values above99 format incorrectly but do not add additional digits; do not claim that simple buffer overflow.
- partitions.c pp allocation leaks on successful AddBootNode and misaligned offset early return; MakeDosNode copies parameters, so ownership needs core MakeDosNode trace before final leak classification.
- Hardcoded PartTypes consumes ptyp.id[0] for every unnamed table, including GPT GUIDs; partition type/handler integration remains pending.
- NSD query locally checks pointer and length before writes; do not group this ROM USB implementation with DISK-102.
- UAS DDA DescriptorType/Length use UWORD locals; Poseidon packing definitions explicitly use PKCTRL_UWORD. Reject generic IPTR-output stack-overwrite suspicion for these attributes.
- nAllocMS signal-allocation failure is classified as DISK-235; see its independent driver applicability and platform limitation.
- Failure cleanup preserves freed pipe pointers, and nFreeMS does likewise; protocol-changing rebind can later operate/free stale pipe fields. Need concrete reconnect branch before promotion beyond endpoint DISK-206.
- Worker-startup failure publication is resolved as DISK-234; the later finding traces binding, OpenDevice and asynchronous BeginIO.
- Startup inquiry consumes all 36-byte fields without scsi_Actual check; short successful transport reply can leave inquirydata bytes uninitialized. Geometry capacity likewise ignores scsi_Actual, conditional on transport accepting short data.
- CMD_UPDATE is immediate successful no-op; cached-write flush/ordering root cause applies to ROM USB too. Hardware transport cache commands remain available only through direct SCSI.
- Suspend gates removable polling only; the subsequent Poseidon suspend-contract reconciliation documents automatic transfer-triggered resume and limits the remaining quiescence concern.
- Class expunge utility-library reference leak is resolved as DISK-239.

<a id="disk-209"></a>

### DISK-209 — USB write alignment expression compares inside the shift

- Source: massstorage.class.c nWrite64 alignment condition and nRead64 error-classification condition.
- Trigger: ordinary nonempty aligned USB write, e.g. length512 offset0 on a512-byte-block unit.
- Evidence: expression (dataremain >> BlockShift)<<BlockShift != dataremain lacks parentheses around left shift. C relational precedence parses it as (dataremain >> BlockShift) << (BlockShift != dataremain). Example yields 1<<1=2, treated as misalignment. Read's outer test is correctly parenthesized; its later error classifier shares the expression mistake.
- Impact: ordinary writes spuriously enable/use large-block emulation by default, or fail BADLENGTH when fallback is disabled; unexpected emulation exposes its narrower addressing limitations.
- Confidence: high; no hardware-dependent premise.

<a id="disk-210"></a>

### DISK-210 — USB seek passes an uninitialized command pointer

- Source: massstorage.class.c nSeek64.
- Trigger: valid aligned TD_SEEK/TD_SEEK64 on a unit routed through the worker.
- Evidence: local SCSICmd assigns data/length/CmdLength/flags/sense fields but never scsi_Command=cmd10. nScsiDirect dereferences its command pointer.
- Impact: stack-garbage pointer access/crash or random SCSI command instead of SEEK10.
- Confidence: high. Final direct dispatcher trace will establish exact first dereference.

<a id="disk-211"></a>

### DISK-211 — USB large-block emulation drops high LBA bits

- Source: massstorage.class.c nRead64Emul/nWrite64Emul.
- Trigger: 64-bit IO at >=2^32 physical blocks, routed to emulation (unaligned512-byte access on larger blocks, or DISK-209 write fallback).
- Evidence: emulation combines offset halves into ULONG startblock and uses READ10/WRITE10 only. Unlike native paths, it never computes/retains startblockhigh or emits CDB16.
- Impact: reads/writes alias low32-bit LBA; writes can corrupt earlier disk regions.
- Confidence/configuration: high, emulation enabled on sufficiently large device. For512-byteblocks threshold2TiB; for4096-byteblocks threshold16TiB.

<a id="disk-212"></a>

### DISK-212 — USB emulation counts a partial native-block tail as transferred

- Source: massstorage.class.c nRead64Emul/nWrite64Emul aligned bulk branch.
- Trigger: large-block device and emulation request starts at native boundary with length>=one native block but not native-block multiple, e.g.4608bytes on4096-byteblocks.
- Evidence: bulk branch sends datalen4608bytes with CDB block count datalen>>12=1, then advances caller/remaining by all4608bytes. It does not split4096-bytebulk plus512-byte partial tail, and ignores scsi_Actual.
- Impact: phase error on strict device; transport permitting short data can report untouched tail as read or omit tail write while reporting full success.
- Confidence: high for CDB/data inconsistency; transport outcome conditional. Related full accounting ignores actual lengths in native paths too.

Additional block-layer outcomes:
- nSeek64 ignores LBA highbits even after its missing command pointer is fixed; group with DISK-211's CDB10 narrowing if final root-cause organization permits.
- All IO loops credit requested datalen rather than scsi_Actual; transport short/residue validation must determine current reachability.
- Emulation scratch allocation failure returns BADADDRESS without assigning ioreq->io_Error/io_Actual. Worker discards returned LONG and replies, so request can appear successful with stale high-offset actual. Promote as independent failure-path candidate in final pass.
- Native CDB16 transfer-count shift is classified as DISK-237; the finding distinguishes undefined C arithmetic from target-dependent emitted behavior.
- MaxTransfer config computes1UL<<(cdc_MaxTransfer+16), with no bounds validation. Default5 means2MiB and fits CDB10 sectorcount; unusual configured large values can exceed16-bitcount or invoke undefined shift.
- No disk-capacity extent validation in block wrappers. They rely on device error handling; do not claim guaranteed out-of-capacity successful writes.
- Native wrappers make packed ULONG writes at CDB offsets2/6 using potentially unaligned pointers, conditional on target alignment requirements.
- Write-protect state is queried separately but nWrite64 does not enforce it locally. Devices should reject protected writes; cache/state contract pending removable task review.

<a id="disk-213"></a>

### DISK-213 — USB packet subclass copies arbitrary CDB length into twelve-byte stack buffer

- Source: massstorage.class.c nScsiDirect, ATAPI/FDDATAPI/UFI branch.
- Trigger: direct SCSI request with CDB length>12 on one of those subclasses, SIMPLE_SCSI disabled or command passing its filter.
- Evidence: local cmd12[12] is zeroed then CopyMem(command,cmd12,scsi_CmdLength) with no bound. CDB16 on ATAPI is sufficient when not filtered.
- Impact: stack overwrite before transport submission.
- Confidence: high; packet-subclass configuration conditional. Short/zero CDB fields are likewise read before per-opcode length validation, extending DISK-182/017's validation pattern.

<a id="disk-214"></a>

### DISK-214 — USB READ6/WRITE6 zero count translates to zero transfer

- Source: massstorage.class.c nScsiDirect six-to-ten-byte translation.
- Trigger: READ6/WRITE6 with count byte0 (256blocks in six-byte command) under MODE_XLATE/default or packet subclass translation.
- Evidence: translated count simply copied into cmd12[8], with cmd12[7] left zero. READ10/WRITE10 zero count means no blocks, not256.
- Impact: valid256-block request becomes no-op/phase error; transport may report incorrect completion relative to requested data.
- Confidence: high for semantic mismatch, runtime response depends on transport/device.

<a id="disk-215"></a>

### DISK-215 — Default SimpleSCSI rejects the driver's own large-disk CDB16

- Source: massstorage.class.c nLoadClassConfig default flags, nRead64/nWrite64 and nScsiDirect SIMPLE_SCSI whitelist.
- Trigger: native IO beyond32-bitphysical LBA on default configured unit, requiring READ16/WRITE16.
- Evidence: wrapper deliberately emits CDB16 when highLBA nonzero; whitelist permits READ6/10 and WRITE6/10 but omits16, falling into unsupported-command failure. NSD still advertises64-bitcommands and geometry diagnostic claimsIO will work beyond2TB.
- Impact: large-disk native IO fails under default settings. Emulation narrowing in DISK-211 is separate and can alias data instead of hitting this whitelist.
- Confidence: high, default flags; users disablingSimpleSCSI avoid the filter.

<a id="disk-216"></a>

### DISK-216 — Mode translation allocation failure submits incompatible original buffer

- Source: massstorage.class.c nScsiDirect MODE_SENSE6/MODE_SELECT6 translation.
- Trigger: psdAllocVec(scsi_Length+4) fails (or MODE_SELECT inputlength<4).
- Evidence: code still sets usecmd10=TRUE and sends translatedCDB10; cloned SCSICmd keeps original six-byteformat data pointer/length because replacement buffer assignment is conditional. No allocation error return.
- Impact: MODE_SELECT can transmit malformed parameters with changed semantics; MODE_SENSE writes ten-byteheader into caller's six-byteformat buffer and reports success without back-conversion, corrupting interpreted result.
- Confidence: high; no overflow claimed beyond transport host buffer clipping. ULONG length+4 overflow also possible on malformedextreme lengths, separate bounds consideration.

<a id="disk-217"></a>

### DISK-217 — SimpleSCSI all-pages response writes into a zero-length buffer

- Source: massstorage.class.c nScsiDirect MODE_SENSE6 page0x3f emulation.
- Trigger: valid six-byte MODE_SENSE command requesting allpages with scsi_Length=0 and NULL/empty data storage.
- Evidence: memset(data,0,length) is followed unconditionally by data[0]=lengthheader and reports scsi_Actual=4; only later page bodies have lengthguards.
- Impact: one-byte NULL/out-of-boundswrite, and actualcount exceeds suppliedcapacity.
- Confidence: high, SIMPLE_SCSI enabled by default. Related final MODE_SENSE6/10 write-protect inspection reads data[2]/[3] without minimum actual/length checks whenUnitReady; final consolidation should group response-length validation.

<a id="disk-218"></a>

### DISK-218 — USB media replacement preserves the previous medium's block size

- Source: massstorage.class.c nRemovableTask media transitions, nRead64/nWrite64 and nGetBlockSize.
- Trigger: removable reader changes medium to one with different sector size without interface rebind, PFF_REM_SUPPORT disabled(default).
- Evidence: removal/insertion incrementsChangeCount but never resetsBlockSize/BlockShift. Insertion only refreshesgeometry whenREM_SUPPORT or BlockSize==0. Read/write queriescapacity only whenBlockSize==0, so existingnonzero size persists until another explicitgeometry request.
- Impact: IO issued with stalebyte/LBAconversion and transfercount, potentially accessingwrongmedium sectors or failingphase checks. Partition scanner may refreshgeometry as a sideeffect, but requests/callbacks can run before it and its success is not required by thisstate machine.
- Confidence: medium-high conditional on differentmediumblocksize and ordering; same-sizeordinarycards unaffected.

<a id="disk-219"></a>

### DISK-219 — Removable task leaks its IO reply port on every restart

- Source: massstorage.class.c nAllocRT and nFreeRT.
- Trigger: removabletask stops/restarts afterDOS appears or alldevices disappear/reappear.
- Evidence: nAllocRT always createsnh_IOMsgPort; successful cleanup frees timerport/request but never deletes/resetsnh_IOMsgPort. Nextallocation overwritespointer.
- Impact: memory/signalresources leak acrossrestarts, eventually preventingpolling or otherportcreation.
- Confidence: high; allocation-failurecleanup does delete it, successfulcleanup does not.

Additional outcomes:
- nSeek64's missingcommand assignment is confirmed reachable: nScsiDirect reads scsi_Command[0] immediately in its defaultSimpleSCSI filter.
- Mode translation modifies originalMODE_SENSECDBallocation byte and scsi_Length for255byterequests, affectingcaller reuse; not necessarily forbiddenAPIownership, recordcompatibilityissue.
- FAKE_INQUIRY returnsGOODwithfilled36bytes but leaves scsi_Actual=0 fromentry reset; producer/reporting mismatch, candidateforfinalpromotion. Vendor/product strings use strlen withoutNULLguard; coreattribute defaults need trace.
- MODE_SELECT translation leavesreservedheaderbytes zero only if psdAllocVec clearsmemory; allocationcontractpending. Back-conversion leavesMODE_SELECT actual0 despite transmittedbytes.
- Removablepoller reads sensebytes2/12 afteranytransporterror withoutcheckingSenseActual or zero-initializingbuffer, enablingstalesensemedia-state decisions aftererrors. Transportclearingcontractpending.
- nFreeMS/nFreeRT set taskpointerNULLunderForbid justbeforefunctionreturn; joinwaitobservesNULLratherthanexplicitexittermination. SingleCPUForbidprotects tail; SMP assumptionsneedfinalclassification.
- Removabletask causesnotificationhandlerswhiledevBeginIO canadd/removelistwithout sharedlock; Forbidonlyinmutatingcaller andtraversalnotForbid. SingleCPUpreemption/listmutation possible; callbacklifetimecontractpending.
- USB physicaldisconnectdrainsrealunitportinnFreeMS, unlikeDISK-030Workbenchconditionaldrain. RejectclaimROMhasnodrain. BeginIOdenycheckandPutMsgarenotatomicwithdrain, so concurrentenqueueafterdrain remainsintegrationrace.
- nIOCmdTunnelForbidspansdenycheckandPutMsgonUP, thenwaitsitsreplyport. SharedfakeIOrequestisusedseriallybyremovabletaskinreviewedcallers.
- Classconfigcopystringslackvalidationcanleaveunterminatedhandler/controlnames; actualPoseidoncfgchunklengthvalidationandGUIconstraintsremainpending.

<a id="disk-220"></a>

### DISK-220 — USB automount dereferences unvalidated DOS startup values

- Source: `rom/usb/classes/massstorage/massstorage.class.c:4880`, `FindMatchingDevice`, and `MatchPartition` at 4760.
- Trigger: FAT/CD automount scans a DOS device whose startup BPTR converts to an address above 0x1000 but does not refer to a valid FileSysStartupMsg. DOS startup values are not universally filesystem startup structures.
- Evidence: the previous TypeOfMem startup guard is commented out. The code reads fssm_Environ and fssm_Device before checking the derived pointers. Checking the derived addresses cannot protect the initial dereference. Environment validation additionally accepts de_TableSize > 0 rather than requiring entries through DE_DOSTYPE before MatchPartition reads those fields.
- Impact: invalid memory access during unrelated USB media mounting; short valid environment vectors can also be read beyond their declared extent.
- Confidence: high for missing validation; occurrence depends on other installed DOS device startup formats. No arbitrary malformed on-disk input alone is claimed to create such a DOS entry.

Additional outcomes and rejected candidates:
- ReadRDSK and ReadPART have no callers in this class. Current removable mounting delegates partitioned media to CheckPartitions in partitions.c. FAT/CD fallback sets rdb_FileSysHeaderList=NIL_PTR before CheckPartition/MountPartition, bypassing the old on-disk ReadFSHD/BuildFileSystem branch. Old helpers contain unchecked LSEG SummedLongs arithmetic, chain cycles, raw endian comparisons and fixed-layout copying, but these are dormant-code observations, not currently reachable disk-input candidates.
- CheckFATPartition has only a startblock=0 caller. Its environment starts at cylinder zero intentionally for the superfloppy fallback; do not report an active nonzero-partition-offset bug in this function. FAT MatchPartition only compares unit/device/DosType, but this path mounts a whole-device superfloppy and reuse across removable media is intentional; no independent wrong-partition candidate established.
- Rejected GUI 32-bit destination overwrite under the GCC build: workbench/libs/muimaster/macros.h:339 implements get() through an IPTR union temporary and copies the destination's own type. The non-GCC macro directly casts the destination but is configuration conditional and not a demonstrated current build failure. Big-endian narrow union extraction is a separate macro-wide question outside this localized review.
- AutoDetectMaxTransfer checks allocation/open failures and frees all allocated resources. It divides by BlockSize after an unchecked geometry DoIO; if the geometry request fails while BlockSize remains zero, this can divide by zero. On a transfer error it retains the failing MaxTransfer value rather than reverting to the preceding successful value; only the memcmp failure branch decrements it. Both are pending localized candidates for final consolidation.
- FindDeviceNode returns a DOS node after releasing the DOS list lock; CheckPartition later dereferences its startup. Concurrent RemDosEntry/free can invalidate it. Trace ownership/lifetime conventions before promoting the race.
- GUI strings are capped below their destination lengths, making their strncpy calls terminated for normal UI input. Persisted configuration remains a separate validation surface. Successful cleanup's trailing Forbid is intentional task-exit serialization on a single CPU and is not itself a missing Permit bug.

<a id="disk-221"></a>

### DISK-221 — CBI command-stall retry can return success before transferring data or status

- Source: `rom/usb/classes/massstorage/massstorage_cbi.c`, final command-block-error branch of nScsiDirectCBI.
- Trigger: initial ADSC command submission stalls, nBulkClear runs, and the second ADSC succeeds.
- Evidence: the retry occurs inside the else branch selected for the initial failure. On successful retry ioerr becomes zero, but control never re-enters the data/status branch. rioerr remains zero and the function exits at if(!rioerr). No requested data phase or command status occurs.
- Impact: command falsely completes successfully, leaving the device waiting for data or an unread status. Block wrappers can report the full request completed while buffers were untouched or writes were not sent.
- Confidence: high; CB/CBI transport and recoverable initial command stall required.

<a id="disk-222"></a>

### DISK-222 — USB block wrappers credit requested bytes after short successful transport transfers

- Source: `rom/usb/classes/massstorage/massstorage.class.c:2990` and `:3153`, nRead64/nWrite64, with analogous emulation paths; transport scsi_Actual producers in bulk/CBI/UAS files.
- Trigger: device returns GOOD status with a short data transfer, including accepted RUNTPACKET or a transfer aborted when CBI status arrives first.
- Evidence: transports assign scsi_Actual from psdGetPipeActual/stream results. Wrappers only check nScsiDirect's error return and then advance dataoffset/dataremain/LBA by datalen, ignoring scsi_Actual. Bulk deliberately ignores CSW residue as an additional firmware compatibility policy.
- Impact: successful io_Actual overstates bytes transferred; reads expose old buffer contents as disk data and short writes are acknowledged as complete.
- Confidence: high for accounting defect; device behavior determines frequency. DISK-212 separately describes command/data length disagreement caused by emulation chunking.

<a id="disk-223"></a>

### DISK-223 — USB mode-page parser accepts incomplete pages and miscounts block descriptors

- Source: `rom/usb/classes/massstorage/massstorage.class.c:1973`, nGetModePage, geometry consumers at 2394/2424/2446.
- Trigger: SIMPLE_SCSI disabled and a short/malformed MODE_SENSE page with block descriptors or insufficient geometry fields.
- Evidence: after res += res[3], the subtraction reads res[3] from the shifted pointer rather than the original descriptor length. Page validation compares remaining bytes with res[1], omitting the two-byte page header and required caller field extent. Geometry then reads indices through 11 without further length checks.
- Impact: geometry can be derived from bytes outside returned data; a page near the end of the fixed buffer can cause a physical out-of-bounds read. Example: actual=255, descriptor length=249, page starts at offset253 and callers read through offset264 if page ID/length pass the defective remaining-byte check.
- Confidence: high; nondefault SimpleSCSI-off configuration and malformed device response. Defaults synthesize pages and avoid this device-page path.

Further reviewed outcomes:
- CDB16 transfer-count first byte shifts a 32-bit ULONG by BlockShift+24 (33 for 512-byte sectors), which is undefined C behavior. Record alongside large-LBA construction candidates; exact generated count is target/compiler dependent.
- nRead64/nWrite64 cast addresses cmd10+2 and cmd16+2/+6 to ULONG pointers and store multi-byte values. These addresses are not guaranteed four-byte aligned. Strict-alignment targets can fault; consolidate with architecture-sensitive wire-buffer access after checking compiler packed-access conventions.
- Bulk NAK-timeout retry resubmits a new CBW after data/status timeout without reset or first collecting the previous CSW. It can encounter the preceding command's remaining phase/status and desynchronize BOT, in addition to DISK-036's unlimited retry budget. Phase-specific recovery needs separate protocol verification before final promotion.
- UAS consumes one status IU without validating its returned tag against the issued tag or dispatching READ_READY/WRITE_READY IU identifiers. Unknown IU/status combinations can be interpreted as final completion. These extend DISK-035's status validation surface; protocol-layout/endian correctness remains pending authoritative local contract evidence.
- nCBIRequestSense accesses sense bytes 2/12/13 without ensuring actual returned extent; the outer CBI path replaces the transport actual with min(caller length,18). Bulk autosense similarly indexes bytes beyond short returned actual. Consolidate these with DISK-034 and removable-state response validation.
- nRelaxNakTimeout's ncm argument is unused; no correctness candidate. Debug dumpmem bounds its reads by len; no independent candidate found.

<a id="disk-224"></a>

### DISK-224 — Workbench USB whole-device mount expresses sectors as cylinders

- Source: `workbench/devs/USB/classes/MassStorage/storage_volumes.c`, USBMSS_AddVolume environment construction.
- Trigger: a removable USB disk is mounted as a whole device, or its initial boot node is used directly before DOS partition processing.
- Evidence: de_Surfaces=255 and de_BlocksPerTrack=63, but de_HighCyl=msu_blockCount (READ_CAPACITY last LBA), rather than converting capacity to that cylinder geometry. CheckPartitions deliberately reinserts removable-device boot nodes without rewriting the environment.
- Impact: filesystem-visible extent is exaggerated by roughly 16065; filesystem calculations can issue out-of-range accesses or interpret incorrect volume boundaries. The transport's own bounds behavior determines whether the requests fail, and no out-of-device physical write is asserted.
- Confidence: high for geometry mismatch; actual handler usage/configuration determines consequences. This is distinct from DISK-032's device GETGEOMETRY count issue.

<a id="disk-225"></a>

### DISK-225 — Workbench USB discovery dereferences unchecked boot-node allocation

- Source: `workbench/devs/USB/classes/MassStorage/storage_volumes.c`, USBMSS_AddVolume immediately after successful MakeDosNode.
- Trigger: AllocMem(sizeof(struct BootNode)) fails after the device node was created.
- Evidence: bn->bn_DeviceNode and other fields are assigned without testing bn.
- Impact: NULL dereference while discovering a disk under memory pressure.
- Confidence: high. Adjacent constructor/task allocations also need final grouping with earlier initialization-failure observations rather than duplicate IDs.

<a id="disk-226"></a>

### DISK-226 — USB partition automounters divide by unvalidated cylinder geometry

- Source: `rom/usb/classes/massstorage/partitions.c`, AddPartitionVolume; `workbench/devs/USB/classes/MassStorage/storage_volumes.c`, AddPartitionVolume; parser contract in `rom/partition/partitionrdb.c:299` and GetPartitionAttrs.
- Trigger: a checksum-valid RDB partition environment supplies zero de_Surfaces or de_BlocksPerTrack, or their product truncates to zero in ULONG.
- Evidence: RDB loading copies these values without positivity validation; GetPartitionAttrs copies the resulting DosEnvec. Both consumers compute blockspercyl from them, then evaluate i % blockspercyl before any zero check. Even i=0 (ordinary top-level RDB) divides by zero.
- Impact: automount task exception instead of rejecting malformed partition metadata.
- Confidence: high. Parser-side environment-count overflow is already DISK-068; this candidate covers accepted geometry values with otherwise bounded metadata.

<a id="disk-227"></a>

### DISK-227 — USB mounting leaks copied parameter packets on success and skipped geometry

- Source: both USB AddPartitionVolume functions and Workbench USBMSS_AddVolume; allocation ownership traced through `rom/expansion/makedosnode.c`.
- Trigger: successful partition mount, nonaligned nested partition skip, or any USBMSS_AddVolume path after pp allocation.
- Evidence: AddPartitionVolume returns before FreeVec(pp) on success and geometry skip. MakeDosNode allocates/copies its environment and names, so it does not retain pp. USBMSS_AddVolume's AllocMem(24*sizeof(IPTR)) packet is never freed on any exit.
- Impact: repeated media/discovery/mount operations leak public memory. Device-node/handler allocations also leak on later failure and require matching cleanup; no claim that successful published DOS nodes themselves should be freed.
- Confidence: high.

Other integration outcomes:
- Both partition walkers retrieve bootable but ignore PT_AUTOMOUNT; RDB PBFF_NOMOUNT and GPT NOMOUNT therefore do not suppress USB automount. Existing convention and comparison with core boot integration remain to trace before promotion.
- ROM CheckTables returns TRUE merely for a recognized table, even if every leaf was skipped or lacked a supported handler. The removable caller then skips FAT/CD fallback; whether fallback is appropriate for a recognized but unmountable table is policy-dependent, not yet a bug claim.
- Workbench IsRemovable ignores geometry request error and reads an uninitialized dg_Flags on failure. Workbench AddPartitionVolume opens dos.library then calls DeviceProc without checking that open result. DOS availability normally holds for the process caller; failure handling remains a candidate assumption.
- Workbench Storage_Init sets up its pool/lists and returns success even if USB superclass object creation or AddClass registration fails. Storage_Expunge deletes the pool without traversing cached units/tasks; cached workers retain module code/static-data pointers. Trace library-open references before claiming an expunge use-after-free.
- The baseline commit and clean worktree were rechecked at this continuation's start. No project files were changed and no build/tests were run.

<a id="disk-228"></a>

### DISK-228 — Boot scanner duplicate-name suffix overflows a full partition-name buffer

- Source: `rom/dosboot/bootscan.c`, AddPartitionVolume duplicate mount-name loop.
- Trigger: a valid 31-character partition name collides with an existing mount-list name, or a generated long name leaves fewer than two spare characters.
- Evidence: name[32] receives a properly clipped 31+NUL PT_NAME. The first collision executes strcat(name,".1") without length check, writing the suffix and terminator beyond the stack array. A 30-character name also overflows its final terminator.
- Impact: boot-time stack corruption from otherwise valid partition names. Subsequent collision handling only increments the last character and does not provide an arbitrary-length numeric suffix.
- Confidence: high; duplicate long name required.

<a id="disk-229"></a>

### DISK-229 — Automatic partition mounting ignores explicit no-mount flags

- Source: `rom/dosboot/bootscan.c`, ROM USB partitions.c and Workbench USB storage_volumes.c, each AddPartitionVolume; flag contract `rom/partition/include/libraries/partition.h:92`; RDB/GPT PT_AUTOMOUNT implementations.
- Trigger: an otherwise supported RDB partition with PBFF_NOMOUNT or GPT partition with GPT_PF1_NOMOUNT is discovered by these walkers.
- Evidence: PT_AUTOMOUNT is documented as whether the partition will be automatically mounted. Both parsers expose the flag. The walkers query PT_BOOTABLE but never PT_AUTOMOUNT, and still call AddBootNode with ADNF_STARTPROC. Bootable FALSE merely changes priority and does not skip registration/startup.
- Impact: partitions explicitly excluded from automatic mounting are registered and started, potentially giving unintended filesystem access. This is a flag-policy violation rather than a claim of resulting on-disk corruption.
- Confidence: high for RDB/GPT flag omission; named partitions with known DosType/handler required.

Integration additions/corrections:
- DISK-226 also affects core `rom/dosboot/bootscan.c`'s i % blockspercyl/division. Core checks DosType/TableSize but does not validate positive geometry.
- GetOffset in all three walkers accumulates parent start offsets into ULONG, and getStartBlock in partition_support.c also returns ULONG. Nested absolute offsets at/above 2^32 native blocks truncate before reaching the otherwise 64-bit ReadPartitionDataQ/WritePartitionDataQ byte offset. This extends the partition capacity/address-width family (DISK-059/060); no new ID until deduplication.
- Rejected unbounded generic CheckTables recursion: MBR/GPT check functions accept only disk roots; EBR accepts MBR extended partitions; RDB accepts root or designated MBR/EBR types and rejects RDB children. These restrictions bound the generic nesting structure. EBR's own cyclic link chain remains DISK-062.
- Core scanner correctly inherits device DMA mask/MaxTransfer/BufMemType only where both environment vectors declare those fields. It ignores zero/unknown DosType, copies pp with flags initialized, and uses the expansion mount-list semaphore. MakeDosNode/AddBootNode semaphore nesting is recursive for the owning task; no independent deadlock asserted.
- FindFileSystem/AddBootFileSystem call crosses into already-reviewed partition filesystem-loading paths; their on-disk parser defects remain existing candidates rather than duplicate boot-scanner findings.

<a id="disk-230"></a>

### DISK-230 — Bootblock checksum loop wraps its 16-bit index for large bootblocks

- Source: `rom/dosboot/bootstrap.c:104`, BootBlockCheckSum; GetBootNodeDeviceUnit at58 and dosboot_BootBlock at194.
- Trigger: a boot node declares at least65536 bytes of boot blocks, allocation succeeds and CMD_READ returns success. RDB environment metadata supplies de_BootBlocks/de_SizeBlock and the scanner preserves them; e.g.128 boot blocks of512 bytes produces65536.
- Evidence: bootblock_size is ULONG but loop index i is UWORD and advances by4. At65532 the next increment wraps to0, leaving i < bootblock_size forever. The size is computed from metadata without an upper bound.
- Impact: boot task hangs indefinitely while checksumming large declared bootblock data. Applies to all builds; non-m68k still performs checksum before its boot-code macro returns NULL. Ordinary1024-byte floppy bootblocks do not trigger it.
- Confidence: high.

Additional boot consumer outcomes:
- de_BootBlocks * de_SizeBlock * sizeof(ULONG) narrows to ULONG without overflow/minimum validation. The result remains a multiple of four, so a partial-word checksum overread is not established. A value below twelve bytes lacks the executable prefix, but reaching buffer+12 additionally requires a passing checksum and matching filesystem DosType; see the reconciled limitation below.
- Bootblock CMD_READ checks io_Error but not io_Actual. A successful short read can checksum uninitialized AllocMem bytes. This is the same short-success consumer pattern as other findings; no claim of booting arbitrary code without a matching checksum/filesystem ID.
- Bootblock read always uses physical io_Offset=0 despite the boot node environment potentially describing a nonzero partition. Native m68k partition bootblock applicability needs convention tracing before promoting a wrong-partition finding.
- DevicePresent leaves a successfully opened device owned by its caller, which closes it in the inspected branches. Bootblock returned init code is called after closing/freeing the request and reply port, and those pointers are nulled; reject a post-init double-free suspicion in current code.
- Menu disk-presence slice initializes io_Actual=1, requires a successful zero change-state result, then closes the opened device and destroys the request/port; no independent request lifetime issue found in this slice.

<a id="disk-231"></a>

### DISK-231 — USB persisted transfer limit is used as an unchecked shift count

- Source: `rom/usb/classes/massstorage/massstorage.class.c:946`, `:997` configuration copies; maxtrans expressions at2655/2764/2895/3066 and mount environment5228/5336.
- Trigger: saved class/device configuration contains cdc_MaxTransfer outside the GUI range0–5, e.g.16 or UINT32_MAX.
- Evidence: raw config copy accepts the field without clamping. IO computes 1UL << (cdc_MaxTransfer+16) then narrows to ULONG. On32-bit unsigned-long targets value16 shifts by32, undefined; on64-bit targets that result narrows to zero, and larger values still exceed shift width. A zero maxtrans leaves datalen zero and dataremain unchanged in transfer loops.
- Impact: malformed/incompatible persisted settings can produce undefined arithmetic, nonadvancing IO loops, or invalid command counts. Ordinary GUI-produced settings avoid this trigger.
- Confidence: high for missing bounds/arithmetic; precise result of undefined shifts depends on target/compiler. No remote disk metadata trigger claimed.

<a id="disk-232"></a>

### DISK-232 — USB raw configuration strings lack termination validation before mount conversion

- Source: same class/binding config loads, CheckFATPartition/AutoMountCD and c2bstr macros at3875/3890.
- Trigger: raw saved config fills the complete fixed-size FATDOSName/handler/control array without a NUL.
- Evidence: raw CopyMem overwrites defaults including their terminators, without adding terminators afterward. c2bstr walks until NUL with no source/destination bound; FATDOSName is copied into a32-byte on-disk-style partition name. Handler strncpy source reads up to84 bytes from64-byte config arrays, and handler/control uses elsewhere require termination.
- Impact: reads beyond configured fields and potentially writes beyond name/control destinations while mounting a device. Adjacent zero fields may terminate some particular payloads early; the field contract is not enforced and full neighboring nonzero fields can extend the walk.
- Confidence: high for validation omission, conditional on corrupt/incompatible saved configuration. Normal GUI lengths leave room for NUL and are not asserted vulnerable.

Configuration bounds clarification:
- Class copy length is min(chunk-declared payload length, local default payload length), so it is bounded at the destination structure size. Do not report an unconditional destination-structure overflow from these CopyMem calls alone. Shared IFF parser malformed declared-length handling belongs to its separate provider contract.
- Strings/numeric values are stored in native structure layout with IPTR fields; cross32/64-bit configuration portability may misplace fields even when the byte count is bounded. This is a compatibility observation needing supported migration expectations, not a promoted standalone candidate.

<a id="disk-233"></a>

### DISK-233 — SDHCI transfer completion uses the block-gap interrupt bit

- **Severity / confidence:** Medium conditional completion misrouting; high source/register confidence, ordinary transfer failure not established.
- **Source:** compiler/include/hardware/sdhc.h:89, 108; arch/arm-native/soc/broadcom/2708/sdcard/sdcard_bcm2708init.c:41–47, 62–63; rom/devs/sdcard/sdcard_bus.c:1198–1220, 1325–1327, 1572–1604.
- **Trigger / trace:** SDHCI_INT_DATA_END is defined as 1<<2 rather than 1<<1. Initialization consequently enables/masks the wrong completion bit; DMA's listener-claim mask and task lost-interrupt recovery also test bit 2. Actual transfer completion at bit 1 does not satisfy these tests, while a block-gap event can be interpreted as completion. The [Broadcom register manual](https://www.raspberrypi.com/wp-content/uploads/2012/02/BCM2835-ARM-Peripherals.pdf) identifies the two distinct bits.
- **Impact / limits:** Data-completion interrupt routing and acknowledgment are incorrect. Crucially, WaitCmd:1425–1447 services any remaining DataListener after polling, so ordinary present-state completion can still run FinishData and its cache/bounce cleanup. Do not claim universal DMA data loss or hang. An early block-gap event can select completion prematurely, but current ordinary transfers do not enable deliberate stop-at-gap; that impact is conditional on an actual such event. Current raspi-aarch64 includes this backend; raspi-arm/armeb choose SDHOST, whose synchronous transfer code avoids this mask path. Source-only, no controller execution.

<a id="disk-234"></a>

### DISK-234 — ROM USB publishes units whose worker startup failed

**Confidence:** high. **Trigger/configuration:** fresh mass-storage binding where psdSpawnSubTask fails, or worker pipe/message-port allocation fails. **Impact:** OpenDevice succeeds for a nonfunctional unit; asynchronous requests are queued without a live consumer and can remain unreplied.

`rom/usb/classes/massstorage/massstorage.class.c:401–451` allocates a zero-cleared NepClassMS, publishes it in nh_Units, and records firstncm before starting the worker. Thus ncm_DenyRequests initially remains FALSE. Lines 657–693 wait when a worker was spawned but neither remove/disable the unit when ncm_Task remains NULL nor reject the binding; the cleanup code is commented out and firstncm is returned. nAllocMS allocation failure sets ncm_Task NULL and signals the starter (1847–1858), without setting DenyRequests. `dev.c:78` accepts a unit solely on !DenyRequests, while asynchronous BeginIO paths (260–266 and 292–298) use the same predicate before PutMsg. The spawn-failure case also leaves the unit port without an initialized consumer. Separate from DISK-205's cancellation queue mismatch and the disconnect queue-drain race.

<a id="disk-235"></a>

### DISK-235 — ROM USB fails to reject exhaustion of the unit-port signal

**Confidence:** high for unchecked allocation; exact invalid-signal behavior is platform dependent. **Trigger:** AllocSignal(-1) fails during successful nAllocMS startup. **Impact:** the published PA_SIGNAL request port has an invalid signal number; requests cannot reliably wake its worker, with undefined shift/signalling behavior possible.

`rom/usb/classes/massstorage/massstorage.class.c:1737–1742` stores AllocSignal(-1) into mp_SigBit and proceeds without testing for -1. Subsequent pipe allocation can succeed and the worker is marked active. This is an independent driver manifestation of the exhausted-signal pattern documented for SD as DISK-203. No claim is made that normal systems exhaust signals or that one particular kernel necessarily crashes.

USB startup reconciliation: the two unnumbered observations in the earlier class pass are now DISK-234/235 rather than unresolved candidates. Source-only review; no failure injection performed.

<a id="disk-236"></a>

### DISK-236 — USB emulation scratch allocation failure is not returned in the IO request

**Confidence:** high. **Trigger:** a request reaches large-block emulation and allocation of the first/replacement native-block scratch buffer fails. **Impact:** the worker replies without the allocation error; a request entering with io_Error zero can appear successful despite no transfer. For 64-bit commands io_Actual can retain its input high-offset word instead of a transfer count.

`rom/usb/classes/massstorage/massstorage.class.c:2677–2683` and 2785–2792 return IOERR_BADADDRESS as a function result but do not assign io_Error or reset io_Actual. The enclosing nRead64/nWrite64 propagate that function result (2915/3086), but the worker calls them without capturing it and immediately ReplyMsg (1355–1379). Other validation errors explicitly assign request fields, and successful emulation assigns io_Actual at its end; neither happens on this early return. Distinct from DISK-234's worker startup failure and DISK-212's successful-transfer accounting error. Resolves the earlier unnumbered scratch-allocation observation.

<a id="disk-237"></a>

### DISK-237 — USB READ/WRITE(16) count construction shifts a 32-bit value beyond its width

**Confidence:** high for undefined C arithmetic; emitted CDB effect depends on compiler/target. **Applicability:** native READ/WRITE(16) path on a high-LBA request; SimpleSCSI must permit the command (see DISK-215). **Impact:** transfer-count high byte can be wrong, potentially requesting substantially more device blocks than the data buffer describes, or producing command rejection.

`rom/usb/classes/massstorage/massstorage.class.c:2965` and 3135 assign cmd16[10] from `datalen >> (ncm_BlockShift + 24)`. datalen is ULONG, explicitly a 32-bit unsigned type (`compiler/include/exec/types.h:39–43`). With ordinary 512-byte sectors BlockShift is 9, making the shift 33; 4096-byte sectors produce 36. The C expression is undefined, although the intended upper count byte is zero for a byte count limited to ULONG. A 64-bit CPU does not make ULONG 64 bits. No compiler assembly or hardware behavior is asserted. Separate from DISK-211's emulation LBA truncation and DISK-231's configuration-controlled shift.

<a id="disk-238"></a>

### DISK-238 — Synthesized USB inquiry reports zero returned bytes

**Confidence:** high. **Applicability:** ROM USB PFF_FAKE_INQUIRY configuration/fallback, INQUIRY with a data buffer of at least 36 bytes. **Impact:** direct-SCSI clients see successful status with zero data returned despite a populated inquiry response, defeating length-based discovery/validation.

`rom/usb/classes/massstorage/massstorage.class.c:3186` resets scsi_Actual to zero on entry. The fake inquiry branch (3400–3430) fills the response, sets CmdActual, Status and SenseActual, then returns success without setting scsi_Actual. This is local synthesized-response accounting, separate from DISK-222's block wrappers ignoring transport actual counts. The branch's manufacturer/product pointer safety remains a separate dependency contract; no NULL-string crash is asserted here.

<a id="disk-239"></a>

### DISK-239 — ROM USB class expunge retains its utility-library reference

**Confidence:** high. **Impact:** resource/reference leak across successful class load/unload cycles; utility.library cannot release this reference. **Trigger:** successful initialization followed by libExpunge's successful branch.

`rom/usb/classes/massstorage/massstorage.class.c:46` opens utility.library and retains nh_UtilityBase. Initialization failure closes it (104), but successful libExpunge (142–195) closes ExpansionBase, PartitionBase and PsdBase and never UtilityBase. A diagnostic at 150 prints that it is closing utilitybase without actually calling CloseLibrary. Generated class teardown does not own this explicit manually opened field. This resolves the prior cleanup observation; no worker-use-after-free claim is bundled with this leak.

<a id="disk-240"></a>

### DISK-240 — Disk-image preference saves discard write/close failures and overwrite the previous file

**Confidence:** high. **Trigger:** disk-full, handler write failure, or buffered final-write failure while saving unit preferences. **Impact:** the existing mount/image settings are replaced by a truncated or incomplete XML file, while WritePrefs returns TRUE; subsequent startup can lose the stored image/configuration. Applies to disk-image preference saves, not backing-image sector writes.

`workbench/devs/diskimage/prefs/writeprefs.c:37–50` opens the destination directly with MODE_NEWFILE, ignoring every FPrintf/FPutC result, sets res TRUE, and ignores Close's result. WritePrefsObject and Indent are void and cannot propagate errors. The local DOS Close contract (`rom/dos/close.c:28–37`) explicitly says buffered final writes can fail and Close then returns zero. `device/unit.c:447–457` also ignores the helper's return while saving both ENV and optional ENVARC paths. Opening the existing destination for replacement makes the previous complete preferences unavailable before the new document is known to be written. This is separate from DISK-188's XML text escaping: even strings requiring no escaping are vulnerable. No disk-full simulation was run.

Preference writer reconciliation: the earlier unnumbered partial/non-atomic save observation is now DISK-240. Public DictSetObjectForKey can be misused with already-owned objects or cyclic dictionary insertion, but in-tree mount callers construct fresh objects; do not promote caller-contract misuse as an ordinary disk-triggered memory bug without a concrete supported path.

<a id="disk-241"></a>

### DISK-241 — USB transfer-limit autodetection divides by an unavailable block size

**Confidence:** high. **Trigger:** user invokes autodetection for a ready unit with BlockSize zero, then its TD_GETGEOMETRY fails without populating that field (for example, a transient device/transport failure). **Impact:** divide-by-zero in the configuration task instead of a reported detection failure.

`rom/usb/classes/massstorage/massstorage.class.c:6271–6280` issues geometry conditionally but ignores DoIO's result and immediately computes `(memsize >> 1) / cncm_BlockSize`. UnitReady is checked earlier but does not guarantee a later geometry command succeeds. The helper allocates/opens successfully before this path. No hardware fault injection performed. Separate from DISK-208's mode-page cylinder division.

<a id="disk-242"></a>

### DISK-242 — USB transfer-limit autodetection retains the failing probe limit

**Confidence:** high. **Trigger:** smaller chunk probes succeed, a larger probe returns an IO error during autodetection. **Impact:** the active shared transfer limit stays at the size that just failed; subsequent ordinary IO can repeat the oversized transfer failure.

`rom/usb/classes/massstorage/massstorage.class.c:6354–6357` increments cdc_MaxTransfer after successful probes. The next larger read's IO-error branch (6334–6340) sets bail and exits without restoring the last successful value or original configuration. Only the data-mismatch branch decrements the limit (6344–6349). The helper mutates ncm_UnitLUN0's CDC directly, so the failed value is active even before any separate settings-save operation. A failure at the first 64KB probe has no established successful replacement; this finding specifically uses a later larger-size failure. No claim that every IO error proves a device transfer-size limit.

Autodetection reconciliation: both prior localized observations are now explicit candidates with distinct triggers. Cleanup of request/port/memory is otherwise present on the ordinary allocation/open failure paths.

<a id="disk-243"></a>

### DISK-243 — WD33C93 build lists a C file containing a raw diff header

**Confidence:** high, source/build-definition evidence; no build performed. **Applicability:** kernel-hidd-scsiwd33c93 target. **Impact:** this optional hardware backend cannot compile through the listed source set.

`rom/hidds/scsi/wd33c93/mmakefile.src:5–9` includes wd33c93 in FILES. The entire `wd33c93.c:1` is an un-commented `diff --git a/rom/hidds/scsi/wd33c93/wd33c93_busclass.c b/rom/hidds/scsi/wd33c93/wd33c93_busclass.c` line, not C code. This is separate from DISK-138's generic SCSI include capitalization defect.

<a id="disk-244"></a>

### DISK-244 — PPC disk-image endian stores declare written memory as input only

**Confidence:** high for the compiler-contract violation; actual miscompilation depends on optimization/context. **Applicability:** GNU C with __PPC__ defined, including standalone powerpc-aros build selecting that compiler macro. **Impact:** compiler can retain stale knowledge about buffers after byte-swapped stores, producing incorrect decoded samples/image data. No generated assembly or runtime failure is claimed.

`workbench/devs/diskimage/include/endian.h` GNU PPC wswap16/wswap32/wswap64 implementations use sthbrx/stwbrx but describe destination memory only as input `Z` operands, with empty output lists and no memory clobber. The implementation writes those locations without declaring the modification. [GCC extended-asm documentation](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html) distinguishes input reads from output modifications and explains that the compiler does not interpret assembler instructions to infer their effects. It also states no-output asm is implicitly volatile: the earlier potential elimination explanation is corrected; implicit volatility does not declare the memory write.

On big-endian PPC, wle16 delegates to wswap16; active audio callers include mp3_mpega.c:117–118/165–166 and wavpack.c:222–223. The standalone Makefile.ppc-aros explicitly selects powerpc-aros-gcc with -O3. Native-endian wbe32 on big-endian PPC uses a C assignment, so do not extend this finding to every wbe32 use merely because it appears in the same header. Macro selection remains conditional on the toolchain defining __PPC__; no build was run to verify predefined macros.


Final macro check: arch/ppc-all/include/aros/cpu.h defines __PPC__, corroborating the GNU/PPC selection when that target header is included. No generated machine code was inspected.

<a id="disk-245"></a>

### DISK-245 — Disk-image name query checks the output pointer instead of allocation result

**Confidence:** high. **Trigger:** DITAG_GetImageName with a valid STRPTR* output for an inserted image, ASPrintf allocation fails. **Impact:** UnitInfo/control returns success with NULL filename, making allocation failure indistinguishable from an empty unit to callers.

`workbench/devs/diskimage/device/unit.c:256–260` assigns the ASPrintf result through ti_Data but tests !ti_Data instead of the newly assigned result. A valid caller pointer is nonzero even when the allocation failed. UnitInfo (`device/main_vectors.c:123–133`) supplies an error output and returns it, but this branch never sets it on allocation failure. This does not claim a NULL caller output pointer is supported; the defect occurs with a valid output pointer.

<a id="disk-246"></a>

### DISK-246 — Disk-image mount persists an uninitialized path after name-query failure

**Confidence:** high for the source failure path; occurrence depends on a successful mount despite a failed path query. **Trigger:** Open and name allocation succeed, NameFromFH fails before initializing its output, then plugin opening succeeds. **Impact:** preference string creation scans uninitialized stack data, potentially beyond fullpath, and stores a garbage path. No deterministic leaked contents or exact crash address is asserted.

`workbench/devs/diskimage/device/unit.c:218` declares uninitialized fullpath[512]. InsertDisk ignores NameFromFH's return (559–561), independently proceeds to plugin opening, and the successful-image branch later calls AllocPrefsString(fullpath) (227–229). `rom/dos/namefromfh.c:49–72` can fail allocating its FileInfoBlock or obtaining ParentOfFH before writing any output; its documented NUL guarantee applies only on success. Thus this is not dependent on assuming every too-long-name failure leaves an unterminated string. A transient failure in that preliminary DOS query followed by successful plugin allocation establishes the candidate. Separate from DISK-188's XML escaping and DISK-240's write failure.

Disk-image control reconciliation: the previous unnumbered name-query and path-persistence observations are resolved as DISK-245/246.

<a id="disk-247"></a>

### DISK-247 — USB CDB construction stores ULONG through unaligned byte-array pointers

**Confidence:** high for absent C alignment guarantee; hardware fault/miscompilation depends on target and generated instructions. **Applicability:** ROM mass-storage on targets requiring aligned ULONG accesses; not an assertion that x86 faults. **Trigger:** normal READ/WRITE(10), emulated block IO, SEEK(10), or high-LBA READ/WRITE(16) construction. **Impact:** an alignment fault before command submission is possible.

`rom/usb/classes/massstorage/massstorage.class.c` declares CDB storage as UBYTE arrays and casts addresses at byte offsets2/6 to ULONG*. Stores occur at 2711/2734, 2820/2863, 2963–2964/2977, 3040 and 3133–3134/3147. These are ordinary ULONG pointer dereferences, with no packed structure/type, memcpy, bytewise encoding or explicit unaligned-access adapter to constrain compiler behavior. Arrays have no guaranteed four-byte alignment; even a four-byte-aligned base makes offset2 misaligned. Re-reading module mmakefile.src finds no special packed-access configuration. riscv64-opensbi/boot/mmakefile.src:92 includes this class, establishing an actual non-x86 build consumer, without claiming every RISC-V implementation rejects misalignment.

The seek path independently has DISK-210's uninitialized scsi_Command pointer; aligned normal READ(10) requests establish this candidate without depending on that prior defect. No build, disassembly or hardware test performed. This consolidates the earlier two wire-buffer alignment observations into one conditional architecture candidate.

<a id="disk-248"></a>

### DISK-248 — USB change-notification traversal races with handler removal

**Confidence:** high for absent traversal/lifetime synchronization; occurrence requires scheduling overlap. **Trigger:** a client removes and releases its registered change-interrupt request while the removable poller or ejecting worker holds that request pointer. **Impact:** freed-request/list-pointer reads or invocation of a released interrupt handler. Applicable even to a preemptive single-CPU build, without assuming SMP.

`rom/usb/classes/massstorage/dev.c:215–229` mutates ncm_DCInts under Forbid/Permit and completes TD_REMCHANGEINT after Remove. In contrast, the media poller traverses the same caller-owned IO requests and dereferences io_Data/ln_Succ (`massstorage.class.c:4026–4031`) without holding Forbid or a shared lifetime lock. The worker's ejection traversal (1419–1424) does likewise. If preempted after loading a node, a removing client can complete removal and release/reuse its request before traversal resumes. Forbid inside the mutating task prevents that task being preempted; it does not retrospectively protect pointers retained by another task.

No blanket claim that Cause executes concurrently or that a client may free a still-registered handler. The supported remove-then-release sequence is sufficient. A fix must protect both list traversal and handler lifetime, not only list insertion. This resolves the earlier unnumbered notification ownership observation; no concurrent execution test was run.

<a id="disk-249"></a>

### DISK-249 — SD 32-bit reads compare against a truncated sector capacity

**Confidence:** high for arithmetic; applicability is the maximum representable CSD-v2 sector count. **Trigger:** recognized CSD-v2 card with C_SIZE0x3fffff, yielding2^32 sectors, ordinary nonempty CMD_READ within its first4GB of byte addresses. **Impact:** every such 32-bit read is rejected as outside the disk, despite its address being valid; 64-bit read dispatch uses the full capacity. No hardware of that size was tested.

`rom/devs/sdcard/sdcard_bus.c:431–444` computes cylinders as (C_SIZE+1)*512, then multiplies by2 sectors after conversion toUQUAD. Maximum22-bit C_SIZE therefore produces2^32 sectors without an intermediate capacity wrap. `sdcard_device.c:79` compares block+count against only the low32 capacity bits, which are zero at this boundary. Read64 at135 and Write32 at189 instead compare against full sdcu_Capacity. This is not a claim that a 32-bit byte offset can address the entire card: even block0/count1 is incorrectly rejected. Geometry's public32-bit saturation is a separate representational limit.

This resolves the earlier extreme-capacity observation. SD command argument width/addressing above2^32 blocks remains separate and is not needed for this failure.

<a id="disk-250"></a>

### DISK-250 — NVMe namespace sector exponent is used without a supported-range check

**Confidence:** high for unchecked arithmetic; trigger requires unsupported/inconsistent controller Identify data, not ordinary512/4096-byte formats. **Impact:** undefined shifts during discovery/IO and invalid geometry/transfer counts, instead of refusing an unsupported namespace.

`rom/devs/nvme/nvme_busclass.c:500` masks flbas to a16-entry index but does not validate the selected format against nlbaf or validate its ds. Discovery computes `nsze << (ds - 9)` (550/553/573) and stores ds as au_SecShift (571). With ds below9, the shift count is negative after integer promotion. With ds32 or higher, nvme_io.c:57 right-shifts ULONG len by at least its32-bit width; that calculation precedes ordinary IO validation. The driver needs a supported exponent range for its own types regardless of whether a controller normally reports valid format descriptors. Separate from DISK-085, which occurs with an ordinary valid4KB exponent. No malformed controller simulation performed.

Namespace format reconciliation: active flbas masking prevents a simple sixteen-entry array-index overflow. The nlbaf enumeration reads entries only inside disabled debug D() expressions in the ordinary configuration; do not promote it as an unconditional runtime out-of-bounds read. Metadata-bearing formats are not explicitly rejected and IO supplies no metadata support, but exact behavior depends on format/protection settings; retain an unsupported-format limitation rather than assert silent corruption without a protocol trace.

<a id="disk-251"></a>

### DISK-251 — VirtIO configuration is read once without generation validation

- **Severity:** Medium (incorrect/stale geometry and capacity). **Confidence:** High for missing mechanism; impact requires concurrent configuration change or resize.
- **Source:** `rom/devs/virtio/virtio_hw.c:249–286`, `virtio_controllerclass.c:100`, `virtio_busclass.c:43–63,152–161`; public common-config declaration includes config_generation.
- **Trace:** The sole call reads capacity and optional geometry/block size independently without reading or retrying against config_generation. The interrupt routine reads and discards the ISR byte and signals only the used-queue worker; that worker never refreshes device configuration or the unit's capacity snapshot.
- **Trigger/impact:** Host configuration changes during attachment can produce a mixed snapshot; a capacity resize after attachment leaves the exposed unit capacity unchanged. On a target where the 64-bit capacity load uses multiple accesses, a concurrent update can additionally yield a torn value. Stable configurations do not demonstrate this failure, and no hardware execution is claimed.
- **Contract:** [OASIS VIRTIO 1.2](https://docs.oasis-open.org/virtio/virtio/v1.2/virtio-v1.2.html), device-configuration-space provisions, defines generation checking for consistent configuration reads. This is separate from existing static logical-sector scaling and transfer-limit candidates.

<a id="disk-252"></a>

### DISK-252 — UAS command and status structures disagree with the implemented wire protocol

- **Severity:** High (UAS interoperability failure or incorrect status). **Confidence:** High from comparison with an independent primary implementation; no device execution.
- **Source:** `compiler/include/devices/usb_massstorage.h:99–130`; `rom/usb/classes/massstorage/massstorage_uas.c`, nUasDoCommand.
- **Evidence:** [Linux's maintained UAS declarations](https://raw.githubusercontent.com/torvalds/linux/master/include/linux/usb/uas.h) put a big-endian 16-bit tag at bytes 2–3, task attributes at byte 4, status at byte 6, and sense length at bytes 14–15 followed by sense at byte 16. AROS places task attributes at byte 3, a little-endian 32-bit command tag at bytes 4–7, status at byte 2, and sense length/data at bytes 8/12. The command builder and result parser actively use those declarations.
- **Trigger/impact:** A UAS device implementing the independent wire layout receives tag zero from the command's zeroed bytes 2–3 while the intended tag occupies other command fields. Returned status is read from a tag byte instead of its actual status field; sense copying starts at the wrong offset and decodes the wrong length. Thus ordinary UAS commands/status handling cannot be assumed correct. This is a wire-format root cause separate from DISK-035's failure to validate completion identity/extent.
- **Limits:** The comparison establishes disagreement with a maintained primary UAS implementation, not hardware reproduction or independent verification of every UAS revision. The earlier deferred wire-layout observation is now promoted on this evidence; bulk and CBI do not use these IU structures.

<a id="disk-253"></a>

### DISK-253 — Legacy Expat clib prototypes omit parameter types

- **Severity / confidence:** medium conditional build incompatibility / high confidence in declaration mismatch; compiler outcome unverified.
- **Source:** workbench/devs/diskimage/include/amigaos/clib/expat_protos.h:18–21, 61, 65 and 70; include/amigaos/proto/expat.h:7–9; sfd/expat_lib.sfd.
- **Trigger / affected configuration:** a non-GCC legacy client, including VBCC, includes proto/expat.h and consumes the checked-in clib header before successful regeneration. plugins/dmg/dmg.c:39–42 follows that path on non-AROS builds.
- **Evidence / impact:** declarations such as XML_ParserCreate(const encodingName) omit XML_Char *; Create_MM similarly omits three pointer types, and SetBase, SetEncoding and ParserReset repeat the omission. The SFD declares the correct pointer types. A conforming modern compiler must diagnose the missing type; a legacy implicit-int extension interprets the parameter as an integer, losing the source-level pointer contract and potentially rejecting normal pointer arguments. This is stronger than the intentional LONG pointer representation in other 32-bit compatibility headers. No runtime corruption is asserted.
- **Limits:** the standalone Makefile has fd2pragma generation rules capable of replacing these headers when dependencies require it, so failure depends on whether the checked-in header is consumed or regenerated and on compiler acceptance. Modern AROS DMG uses expat.h directly and is outside this trigger. No compiler, build or test was run.

<a id="disk-254"></a>

### DISK-254 — HDTool passes uninitialized autosense fields to ATA emulation

- **Severity / confidence:** high conditional memory corruption / high confidence in source trace; stack contents and actual runtime outcome unverified.
- **Source:** workbench/c/HDTool/deviceio.c:63–77; list.c printHD/listHDs; rom/devs/ata/ata.c:483–504; rom/devs/ata/scsiemu.c:160–161, 237–260.
- **Trigger:** listing a direct-access ATA unit through HDTool, with stack garbage in scsicmd.scsi_Flags selecting old autosense, or selecting ordinary autosense with a nonzero garbage sense length.
- **Evidence:** identify initializes only four fields of an automatic SCSICmd and submits opcode 0xEC. ATA dispatch reaches SCSIEmu. Its unknown-command error creates 32 bytes of sense; old autosense selects four bytes regardless of supplied sense length, while ordinary autosense uses the uninitialized length. CopyMem writes the selected bytes through uninitialized scsi_SenseData before returning the failure. Thus command rejection does not prevent the invalid write.
- **Impact / applicability:** crash or overwrite at a garbage destination, depending on actual flags, length and pointer. This is a client request-initialization defect distinct from driver bugs accepting a properly formed short sense buffer. Applies to this HDTool listing path and ATA direct-access emulation; no successful IDENTIFY transfer or damage to disk contents is claimed. No runtime reproduction, build or test performed.

<a id="disk-255"></a>

### DISK-255 — Partition command discards table-write failures

- **Severity / confidence:** high conditional persistence correctness / high source confidence.
- **Source:** workbench/c/Partition/main.c:592–613; rom/partition/writepartitiontable.c; closepartitiontable.c.
- **Trigger / evidence:** any nonzero return from a final child/container/root WritePartitionTable is ignored. The command closes and discards changed in-memory tables, continues publishing ancestor tables, prints completion and returns success with error still zero. The library expressly returns write errors and writes only the selected table.
- **Impact / limits:** a failed child write can leave successfully published ancestors referring to unpersisted child metadata; other failures silently lose requested changes. Exact resulting on-disk bytes depend on which writes succeed and the prior layout. No atomicity guarantee or successful rollback is claimed. Applies to the command's final save path, independently of driver-specific error-generation defects; not runtime-reproduced.

<a id="disk-256"></a>

### DISK-256 — Partition command reads pointer-width attributes written as LONG

- **Severity / confidence:** high conditional placement correctness / high source confidence.
- **Source:** workbench/c/Partition/main.c:688, 701–718; rom/partition/partitionmbr.c:333–335; partitionebr.c:361–363.
- **Trigger / evidence:** CreateMBRPartition uses IPTR reserved (uninitialized) and leadIn (zeroed), but table handlers write only LONG bytes. On a 64-bit target reserved retains unwritten bytes if subsequently used by lowcyl == 0. For EBR leadIn, a big-endian 64-bit LONG write sets the upper half; the full-width nonzero value then adjusts lowcyl incorrectly. Little-endian zero-initialized leadIn does not have that particular failure.
- **Impact / limits:** garbage or scaled reserved/lead-in values distort a proposed start cylinder and can produce invalid placement. MBR/EBR creation lacks a universal range/disjointness guard, but the resulting exact wrapped range depends on geometry and truncation. Active default MBR creation generally supplies a nonzero lowcyl, so uninitialized reserved requires the lowcyl-zero branch; do not call it unconditional. Relevant 64-bit big-endian EBR execution is an architecture-conditional path, not verified deployment. Distinct from DISK-059's existing-address narrowing; no code executed.

<a id="disk-257"></a>

### DISK-257 — Partition command dereferences a missing extended container on a full MBR

- **Severity / confidence:** medium command crash / high source confidence.
- **Source:** workbench/c/Partition/main.c:373–438, 458–475; rom/partition/partitionmbr.c:261–316.
- **Trigger:** partitioning an existing MBR with four occupied primary slots and no EBR extended partition, without WIPE.
- **Trace:** enumeration sets partCount to four. Attempted extended-container creation returns NULL because the MBR add handler finds no unused slot. The command leaves error zero; partCount == 4 then selects parent = extPartition, still NULL, and dereferences parent->de during cylinder conversion.
- **Impact / limits:** command crash instead of reporting no available slot. This trace does not require allocation failure or corrupted metadata, and occurs before the final save phase; no disk write from this particular failure is asserted. A related EBR-table creation failure can also leave extPartition NULL after incrementing the count, but exact other-count selection depends on gaps. Source-only, not reproduced.

<a id="disk-258"></a>

### DISK-258 — Pure-RDB MAXWORK can omit the requested work partition

- **Severity / confidence:** medium incorrect partition layout / high source confidence.
- **Source:** workbench/c/Partition/main.c:249–252, 485–490, 534–558; args.c MAXWORK/WORKSIZE definitions.
- **Trigger:** SCHEME rdb with a positive SYSSIZE and MAXWORK, no WORKSIZE, and enough free space for Work that remains below the selected work filesystem's maximum-size cap.
- **Trace:** workSize starts zero and is assigned from WORKSIZE only when present, or from the maximum-size cap when exceeded. MAXWORK's ordinary uncapped path leaves it zero. The pure-RDB branch creates Work only when workSize != 0, so it silently omits Work while persisting System and reporting completion.
- **Impact / limits:** requested free-space Work partition is absent; no inherent corruption of System is claimed. Capped MAXWORK can set a nonzero workSize and take another path, so this is not every MAXWORK invocation. The nested non-RDB branch uses a different condition and is outside this specific omission. No code executed.

<a id="disk-259"></a>

### DISK-259 — Installer FFS collectors probe a node past the boot-sector buffer

- **Severity / confidence:** High conditional memory safety; high source confidence.
- **Source:** C:Install collectBlockList:174–179, stage2_firstblock[128] and installStageFiles call; Install-grub.c:713–718/changeStage2; Install-grub2.c:867–873/updateCoreIMG.
- **Trigger / trace / impact / limits:** Each collector starts blk_count=0 and reads node zero before decrementing, while callers supply a pointer one past the first 512-byte sector. C:Install uses an exact 512-byte static array; GRUB installers use sector-size boot buffers, normally 512. The initial read exceeds those allocations on 512-byte geometry. Adjacent garbage matching a contiguous run can additionally cause an out-of-bounds count write. Larger buffers limit allocation-bound claims but still consult bytes outside the initialized list. This is one shared root cause, not three candidates; no malformed filesystem is required, runtime outcome unverified. Source-only; no build or execution.

<a id="disk-260"></a>

### DISK-260 — GRUB boot-volume helpers dereference failed volume creation

- **Severity / confidence:** Medium command crash; high source confidence.
- **Source:** Install-grub.c:629–631; Install-grub2.c:692–694; their initVolume and main callers.
- **Trigger / trace / impact / limits:** After partition validation succeeds, initVolume can return NULL on allocation, request/port creation or OpenDevice failure. getBBVolume immediately assigns volume->partnum and accesses its buffer without checking. Both active main callers reach this helper. The trigger is the failed initialization, not normal successful boot-volume creation. This groups the copied helper defect; no disk corruption claimed. Source-only; no build or execution.

<a id="disk-261"></a>

### DISK-261 — Boot installers return success for failed installation paths

- **Severity / confidence:** Medium failure reporting; high source confidence.
- **Source:** workbench/c/Install.c main; Install-grub.c main; Install-grub2.c main.
- **Trigger / trace / impact / limits:** C:Install and the older GRUB installer always return zero despite argument/open/read/write/install errors. GRUB2 initializes RETURN_OK and misses argument failure, absent startup lookup and failed stage-volume creation, although final publication failure does set RETURN_ERROR. Scripts cannot reliably detect those failed attempts. Group by lost error propagation, preserving each distinct covered branch; no claim that every GRUB2 error is swallowed. Source-only; no build or execution.

<a id="disk-262"></a>

### DISK-262 — C:Install replaces bootblock after failed preservation write

- **Severity / confidence:** High conditional data loss; high source confidence.
- **Source:** workbench/c/Install.c:236–245, 295–302; VF_MOVE_BB setup in initVolume.
- **Trigger / trace / impact / limits:** For a detected filesystem signature in block zero on the non-MBR path, installStageFiles first writes its saved block to block one but ignores the return. It continues collecting/patching stages and can successfully replace block zero with stage1 even if preservation failed. The original boot/filesystem header can then be lost without its intended relocated copy. Requires the failed block-one write and subsequent successful publication; a permanently failing disk may instead stop later. Distinct from raw-driver corruption and table-write error reporting; not reproduced. Source-only; no build or execution.

<a id="disk-263"></a>

### DISK-263 — Partition and installer I/O retains ordinary commands after required 64-bit negotiation fails

- **Severity / confidence:** high conditional wrong-region I/O / high confidence in fallback and consumer semantics; driver/query failure trigger conditional.
- **Source:** rom/partition/partition_support.c PartitionNsdCheck; openrootpartition.c:60–61; readpartitiondata.c/writepartitiondata.c; workbench/c/Install.c nsdCheck/readwriteBlock; arch/all-pc/boot/grub-tools/Install-grub.c nsdCheck/readwriteBlock; grub2-tools/Install-grub2.c:193–231, 308–321.
- **Trigger:** an actual request has a byte offset >=2^32, the capacity test does request NSD support, but the device query fails, returns rejected geometry/type/accounting, or omits the needed 64-bit read/write command.
- **Trace:** defaults remain CMD_READ/CMD_WRITE. The negotiation helpers return no fatal status, and callers continue raw I/O with low offset in io_Offset and high bits in io_Actual. Ordinary ATA cmd_Write32 reads only io_Offset, whereas cmd_Write64 explicitly combines io_Actual (ata.c:167–190, 222–236). Hosted Host_Seek likewise supplies a zero high half. Thus ordinary-command fallback has no generally valid high-offset interpretation. Some backends may reject the command or apply extra conventions, so wrong-region completion is conditional, not every device failure.
- **Impact / grouping:** a high-offset write can alias its low 32-bit region instead of failing safely; a read can silently use the wrong location. Unlike DISK-060, this path requires no arithmetic overflow and persists after widening the threshold. Group the shared unchecked negotiation contract across the library and three copied installer helpers. Query-error responses other than IOERR_NOCMD in the installer helpers are treated as potential successful responses; their uninitialized-field access requires a misleading matching io_Actual/SizeAvailable and remains a separate limited observation. Source-only, no I/O performed.

<a id="disk-264"></a>

### DISK-264 — BHFormat device listing retains a pointer to an expired InfoData local

- **Severity / confidence:** medium conditional display corruption/crash / high source confidence.
- **Source:** workbench/system/BHFormat/gui.c:255–283; ComputeCapacity:207–232.
- **Trigger / trace:** a listed device successfully locks and Info succeeds; a matching volume is found. pInfoData points to infoData declared inside the if(lock) block. That object expires at the block's end before ComputeCapacity reads its block counts; subsequent SFormatEntry construction can reuse the storage.
- **Impact / limits:** invalid lifetime reads can misreport usage or feed a zero/garbage id_NumBlocks to the unchecked percentage division. A missing volume or NULL pInfoData avoids the affected ComputeCapacity branch. This is the GUI listing contract, not disk I/O or damage to media. No runtime reproduction or specific compiler stack reuse asserted.

<a id="disk-265"></a>

### DISK-265 — BHFormat CLI proceeds after failed or exhausted confirmation input

- **Severity / confidence:** high conditional unintended formatting / high source confidence.
- **Source:** workbench/system/BHFormat/cli.c prompt loop before bGetExecDevice.
- **Trigger / trace:** the insert-disk prompt reads a non-Ctrl-C/non-Return character, then the next Read returns EOF/error. The loop ends because cch != 1; the only subsequent cancellation test is ch == 3. The remembered other character passes it and execution proceeds to full or quick formatting without Return confirmation. EOF before any successful character also leaves ch uninitialized.
- **Impact / limits:** the command can perform its already-selected destructive operation after failed confirmation input. Requires the caller to reach the prompt with valid device/name arguments and supply exhausted/failing input; an ordinary interactive Return follows the intended path. Distinct from permissions or device write protection, which can still reject I/O. Source-only, no formatting performed.

<a id="disk-266"></a>

### DISK-266 — BHFormat uses 64-bit commands at an unqueried exactly-4-GiB boundary

- **Severity / confidence:** Medium conditional partial-format failure; high source confidence.
- **Source:** workbench/system/BHFormat/main.c:504, 714–720.
- **Trigger / trace / impact / limits:** The device query is conditional on exclusive ibyEnd > 2^32, but the transfer loop chooses a 64-bit command on offset+length >= 2^32. A partition ending exactly at that boundary skips the query and uses FORMAT64/READ64 for its final chunk. A device supporting only ordinary commands can reject that chunk after earlier formatting. This is a boundary mismatch, not an asserted wrong-region transfer: the last chunk starts below 2^32. No runtime reproduction. Source-only; no build or execution.

<a id="disk-267"></a>

### DISK-267 — BHFormat GUI reads an uninitialized volume-name string

- **Severity / confidence:** Medium conditional memory disclosure/crash; high source confidence.
- **Source:** workbench/system/BHFormat/gui.c:487, Workbench named-device branch, volume initialization and later MSG_VOLUME formatting.
- **Trigger / trace / impact / limits:** A Workbench device argument with wa_Lock==0 initializes the device name but not local szVolumeName; pdlVolume remains NULL so the later conditional copy also skips it. GUI text formatting subsequently consumes szVolumeName as a C string. Stack bytes can be displayed and the read may pass the array if no NUL is encountered. Volume-lock and selection-dialog paths initialize it. This is display-string ownership, not on-disk data disclosure; exact stack output unverified. Source-only; no build or execution.

<a id="disk-268"></a>

### DISK-268 — BHFormat GUI exits success after a failed format callback

- **Severity / confidence:** Medium erroneous failure reporting; high source confidence.
- **Source:** workbench/system/BHFormat/gui.c btn_format_function:472–477; rcGuiMain:709–710.
- **Trigger / trace / impact / limits:** The callback computes local failure/success state and prints relevant errors, then always requests the same Quit ID. It never propagates its rc. rcGuiMain sets RETURN_OK after application execution regardless of format/read/verification/filesystem failure in the callback. Thus automation or launchers receive success for those GUI operation failures. Initialization failures before execution still retain RETURN_FAIL; do not assert every GUI failure is swallowed. Distinct from DISK-261 installer paths, same error-reporting class. Source-only; no build or execution.

<a id="disk-269"></a>

### DISK-269 — Partition WIPE submits an unchecked buffer and ignores erasure failures

- **Severity / confidence:** High conditional invalid I/O/persistence behavior; high source confidence.
- **Source:** workbench/c/Partition/main.c:874–882; rom/partition/writepartitiondata.c.
- **Trigger / trace / impact / limits:** A failed 4096-byte AllocMem returns NULL, but recursive WIPE passes it directly to WritePartitionDataQ. The helper forwards it as io_Data without checking. Driver behavior can reject or dereference it, so a crash/invalid source read is conditional on backend handling. Independently, a failed erasure write is ignored while the command continues destroying the partition table. This can report successful WIPE with stale recognizable filesystem headers retained; destruction of the table itself remains the explicitly requested operation. Fixed 4096 bytes must also satisfy the active sector-size multiple contract. Group the unchecked WIPE preparation/error path here; do not claim every driver dereferences NULL or that WIPE promises secure erasure. Source-only; no build or execution.

<a id="disk-270"></a>

### DISK-270 — HDTool queries active status for bootable and automount displays

- **Severity / confidence:** Low incorrect diagnostic output; high source confidence.
- **Source:** workbench/c/HDTool/list.c:88–104; rom/partition/include/libraries/partition.h:89–92.
- **Trigger / trace / impact / limits:** For supported PTA_BOOTABLE/PTA_AUTOMOUNT attributes, printPartitionInfo passes PT_ACTIVE rather than the corresponding property tag. These three tags are distinct. If active is supported, both displayed values can repeat the active state; if it is unsupported and its destination is left untouched, the uninitialized local can be printed. Trigger is partition-detail listing; no write occurs. Distinct from parser corruptions and other unit-enumeration limitations. Source-only; no build or execution.

<a id="disk-271"></a>

### DISK-271 — BHFormat leaks the device-list read lock when a Workbench volume has no matching device

- **Severity / confidence:** Medium conditional resource/concurrency failure; high source confidence.
- **Source:** workbench/system/BHFormat/gui.c:529–541, cleanup at 713; workbench/system/BHFormat/main.c:284–404, FreeAll.
- **Trigger / trace:** The Workbench volume argument branch acquires LockDosList(LDF_DEVICES | LDF_READ), searches for a device with the volume lock's handler task, and jumps directly to cleanup if no device matches. That jump bypasses bGetDosDevice, which releases the already-held lock on the normal path. Neither GUI cleanup nor FreeAll releases this lock.
- **Impact / limits:** This error path leaves an acquired DOS list read lock unreleased and can interfere with later list mutation. Requires a volume lock whose task lacks a corresponding listed device; ordinary matching-volume startup reaches the releasing helper. Exact task-exit recovery and resulting blocking duration have not been reproduced. Source-only; no build or execution.

<a id="disk-272"></a>

### DISK-272 — QuickPart divides by zero when every enumerated drive lacks usable capacity

- **Severity / confidence:** Medium conditional startup failure; high source confidence.
- **Source:** workbench/tools/QuickPart/QP_FindDisks.c unitenumFunc and FindDisks.
- **Trigger / trace:** Fixed/SSD/RAID units are added to FoundDrives even when OpenRootPartition fails. Their cleared size stays zero and qped_DriveLargest remains zero if no drive produces positive capacity. With a nonempty list, FindDisks evaluates (newDrive->qpfd_Size * 100) / qpED.qped_DriveLargest without a divisor guard.
- **Impact / limits:** An entirely inaccessible or zero-capacity enumerated drive set can fail GUI startup through integer division by zero. No drives means no loop and avoids this expression; at least one positive capacity prevents this root cause. This is enumeration/display failure, not an on-disk write. Source-only; no runtime reproduction.

<a id="disk-273"></a>

### DISK-273 — QuickPart disk objects omit device and root-partition ownership cleanup

- **Severity / confidence:** Medium conditional resource exhaustion; high source confidence.
- **Source:** workbench/tools/QuickPart/QP_ccDisk.c QPDisk__OM_NEW, QPDisk__MUIM_QPart_Disk_ScanForParts, QPDisk__OM_DISPOSE.
- **Trigger / trace:** OM_NEW successfully opens the device to construct the object but never closes that open. ScanForParts opens the same request again and closes only its own scan open, then retains a newly opened root partition handle and possibly its open table. OM_DISPOSE deletes the request and port without CloseDevice or CloseRootPartition. A failed superclass construction after the initial successful open also lacks cleanup of the open, request and port.
- **Impact / limits:** Object disposal or failed construction leaves acquired device references/resources unreleased; successful partition scans additionally retain the partition library's root resources. Backend-specific open accounting determines the exact leak size and consequences. Do not infer a universal immediate crash or proven disk corruption. Repeated scans can also replace retained handles without releasing their predecessors; caller frequency remains to be traced in the remaining QuickPart units. Source-only; no build or execution.

QuickPart's identification helper also constructs an incompletely initialized SCSICmd, matching DISK-254's sense-buffer preparation root cause; retain this additional caller for deduplication rather than numbering the same contract violation again. Other newly observed allocation and display-buffer issues await caller/configuration classification.

<a id="disk-274"></a>

### DISK-274 — QuickPart loses trailing free-space markers when inserting a partition

- **Severity / confidence:** Medium incorrect partition-layout presentation; high source confidence.
- **Source:** workbench/tools/QuickPart/QP_ccPartitionContainer.c QPPartitionContainer__MUIM_QPart_ccPartitionContainer_AddPart.
- **Trigger / trace:** For an existing free-space node containing a partition with at least 512 sectors before its start, the insertion routine first changes the node's End to Part_Start - 1. It then tests and constructs the post-partition spacer using that already-shortened End. A normally ordered partition ending after its start cannot satisfy Part_End <= shortened_End - 512, so the original remaining space after the partition is lost from the layout. For example, free sectors 0–9999 and a partition 1000–1999 leave the pre-marker 0–999 but fail to create the expected trailing marker.
- **Impact / limits:** Valid disk layouts can display missing free space and subsequent entries are processed against an incomplete marker model. The hardcoded 512 is compared to sector-index values, not a verified byte-size adjustment. These routines manipulate GUI/internal child nodes; no disk write or overlapping on-media allocation is established here. Independent of allocation failure and malformed-media geometry. Source-only; no build or execution.

Additional bounded observations from the complete container read: InsertPartEntry checks newPartNode allocation but later unconditionally uses &newPartNode->PCCN_Node when Group_InitChange succeeds; OM_NEW dereferences its unchecked contSpacerNode allocation; geometry ratios and weight denominators lack zero checks; only the final newly created partition's nested container is recursively scanned. Each remains pending grouping/caller/configuration classification rather than automatically treating all as independent confirmed defects.

<a id="disk-275"></a>

### DISK-275 — QuickPart subtracts a sector from every leaf partition capacity

- **Severity / confidence:** Low incorrect capacity display; high source confidence.
- **Source:** workbench/tools/QuickPart/QP_ccPartition.c QPPartition__OM_NEW leaf-partition branch.
- **Trigger / trace:** The byte-size expression computes (((HighCyl - LowCyl + 1) * Surfaces * BlocksPerTrack) - 1) * SectorSize. The inclusive cylinder interval already yields the sector count; subtracting one changes the count into the final relative sector index. Thus an N-sector leaf is displayed using (N-1) sectors before unit scaling.
- **Impact / limits:** Capacity labels and selection details understate the partition by one logical sector. Coarse unit rounding can hide the discrepancy for large partitions. This is distinct from native-width product overflow and does not affect raw partition I/O or write placement in these classes. Source-only; no build or execution.

<a id="disk-276"></a>

### DISK-276 — QuickPart dereferences failed partition-display node allocations

- **Severity / confidence:** Medium conditional GUI crash; high source confidence.
- **Source:** workbench/tools/QuickPart/QP_ccPartitionContainer.c InsertPartEntry and QPPartitionContainer__OM_NEW.
- **Trigger / trace:** OM_NEW allocates contSpacerNode and immediately assigns its fields without checking NULL. Independently within the same display-node preparation family, InsertPartEntry conditionally initializes newPartNode only when AllocVec succeeds, but a subsequent successful Group_InitChange unconditionally passes &newPartNode->PCCN_Node to AddTail. Both are reached while constructing/scanning partition layouts.
- **Impact / limits:** Memory pressure can turn recoverable display-node allocation failures into invalid pointer writes/list insertion and a crash. The InsertPartEntry path additionally requires Group_InitChange to succeed. These are GUI/internal list allocations; no raw disk write is established. Group the common missing failure handling here rather than multiplying findings for every field dereference. Source-only; no build or execution.

QuickPart remaining normalization observations: disk/model string and locale-hook allocations have additional unchecked consumers; locally allocated strings/hooks/child-node records lack disposal; UUIDBase acquisition is unchecked before selection's UUID_Unparse, although color mapping checks it; custom-class/attribute-base cleanup and GUI construction failure unwinding are incomplete; partition selection's tick event handler is not explicitly removed in MUIM_Cleanup/OM_DISPOSE. These require framework/ownership contracts before assigning stronger consequences. The native-width geometry products and hardcoded-sector spacer thresholds remain separately documented conditional arithmetic/layout observations.

<a id="disk-277"></a>

### DISK-277 — HDToolBox leaves a freed node linked after partition creation fails

- **Severity / confidence:** Medium conditional use-after-free/list corruption; high source confidence.
- **Source:** workbench/tools/HDToolBox/partitions.c addPartition, freePartitionNode, freePartitionList; workbench/tools/HDToolBox/gui.c addpartitiongadgets.ok callback.
- **Trigger / trace:** addPartition allocates a UI partition node and immediately Enqueue/AddTail links it into table->listnode.list. If the later AddPartitionTags returns NULL (allocation failure, full/unsupported table or another rejected creation), it calls freePartitionNode without Remove. That function frees node resources and the node itself but does not unlink it. The caller reports failure and leaves the owning table list live.
- **Impact / limits:** Subsequent traversal, validation or cleanup of that table list accesses a freed node; cleanup's freePartitionList can additionally attempt to remove/free it again. Requires a library creation failure after successful UI-node allocation. Ordinary successful creation keeps a live linked node. This is UI ownership corruption; no successful on-media creation is required, and no universal disk corruption is asserted. Source-only; no build or execution.

Pending bounded observations from these reads: validValue excludes an existing partition's last sector from its occupied interval and validates individual endpoint values rather than whole ranges; addPartition copies its UI DosEnvec before adjusting the caller's LowCyl for lead-in, but on success refreshes that copy from partitionlib (so no persistent stale-copy claim without failure-specific evidence); makePartitionTable destroys the old in-memory table before replacement creation succeeds; SCSI identify does not check scsi_Actual before consuming response fields; type-escape helpers have no destination-capacity parameter. GUI/handler contracts must classify which configurations make these observable.

<a id="disk-278"></a>

### DISK-278 — HDToolBox endpoint-only resize validation allows overlapping partitions

- **Severity / confidence:** High conditional overlapping on-media layout; high source confidence.
- **Source:** workbench/tools/HDToolBox/partitions.c validValue; workbench/tools/HDToolBox/gui.c lowcyl/highcyl/totalcyl callbacks, resize OK and saveChanges; rom/partition/partitionmbr.c PartitionMBRSetPartitionAttrs.
- **Trigger / trace:** Editing HighCyl checks only the proposed final sector with validValue, which asks whether that one value lies inside another partition. It does not test the interval between the retained LowCyl and new HighCyl. On a disk large enough, existing A at cylinders 10–19 and B at 30–39 can accept A.HighCyl=50: the new endpoint is past B and in the disk bounds, yet the new A interval contains B. Resize OK submits this environment; the MBR setter encodes it without disjointness checks, and a subsequent successful Save publishes it. LowCyl/total-cylinder edits share the endpoint-only approach.
- **Impact / limits:** User edits can create overlapping partition entries, allowing later filesystem writes through one partition to overwrite another. Requires the user to accept and save the edited layout; no resize data migration is claimed. Even a correct single-endpoint occupied-sector comparison would not prevent the spanning example. Existing last-sector exclusion is an additional boundary weakness in this validation family. MBR publication is traced explicitly; do not assume every other handler has identical acceptance semantics. Source-only; no media operations, build or execution.

Additional resize ordering observations remain grouped for follow-up: low/high edits do not explicitly enforce LowCyl <= HighCyl, total-cylinder input accepts zero, and native-width products may wrap. Exact malformed interval encoding and other table handlers need further contract classification before extending this finding's impact beyond the demonstrated overlap.

<a id="disk-279"></a>

### DISK-279 — HDToolBox cleans up nested partition tables after their handles have been freed

- **Severity / confidence:** Medium conditional use-after-free/double cleanup; high source confidence.
- **Source:** workbench/tools/HDToolBox/gui.c GB_REMOVE_ENTRY partition branch; workbench/tools/HDToolBox/partitions.c freePartitionNode/freePartitionList; workbench/tools/HDToolBox/partitiontables.c freePartitionTable; rom/partition/deletepartition.c, partitionmbr.c PartitionMBRFreeHandle, partitionrdb.c PartitionRDBFreeHandle.
- **Trigger / trace:** Removing a partition calls DeletePartition(partition->ph) before freePartitionNode(partition). For a container with an open nested table, the partitionlib delete handler closes its nested table recursively and frees handles. The GUI node still has a non-NULL table wrapper. freePartitionNode therefore recursively cleans its GUI children and calls freePartitionTable, which calls ClosePartitionTable(node->ph) on a freed handle. Nested children with their own tables can reach the same stale-handle cleanup earlier in recursion.
- **Impact / limits:** Deleting a partition container can read freed handle state and attempt a second table cleanup if allocator reuse makes the stale table pointer nonzero. Exact crash/double-free outcome depends on reuse; the invalid handle access does not. A leaf whose GUI node has table==NULL avoids this extra ClosePartitionTable path. Requires an opened nested partition-table wrapper and invocation of the removal callback; no ordinary leaf-delete failure asserted. Distinct from DISK-277's linked node after failed creation. Source-only; no build or execution.

Settings follow-up observations retained for normalization: setters ignore returned errors while marking local edits changed; DOS environment custom boot-block logic treats de_TableSize as a count although the vector contract is a last-index value (trace consumers before assigning exact impact); type-editor escape conversion lacks a capacity bound and its edit hook merely logs input. Type-string setter writes cur[-1] for id_len==0. Deletion of plain leaf UI nodes avoids calling ClosePartitionTable because their table wrapper is NULL, limiting DISK-279 as described.

<a id="disk-280"></a>

### DISK-280 — HDToolBox writes a mountlist icon using a freed pathname

- **Severity / confidence:** Medium conditional use-after-free/wrong output path; high source confidence.
- **Source:** workbench/tools/HDToolBox/gui.c createml_function.
- **Trigger / trace:** The export callback allocates pathname, opens the new mountlist with it, then immediately FreeMem(pathname, len). After writing and closing the file, successful GetDiskObject("ENVARC:Sys/def_Mountlist") leads to PutDiskObject(pathname, icon), using the freed pathname.
- **Impact / limits:** Icon creation reads released memory as a path and can fail, use an altered destination, or read beyond the old allocation depending on reuse. Requires successful file creation and availability of the default icon. File creation itself used the still-live path; do not assert the mountlist file was originally opened at an arbitrary location. Source-only; no build or execution.

<a id="disk-281"></a>

### DISK-281 — HDToolBox exports nested partitions with an out-of-bounds unit lookup

- **Severity / confidence:** High conditional wrong-device mount configuration; high source confidence.
- **Source:** workbench/tools/HDToolBox/gui.c createml_function argu[0]; workbench/tools/HDToolBox/partitions.c newPartition/findPartitions; harddisks.h and partitions.h.
- **Trigger / trace:** newPartition sets pn->root to its immediate containing partition. For a direct disk child that object is the leading root_partition in an HDNode, so ((struct HDNode *)partition->root)->unit is valid. For a nested leaf (for example RDB inside an MBR container), root is an independently allocated HDTBPartition with no trailing HDNode.unit. Export still performs the cast and reads beyond that allocation.
- **Impact / limits:** The generated Unit field can be arbitrary or the read can fail. Mounting this exported configuration can fail or select a different existing unit with the same device name; subsequent writes then depend on the user mounting/using it and backend geometry acceptance. Direct disk children avoid the incorrect layout assumption. The unit read is independently invalid even when nearby memory happens to contain the correct number. Source-only; no mount/export execution.

Further mountlist observations pending normalization: GetStartCyl divides summed parent-sector offsets by the leaf's sectors-per-cylinder without preserving a remainder, shifting export starts for nonaligned mixed virtual geometries; its sums/products are ULONG; VFPrintf/Close/PutDiskObject errors are ignored; mountlist numeric arguments use ULONG fields while the DOS environment is native width. Scope and architecture contracts must bound these independently before claiming every export corrupts data.

<a id="disk-282"></a>

### DISK-282 — HDToolBox copies unbounded type identifiers into fixed 32-byte arrays

- **Severity / confidence:** Medium conditional memory corruption; high source confidence.
- **Source:** workbench/tools/HDToolBox/hdtoolbox_support.c strcpyESC; workbench/tools/HDToolBox/prefs.c parseType and Default parsing; workbench/tools/HDToolBox/gui.c partitiontypegadgets.hexid acknowledgment; rom/partition/include/libraries/partition.h PartitionType.
- **Trigger / trace:** PartitionType.id has 32 bytes. strcpyESC has no destination length and copies every ordinary character or decoded escape until the source NUL. Preferences permit identifiers in a 256-byte ReadItem buffer and reject only zero IDLen, never bounding the decoded identifier length to sizeof(id). parseType and Default both pass their 32-byte arrays to this helper. The interactive type acknowledgment uses the same helper; its editor hook does not constrain decoded length. A preference identifier with more than 32 ordinary characters directly overwrites neighboring fields and, with sufficient length, the allocation.
- **Impact / limits:** Edited/malformed preferences can corrupt type-node or table-node memory during startup. Interactive input can corrupt the global type-gadget structure when its accepted length exceeds 32. The preference trigger is independent of the GUI's default maximum string length. Valid short identifiers avoid the overflow. This is identifier decoding/preparation, not proof of a successful arbitrary partition-table write. Group all these callers under the same unbounded helper contract. Source-only; no build or execution.

Related length observations retained here for normalization: IDLen itself is stored without a sizeof(id) limit, allowing later findPartitionType memcmp and setTypeString reads beyond the array; setTypeString writes before its local buffer when id_len==0. These are adjacent missing identifier validation contracts with distinct read/empty-input triggers, not automatically independent reproduced defects.

<a id="disk-284"></a>

### DISK-284 — InstallAROS accepts a backup that was never opened before overwriting the original

- **Severity / confidence:** High conditional loss of undo protection; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_diskio.c BackUpFile; workbench/tools/InstallAROS/ia_stageclass.c copy_backup/copy_retry/copy_skip.
- **Trigger / trace:** BackUpFile initializes err=FALSE and only sets it on Read==-1 or Write==-1. Failure to open either the original for reading or its backup destination skips the copy but still returns !err, i.e. TRUE. With undo enabled, the caller treats that result as a valid backup, proceeds to Open(dstFile, MODE_NEWFILE) and replaces the existing file. A successful replacement then queues the prepared undo record even though the backup may never have existed.
- **Impact / limits:** A failed backup destination open can silently remove the requested rollback protection before destructive file replacement. Requires an existing target chosen for overwrite, undo enabled, backup open failure, and later successful target replacement; failed directory preparation already returns FALSE and avoids this particular path. This is installation file preservation, not a raw block-driver write. Short positive backup writes and ignored Close errors are additional incomplete-transfer hazards in this same backup validity contract, but exact backend behavior has not been reproduced. Source-only; no copying, formatting, build or execution.

<a id="disk-285"></a>

### DISK-285 — InstallAROS overflows its partition command buffer with ordinary option combinations

- **Severity / confidence:** High conditional stack corruption during destructive-command preparation; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_stageclass.c InstallStage__MUIM_Partition.
- **Trigger / trace:** tmpcmd has 150 bytes and tmparg has 100. The function constructs the command with sprintf and then unbounded strcat for EFI, scheme, sizes, names, types and WIPE. For example, DEVICE=ata.device UNIT=0, EFI enabled, RDB scheme, SYSSIZE=1024, SYSNAME="DH0", SYSTYPE=FFSIntl, WORKSIZE=1024, WORKTYPE=SFS and WORKNAME="DH1" already total 172 command characters before the terminator, without WIPE. Each is an available preparation option in this routine, and no combined-length check exists.
- **Impact / limits:** Valid short inputs and ordinary option combinations can overwrite the command's stack storage before SystemTagList. Exact crash/command alteration depends on stack layout; no arbitrary disk write outcome is asserted. Larger user-supplied device/name strings can also overflow the initial or temporary formatting buffer. Requires entering the in-progress partition method; paths that skip partitioning do not build this command. Source-only; no command execution, build or formatting.

Additional stage observations retained for normalization: NextStep always advances to Done after MUIM_IC_Install, so actual success/failure text must be traced inside that method before asserting false success; GetPartitionSize's LONG GB-to-MB multiplication can wrap and evade upper-limit comparisons for extreme inputs; allocation failures in stage startup/license/retry prompt construction are unchecked; cancellation QuitInstall only changes state while InProgress, with initialization/partition-stage state contracts still to verify.

<a id="disk-286"></a>

### DISK-286 — InstallAROS continues installation after a requested format fails

- **Severity / confidence:** High conditional unintended installation into an existing filesystem; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_stageclass.c InstallStage__MUIM_Format and InstallStage__MUIM_IC_Install; ia_diskio.c FormatPartition.
- **Trigger / trace:** MUIM_Format calls FormatPartition for selected system/work volumes. A FALSE result changes neither inst_success nor the failure UI; it merely skips the gauge update. The install method ignores DoMethod(self, MUIM_Format)'s return and proceeds with locale/core/extras copying whenever inst_success remains InProgress. A failed system format can also be hidden by a later successful work format because the local success variable is reused.
- **Impact / limits:** If formatting fails while the existing destination remains writable/mounted, installation can overwrite files on the old filesystem without the requested fresh format. A device that remains inaccessible may instead fail later copying; do not assert all format failures write successfully. Requires format selected and a relevant format/Inhibit failure, followed by an accessible target. This is a missing stage failure gate, distinct from disk-driver write errors and DISK-284's rollback protection. Source-only; no formatting, copying, build or execution.

Other completed-stage observations awaiting normalized bounds: main install end always deletes undo backups, even after failure/cancellation; environment variables and the post-install script run without an InProgress guard after earlier cancellation; normal copy Write only tests -1 and ignores short positive counts/Close failure; locale/input executable paths are swapped and opened with an initial quote but no closing quote; license/startup allocation failure cleanup is incomplete. The final Done-page display and main exit-status contract remain to be traced before asserting a universal success report.

<a id="disk-287"></a>

### DISK-287 — InstallAROS ignores Cancel in the GRUB1 filesystem compatibility warning

- **Severity / confidence:** High conditional unwanted partition operation; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_bootloader.c BOOTLOADER_PartFixUp; ia_stageclass.c EPartitioningStage.
- **Trigger / trace:** With GRUB1 selected and a non-FFS system filesystem, PartFixUp presents Continue/Cancel-partition warning and returns FALSE unless Continue is selected. EPartitioningStage invokes the function without testing its return, then proceeds to MUIM_Partition for automatic partition methods.
- **Impact / limits:** The user's cancellation choice does not stop the planned partition command, including WIPE when that method was selected. Requires GRUB1/non-FFS and an automatic partition method reaching the warning; no claim for GRUB2 or paths without that prompt. Independent of whether the external partition command subsequently succeeds. Source-only; no partition execution.

<a id="disk-288"></a>

### DISK-288 — InstallAROS marks bootloader progress complete without checking execution failure

- **Severity / confidence:** Medium false completion/unbootable installation; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_bootloader.c BOOTLOADER_DoInstall; ia_stageclass.c bootloader installation stage.
- **Trigger / trace:** After preparing C:Install-grub or C:Install-grub2, DoInstall calls Execute and discards its return, then sets the gauge to 100. It never marks inst_success failed for an execution failure. The caller receives no status because DoInstall returns void.
- **Impact / limits:** Missing/rejected bootloader execution can leave an installation without the requested bootloader while stage state remains InProgress. Existing bootloader state can still boot some systems, so universal boot failure is not asserted. Related standalone installers may themselves incorrectly return success (DISK-261); that is a distinct earlier layer. Boot-file copy failures can already change inst_success, but DoInstall's final Execute is also not gated on that updated state. Source-only; no command execution, build or disk operations.

Additional bootloader observations retained: FindWindowsPartition assumes readable active/type attributes and MBR-like byte IDs without checking table type; generated Windows entries hardcode hd0 irrespective of chosen target/firmware mapping; fixed tmp/cmd[256] formatting lacks combined-length checks; ignored Rename/config append errors; GRUB1 CreateDir lock is not released; common boot skip paths are omitted from core copying even when bootloader installation is unselected. Remaining main/options contracts determine applicability.

<a id="disk-289"></a>

### DISK-289 — InstallAROS truncates the selected disk unit to eight bits

- **Severity / confidence:** High conditional wrong-device operation; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_driveselectclass.c:428–499, DriveSelect__MUIM_HandleEvent and DriveSelect__MUIM_Initialize; ia_stageclass.c:1081–1097.
- **Trigger / trace:** OM_SET formats MUIA_DriveSelect_Unit through an explicit UBYTE cast into unttmp[4]. Popup acceptance reads the integer input and passes both device and unit tags to this method, which updates the persistent destination-unit option from the truncated string. Existing-partition startup messages and discovered boot-device defaults also pass their unit through this setter. Thus unit 256 becomes 0 for the same device. MUIM_Partition later obtains that cached value for the Partition command's UNIT argument.
- **Impact / limits:** Where a device exposes units outside 0–255, choosing such a unit can redirect subsequent partition operations to another unit, including a disk containing existing data. Requires the device/configuration to have such a unit and the installer path to proceed; ordinary units 0–255 are unaffected. No assertion that every driver exposes large units, and no runtime reproduction or media write was performed.

<a id="disk-290"></a>

### DISK-290 — InstallAROS looks for GRUB1 stage files in the wrong source directory

- **Severity / confidence:** Medium conditional installation failure; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_bootloader.c BOOTLOADER_DoInstall and GRUB_COPY_FILE_LOOP; arch/all-pc/boot/grub/mmakefile.src; arch/i386-pc/boot/grub/mmakefile.src.
- **Trigger / trace:** The common source path is source_Path plus boot. The GRUB1 branch appends stage1, stage2_hdisk and menu.lst.DH0 directly, without adding the grub subdirectory. The build installs all three under boot/grub, and BOOTLOADER_InitSupport itself detects boot/grub/stage1. A normal detected GRUB1 source therefore does not supply the requested boot/stage1 etc. GRUB2 explicitly appends its subdirectory and is unaffected by this path error.
- **Impact / limits:** GRUB1's file-copy step fails or depends on unrelated files left at the incorrect pathname. Subsequent unchecked execution is DISK-288, not another consequence claimed as tested. Requires a build selecting INSTALL_BL_GRUB/GRUB1 and the standard packaged layout; custom layouts with duplicate files can mask it. No installer or boot command was run.

<a id="disk-291"></a>

### DISK-291 — InstallAROS advances to its completion page after recorded installation failure

- **Severity / confidence:** Medium misleading installation status; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_stageclass.c InstallStage__MUIM_IC_NextStep EInstallStage:761–787; ia_main.c completed-page construction; ia_stageclass.c file-copy failure assignments.
- **Trigger / trace:** EInstallStage calls MUIM_IC_Install and then unconditionally selects EDoneStage, disables Back/Cancel and assigns the done-stage next state. File-copy errors can set inst_success to MUIV_Inst_Failed, but this transition does not inspect that value. The completion label is preconstructed from MSG_DONE, with reboot/USB variants selected earlier rather than from the final result.
- **Impact / limits:** An incomplete install can finish on the completion page despite an earlier failure. An earlier error requester may still have appeared, so this is not a claim of silent failure throughout. Unlike DISK-286 or DISK-288, this applies to failures actually recorded in inst_success. Source-only.

<a id="disk-292"></a>

### DISK-292 — Installers copy unbounded device prefixes into 32-byte stack buffers

- **Severity / confidence:** Medium conditional memory corruption; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_diskio.c getDiskFSSM:256–270; ia_stageclass.c EInstallMessageStage:588–602; ia_main.c destination-name StringObject; workbench/libs/muimaster/classes/string.c String__OM_NEW; arch/all-pc/boot/grub-tools/Install-grub.c:115–136; arch/all-pc/boot/grub2-tools/Install-grub2.c:133–155; workbench/c/Install.c:400–407.
- **Trigger / trace:** getDiskFSSM copies every byte before a colon into dname[32], without checking i against its capacity, and then appends a NUL. The boot-info guess calls it with the editable destination name plus :boot/grub. That StringObject sets no MaxLen; AROS's string class defaults to an 80-byte buffer. A 32-character name writes the terminator past dname, and longer accepted names also overflow during copying, before filesystem lookup can reject them.
- **Impact / limits:** Selecting bootloader installation with a sufficiently long destination name can corrupt the stack. Normal short device/partition names do not trigger it. Its UBYTE index additionally wraps on direct inputs of at least 256 characters without an earlier colon; that longer input is outside the ordinary 80-byte UI path. The standalone GRUB helpers repeat the same unchecked prefix copy; they append both colon and NUL afterward, so a 31-character prefix can overflow the final byte even when the initial copy/lookup terminator fit. C:Install repeats the 32-byte unchecked copy in its drivename lookup (its exact index type differs). CLI-provided overlong prefixes reach these helpers before lookup rejects an unknown name. These are grouped copied helper manifestations, not four independent bugs. The unterminated Initialize buffer discussed above is a different, rejected normal-path claim.

<a id="disk-293"></a>

### DISK-293 — InstallAROS and GRUB helpers traverse the DOS device list without its lock

- **Severity / confidence:** Medium conditional concurrent invalid access; high source confidence, conditional lifetime impact.
- **Source:** workbench/tools/InstallAROS/ia_diskio.c getDiskFSSM; arch/all-pc/boot/grub-tools/Install-grub.c:128–132; arch/all-pc/boot/grub2-tools/Install-grub2.c:147–151; rom/dos/lockdoslist.c; rom/dos/unlockdoslist.c.
- **Trigger / trace:** getDiskFSSM calls LockDosList(LDF_READ), then FindDosEntry(..., LDF_DEVICES). LockDosList obtains di_DevLock only if flags include LDF_ALL (device/volume/assign bits); LDF_READ alone returns the pseudo-list handle without locking that list. The matching UnLock call also does not release a list lock. Device nodes can therefore change while FindDosEntry traverses them or while dn_Startup is accessed afterward.
- **Impact / limits:** Requires concurrent DOS-device removal/change; stable startup lists mask it. The returned startup-message pointer is borrowed and used outside the helper as well, so merely adding one missing flag would not by itself establish all later lifetime safety. Both standalone GRUB getDiskFSSM helpers use the identical LDF_READ-only sequence and share this missing-lock root cause. C:Install includes LDF_DEVICES and is not a missing-lock manifestation, although borrowed-pointer lifetime after unlock remains conditional. Do not confuse this with category-mask mismatches where DEVICES and VOLUMES both operate on the same AROS semaphore.

<a id="disk-294"></a>

### DISK-294 — InstallAROS environment helpers overflow their fixed pathname buffer

- **Severity / confidence:** Medium conditional memory corruption or wrong file selection; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_env.c create_environment_variable/read_environment_variable; ia_main.c source_Path construction and EXTRASPATH read.
- **Trigger / trace:** Both helpers sprintf envarchiveDisk into env_variable_path[100] without a length limit, and then index the last character or append a colon. Main can derive source_Path from a program-directory path held in a 256-byte buffer and passes it to read_environment_variable. A source prefix of 100 or more bytes overflows before AddPart's later bound can help. Empty direct inputs also read env_variable_path[-1]. Both AddPart return values are ignored, so a path that cannot accommodate the environment suffix/name may still be opened at an incomplete path.
- **Impact / limits:** Requires a long source location, oversized direct helper argument, or insufficient suffix space. Ordinary short volume names and standard short source roots avoid the overflow. An incomplete writable path can cause wrong-file truncation in create_environment_variable, but whether such a file exists and can open remains configuration dependent. No files were opened through these helpers in this review.

<a id="disk-295"></a>

### DISK-295 — The m68k floppy host installer mishandles short reads and writes

- **Severity / confidence:** High conditional damaged boot image; high source confidence.
- **Source:** arch/m68k-amiga/boot/floppy/install.c main.
- **Trigger / trace:** read of the 1024-byte boot block is accepted whenever it is nonnegative, without requiring 1024 bytes. A truncated image beginning with DOS passes the signature test; RootBlock bytes not supplied by a sufficiently short read can remain uninitialized, while the program builds and writes a complete boot block. Negative read errors close fd but continue through lseek, signature inspection and later processing instead of returning. The final write only checks negativity, so a positive short write returns EXIT_SUCCESS with a partial boot block.
- **Impact / limits:** Requires truncated input or host I/O failure/short completion; the ordinary full-transfer image path is unaffected. argv[1] is also used without validating argc, a separate invocation-error observation. This is an explicitly built host utility, not the AROS trackdisk driver's I/O contract. Source-only.

<a id="disk-296"></a>

### DISK-296 — Parthenope RDB checksums treat a byte limit as a checksum word count

- **Severity / confidence:** Medium conditional out-of-bounds read/crash; high source confidence.
- **Source:** arch/ppc-sam440/boot/parthenope/tools/parthenope-install.c checksum/read_block/find_rdb; src/rdb.c checksum/get_rdb/get_partition_info.
- **Trigger / trace:** checksum copies exactly 512 bytes into an allocation, reads amiga_SummedLongss, caps it at 512 and then advances an int32_t pointer that many times. The valid allocation holds 128 words. Header values 129–512 therefore read beyond it; values above 512 are reduced to a still-invalid 512-word read. find_rdb calls read_block/checksum on candidate blocks before testing their RDSK identifier, so arbitrary scanned contents can reach the count.
- **Impact / limits:** A malformed or non-RDB sector can trigger a host-process read overrun before the installer safely rejects the disk. Correct 128-or-shorter word counts avoid this bound error. The utility interprets fields in native endian order without conversion; big-endian standalone builds are its assumed working configuration, and little-endian interpretation is an additional portability limitation. The firmware RDB reader contains the identical checksum implementation, calls it on RDSK/PART sectors and is selected by src/mmakefile.src; thus the root cause also affects that firmware path, not only the standalone installer. No disk scan executed.

<a id="disk-297"></a>

### DISK-297 — Parthenope's standalone installer reads a full payload past the final boot-file fragment

- **Severity / confidence:** Medium conditional memory disclosure or crash; high source confidence.
- **Source:** arch/ppc-sam440/boot/parthenope/tools/parthenope-install.c install_slb/main.
- **Trigger / trace:** main allocates st.st_size bytes and sets slb_size to floor(size/4). install_slb steps by 123 words; although its final-block branch adjusts the checksum count for the remaining words, memmove always copies 123 words from slb+offset. For any nonempty boot file whose word count is not a multiple of 123 and whose allocation lacks enough trailing bytes, the last copy reads beyond that allocation. A one-word file requests a 492-byte copy from a four-byte allocation.
- **Impact / limits:** Can crash or place unrelated process memory in the boot block's unused payload tail, which write_block persists as all 512 bytes. Exact allocator layout determines the observed result. Full 123-word multiples avoid this particular overread. Source-only, standalone autotools installer configuration.

<a id="disk-298"></a>

### DISK-298 — Parthenope's standalone installer allocates boot blocks outside the declared reserved range

- **Severity / confidence:** High conditional writes outside reserved metadata; high source confidence.
- **Source:** arch/ppc-sam440/boot/parthenope/tools/parthenope-install.c next_free_block/install_slb; src/rdb.h reserved range fields and IS_FREE.
- **Trigger / trace:** install_slb starts free_block_pointer at zero and passes RDBBlocksHi-RDBBlocksLo as a maximum absolute block number. next_free_block increments that pointer and examines blocks 1 onward; it never adds RDBBlocksLo. For a reserved range Lo=100, Hi=199, the search instead considers low blocks through roughly 99, and its IS_FREE predicate can accept any non-metadata identifier there. write_block then uses the selected number directly. The do-while also examines at least one block before its maximum condition, permitting a boundary overrun on a later call.
- **Impact / limits:** A nonzero reserved low bound can direct writes outside the disk's reserved RDB area, potentially into existing data. Requires standalone execution on such geometry and sectors accepted by IS_FREE. Conventional Lo=0 masks the base-offset mistake; malformed/short reads and the broad free predicate remain additional conditions, not assurances of a valid free sector.

<a id="disk-299"></a>

### DISK-299 — Parthenope's standalone installer commits a new boot-chain head after allocation or I/O failure

- **Severity / confidence:** High conditional destroyed boot chain; high source confidence.
- **Source:** arch/ppc-sam440/boot/parthenope/tools/parthenope-install.c erase_slb/install_slb/write_block/main.
- **Trigger / trace:** main erases the existing SLB before installing its replacement. install_slb returns void and can return early for insufficient free blocks; after obtaining the first block it changes rdb_BootBlockList before completing the chain. main nevertheless calculates and writes the RDB and returns zero. Allocation additionally requests a successor even for the last payload block before marking that block's successor IDNAME_FREE, so one usable payload block alone is insufficient for a one-block replacement. read/fseek/fwrite results are ignored throughout, including the final RDB write.
- **Impact / limits:** An exhausted reserved area or write failure can leave the old loader erased and a partial/unwritten new chain referenced while the program reports success. No rollback or error result reaches main. Separate from DISK-298's wrong allocation range and DISK-297's source overread. Source-only, standalone installer configuration.

<a id="disk-300"></a>

### DISK-300 — SFSformat returns success after reported formatting failure

- **Severity / confidence:** Medium misleading command status; high source confidence.
- **Source:** rom/filesys/SFS/SFSformat/SFSformat.c main; mmakefile.src.
- **Trigger / trace:** A failed Inhibit or ACTION_SFS_FORMAT prints an error but does not change the final return(0). Unknown device, invalid arguments and failed dos.library opening also end at zero. The packet handler's DOSFALSE result is correctly tested for the error message; it is the process status that is discarded.
- **Impact / limits:** Scripts relying on the exit status can proceed as if a requested format succeeded. A human may see the printed error; this is not a claim that every failure is silent or that the formatter writes the wrong disk. Declining the confirmation is not itself classified as an error. Source-only.

<a id="disk-301"></a>

### DISK-301 — Parthenope repeatedly reads the same partition block after a bad checksum

- **Severity / confidence:** Medium boot hang; high source confidence.
- **Source:** arch/ppc-sam440/boot/parthenope/src/rdb.c get_partition_info; src/mmakefile.src.
- **Trigger / trace:** The partition-chain loop reads block, recognizes PART and checks its checksum. A nonzero checksum immediately continues without updating block or consuming the requested partition index. A persistent bad-checksum PART block is read indefinitely. Valid cyclic chains do not establish the same infinite-loop claim: part decreases on each valid PART and a finite requested index eventually breaks; they can instead produce repeated partition entries.
- **Impact / limits:** A damaged partition block can hang firmware partition discovery instead of rejecting the disk. Requires the descriptor read to keep returning that block successfully. General filesystem parsing is not implicated, and no firmware was run.

<a id="disk-302"></a>

### DISK-302 — Parthenope copies an RDB counted partition name as an unbounded C string

- **Severity / confidence:** Medium wrong names or conditional memory corruption; high source confidence.
- **Source:** arch/ppc-sam440/boot/parthenope/src/rdb.c get_partition_info/RdbPartitionTable_init; src/rdb.h PartitionBlock; src/uboot.h disk_partition_t; src/support.c strcpy.
- **Trigger / trace:** pb_DriveName is a 32-byte counted-name field, but strcpy scans it until a NUL and copies the count prefix along with its contents into info->name[32]. The table later skips the prefix by taking name+1. A valid maximum 31-character name fills that source field; even if the immediately following reserved byte is zero, the copy writes 33 bytes and places the terminator in info->type. The subsequent filesystem-type construction overwrites that terminator, so the exported name runs into the type string. Malformed nonterminated following bytes allow a still longer read/copy.
- **Impact / limits:** Maximum-length valid partition names become incorrect firmware names; malformed metadata can cause out-of-bounds source reads and destination writes beyond the name and possibly the containing allocation. Ordinary short counted names with a zero-filled unused tail mask the representation mismatch. This is firmware integration selected by src/mmakefile.src; no boot was performed.

<a id="disk-303"></a>

### DISK-303 — VirtIO identifies modern block devices by subsystem ID instead of product ID

- **Severity / confidence:** Medium disk-discovery failure or wrong-device claim; high source and protocol confidence.
- **Source:** rom/devs/virtio/virtio_init.c Enumerator:199–225; include/hardware/virtio.h:16–23.
- **Trigger / trace:** Every vendor-0x1af4 product in the modern 0x1040–0x107f range is accepted only if subsystem ID equals 2. A modern block device is product 0x1042 regardless of its informational subsystem ID. Thus a conforming block device with subsystem ID 0x40 or another environmental value is skipped. A modern non-block product with subsystem ID 2 passes the class filter.
- **Impact / limits:** Valid disks can be undiscoverable; a non-block device can enter the block-driver attachment path, although feature/queue setup may reject it later. Transitional product 0x1001 takes its separate acceptance branch and does not require the erroneous subsystem check. No hardware attachment or I/O was run. Supported by the [OASIS VIRTIO 1.2 PCI discovery requirements](https://docs.oasis-open.org/virtio/virtio/v1.2/virtio-v1.2.html).

<a id="disk-304"></a>

### DISK-304 — HDTool dereferences an omitted optional argument before reporting usage

- **Severity / confidence:** Medium command crash; high source confidence.
- **Source:** workbench/c/HDTool/main.c template/main/doCommand; list.c list; setup.c setup; install.c install; rom/dos/readargs.c:488, 603–610, 803–808.
- **Trigger / trace:** The outer template accepts COMMAND/A,ARGS/F, initializes both result slots to zero and passes the second result straight to its selected handler. With only LIST, SETUPS or INSTALL, optional ARGS is absent and remains NULL. Each handler constructs its RDArgs source with strlen(args) before checking its own required arguments.
- **Impact / limits:** A missing-arguments invocation crashes instead of returning a usage error. Supplying an actual argument tail avoids this particular NULL dereference, though later parsing may correctly reject its contents. This is a command consumer failure and does not itself write a disk. No HDTool command was executed.

<a id="disk-306"></a>

### DISK-306 — ATA PCI probe error exits unmap the physical BAR instead of its mapping

- **Severity / confidence:** Low mapping leak, conditional greater invalid-unmap effect; high source confidence.
- **Source:** rom/hidds/ata_pci/probe.c:220, 247, 292, 319; rom/hidds/pci/pcidriverclass.c:180–185.
- **Trigger / trace:** MapPCI returns hwhba for hba_phys. CAP_SAM rejection and timer-open failure call UnmapPCI with hba_phys; the normal path uses hwhba. The declared provider contract takes the address returned by MapPCI.
- **Impact / limits:** On a nonidentity mapping backend with meaningful unmapping, these failure exits can leak the acquired mapping or act on an unrelated address. Generic UnmapPCI is a no-op, so the mismatch does not establish generic x86 MMIO corruption. Requires a rejected probe or timer failure plus the relevant backend mapping behavior. Source-only.

<a id="disk-307"></a>

### DISK-307 — ROM USB mass storage allows explicit suspend without draining active I/O

- **Severity / confidence:** Medium conditional interrupted commands/timeouts; high source confidence in missing serialization, conditional hardware impact.
- **Source:** rom/usb/classes/massstorage/massstorage.class.c:868–878; rom/usb/poseidon/poseidon.library.c:3702–3840, 4705–4765; rom/usb/classes/hub/hub.class.c:334–376, 1479–1517.
- **Trigger / trace:** The class accepts suspend after setting ncm_Running false, which suppresses media polling but does not stop regular disk transfers. The core's IOBusyCount veto is in its unsupported-class fallback; this class supplies a successful callback. Hub suspend directly sends SET_FEATURE(PORT_SUSPEND) without a transfer drain. An explicit suspend can therefore overlap a previously submitted mass-storage command.
- **Impact / limits:** The controller/device can interrupt or time out that command; exact recovery/data consequences depend on the controller and are untested. Ordinary automatic idle suspend starts after inactivity, and new pipe submissions resume a suspended device, so neither universal idle failure nor permanent sleeping on later I/O is claimed. No USB operation executed.

<a id="disk-308"></a>

### DISK-308 — Disk-image Reaction GUI checks the wrong hook after allocation

- **Severity / confidence:** Medium allocation-failure crash; high source confidence.
- **Source:** workbench/devs/diskimage/ra_gui/main.c:161–165.
- **Trigger / trace:** CreateExtHook for ReloadPluginsHook fails. The following guard checks the already-created DiskChangeHook, so it passes, and ReloadPluginsHook->task/sigmask dereference NULL.
- **Impact / limits:** Requires the Reaction GUI path, a device version enabling reload-hook support and failure of the second hook allocation. Ordinary successful allocation and the different Zune GUI do not trigger it. This is a callback-registration client defect; no disk-data corruption asserted. Source-only.

<a id="disk-309"></a>

### DISK-309 — ATA assumes 512-byte logical sectors for every non-ATAPI disk

- **Severity / confidence:** High conditional addressing/transfer-size mismatch; high source confidence, media-write consequence conditional on hardware.
- **Source:** rom/devs/ata/lowlevel.c:1766–1799, 1213–1221; ata.c:65–95.
- **Trigger / trace:** Identify logical-sector fields are converted but do not choose the sector shift. Non-ATAPI attachment sets au_SectorShift=9; byte-to-LBA conversion and requested-sector byte counts then use 512-byte logical units. A disk exposing larger native logical sectors receives an LBA/count describing a different byte region/length than the caller intended.
- **Impact / limits:** Such disks lack correct logical-size adaptation and can fail transfers or access unintended regions if the controller completes the translated operation. Successful DMA overrun or a specific on-media result is not established without the controller transfer-length contract. A 512e disk with 512-byte logical sectors is unaffected merely because its physical sector is larger. Source-only; no hardware or driver execution.

<a id="disk-311"></a>

### DISK-311 — InstallAROS counts incomplete file transfers as successfully copied files

- **Severity / confidence:** High conditional incomplete installation/backup data; high source confidence.
- **Source:** workbench/tools/InstallAROS/ia_stageclass.c:1915–1986, InstallStage__MUIM_IC_CopyFile; ia_diskio.c BackUpFile; rom/dos/write.c Write result contract; rom/dos/close.c Close failure contract.
- **Trigger / trace:** CopyFile treats Write(to,buffer,s) as successful for every result except -1. A result 0 <= n < s loses the remaining bytes and advances to the next input chunk or EOF; the destination is then counted as one copied file. The loop also interprets a positive Read smaller than the buffer as EOF instead of continuing to a zero result, so a provider returning partial positive reads with more data remaining truncates the copy. Close(to)'s outcome is ignored. Backup copying shares missing count/finalization validation.
- **Impact / limits:** A short successful transfer or finalization error can leave incomplete destination/backup files while file-count validation appears satisfied. Requires a provider/error path returning the relevant result; ordinary complete writes and EOF-shaped short final reads are unaffected. Missing backup opens remain DISK-284's separate root cause. No DOS I/O, copying or error injection performed.

<a id="disk-312"></a>

### DISK-312 — HDToolBox mountlist export rounds away a parent partition's sector offset

- **Severity / confidence:** High conditional wrong-region mount configuration; high source confidence.
- **Source:** workbench/tools/HDToolBox/gui.c:1208–1234, 1305, 1319–1320, mountlist output.
- **Trigger / trace:** GetStartCyl sums ancestor starts in sectors and divides by the leaf's sectors-per-cylinder, discarding a remainder. The mountlist adds that quotient to both leaf LowCyl and HighCyl while retaining leaf Surfaces/BlocksPerTrack. For a parent start of 1001 sectors and a leaf spc of 100, the exported offset is 1000 sectors, one sector before the actual start. Different virtual geometries are explicitly considered by the helper, but it does not reject or adapt the nonintegral result.
- **Impact / limits:** Using the exported mountlist can expose the wrong disk region and later filesystem writes can affect unrelated bytes. Requires nested/nonaligned geometry, export, and subsequent use; ordinary exactly divisible ancestor starts and top-level partitions avoid this rounding defect. Native ULONG overflow is an additional bounded arithmetic observation, not required by the small example. DISK-281's wrong unit lookup is independent and can coexist. Export does not itself write filesystem data. Source-only.

<a id="disk-315"></a>

### DISK-315 — PC floppy initialization ignores worker-task allocation failure

- **Severity:** Medium (opened device cannot process queued I/O). **Confidence:** High for the allocation-failure path; source-only.
- **Source:** `rom/devs/trackdisk/trackdisk_device.c:273-276,325-365,567-642`.
- **Trigger/trace:** `TD_InitTask` returns zero if its task-data, stack or MemList allocation fails. Device `Init` ignores that result and returns success after recording any detected floppy units. `Open` can then succeed for a present unit, while ordinary nonquick I/O and the quick read/write/seek/format commands pass `&TDBase->td_TaskData->td_Port` to `PutMsg`. The task-data pointer was never assigned on this failure path.
- **Consequence:** The first queued request dereferences a null task-data pointer instead of failing device initialization or the request. Propagate task-creation failure and unwind the units/interrupt servers, or reject opens and queued requests when no worker exists. A failed `NewAddTask` after allocation is a separate unverified startup contract and is not needed for this finding.

<a id="disk-316"></a>

### DISK-316 — PCI ATA interrupt-registration failure leaks its DMA buffer

- **Severity:** Low to medium (resource leak on failed bus creation). **Confidence:** High for the stated failure path; source-only.
- **Source:** `rom/hidds/ata_pci/ata_pci_busclass.c:104-137,154-189,217-240`; `rom/hidds/ata_pci/probe.c:636-663`.
- **Trigger/trace:** A probed PCI ATA bus with a DMA base successfully allocates `data->dmaBuf`, but `HIDD_PCIDevice_AddInterrupt` fails. The constructor frees the interrupt record and invokes `OOP_DoSuperMethod` for `moRoot_Dispose`, bypassing `PCIATABus__Root__Dispose`, which is where `HIDD_PCIDriver_FreePCIMem` releases the DMA buffer.
- **Consequence / limits:** The DMA buffer remains allocated after failed bus creation. Repeated failed probes can accumulate this leak. The probe caller does release the bus record and PCI device reference because the ownership marker is set only after successful interrupt registration; those resources are excluded from this finding. Free the buffer in constructor failure cleanup or route disposal through the subclass safely. No allocation failure or interrupt registration failure was reproduced at runtime.



### DISK-322 — PCI ATA never releases runtime PCI attribute-base references

- **Severity / confidence:** Low (two OOP IID references retained for the module lifetime); high source confidence, conditional on successful PCI discovery.
- **Source:** `rom/hidds/ata_pci/probe.c:433-442,541-550`; `rom/hidds/ata_pci/ata_pci_intern.h:15-34`; `rom/hidds/ata_pci/class_init.c:100-104`; `rom/hidds/ata_pci/ata_pci.conf:1-8`.
- **Trigger / trace:** On the first PCI scan, `ata_bus_Detect` obtains attribute bases for `IID_Hidd_PCIDevice` and `IID_Hidd_PCIDriver` into `PCIDeviceAttrBase` and `PCIDriverAttrBase` using the ID-list suffix at `ATTR_OFFSET`. The module's only expunge hook calls `OOP_ReleaseAttrBasesArray` starting at `hiddAttrBase` with the mandatory suffix, so it releases `IID_Hidd`, bus, ATA bus and HW bases but never these first two runtime-acquired PCI bases. The configuration marks this HIDD `noexpunge`, making the retained references last for the system lifetime; the omission remains if expunge is invoked through an unusual teardown path.
- **Impact / limits:** Successful use with the PCI subsystem leaves two IID bucket reference counts elevated until shutdown, preventing those buckets from returning to their prior reference state. This does not establish a growing leak during ordinary operation because the code acquires the pair only once, guarded by `PCIDeviceAttrBase == 0`. Release the PCI suffix during teardown or make the references explicitly process-lifetime owned. No HIDD was loaded or removed; source-only.

### DISK-323 — USB MassStorage reports non-CD SCSI media writable when write-protected

- **Severity / confidence:** Medium (a protected disk may be advertised as writable and fail later writes); high source confidence for the status path, conditional on a non-CD write-protected SCSI LUN.
- **Source:** `workbench/devs/USB/classes/MassStorage/storage_device.c:94-97,244-292,300-349,357-397`; `rom/filesys/afs/os_aros_support.c:427-432`; `rom/filesys/SFS/FS/deviceio.c:103-116`; `rom/filesys/pfs3/fs/volume.c:1026-1037,861-873`; `workbench/devs/USB/classes/mmakefile.src:1`; `workbench/devs/USB/classes/MassStorage/mmakefile.src:5-10`; `workbench/devs/USB/mmakefile.src:3-8`.
- **Trigger / trace:** `cmd_ProtStatus` returns `io_Actual = -1` only for the INQUIRY peripheral type `DG_CDROM`; all other logical units are reported writable. SCSI SBC-3 §6.3.1 defines the MODE SENSE header WP bit for direct-access media, but this unit keeps no write-protect state and does not query that bit in its `TD_PROTSTATUS` implementation. The Mode Sense path preserves the first eight response bytes, including the MODE SENSE(10) header field. If writes fail, handlers reset and retry before returning `TDERR_NotSpecified`. The MassStorage HIDD class is selected by the `hidd-usb-classes` build edge.
- **Consequence / limits:** AFS, SFS and PFS3 treat zero `TD_PROTSTATUS` as writable, so a write-protected non-CD medium can be mounted or advertised as writable and fail only when a write is attempted. Track and return the LUN's protection state through `TD_PROTSTATUS`. Trigger requires a non-CD SCSI LUN whose medium is write-protected; no device or filesystem was run. Source-only. The SCSI behavior is specified in [SBC-3 §6.3.1](https://t10.org/ftp/t10/document.05/05-344r0.pdf).

<a id="disk-321"></a>

### DISK-321 — AHCI init leaks partial attribute bases when initial acquisition fails

- **Severity / confidence:** Low (persistent OOP IID reference leak after failed module initialization); high source confidence, conditional on an attribute-base acquisition failing after earlier IDs succeeded.
- **Source:** `rom/devs/ahci/ahci_init.c:55-64,163-190,254`; `rom/devs/ahci/ahci.conf:1-10`; `rom/oop/obtainattrbasesarray.c:60-71`; `rom/oop/obtainattrbase.c:103-123`; `compiler/autoinit/functions.c:53-69`.
- **Trigger / trace:** AHCI asks for eight attribute bases in the order matching its eight consecutive fields (`ahci_intern.h:79-86`). `OOP_ObtainAttrBasesArray` stores zero for a failed IID, continues, and returns a nonzero failure count while retaining earlier successful references. `AHCI_Init` enters its setup/cleanup block only when that result is zero (`ahci_init.c:165-166`); on a nonzero result it skips the block's `OOP_ReleaseAttrBasesArray`, deletes the pool, closes Utility, and returns FALSE. Unlike ATA, SCSI, WD33C93 and PCI ATA, AHCI registers no ExpungeLib cleanup hook, and its configuration is `noexpunge`; generated failed-init rollback therefore has no callback that releases the successful prefix (`compiler/autoinit/functions.c:60-69`).
- **Impact / limits:** The failed initialization leaves each successfully obtained IID reference elevated in `oop.library`, retaining its bucket; retrying the module can accumulate those references. This requires an earlier IID obtain to succeed and a later first-time IID name allocation to fail. Release nonzero entries before returning FALSE or register a rollback hook. No allocation failure or retry was run; source-only.

<a id="disk-320"></a>

### DISK-320 — PCI ATA init releases attribute bases against the wrong IDs after method-base failure

- **Severity / confidence:** Medium (corrupts OOP base bookkeeping during failed initialization); high source confidence, conditional on method-base acquisition failure.
- **Source:** `rom/hidds/ata_pci/class_init.c:24-35,63-64,85-94`; `rom/hidds/ata_pci/ata_pci_intern.h:15-34`; `rom/oop/releaseattrbasesarray.c:50-65`.
- **Trigger / trace:** `pciata_init` obtains attributes using `&attrBaseIDs[AB_MANDATORY]`, so the four entries starting at `IID_Hidd`, `IID_Hidd_Bus`, `IID_Hidd_ATABus`, and `IID_HW` fill the four fields starting at `hiddAttrBase`. If the later method-base acquisition fails, cleanup instead passes the full six-ID `attrBaseIDs` list from its beginning with the destination still at `hiddAttrBase`. The release helper advances both arrays for every ID. It therefore pairs those four acquired values with the two preceding PCI IDs and then also reads the following method-base fields as if they were attribute bases; it zeros the visited fields as it goes.
- **Impact / limits:** This can release references under unrelated interface IDs, leak the four acquired attribute-base references, and overwrite the first two method-base fields during failed initialization. This is limited to the conditional method-base failure path. Release using the same suffix as acquisition, or track each acquisition range explicitly. No method-base failure was induced; source-only.

<a id="disk-319"></a>

### DISK-319 — USB MassStorage device initialization dereferences failed pooled allocation

- **Severity / confidence:** High conditional module-initialization fault under memory pressure; high source confidence.
- **Source:** `workbench/devs/USB/classes/MassStorage/storage_device.c:784-803`; `rom/exec/allocpooled.c:28-56`.
- **Trigger / trace:** `StorageDev_Init` requests a `mss_device_t` plus its negative vector table from the class pool with `AllocPooled`, adds the vector-table size to the returned pointer, and immediately writes `dev->mss_static` and device metadata without checking for NULL. `AllocPooled` explicitly returns NULL when it cannot satisfy the allocation. The pointer arithmetic then produces a low address derived from NULL, and the first field write accesses that invalid address instead of returning an initialization failure.
- **Impact / limits:** Loading the selected USB MassStorage HIDD class while its pooled allocation fails can fault during module initialization or corrupt low memory, preventing storage-device setup. Check the allocation before pointer adjustment, propagate initialization failure, and leave the class cleanup path consistent. The HIDD USB aggregate selects this class for the Sam440 and Efika package edges (`workbench/devs/USB/classes/MassStorage/mmakefile.src:1-12`; `arch/ppc-sam440/boot/mmakefile.src:40-53`; `arch/ppc-chrp/efika/boot/mmakefile.src:83-91`). Those manifests establish build selection only; no image or allocation failure was run. Source-only.

<a id="disk-317"></a>

### DISK-318 — NVMe submission-queue creation failure retains an unusable I/O queue

- **Severity:** Medium (DMA memory and task leak on controller setup failure). **Confidence:** High for the source path.
- **Source:** `rom/devs/nvme/nvme_busclass.c:365-374,405-456,470-471,675-680`; `rom/devs/nvme/nvme_controllerclass.c:549-559`; `rom/devs/nvme/nvme_busclass.c:78-105`.
- **Trigger/trace:** The completion-queue creation command succeeds, but the following submission-queue command fails, times out or reports an error. `NVMEBus_Start` only logs that failure. It neither tears down the controller's new completion queue nor releases the host queue, and increments `queuecnt` only on success. The I/O task was already started and waits forever. When this was the first I/O queue, `queuecnt` stays zero, namespace attachment is skipped, and the start method still returns `TRUE`. Controller disposal iterates only through indexes zero to `queuecnt`, so it also misses the retained `dev_Queues[1]` allocation.
- **Consequence:** A failed startup can leave an orphan I/O task and DMA allocations, with the controller advertised as started but no usable namespace. Stop the task and delete the created completion queue before freeing host resources; propagate failure when no I/O queue can be registered. DISK-087 covers the distinct branches that free a queue while its task still holds it. No command failure or controller was run; source-only.

### DISK-317 — Withdrawn: legacy ATA PC Card configuration tuple path is unselected

- **Scope:** The legacy `ata_amiga.c` body is not named by the current m68k Amiga ATA override, which selects only `waitnano` (`arch/m68k-amiga/devs/ata/mmakefile.src:7-12`). The core ATA manifest likewise does not name it (`rom/devs/ata/mmakefile.src:20-47`). This therefore remains an inactive-source observation, not a retained selected-path candidate.
- **Source:** `arch/m68k-amiga/devs/ata/ata_amiga.c:508-565,577-600,615-655`; `arch/m68k-amiga/card/copytuple.c:163-179`.
- **Trigger / trace:** `detectcard` accepts a `PCCARD_TPL_CONFIG` tuple whenever its declared data length (`tuple[1]`) is at least five. It then decodes a base-address byte count of one to four and a register-mask byte count of one to four from `tuple[2]`, and reads both variable-length fields from `tuple[4]` onward without checking that their combined count fits the declared data. A tuple declaring five data bytes but specifying four base bytes and four mask bytes passes the guard; the parser reads five bytes beyond the declared tuple data. `CopyTuple` copies only the actual declared data bytes and does not clear the rest of the caller's reused 258-byte buffer, so these extra bytes come from earlier tuple contents or uninitialized stack storage rather than the card's configuration tuple.
- **Consequence / limits:** If this legacy body is built separately, the probe can derive the wrong `configbase` or `configmask` and still return success. `initializecard` uses those values to decide whether and where to write configuration registers. The trigger also requires an inconsistent or truncated card tuple; a conforming tuple with all advertised bytes is unaffected. Validate `tuple[1] >= 2 + base_bytes + mask_bytes` before reading if the path is revived. No PC Card tuple or hardware was tested; source-only.

## Historical aliases and withdrawn entries

<a id="disk-022"></a>

### DISK-022 — Merged into DISK-006

This historical ID refers to hostdisk's publication-before-IOF_QUICK-clear manifestation. It is not an additional root cause. See DISK-006.

<a id="disk-283"></a>

### DISK-283 — Withdrawn: drag-rounding example was arithmetically incorrect

Rechecking the example invalidates this candidate. With spc=100, LowCyl=10, HighCyl=19 and diff=1, the resulting start is 1001 and inclusive end is 2000. LowCyl becomes 10 and HighCyl becomes floor(2001/100)-1 = 19, preserving ten cylinders. More generally, adding the same diff to both bounds of an initially cylinder-aligned partition preserves their cylinder-count difference under these formulas. The earlier claim of HighCyl=18 and a lost cylinder was an arithmetic error. Retain this ID for audit history; exclude it from bug counts. Mixed geometry, malformed existing intervals and overflow remain separate, conditional observations requiring their own evidence.

<a id="disk-305"></a>

### DISK-305 — Merged into DISK-244

This repeated confirmation is the same source/root cause as [DISK-244](#disk-244). Keep the ID for audit history; exclude it from candidate counts.

<a id="disk-310"></a>

### DISK-310 — Merged into DISK-233

This repeated confirmation is the same source/root cause as [DISK-233](#disk-233). Keep the ID for audit history; exclude it from candidate counts.

<a id="disk-313"></a>

### DISK-313 — Merged into DISK-245

This repeated confirmation is the same source/root cause as [DISK-245](#disk-245). Keep the ID for audit history; exclude it from candidate counts.

<a id="disk-314"></a>

### DISK-314 — Merged into DISK-246

This repeated confirmation is the same source/root cause as [DISK-246](#disk-246). Keep the ID for audit history; exclude it from candidate counts.

## Implementation inventory snapshot

This snapshot was generated from the unchanged baseline directories and reconciled with the implementation counts and build selections in the coverage records. It includes local optional/legacy C files, so its counts do not imply every file is active in every build. Public/shared headers and the listed direct integration consumers are covered separately above.

- `rom/hidds/base/storage` (5 C files): `storage_busclass.c`, `storage_class.c`, `storage_controllerclass.c`, `storage_init.c`, `storage_unitclass.c`.
- `rom/devs/ata` (13 C files): `ata.c`, `ata_busclass.c`, `ata_controllerclass.c`, `ata_init.c`, `ata_smart.c`, `ata_trim.c`, `ata_unitclass.c`, `bootwait.c`, `lowlevel.c`, `scsiemu.c`, `timer.c`, `waitnano.c`, `waitto.c`.
- `rom/hidds/ata_pci` (6 C files): `ata_pci_busclass.c`, `ata_pci_controllerclass.c`, `class_init.c`, `interface_dma.c`, `interface_pio.c`, `probe.c`.
- `arch/m68k-amiga/hidd/gayle_ata` (5 C files): `class_init.c`, `fastata_busclass.c`, `gayleata_busclass.c`, `interface_pio.c`, `probe.c`.
- `arch/m68k-amiga/devs/ata` (4 C files): `ata_amiga.c`, `bus_pci.c`, `dma_generic.c`, `waitnano.c`.
- `arch/ppc-chrp/efika/ata` (2 C files): `ata_bestcomm.c`, `lowlevel_mpc5200b.c`.
- `arch/ppc-sam440/ata` (1 C files): `bus_ppc460.c`.
- `rom/devs/ahci` (15 C files): `ahci.c`, `ahci_aros.c`, `ahci_aros_bus.c`, `ahci_attach.c`, `ahci_busclass.c`, `ahci_cam_aros.c`, `ahci_controllerclass.c`, `ahci_init.c`, `ahci_io.c`, `ahci_pm.c`, `ahci_scsi.c`, `ahci_unitclass.c`, `bus_pci.c`, `subr_prf.c`, `timer.c`.
- `rom/devs/nvme` (12 C files): `nvme_busclass.c`, `nvme_controllerclass.c`, `nvme_hw.c`, `nvme_init.c`, `nvme_io.c`, `nvme_prp.c`, `nvme_queue.c`, `nvme_queue_admin.c`, `nvme_queue_io.c`, `nvme_sgl.c`, `nvme_timer.c`, `nvme_unitclass.c`.
- `rom/devs/virtio` (8 C files): `virtio_busclass.c`, `virtio_controllerclass.c`, `virtio_hw.c`, `virtio_init.c`, `virtio_io.c`, `virtio_queue.c`, `virtio_timer.c`, `virtio_unitclass.c`.
- `rom/devs/scsi` (10 C files): `bootwait.c`, `lowlevel.c`, `scsi.c`, `scsi_busclass.c`, `scsi_controllerclass.c`, `scsi_init.c`, `scsi_unitclass.c`, `timer.c`, `waitnano.c`, `waitto.c`.
- `rom/hidds/scsi/wd33c93` (4 C files): `wd33c93.c`, `wd33c93_busclass.c`, `wd33c93_init.c`, `wd33c93_probe.c`.
- `rom/devs/sdcard` (8 C files): `bootwait.c`, `sdcard_bus.c`, `sdcard_device.c`, `sdcard_init.c`, `sdcard_ioops.c`, `sdcard_mmcunit.c`, `sdcard_sdscunit.c`, `timer.c`.
- `arch/arm-native/soc/broadcom/2708/sdcard` (5 C files): `sdcard_bcm2708bus.c`, `sdcard_bcm2708init.c`, `sdcard_bcm2708time.c`, `sdcard_sdhost_bus.c`, `sdcard_sdhost_init.c`.
- `rom/usb/classes/massstorage` (7 C files): `debug.c`, `dev.c`, `massstorage.class.c`, `massstorage_bulk.c`, `massstorage_cbi.c`, `massstorage_uas.c`, `partitions.c`.
- `workbench/devs/USB/classes/MassStorage` (6 C files): `storage_device.c`, `storage_init.c`, `storage_stubs.c`, `storage_task.c`, `storage_volumes.c`, `storageclass.c`.
- `rom/devs/trackdisk` (2 C files): `trackdisk_device.c`, `trackdisk_hw.c`.
- `arch/m68k-amiga/devs/trackdisk` (2 C files): `trackdisk_device.c`, `trackdisk_hw.c`.
- `arch/m68k-amiga/disk` (1 C files): `disk_intern_init.c`.
- `arch/all-hosted/devs/hostdisk` (3 C files): `automount.c`, `hostdisk_device.c`, `hostdisk_host.c`.
- `arch/all-unix/devs/hostdisk` (2 C files): `geometry.c`, `hostdisk_host.c`.
- `arch/all-mingw32/devs/hostdisk` (1 C files): `hostdisk_host.c`.
- `arch/all-linux/hostdisk` (1 C files): `geometry.c`.
- `arch/all-darwin/hostdisk` (1 C files): `geometry.c`.
- `workbench/devs/diskimage/device` (12 C files): `init.c`, `init_aros.c`, `io.c`, `locale.c`, `main_vectors.c`, `password.c`, `plugin_vectors.c`, `plugins.c`, `progress.c`, `scsicmd.c`, `tempfile.c`, `unit.c`.
- `workbench/devs/diskimage/plugins` (32 C files): `adf.c`, `ccd.c`, `ciso.c`, `cpc.c`, `crypt.c`, `cue/cue.c`, `cue/fdio.c`, `cue/flac.c`, `cue/malloc.c`, `cue/select.c`, `cue/stdio.c`, `cue/string.c`, `cue/wavpack.c`, `d64.c`, `daa.c`, `dax.c`, `dmg/adc.c`, `dmg/base64.c`, `dmg/dmg.c`, `dms.c`, `fdi/fdi.c`, `fdi/fdi2raw.c`, `generic.c`, `gi.c`, `ipf.c`, `iso.c`, `mds.c`, `nrg.c`, `stub_aros.c`, `uif.c`, `xad.c`, `xpk.c`.
- `workbench/devs/diskimage/audio` (7 C files): `aiff.c`, `flac.c`, `mp3_mpega.c`, `mp3_mpg123.c`, `vorbis.c`, `wave.c`, `wavpack.c`.
- `workbench/devs/diskimage/dms` (10 C files): `crc_csum.c`, `getbits.c`, `maketbl.c`, `tables.c`, `u_deep.c`, `u_heavy.c`, `u_init.c`, `u_medium.c`, `u_quick.c`, `u_rle.c`.
- `workbench/devs/diskimage/prefs` (3 C files): `prefs.c`, `readprefs.c`, `writeprefs.c`.
- `workbench/devs/diskimage/support` (27 C files): `aros/asprintf.c`, `aros/asprintfpooled.c`, `checkbptr.c`, `checklib.c`, `copystringbstrtoc.c`, `deletelibrary.c`, `diskimagedevice.c`, `dos64.c`, `envvar.c`, `getattrs.c`, `getcurrentdir.c`, `hooks.c`, `istext.c`, `lists.c`, `localeinfo.c`, `messages.c`, `mutexes.c`, `paths.c`, `ports.c`, `reallocbuf.c`, `semaphores.c`, `setprocwindow.c`, `strlcpy.c`, `swab2.c`, `tooltypes.c`, `translatefuncs.c`, `trimstr.c`.
- `rom/partition` (33 C files): `addbootfilesystem.c`, `addpartition.c`, `closepartitiontable.c`, `closerootpartition.c`, `crc32.c`, `createpartitiontable.c`, `deletepartition.c`, `destroypartitiontable.c`, `findfilesystema.c`, `fsloader.c`, `getfilesystemattrsa.c`, `getpartitionattrs.c`, `getpartitiontableattrs.c`, `loadbootfilesystems.c`, `loadfilesystem.c`, `openpartitiontable.c`, `openrootpartition.c`, `partition_init.c`, `partition_platform.c`, `partition_support.c`, `partition_types.c`, `partitionami_init.c`, `partitionebr.c`, `partitiongpt.c`, `partitionmbr.c`, `partitionrdb.c`, `querypartitionattrs.c`, `querypartitiontableattrs.c`, `readpartitiondata.c`, `setpartitionattrs.c`, `setpartitiontableattrs.c`, `writepartitiondata.c`, `writepartitiontable.c`.
- `workbench/c/Partition` (2 C files): `args.c`, `main.c`.
- `workbench/c/HDTool` (5 C files): `deviceio.c`, `install.c`, `list.c`, `main.c`, `setup.c`.
- `workbench/system/BHFormat` (4 C files): `cli.c`, `gui.c`, `locale.c`, `main.c`.
- `workbench/tools/QuickPart` (14 C files): `QP_FindDisks.c`, `QP_Gui.c`, `QP_Main.c`, `QP_PartitionColors.c`, `QP_ccApp.c`, `QP_ccDisk.c`, `QP_ccFree.c`, `QP_ccOpGrp.c`, `QP_ccOperation.c`, `QP_ccPartition.c`, `QP_ccPartitionContainer.c`, `QP_ccTxt.c`, `QP_globals.c`, `QP_locale.c`.
- `workbench/tools/HDToolBox` (11 C files): `debug.c`, `devices.c`, `gui.c`, `harddisks.c`, `hdtoolbox_support.c`, `locale.c`, `main.c`, `partitions.c`, `partitiontables.c`, `prefs.c`, `ptclass.c`.
- `workbench/tools/InstallAROS` (11 C files): `ia_bootloader.c`, `ia_diskio.c`, `ia_driveselectclass.c`, `ia_env.c`, `ia_installclass.c`, `ia_locale.c`, `ia_main.c`, `ia_optionclass.c`, `ia_packages.c`, `ia_stageclass.c`, `ia_volumeclass.c`.

Additional directly reviewed files include workbench/devs/{ramdrive_device,fdsk_device}.c; workbench/c/Install.c; both arch/all-pc/boot/grub*-tools/Install-grub*.c; rom/filesys/SFS/SFSformat/SFSformat.c; arch/m68k-amiga/boot/floppy/install.c; Parthenope arch/ppc-sam440/boot/parthenope/tools/parthenope-install.c and src/{rdb,context,support,ext2}.c; ROM/driver boot/partition registration and trackdisk preference/IPrefs sources. Exact interfaces, selectors and dependency slices are recorded in the review log. Bundled codec directories are deliberately omitted from the all-C implementation snapshot because only their wrapper/allocation/ABI/build boundary is claimed reviewed.

## Review log and coverage evidence

The following log preserves historical source reads, tentative observations, later corrections and caller tracing. References to unfinished work describe the checkpoint at which they were written; the final coverage matrix and reconciliation ledger supersede them. A tentative observation is not an additional confirmed bug unless its later disposition supports that interpretation.

### Review log

- Shared storage class/controller/bus/unit source read; allocation, enumeration and lifetime contracts inspected. Initialization, headers and registration callers were subsequently reviewed; see the shared-storage reconciliation.
- VirtIO functional C implementation read (I/O, queue, hardware, probe, controller, bus and unit). Timer/header/build integration, initialization-failure cleanup, transport constraints and architecture applicability were subsequently covered in the VirtIO continuation and reconciliation records.
- NVMe I/O and I/O queue read at this checkpoint; full controller/queue/DMA coverage is documented in subsequent passes.
- Inventory confirmed VirtIO is a block driver and discovered both USB mass-storage implementations and platform ATA backends. Full build inventory remains open.

### Additional review progress
- ATA top-level request handling and SCSI emulator read. Low-level PIO/DMA and platform code still pending.
- AHCI request handling, SCSI translation and AROS DMA/IRQ shim read. Core controller, port recovery and platform lifecycle still pending.
- NVMe PRP/SGL builders, generic queue allocation and hardware queue walker read at this checkpoint. Subsequent passes complete namespace/controller/admin/lifecycle coverage.

### Inventory refinements
- Hosted disks also have Windows (`arch/all-mingw32/devs/hostdisk`) and Darwin geometry (`arch/all-darwin/hostdisk`) backends; those sources have now been read.
- `arch/m68k-amiga/nvdisk` belongs to nonvolatile.library support, not a block disk driver. Its inclusion will be reassessed from build definitions rather than counted as a disk-driver review.
- SD request dispatch and sector transfer implementation read; MMC/SDSC speed switching read. SD bus/controller discovery and host command/error recovery still pending.
- Hostdisk shared request/task implementation plus generic, UNIX, Windows and Linux/Darwin geometry sources read. Automount, host ABI declarations and build integration were completed in the subsequent hosted source-coverage milestone.

### USB review progress
- Workbench USB class/task/device coverage was partial at this checkpoint; the later remaining-files milestone completes open/close/abort, headers, init, volumes and stubs.
- Poseidon UAS source fully read; bulk and CBI substantial sections read, with truncated initial reads explicitly not counted as complete coverage. Remaining bulk reset/recovery portions and class/device/discovery/partition sources pending.

### Disk-image review candidates

### Additional coverage notes
- Disk-image dos64 support, generic/CISO/DAX plugins and plugin build definitions read completely. CD audio SCSI dispatch traced to CUE_ReadCDDA; this is not a full CUE review.
- Floppy ETD change-count comparison differs from diskimage; held for contract validation rather than asserted as a confirmed candidate.
- Overall review remains incomplete; inventory and earlier progress notes specify substantial remaining driver, recovery, partition and image-parser work.

### Floppy review candidates

### Floppy coverage update
- PC and Amiga trackdisk device/hardware sources, internal headers and build definitions fully read. Main transfer, cache, DMA, media and dispatch paths traced. Device configurations, public APIs and disk.resource arbitration were subsequently reconciled in the closing coverage records.
- PC BeginIO's publication-before-IOF_QUICK-clear is another instance of DISK-006 (not a separate root cause).
- ETD change-counter inequality remains a contract-validation item; no external protocol assumptions were promoted to a finding.

### File/RAM-backed device coverage and shared roots
- `workbench/devs/fdsk_device.c`, `ramdrive_device.c`, their GCC headers and build definitions read; formatter and startup/shutdown dispatch included. Both configuration files were subsequently read in the RAM/file-backed build reconciliation.
- FDSK TD_ADDCHANGEINT adds the request node to changeints and then unconditionally replies it (`fdsk_device.c:603,626`), the same node-ownership defect as DISK-021. Eject notification traverses the resulting corrupt list.
- FDSK publishes requests before clearing IOF_QUICK (`fdsk_device.c:284`), another DISK-006 instance.
- FDSK CMD_UPDATE is immediate and never flushes the backing file, sharing DISK-023's ordering/durability defect. FDSK's zero-progress Write loop also shares DISK-024.
- FDSK getgeometry ignores ExamineFH failure and uses an uninitialized FileInfoBlock (`fdsk_device.c:454`); initial/reinsert protection checks similarly ignore failure. Candidate impact is incorrect geometry/protection when the backing handler cannot examine a handle; confidence high, severity medium.
- FDSK extended transfer commands ignore iotd_Count entirely. Exact ETD contract validation remains pending together with the floppy inequality issue.

### Partition review candidates

### Partition coverage update
- GPT handler and header, partition support/platform helpers, root open/close, table open, attribute getter and raw read/write API fully read. GPT discovery/read/write/attribute paths traced.
- MBR/EBR/RDB handlers, filesystem loaders, remaining public wrappers and platform overrides remain pending; this is not full partition-library coverage.
- Floppy/FDSK/RAM configuration files inspected; their module wiring agrees with the source inventory.

### MBR/EBR coverage update
- MBR and EBR handlers and MBR header fully read; public AddPartition/SetPartitionAttrs traced. Discovery, editing, writing and destruction paths included.
- Root geometry zero divisors, zero/overflow partition sizes, and entry bounds are still being checked across RDB and callers before grouping the shared validation candidates.

### RDB and filesystem-loader candidates

### RDB coverage update
- RDB handler and fsloader fully read, together with hardblocks structures, filesystem lookup/load/boot queuing wrappers, initialization, table close/create/write/destroy wrappers and ELF loader seek call sites.
- Remaining partition wrappers, type mapping/CRC, legacy build files and architecture overrides still require review. RDB creation with zero CylSectors loops on zero while doubling its reserved size; this joins the pending shared geometry-validation review.

### Partition and floppy-resource coverage milestone
- All C sources, private/public headers, configuration and build definitions under rom/partition have now been read, including legacy Amiga initialization/build macros and the Amiga architecture override. The legacy Makefile.gcc is separate from the modern mmake build and references obsolete initialization fields; it has not been built or validated as a supported target.
- All rom/disk sources/config/build definitions and arch/m68k-amiga/disk sources/build definitions read. Generic disk.resource initialization explicitly returns failure; only the Amiga hardware backend provides functionality here.
- Architecture partition buffer defaults contain no additional substantiated candidate. CRC lookup/calculation and simple public dispatch wrappers contain no additional candidate in this pass.
- Integration beyond partition.library (driver mount/partition registration and tools using these APIs) remains pending. Full subsystem coverage is still incomplete.

### NVMe continuation coverage
- Initialization/probe/open/close, controller constructor/disposal/reset callback, admin submission/completion and timer helpers read fully. Earlier queue/addressing reviews are retained.
- Bus namespace/queue creation and cleanup still need full reading; several admin timeout sites there will be traced before extending DISK-082. Hardware protocol endian/alignment/capability assumptions remain pending, not counted as verified external specifications.

### NVMe namespace/queue review candidates

### NVMe cross-path evidence
- DISK-082 also affects Hidd_NVMEBus_Start: all admin operations reuse one stack busehandle and one DMA buffer. A timed-out command retains this pointer while subsequent commands register the same handler again. Late completions can satisfy the wrong wait or overwrite the buffer being parsed for another namespace; final buffer free/return leaves late DMA and callbacks live.
- Essential unit geometry, au_Bus and transfer cap are initialized only inside if(ExpansionBase). A missing expansion.library therefore leaves allocated units with unusable zero/default I/O state; the normal boot configuration provides this library, so this is a conditional initialization-path candidate.
- PRP/SGL construction, DMA helpers, controller/bus/unit source and configuration/build wiring reviewed. Remaining NVMe protocol header and PCI/cache implementations must be checked before considering architecture assumptions verified. No runtime or hardware testing performed.

### NVMe source coverage milestone and DMA contracts
- Every C source and header in rom/devs/nvme, the configuration and mmake definitions have been read. Main constructor, namespace, queue, PRP/SGL, dispatch, completion, reset and disposal paths reviewed. No active per-I/O timeout/recovery mechanism exists after submission; AbortIO explicitly does nothing. This is a documented limitation rather than proof every stalled-device scenario is independently a bug.
- Generic PCI AllocPCIMem provides 4096-byte alignment and MEMF_31BIT memory, not arbitrary negotiated NVMe-page alignment. Controllers requiring a larger page remain an alignment compatibility candidate; no external specification was consulted in this source-only pass.
- CachePreDMA/CachePostDMA contracts and generic x86, PPC-CHRP and AArch64 implementations inspected. They require paired pre/post operations and explicitly translate virtual addresses on PPC/AArch64. This substantiates the architecture conditions in DISK-007 and DISK-014; generic x86 implementations are no-ops.
- NVMe submission/completion queues and Identify buffers are accessed without explicit cache pre/post pairing, whereas transfer data uses the helpers. Applicability depends on the PCI backend providing coherent/uncached allocations; this remains conditional rather than an unconditional claim for x86.
- NVMe failure cleanup also leaves PCI ownership/BAR mapping/controller object allocations unreleased in several constructor branches. The noexpunge configuration limits ordinary lifecycle exposure, but startup failure can still leak resources.

### ATA transfer-engine continuation

### ATA coverage checkpoint
- All 2,778 lines of lowlevel.c read, including transport, splitting, identify/geometry, modes and reset/probe. Device dispatch, bus lifecycle, PCI DMA/PIO and platform adapters remain under review.
- High-address DMA bounce allocation is unchecked and failed reads still copy bounce data to the caller; provider behavior needs tracing.
- Poll mode waits only for BSY and does not explicitly service PIO data or wait for DMA completion; boot-option and provider contracts need tracing.

### PCI ATA DMA review evidence
- interface_dma.c and interface_dma.h read fully; constructor allocation and PRD alignment traced. The apparent items > PRD_MAX off-by-one does not itself prove an allocation overrun: backing storage explicitly accommodates PRD_MAX + 1 entries plus alignment slack. Do not count it as an unconditional memory-corruption bug.
- Setup rejects physical addresses beyond 32 bits but does not check a NULL caller/bounce buffer. The common high-address bounce allocation failure can therefore proceed with address zero on implementations that identity-map it; allocation failure remains a candidate pending the remaining provider review.

### PCI ATA bus/PIO coverage checkpoint
- ata_pci_busclass.c and interface_pio.c read fully: IRQ dispatch, constructor failure, disposal, attributes, interface setup, mode and shutdown paths. PIO normal callers provide even lengths; generic non-x86 odd-length loops are therefore not counted independently without a reachable odd-length caller.
- Constructor interrupt-registration failure calls only the superclass disposer, bypassing PCI subclass DMA-buffer cleanup. This leaks the allocated PRD buffer on that failure path; probe ownership cleanup remains to be traced.
- IRQ registration precedes assignment of the subclass callback through Root.Set; a spurious early interrupt could call an unset callback. Superclass constructor sequencing remains to be inspected before assigning a finding.

### ATA bus-construction sequencing evidence
- ata_busclass.c now read fully, including interface ownership, start/open, reset and enumeration; PCI controller subclass also read fully.
- PCI callback assignment occurs in Hidd_ATABus_Start after the constructor registers the hardware IRQ. No superclass constructor assignment closes the early-interrupt window. Its manifestation depends on a pending/spurious device interrupt before scanning disables IRQs; this candidate is now detailed as OD2575 in AROS-other-drivers.md. No hardware interrupt was induced.
- Root.Dispose frees interface allocations and removes reset callbacks but does not stop the permanent bus task or dispose units. Reachable disposal and controller removal callers still require review.

### ATA controller/unit/timing coverage
- ata_controllerclass.c, ata_unitclass.c, bootwait.c, timer.c, waitto.c and generic waitnano.c read fully.
- RemoveBus explicitly rejects unloading, so ordinary bus hot-unload is unsupported. The absence of bus-task teardown is not promoted to an unconditional runtime use-after-free without a reachable disposal caller.
- Unit identify allocation has a checked failure path and matching pooled free. Timer creation/open failures unwind request and port allocations; wait helper aborts and joins its timer before reuse.
- Controller daemon task creation failure returns NULL without disposing the already-created superclass object. This is an allocation/resource leak on startup failure. Controller disposal only calls its superclass for objects registered in ata_Controllers, so cleanup of a partially constructed unregistered controller needs explicit handling.
- Generic calibration doubles a ULONG scale through 0x80000000; if the last iteration still measures below 10ms, it wraps to zero and loops forever. This requires an extreme timing/calibration failure and remains a low-applicability candidate rather than a normal-hardware assertion.
- DISK-097 boot impact confirmed by the full boot-wait resident source. No build or runtime tests performed.

### ATA platform build/coverage checkpoint
- Amiga dma_generic.c and waitnano.c read fully. The current Amiga mmake override compiles only waitnano; dma_generic is legacy inactive source in this definition.
- Sam440 bus_ppc460.c and its mmake read fully. Its build_archspecific invocation is commented out, and its interface uses obsolete ata_BusDriver/ata_RegisterBus APIs. Findings in that file are not claimed as active current-build defects.
- Amiga waitnano truncates nanoseconds before deciding whether to execute any busywait. The common 400ns call becomes zero and performs no deliberate delay, whereas its own comment says each CIA access would already exceed that duration. This is a platform timing candidate; instruction execution may provide sufficient incidental delay on some hardware, so impact requires hardware validation.
- The subsequent complete 727-line ata_amiga.c pass and build reconciliation resolve this earlier unread legacy source.

### Amiga ATA inventory correction
- All 727 lines of legacy arch/m68k-amiga/devs/ata/ata_amiga.c now read, including Gayle probing, IDE doubler, interrupt and PCMCIA configuration paths.
- Current Amiga boot mmake names ata_gayle, implemented by arch/m68k-amiga/hidd/gayle_ata. Its module definition compiles class_init, probe, gayleata_busclass, fastata_busclass and interface_pio. These active sources are added to the required inventory and remain to be reviewed.
- Legacy ata_amiga uses obsolete ata_RegisterBus/ata_BusDriver interfaces and is absent from the current ATA override (only waitnano). Its unchecked graphics-library dereference, disabled-interrupt PCMCIA busy waits and unbounded debug tuple-string parsing are recorded as legacy observations, not asserted as active-build findings.
- This distinction prevents treating obsolete platform source as evidence that the active Gayle implementation has the same defects.

### Active Gayle/FastATA coverage checkpoint
- interface_pio.c, gayleata_busclass.c and fastata_busclass.c read fully. Subsequent Gayle/FastATA milestone reads complete probe, class setup, headers and module configuration.
- Gayle also registers/enables IRQs before Hidd_ATABus_Start assigns its callback, sharing the conditional early-interrupt window already recorded for PCI ATA.
- FastATA interrupt creation returns FALSE and its handler is a stub; constructor ignores the failure. Probe reachability must be checked before classifying the resulting absent-completion behavior.

### Gayle/FastATA source coverage milestone
- Every C source/header and module config in arch/m68k-amiga/hidd/gayle_ata read, together with boot/module wiring.
- FastATA selection is guarded by ENABLE_ATAPOWERFLYER, commented out in probe.c and absent from module flags. Its IRQ stub is therefore not reachable through the default probe; retain as an optional unfinished configuration limitation.
- Port doubler detection is also gated by probedbus->doubler, which starts zero and is not initialized to enable detection. Default active code consequently never enters that detection branch.
- AddBus failure checks probedbus->atapb_Node.ln_Succ after the successful constructor has transferred ownership and the disposer may already have freed probedbus. The shared HW_AddDriver failure/disposal contract needs further tracing before this is promoted.

### Shared AddBus contract verification
- StorageController AddBus synchronously disposes successfully constructed buses when SetUpBus fails; this is proven by implementation and documented contract, not inferred solely from a comment in the probe.
- No successful bus-removal implementation is provided by the shared controller. This supports keeping ordinary hot-unload teardown claims conditional.

### PCI manifestation of DISK-100 confirmed
- rom/hidds/ata_pci/probe.c, ata_bus_Detect, uses the identical post-AddBus failure marker read. PCI bus disposal frees probedbus before AddBus returns NULL on SetUpBus failure. Unlike the Gayle manifestation, this does not depend on DISK-099. Empty legacy ports with KeepEmpty false are a routine reachable trigger; PIO/task allocation failures also reach it.

### PCI ATA probe coverage
- probe.c read fully, including native/compatibility channels, SATA legacy switch, ownership, fallback ISA ports and AddBus registration.
- DO_SATA_HANDOFF is inactive by default; guarded code contains logical && tests in place of bit masks and stale helper calls, so those are optional-build observations, not active default findings.
- CAP_SAM rejection unmaps the original physical BAR rather than the mapped hwhba pointer used on the normal path. Mapping backend behavior remains to be traced before asserting impact.

### PCI ATA source coverage milestone
- All C sources, private headers, configuration and mmake definition in rom/hidds/ata_pci have been read. Current module compiles probe, initialization, controller/bus classes and both PIO/DMA adapters.
- PCI initialization correctly obtains IID_Hidd_StorageController, unlike the Gayle assignment in DISK-099. Header layout confirms ctrllrDevice is embedded, substantiating DISK-101.
- The runtime PCI scan obtains PCIDeviceAttrBase and PCIDriverAttrBase but the noexpunge expunge hook releases only the later mandatory suffix; this lifetime leak is DISK-322. The method-base failure branch calls OOP_ReleaseAttrBasesArray starting at hiddAttrBase with the entire attrBaseIDs list, including two PCI entries preceding that field in the allocation layout. This releases wrong interface/base pairs, reads into the adjacent method-base fields and overwrites them; promoted to DISK-320. The initial partial-attribute-array failure in AHCI skips release and has no registered ExpungeLib rollback; retained as DISK-321. A caller recheck found that ATA, SCSI, WD33C93 and PCI ATA do have registered ExpungeLib cleanup hooks that release their acquired prefixes after failed INITLIB, so those initial-acquisition cases are excluded from DISK-321.
- Several early initialization failures leave utility.library open or storage-root objects unreleased. noexpunge limits ordinary unloading but does not eliminate startup-failure leaks.
- DMA descriptor cap/physical contiguity and platform mapping restrictions remain conditional protocol/backend concerns; no hardware specifications or runtime tests were used.

### ATA request-dispatch continuation
- ata.c lines 1–900 reread without truncated output, covering read/write32/64, change listeners, geometry, direct SCSI, command tables, BeginIO and AbortIO.
- Ordinary read/write bounds sums operate on sector counts after shifting byte offsets, so the initially suspected ULONG overflow is not reachable with normal 512-byte sectors and ULONG byte lengths. No separate wrap finding assigned.
- AbortIO scans/removes only queued requests, with the message-port spinlock on SMP, and replies after releasing locks. In-flight cancellation is explicitly unsupported.
- CMD_UPDATE still does no hardware flush (DISK-010). `HD_TRIMCMD` is present in ATA's map and NS device-command list; its supported-drive branch returns success while the operation is compiled out, and its `ATAFEATURE_TEST_AVAIL` path writes no magic value. This is DISK-325. `HD_SMARTCMD` is also mapped/listed, but the ATA handler currently returns `IOERR_NOCMD`; this advertised-command mismatch is now detailed in DISK-326.

### ATA worker/daemon coverage checkpoint
- ata.c now read completely; ata_trim.c, ata_smart.c and mmake read completely. The SMART handler returns `IOERR_NOCMD` for present-media requests despite the advertised command list (DISK-326). ATA dispatches TRIM and returns success on capable drives without executing it (DISK-325). The separate `rom/devs/scsi/scsi.c` map does not contain either command; its TRIM/SMART code is not a current public-I/O path.
- Daemon timer/calibration failure signals the constructor handshake but leaves ac_Daemon unset. Constructor registers the controller as successful; later disposal signals that unset task and waits for a handshake from a nonexistent daemon. Resource failure also silently disables media polling.
- Failed unit setup disposes the unit while ab_SelectedUnit may still reference its instance. IRQ handler dereferences that selected pointer if a transfer handler remains, linking this failure path to DISK-094 and requiring final setup cleanup tracing.
- Daemon request list temporarily reuses request nodes for DoIO and restores links afterward under DaemonSem. No separate corruption claim assigned without an intervening list mutation.

### ATA manifestation of DISK-089
- rom/devs/ata/ata_init.c, ata_RegisterVolume, likewise declares an uninitialized pp[24] and never assigns pp[3] before MakeDosNode. This supplies indeterminate device-open flags. Several DosEnvec entries within the declared table are also omitted, including SECORG, PREALLOC and INTERLEAVE, so registration passes indeterminate geometry/configuration values in addition to flags.

### ATA initialization/volume coverage checkpoint
- ata_init.c read fully, including registration, option parsing, resource setup, open/close and expunge.
- Successful ata_RegisterVolume returns before CloseLibrary(ExpansionBase), leaking one library open per registered unit. Failed node construction also retains its allocated shared ID.
- Expunge contains post-HW_RemoveDriver accesses and additional disposal/free of a controller the generic HW removal normally disposes. Current ata configuration applicability must be checked before classifying this as a reachable lifecycle finding.

### ATA interface/configuration and Efika coverage
- ata.conf and ata_bus.h read fully. Unit_InS/Unit_OutS directly invoke platform vectors with the exact supplied byte length; there is no short-length adapter shielding Gayle from DISK-098.
- ATA device configuration does not specify noexpunge, unlike PCI/Gayle HIDDs. Expunge's double-disposal concern therefore cannot be dismissed solely by module configuration; generic HW removal permission and caller sequencing still need inspection.
- Both Efika backend C sources, make.opts and mmake read fully. The build override explicitly lists lowlevel_mpc5200b and ata_bestcomm, but source still relies on obsolete ata_BusDriver, ab_IntHandler/ab_IRQ and ata_RegisterBus absent from the current interface. This is a stale integration/build candidate, not proof of a functioning current Efika driver.
- Efika source observations: replaces an already-created ata_MemPool without freeing it; required OF resource/properties are unchecked before MMIO and BestComm accesses; missing taskid leaves 0xffffffff used as a task-table index; busy MMIO waits have no timeout. Runtime claims remain conditional on restoring this legacy interface/build.
- No project code changed, build attempted or tests run.

### Shared hardware removal evidence
- HW AddDriver/RemoveDriver implementations and storage configuration inspected. Successful removal is a disposing operation, not merely detachment.
- Storage's shared ID ReleaseID remains an empty stub; previously allocated IDs are never reclaimed. This affects lifecycle/failed registration cleanup, although normal persistent boot devices may not require reclamation.

### Shared storage source coverage milestone
- storage_init.c, storage_intern.h, bus/unit classes and mmake fully read; combined with earlier storage_class/controller/config/header review, all module sources and build inventory covered.
- Initialization allocates a shared pool and obtains utility/HW resources without unwinding them after several failures. Expunge is empty, but module config is noexpunge; startup failure leaks remain applicable.
- The protected pool serializes pool allocation, not cs_IDs/SIDS_IDs list operations. ID allocation's own comment acknowledges missing locking. Concurrent ATA bus workers can register volumes through this allocator, so duplicate ID selection and list-link races are a source-supported concurrency candidate in addition to DISK-001 decimal formatting.
- Base bus/unit classes are forwarding stubs; concrete drivers own geometry, units and lifecycle behavior. No additional transfer/bounds behavior exists here.
- Shared RemoveBus remains unsupported; AddBus list locking and enumeration locking are explicit and consistent for supported operations.

### ATA private-header continuation
- ata.h base/controller/bus definitions through DriveIdent introduction inspected. ab_BusNum is UBYTE while the global bus counter is int; systems with over 256 registered ATA channels alias bus numbers and OpenDevice lookup. This is an extreme-scale limitation, pending unit-number type checks.

### ATA header coverage milestone
- ata.h read fully, including packed Identify layout, unit transfer/accounting fields and flags. Both exported hardware/ATA HIDD headers read fully; register/vector contracts match the transport callers.
- au_UnitNum is ULONG but derives from UBYTE ab_BusNum, confirming the over-256-channel alias limitation. This is not presented as a common hardware defect.
- Identify capacities are initialized from sector-count fields, supporting DISK-018's READ CAPACITY off-by-one interpretation rather than treating au_Capacity as a last-LBA value despite its misleading header comment.
- Sector shift is fixed to 9 for ATA disks; the Identify logical-sector size fields are endian-converted but unused. Native logical sectors larger than 512 remain a compatibility/correctness candidate requiring controller/device configuration assumptions.
- Main ATA C sources, private/exported headers and module definition have now been read. Remaining final ATA work: direct-SCSI review reconciliation, polling/bounce failure findings and cross-driver command contracts. Full disk subsystem coverage is still incomplete.

### ATA bounce transfer accounting evidence
- Failed bounced reads copy the full chunk into the caller even when ata_exec_cmd reports failure; the bounce data can be zero/stale/partial. Read wrappers set io_Actual to zero on any error even if earlier chunks completed. This underreports partial progress and warrants a separate accounting candidate in final consolidation.
- Bounce allocation spans the entire request even though commands are split, increasing resource exhaustion exposure; this alone is an efficiency issue, not independently classified as corruption.

### ATA SCSI reconciliation
- scsiemu.c reread fully. DISK-017 includes direct transfer bypass of dispatcher bounds/buffer-length checks; direct large READ/WRITE LBAs use 28-bit command encoding and can alias high requested addresses.
- SYNCHRONIZE CACHE is also a successful no-op in SCSIEmu, extending DISK-010 beyond CMD_UPDATE.
- MODE SENSE writes block descriptors at out rather than the advanced p pointer, overwriting its header; response formatting remains an additional candidate for consolidation.

### AHCI class/build continuation
- Current mmake inventory verified: initialization, I/O/SCSI/CAM shims, AROS bus/DMA helpers, core, attachment, port multiplier, PCI probe, timers/formatting and controller/bus/unit classes.
- Controller, bus and unit class sources read fully. Controller disposal removes its list node but never invokes superclass disposal or releases attached hardware. Bus disposal invokes superclass without removing its registered reset callback or clearing ap_Object. Reachable disposal callers will be traced before lifecycle candidates are finalized.
- Unit string copies read fixed-width Identify fields directly, which are fixed-width backing arrays; unlike DISK-008 this does not itself establish an overread.
- Hidd_AHCIBus_Start returns TRUE unconditionally; actual scanning and unit creation occur in the core/CAM path. Disabled class scaffolding is not counted as missing disk functionality without tracing that path.
- Core ahci.c and ahci_pm.c still require full reading; AHCI source coverage is not yet complete.

### AHCI core coverage checkpoint
- ahci.c lines 1–820 read fully: global reset, port/CCB/DMA allocation and failure unwind, port initialization, interrupt masks and link power management.
- Several M_WAITOK/M_INTWAIT allocations are immediately dereferenced (temporary port, real port, target structures). The AROS kmalloc shim must be checked before treating inherited BSD nonfailure assumptions as valid or reporting allocation failures.
- Port helper startup return is ignored; worker creation and completion contracts remain to be traced in ahci_aros.c.
- Error CCB is assigned only for slot 1. Controllers exposing fewer than two slots need compatibility/reachability checks before asserting a NULL error-CCB failure.
- No new unconditional candidate assigned from these core sections yet; remaining state machine, interrupt, timeout/reset and command issuance sections are still pending.

### AHCI core state-machine continuation
- ahci.c lines 821–1250 read fully: direct/port-multiplier state transitions, hotplug notification, exclusive probe access, port free, command/FIS start-stop and CLO entry.
- ahci_port_free computes its AHCIBase through ap->ap_sc before checking ap_sc. Port allocation has early failures before assigning ap_sc, so failure cleanup can dereference NULL. AROS bus_space_subregion's actual failure behavior must be traced to establish configuration reachability.
- Port free ignores ahci_port_stop failure and proceeds to free DMA buffers even when CR/FR may remain active. Actual detach/error callers and global-controller disable sequencing remain to be checked before finalizing a DMA-after-free candidate.
- Port-multiplier state-machine EINFO loop has no iteration bound if a device continually reports changes; this is a responsiveness limitation pending PM register/acknowledgement review.
- Remaining core reading begins at line 1251.

### AHCI reset review continuation
- ahci.c lines 1251–1650 read fully: CLO completion, reset dispatch, soft-reset FIS sequence and beginning of COMRESET/retry setup. Interrupt handling has not yet been reached in this core sequence.
- Softreset's device-signature mismatch assigns EBUSY and then immediately overwrites error with zero, returning success and advancing to identification. This is a source-supported reset-status candidate; downstream identification may still reject the replacement and limits the impact claim.
- Softreset dereferences ahci_get_err_ccb without checking availability; error-slot initialization/capability handling and callers remain to be traced.
- COMRESET ignores several port-stop/start/wait failures and continues reset; consequences must be evaluated with the remaining retry and hard-reset code before independent findings are assigned.
- Next core section starts at line 1651.

### AHCI hard-reset/removal continuation
- ahci.c lines 1651–2050 read fully: COMRESET detection/retry, hard-reset completion, hardstop and active/saved/pending request draining.
- Hardstop explicitly completes active, saved and queued CCBs as TIMEOUT rather than silently abandoning requests. Completion callback ownership and DMA cleanup sequencing still need cross-check against the remaining core.
- COMRESET's detection loop subtracts ahci_os_softsleep and tests loop == 0 rather than <= 0 after exit. AROS sleep helper return values must be checked to determine whether overshoot can misclassify timeout.
- Hardreset start failure assigns EBUSY inside the success switch arm and exits without taking the ordinary failed-port hardstop branch. Resulting probe state and downstream state-machine retry behavior remain candidates for caller tracing.
- Next core reading starts at line 2051; no new numbered finding assigned from this slice.

### AHCI DMA/poll/issuance continuation
- ahci.c lines 2051–2470 read fully: signature detection, PRDT loading/unloading, polled completion, timeout setup, exclusive access and pending-queue entry.
- PRDT byte-count limits and single-segment AROS mapping remain covered by DISK-014/015; callback ignores its error argument and trusts segment count, requiring shim contract tracing.
- ahci_poll documentation permits NULL timeout_fn but implementation calls it unconditionally after timeout. All actual callers must be checked before assigning a reachable NULL-callback finding.
- Exclusive-access timeout-starvation suspicion not promoted after caller/lock tracing: the only selected caller is `ahci_port_state_machine`, invoked once by the port worker during initial startup before CAM attachment and public unit I/O; the initial PM probe serializes its own synchronous commands. The worker does not run this state machine again after startup (see DISK-125), so no selected ordinary active request was found that can be stranded in `ahci_beg_exclusive_access`.
- Next core reading starts at line 2471; interrupt body has not yet been reached.

### AHCI issuance/interrupt continuation
- ahci.c lines 2471–2900 read fully: NCQ/non-NCQ batching, global IRQ dispatch, helper-thread intervention, per-port interrupt entry and taskfile-error slot selection.
- Normal IRQ path uses a nonblocking port lock and defers blocking recovery by masking port IRQs and signalling the helper. This contract makes helper creation and signalling correctness essential; AROS worker implementation remains pending.
- Non-NCQ issuance breaks after ordinary commands but may continue after an exclusive/autosense command, the reverse of its comments. Pending queue/state conditions and callers must be traced before claiming simultaneous exclusive issuance.
- Error processing ignores port-stop return and marks stopped regardless; this strengthens the earlier conditional concern that recovery can treat a still-active engine as stopped. Remaining completion/restart logic must be read before assigning the root-cause finding.
- Next core reading starts at line 2901.

### AHCI core completion/recovery review
Coverage: rom/devs/ahci/ahci.c lines 2901–4343 (end of file) read consecutively, completing the core source review. Reviewed interrupt recovery, hotplug, CCB reservation, NCQ error log, DMA allocation, timeout and feature-command paths. Platform glue, timers, attachment and port multipliers remain pending.

### AHCI AROS timer/helper coverage
Read rom/devs/ahci/timer.c, timer.h and ahci_aros.c completely; traced kthread_create and timeout issuance in ahci_aros.h/ahci.c and initialization waits in attachment/bus callers.

### AHCI attachment and PCI discovery review
Coverage: ahci_attach.c (860 lines), bus_pci.c (218 lines), ahci_init.c (257 lines) read completely. Re-read resource/interrupt adapters in ahci_aros_bus.c, CAM registration/detachment lines 1–260 and 330–435, and port interrupt enable/worker callers. Working tree remains clean at the baseline commit.

### AHCI port-multiplier and CAM probing review
Coverage: ahci_pm.c lines 271–1083 read fully, completing this file. ahci_cam_aros.c lines 435–1020 read fully, completing the file together with prior registration/volume review. Traced logical-sector use in ahci_io.c and ATA identify layout in include/devices/atascsi.h.

### AHCI SCSI and public I/O review
Coverage: ahci_scsi.c (738 lines) and ahci_io.c (477 lines) read completely in bounded chunks. Traced sense layout in ahci_scsi.h and residual accounting in ahci.c. This confirms DISK-006, 009, 010 and 012–016 rather than introducing duplicate entries.

### AHCI DMA allocation, media changes and final contract tracing
Rechecked command-header/table/RFIS tag sizes against slab allocation and all per-slot accesses; read ahci_intern.h, include/hidd/ahci.h and ahci_aros.h shim definitions; rechecked build/config and state-machine callers. No source changes, builds or tests performed.

### VirtIO initialization/timer and lifecycle follow-up
Coverage: virtio_init.c, virtio_timer.c/h, virtio_hw.h and virtio_queue.h read fully; controller/bus constructors, disposers and startup re-read fully; queue allocation/free and feature/configuration readers rechecked. mmakefile.src confirms these sources form kernel-virtio; configuration disables routine library expunge.

### VirtIO capability/header contract review
Coverage: virtio_hw.c re-read fully; virtio_intern.h, virtio_debug.h, include/hardware/virtio.h and include/hidd/virtio.h read fully. PCI Base getter contract inspected in rom/hidds/pci/pcideviceclass.c:795–830. Build search finds kernel-virtio in i386-pc, x86_64-pc and riscv64-opensbi boot definitions (MM directives, not evidence that each image actually enables it). All files in rom/devs/virtio now have documented source review; remaining global build/config applicability and final report consolidation remain open.

### Hostdisk headers, automount and lifecycle completion
Coverage: arch/all-hosted/devs/hostdisk/automount.c, hostdisk_device.h, hostdisk_host.h, hostdisk.conf, hostdisk.txt and mmakefile.src read fully. UNIX/Windows host headers and UNIX/Windows/Linux/Darwin build overrides read fully. Rechecked worker setup/exit and Open/Close/Cleanup contracts. Combined with earlier source review, all inventoried hostdisk implementation files have review outcomes; runtime host ABI and dynamic libc symbol availability remain untested as required.

### Generic SCSI subsystem continuation
Coverage: rom/devs/scsi/scsi.c lines 261–1082 read fully, completing this file with earlier lines 1–260. lowlevel.c and mmakefile.src re-read fully. Traced bus startup (scsi_busclass.c:939–1005), unit transfer stubs and constructor (scsi_unitclass.c:35–105). Subsequent SCSI passes and the final local inventory reconciliation complete init/classes/timers/public/private header coverage.

Current implementation outcome: this subsystem is an unfinished ATA-derived skeleton, not a functioning general SCSI data path. scsi_InitBus marks devices only NONE/UNKNOWN; its reset/type-identification call is compiled out. Start refuses to start a worker when both device types are below DEV_ATA. scsi_setup_unit's real identification/transfer setup is compiled out and merely returns TRUE. Unit read/write/eject callbacks remain unsupported-command stubs; HD_SCSICMD always returns BADADDRESS. No architecture SCSI overrides or boot kernel-scsi references were found in the scoped arch search. This does not replace final global inventory verification.

Confirmed source observations, reachability explicitly limited:
- NSCMD_DEVICEQUERY writes a full result without checking data/length, matching ATA DISK-102; requires an exposed unit, which current discovery does not produce.
- CMD_UPDATE is a no-op, matching DISK-010, but no current successful data-write path exists here.
- Bus worker creation failure retains DetectionSem shared acquisition, matching ATA DISK-097, but current built-in discovery cannot reach its usable-device branch.
- scsi_InitBus opens a timer unchecked and calls WaitTO even when NULL. This is reachable if a bus backend invokes Start under timer allocation failure, regardless of usable disk detection.
- BusTaskCode disposes an instance-data pointer instead of unitObj on setup failure; current scsi_setup_unit always TRUE masks this branch.
- cmd_TRIM success body is compiled out; neither SMART nor TRIM is dispatched by current command map, so do not label these as current public-I/O successes.
- GetBlkSize returns sector shift rather than bytes; exported extension contract must be read before deciding whether that is a defect.
- Change-monitor request node save/restore avoids losing permanent daemon-list linkage around DoIO; still requires daemon/task teardown review.
- Raw copy of DriveIdent on special geometry length 514 is inherited API behavior, not automatically a bounds bug (copies 512 into a declared 514-byte destination).
No new unconditional functional-I/O finding assigned from these latent skeleton branches.

### SCSI initialization, daemon and timer review
Coverage: scsi_init.c (372), scsi_controllerclass.c (123), timer.c/h, waitto.c, waitnano.c and bootwait.c read fully. scsi_unitclass.c read fully across constructor and attribute/disposal chunks. scsi_busclass.c lines 1–200 and 410–665 re-read at this checkpoint. The subsequent coverage-completion record closes lines 201–409 and 666–938, full scsi.conf and all headers.

### Generic SCSI component review outcome
Coverage completion: scsi_busclass.c remaining lines 201–409 and 666–938 read consecutively; combined with prior chunks/start/end, entire 1037-line file reviewed. scsi.h, scsi_bus.h, include/hidd/scsi.h, include/hardware/scsi.h, include/devices/scsicmds.h and full scsi.conf read. All files inventoried under rom/devs/scsi now have source-only review outcomes.

Outcome: unfinished ATA-style skeleton, with no working discovery-to-transfer path found. DISK-137 covers permitted unload retaining daemon; DISK-138 covers the current case-sensitive compilation obstacle. Runtime candidates below are conditional on the build obstacle being resolved and a caller instantiating bus/controller objects, not evidence of shipping SCSI disk corruption.
- Controller SetUpBus compiles out GetPIOInterface/GetDMAInterface, then calls Start. Start invokes scsi_InitBus, whose PIO_OutAlt/PIO_Out/PIO_In dereference pioInterface. Unless a backend has explicitly instantiated that interface itself, bus registration follows a NULL interface. No such backend was located in arch inventory.
- Default bus attribute getters dereference busVectors/pioVectors without checks. These vector tags are documented mandatory; absent vectors are an invalid subclass contract, so distinguish that from the controller's own skipped interface setup.
- Repeated GetPIOInterface/GetDMAInterface allocate replacement objects without freeing earlier ones. No repeated live caller found; retain potential object resource leak.
- Bus numbering uses UBYTE storage with an int counter, aliasing after 256 buses; no current actual controller inventory supports that scale.
- GetBlkSize's shift-return contract remains undocumented beyond signature and inherited implementation; no independent byte-size finding asserted.
- Bootwait locks DeviceList appropriately (system read lock on SMP when available) and waits DetectionSem; startup worker-failure leakage would block it if the currently unreachable usable-device branch were activated.
- Transfer geometry/bounds, flush, media monitoring and cancellation cannot be evaluated as successful runtime disk operations because callbacks remain stubs and public SCSI passthrough is disabled. This limitation is explicit rather than a claim of verified correctness.

### Disk-image CCD/MDS metadata and transfer review
Coverage: plugins/ccd.c (527 lines) and plugins/mds.c (536 lines) read fully. Build definitions explicitly include both plugins with stub_aros. Reviewed file ownership, directory restoration, linked-track construction, endian decoding, sector extraction, geometry and CDDA transfers.

### Disk-image NRG/GI review
Coverage: plugins/nrg.c (624 lines) and plugins/gi.c (271 lines) read fully; shared DOS2IOErr implementation traced in device/unit.c:477–501.

### Disk-image CPC/Crypt review
Coverage: plugins/cpc.c and plugins/crypt.c read fully; caller TDRead/TDWrite confirmed to dispatch without independent capacity checks. CPC has an active build rule/default plugin dependency. Crypt has no rule in the inspected plugin mmakefile and findings are conditional on separately building/enabling that source. GI has a build rule but is excluded from the default plugin dependency list.

### Disk-image UIF review
Coverage: plugins/uif.c read fully, including footer, compressed index, hash construction, cached blocks and read accounting. UIF is included in the default plugin build dependencies. Worktree remains clean at the baseline commit.

### Archive wrappers and temporary-image ownership
Coverage: plugins/xad.c, plugins/xpk.c and device/tempfile.c read fully; InsertDisk/RemoveDisk ownership traced. Both wrappers have build rules but are absent from default plugin dependency list.

### IPF disk-image backend
Coverage: plugins/ipf.c read fully and device geometry defaults traced. No IPF build rule exists in the inspected AROS plugin mmakefile; findings apply if this backend is separately enabled with capsimage.device.

### DAA backend review
Coverage: plugins/daa.c read fully. Build rule exists but default plugin dependency disables DAA.

### DMG decoder helpers (main parser review still pending)
Coverage: dmg/adc.c, adc.h, base64.c and base64.h read fully; XML-data and ADC-read call sites traced. DMG is in default plugin dependencies.

### DMG main parser and transfer review
Coverage: dmg/dmg.c read fully. Together with the preceding decoder-helper review this completes the DMG backend source pass.

### DMS extraction wrapper and initial decoder review
Coverage: plugins/dms.c, dms/u_rle.c and dms/getbits.c read fully. Remaining DMS dictionary/Huffman decoders and headers still pending. DMS is built by default.

### DMS dictionary and Heavy Huffman decoders
Coverage: dms/u_quick.c, u_medium.c, u_heavy.c, maketbl.c, u_init.c and xdms.h read fully. DEEP, static tables and checksum helpers remain pending.

### DMS decoder coverage completion
Coverage: u_deep.c, crc_csum.c and tables.c read fully. All DMS decoder sources named by the plugin build rule and xdms.h are now reviewed.
- DEEP's match loop (u_deep.c:78–81) has the same unchecked output expansion as DISK-163; include DEEP in that finding.
- DEEP uses the same unbounded DROPBITS input reader covered by DISK-161.
- DEEP initializes its adaptive tree internally; no metadata-supplied tree array counts like Heavy are present. Frequency reconstruction uses fixed constants matched to struct arrays and a sentinel frequency. No independent tree corruption finding established.
- CRC table indices are masked to 255; checksum loops obey caller lengths. Wrapper checks track and header input lengths before CRC use; no independent checksum buffer defect found.
- MEDIUM/DEEP static table indices use byte-sized bit results, within 256-entry d_code/d_len tables; no separate lookup-table overflow.
- DMS completeness now covers wrapper, input-bit macros, RLE, QUICK/MEDIUM/DEEP/HEAVY, table construction, dictionary reset, checksum and static tables. No execution or tests performed.

Editorial correction: DISK-160 title now names its proven zero-run nonprogress mechanism; ordinary partial-block DMG requests were explicitly found correct.

### FDI wrapper and decoder header construction
Coverage: plugins/fdi/fdi.c read fully; fdi2raw.c:1939–2104 header/free/getters and revolution-copy path inspected. Remaining 2190-line decoder body is still pending. FDI has a build rule but is deliberately excluded from default dependencies.

### FDI decoder internals, first slice
Coverage: fdi2raw.c:1–400 read consecutively. Remaining body and decoder call contracts pending.

### FDI decoder second slice
Coverage: fdi2raw.c:401–850 and fdi2raw.h read fully; call search confirms amiga_check_track is disabled at line 2172.

### FDI sector-token dispatch review
Coverage: fdi2raw.c:851–1370 read consecutively. Pulse decoding and final load path remain pending.

### FDI pulse decoder slice
Coverage: fdi2raw.c:1371–1770 read consecutively, distinguishing disabled older fdi2_decode from active implementation.

### FDI decoder coverage completion
Coverage: fdi2raw.c:1771–1940 and :2105–2190 read fully. Combined consecutive slices cover all 2190 lines; fdi.c/header/build rule also reviewed.

### CUE libc adapters
Coverage: plugins/cue/fdio.c, malloc.c, string.c, stdio.c and select.c read fully; initialization calls traced in cue.c.

### CUE WAV/AIFF stream readers
Coverage: audio/wave.c and audio/aiff.c read fully. CUE uses these stream callbacks for CD audio; build-path applicability remains to verify.

### FLAC audio and embedded-cuesheet wrappers
Coverage: audio/flac.c and plugins/cue/flac.c read fully; remaining audio codecs and main CUE index-fix contract still pending.

### Vorbis and mpg123 audio adapters
Coverage: audio/vorbis.c, audio/mp3_mpg123.c and corresponding stream headers read fully; local mpg123 API signature checked.

### WavPack audio and embedded-cuesheet wrappers
Coverage: audio/wavpack.c and plugins/cue/wavpack.c read fully.

### CUE main parser and index normalization
Coverage: plugins/cue/cue.c:1–690 read consecutively; transfer body completion remains pending.

### CUE transfer coverage completion
Coverage: cue.c:690–end and cue.h read fully; entire CUE main parser/transfer source now reviewed.

### Alternative MPEGA backend and audio header coverage
Coverage: audio/mp3_mpega.c and include/audio/mp3_mpega.h read fully; other audio stream headers inspected (some combined output truncated, requiring final header completeness check). MPEGA alternative is excluded by current USE_MPG123 build flags.
- Stereo buffers are allocated to MPEGA_PCM_SIZE and released on close/error. Decode count is trusted against that external contract; no independent allocation leak found.
- Seek implements replay from zero for backward requests and discards forward samples; cached frames preserve seek offsets locally.
- Length derives from duration estimate and narrows to ULONG; gapless exact sample count is not ensured.
- EOF is padded with silence; non-EOF decode errors return short bytes. LONG pointer casts share FLAC/Vorbis architecture limitations.
- MPEGA module-global library base is set per image, with individual image library references. Cross-image close/lifecycle and library contract remain conditional observations.
- No additional substantiated independent candidate promoted in this backend.

CUE build applicability investigation: config/make.tmpl build_prog stores source names as supplied; default usetree=no strips directory only from object names, not its source inventory. Existing plugins/mmakefile.src uses audio/<name> but files reside ../audio/<name>. Exact gen_cxxrules source resolution must be inspected before final build-failure classification.

### Disk-image build source resolution
Coverage: config/make.tmpl build_prog and rule_compile_multi source-directory construction inspected.

### Disk-image string/path/byte-swap helpers
Coverage: support/swab2.c, m68k/swab2.s, paths.c, strlcpy.c, copystringbstrtoc.c, trimstr.c implementation bodies and support/mmakefile.src reviewed.
- swab2 handles pairs and ignores an odd trailing byte; loops match requested even byte counts. In-place use is safe under proper alignment. Portable C casts to ULONG/UWORD require alignment that general caller buffers may lack; architecture/caller applicability pending.
- TrimStr leaves exactly one leading whitespace character (condition i>1) and uses overlapping strcpy for longer prefixes. Current located callers use it for GUI hotkey strings; record as peripheral support defect rather than an independent disk correctness candidate.
- CopyStringBSTRToC subtracts one from dst_size without rejecting zero; current located callers pass nonzero fixed buffers, so no reachable zero-destination finding claimed.
- CombinePaths allocates combined length plus slack and ignores AddPart failure; no ordinary short-buffer trigger found for current bounded filename inputs.
- Strlcpy body correctly limits terminator space; Strlcat requires already terminated destination. strlcpy.c is absent from support CFILES, presumably system utility symbols are intended; do not claim missing symbols without link contract.
- m68k swap helper preserves d2-d4 and operates on four-word groups; no independent count overrun found.
- Remaining support ownership/concurrency helpers still pending.

### Disk-image support ownership/concurrency helpers
Coverage: implementation bodies of messages.c, ports.c, mutexes.c, semaphores.c, lists.c, hooks.c, aros/asprintf.c, aros/asprintfpooled.c, envvar.c, getcurrentdir.c, setprocwindow.c, deletelibrary.c, diskimagedevice.c and getattrs.c reviewed.
- Message/list/port constructors initialize the fields they use and return NULL on allocation failure; destruction assumes callers have drained queues/released locks, requiring device lifecycle tracing rather than a standalone helper defect.
- Mutex wraps SignalSemaphore consistently; no independent lock inversion found inside these wrappers.
- ASPrintf copies va_list for counting, allocates counted size and bounds emission. Pooled variant uses same writer; no local format-buffer overrun established.
- Hook constructor accepts arbitrary size and writes Hook fields, but reviewed callers must pass sizeof(Hook) or larger; caller inventory remains final integration work.
- OpenDiskImageDevice uses module-global request/port/base and overwrites them on repeated opens; it is a single-session application helper, not a per-unit device backend. No repeated-open caller established.
- Environment string allocation uses length returned via IoErr; GetVar reread can truncate if variable grows concurrently, but remains within buffer.
- GetCurrentDir temporarily sets current process directory to zero then restores it; process-local use is expected.
- SetProcWindow assumes FindTask is a Process. Locate non-process callers before claiming invalid process-field access.
- GUI tag/attribute varargs handling and locale/tooltype helpers are peripheral; remaining support bodies still need coverage before inventory closure.

### Disk-image SCSI dispatcher and plugin loader
Coverage: device/scsicmd.c, device/plugins.c, device/plugin_vectors.c and include/bytepack.h read completely. Support tooltypes.c, localeinfo.c and translatefuncs.c implementation bodies also reviewed; no independent disk correctness candidate established in their ordinary caller contracts.

### Disk-image unit lifecycle and control vectors
Coverage: device/unit.c, main_vectors.c, init_aros.c, init.c and io.c read completely; support message/list destruction contracts rechecked.

### Disk-image preference persistence, API configuration and remaining support
Coverage: prefs/prefs.c, prefs/readprefs.c, prefs/writeprefs.c and prefs/mmakefile.src read completely; device/password.c, progress.c, locale.c, device build/configuration, three device entry stubs and support/amigaos assembly implementations read completely. Worktree remains clean at reviewed commit 282a0454356dc2b9554ab023997d7f7ca24fc03e.

### SD/MMC renewed coverage: request layer and registration
Coverage: rom/devs/sdcard/sdcard_device.c, sdcard_ioops.c, sdcard_init.c, sdcard_sdscunit.c, sdcard_mmcunit.c, timer.c, bootwait.c, sdcard_unit.h, sdcard_bus.h, common/platform build definitions and sdcard.conf read completely. sdcard_bus.c lines 1–690 and 1670–1801 read; remaining ADMA/command/completion/IRQ middle still pending. Platform unit allocation sites traced; full platform source review still pending.

### SD/MMC transport and platform backend completion
Coverage: remainder of sdcard_bus.c read completely; all platform C files and internal headers in arch/arm-native/soc/broadcom/2708/sdcard read completely. Combined with preceding request/header/build pass, all implementation files in these two SD directories have source-review outcomes. Hardware register contracts, DMA resource/cache implementation and final public header cross-check remain integration dependencies.

### SD integration follow-up
Additional dependency coverage: ARM-native kernel/virtualtophysical.c and exec/cachecleare.c read completely; Broadcom dma.resource DMAAllocChannel/DMAFreeChannel implementations read to establish SDHOST allocation/reset contracts. Resource is shared beyond disks; other operations are not claimed fully reviewed in this slice.

### ROM USB mass-storage device, binding and geometry pass
Coverage: rom/usb/classes/massstorage/dev.c, partitions.c, massstorage.h, mmakefile.src and massstorage.conf read completely. massstorage.class.c lines 1–2645 read completely; remaining IO translation, removable task and automount bodies still pending. Transport files were reviewed earlier; final applicability/revalidation remains.

### ROM USB block read/write and seek translation
Coverage: massstorage.class.c lines 2646–3172 read completely (emulated/native read/write and seek).

### ROM USB direct SCSI and removable task
Coverage: massstorage.class.c lines3173–4442 read completely (direct dispatcher, translation/filtering, transfer lock, configuration store, removable task allocation/cleanup, DOS opening/unmount/tunneling and write-protect query).

### USB ROM automount and settings coverage continuation

Reviewed `rom/usb/classes/massstorage/massstorage.class.c` lines 4443–6433, completing the main class implementation, plus `massstorage.class.h`, `dev.h`, `common.h`, `debug.c`, and `debug.h`. This includes old RDB helpers, DOS device matching, FAT superfloppy and ISO mounting, all settings UI construction/event handling, transfer-size detection, and cleanup. Transport bodies and cross-library contracts still require final coverage reconciliation.

### USB transport revalidation and producer/consumer contracts

Read all of `massstorage_cbi.c`, `massstorage_bulk.c`, and `massstorage_uas.c` contiguously, including reset/clear, autosense, retry and stream fallback paths. Existing DISK-033–036 are reaffirmed. Traced Poseidon allocation pool initialization, psdAllocVec, endpoint/interface freeing, and psdGetPipeActual. The pool uses MEMF_CLEAR: reject the MODE_SELECT reserved-byte-uninitialized suspicion. Endpoint/interface cleanup actually frees endpoints, strengthening DISK-206's stale-pointer lifetime impact.

### Workbench USB remaining files and partition consumer contracts

Reviewed `workbench/devs/USB/classes/MassStorage/storage_init.c`, `storage_volumes.c`, `storage_stubs.c`, `storage.h`, `include/mstorage.h`, `mmakefile.src`, and `mstorage.conf` completely. Together with the earlier storageclass/storage_task/storage_device reads, all functional source files in this component now have review outcomes. The build definition explicitly includes storage_volumes. Re-read ROM partitions.c and traced GetPartitionAttrs and MakeDosNode. MakeDosNode copies strings/environment into its own allocation, so callers may release the parameter packet after construction. PT_NAME is clipped to 31+NUL: reject the suspected GPT-name overflow into the ROM 32-byte name buffer. GPT advertises PT_NAME, so the 'unnamed GPT treated as MBR ID byte' suspicion is rejected for the current parser.

### Core boot partition integration

Read `rom/dosboot/bootscan.c` and its build definition completely. Traced nested-table restrictions in MBR/GPT/RDB/EBR parser checks and readPartitionData/getStartBlock contract. Bootstrap/menu/UI are not claimed reviewed by this entry; disk command consumers there remain inventory follow-up.

### Boot disk-command consumer continuation

Read all of `rom/dosboot/bootstrap.c`, `arch/m68k-all/dosboot/bootcode.c`, and `rom/dosboot/cleanup.c`. Read dosboot_init.c initialization through line255 and menu.c disk-presence command slice285–345; these partial reads do not claim complete GUI/menu coverage. Boot block I/O, device-presence probe ownership, m68k boot-call ABI and returned init-code cleanup were traced.

### Boot initialization and AHCI helper coverage reconciliation

Completed remaining `rom/dosboot/dosboot_init.c` lines255–end and read `dosboot_intern.h`. Device enable filtering handles a failed enable-array allocation by retaining nodes; successful storage is freed after use. Retry dispatch preserves bootblock request cleanup. General screen/animation/menu presentation is outside disk-command coverage; the disk-presence slice was separately reviewed.

Read `rom/devs/ahci/subr_prf.c` completely and traced every in-component formatter caller. Re-read `mmakefile.src`, `ahci.conf`, and read `timer.h` and `ahci_scsi.h` completely. Build list includes subr_prf, all AHCI transport/adapter files and three HIDD classes; configuration is noexpunge.

Formatter review outcomes:
- Active in-component formatting calls use bounded ksnprintf for port/target/revision names, and kvcprintf callback output for logging. snprintf_func reserves one byte for terminator and does not write a size-zero buffer; no overflow established in these callers.
- Unbounded ksprintf/kvsprintf and allocation-return-unchecked kvasnrprintf have no callers in the AHCI component. They are dormant exported/internal helper observations, not active disk-path findings.
- ksprintn's digit conversion masks values to four bits although radix accepts2–36; radices above16 produce wrong characters, but current AHCI callers use decimal/hex. Signed minimum negation is undefined in the formatter for INTMAX_MIN; no current caller supplies such a value. Record these as helper limitations rather than inflate the disk candidate list.
- scsi_sense_data's full size includes14 extra bytes beyond its18-byte minimum. This confirms DISK-120's distinction between the18-byte request allocation and full-structure clearing. Byte-array CDB address/length fields have no inherent alignment requirement; cast-based accesses in implementations still require separate target classification.
- Remaining AHCI header inventory reconciliation includes ahci.h, ahci_aros.h, ahci_intern.h, pmreg.h, pci_ids.h, and exported hidd/atascsi headers; this entry does not claim those unread portions complete.

### AHCI header completion

Read completely: `ahci_intern.h`, `ahci_aros.h`, `ahci.h` (704 lines), `pmreg.h`, `pci_ids.h`, exported `include/hidd/ahci.h`, and `include/devices/atascsi.h`. Together with the previous header/helper and functional-source entries, every source/header/config/build file in rom/devs/ahci now has a review outcome. Generic hardware/PCI/cache providers are traced only as needed and are not claimed comprehensively reviewed by this component entry.

Header/adapter outcomes:
- bus_addr_t and bus_size_t are IPTR, so the mapping defect in DISK-014 is not an inherent32-bit typedef truncation. bus_dmamap_t is APTR; hardware address fields separately split low/high words.
- DMA pre/post direction constants encode write-to-device as DMA_ReadFromRAM and post completion with bit31. Cache pairing/mapping implementation remains the evidence for DISK-014; constants alone do not prove a transfer bug.
- pci_write_config width1 assigns cw.val instead of cb.val, leaving the submitted byte message's value uninitialized. All current AHCI callers use width2 or4 (Intel0x92, PCI command, ATI workaround). Record dormant helper defect; no active AHCI discovery corruption claimed.
- bus_space_barrier is an empty adapter and MMIO access uses raw native-endian volatile32-bit loads/stores. This is an architecture-sensitive contract limitation on weakly ordered/big-endian targets, not a universal x86 runtime failure. Final build/target classification must accompany any promoted barrier/endian candidate.
- AHCI command header/table/PRDT and ATA identify/FIS structures are packed. Header/table layout definitions support the existing PRDT, sector-size, residual and wire-access findings; no new layout mismatch established merely from these declarations.
- kthread_create returns ENOMEM when NewCreateTask fails; callers' ignored/failure handling remains DISK-112/113 rather than a defect in this helper.
- TAILQ operations leave removed node links unchanged, as ordinary BSD-style list macros do; caller double removal/concurrent lifetime findings must establish actual misuse, not infer invalidity from retained links alone.
- Slot count/port count macros decode advertised counts plus1; port array supports32 and PM array16. No header-level bounds inconsistency identified. Error CCB reserved slot1 and queue-depth interactions remain DISK-128's implementation trace.

### USB persisted configuration contract follow-up

Traced nLoadClassConfig/nLoadBindingConfig to Poseidon psdGetCfgChunk and psdAddCfgEntry. GetCfgChunk returns a copied raw chunk; it performs no class-specific string or numeric validation. Normal GUI input bounds do not establish bounds for persisted raw configuration.

### Finding reconciliation ledger

Started structural deduplication while retaining historical IDs for traceability. DISK-022 is merged into DISK-006 and is not counted independently. DISK-084 remains independent from DISK-008: strlen overreading an unterminated wire field is a producer-validation defect, whereas fixed-width copying beyond a short allocated C string is a consumer-length defect. Their overlapping NVMe manifestation is cross-referenced rather than removing either root cause.

Current numbered entries run through DISK-270; DISK-022 is a merged historical alias of DISK-006. Numbering is therefore not a count of independent confirmed bugs. All findings remain source-derived candidates. The component index accounts for all active candidate IDs; the historical alias is linked separately below. Chronological notes contain corrections and documented limitations. Final per-component coverage reconciliation is still required.

Next reconciliation work: resolve unnumbered candidate observations and explicit pending contracts; verify final source/header/build inventory for disk-image and consolidate already completed SD/public hardware records; reconcile driver-specific startup/cleanup/flush observations; reorganize findings by component with consistent impact/confidence/configuration/source fields. Completion has not been asserted.

### Disk-image shared header reconciliation, first slice

Read `include/dos64.h`, `td64.h`, `endian.h`, `support.h`, and `include/mmakefile.src` completely. The include build copies devices/diskimage.h into public AROS headers; public API and plugin-specific/header inventory remain to reconcile.

Outcomes:
- dos64.h contains misspelled #incude under !EXEC_TYPES_H. Current source search found no direct C inclusion of this header in the diskimage tree, and normal support users include Exec types first. Record dormant standalone-header build defect rather than an active default disk driver failure.
- TD64 numeric definitions match the existing24–27 command family; no independent mismatch identified.
- Endian helpers use fixed-width types, and swap64 places swapped low/high halves correctly. Raw load/store helpers cast pointers to16/32/64-bit types, so their callers must provide alignment; no blanket claim that every parser misaligns them.
- PPC GCC byte-reversed write inline assembly specifies pointed memory as an input operand and has no memory output/clobber or volatile qualifier. The compiler is not told the operation writes memory, so compiler memory accounting/order around consumers can be incorrect; no-output asm is implicitly volatile, so lack of an explicit volatile keyword alone does not establish elimination. Active audio/image decoder callers use wle16/wbe32; this remains a PPC-specific candidate pending compiler/target macro selection rather than a claimed x86 failure.
- Macro swap16/32/64 evaluate arguments multiple times in fallback implementations. Inspected callers pass ordinary values/field reads; no side-effect expression failure established.
- support.h's function declarations were checked against the previously reviewed support bodies. Declaration-only external compatibility headers (AmigaOS OpenSSL/proto/inline sets) need applicability classification; this read does not claim their entire vendor API surface audited.

### Disk-image public API and audio header coverage

Read `include/types.h`, `include/devices/diskimage.h`, and every `include/audio/*.h` completely (AIFF, WAVE, FLAC, Vorbis, mpg123/MPEGA MP3 alternatives, WavPack). Checked declarations against the already-reviewed audio and plugin bodies.

Outcomes:
- AROS int32/uint32 aliases remain LONG/ULONG and int64/uint64 remain QUAD/UQUAD. No typedef-based assumption that LONG becomes64 bits is valid here; this supports DISK-177's size_t-versus-LONG destination distinction on64-bit builds.
- Audio stream prefix is consistent across adapters: ULONG decoded length, close callback, read callback with ULONG byte offset/length. Large decoded lengths beyond4GiB are not representable in this interface; exact truncation/overflow remains implementation-specific and is not a new header-only corruption claim.
- FLAC header explicitly declares two owned PCM pointers, corroborating DISK-176's missing PCM cleanup. MP3 alternative headers share a guard/type intentionally for mutually exclusive implementations; they are not required to coexist in one translation unit.
- AIFF/WAVE headers test an include guard but never define it, so double inclusion can redefine the stream types. Current reviewed implementation includes each once; record header robustness defect without claiming default build failure.
- Public CDTrack length/offset/sectors use UQUAD, so parser truncations into32-bit temporaries are implementation defects rather than an unavoidable public-track limit. CDDA frame/address callback remains ULONG by declared interface.
- Public DITAG definitions provide no instruction requiring ErrorStringLength to precede ErrorString. Therefore DISK-186's tag-order dependency cannot be dismissed as violating an explicit ordering rule in this public header. Retained caller pointers in DISK-184 are likewise not justified by an ownership-transfer declaration here.
- Plugin API version is fixed at8 and callback wrappers perform direct dispatch; loader null-callback checks must remain implementation evidence. Builtin/footer/user-choice flags match the previously reviewed selection code. No independent declaration/callback signature mismatch found in this slice.
- Public device header includes support.h under SUPPORT_H although support.h defines DISKIMAGE_SUPPORT_H; the latter still prevents recursive content inclusion. Reject a recursive-preprocessor overflow claim from the guard-name mismatch alone.

### Disk-image callback and ABI header reconciliation

Read `include/interfaces/diplugin.h`, `include/aros/proto/diskimage.h`, `include/aros/defines/diskimage.h`, and `device/include/diskimage_device.h` plus `progress.h` completely.

Outcomes:
- AROS public MountImage macro passes unit inD0 and filename inA0, consistent with the implementation declaration and strengthening DISK-187's separately documented configuration/generator register mismatch. Native register-insensitive ABI builds remain conditional exclusions from that mismatch.
- DiskImageUnit explicitly retains ErrorPtr/ErrorString/Password/Screen fields across control messages. No header establishes transferred ownership or lifetime extension for caller storage; existing DISK-184/186 lifecycle traces remain valid candidate evidence.
- DIPluginIFace's AROS SetDiskImageErrorA takes RAWARG, and the variadic wrapper builds IPTR argument slots before invoking it synchronously. The temporary array is used during the call; no independent escaped-array lifetime issue established here.
- Plugin callback declarations use APTR Self versus typed interface pointers across implementation prototypes. These represent the same pointer calling convention on reviewed AROS targets; do not promote a runtime ABI mismatch merely from the C pointer-type spelling.
- Generated UnitControl variadic header falls back to ULONG slots without __HAVE_IPTR_ATTR__;64-bit tag/data compatibility is therefore configuration-dependent. Trace the compiler feature macro before promotion; current callers may use UnitControlA or modern generated declarations instead.
- ProgressBar is a dummy structure onAROS, consistent with previously reviewed no-op progress implementation. Non-AROS UI bases are separate compatibility state rather than active AROS ownership.
- HeaderTestSize/FooterTestSize are UWORD in device base while plugin TestSize isULONG. Oversized external plugin declarations can narrow selection-buffer lengths; supported in-tree plugin sizes and loader checks must establish an actual trigger before adding a candidate.

### Disk-image plugin header/build reconciliation

Read all four plugin-private headers (`dmg/adc.h`, `dmg/base64.h`, `cue/cue.h`, `fdi/fdi2raw.h`) and re-read the complete plugins/mmakefile.src. Default plugin set isCCD/CISO/CPC/DAX/DMG/DMS/MDS/NRG/UIF. DAA/GI/CUE/FDI are excluded from the default aggregate but have explicit targets. XAD/XPK have explicit build targets outside the aggregate. IPF/Crypt are source-reviewed compatibility plugins without corresponding targets in this modern file.

Outcomes:
- CUETrack deliberately begins with the same fields as public CDTrack, then adds file/index state. AUDIO_STREAM prefix matches the concrete audio headers; no prefix-layout mismatch found in the declarations.
- CUE file binary length isQUAD and track lengthsUQUAD, while CUEImage total_blocks isULONG and track countersUBYTE. Large-image/track-count limitations must be tied to parsing arithmetic rather than inferred solely from field widths.
- ADC declares both source and destination extents explicitly; DISK-156/157 are implementation violations/incorrect decoding despite this bounded signature.
- Base64 cleanup API has no explicit length and expects terminated text; empty text still satisfies that contract, so DISK-158 is not dismissed as a caller violation.
- FDI loadtrack has no destination-capacity argument for mfmbuf/tracktiming. This corroborates the decoder-bound limitation in DISK-169/170; caller allocation versus output units remains the finding evidence.
- Loader HeaderTestSize/FooterTestSize narrowULONG plugin TestSize intoUWORD (plugins.c188–193). This can underallocate tests for an external plugin declaring>=65536 bytes. In-tree reviewed plugins use small format probes; record external-plugin contract limitation pending supported plugin trust expectations, not an in-tree image-triggered overflow.
- CUE's build list still names audio/... relative to plugins although source lives in sibling../audio; DISK-181 remains a source/build-definition candidate with no build executed. License/default-selection exclusions do not exempt those optional source implementations from the review, but applicability must remain explicit.

### SD/MMC public hardware header reconciliation

Read `compiler/include/hardware/mmc.h`, `compiler/include/hardware/sdhc.h`, and Broadcom `include/hardware/sdhost.h` completely, then traced DATA_END/version constant consumers. This completes these previously pending public-header reads; no hardware execution or specification verification is claimed.

Outcomes:
- SDHOST exposes separate SDHSTS command timeout, data timeout, CRC16/CRC7 and FIFO errors. SDHSTS_ERROR_MASK includes them, supporting DISK-196's distinction from the separate SDCMD_FAIL test. Their actual hardware propagation remains explicitly conditional.
- MMC response/data flags match the transport's flag-driven command construction; no new declaration-only mismatch found. MMC high-speed switch behavior remains DISK-020's control-flow finding.
- SDHCI_INT_DATA_END is defined as bit2 and used by initialization interrupt masks, FinishData and WaitCmd completion recovery. Its hardware bit assignment requires authoritative register verification before promotion; this is now a specific unresolved protocol contract, not assumed correct merely because producer/consumer share the macro.
- SDHCI_HVERS_SPEC300 is defined as1<<2 while platform capability selection compares the numeric spec field >=2 directly. The constant is not used by the inspected backend, so even a mistaken constant does not establish active version-selection failure there.
- Host ADMA32/64 selection and capability bits are represented separately from64-bit caller virtual pointers. This supports keeping DISK-195's CPU-pointer truncation separate from hardware descriptor-address limitations.
- SDHCI power macros combine unparenthesized bitwise OR terms; actual caller contexts need tracing before a precedence failure is claimed. No independent active expression failure established by this header read.

### Disk-image preference, packing and decoder header reconciliation

Read complete include/prefs.h, include/device_locale.h, include/bytepack.h, include/scsicmd.h, dms/xdms.h and prefs/mmakefile.src. Reconciled header consumers by source search. Baseline HEAD remains 282a0454356dc2b9554ab023997d7f7ca24fc03e and git status remains clean.

- PrefsObject owns a node, parent/key and a tagged value union; the build explicitly includes prefs/readprefs/writeprefs. Header declarations do not introduce an independent width mismatch; preference parser/writer implementation observations remain separate.
- Generated device locale IDs are contiguous 0–23 and the array contains all 24 entries. Arbitrary unknown IDs still require implementation/caller validation; absence of an array bounds guard alone does not show an active invalid caller.
- BytePackWrite8 stops at end, and multi-byte writers inherit that clipping. BytePackWriteText clips its requested field length before copying and replaces the final copied byte with NUL. This truncates the last character for exactly full fields such as the eight-byte inquiry vendor, but no buffer overflow follows from this helper. Standard inquiry fixed-width string padding semantics require separate protocol confirmation before treating the truncation as a candidate.
- scsicmd.h contains dormant conversion defects: FRAMES2MS(fr) expands the unrelated identifier ms; MSF2MS omits the plus operator between minute and second terms. No in-tree expansion was found, so these are unused header defects, not active disk IO candidates. Address/frame macros used by the driver remain distinct.
- xdms_data declares no compressed-input end bound. DROPBITS repeatedly reads *indata++ while replenishing the bit buffer; this corroborates DISK-161 rather than adding another root cause. Heavy decoder table extents match the array limits underlying DISK-164.

Bundled compression/XML library implementation directories and platform compatibility declaration trees must be explicitly reconciled against disk-image dependencies in the final inventory. They are not implicitly marked reviewed by this header pass.

### Disk-image library build applicability reconciliation

Read diskimage/mmakefile, device/mmakefile.src, all three bundled-library Makefile.x86-aros files, and workbench/libs/expat/mmakefile.src through its library/include targets. The aggregate modern disk-image target builds prefs, device, GUI, mount tools and plugins. It does not build the diskimage-local expat_library/bz2_library/z_library directories. Device build explicitly depends on workbench-libs-expat, which fetches Expat 2.8.2 sources into PORTSDIR and compiles that library. This is a build-file fact at the reviewed commit, not a claim about whatever library a running installation loads.

The bundled directories are not dead source: their standalone legacy i386-aros makefiles explicitly compile their parser/compressor sources together with local init/malloc/assembly stubs, and install the same expat.library/z.library/bz2.library names requested by preferences and plugins. Accordingly:

- Modern disk-image callers and dependency length/ownership contracts remain in primary review. Modern externally fetched generic codec/parser implementations are dependencies, not separate disk drivers; their absent fetched sources cannot be claimed source-reviewed here.
- Bundled compatibility-library wrappers and their ABI/allocation lifecycle need a documented source outcome for the standalone configurations. Generic compression/XML algorithms should be identified as dependency code, with no blanket bug-free assertion. Existing plugin-level input/output validation candidates remain applicable regardless of which library provides decoding.
- Active request sites are readprefs.c:65 and dmg.c:311 for Expat; ciso.c:172, dax.c:193, daa.c:253, uif.c:217 and dmg.c:412 for z.library; dmg.c:425 for bz2.library. Optional PNG GUI imagery also requests z.library; it is not a block-image backend.

This resolves the build-selection portion of the pending bundled-library inventory; wrapper reads and final declaration-tree classification remain outstanding. No sources were fetched and no builds were run.

### Bundled disk-image decoder allocator coverage

Read all of expat_library/malloc.c, bz2_library/malloc.c and z_library/malloc.c. Traced Expat default callback installation, buffer-growth error handling and parser cleanup; inspected each init.c malloc_init/malloc_exit call site.

- All three pool allocators prepend a size_t header and serialize pool operations with a semaphore. malloc_init creates a fresh pool once at library initialization; do not extend DISK-179's per-plugin repeated initialization to these single library-init sites.
- The Expat realloc failure has a concrete parser cleanup double-free trace and is now an additional manifestation of DISK-173, preserving root-cause deduplication rather than adding an independent ID. Its standalone build applicability is explicit.
- All three wrappers lack overflow checking for sizeof(size_t)+size; z/bz2 allocation request constraints require caller evidence before an image-driven underallocation is claimed. Pool payload alignment after the prefix remains ABI dependent. These are limitations, not proven active overflow findings.
- malloc_init frees its semaphore on pool failure and returns FALSE; each inspected library-init caller deletes the library and returns NULL rather than invoking malloc_exit on that failed initialization. Reject an unconditional double-free claim for this failure path. Full init/expunge/assembly ABI wrapper reads remain outstanding.

### Bundled compression library lifecycle coverage

Read bz2_library/init.c completely and z_library/init.c:1–380 (declarations, vectors, resident init and full open/close/expunge). These are standalone compatibility configurations identified above, not the modern aggregate's fetched dependency implementations.

- Both libraries create their allocation pool during initialization and delete it only after lib_OpenCnt reaches zero. Open increments the count and clears LIBF_DELEXP; Expunge while open sets LIBF_DELEXP and refuses teardown. No unconditional live-open pool deletion found.
- Both Close handlers omit delayed expunge at zero references. The later Exec/generated-library contract reconciliation establishes the compatibility lifecycle defect and explains that another explicit removal can reclaim the library.
- bz2 buffer-to-buffer wrappers expose ULONG* destination lengths and pass them as unsigned int*. For the explicit i386 standalone build both are 32-bit; no size_t-to-LONG overwrite analogous to DISK-177 is established. Pointer-wide port applicability must use its actual C type ABI, not infer from pointer size alone.
- BZ2 stream and buffer wrappers directly return decoder statuses. The local bz_internal_error calls Alert for internal invariant failures; malformed-input reachability requires an algorithm trace before treating that as a disk-triggered crash.
- z init computes the dynamic CRC table before publishing the library, avoiding an uninitialized first-use table race in this wrapper. All z/Expat init bodies and assembly stubs are covered by the subsequent completion records.

### z.library wrapper completion and delayed-expunge contract resolution

Read z_library/init.c:381–EOF, completing that file, and complete rom/exec/closelibrary.c and remlibrary.c as dependency contracts. Inspected generated Close implementation tools/genmodule/writestart.c:1507–1520 and SDI library example Close:502–520, plus bundled Expat Close:666–680.

- z wrapper functions directly return algorithm status and retain caller-supplied stream/buffer extents. No independent ignored decoder error found in these wrappers. ULONG* versus zlib uLongf* compatibility still depends on standalone target's C unsigned-long ABI; the reviewed standalone i386 build is 32-bit. No pointer-wide overwrite is inferred without a supported wide build.
- Exec CloseLibrary invokes the library Close vector; it does not inspect LIBF_DELEXP and call Expunge on the library's behalf. Both the generated implementation and SDI example perform that transition inside Close when open count reaches zero. Thus the bundled z/bz2/Expat Close omission is a source-supported delayed-expunge behavior defect: an explicit removal requested while still open is deferred but not honored on the last close. Another removal/memory-expunge attempt can still reclaim it, so do not call it a permanent unreachable leak.
- This compatibility lifecycle observation is retained as a low-impact dependency defect rather than a new disk data-integrity finding. It has a concrete contract and conditional standalone applicability, resolving the prior pending question. It does not imply that codec memory can be freed while clients retain a valid open-library reference.

Expat init/vector functions and assembly entries are covered in the subsequent completion record; generic algorithm/declaration dependency classification remains separate.

### Bundled Expat declarations and platform entry-stub coverage

Read expat_library/init.c:1–450, all three bundled-library stub_x86.s/stub_ppc.s files, and bz2/z stub_m68k.s completely. Expat wrapper declarations cover parser creation, parse/buffer operations, handlers, status, reset/stop/resume and memory callbacks. These declarations retain pointer types for parser/handler/storage arguments; no immediate pointer-width truncation is established merely from register annotations. The subsequent Expat wrapper completion covers the vector definitions and all remaining function implementations.

All three x86 entry stubs use `movl -1, %eax`, which in AT&T syntax loads memory at address -1 rather than an immediate return value. The PPC counterparts use `li 3,-1`, and m68k uses `moveq #-1,d0`, confirming intended reject-execution return value. This is a source-supported compatibility entry defect when the library binary is invoked as an executable; normal OpenLibrary resident initialization does not use that executable entry. It is retained with explicit limited applicability instead of claiming routine image parsing crashes. These are the same entry-stub pattern previously observed in other disk-image compatibility code, not three independent disk bugs. No assembly or execution was performed.

### Bundled Expat wrapper completion

Read expat_library/init.c:451–1932 (EOF), completing all 1,932 lines together with the preceding declaration pass. Reconciled vector numbers 1–69 with declarations and function definitions.

- Parser creation/free, parse/buffer, handler installation, status/reset/stop/resume and memory wrappers pass their declared arguments to the matching XML function. Parse and memory methods return underlying results; no independent wrapper-level ignored-error or swapped-argument failure found.
- LibInit publishes only after malloc_init succeeds; failed initialization deletes the library. LibExpunge refuses while references remain and deletes the pool/library only at zero references. The previously resolved delayed-expunge omission also affects this Close implementation.
- Return types for byte indexes (long) and line/column numbers (int) follow this legacy wrapper API; large-document narrowing remains a library ABI limitation rather than a demonstrated disk-block addressing defect. Struct-valued ExpatVersionInfo return matches its declaration; no concrete supported-target mismatch established.
- XML_MemRealloc delegates to the parser callback and therefore inherits DISK-173 when using the bundled default pool allocator; it does not independently free caller memory. Custom callback suites are forwarded unchanged.

All bundled expat/z/bz2 init/malloc and their listed entry stubs now have source-review outcomes. This does not assert full review of generic XML/compression algorithm internals; dependency boundaries and caller validation remain explicit in the library applicability reconciliation.

### WD33C93 and nonvolatile inventory reconciliation

Read every file under rom/hidds/scsi/wd33c93: four C files, intern header, configuration and mmake definition (453 lines total). The backend is skeletal: bus constructor requires DriverData but does not initialize hardware/instance state; only Root methods and Shutdown are exported, with no transfer implementation. Probe allocates a generic SCSI controller object for recognized A2091 product IDs, clears CDF_CONFIGME and stores its own driver reference, but never creates/attaches a WD33C93 bus or populates the probed-bus list. Document unsupported/incomplete hardware IO rather than pretend this is an operating disk backend. The `SCSI=` comparison uses length4 against a five-character prefix and disable merely logs; no functional disable policy is implemented. No working runtime is claimed even if DISK-243 were fixed.

Read arch/m68k-amiga/nvdisk/mmakefile.src: it explicitly extends workbench/libs/nonvolatile/nvdisk with akiko, establishing nonvolatile-library support rather than disk/block-device IO. Its initial inventory row is corrected accordingly; no block-driver coverage gap is concealed by its disk-like directory name.

### AHCI public hardware header completion

Read all 284 lines of compiler/include/hardware/ahci.h and traced consumers in AHCI and PCI ATA probe. Compared capability masks/register offsets with rom/devs/ahci/ahci.h.

- Public port layout has 32 ULONG fields (128 bytes); HBA fields reserve through byte0xff and place the port array at0x100. These agree with internal AHCI_PORT_SIZE/register definitions. Command header has eight ULONGs; command table prefix accounts for128 bytes before PRDT. No source-derived layout-size mismatch found. Packed structs preserve layout but do not themselves establish physical alignment/DMA mapping, already covered by DISK-014/015.
- PRDT byte-count mask is0x3fffff, consistent with the single-entry maximum underlying DISK-015. High/low address fields exist; do not confuse availability of a high word with correct physical address translation.
- CW0_PMP_MASK is0x1f while the internal implementation supports16 PM ports; no consumer of that public mask was found. Do not promote an unused constant as an active command corruption candidate. Main AHCI uses its own detailed internal hardware definitions.
- PCI ATA probe consumes public ahci_hwhba/CAP_SAM; its mapping-lifetime observation remains separate. Header masks/layout agreement does not resolve whether probe unmaps the correct CPU mapping.

This closes the previously named AHCI public hardware-header gap. No register specification or hardware validation is claimed by comparison between local definitions.

### USB reconnect pipe ownership contract resolution

Re-read binding cache selection (massstorage.class.c:376–445), complete nAllocMS protocol/pipe setup and failure cleanup, nFreeMS pipe teardown, and Poseidon psdFreePipe implementation (poseidon.library.c:4651–4681). Traced IFA_IDString construction at8426–8438.

- psdFreePipe actually frees the PsdPipe, after endpoint teardown and device reference decrement; it is not an idempotent disposal marker. Retaining pipe fields after freeing therefore remains a useful ownership warning.
- However, the previously suggested UAS-to-BOT/CBI protocol-changing rebind does not establish an in-tree stale-pipe path: cached units require identical device and interface ID strings, and interface IDs include interface protocol, subclass, class, alternate setting and (where needed) configuration number. A protocol change selects a different unit cache entry. Reject that specific hypothetical trigger rather than promoting it as a reconnect bug.
- Same-protocol successful rebind overwrites every required common/protocol pipe field before publishing ncm_Task. An interrupt endpoint that disappears while protocol/IDs remain unchanged could leave EPIntPipe stale; descriptor-change/error paths would need a concrete supported sequence before independently claiming double-free. Retain as a conditional lifecycle limitation, not a substantiated new candidate.
- This does not dismiss DISK-206: UAS endpoint collector's old endpoint objects can survive same-ID disconnect/reconnect because endpoint addresses/descriptors themselves are not included in the interface ID and the collector skips already-nonNULL fields. Existing endpoint-lifetime evidence remains the basis of that candidate.

The stale-pipe observation is now resolved into a rejected protocol-change trigger and an explicit descriptor-dependent limitation. No further unproven pipe double-free is added to the numbered list.

### Core file inventory audit and NVMe declaration completion

Enumerated implementation/header/config/mmake files under NVMe, VirtIO, PC/Amiga trackdisk and shared storage against report filename references. Filename occurrence is only a documentation locator, not proof of reading or absence of coverage. The audit identified NVMe declaration coverage needing explicit records.

Read complete nvme.conf, nvme_debug.h, nvme_hw.h, nvme_queue.h, nvme_queue_admin.h, nvme_queue_io.h, nvme_timer.h, include/hidd/nvme.h and all376 lines of nvme_intern.h. Public hardware/nvme.h remains a distinct next file.

- Configuration is noexpunge, with controller/bus/unit Root interfaces and explicit BeginIO/AbortIO. Teardown candidates still apply to failure/disposal paths; do not imply normal runtime library expunge is enabled.
- completionevent_handler includes inline and growable DMA maps; release performs CachePostDMA per recorded segment, then frees/reset map storage. This corroborates the NVMe DMA bookkeeping contract and keeps DISK-007 specifically VirtIO rather than extending it to NVMe.
- Map growth doubles capacity with no overflow guard; supported request length/page segmentation constraints must establish reachable needed counts before claiming a wrap/underallocation. The header alone is not an image-triggered overflow finding.
- au_SecCnt, low/high sector fields and hardware SQ/CQ DMA addresses are UQUAD, while byte transfer limits and per-segment lengths are ULONG. This matches the intended distinction between64-bit disk addressing and bounded individual IO, not an automatic large-disk failure.
- Queue depth/head/tail/command identifiers are UWORD and completion status volatileUWORD. Readiness versus callback ownership still depends on completion order (DISK-086); volatile does not prove lifecycle synchronization.
- Header prototypes agree for queue allocate/free and submit signatures; no standalone declaration mismatch found.

### NVMe public hardware and build-header completion

Read all310 lines of rom/devs/nvme/include/hardware/nvme.h and complete mmakefile.src. All nine private/public NVMe headers and configuration now have explicit records across this and the preceding pass. Build lists nine device translation units (init/timer/io/hw/prp/queue/admin/ioqueue/sgl), three class units and copies both hardware/HIDD include directories. It does not silently omit SGL from the module.

- Command union members account for64 bytes using fixed-width Exec types; completion layout accounts for16 bytes. Register positions have naturally aligned64-bit CAP/ASQ/ACQ and32-bit intermediate fields. No source-derived command/completion padding mismatch found on ordinary4/8-byte alignment ABIs. Whole-width MMIO emission on32-bit targets remains compiler/platform-dependent and is not proved by a natural-layout comment.
- Namespace nsze/ncap/nuse are64-bit sector counts, with per-format data-size exponent ds. This supports keeping capacity scale/endian handling (DISK-085/091) tied to consumer arithmetic rather than declaration widths. Sixteen LBA-format records are explicitly reserved; active format selection bounds need consumer checks.
- Controller sn/mn/fr are fixed20/40/8-byte arrays, not NUL-terminated strings; this directly corroborates DISK-084. Queue IDs and completion command identifiers are16-bit, matching queue bookkeeping fields.
- Header defines nvme_cmd_flush opcode; CMD_UPDATE omission is an implementation behavior (DISK-010), not absence of a protocol constant.
- CAP stride/timeout macros mask their intended fields; queue/controller maximum feature handling still needs actual consumer traces, not assumed validated by definitions.

No new independent declaration/layout candidate assigned. Remaining NVMe inventory work is final explicit source/build reconciliation and observation classification, not unread local hardware headers.

### Partition start-offset width reconciliation

Re-read all three boot/USB GetOffset implementations and their mount-cylinder adjustment call sites, partition_support.c:getStartBlock, and raw read/write offset expressions.

- All three consumers return ULONG sector offsets, and getStartBlock also returns ULONG. A later UQUAD destination in read/write does not recover high sector bits already discarded by the helper. The integration manifestations are now attached to DISK-059.
- Raw expressions multiply `(getStartBlock + StartBlock)` by DosEnvec sector-byte size before assigning UQUAD; actual expression width depends on StartBlock/geometry typedefs. DISK-060's command-capability choice remains separate from helper narrowing. Do not claim changing only the final offset variable fixes the pipeline.
- Geometry modulo-zero remains DISK-226 and no-mount flags DISK-229. These require different fixes and are not merged into address-width accounting.
- Generic nested-table cycle suspicion remains rejected by parser admissibility rules; EBR's own linked-block cycles remain DISK-062. This width trace does not reopen an unsupported generic recursion claim.

### PC/Amiga floppy header and build completion

Read complete trackdisk_device.h, trackdisk_hw.h, trackdisk.conf and mmakefile.src for both rom/devs/trackdisk and arch/m68k-amiga/devs/trackdisk. Both builds list their own device/hardware C pair and both configurations disable library expunge. These declarations/build files now have explicit coverage, supplementing the earlier full functional source passes.

- PC supports two units and a per-unit DMA/cache marker; its seven-byte result array corroborates DISK-044's stale result-byte interpretation. Fixed geometry is2880 sectors,18 sectors/side,2 sides,80 cylinders; larger formats are explicitly unsupported, not silently assumed reviewed.
- PC DMA helpers program address/page/count separately and document caller responsibility for boundaries and channel-size constraints. Channel2 uses a16-bit counter and separate page, corroborating DISK-046. General unused channel5–7 paths are utility declarations, not additional active floppy claims. Header warns flip-flop programming requires a DMA lock; whole-system DMA concurrency requires another active shared-controller user before an independent race is established.
- Amiga declares four units, one shared raw/data buffer with explicit owner unit/track and dirty flag, plus four read-only cache slots with separate unit/track markers and per-slot pointers. This confirms the shared-owner distinction in DISK-049 and partial slot allocation exposure in DISK-051.
- Unit listener lists store caller registrations rather than copied interrupt objects; lifetime must be enforced in implementation. No new declaration-only callback bug assigned.
- Buffer/sector/track fields are deliberately mixed-width; DISK-047 relies on unchecked request-to-track narrowing, not merely the existence of byte-sized unit fields. Hardware prototypes match their platform's implementations and include correct platform resource dependencies.

No independent header-layout candidate found. Final public trackdisk/disk.resource API and source inventory reconciliation remain separate.

### Public floppy API and disk-resource arbitration reconciliation

Read `compiler/include/devices/trackdisk.h` and `compiler/include/resources/disk.h` completely, and rechecked `rom/disk/getunit.c`, `giveunit.c`, `allocunit.c`, `freeunit.c`, and `disk_intern.h`. The public disk-resource layout uses a message at the start of each resource-unit record, a waiting list, and a shared flags byte containing both unit-allocation bits and the active-arbitration bit. This supports the existing DISK-080 wakeup-message and DISK-081 non-atomic allocation findings; it does not create separate findings. The public trackdisk header declares change-interrupt commands but supplies no ownership/lifetime documentation, so its declarations alone cannot establish additional handler-lifetime violations. Geometry and extended-request declarations were checked against the driver headers. No additional candidate was established in this pass.

### disk.resource build inventory closed

Re-read all eight C implementations and the internal header under `rom/disk`, `disk.conf`, both modern mmake definitions, and the Amiga `disk_intern_init.c` override. The build lists six public functions plus common initialization and selects the Amiga hardware initialization through `%build_archspecific`; the generic initializer intentionally fails, so it is not an operational portable disk resource. All files in these two resource directories now have source-review outcomes. Amiga interrupt dispatch delegates only through the current resource-unit callbacks; hardware interrupt acknowledgement is local to the sync/block wrappers. Unit indexes are used without validation in the public allocation and ID functions, but this pass found no valid in-tree caller supplying indexes outside the four-unit contract, so that caller-precondition limitation is not promoted to an independent finding. No additional root cause beyond DISK-080 and DISK-081 was established. This closes the resource inventory, not the entire disk subsystem.

### Shared storage remaining observations classified

Rechecked `storage_init.c`, `storage.conf`, `mmakefile.src`, and the AllocateID/ReleaseID implementation against ATA, AHCI and NVMe registration callers. The build lists exactly storage_init, storage_controllerclass, storage_busclass, storage_unitclass and storage_class; the exported header is copied from include/storage.h. Previously recorded complete reads cover these files.

- **Startup resource leak (source-supported, low severity):** `rom/hidds/base/storage/storage_init.c:29–65` opens utility.library and creates the hardware-root object before pool creation and driver registration. A failed pool creation returns FALSE without closing the library or releasing the root reference; a failed HW_AddDriver also leaves the created pool. `Storage_Expunge` performs no cleanup. This is a failed-initialization leak, not a demonstrated disk-data corruption or ordinary successful-module-unload defect: storage.conf explicitly selects noexpunge. The manual acquisitions have no corresponding unwind in this implementation.
- **ReleaseID limitation (source-supported, low severity; caller-dependent):** `storage_class.c:180–183` ignores the supplied IDHandle and neither removes its list node nor frees its name/allocation. Calling the exposed ReleaseID method therefore cannot reclaim an ID. Searches of the current ATA/AHCI/NVMe consumers found AllocateID calls but no matching ReleaseID calls, so normal in-tree registration does not establish repeated release/reallocate leakage independently of those callers' own lifetime handling. Persistent boot-device allocations can be intentional. Preserve this as an unimplemented advertised operation, not a proven hotplug failure.

These observations resolve the former pending classification notes. DISK-001, the allocation-failure finding and DISK-106 remain the independently recorded namespace correctness candidates; the resource observations above do not duplicate their root causes.

### VirtIO build and dynamic-configuration reconciliation

Rechecked the eight C files listed by `rom/devs/virtio/mmakefile.src` against the directory inventory: init, timer, io, hw, queue, controllerclass, busclass and unitclass. All implementation, private/public header, configuration and build files have earlier complete-read outcomes. Modern boot mmake dependency lists reference kernel-virtio in i386-pc, x86_64-pc and riscv64-opensbi; this establishes build selection paths, not successful builds or working PCI mappings on each architecture. The previously pending dynamic-configuration observation is now DISK-251. Final candidate normalization remains open.

### NVMe final local file/build reconciliation

The directory contains twelve C implementations: nvme_init, nvme_timer, nvme_io, nvme_hw, nvme_prp, nvme_queue, nvme_queue_admin, nvme_queue_io, nvme_sgl, nvme_controllerclass, nvme_busclass and nvme_unitclass. The mmake device/class lists include all twelve. Earlier consecutive source passes and the explicit nine-header reconciliation cover the complete local implementation inventory, configuration and build definition. Re-read nvme_sgl.c completely in this pass. Modern boot dependency definitions reference kernel-nvme in i386-pc, x86_64-pc, aarch64-raspi and riscv64-opensbi; their presence does not establish working hardware on each platform or big-endian deployment.

Remaining unnumbered observations are retained with bounded claims: queue/list alignment for controllers selecting pages larger than the allocator's documented 4096-byte guarantee is a conditional compatibility concern, not an unconditional defect on 4 KiB-page x86 controllers. Queue and Identify cache coherency depends on the PCI allocation/mapping backend; explicit transfer-data DMA bookkeeping alone does not prove those other allocations coherent. Startup failure cleanup leaks and geometry initialization gated by a successful expansion.library open are source-supported conditional failure-path observations. These do not establish ordinary successful-boot failures when the library and platform contracts hold. No per-I/O timeout or cancellation implementation is supplied after submission; this remains a stated recovery limitation. Existing numbered timeout/teardown/completion candidates identify the stronger concrete ownership violations.

### Finding-index integrity reconciliation

Checked the current report's 270 numbered headings and 270 explicit anchors against its component index. There are 269 non-alias candidate IDs, each represented in that index, with no duplicate numbered headings or missing anchors. [DISK-022](#disk-022) remains solely as a historical alias merged into DISK-006; it is not a second candidate. These counts describe report entries, not independent confirmed defects. All indexed titles agree with their corresponding active finding headings.

Revalidated the current worktree baseline as `282a0454356dc2b9554ab023997d7f7ca24fc03e`; git status is clean. This is artifact/reference verification only, not a build, test or execution of project code. Finding completeness and whole-subsystem coverage remain separate unfinished gates.

### ATA native-sector observation reconciled

Rechecked identification (`rom/devs/ata/lowlevel.c:1766–1799`), request conversion (`ata.c:65–95`) and block transfer construction (`lowlevel.c:1213–1221`). Non-ATAPI disks unconditionally use au_SectorShift=9; the logical-sector Identify fields are converted but do not select the shift. Both the byte-to-LBA conversion and the transfer byte count use this same assumed shift. Thus an attached disk whose actual logical sectors exceed 512 bytes has no supported size adaptation here: a byte request is translated into counts of 512-byte units while the disk command operates in its own logical units. The source establishes this compatibility/correctness limitation, but successful DMA overruns or a particular controller's response must not be asserted without its transfer-length/error contract. A 512e disk exposing 512-byte logical sectors does not trigger this observation merely because its physical sectors are larger. This resolves the previous unspecified native-sector assumption; retain it as a conditional observation distinct from AHCI's reversed-word defect (DISK-116).

### SCSI local inventory/build reconciliation closed

Compared current directories with both mmake source lists. Generic scsi.device includes seven device units (scsi_init, scsi, lowlevel, timer, bootwait, waitnano, waitto) and three class units (scsi_controllerclass, scsi_busclass, scsi_unitclass). These are exactly the ten local C implementations. The six headers are scsi.h, scsi_bus.h, timer.h and the three exported include headers. Earlier full-source completion records explicitly close the once-pending scsi_busclass chunks and full configuration/header reads; those earlier pending notes describe historical progress, not an outstanding unread portion.

The WD33C93 build lists all four local C files, alongside its internal header, configuration and build file. Its complete-read outcome remains an unfinished optional backend, with the raw diff-header compilation candidate DISK-243 and no implemented transfer engine. Generic SCSI's separate case-sensitive include-name candidate remains DISK-138. A build-definition reference proves inventory inclusion only; neither module was built or run. No additional root cause was established by this reconciliation.

### SD/MMC source/build inventory reconciliation

Compared the generic mmake list with all eight C implementations in rom/devs/sdcard (bootwait, init, device, bus, mmcunit, sdscunit, ioops and timer). Compared Broadcom's platform definition with all five platform C implementations (bcm2708time, bcm2708init, bcm2708bus, sdhost_init and sdhost_bus) and its two internal headers. Earlier complete source passes plus subsequent public SDHCI header reconciliation cover these implementations and declarations. The raspi-arm and raspi-armeb selections compile SDHOST plus the common time helper; raspi-aarch64 includes both SDHOST and SDHCI, selected at runtime. Consequently the earlier hypothetical big-endian SDHCI PIO byte-order path is not established by the current armeb build selection.

DMA assumptions remain explicitly limited: direct control-block bus aliasing can fail if its allocation lies outside the platform's identity-translated range, but low-memory allocations do not demonstrate that failure. Contiguity and addressability must follow the active platform allocator/cache contracts; allocation flags alone do not prove a DMA-visible physical address. Existing DISK-195 covers the independent narrowed data pointer, and DISK-193 covers software timeout ownership. The exactly-2^32-sector Read32 boundary has been promoted separately to DISK-249; it is no longer merely a pending capacity observation. Startup cleanup leaks remain conditional failed-initialization resource defects under noexpunge. No additional root cause was established by this inventory pass.

### AHCI inventory and mapping observations reconciled

The AHCI build explicitly lists all fifteen local C implementations: twelve device/support units and three class units. Complete recorded source passes cover these plus nine local/exported headers, configuration and build; the separate compiler hardware/AHCI header was subsequently read completely. Modern boot dependency references occur in i386-pc and x86_64-pc. The empty bus_space_barrier and native-endian MMIO adapters remain conditional architecture limitations; these two demonstrated build selections do not establish a weakly ordered or big-endian deployment failure.

**ATA PCI probe mapping cleanup candidate:** `rom/hidds/ata_pci/probe.c:220,247,292,319` retains the mapped CPU address in hwhba, but the CAP_SAM rejection and timer-open-failure exits pass the original hba_phys BAR address to UnmapPCI. The successful path instead passes hwhba. The public implementation contract (`rom/hidds/pci/pcidriverclass.c:180–185`) describes unmapping the area returned by MapPCI. On a backend with nonidentity mappings and meaningful unmapping, those error exits fail to release the acquired mapping and may request an invalid unmap. Confidence is high in the argument mismatch, conditional in backend impact; severity is low for a mapping leak, potentially greater if a backend acts on the unrelated address. Generic UnmapPCI is a no-op, so no generic x86 corruption is claimed. This resolves the previously pending probe-lifetime observation without bundling it with operational AHCI transfer DMA defects.

### ROM USB suspend/core contract reconciliation

Traced class suspend/resume dispatch (`massstorage.class.c:868–878`) and all ncm_Running uses into Poseidon `psdSuspendBindings`/`psdSuspendDevice` (`poseidon.library.c:3702–3840`) and transfer submission (`psdDoPipe`/`psdSendPipe`, 4705–4765). The class disables removable-media polling while suspended and re-enables it on resume; regular disk requests continue through the worker. That alone does not prove requests are submitted to a sleeping device: both core pipe submission APIs check PDFF_SUSPENDED and call psdResumeDevice before queueing the request. Reject an unconditional “ordinary I/O never resumes” candidate.

A narrower quiescence concern remains conditional: a suspend-capable class is trusted when its callback returns TRUE, whereas the core's IOBusyCount veto is in the unsupported-class branch. The mass-storage callback does not wait for active transfers or serialize against a new command before returning TRUE. Establishing an actual in-flight transfer disruption requires the hub's suspend transition and worker/core locking sequence; these slices alone do not prove that sequence. Preserve this as an explicit unresolved timing limitation, not an additional established corruption finding. These are dependency contract slices, not a claim to have reviewed the entire Poseidon core.

### ROM USB suspend timing follow-up

Traced the hub method dispatch and actual downstream-port transition in `rom/usb/classes/hub/hub.class.c:334–376,1479–1517`. Hub suspend requests can be handled through the hub task and issue SET_FEATURE(PORT_SUSPEND) directly; this function does not inspect the child's IOBusyCount or drain its outstanding transfers. It sets DA_IsSuspended only after the control request succeeds. Together with the mass-storage class's unconditional successful suspend callback, this establishes a source-supported **missing quiescence candidate** when an explicit suspend overlaps active mass-storage I/O. The child worker is not stopped or locked by changing ncm_Running, and the hub/core slices do not add a compensating drain.

Confidence is high for the missing drain/transaction gate, conditional for hardware disruption; severity medium for interrupted commands or timeouts. Automatic idle suspend ordinarily starts after inactivity, so do not present every idle suspend as defective. Automatic resume on subsequent pipe submission avoids the previously rejected permanent-sleep claim, but cannot protect an already submitted transfer from this transition. An actual controller's handling of the overlap and any ensuing transport recovery remain unexecuted. This supersedes the earlier unresolved hub-sequence note: the source path is now traced and the timing trigger is explicitly bounded.

### ROM USB local build inventory and UAS claim boundaries

Compared all fourteen files under rom/usb/classes/massstorage with the mmake list. Its seven C units (class, bulk, CBI, UAS, dev, debug, partitions), five headers, configuration and build definition are all accounted for by recorded complete reads. Re-read the complete UAS transport in this pass. The current source does not validate returned status tags or reject unknown IU identifiers; DISK-035's source-derived completion-validation claim stands without assuming a particular external wire layout.

The little-endian UAS tag/sense-length conversions and local IU structure layouts require an authoritative protocol comparison before assigning a separate wire-format defect; local declarations alone are not such proof. No additional endian/layout candidate is claimed here. Likewise, data-before-status ordering and missing READ_READY/WRITE_READY handling are retained as protocol interoperability concerns whose stream/nonstream applicability must be established before asserting universal failure. Short sense-response extent validation in bulk/CBI is grouped with DISK-034 rather than counted again. Reconnect endpoint ownership was resolved in the dedicated cache-key/FreePipe pass; suspend quiescence now has its traced conditional observation.

### UAS ready-handshake applicability resolved

Compared AROS nUasDoCommand with [Linux v6.12's UAS implementation](https://raw.githubusercontent.com/torvalds/linux/v6.12/drivers/usb/storage/uas.c), queuecommand and uas_stat_cmplt. The independent driver initially withholds data submission when streams are disabled, then submits the corresponding data transfer after READ_READY/WRITE_READY and continues status processing. AROS instead synchronously completes its data operation before requesting its single status IU and has no ready-IU dispatch. This establishes a conditional nonstream interoperability candidate: a device waiting for the ready handshake can leave AROS waiting on data while the ready IU remains unread. It is not a universal stream-mode failure. The missing ready-state handling extends DISK-035's completion state-machine observation; DISK-252 separately covers the malformed wire layout, which can prevent reaching this later phase. Hardware timeout behavior is not reproduced. This closes the prior stream/nonstream applicability question on independent primary-source evidence.

### Disk-image callback caller reconciliation

Traced disk-change callbacks (`device/unit.c:615–621`) and reload callbacks (`device/main_vectors.c:231–241`) into both in-tree GUI registrations. Zune's SignalFunc (`zune_gui/main.c:425–429`) and Reaction's SignalTaskFunc (`ra_gui/main.c:442–447`) only signal their owning task. They do not synchronously invoke device control APIs from the callback, so no ordinary in-tree callback deadlock is established. Callbacks still execute while the device holds its list/plugin semaphores; custom callbacks that synchronously wait on another task using those APIs can introduce lock inversion. Preserve this as a caller-dependent API limitation, not an unconditional finding, and do not assume recursive same-task semaphore acquisition itself proves a deadlock.

An incidental integration candidate was found in the Reaction GUI registration: `ra_gui/main.c:161–165` allocates ReloadPluginsHook but checks DiskChangeHook, which is already non-NULL. If the new allocation fails, the next statement dereferences NULL. Confidence high; medium severity for GUI startup crash under allocation failure. This is a callback-registration client defect, not a disk-data or device-core corruption claim. General GUI presentation remains outside the primary review.

### Hosted-disk inventory/build reconciliation closed

Rechecked all five hosted build definitions against the full current hostdisk inventory. The shared module compiles hostdisk_device, hostdisk_host and automount; UNIX replaces the host interface and adds geometry, Windows replaces the host interface, and Linux/Darwin select their geometry overrides. Prior complete reads cover all eight C implementations, three host-interface headers, shared device header, configuration, documentation and five build definitions. The Darwin selection deliberately uses `arch=darwin-` and documents omission of geometry on iOS. Host ABI and dynamically loaded host symbols remain unexecuted.

The Linux file's dependency directive incorrectly names workbench-darwin-$(CPU), while its actual `%build_archspecific` selection names linux. This is a concrete build-definition inconsistency with conditional scheduling impact, not proof that Linux geometry is absent from every build. Preserve that low-severity candidate with the exact file `arch/all-linux/hostdisk/mmakefile.src:3`. Existing cleanup/quick-reply/host-short-transfer findings retain their platform conditions; this inventory pass does not establish additional transfer root causes.

### RAM/file-backed module inventory closed

Read ramdrive.conf and fdsk.conf completely and rechecked both GCC headers and the module entries in workbench/devs/mmakefile.src. The entries compile exactly ramdrive_device.c and fdsk_device.c respectively; neither configuration selects noexpunge. BeginIO/AbortIO mappings agree with the reviewed entry points, and ramdrive exposes killrad0/killrad in its extra function list. All six device-specific implementation/header/configuration files now have recorded source outcomes. No additional declaration/build candidate was established. Existing node-ownership, quick-reply, bounds, zero-progress write, geometry/protection and flush observations retain the recorded source traces. Extended-command change-count semantics remain a separate API-contract observation, not an unread configuration file.

### Workbench USB build inventory reconciliation

Rechecked both mmake files against the eleven-file directory inventory. The HIDD module compiles storage_init, storageclass, storage_task, storage_device and storage_volumes; storage_stubs is associated with `%make_hidd_stubs` rather than the five module translation units. Prior complete reads cover all six C files, both headers, mstorage.conf and the main build definition; the include build definition was read here and copies mstorage.h into the usb include directory. No missing implementation file was found.

Rechecked Storage_Expunge: it releases attribute bases and deletes the shared pool, with no explicit worker shutdown in that routine. This alone does not establish normal expunge while live class instances/worker tasks remain: such a lifecycle candidate depends on generated class ownership/open counts and how the old USB HIDD stack removes bindings. Retain this as a bounded teardown-contract limitation, not a demonstrated ordinary disconnect use-after-free. DISK-030 already supplies the stronger concrete queued-request disconnect outcome. This component's local source/build inventory is closed; final candidate normalization remains separate.

### ATA core/PCI and legacy-platform build reconciliation

Rechecked all five local mmake definitions and Efika make.opts against the C inventory. Core ATA's thirteen translation units and PCI ATA's six are all explicitly listed by their modern builds and covered by earlier complete reads. The Amiga device override lists only waitnano; ata_amiga, dma_generic and its legacy bus_pci are not implicitly activated by residing in that directory. Active Gayle/FastATA HIDD coverage remains the separate platform component recorded earlier. Sam440's bus_ppc460 build invocation is commented out. Efika explicitly lists its two legacy units, but their obsolete interfaces remain a source/build compatibility defect, not evidence of a functioning current backend.

Thus runtime candidates from inactive Amiga/Sam440 code retain their legacy applicability; their lack of active build wiring is now explicitly resolved rather than pending. Core native-logical-sector behavior has its separate bounded reconciliation. This closes these build-list comparisons; final active Gayle file inventory and consolidated candidate formatting remain separate gates.

### Active Gayle/FastATA inventory closed

Rechecked all nine files in arch/m68k-amiga/hidd/gayle_ata. The mmake definition includes exactly its five C files (class_init, probe, gayleata_busclass, fastata_busclass, interface_pio); the two private headers and ata_gayle.conf complete the inventoried implementation/configuration set. Earlier complete reads cover all nine. Amiga boot references kernel-hidd-gayleata-kobj. The module selects noexpunge, and ENABLE_ATAPOWERFLYER remains commented out with no enabling module flag. Therefore FastATA's stub IRQ handling is an optional unfinished-backend limitation; compiling its class file does not prove default probe reachability. DISK-099's method-base mismatch and the existing PIO/failure-cleanup findings remain source-supported with their recorded applicability. No new root cause was established by this final active-platform inventory comparison.

### Floppy module inventory closed

Compared both trackdisk directories with their mmake source lists: each contains precisely two C implementations, two headers, a configuration and a build definition. Both device/hardware source pairs have complete-read records; the later declaration/configuration pass and public trackdisk/disk-resource pass complete those inventories. PC and Amiga are distinct module selections with separate hardware implementations, not interchangeable backend files. No additional missing-file or source-list mismatch was established in this pass. Together with the closed disk.resource and RAM/file-backed inventories, the combined component table now reflects the completed local reads while retaining ETD/caller-contract limitations separately.

### Partition-library local build inventory reconciled

Rechecked rom/partition/mmakefile.src, the exported-header copy definition, the Amiga platform override and legacy Makefile.gcc. The modern module explicitly includes partition_init, twenty-two public-function units and nine parser/support units. The only additional local C initialization unit, partitionami_init, belongs to the separately reviewed legacy Makefile.gcc path. The Amiga modern override replaces partition_platform; it does not select that legacy initializer. All local source/header/configuration files have earlier complete-read outcomes, including the platform override and legacy macro/initialization limitations. This closes the library's build-list comparison without treating the legacy parser subset as the modern feature set.

Integration coverage remains distinct: boot scanning and both USB consumers have traced offset, automount, geometry and parameter-packet contracts; general filesystem implementations/UI are excluded. Driver boot-packet omissions are grouped under DISK-089 with their per-driver reachability, rather than assigned fresh IDs here. Final bootblock request-size and metadata integration reconciliation remain open.

### Bootblock size/accounting observation reconciled

Rechecked `rom/dosboot/bootstrap.c:84–140,203–224`. Multiplication into ULONG is unchecked, but the computed size is a multiple of four even after wrapping. Therefore the earlier suggestion of a partial-word checksum read beyond the allocation is not established by this formula and is withdrawn. Sizes of four or eight bytes still fail to supply the twelve-byte executable bootblock prefix; however, reaching buffer+12 additionally requires a passing checksum and matching FileSystem.resource DosType. Preserve the minimum-size/overflow issue with those conditions rather than assert unconditional header overreads. Large-size checksum nontermination remains the independently concrete DISK-230.

The short-success observation remains source-supported: MEMF_CHIP allocation is not cleared and io_Actual is never compared with the requested size before checksumming. A driver returning success with fewer bytes can expose untouched allocation contents to this parser; boot execution still requires checksum and recognized DosType. The physical-offset-zero convention is retained as an unverified partition-boot applicability limitation, not promoted to a wrong-partition defect without its boot ABI convention. These classifications close the size/accounting pending note while explicitly preserving the convention limit.

### Report consistency correction

Updated the original bootblock size observation to remove its superseded partial-word-overread claim, advanced the numbering ledger through DISK-252, and aligned the coverage table with completed USB, callback and partition/boot reconciliation. Outstanding disk-image inventory and candidate normalization are still explicitly retained.

### Disk-image modern build selection reconciliation

Re-read the aggregate mmake file and complete device/plugin/support definitions. The device compiles its AROS initialization and eleven other device units, plus built-in generic/ADF/D64/ISO backends. External default plugin dependencies select CCD, CISO, CPC, DAX, DMG, DMS, MDS, NRG and UIF. DAA/GI/CUE/FDI have explicit build rules but their normal aggregate dependency entries are disabled; XAD/XPK also have explicit rules without inclusion in that default aggregate. Findings in those optional backends retain optional-build applicability rather than being presented as automatically loaded by every diskimage installation.

CUE's optional modern rule lists its codec adapters and compatibility wrappers; DMG lists base64/ADC, DMS lists all ten bundled decoder units, and FDI lists fdi2raw. These are covered by prior consecutive source passes. The support definition lists the AROS formatting wrappers and shared file/path/message/semaphore helpers; their earlier complete reads remain distinct from declaration-only compatibility include trees. Modern codec/compression library dependencies are external/shared libraries, not implicitly the bundled legacy z/bz2/Expat library directories. Full compatibility declaration-tree and remaining standalone build comparison are still open.

### Disk-image standalone selection comparison

Read the source/object selection portions (lines 1–70) of Makefile.x86-aros and Makefile.ppc-aros, and inspected the m68k standalone Makefile's object and target lists. These standalone AROS builds use device/init.c and architecture entry stubs, unlike modern mmake's init_aros.c. Both explicitly build the bundled z/bz2/Expat libraries and include optional CUE/DAA/GI/FDI plugins in their all target. The m68k target additionally selects IPF/XAD/XPK/Crypt and MPEGA audio. Therefore legacy wrapper findings have concrete standalone build selections even when absent from the modern default aggregate. This is a selection comparison, not a claim that all later standalone recipes were read in this pass; the truncated initial m68k output is not counted as a new full-file read. Existing earlier source/build records cover the implementations themselves.

### Standalone AROS build recipes completed

Read Makefile.x86-aros lines 71–end and the complete diff against Makefile.ppc-aros. Together with both earlier first-70-line reads, this covers both standalone AROS build files fully: their only differences are toolchain/metadata target, entry stub, release directory and recursive architecture-specific library Makefile selection. Read the m68k Makefile lines 71–end separately, avoiding reliance on the earlier truncated output. This establishes its remaining link/dist/clean/revision behavior, but does not by itself upgrade its earlier truncated prefix to a complete consecutive read.

Both standalone AROS builds link the device with libprefs, whereas their Zune GUI target lists only libsupport; whether unused/required prefs functions are referenced by that GUI must be traced before claiming a link defect. No such defect is promoted from object-list comparison alone. Release copying and cleanup recipes were inspected as source; none was executed.

### Standalone build prefix and GUI-link observation closed

Read the m68k Makefile lines 1–70 without truncation; together with the preceding remainder read, the full file now has explicit coverage. The previously noted absence of libprefs from standalone Zune GUI linking is not an established defect: inspected prefs.h's exported allocation/dictionary/read/write API names and searched all Zune GUI C sources, finding no preference-library calls or header inclusion (only a UI string named Prefs). Reject the missing-prefs-link suspicion for the current source. This closes the outstanding standalone recipe-read and object-list-only observation; no linker was invoked.

### Disk-image compatibility declaration-tree inventory started

Enumerated all 179 files beneath workbench/devs/diskimage/include: 123 AmigaOS compatibility files, sixteen AROS files, eight revision files, seven audio headers, four library headers, two device headers and nineteen other files. This is inventory evidence, not proof of complete content review. A search for inline/assembly markers identifies bytepack.h, endian.h, SDI_compiler.h, aros/ctype.h and two additional files whose matches may be declarations/comments. Endian/bytepack already have explicit complete reads; SDI/compiler and other executable compatibility macros require final classification.

Read aros/ctype.h completely. Its isdigit wrapper has a simple bounded comparison; its isspace wrapper recognizes only space/tab/newline/carriage return, omitting vertical tab/form feed. This is a locale-independent subset used by legacy compatibility paths, not proof of an ordinary image-parser failure without a caller/input trace. Preserve the behavior as a conditional parsing limitation. MPEGA's assembly-marker match occurs in callback documentation in the inspected prefix, demonstrating why keyword hits cannot alone classify a file as executable inline code. Remaining header content coverage is still explicit work.

### SDI compiler compatibility header completed

Read all 241 lines of include/SDI_compiler.h. This file supplies compiler-specific calling-convention/register/attribute macros and fallback type/offset definitions; its assembly marker is a declaration modifier, not an inline assembly routine performing transfers. AROS GNU builds select their own stack-parameter convention and do not apply the non-AROS ULONG/IPTR fallback. Therefore this header does not independently establish pointer truncation on 64-bit AROS. Actual callback ABI correctness depends on the companion SDI hook/library/interrupt macros and active callers; no new candidate is assigned merely from fallback syntax for an unselected compiler. The declaration-tree inventory now has an explicit complete-read record for this compiler adapter.

### Remaining inline-marker hits classified

Read complete include/amigaos/xpk/xpkprefs.h (133 lines) and the remaining include/libraries/mpega.h lines 76–165, completing that header with its earlier prefix read. Both assembly keyword matches are commented callback examples; neither header defines an inline transfer routine. XPK preferences declare semaphore ownership conventions and structures, while MPEGA declares stream/control/access structures and frame limits. No additional active inline-code candidate was found. This closes all six files identified by the initial inline/assembly marker search, but does not substitute for checking generated call macros and remaining compatibility declarations.

### AROS diskimage generated API declarations completed

Read complete include/aros/{proto/diskimage.h,clib/diskimage_protos.h,defines/diskimage.h} and include/mmakefile.src. Vector slots 7–14 and argument declarations agree with the previously reviewed public device vectors/configuration. UnitControl's generated stack tag array uses an iptr-attributed type when the toolchain advertises it, otherwise ULONG; a 64-bit fallback mismatch is conditional on that compiler feature macro being absent and is not established by the header alone. No additional active argument/vector mismatch was found. Modern include copying exports devices/diskimage.h; these generated proto/defines compatibility files are additionally used by standalone source include paths rather than all being copied by this one mmake rule. Remaining four external-library header families require their own classification.

### AROS bzip2 compatibility call headers completed

Read complete proto/bz2.h, clib/bz2_protos.h and defines/bz2.h. Checked all nine exported operations against the bundled library vector table slots 5–13 and its already reviewed wrappers. Argument order/register assignments agree; buffer length pointers are ULONG and the wrapper intentionally casts them to the compression API's unsigned-int representation. No independent vector/argument mismatch was established for the selected AROS ABI. These headers provide library calls rather than additional decompression algorithms; existing allocator/lifecycle observations retain their standalone-library applicability.

### AROS zlib compatibility call headers completed

Read complete proto/z.h, clib/z_protos.h and defines/z.h. All twenty generated operations correspond to bundled z_library/init.c vector slots 5–24 in the same order; buffer, dictionary and checksum argument declarations agree with the previously read wrappers. No extra bounds/ownership logic exists in these call macros. This closes this generated API family without asserting review of an externally fetched compression implementation. Existing DISK-173 allocation semantics and wrapper lifecycle observations remain separate root causes.

### AROS MPEGA compatibility call headers completed

Read complete proto/mpega.h, clib/mpega_protos.h and defines/mpega.h, and compared the active audio/mp3_mpega.c calls. All seven operations use consistent vector slots 5–11. decode_frame's prototype describes an array of PCM-channel pointers, whereas its call macro annotates the argument as WORD *; the caller supplies its channel-pointer array. Both carry the same pointer value through the library-call adapter, so the type annotation discrepancy alone does not establish wrong-address runtime behavior. Preserve it as a declaration-quality limitation, not another transfer defect. The current modern CUE rule selects MPG123; MPEGA remains an alternative/legacy configuration as recorded earlier. No external decoder implementation review is claimed.

### AROS Expat compatibility declaration pass, first half

Read proto/expat.h and clib/expat_protos.h completely, and defines/expat.h lines 1–205. Parser creation, parse/buffer access and callback registration macros through slot 33 agree with the already reviewed bundled wrapper interface order and argument roles. This pass records only the first half of the generated macros; lines 206–405 remain to complete before declaring this family fully covered. Existing bundled allocation-failure behavior is DISK-173, independent of these declaration checks. Modern shared/fetched Expat remains distinct from this standalone compatibility library.

### AROS Expat generated call family completed

Read defines/expat.h lines 206–405, completing the family with the preceding proto/clib/first-half pass. Compared every slot against bundled expat_library/init.c's vector table: 65 exported operations occupy slots 5–69 in matching order, including parser-state and allocator calls. No independent generated vector mismatch was found. Return-width differences between an external Expat version and the legacy wrapper remain an API-version/ABI limitation rather than proof that this bundled table dispatches incorrectly. This completes all sixteen files in include/aros (five proto/clib/defines families plus ctype) with explicit content records. The larger AmigaOS compatibility tree and remaining companion SDI macros are still separate inventory work.

### SDI hook adapter completed

Read SDI_hook.h completely. Its AROS hook construction uses HookEntry with the C callback in h_SubEntry, and its dispatcher gate uses IPTR return values and AROS user-function wrappers. This agrees with the previously traced Zune SignalFunc registration and does not establish a pointer-width truncation in that active path. Non-AROS m68k/MorphOS branches use distinct register/emulation conventions and remain configuration-dependent. InitHook copies entry/subentry and assigns data without reallocating ownership. No new callback-ABI candidate was established from this adapter; semaphore/reentrant-caller limits remain the separately reconciled callback observation.

### SDI interrupt adapter completed and applicability checked

Read SDI_interrupt.h completely and searched disk-image C sources for its inclusion and interrupt/handler construction macros. No in-tree C use was found. Its MorphOS trampoline uses an emulation gate; other branches use register/declaration macros directly and static Interrupt initializers. Since the source search establishes no active disk-image expansion, potential ABI concerns in this unused compatibility adapter are not promoted to operational disk-image findings. This closes its content review and records the limit of the caller search, without asserting every preprocessor include graph was compiled.

### SDI varargs adapter completed

Read SDI_stdarg.h completely and traced its usages in formatting helpers and device progress/error wrappers. VA_ARG is deliberately a raw argument-area adapter here, not the standard operation consuming one typed argument. Consequently its returning the va_list/overflow area is not independently evidence that the next argument is omitted. The AROS SetDiskImageError path uses an IPTR array macro and RAWARG declaration instead of the non-AROS linear-varargs wrapper. Formatting helpers have their separately recorded va_copy handling. No additional active varargs defect was established in this pass; unsupported legacy ABI assumptions remain configuration-specific.

### SDI miscellaneous adapter completed

Read all 120 lines of SDI_misc.h. It defines RawDoFmt character-output callback adapters, not task/process entry wrappers. Searched disk-image C sources for its inclusion and PUTCHARPROTO expansion and found no use. Its example writes through a caller-provided pointer but is comment text, not compiled buffer-writing code. No active disk-image candidate is established by the unused compatibility macros; this records complete content review and explicit caller-search applicability. SDI_lib.h remains the last SDI companion header awaiting its own complete read.

### SDI library adapter and companion set completed

Read all 238 lines of SDI_lib.h and searched disk-image C sources for its inclusion/library-prototype/vector macros. No active C expansion was found; the reviewed bundled/device entry points use their own AROS library wrappers. SDI_lib distinguishes ABI_V1 and older stub naming, OS4 direct functions and MorphOS gates, while omitting separate vararg-vector entries on relevant branches. These unused compatibility choices do not establish a current disk-image dispatch failure. This completes the six local SDI companion headers (compiler, hook, interrupt, stdarg, misc and lib) with explicit applicability records. The generated AmigaOS external-library declarations remain the larger outstanding compatibility-tree group.

### Disk-image AmigaOS compatibility shim review

Read the complete fcntl.h, string.h, sys/types.h, sys/stat.h and devices/newstyle.h under include/amigaos. The VBCC/non-Newlib path intentionally supplies narrow POSIX declarations and zero permission constants; these do not establish an active modern AROS addressing defect. The other sys/stat.h branch incorrectly forwards to fcntl.h rather than sys/stat.h. No disk-image source includes this shim directly, and the located CUE fdio.c fstat stub does not include sys/stat.h; therefore this is an unused compatibility-header defect, not an established disk runtime failure or a new numbered candidate. The NSD structure and command declarations do not add a new discrepancy to the reviewed device-query contract.

Rechecked numbered heading/anchor uniqueness after DISK-252 and corrected the stale integrity counts. This verifies report structure only. Remaining AmigaOS API-header coverage and final candidate normalization are still open.

### Legacy disk-image call-header families reconciled

Read all fifteen AmigaOS clib, inline and proto headers for diskimage, bz2, z, mpega and capsimage (each family has three files). Also read the complete diskimage, bz2 and z SFD declarations. Diskimage's vector offsets 42–84 and register arguments agree with its SFD; the varargs UnitControl entry deliberately shares UnitControlA's vector. Bz2's nine vectors run from 30–78, and z's twenty vectors from 30–144, with register selections matching the SFD and previously reviewed wrapper implementations.

Some clib declarations represent pointers as LONG (z stream/buffer parameters and bz2 source buffers), while VBCC inline declarations cast/use pointers and the SFD uses pointer types. These clib files explicitly restrict themselves to 32-bit integers, and the m68k target is a 32-bit ABI. This is a declaration-quality and compiler-compatibility limitation, not evidence of current 64-bit AROS pointer truncation: modern AROS uses its separate headers. A compiler rejecting the inconsistent source-level pointer declarations remains conditional; no compilation was performed.

MPEGA's inline pcm argument is WORD **, matching its clib declaration and the intended two-buffer API. CAPS uses device-base vectors 42–120, with consistent declarations/macros within the family. Its implementation and an independent external CAPS vector specification are not in this local comparison, so external ABI correctness is not claimed. GCC/other-compiler proto branches rely on SDK inline or pragma headers, while the supplied inline/*_protos.h files serve VBCC. Absence of those SDK files from this repository alone is not a build defect. No new numbered candidate follows from these fifteen headers. The Expat and remaining external API-header families still require their own coverage reconciliation.

### Legacy Expat compatibility family completed

Read all three AmigaOS Expat headers (clib 84 lines, proto 23, inline 203) and its complete SFD. The inline operations occupy consecutive offsets 30–414 in the same 65-operation order as the SFD and previously reviewed bundled vector table. Pointer casts retain values in the intended 32-bit legacy ABI. Callback code pointers are forwarded in address registers; portability to a different ABI is not claimed. The declarations expose a separate source-level issue below. This closes the Expat compatibility family, not the remaining external-library headers.

### XAD, XPK and AmiSSL master call-header coverage

Read all nine AmigaOS clib/inline/proto files for xadmaster, xpkmaster and amisslmaster: 471 lines total. Compared declarations, macro arguments and register assignments within each family and located actual callers in plugins/xad.c, plugins/xpk.c and plugins/crypt.c. XAD uses allocation/info/extraction/free entry points; XPK uses object allocation, examination, unpacking and free; Crypt uses InitAmiSSLMaster, OpenAmiSSL and CloseAmiSSL. Their supplied wrappers consistently forward those argument lists.

XAD's varargs wrappers preserve the tag-list address register (a0, a1, a2 or a3 as required), point it at the stack arguments after the save, call the corresponding A vector, and restore it. XPK follows the same pattern for its tag wrappers. These are VBCC-specific m68k stubs, not generic C varargs implementations; no cross-architecture ABI guarantee is inferred. The C99-gated inline varargs macros rely on the compiler mode or an external stub when unavailable. This is a configuration prerequisite, not a demonstrated unresolved-link defect without the link environment. AmiSSL master has five consecutive vectors 30–54 with consistent clib and inline signatures.

No additional root cause was established. External XAD/XPK/AmiSSL implementations and independent vector specifications were not inspected in this pass, so internal header agreement is not proof of external protocol correctness. Existing plugin allocation, cleanup and extraction candidates remain governed by their previously traced caller evidence. The large AmiSSL/OpenSSL declaration forest and GUI external-call families remain separate coverage items.

### Crypt plugin AmiSSL declaration boundary reviewed

Re-read Crypt's library initialization, password/key setup, cleanup and read path, and read complete AmiSSL clib/proto, libraries/amissl.h, libraries/amisslmaster.h, amissl/amissl.h and openssl/idea.h. Read inline/amissl_protos.h lines 1–35 and 7198–7224, covering every AmiSSL wrapper Crypt actually invokes: InitAmiSSLA, CleanupAmiSSLA, IsCipherAvailable, idea_set_encrypt_key, idea_set_decrypt_key and idea_cbc_encrypt. Their local signatures/register argument lists agree with the declarations and plugin call argument order. IDEA_BLOCK=8 matches the two-ULONG IV on the supported m68k ABI; the 512-byte operation is a whole number of cipher blocks. No independent wrapper bug was established.

The aggregator tests PROTO_AMISSL_H but the supplied proto header defines _PROTO_AMISSL_H. Consequently AMISSL_AMISSL_H is not set by its intended path, and OpenSSL headers repeatedly request proto/amissl.h. The actual proto guard still stops re-entry; this mismatch alone does not prove infinite include recursion. Individual OpenSSL include guards also prevent repeated type bodies. Retain as a compatibility-header maintenance observation, not a new disk failure. External SDK header substitutions can change this behavior.

Coverage is deliberately bounded: the 7,575-line AmiSSL inline header was inspected at the six used calls, not read in full. Its unrelated TLS, ASN.1 and other exported functions are external API declarations, not an implemented disk transport. This pass does not claim an exhaustive OpenSSL library or cryptographic-security audit. The complete local declaration-forest inventory remains separate from the reviewed Crypt call boundary and from existing image bounds/ownership candidates. No code, preprocessor, compiler or tests were executed.

### Repository-wide block-command inventory cross-check

Repeated repository-wide path/build inventory and searched C implementations under arch, rom and workbench for NSCMD_TD_READ64 and TD_GETGEOMETRY. All driver implementations located by these signatures fall within the named driver families. Remaining matches are filesystem consumers, diagnostic utilities, format/install tools and bootloader installers. This search is a supplementary discovery method, not proof that every driver must contain either signature. Disk-named datatypes/fonts/catalogs are not storage drivers. The worktree remains clean.

The search exposes a concrete remaining integration gate: workbench/system/BHFormat/main.c submits format/read/update commands; workbench/c/Install.c and both arch/all-pc/boot/grub-tools/Install-grub.c and grub2-tools/Install-grub2.c select 64-bit read/write commands; workbench/c/HDTool/deviceio.c fetches geometry. Their block-command negotiation, bounds and write accounting need explicit consumer-contract review or a documented scope limitation; no complete-source coverage is claimed here. Likewise workbench/c/Partition and trackdisk preferences need inventory/build classification for direct partition/configuration integration. General filesystem algorithms remain outside the primary scope, but their relevant request contracts can be traced as needed. These newly explicit inventory items prevent treating the driver-local completion records as proof of the whole integration boundary.

### HDTool consumer review: adapter and partition listing

Read deviceio.c, deviceio.h, mmakefile.src and the complete list.c. The five-unit build includes main, deviceio, install, list and setup; the latter three implementation coverage gates are not all closed by this pass. Contrary to the inventory signature, iscorrectType sets up TD_GETGEOMETRY but its DoIO and type filtering are inside #if 0: it always returns TRUE. No uninitialized geometry consumption occurs there. openIO's failure paths free the request/port; closeIO is called only after successful open in the located list caller.

New source-supported integration observations, pending final numbering/deduplication:

- **HDTool partition attributes (low severity, high confidence):** list.c:87–104 asks PT_ACTIVE while printing bootable and automount, despite checking support for PTA_BOOTABLE/PTA_AUTOMOUNT. rom/partition/include/libraries/partition.h:89–92 defines these as three distinct tags. The display therefore reports active status as both other properties when supported, or can print an uninitialized local if the active tag is unsupported and left untouched. Trigger is HDTool partition-detail listing; no disk write occurs.
- **HDTool identification string conversion (medium conditional memory safety, high confidence in helper defect):** deviceio.c:49–60 trims backwards without a lower bound. A returned model field consisting entirely of zero/space bytes walks before the destination and can overwrite earlier stack bytes. A field ending in a nonblank byte receives no guaranteed trailing NUL, and trimming begins at byte 38 of a 40-byte field rather than byte 39. identify calls it with 40 bytes and printHD supplies char id[64]. The current printHD prints the literal "id", so the unterminated output is not presently passed to printf as a string. The backward overwrite still requires identify's nonstandard one-byte 0xEC HD_SCSICMD to succeed on the selected backend; no active supported backend accepting it is established yet.
- **HDTool SCSI request initialization (conditional, requires backend acceptance):** deviceio.c:64–75 leaves scsi_Flags, sense pointer/length and output/accounting fields uninitialized. Existing ATA/ATAPI paths consult flags/sense fields, but rejection of its nonstandard command can prevent a dangerous transfer. Do not assert a current data transfer or arbitrary sense write without completing that acceptance trace.

The list.c scsi.devce spelling excludes normal scsi.device from its six-unit branch; other names default to one unit, and probing stops at the first failed unit. This is a bounded enumeration limitation, not evidence that disk drivers lose units. Partition attribute return checking and argument parsing remain integration observations to reconcile with library contracts and main/setup callers. No code was executed.

### HDTool implementation and ATA rejection trace completed

Read main.c, install.c, setup.c and their three headers, completing the five implementation units, four headers and build file. Install/setup only print proposed commands or configuration; they do not perform disk writes. Optional ARGS/F reaches handlers that immediately call strlen(args) without checking NULL; this is a candidate command-line crash pending ReadArgs omission-contract confirmation. The source also replaces the terminator with a newline within the supplied string; no extra-byte overwrite is asserted without its allocation contract.

ATA direct-access cmd_DirectScsi routes the one-byte 0xEC request to SCSIEmu. That opcode takes the default illegal-request path and returns an error, so identify does not reach w2strcpy on this backend. The trimming defect remains conditional for a backend accepting the request. However, rejection itself writes autosense when uninitialized flags enable it, closing the earlier unresolved request-initialization question and superseding the claim that acceptance was required.

### Partition command complete local-source pass

Read args.c, args.h, mmakefile.src and all 885 lines of main.c in consecutive chunks. The build contains exactly main and args. Traced WritePartitionTable and ClosePartitionTable public contracts: writes return zero/error and affect only the selected table; closing discards changes and clears root->table. New observations below need final root-cause normalization and further handler tracing where indicated.

- **Unchecked partition-table writes (high conditional persistence correctness, high source confidence):** main's final save ignores all WritePartitionTable return values, closes the changed tables, and prints completion/returns success while error remains zero. A failed child-table write can be followed by successful parent/root publication; no rollback is provided. Concrete trigger is a write/protection/device error during final save. The library explicitly returns errors, so this is a caller defect independent of individual backend write failures.
- **Narrowed geometry arithmetic (high conditional placement correctness):** CylToBlock casts only after multiplying LONG cylinder by the native-width surfaces/blocks-per-track product. FindLargestGap/CheckGap similarly assign native-width products to QUAD after evaluation. BlockToCyl returns LONG after division. On 32-bit targets, an actual block position above the product's range wraps before widening and can distort free-gap selection; on 64-bit native-width targets some products have a wider range. Final overlap/handler-rejection outcome needs tracing, and malformed zero geometry can separately cause division by zero. MBsToCylinders widens its byte numerator using MB's LL type, so do not claim its numerator has the same 32-bit multiplication defect.
- **Default filesystem documentation mismatch (low, high confidence):** no SYSTYPE selects dos3/FFSIntl despite the command's documentation promising SFS; WORK defaults to SFS. This affects filesystem type and size cap, not inherently media corruption.
- **RDB MAXWORK behavior (medium, high confidence):** pure-RDB work creation is guarded by workSize != 0, even though MAXWORK is accepted and used for sizing earlier. MAXWORK without WORKSIZE leaves workSize zero on uncapped gaps, so a requested work partition is omitted. Cases where the size cap sets workSize require separate classification.
- **WIPE allocation/write accounting (high conditional, requires write-helper trace):** recursive destruction does not check AllocMem before passing its buffer to WritePartitionDataQ, and ignores the data-write result before destroying tables. Whether NULL is rejected or dereferenced depends on that helper/driver path. No allocation success is assumed.
- **Scheme/configuration checks:** GPT selects the generic non-RDB creation path, whose CreateMBRPartition passes one-byte type IDs; GPT's setter accepts sixteen-byte types. Trace add/handler behavior before asserting a valid GPT entry or corruption. Unsupported BOOTTYPE has an empty branch; requested boot creation can be silently omitted. BOOTSIZE/SYSSIZE/WORKSIZE values have no positive-range validation before unsigned cylinder conversion; exact failure modes remain handler-dependent.

Pure RDB aliases diskPart and parent/root, so the final save/close attempts that root again after close; ClosePartitionTable sets its table to zero, preventing a double-free in the public wrapper. The second write returns an ignored no-method error; do not classify this alias alone as a UAF. Failure paths choosing a NULL extPartition, gap rounding, reserved-attribute widths and partial container creation remain explicit follow-up contracts. No partitioning code was executed.

### Partition GPT, WIPE and attribute-width follow-up

Traced AddPartition, CreatePartitionTable, WritePartitionDataQ and the current GPT function table, plus MBR/EBR table-attribute handlers. GPT has NULL create/add entries. A fresh SCHEME gpt invocation therefore fails CreatePartitionTable rather than constructing an invalid one-byte GPT type. On an existing GPT root, AddPartition returns NULL and creation fails; the type setter's sixteen-byte check is not reached through successful creation. Reclassify the one-byte-type observation as an unsupported command configuration, not proof of malformed GPT writes. Existing table modifications and earlier WIPE actions still require their own error handling.

WritePartitionDataQ forwards Buffer directly into io_Data and DoIO without NULL validation. Thus WIPE's unchecked 4096-byte allocation does submit a NULL buffer when allocation fails; a driver may reject it or dereference it. Retain high confidence in invalid submission and ignored write status, but backend-specific memory corruption remains conditional. If a sector size exceeds or does not divide 4096, the fixed WIPE length additionally violates the helper's stated whole-block prerequisite; no active such media is assumed.

A separate concrete width defect exists in CreateMBRPartition: reserved and leadIn are IPTR, while MBR PTT_RESERVED and EBR PTT_MAXLEADIN handlers write through LONG *. On a 64-bit target, reserved's unwritten upper bytes remain uninitialized, and its subsequent arithmetic reads the full IPTR. leadIn starts zero, so a little-endian 32-bit write retains a zero upper half, but a big-endian 64-bit write occupies the high half and scales the value incorrectly. The corresponding reserved local in CreateRDBPartition is ULONG and does not have this exact mismatch. Severity is high conditional placement correctness; confidence high in width mismatch, final added-partition bounds/rejection outcome still follows the handler trace. This caller mismatch is separate from DISK-059's narrowing of existing partition addresses.

These follow-ups supersede the prior tentative GPT acceptance and WIPE helper questions; they do not close final partition-command candidate normalization. No project code was run.

### Partition creation-handler validation reconciliation

Re-read MBR AddPartition/SetDosEnvec, EBR AddPartition and RDB AddPartition. These paths do not independently validate the proposed range against all existing partitions: MBR finds a slot and converts geometry; EBR orders by start and inserts a handle; RDB orders by low cylinder. Thus a malformed caller range is not universally rejected. High-cylinder clamping in the command does not validate low <= high or disjointness. Arithmetic observations remain architecture/range-dependent rather than a universally reproduced overlap. DosEnvec fields are IPTR, confirming 32-bit evaluation on 32-bit targets and wider evaluation on 64-bit targets.

A definite NULL-parent path remains: an existing four-entry MBR with no extended container makes CreateMBRPartition fail to find a free slot, leaving extPartition NULL without setting error. partCount == 4 subsequently selects parent = extPartition; BlockToCyl(&parent->de, ...) dereferences it. This is a normal full-primary-table rejection case, not malformed media. Retain as a source-supported command crash candidate pending final normalization.

### Partition candidate normalization checkpoint

Promoted the source-supported full-MBR and uncapped pure-RDB MAXWORK observations to DISK-257/258 with bounded triggers. Remaining command observations include narrow free-gap products, unchecked WIPE buffers/results, unsupported BOOTTYPE, default-filesystem documentation mismatch and positive-size validation. Existing failure/accounting findings are not counted again merely because chronological notes repeat them. Baseline revalidated as 282a0454356dc2b9554ab023997d7f7ca24fc03e.

### BHFormat block-command pass: partial main coverage

Read main.c:440–790 (device open/query/allocation, format/update/clear, verify and transfer loop), main.c:360–440, format.h and mmakefile.src. The build includes main, cli, gui and locale; this does not close complete command coverage. Located geometry construction at 332–342: track/cylinder/offset variables are unsigned 64-bit, with explicit widening of track multiplication. No generic 32-bit cylinder-product bug is inferred from variable names alone.

Source-supported observations pending caller/driver reconciliation:

- **Exactly-4-GiB negotiation mismatch (medium failure after partial formatting, high source confidence):** bGetExecDevice queries 64-bit command support only when ibyEnd > 2^32, whereas bTransferCylinder selects the 64-bit command when offset + length >= 2^32. A partition whose exclusive end equals exactly 2^32 skips negotiation but its final transfer uses FORMAT64/READ64. A driver supporting only ordinary commands can therefore reject the final segment after earlier segments have been formatted. The request starts below 2^32, so an ordinary command could represent its start; no wraparound access or wrong-region write is asserted merely from the command choice.
- **Successful short transfer accounting (medium conditional):** bTransferCylinder advances by requested cbyLength after zero DoIO error, without checking io_Actual. Formatting may proceed after a short successful write; verification may compare stale/uninitialized read-buffer bytes after a short read. Need reconcile active device short-success contracts before claiming a routinely reachable event.
- **Zero/non-block transfer selection (conditional malformed environment):** suspect-IDE mode can choose MaxTransfer directly; a zero value causes cbyLeft never to decrease if the driver accepts zero-length commands. A non-sector-aligned limit can produce invalid chunks. Cylinder/track sizes and MaxTransfer preconditions need the earlier environment/caller pass.
- **Command support completeness:** query checks only READ64, then later uses FORMAT64; it assumes a driver implements the entire NSD family. Trace active supported-command tables and the public contract before classifying a separate omission. Null SupportedCommands on a malformed successful query is also a contract-dependent path, not assumed current driver output.

Format checks update and clear errors and does not silently ignore those return codes. Allocation cleanup is centralized in FreeExecDevice, but its callers still require review for failed-open exits. No format operation, build or test executed.

### BHFormat main and CLI source coverage completed

Read main.c:1–359 and 790–932, completing all 932 lines with the earlier consecutive slices. Read the complete cli.c and GUI slices 162–207 and 418–480. Main always calls FreeAll after its chosen frontend returns, closing the remaining request/port/allocation/inhibit state even when CLI jumps to cleanup on bGetExecDevice failure. Reject the generic failed-open cleanup-leak suspicion. GUI full-source and locale declarations remain separate local inventory work.

The environment validation checks startup/envec memory and table size, but does not require nonzero sector/track/head sizes, ordered cylinders, bounded cylinder byte counts or a usable MaxTransfer. Thus zero/nonaligned environment observations are not eliminated by setup. cbyTransfer is ULONG while cbyCylinder/cbyTrack are 64-bit; a selected transfer exactly divisible by 2^32 can narrow to zero. On 32-bit allocation/size_t targets a cylinder exceeding their representable size can also narrow the allocation while subsequent pointer-limit/fill arithmetic retains the 64-bit value. These are malformed/extreme geometry conditions, not ordinary disk-capacity triggers; individual drivers' normal small cylinder geometry does not demonstrate them.

Both frontend format loops use ULONG icyl against IPTR HighCyl. If HighCyl >= ULONG_MAX, increment wraps and the loop can continue indefinitely/revisit lower cylinders; a 64-bit LowCyl can narrow before the first iteration. Existing DosEnvec parser/configuration findings provide possible bad inputs, but no ordinary mounted disk is asserted to have such cylinder counts.

A separate CLI confirmation defect is source-visible: ch is uninitialized, Read's byte count is scoped inside the prompt loop, and after EOF/error the code only checks ch == 3. EOF after any non-Ctrl-C/non-Return character exits the loop and proceeds to formatting without Return confirmation; EOF before any character additionally reads uninitialized ch. Trigger is input exhaustion/error at the insert-disk prompt. This is a formatter caller-flow defect, not a driver write-protection bypass. The loop correctly blocks filesystem initialization after a detected format/verification error through formatOk.

The completed main pass also bounds the short-transfer observation: verification compares the full requested cylinder, but no io_Actual validation is introduced elsewhere in main/CLI. Runtime short-success reachability still depends on the active driver's contract. No format or other project code was executed.

### BHFormat GUI and local build inventory completed

Read gui.c:1–162, 207–418 and 480–732, closing its 732-line implementation with the previously recorded 162–207 and 418–480 slices. Read locale.c and locale.h completely. Together with complete main/CLI, format.h and mmakefile, all four local implementation units and two handwritten headers are covered; generated strings/catalog text is a localization dependency rather than a disk algorithm. No build was performed.

New GUI integration observations pending numbering/normalization:

- **Expired InfoData local (medium conditional display/crash, high source confidence):** VolumesToList declares infoData inside if(lock), stores its address in pInfoData, then exits that scope before ComputeCapacity(volume, pInfoData). When a matching volume is found, ComputeCapacity reads an object whose lifetime ended; overwritten block counts can also reach its unchecked id_NumBlocks divisor. No lifetime extension is supplied by storing the pointer. Other listing paths leave pInfoData NULL and avoid this read.
- **Uninitialized selected-device volume string (medium conditional display memory read):** rcGuiMain's Workbench name/device argument branch sets the device but leaves szVolumeName uninitialized when pdlVolume is NULL; later RawDoFmtSz formats it as a string. The lock/volume and selection-dialog branches initialize it. This can display arbitrary stack bytes or run past the array; exact output/crash unverified.
- **Ignored operation result (medium false success, high confidence):** btn_format_function tracks a local rc for format/verification/filesystem failure, returns the same Quit ID on all outcomes, and never propagates rc. rcGuiMain sets RETURN_OK after application execution irrespective of the callback result. Thus a failed GUI format can exit success even though the callback showed an error. Cancellation-versus-failure return policy is separate, but ordinary device failure does not reach a failing exit code.
- **DOS-list lock leak on missing matching device (medium conditional resource/concurrency):** Workbench lock/volume branch acquires LDF_DEVICES|LDF_READ, searches by task, then jumps to cleanup when no match; that exit does not unlock it. Successful discovery passes the lock to bGetDosDevice, which unlocks it. The missing-match trigger can depend on stale/removed volume state; do not assert ordinary success leaks.

Device/volume name copying uses fixed 32-byte list fields and an initial 128-byte temporary without checking source length. Existing normal <=30-character labels fit; an overlength DOS-list entry requires caller/registration contract tracing before a general overflow candidate. The selection dialog copies its chosen entry before disposing the list/pool, so reject a returned-entry pool UAF. main's FreeAll still handles outstanding device/inhibit state after the GUI returns. Remaining short-transfer and command-capability observations retain their own limits.

### Trackdisk preferences and startup consumer reviewed

Read complete trackdisk_prefs.c (208 lines), trackdisk_prefs.h, locale.c/h, mmakefile.src and public prefs/trackdisk.h. Traced all TDPR/file-path consumers with a repository-wide search, locating and reading complete workbench/c/iprefs/trackdiskprefs.c. The GUI writes six-ULONG unit records followed by one ULONG TAG_DONE; IPrefs reads pairs of ULONG. Both share that 32-bit serialized format, so do not mistake the persisted file for a native-width TagItem array on 64-bit. The final single-ULONG terminator is not returned as a full FRead pair, but EOF ends the load after the unit records; no out-of-bounds read is established by that incomplete terminator alone.

A different native-layout issue affects GUI creation: WindowGroup has TD_NUMUNITS native struct TagItem records followed by ULONG TagChild, an Object *ButtonsGroup, and ULONG TagDone. Main passes the whole struct to MUI_NewObjectA as a TagItem array. On 64-bit targets TagChild is only 32 bits; padding before ButtonsGroup happens to be zero in the static object, so the child tag can work on little-endian targets, but big-endian native ti_Tag reads place the ULONG tag in its high half. TagDone similarly relies on zeroed tail padding. This is a conditional 64-bit big-endian GUI-construction ABI defect; the 32-bit target layout fits its intended array, and no ordinary 64-bit little-endian failure is asserted from the field widths alone.

SavePrefs silently ignores Open/Write/Close failures, then the caller applies settings and exits success; a failed save can truncate or fail to persist the preferences despite apparent success. This is a low-to-medium conditional configuration-persistence candidate, separate from disk-image preference writing's existing root cause unless final grouping explicitly combines their caller patterns. Unit settings are obtained/applied through the documented TDU_PublicUnit prefix; no arbitrary wrong-unit cast is inferred. ControlsToPrefs replaces PubFlags with NOCLICK or zero, discarding any other flag bits; only NOCLICK has been traced as actively used by current drivers. Retry input is constrained to 1–10 in the GUI, while IPrefs accepts any stored ULONG and assigns it to the public retry field; malformed-file retry behavior still needs the driver loop contract before classification.

Generated catalogs and legacy standalone makefile are inventory/configuration dependencies; no complete translation-content audit or standalone build is claimed. Device settings are not executed or modified during this review. This closes the identified preference producer/startup-consumer source pass, with final candidate normalization remaining.

### GRUB2 installer raw-I/O boundary: initial coverage

Read Install-grub2.c:125–352, 630–705 and 760–808 plus its Volume definition. This covers path-to-startup lookup, geometry, NSD negotiation, volume allocation/teardown, raw-I/O wrappers, boot-volume acquisition and a boot-sector publication slice. The 1,465-line source is not yet fully reviewed. Located related helpers in the older Install-grub.c and C:Install without claiming their complete coverage.

- **NSD capacity arithmetic:** startblock/countblock are ULONG, and nsdCheck adds/multiplies them without widening before testing its threshold. This shares DISK-060's capability-selection arithmetic root cause; final trace should group the installer manifestation rather than add a duplicate solely for its file. Distinctly, unsupported/missing READ64/WRITE64 only prints warnings and leaves ordinary commands active; a real high-offset operation then supplies high bits in io_Actual to an ordinary command. Wrong-region effects depend on the driver's ordinary-command interpretation.
- **Address arithmetic:** _readwriteBlock widens startblock + block only after the ULONG addition, so a sector sum beyond 2^32 wraps even though offset itself is UQUAD. fillGeometry also stores native-width cylinder products into ULONG fields. Extending DISK-059's narrowing group requires bounded caller arguments; no arbitrary large ordinary disk is assumed.
- **Boot-volume allocation/error handling:** getBBVolume dereferences initVolume's return without checking NULL. Successful partition validation followed by allocation/open failure causes a NULL dereference. It also ignores its initial readBlock error, tests the cleared/stale blockbuffer, and can treat failed RDB detection as available boot space; later boot publication rereads before writing, so final RDB overwrite cannot be asserted from this slice alone.
- **Startup name bounds/lifetime:** getDiskFSSM copies path bytes into dname[32] with an unbounded UBYTE index until colon/NUL, then appends another byte. An overlong device prefix overruns the array; a >=256-byte prefix can also wrap the index. It unlocks the DOS list before examining dn->dn_Startup, so concurrent removal lifetime requires the DOS-node contract. No current concurrent removal is assumed.

Raw wrappers propagate DoIO's error value but do not check short-success io_Actual. writeBootIMG's inspected path checks the later old-MBR read result before writing, preserving that specific error gate. Source-only review; no boot data read or written.

### GRUB2 filesystem sector-list integration pass

Read Install-grub2.c:352–630 and 808–1197 consecutively, plus updateCoreIMG's call-site slice 1270–1334. This completes filesystem/partition identification and both FFS/SFS sector-list collectors; full installer coverage still requires the opening/header region, 705–760 and remaining publication/main code. Filesystem algorithms generally remain out of scope, but these collectors directly construct boot-sector references and are part of this integration boundary.

FFS starts blk_count at zero and first tests blocklist[blk_count].sector_lo/count before decrementing. Its caller supplies blocklist=(struct BlockNode *)&buffer[128], one past the 512-byte sector. Therefore the initial probe reads beyond that sector, whose preceding fourteen nodes were cleared but whose node zero was not. If adjacent bytes look like a matching run, it can update count outside the intended list instead of creating the first negative-index node. Exact allocation extent at the outer caller remains to trace; a buffer larger than 512 may keep the access within its allocation while still consulting unrelated/uninitialized data. The negative-node exhaustion guard prevents an unbounded downward list write; do not conflate that with the initial node-zero access.

The FFS collector follows extension-block pointers until zero without cycle detection or metadata checksum/bounds validation. SFS collectors similarly trust node counts, record sizes, node-number differences and tree pointers. Its object-node and extent-tree read calls ignore returned errors before using the old blockbuffer; leaf index and B-tree entry offsets are not checked against SizeBlock*4. String skipping searches for two NUL bytes without a block-end limit. Zero interior node divisors, metadata cycles or out-of-range offsets can thus divide by zero, loop or read past the buffer on malformed/read-failed metadata. These are installer parser candidates, not claims about normal FFS/SFS behavior or the separate filesystem driver. Root/object-container reads that do check retval retain their error gates.

isvalidFileSystem bounds partition matching by the volume's starting block, uses narrow ULONG ancestor/geometry offsets, and narrows logical partition number to BYTE. This supports only the explicitly selected legacy layout/partition-number range; final grouping with DISK-059 requires the actual high-range trigger. A valid standard volume does not establish malformed-input traversal. No disk code executed.

### GRUB2 installer full-source/build milestone

Read remaining Install-grub2.c:1–78, 705–760 and 1197–1465; previously inspected Volume/constant region covers 78–125. The complete 1,465-line implementation and mmakefile.src are now recorded. Build lists this single source and the external GRUB2 include directory; external GRUB itself is not audited by this milestone.

The outer ownership trace resolves the FFS initial node-zero observation: main obtains bbvol with initVolume's blockbuffer allocated as de_SizeBlock*4 bytes. installGrubFiles passes that buffer to updateCoreIMG, which passes buffer+512 as blocklist. For conventional 512-byte geometry, FFS's initial blocklist[0].sector_lo/count reads beyond the actual allocation, not merely an intended subregion; if its condition happens to match it writes count beyond the allocation. Larger sectors can keep this access inside the allocation but outside the initialized first-sector list. The collector's first negative entry should be selected before testing any existing run. This is independent of malformed filesystem metadata.

Publication/caller findings:

- getBBVolume's failed initVolume dereference is active from main. If block zero is an RDB, it prints no boot space but does not uninitVolume before returning NULL, leaking the open device/request/buffer/port; no double-free is asserted.
- main initializes ret=RETURN_OK and leaves it unchanged when ReadArgs fails, getDiskFSSM returns NULL, or getGrubStageVolume returns NULL. Its else-if(fssm) after if(fssm != NULL) is unreachable. Therefore several failed installer attempts report errors but return success. The actual installGrubFiles failure does set RETURN_ERROR.
- writeCoreIMG checks its Seek/Read/Write byte counts; however updateCoreIMG ignores Close's outcome before boot-sector publication. flushFS tries ten Inhibit operations but returns no success flag, so exhausted flush attempts do not stop raw-sector traversal. Concurrent relocation/cache coherency outcomes require filesystem contracts, not assumed universal failure.
- Source buffers/coreimgpath and bootimgpath begin uninitialized, but grubpath must contain a colon to reach this caller. The current AddPart implementation treats a fully qualified filename as replacement and does not scan the previous destination. Therefore reject the simplistic uninitialized-initial-AddPart claim on that successful path. AddPart failures remain unchecked: an overlong qualified path can leave an uninitialized destination later passed to Open; its length trigger needs bounded input.

Boot file pointer/list patching assumes 512-byte sectors and x86 little-endian structures; this module targets all-pc and BIOS GRUB2, so its lack of big-endian conversions alone is not an active deployment bug. Partnum is rejected by main before the partition-boot path. Final normalization and comparisons with the older installer/C:Install remain open. No installer executed.

### Older GRUB installer shared-helper comparison

Read Install-grub.c:90–330, 621–763 and 1084–1139. This covers startup lookup, geometry/NSD/open/cleanup/raw-I/O, boot-volume creation and its FFS block collector plus the main installation call slice. Its full 1,155-line implementation remains incomplete. The older helper repeats GRUB2's unbounded device-name copy, narrow geometry/add-before-widen arithmetic, warning-only unsupported 64-bit command fallback, unchecked boot-volume allocation and ignored initial read. These should be grouped by corresponding root cause where applicability matches, not counted again per installer.

The older FFS collector repeats the initial blocklist[0] read before decrementing; callers use the end of the first sector for the downward list, as in GRUB2. Unlike GRUB2's numeric exhaustion guard, its capacity check reads blocklist[blk_count-1].sector and assumes encountering nonzero pre-list boot code will stop expansion. A malformed/unusual stage2 prefix containing zero words can let it walk beyond the intended twenty cleared nodes and overwrite earlier boot code; enough zero prefix can move outside the buffer. Exact reachable stage2 image bytes and caller allocation remain a follow-up, so this is bounded as input-dependent rather than normal-image proof.

Both FFS collectors coalesce contiguous blocks by incrementing a UWORD count without checking its maximum. A run extending beyond 65,535 represented blocks wraps the count, corrupting the boot loader's list accounting; outer file-size restrictions and segmentation contracts still require checking before claiming an ordinary large image trigger. Metadata cycles are not rejected. Existing valid small standard GRUB files do not trigger these size/cycle conditions. The newer explicit list-capacity guard is not proof of this independent per-node count safety.

The older main call slice invokes installStageFiles without consuming its BOOL result, suggesting a false-success path; the full main return handling must be read before promotion. The block relocation comment says block zero is moved, but the inspected main slice only rereads it; lower publication helpers must resolve that convention before asserting a missing write. No installation performed.

### Older GRUB installer complete-source/build milestone

Read opening/constants 1–90, partition matching 330–621 (reread 435–485 untruncated), publication helpers 763–1084, final main 1139–1155 and mmakefile.src. These close the full 1,155-line source with prior consecutive slices. Its build selects one source and the external legacy GRUB include directory; external GRUB implementation remains outside this installer-source milestone.

The full main confirms it always returns zero, including ReadArgs, volume lookup/open, boot read and installStageFiles failures. That extends the shared installer false-success root cause: GRUB2 propagates final publication failures but misses earlier ones, while the old installer propagates none. No duplicate count per failure branch is warranted.

writeStage2 scans the second 512-byte sector's version string from byte 18 without an end bound, then copies the translated GRUB directory and appends /menu.lst without checking remaining space. A missing NUL version string or a long otherwise-valid qualified directory can overrun the sector buffer; the actual blockbuffer allocation is the boot volume's sector size, normally 512. This is an input-length/boot-image parser candidate distinct from the shared list-node-zero bug. Ordinary x86 alignment is supported here; no strict-alignment target is inferred.

The older collector returns first_block relative to the filesystem; writeStage1 adds s2vol->startblock once. GRUB2's collector adds startblock before returning, and its publication stores that absolute value without adding again. Reject a supposed universal missing/doubled partition-start bug from comparing only one side. The older main's block-relocation comment remains inconsistent with its actual reread, but getBBVolume clears bytes 0–445 before the later test, eliminating the expected DOS identifier there. This is a possible direct-filesystem support-path defect whose intended GRUB/file-system ABI requires more evidence; do not label it a reproduced filesystem overwrite.

Both boot publication functions preserve the old MBR partition area only after a checked successful raw read and refuse the final write when that read fails. This bounds earlier ignored-probe-read concerns: a later successful read can recover; a later failure stops publication. A failed earlier RDB-detection read followed by a later successful read still needs separate classification because the protection test is not repeated. No bootloader was installed or project code executed.

### C:Install complete-source consumer review

Read all 614 lines of workbench/c/Install.c in consecutive slices 1–180, 180–330 and 330–614; confirmed its source selection in workbench/c/mmakefile.src. This command contains legacy GRUB-stage logic despite its floppy-oriented documentation. Shared NSD threshold overflow, add-before-widen offset arithmetic, unbounded 32-byte device-prefix copy, warning-only unsupported 64-bit fallback and always-zero main exit belong with corresponding installer root causes already traced; they are not independent bugs merely because copied into a third utility.

Its FFS collector receives the end of the exact global ULONG stage2_firstblock[128]. Initial node-zero reads therefore exceed the entire static array, independently of variable sector allocation. It also clears entries backwards until a zero sentinel without a numerical bound, ignores the first header read result, and uses the older sentinel-based downward-capacity check. Malformed boot-file list bytes can drive pre-array writes; standard-image assumptions cannot replace a source bound. Per-node UWORD count overflow and extension-chain cycle limitations are shared with the older collector.

Independent caller/lifecycle observations:

- initVolume failure cleanup after successful OpenDevice does not call CloseDevice when boot reads/type validation fail; any created IORequest is also never deleted on its failure paths. It frees buffer/volume and deletes the message port, leaving the request/device open state leaked. The success uninitVolume does close/delete these resources.
- initVolume reads block one when block zero lacks DOS magic but does not gate subsequent magic/type validation on that second read's retval; stale data can drive type decisions after error. Exact erroneous acceptance requires matching retained bytes and is conditional.
- VF_MOVE_BB installation writes the saved first block to block one but ignores that return before replacing block zero with stage1. A failed preservation write followed by successful boot publication can destroy the original filesystem signature/boot data without a valid retained copy. This closes the intended relocation convention for this command; it differs from the separate older installer.
- removeBootCode always reads block one and writes it to block zero if its first ULONG resembles DOS magic; its applicability depends on a previously relocated layout. No generic normal-media overwrite is asserted outside that convention. CHECK is explicitly unimplemented and prints that fact; do not count that as a covert verification success.

Fixed 512-byte reads/writes assume the allocated block size is at least 512; a smaller supplied de_SizeBlock creates a buffer-length mismatch, while non-512 logical media may reject the requests. Device-environment acceptance must bound that malformed/unsupported geometry path. No bootblock was read or written during this source-only review. Final normalized grouping of installer candidates remains open.

### Capability-selection reconciliation checkpoint

Revalidated default command initialization and ordinary/64-bit ATA offset handling. Promoted unsuccessful required negotiation as DISK-263 and linked the installer arithmetic manifestations to DISK-060, separating two root causes with different fixes. Existing address-sum truncation remains DISK-059-related evidence requiring precise grouping; do not count every copied helper independently.

### Integration coverage and normalization checkpoint

Updated the top coverage table to reflect completed consumer reads rather than leaving the earlier discovered format/install inventory as wholly unread. Numbered two formatter root causes with concrete control/lifetime traces. Other unnumbered consumer observations remain retained with their confidence/trigger limits and still need final grouping; this milestone is not a completed subsystem audit.

### Trackdisk retry and legacy preference build classification

Correction to the earlier retry classification: both current trackdisk implementations consume `tdu_RetryCnt`. The generic PC driver initializes signed `int` read/write, seek and format retry counters directly from the public `UBYTE` and uses decrementing `do` loops. A zero counter therefore becomes negative after the first failed attempt and takes an extremely long path toward signed overflow, rather than ending the retry. IPrefs accepts an unchecked stored `ULONG` and narrows it into that byte, so a manually edited or malformed preset containing zero (or a multiple of 256) can reach this path; the shipped GUI slider selects only 1–10. This source-only cross-check is numbered GD2339 in the graphics/desktop report. The m68k-Amiga driver's retry expression adds one before multiplying, so the zero-value loop claim is limited to the generic PC driver. Read the legacy makefile completely: it targets a non-AROS libabox/prefs_common build, while the current mmake explicitly compiles locale and trackdisk_prefs. Its lowercase install prerequisite and uppercase output differ on case-sensitive hosts, but this is an optional legacy recipe issue, not an active modern target failure.

### Current remaining review gates

Historical pending notes above describe the state at their individual checkpoints. Later reads completed the AHCI headers, active Gayle build inventory, full MBR handlers, Partition/HDTool/BHFormat consumers, both GRUB installer sources, C:Install, and trackdisk preference producer/consumer sources. Those earlier pending labels are superseded by their later coverage records.

The audit remains active. Remaining gates include checking the complete consumer inventory for additional partition/format/install utilities; finishing classification of the legacy generated disk-image header forest and external declaration boundaries; normalizing retained unnumbered observations with caller contracts; deduplicating findings across shared helpers; and reconciling the final component coverage table and findings index. Numbered entries are source-review candidates, not a count of independently reproduced defects.

### Additional consumer inventory reconciliation

A repository-wide filename inventory found previously unrecorded active consumers: workbench/tools/QuickPart, workbench/tools/HDToolBox, and workbench/tools/InstallAROS. Their full source/build reviews remain required. Also identified rom/filesys/SFS/SFSformat/SFSformat.c and platform boot installers arch/ppc-sam440/boot/parthenope/tools/parthenope-install.c, arch/m68k-amiga/boot/floppy/install.c, and arch/i386-pc/boot/floppy/install.c for build-selection and direct-media interface review. Filesystem implementation internals remain distinct from these direct formatting/install consumers.

QuickPart mmakefile.src was read completely: the active executable lists fourteen translation units. QP_FindDisks.c and QP_ccDisk.c were read completely in this pass; the other twelve units and supporting headers remain pending. Storage enumeration records a drive even if OpenRootPartition fails, retaining cleared zero size. No source was built or executed. The worktree remains clean at the target commit.

### QuickPart operation/container contract review

Read QP_ccOperation.c, QP_ccOpGrp.c and QP_ccPartitionContainer.c completely, bringing this pass to five of fourteen executable translation units. Operation objects currently represent and rearrange GUI objects; these files contain no partition-table publication. QP_ccDisk's SaveDisk dispatch returns zero without writing. Do not describe visual operation staging as a proven media mutation. Re-read OpenPartitionTable/ClosePartitionTable and the MBR free-handler contract: opening is not reference-counted and closing frees child handles recursively. QuickPart retains child handles in GUI objects after FindParts closes their owning table; nested container refresh/repeated scan paths need further caller tracing before normalizing their possible stale-handle use. Also retain the repeated-open overwrite/leak observation for DISK-273; do not assume nested OpenPartitionTable calls merely acquire a shared reference.

### QuickPart selection and ownership follow-up

Read QP_ccApp.c and QP_ccPartition.c completely, bringing executable coverage to seven of fourteen translation units. Selection obtains cached partition type, DOS environment, position and descriptive strings; DeleteActive stages GUI objects and free-space nodes without calling DeletePartition or WritePartitionTable. Retained partition-handle pointers alone therefore do not prove a use-after-free during ordinary selection/deletion. Nested container refresh remains the relevant ownership-sensitive path.

Rejected a current AROS lock-leak claim based solely on QP_ccPartition's LockDosList(LDF_DEVICES|LDF_READ) / UnLockDosList(LDF_VOLUMES|LDF_READ) mismatch: full reads of rom/dos/lockdoslist.c and unlockdoslist.c show that both category masks obtain/release the same di_DevLock through LDF_ALL. The API documents matching flags, so portability remains a contract observation, but no different semaphore is released in this implementation. This does not affect DISK-271, whose path skips unlocking entirely.

Selection's incremental snprintf uses would-have-written lengths without clamping the accumulated offset before subsequent calls, and UUID_Unparse lacks a remaining-space bound. Candidate overflow requires combined generated labels/device names/localized text large enough to exhaust partDesc[256]; remaining string-definition/header reads must bound the normal cases. Partition lookup also uses a prefix device-name comparison rather than equality and treats arbitrary nonzero startup BPTRs as startup structures after IsFileSystem; retain for caller-contract classification. No media mutation is inferred.

### QuickPart complete local source/build coverage

Completed QP_Main.c, QP_ccFree.c, QP_ccTxt.c, QP_Gui.c (comment-only), QP_globals.c, QP_locale.c and QP_PartitionColors.c, plus every local .h file. All fourteen translation units listed by mmakefile.src are now source-read. Generated locale strings/catalog translations are data, not independently reviewed storage algorithms. QP__SizeName extends through YB and bounds the UQUAD scaling loop; reject an ordinary 64-bit capacity suffix-array overflow claim. Ordinary type labels are fixed short strings; selection text overflow remains conditional on aggregate text lengths/translation/device-name contracts, not asserted for the short default labels alone.

Startup has no active Apply/Undo publication callbacks, Create stays disabled, and the SaveDisk method is a stub. This is an explicit incomplete editing feature: staged GUI deletion cannot be committed by these paths. Do not count absence of an implemented writer as proof of silent on-media deletion. Main calls FindDisks once; no repeated ordinary user rescan was located. The retained stale child partition handles remain an ownership hazard for externally invoked refresh methods, not a proven use-after-free on ordinary cached selection. Parent FindParts recurses only into the final newly created partition's contents, leaving earlier nested containers unscanned; retain as a layout coverage limitation.

### HDToolBox initial source/build and save-contract coverage

Read the current mmakefile.src completely: ten translation units (main, devices, gui, harddisks, hdtoolbox_support, locale, partitions, partitiontables, ptclass, prefs); debug.c is not in this active list. Completed harddisks.c, partitions.c, partitiontables.c and hdtoolbox_support.c, and read gui.c save/add-partition/low-cylinder callback sections. Six translation units and the remainder of gui.c are still pending; headers and legacy recipe also remain pending.

The saveChanges function recursively visits children and only unsetChanged after a successful WritePartitionTable. A failed write reports MSG_WriteTableError and retains changed state; reject a generic claim that every failed HDToolBox write is silently marked saved. Its recursion continues to parent writes after child failure, which still needs publication-order classification. Disabled #if 0 mounting helpers and their #error directive do not establish active build failures. harddisks.c zero-initializes SCSICmd and uses a six-byte INQUIRY CDB, avoiding DISK-254's unchecked sense-pointer preparation in this caller.

### HDToolBox resize validation caller trace

Read gui.c:1–300, 420–530 and 580–790, together with previously read save/add/low-cylinder sections, and the resize input construction at 2034–2077. Full GUI source review remains incomplete. Re-read the MBR PT_DOSENVEC setter: it copies the supplied environment and encodes the entry without checking disjointness. Resize OK forwards intermediate environments to SetPartitionAttrsA and marks the parent table for later save. This provides a concrete publication trace for the following candidate.

### HDToolBox settings/deletion review checkpoint

Completed gui.c:790–1200 (DOS environment, mount flags, deletion and selection commands), and targeted resize notifications and type-editor construction. The resize button is wired by lv_doubleclick to open the resize window and populate its partition class, despite the empty GB_RESIZE_MOVE switch branch; DISK-278 therefore is not rejected as an unwired feature. Full GUI read still has intervening selection/mountlist and later construction/notification sections pending.

### HDToolBox mountlist and selection coverage

Completed gui.c:1200–1800, including mountlist export, list ownership helpers, keyboard dispatch and both selection hooks; harddisks.h and partitions.h were read completely to verify object layouts. Ordinary nested-partition deletion is reachable: lv_click enables Remove for LNT_Partition regardless of whether it contains a table, supporting DISK-279's callback trigger. GUI construction/notifications after 1800 and remaining translation units/headers remain required.

### HDToolBox GUI and preference-parser completion

Completed gui.c:1800–end and re-read 530–580 to close the interval gap. The full GUI source is now read, including all resize acknowledgment, mountlist menu and deletion callback wiring. Also read main.c, devices.c, prefs.c and prefs.h completely. Eight of the ten active translation units are complete; ptclass.c, locale.c, remaining headers and the legacy recipe still need review. Main unwinds via uninitEnv even after initialization failures, limiting generic leaked-library claims; it nevertheless returns zero after initialization errors. Device nodes allocate a parent list marker but freeDeviceNode omits UninitListNode for that marker. Preference table/type allocations and icon resources also lack a comprehensive teardown; exact lifetime/resource impact remains grouped for final normalization.

### HDToolBox partition-class complete source read

Read ptclass.c and locale.c completely, completing all ten active translation units. The partition class's drag path does use getBetterDiff to examine full moved intervals, so DISK-278's endpoint-only text resize example must not be generalized to claim that dragging entirely lacks interval checks. getBetterDiff nevertheless mixes inclusive end values with exclusive overlap comparisons and recursively adjusts diffs; malformed/overlapping geometry and native-width arithmetic need bounded follow-up. findSpace, getActive, rendering and mouse conversion all use ULONG block products; zero sectors-per-cylinder, zero drawing cylinder span and tiny gadget widths are unchecked divisors but target/UI construction constraints remain part of their classification. Remaining HDToolBox coverage gates are other local headers, legacy Makefile.gcc, preference data/contracts and final normalization.

### HDToolBox local inventory closure and report navigation

Read every local header, legacy Makefile.gcc, shipped Env-Archive/hdtoolbox.prefs, BUGS/README and legacy-only debug.c. Modern build selection is ten units; the standalone m68k AmigaOS recipe adds debug.c. Its /ram output pathname is legacy recipe data, not a path used for this review. Shipped preferences use short decoded identifiers (MBR one byte, RDB four, GPT UUID parsing); these do not themselves trigger DISK-282. User-edited identifiers remain its concrete trigger. Historical BUGS entries provide context only and are not accepted as reproduced defects.

Updated the front coverage reconciliation and component navigation for DISK-271–283. InstallAROS mmakefile.src was read completely: eleven active C translation units; bootloader macros choose GRUB=1 or GRUB=2 and explicitly include grub2gfx in the latter. Reject an obsolete missing-grub2gfx-selection claim for this baseline. Its source/header review is the next consumer gate. Final normalization remains outstanding for both completed local consumer inventories and the broader subsystem.

### InstallAROS disk-helper complete read and overwrite trace

Read ia_diskio.c (838 lines) and ia_diskio.h completely, plus ia_stageclass.c:1850–2005 to trace the backup consumer. One of eleven active translation units is complete; the stage file is only partially read. Helpers include filesystem-format invocation, DOS-device lookup, partition discovery, recursive file enumeration/copy and backup preparation. FormatPartition uses Inhibit, Format and uninhibit; it is not a raw-sector formatter in this file.

getDiskFSSM repeats the fixed dname[32]/UBYTE unbounded-prefix pattern already observed in the boot installers; retain InstallAROS as an additional caller for shared root-cause normalization. GetDevNameForVolume reads a DOS entry after releasing the volume lock and returns a borrowed device-name pointer after unlocking, requiring concurrent-list lifetime classification. Its category-mask unlock mismatch alone does not establish a current AROS semaphore leak, as verified earlier. CheckPartition and recursive-copy helpers have unchecked allocation consumers; FindPartition leaks earlier rejected-name allocations when overwriting its local name variable. Enumeration breaks on ExAll errors yet returns an ordinary partial count; stage-level propagation remains to be reviewed.

### InstallAROS stage progression and partition-command coverage

Read ia_stageclass.c:1–870 and 900–1210; the cancel-function prefix at 889–899 and the remaining stage ranges are still pending. Source includes startup allocation, option caching, next/previous stage transitions, cancellation continuation and full MUIM_Partition command construction. The partition stage checks SystemTagList's returned command status and reports partitioning failure; do not claim that it universally ignores a nonzero C:Partition exit. However, earlier C:Partition persistence error findings still limit how much a zero exit proves.

### InstallAROS complete stage-source coverage and failure gates

Completed ia_stageclass.c through its dispatcher, including the cancel-prefix gap, directory-copy final condition, locale preferences, installation, formatting, file-copy tail, undo and reboot methods. Together with ia_diskio.c, two of eleven active translation units are now complete. Read ia_stage_intern.h and the actual ForeachNode/ForeachNodeSafe macros; main's identified constructor explicitly sets EnableUndo=TRUE, limiting the uninitialized-undo-list observation to alternate constructors/configurations that disable it. The stored UndoSteps method removes/frees its current node under a non-safe ForeachNode traversal, but no ordinary source caller was located beyond its dispatcher; retain as an exposed-method lifetime candidate with that applicability limit.

### InstallAROS bootloader integration complete source read

Read ia_bootloader.c (425 lines) and ia_bootloader.h completely, bringing active translation-unit coverage to three of eleven. GRUB implementations are enabled on x86 only; ARM bootloader support is commented out. BOOTLOADER_NONE normally disables/unselects the main bootloader option (ia_main.c:639–640), limiting the uninitialized tmp/default-switch execution observation to alternate invocation/state corruption. Re-read rom/dos/execute.c: although its documentation describes command-start success, this baseline implementation returns FALSE for nonzero SystemTagList result. The concrete issue below is that the caller ignores the result entirely, not an assumed universally misleading Execute implementation.

### InstallAROS drive-selection source closure

Read ia_driveselectclass.c completely, including storage enumeration, existing-partition selection, startup-message lookup, popup application, unit display and option-cache updates. Four of eleven active translation units are now complete (diskio, stage, bootloader, drive selection). Reject a normal-path unbounded-string claim for Initialize's devnamebuffer: getDiskFSSM stops at the appended colon before the unterminated tail; the debug-only string print is a separate configuration limitation. Storage enumeration can overwrite a previously allocated partition-name result while continuing through other buses/controllers; resource ownership and attribute-get failure handling remain conditional observations for final grouping. No application, build or disk operation was executed.

### InstallAROS complete local source/header coverage

Read the remaining seven units completely: ia_installclass.c (131), ia_optionclass.c (162), ia_volumeclass.c (349), ia_packages.c (44), ia_env.c (104), ia_locale.c (55), and ia_main.c (1203). Every local header has now been read, including the configuration, install/option/stage, drive-selection, volume, package, locale and bootloader interfaces. All eleven active translation units are covered. Translation catalogs and icon data do not add disk algorithms; locale lookup was reviewed as code, without asserting translation-content review.

Tracing option ownership shows that the option list contains embedded object nodes, while GUI groups own option objects; failed constructions can leave stale nodes, but the ordinary main failure paths may exit before traversing them. Retain this as conditional lifecycle evidence, not an unconditional normal-run dangling-list claim. Popup-only filesystem/size controls have no owner until their window is constructed; unused popups can leak these objects. Main does not dispose InstallObj, does not delete volMcc and omits several path/options/library releases. MUI_DeleteCustomClass explicitly refuses classes with live instances, so reject a claim that this final call unconditionally frees a live class and causes immediate use-after-free: the supported conclusion is leaked instances/classes/resources. Allocation-failure paths in main and stage also contain unchecked results; group these by failed initialization rather than inventing a routine disk-write outcome.

The package install functions are stubs, with only a fixed core skip path implemented. The environment helpers use a fixed pathname buffer. Main enables undo explicitly, so the zero-initialized undo-list issue when the optional constructor flag is false is an alternate-caller limitation rather than the ordinary application's initialization path. The optional post-install script is externally supplied: no tracked InstallAROS-Post-Install file was found; its possible writes are not asserted. The code launches it without checking the failed/cancelled stage state and deletes backup records at the end irrespective of that state. UndoSteps frees nodes during a non-safe iteration, but no ordinary UI invocation was established. These observations remain explicit conditional review outcomes.

### Additional formatter and platform boot-installer source pass

Read SFSformat.c (125 lines), its build and usage text completely, tracing ACTION_SFS_FORMAT to the filesystem packet dispatch to confirm the tag-pointer contract. The text's Return-to-confirm instruction differs from the implementation's explicit y/Y test; record as a documentation discrepancy, not media corruption. Borrowed handler-port lifetime after unlocking remains conditional on concurrent device removal. Its tag array has room for the name, three switches and terminator, so reject a tag-array overflow claim.

Read arch/m68k-amiga/boot/floppy/install.c (101 lines) completely and its host-tool build rule. Read parthenope/tools/parthenope-install.c (273 lines), src/rdb.h, Makefile.am/tools selection and configure.ac completely. Parthenope's modern src/mmakefile builds its firmware source list, not this external tools installer; the tools installer is selected by the standalone autotools build. Its issues are therefore classified as standalone/legacy configurations, not defects reached by every modern AROS install. Parthenope's firmware RDB reader remains a separate dependency gate. No host utility was built or executed and no image or disk was modified.

### Parthenope RDB integration and firmware callback closure

Read src/rdb.c (207 lines), support.c (190), context.c (230), ext2.c (102), and the full rdb/support/context/uboot headers. The copied checksum-count error is grouped under DISK-296 rather than counted twice. Firmware rdb.c uses block_read on the selected device descriptor; block_read has a block-count result, and get_rdb requires one completed block. get_partition_info only tests nonzero for its one-block request; with the declared block-count contract a legitimate one-block success is sufficient, so do not infer ordinary short success here. Capacity start/length products remain unsigned 32-bit, with no comparison to descriptor lba; malformed geometry can wrap. info->size subtracts one from a quantity explicitly documented as a block count; this underreports the length by one block to the external ext2 callback. The callback implementation is external, so the concrete conclusion is wrong advertised capacity, with precise filesystem tail-read effects conditional on it.

RDB geometry and checksum parsing assumes at least 512-byte backing blocks. get_rdb's device-block allocation is read as a 512-byte checksum object; block sizes below 512 would overread, whereas ordinary 512-byte devices avoid that issue. Linked-list cycles, malformed pointers and checksum failures are not bounded by a visited set. get_dev caches the first suitable hard disk (or current firmware-selected unit), ignoring its numeric dev argument; callers only request zero. Source-only, no firmware execution. Filesystem algorithms in dos.c/sfs.c/cdrom.c are outside the primary disk-driver scope; their sector-callback call sites were traced for partition-start and block-size contracts, without claiming complete filesystem review. General menu/ELF/network/firmware code is likewise outside scope.

### Latest coverage reconciliation and remaining final audit

This checkpoint supersedes the earlier consumer gates: QuickPart, HDToolBox, InstallAROS, SFSformat, both PC GRUB installers, C:Install, Partition, HDTool, BHFormat, trackdisk preference producer/consumer, m68k host boot installer and Parthenope RDB/standalone installer have explicit source/build outcomes. The primary driver implementations, local parsers, transport glue, platform selections and bundled wrapper logic have complete-read records. General filesystem implementations, external compression/crypto libraries, unrelated generated API functions and firmware callback implementations remain explicit scope boundaries rather than alleged locally reviewed code.

Re-ran the repository-local disk/ATA/AHCI/NVMe/VirtIO/SCSI/SD/mass-storage build/configuration filename inventory across rom, arch and workbench. Its listed modules correspond to the component inventories above, including the separate m68k disk.resource and nonvolatile nvdisk exclusion. Datatype, PCI-tool and ACPI-tool hits are unrelated catalog paths, not undiscovered block devices. The source tree remains clean at the baseline commit.

Final work still remains: reconcile all numbered candidates with their later corrections and configuration limits; group the remaining unnumbered observations without inflating root-cause counts; verify navigation/anchors and source references; and rewrite the front coverage/status section to reflect the final bounded audit. In particular, DISK-283 is withdrawn rather than a surviving candidate, DISK-022 is an alias, and copied checksum implementations are grouped as DISK-296. No claim that every candidate is independently confirmed or reproduced is made.

### Final classification pass: identification, argument omission and retained lifecycle observations

Rechecked VirtIO's Enumerator: product IDs in 0x1040–0x107f are filtered by subsystem ID 2 rather than by the modern block product 0x1042. Compared the [OASIS VIRTIO 1.2 PCI discovery contract](https://docs.oasis-open.org/virtio/virtio/v1.2/virtio-v1.2.html#x1-1030002): modern device type is encoded in the product ID, while subsystem IDs can reflect the environment and non-transitional devices are advised to use values at least 0x40. The identification observation is now DISK-303. This is protocol reference verification, not project execution.

Rechecked HDTool's command and handlers against rom/dos/readargs.c: absent optional /F arguments stay NULL because argbuf is cleared, ITEM_NOTHING ends parsing and only supplied argbuf entries update the caller array. This closes its missing-argument contract as DISK-304. No CLI command was executed.

The PPC byte-reversed store helpers remain conditional on GNU/PPC macro selection and compiler memory accounting; arch/ppc-all/include/aros/cpu.h defines __PPC__, and the standalone powerpc-aros recipe selects GCC with -O3. Their exact generated machine code is intentionally untested. Promote the missing output-memory description as DISK-305, preserving that limitation. Mapping cleanup, explicit USB suspend quiescence and Reaction hook allocation are normalized as DISK-306–308. ATA native-sector compatibility is DISK-309, bounded to actual logical sectors larger than 512 bytes rather than 512e physical sectors.

VirtIO used-ring len is a defensive response-accounting observation: a successful short device-written read response is not validated before reporting the requested byte count. For a write, used.len describes guest buffers written by the device (including status), not media bytes written, so a small used.len is not proof of a short media write. Do not promote that interpretation. Stable conforming devices can mask the read validation omission; retain it with this limit instead of claiming routine loss on every write.

### Final protocol/transfer-accounting reconciliation

Compared compiler/include/hardware/sdhc.h with Broadcom's [BCM2835 ARM Peripherals register definitions](https://www.raspberrypi.com/wp-content/uploads/2012/02/BCM2835-ARM-Peripherals.pdf), INTERRUPT/IRPT_MASK/IRPT_EN, printed pages 77–81. DATA_DONE is bit 1, whereas bit 2 is BLOCK_GAP; the code's SDHCI_INT_DATA_END is bit 2. Screenshot retrieval failed, so this comparison uses the provider's extracted text, not a claimed visual inspection. Reread BusIRQ and the entire WaitCmd completion tail: the unconditional remaining-listener recovery calls FinishData after polling. Therefore this macro error does not prove that ordinary DMA always omits bounce copy/cache cleanup; the fallback can compensate. DISK-310 preserves the literal mask/dispatch error with that important limit.

Rechecked MUIM_IC_CopyFile and the local DOS Write contract (rom/dos/write.c): nonnegative short results are permitted and describe actual byte counts. InstallAROS checks only -1 and subsequently advances input. Its Short-I/O/Close failure observation is normalized as DISK-311, distinct from DISK-284's acceptance when opening a backup fails. GetStartCyl's division remainder is confirmed at gui.c:1234; mountlist publication uses the rounded quotient. DISK-312 retains the mixed-geometry trigger rather than claiming every export is wrong.

Rechecked disk-image image-name allocation and NameFromFH against the local provider. NameFromFH returns FALSE without writing the caller buffer if its FileInfoBlock allocation fails, and InsertDisk does not gate successful plugin opening/persistence on that result. These close the previously conditional helper observations as DISK-313/314. Their memory-pressure/error prerequisites remain explicit; no allocation failure or media operation was induced.

### Final root-cause reconciliation ledger

Merged repeated confirmations DISK-305 into DISK-244 (PPC endian stores), DISK-310 into DISK-233 (SDHCI completion-bit mask), DISK-313 into DISK-245 (image-name allocation result), and DISK-314 into DISK-246 (failed NameFromFH persistence). They do not represent additional bugs. Updated DISK-233 itself with the verified Broadcom register contract and WaitCmd's compensating data-listener recovery; the earlier incomplete explanation must not be read as proving universal DMA data loss/hang. DISK-022 remains merged into DISK-006, and DISK-283 remains withdrawn. The final ledger supersedes earlier historical numeric counts.

Grouped unbounded device-prefix copies under DISK-292 across InstallAROS, C:Install and both GRUB helper implementations; grouped LDF_READ-only missing list locks under DISK-293 across InstallAROS and the GRUB helpers. C:Install supplies LDF_DEVICES, so it is explicitly excluded from that missing-lock manifestation. Address narrowing, negotiation-threshold overflow and unchecked failed 64-bit negotiation remain distinct DISK-059/060/263 because their triggers and required corrections differ. Wrong mountlist unit, released icon path and dropped parent-offset remainder remain separate DISK-281/280/312. Existing C:Install boot-preservation-write loss is already DISK-262; no duplicate number was added.

Retained observations not assigned an additional bug ID are explicitly limited: startup resource leaks and unimplemented ReleaseID/QuickPart publication; externally invoked stale-handle refresh or reentrant callbacks; unsupported native geometry/configuration values; external plugin test sizes and library/compiler ABI assumptions; old header guards/build recipes; helper functions with no active caller; and raw-read/flush/parser defensive omissions already discussed with their metadata/provider prerequisites. These remain review outcomes in the coverage log and related finding notes. They are not silently promoted to ordinary-path, confirmed independent defects.
### DISK-324 — SCSI device never deletes its memory pool

- **Severity / confidence:** Low (retains the pool allocation for the lifetime of the system after any successful initialization); high source confidence.
- **Source:** `rom/devs/scsi/scsi_init.c:204-218,274-323`; `rom/devs/scsi/scsi_unitclass.c:78,113`; `rom/devs/scsi/mmakefile.src:20-35`; `rom/devs/scsi/scsi.conf:1-10`.
- **Trigger / trace:** A successful `SCSI_init` creates `scsi_MemPool` with `CreatePool`. Unit construction allocates each `DriveIdent` from that pool and unit disposal frees the records, but no code in the selected SCSI implementation calls `DeletePool`. The registered `scsi_expunge` hook releases attribute bases and closes Utility, then returns success without deleting the pool. Thus final device expunge leaves the pool's backing memory allocated; any failed initialization after pool creation also returns through generated rollback to the same hook without deleting it.
- **Impact / limits:** Permanent memory leak per initialized SCSI device lifetime, with pool metadata/backing chunks retained after expunge. The pool itself may remain useful while initialized units exist, but the successful expunge path has already removed every controller before returning. The separate case-sensitive include/build issue is DISK-138; this finding describes the lifetime defect if the module is built and loaded.
- **Evidence limit:** Source-only; no build, load, or expunge was run.

### DISK-325 — ATA reports TRIM success without running the operation

- **Severity / confidence:** Medium (silent discard of a deallocation request and false capability-query result); high source confidence.
- **Source:** `rom/devs/ata/ata.c:585-610,632-664,707-710,744-750`; `rom/devs/ata/ata_trim.c:20-39`; selected caller `workbench/tools/SysExplorer/Modules/Storage/storage_enum.c:293-310`.
- **Trigger / trace:** `HandleIO` initializes `io_Error` to zero, then the command map dispatches `HD_TRIMCMD`; the command is also listed in `NSDSupported`. When the drive reports ATA version 7 or later and the TRIM support bit, `cmd_TRIM` enters its supported branch, but the entire availability response and `ata_TRIMCmd` call are inside `#if (0)`. Thus an actual TRIM request returns success without issuing a device command. The supported-command probe has the same defect: SysExplorer passes `ATAFEATURE_TEST_AVAIL` and expects `TRIM_MAGIC_ID` plus an actual length, but the handler leaves both untouched, so the tool fails to recognize TRIM support.
- **Impact / limits:** Callers can discard their only indication that deallocation was skipped and assume the device accepted TRIM. This applies only to a TRIM-capable ATA device and requests routed through `ata.device`; unsupported drives return `IOERR_NOCMD`. The separate SCSI-device command map does not dispatch SMART/TRIM and is not part of this finding.
- **Evidence limit:** Source-only; no drive, TRIM request, or SysExplorer query was run.

### DISK-326 — ATA advertises SMART but rejects the command

- **Severity / confidence:** Low (SMART telemetry and capability detection are unavailable despite the advertised command); high source confidence.
- **Source:** `rom/devs/ata/ata.c:507-544,546-583,662-700,707-727`; helper `rom/devs/ata/ata_smart.c:20-44`; selected caller `workbench/tools/SysExplorer/Modules/Storage/storage_enum.c:271-290`.
- **Trigger / trace:** Query the selected ATA device with `NSCMD_DEVICEQUERY`: its `SupportedCommands` points at `NSDSupported`, which includes `HD_SMARTCMD`. Then issue a SMART request on an open unit with media present, including the `ATAFEATURE_TEST_AVAIL` probe used by SysExplorer. `HandleIO` initializes `io_Error` to zero (`ata.c:707-710`). The helper `ValidSMARTCmd` has its parameter and command validation compiled out and therefore returns TRUE unconditionally (`:507-544`). The SMART capability response and call to `ata_SMARTCmd` are also inside `#if (0)`; execution falls through to `io_Error = IOERR_NOCMD` (`:561-582`). `ata_SMARTCmd` itself contains no active ATA operation (`ata_smart.c:20-44`). SysExplorer only reports SMART support when the probe succeeds and returns `SMART_MAGIC_ID`, so its selected probe cannot identify SMART support on ATA units (`storage_enum.c:271-290`).
- **Impact / limits:** Clients that rely on the advertised command list may attempt SMART queries and receive `IOERR_NOCMD`; the selected SysExplorer omits its SMART-supported label. This applies when ATA is selected and the unit has media present. A unit without media returns `IOERR_OPENFAIL` earlier. Implement the advertised SMART operations/probe or remove `HD_SMARTCMD` from the supported-command list until it is implemented.
- **Evidence limit:** Source-only; no device query, SMART request, or SysExplorer scan was run.
### DISK-327 — USB MassStorage worker faults when its unit allocation fails and leaves construction waiting

- **Severity / confidence:** Medium (a memory-pressure failure can fault the worker and stall USB device construction); high source confidence.
- **Source:** `workbench/devs/USB/classes/MassStorage/storage_task.c:39-57,82-85,167-170`; `workbench/devs/USB/classes/MassStorage/storageclass.c:314-322`.
- **Trigger / trace:** On a selected Workbench USB MassStorage path, let the worker's `AllocVecPooled` for `mss_unit_t` fail. `CreateUnit` immediately writes `unit->msu_unit.unit_MsgPort.mp_SigBit` without checking the returned pointer (`storage_task.c:42-46`), so the worker dereferences NULL before initialization completes. Its only parent-ready `Signal(parent, SIGF_SINGLE)` is inside the later `if (unit)` branch (`:82-85,167-170`). The constructor starts this task and waits unconditionally for that signal (`storageclass.c:314-322`); there is no failure notification or timeout, so absent a separately pending signal the constructor remains blocked after the worker faults. Check the unit allocation and report startup failure to the parent on every exit path.
- **Affected configuration and trigger:** Workbench USB MassStorage builds selected through the `hidd-usb` class aggregate when a storage device has bulk IN/OUT endpoints and unit-pool allocation fails during worker startup (`workbench/devs/USB/classes/MassStorage/mmakefile.src:5-11`; `workbench/devs/USB/classes/mmakefile.src:2-5`). No USB device or allocation failure was exercised. Source-only.



### DISK-328 — USB MassStorage dereferences invalid HD_SCSICMD data before checking it

- **Severity / confidence:** Medium (invalid passthrough request can fault the unit task); high source confidence.
- **Source:** `workbench/devs/USB/classes/MassStorage/storage_device.c:357-380`.
- **Trigger / trace:** Send `HD_SCSICMD` with a null `io_Data`, or with a `SCSICmd` whose `scsi_CmdLength` is nonzero but `scsi_Command` is null or inaccessible. `cmd_DirectSCSI` casts `io_Data` to `cmd`, then its debug-print loop reads `cmd->scsi_CmdLength` and `cmd->scsi_Command[i]` before entering the later `if (cmd)` check. Even when `cmd` is non-null, the loop dereferences the command pointer before any validation. Since this unit is built with `DEBUG 1`, the `D(bug(...))` trace is enabled for the selected source configuration. Validate the structure, command pointer and length before logging or dispatching.
- **Impact / limits:** A malformed passthrough request can fault the task handling that unit's I/O instead of returning an argument error. The finding concerns invalid `HD_SCSICMD` input; callers that provide a readable `SCSICmd` and CDB are not implicated. Source-only; no request was issued.


### DISK-329 — USB MassStorage writes a full device-query result into a short buffer

- **Severity / confidence:** Medium (query can write beyond the caller-declared buffer); high source confidence.
- **Source:** `workbench/devs/USB/classes/MassStorage/storage_device.c:527-536`.
- **Trigger / trace:** Issue `NSCMD_DEVICEQUERY` with `io_Length` shorter than the size needed through `SupportedCommands`, as reported by peer device implementations. `HandleIO` casts `io_Data` to `NSDeviceQueryResult` and writes every field including the trailing command-list pointer without checking `io_Length`; it then sets `io_Actual` to the full structure size. The selected input and RAM disk implementations reject a too-short query with `IOERR_BADLENGTH` before writing (`rom/devs/input/input.c:126-145`; `workbench/devs/ramdrive_device.c:265-284`). Validate the request length and report `IOERR_BADLENGTH` before populating the result.
- **Impact / limits:** The method can write fields past the caller-declared query buffer and report more bytes than the request allows. This applies only to short `NSCMD_DEVICEQUERY` buffers; normal callers that supply the complete result structure are unaffected. No request was sent. Source-only.

### DISK-330 — ISO high-offset reads wrap the raw-sector seek and return earlier data

- **Severity / confidence:** Medium (an out-of-range high-offset read can return unrelated in-image data as success); high source confidence.
- **Source:** `workbench/devs/diskimage/plugins/iso.c:154-195`; selected command path `workbench/devs/diskimage/device/unit.c:341-349,535-537`.
- **Trigger / trace:** On a built-in 2352-byte-sector ISO image with at least 637,536,272 bytes, submit `TD_READ64` at byte offset `0x100000000` for 2048 bytes. `ISO_Read()` reconstructs the 64-bit logical byte offset from `io_Offset` and `io_Actual` (`:163`), converts it to a 2048-byte block (`:171`), then computes `start_blk * image->sector_size + image->header_part` without widening the `ULONG` operands or checking the requested range against `image->total_blocks` (`:174`). With 32-bit `ULONG`, the expected backing offset `0x126000010` wraps to `0x26000010`; a 1 GiB image contains that wrapped location. `ChangeFilePosition()` therefore seeks inside the file, `FRead()` obtains 2048 bytes, and the plugin returns success with a full actual count. The device's `TD_READ64` path forwards the request to the plugin without a capacity check (`unit.c:341-349,535-537`). This is distinct from DISK-037's Generic plugin passing an untruncated 64-bit offset into the DOS fallback seek.
- **Impact / limits:** An out-of-range high-LBA request is accepted and returns data from an unrelated lower sector. This does not establish incorrect data for in-range requests or a caller-buffer overrun. Widen the raw-offset arithmetic and reject requests whose block range exceeds the opened image capacity.
- **Evidence limit:** Source-only; no image, request, or runtime seek was used.


### DISK-331 — DAA constructor leaks the source file when its first file record allocation fails

- **Severity / confidence:** Low (one DOS file handle remains open after a failed insert); high source confidence.
- **Source:** `workbench/devs/diskimage/plugins/daa.c:229-240,375-384,398-415`; caller `workbench/devs/diskimage/device/unit.c:557-576`.
- **Trigger / trace:** `InsertDisk()` opens the image and sets `unit->Plugin` before calling `Plugin_OpenImage()`. If allocation of the `DAAImage` succeeds but allocation of the first `daa_file_t` fails, `DAA_OpenImage()` enters its error path with a non-NULL image and calls `Plugin_CloseImage()`. At this point `image->files` is NULL, so `DAA_CloseImage()`'s file-list loop does not close the original `file`. `InsertDisk()` closes the handle only when `unit->Plugin` is NULL; after the plugin returns a NULL image, `RemoveDisk()` also skips `Plugin_CloseImage()` because `unit->ImageData` is NULL. No owner remains to close the handle.
- **Impact / limits:** The failed DAA insertion leaks the opened source-file handle. This requires the specific low-memory failure after image allocation and applies to the separately enabled DAA target; the normal plugin aggregate excludes DAA. Close `file` directly when no file-list node owns it, or link ownership before any later failure.
- **Evidence limit:** Source-only; no allocation failure or image insertion was run.


### DISK-332 — DAA total-byte multiplication wraps before widening

- **Severity / confidence:** Medium (reported geometry and read limit can be much smaller than the image); high source confidence.
- **Source:** `workbench/devs/diskimage/plugins/daa.c:182,296-301,419-427,446-449`.
- **Trigger / trace:** `daas` and `image->out_size` are 32-bit `ULONG`s. Their product is evaluated before assignment to 64-bit `image->total_bytes`, so it wraps at 4 GiB on the selected 32-bit m68k target. For example, 65,536 chunks of 65,536 bytes wrap to zero. `total_blocks` and later `DAA_Read()` bounds are then derived from the wrapped capacity.
- **Impact / limits:** large DAA images can report a truncated or zero geometry and reject reads beyond the wrapped limit. Widen an operand before multiplying and validate the resulting sector count against the geometry field. This applies to the explicitly buildable optional m68k DAA plugin; the default aggregate excludes it.
- **Evidence limit:** Source-only; no large image, geometry query, read, or build was used.


### DISK-333 — DAA chunk-table allocation multiplication can underallocate

- **Severity / confidence:** Medium (conditional out-of-bounds table population for a very large size table); medium source confidence due to the large input and memory requirement.
- **Source:** `workbench/devs/diskimage/plugins/daa.c:182,296-320`.
- **Trigger / trace:** `daas` is derived from the on-disk size-table byte count, then multiplied by `sizeof(*image->data)` in 32-bit `ULONG` arithmetic for the allocation. There is no overflow check before the loop writes one `daa_data_t` per entry through `data++`. If the product wraps and the smaller allocation succeeds, the population loop writes beyond the allocated array.
- **Impact / limits:** an extreme DAA size table can produce an undersized metadata array followed by out-of-bounds writes during image opening. Check the multiplication before allocating. This is limited to the explicitly buildable optional m68k target; practical reachability depends on available memory for the large size table and allocation.
- **Evidence limit:** Source-only; no oversized image, allocation, or build was used.
