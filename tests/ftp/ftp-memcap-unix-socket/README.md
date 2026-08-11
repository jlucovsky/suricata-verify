# ftp memcap over the unix socket

`memcap-set ftp` and `memcap-show ftp` were wired to the FTP memcap
*counter* -- the number of times the memcap was hit -- instead of the
configured limit that the parser enforces.

So `memcap-set ftp <value>` reported success while changing nothing that is
enforced, and it clobbered the `ftp.memcap` statistic with the requested
size. `memcap-show ftp` and `memcap-list` printed that counter formatted as
a byte size, so a process that had not hit its memcap reported the FTP limit
as "unlimited" whatever the configuration said.

The test starts with a configured memcap of 10mb and checks that the value
survives a round trip through show/set/show, and that 0 (unlimited) is
accepted the way it is for the other memcaps.

Not covered:

`memcap-list`, which had the same problem as `memcap-show`, is left out
because the command segfaults on an engine that has not built the HTTP
byte-range container: it calls the getter for every entry in the table, and
`HTPByteRangeMemcapGlobalCounter` dereferences `ContainerUrlRangeList.ht`
unconditionally. That is a separate bug, and it predates this one.

`memcap-set` refusing a value below the memory already in use. It needs
`ftp.memuse` to be non-zero at the time the command runs, which means racing
the command against a pcap being processed over the same socket.
