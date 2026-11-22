# Backlog

Ideas, not commitments.

## B1. IPv6 support

proposed 2026-11-22. Today pingr resolves the target, keeps only IPv4 addresses, and exits with "has no IPv4 address (IPv6 not supported yet)".

- Flags: `-4` and `-6` to force a family. Default: an IPv6 literal uses IPv6; a hostname prefers IPv4 (current behaviour) unless `-6` is given. Consider preferring the address order `getaddrinfo` returns once IPv6 is solid.
- Socket: unprivileged `socket(AF_INET6, SOCK_DGRAM, IPPROTO_ICMPV6)`, raw fallback as for IPv4. `net.ipv4.ping_group_range` also gates ICMPv6 ping sockets, so the permission check and message stay the same.
- Echo: ICMPv6 type 128 request, 129 reply. The kernel fills the checksum on datagram sockets.
- Trace mode: hop limit via `IPV6_UNICAST_HOPS`, errors via `IPV6_RECVERR` and `MSG_ERRQUEUE` (`sock_extended_err` with origin `SO_EE_ORIGIN_ICMP6`). Time exceeded is ICMPv6 type 3, unreachable type 1.
- Interface binding: `SO_BINDTODEVICE` works the same; the interface details modal should list IPv6 addresses (it shows "none (IPv4)" today). Link-local targets (`fe80::...%eth0`) need a scope id; either support `%iface` in the target or require `-I`.
- TUI: IPv6 addresses are up to 39 characters; check the hop table and stats bar widths, and add a TestBackend render test with a long address.
- Out of scope unless asked: dual-stack side-by-side mode.

Feasibility: moderate. Mostly parallel code paths for the socket, sockaddr, and error-queue parsing; the stats and TUI stay shared. Test on a host with working IPv6 (node1's IPv6 state needs checking first) and in WSL2, where IPv6 depends on the networking mode.
