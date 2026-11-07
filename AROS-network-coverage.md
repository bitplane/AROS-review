# network file-by-file audit tracker

Source snapshot: `282a0454356dc2b9554ab023997d7f7ca24fc03e`. Inventory: 248 source/header files across the listed directories. Inventory is a starting boundary, not proof that every build-selected override is included. Each row remains **unreviewed** until its control flow, bounds, ownership, errors, and callers have been inspected. Existing findings were obtained by sampling and do not advance this tracker.

Progress: **1 reviewed**, **6 full-body reviews with broader callers pending**, **12 body-inspected with caller checks pending**, **228 unreviewed**. Counts are for the inventory below and will change as the audit advances.

| File | Status | Notes |
|---|---|---|
| `arch/all-mingw32/bsdsocket/accept.c` | Body inspected; Windows callers pending | Re-read the complete selected implementation and its native module manifest (`arch/all-mingw32/bsdsocket/mmakefile.src:3,68-72`): it only emits the not-implemented diagnostic and returns zero without consuming `s`, `addr` or `addrlen`. This is included in candidate 8 with the other hosted BSD socket stubs; the `accept`-dependent caller consequences are assessed at the grouped finding, while wider external Windows clients remain pending. No socket operation was run. |
| `arch/all-mingw32/bsdsocket/bind.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/bsdsocket_init.c` | Full selected body read; broader HostLib failure-unwind contracts pending | N13: a failed `sock_init` after `WSAStartup` bypasses `sock_shutdown`, so Winsock startup remains unbalanced. Caller chain and cleanup condition checked. |
| `arch/all-mingw32/bsdsocket/bsdsocket_intern.h` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/bsdsocket_open.c` | Full selected body read; custom Open/Close vectors, Exec library lifecycle, and initial descriptor-table allocation failure traced; wider task-exit and external client contracts pending | NW20: `BSDSocket_OpenLib()` ignores `SetDTableSize()` failure, publishes the zero-table task base, and returns it as successful Open; `socket()` then reports `EMFILE`. The same task's later opens reuse this base without retrying initialization. Exec `MakeLibrary()` clears task-base storage; pool growth can fail on the first real puddle. Custom vector mapping is selected through `noopenclose`; Open/Close entries are supplied by this source, not generated generic routines. No allocation failure or socket operation was induced. |

MinGW32 per-task descriptor-table initialization pass (NW20 added): read the full custom Open/Close vectors (`arch/all-mingw32/bsdsocket/bsdsocket_open.c:51-155`) and traced the OpenLib symbol selection through `noopenclose` generation (`tools/genmodule/writestart.c:1274-1284,1414-1434,1819-1823,1644-1649`). Confirmed Exec `MakeLibrary()` zeroes task-base storage (`rom/exec/makelibrary.c:108-130`); `CreatePool()` may leave its real puddle allocation until the first `AllocPooled()` (`createpool.c:113-129`); `SetDTableSize()` returns `ENOMEM` without changing its initial zero table fields (`bsdsocket_util.c:37-65`); and `BSDSocket_OpenLib()` ignores the result before tree publication/count increments. Traced `socket()` through `GetFreeFD()` to the zero-slot `EMFILE` result and later same-task OpenLib reuse (`socket.c:50-59`; `bsdsocket_util.c:68-80`; `bsdsocket_open.c:58-64`). The selected `SocketBaseTagList()` is a TODO stub, so that API offers no resize recovery (`socketbasetaglist.c:43-46`). Only these supporting slices of `bsdsocket_util.c` and `socket.c` were reviewed; both full bodies and all other clients remain unreviewed. No allocation or socket operation was performed.
| `arch/all-mingw32/bsdsocket/bsdsocket_util.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/bsdsocket_util.h` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/closesocket.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/connect.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/dup2socket.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/errno.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getdtablesize.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/gethostbyaddr.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/gethostbyname.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/gethostid.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/gethostname.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getnetbyaddr.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getnetbyname.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getpeername.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getprotobyname.c` | Full selected result replacement and cleanup calls checked; external callers pending | Directly calls `CopyProtoEntry`, then `FreeProtoEntry` on a previous successful result; this reaches NW16 and confirms the cleanup path. It reports ENOMEM only when `CopyProtoEntry` returns NULL; NW18's truncated alias array incorrectly remains a success. No Winsock query was run. |
| `arch/all-mingw32/bsdsocket/getprotobynumber.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getservbyname.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getservbyport.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getsocketevents.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getsockname.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/getsockopt.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/host_socket.c` | Full local body, selected startup/shutdown callers and host IRQ allocator contract checked; host scheduling behavior not runtime-verified | N12: after allocating both IRQs, a failed resolver event or thread creation frees only `SocketIRQ`, leaving `ResolverIRQ` allocated. N14: shutdown does not join the worker before freeing its IRQs. |
| `arch/all-mingw32/bsdsocket/host_socket.h` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/inet_addr.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/inet_lnaof.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/inet_makeaddr.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/inet_netof.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/inet_network.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/inet_ntoa.c` | Full body and per-task state initialization checked; external API callers pending | NW15: first successful conversion reverses `FreeVecPooled` arguments; initial `inaddr` is cleared by MakeLibrary, making the first argument NULL while the real pool is passed as memory. NW19: one `Forbid` is paired with two `Permit` calls, including an unconditional second decrement. No host call or scheduler operation was run. |
| `arch/all-mingw32/bsdsocket/ioctlsocket.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/listen.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/netdb_util.c` | Full body and direct `getprotobyname` cleanup caller checked; wider users pending | NW16: pooled alias-vector passed to `FreeVec`, plus reversed `FreeVecPooled` arguments. NW17: failed name copy leaves `p_aliases` uninitialized before `FreeProtoEntry`. NW18: alias-string allocation failure returns a truncated array as success. These findings are conditional on the described resolver/allocation paths; no resolver call or allocation failure was induced. |
| `arch/all-mingw32/bsdsocket/netdb_util.h` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/obtainsocket.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/recv.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/recvfrom.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/recvmsg.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/releasecopyofsocket.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/releasesocket.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/send.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/sendmsg.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/sendto.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/seterrnoptr.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/setsocketsignals.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/setsockopt.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/shutdown.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/socket.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/socket_intern.h` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/socketbasetaglist.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/vsyslog.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/waitselect.c` | Unreviewed | |
| `arch/all-mingw32/bsdsocket/winsock2.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/allocdatabuffer.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/allocdatabuffer.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_api.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_api.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_errlists.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_fdhooks.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_generic.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_generic2.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_kernvars.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_kernvars.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_libcallentry.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_libcalls.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_libtables.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_ndbent.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_netstat.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_raf.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_roadshow.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_sendrecv.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/amiga_syscalls.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/apicalls.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/dns_cache.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/dns_cache.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/gethostnamadr.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/gethtbynamadr.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/getifaddrs.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/getxbyy.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/if_indextoname.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/if_nameindex.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/if_nametoindex.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/ifaddrs.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/inet_ntop.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/inet_pton.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/miami_api.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/miami_api.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/miami_functable.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/res_comp.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/res_debug.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/res_init.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/res_mkquery.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/res_query.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/res_send.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/resolv.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/api/sockargs.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/autodoc/auto_extras.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/autodoc/auto_inetaddr.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/autodoc/auto_netdb.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/autodoc/auto_nonsocket.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/autodoc/auto_protocols.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/autodoc/auto_socket.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/conf.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/conf/conf.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/accesscontrol.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/accesscontrol.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_config.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_config.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_cstat.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_cx.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_cx.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_dhcp.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_dhcp.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_gui.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_gui.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_includes.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_log.c` | Full 753-line body read; selected startup, SysLog, and config-change callers traced; broader callers pending | Candidate 15 traces the cached `ExtLogPort` race through the in-tree SysLog shutdown. Rechecked `log_init`, `GetLogMsg`, `log_deinit`, `log_task`, `log_msg`, `log_poll`, `log_close`, `logOpen`, and `logname_changed`; no separate candidate established in this body pass. `logname_changed` cross-task execution and external callers remain open. |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_log.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_main.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_netdb.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_netdb.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_rc.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_rc.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_rexx.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_rexx.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_subr.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_time.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/amiga_time.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/config_var.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/kern_malloc.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/kern_subr.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/kern_synch.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/subr_prf.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/uio.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/uipc_domain.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/uipc_mbuf.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/uipc_socket.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/kern/uipc_socket2.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/a_if.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/bpf.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/bpf.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/bpf_filter.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/if.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/if_loop.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/if_sana.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/if_sana.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/if_stf.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/if_stf.h` | Unreviewed | |
| workbench/network/stacks/AROSTCP/bsdsocket/net/ipfilter.c | Targeted body range inspected; rest remains unreviewed | GD2539 caller check confirms ipf_check tests directional rules but never reads IPF_KEEP_STATE or maintains flow state (ipfilter.c:294-455). Full body review remains pending under the network ledger. |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/ipfilter.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/netdbpaths.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/netisr.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/netisr.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/pfil.c` | Body inspected; callers pending | Initial audit batch |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/pfil.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/radix.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/raw_cb.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/raw_cb.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/raw_usrreq.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/route.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/rtsock.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/sana2arp.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/sana2arp.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/sana2config.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/sana2config.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/sana2copybuff.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/sana2perror.c` | Reviewed | Error-table bounds and all direct call sites enumerated; finding 10 |
| `workbench/network/stacks/AROSTCP/bsdsocket/net/sana2request.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/igmp.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/igmp.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/in.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/in_cksum.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/in_cksum_neon.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/in_cksum_sse2.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/in_pcb.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/in_proto.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/in_var.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/ip_compat.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/ip_icmp.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/ip_input.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/ip_output.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/raw_ip.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_cc.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_cc_cubic.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_cc_newreno.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_compat.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_debug.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_input.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_output.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_subr.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_syncache.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_syncache.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_timer.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/tcp_usrreq.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet/udp_usrreq.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/icmp6.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/in6.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/in6_cksum.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/in6_cksum_neon.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/in6_cksum_sse2.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/in6_pcb.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/in6_proto.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/in6_var.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/ip6_input.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/ip6_output.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/mld6.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/mld6.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/nd6.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/nd6.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/nd6_nbr.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/nd6_rtr.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/netinet6/udp6_usrreq.c` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/kern/amiga_main_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/kern/amiga_select_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/kern/kern_malloc_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/kern/kern_subr_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/kern/uipc_domain_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/kern/uipc_socket2_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/kern/uipc_socket_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/net/if_loop_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/net/if_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/net/radix_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/net/raw_cb_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/net/raw_usrreq_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/net/route_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/net/rtsock_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/in_cksum_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/in_proto_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/in_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/ip_icmp_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/ip_input_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/ip_output_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/raw_ip_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/tcp_cc_cubic_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/tcp_cc_newreno_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/tcp_usrreq_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/protos/netinet/udp_usrreq_protos.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/sys/a_ioctl.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/sys/domain.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/sys/kernel.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/sys/queue2.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/sys/synch.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/sys/systm.h` | Unreviewed | |
| `workbench/network/stacks/AROSTCP/bsdsocket/version.h` | Unreviewed | |


MinGW32 BSD socket pool-release caller pass (NW15-NW18, 2026-10-20): read the complete `inet_ntoa.c`, `netdb_util.c`, and selected `getprotobyname.c` bodies; searched all direct uses of `FreeProtoEntry`, `FreeStringArray`, `CopyProtoEntry`, and `Inet_NtoA`. Confirmed the first `Inet_NtoA` result starts with the zeroed `inaddr` field (`MakeLibrary` allocates `TaskBase` with `MEMF_CLEAR`) but calls `FreeVecPooled(inaddr, pool)`, reaching `IsManagedMem(NULL)` before the newly returned string is stored (NW15). Protocol-entry cleanup passes its `AllocVecPooled` alias vector to `FreeVec`, and reverses the pool/name arguments to `FreeVecPooled` (NW16); selected `getprotobyname` calls that cleanup when replacing an old entry. The same helper leaves `p_aliases` untouched if the initial name copy fails, then tests it during cleanup (NW17). Alias-string failure breaks the copy loop but returns the non-NULL partial array, which `CopyProtoEntry` reports as a complete entry (NW18). Updated the two reviewed source rows and the direct caller row; broader Winsock API behavior and remaining MinGW32 socket files remain unreviewed. No Winsock calls, allocations, or frees were run. Source-only.


MinGW32 Inet_NtoA nesting-counter follow-up (NW19, 2026-10-20): checked both `Permit()` calls in the complete `inet_ntoa.c` body against Exec's nesting-count implementation and the selected MinGW32 interrupt scheduler. One `Forbid()` at line 51 is followed by `Permit()` at line 57 and an unconditional second `Permit()` at line 65; failed conversions also execute the unmatched decrement. The documented initial enabled count is -1, Forbid increments, and Permit decrements, while the selected scheduler treats negative counts as dispatch-enabled. Added NW19 separately from NW15's reversed FreeVecPooled call. No socket conversion or task scheduling was run. Source-only.

<a id="amiga-log-syslog-teardown-race-2026-11-03"></a>
AROSTCP NETTRACE/SysLog caller recheck (network candidate 15; 2026-11-03): traced `log_init()` caller in `amiga_main.c:401-403`, both `PutMsg(ExtLogPort, ...)` sites in `amiga_log.c:599-600,672-673`, and the in-tree SysLog `main()` teardown in `syslog/main.c:397-405`. SysLog removes its named port and calls AROSTCP `SetSysLogPort()` before draining and deleting the port. The AROSTCP writer can load the old global pointer before that refresh and resume after deletion, because the pointer load and `PutMsg()` are not protected by a shared ownership protocol. Added candidate 15 as a conditional cross-process teardown race. This is caller-slice credit only; full `amiga_log.c` body and other callers remain open. No message, port, process, or interleaving was run; source-only.

<a id="amiga-log-full-body-pass-2026-11-03"></a>
AROSTCP `amiga_log.c` full-body pass (network candidate 15 retained; no new finding; 2026-11-03): read the full 753-line source file and traced `log_init()` from `amiga_main.c`, NETTRACE startup/failure and message-pool setup, `GetLogMsg()`/`log_deinit()` callers in `subr_prf.c` and `amiga_gui.c`, the output/close path, and `logname_changed()` callbacks registered by `config_var.c`. The normal configured message pool uses fixed `LOG_BUFS=5` and `LOG_BUF_LEN=128`; no config path to change these values was found, so multiplication overflow/zero-length concerns were not promoted. No independent defect was established in this body pass. The callback's execution from external/configuration contexts and wider callers remain open. Candidate 15 is confirmed as a conditional cross-task lifetime race between `log_poll`/`log_close` and the in-tree SysLog utility's remove-refresh-drain-delete sequence. No build, message operation, or interleaving was run; source-only.
