# ftp response line accounted against the memcap

The response line payload is allocated by the parser, and its `total_size`
was added to `ftp.memuse` when `FTPResponseWrapperAlloc` built the wrapper --
without ever consulting the memcap. A response could therefore push memuse
past the configured limit; the next allocation that did check was then
denied, so the excess was bounded, but the memcap was not a ceiling.

The reply to USER in this pcap is about 3000 bytes and the memcap is 2048,
so the gap is wide enough that the test does not depend on the exact size of
the FTP structures. Before the fix the response is stored and memuse goes
over the limit; after it the response is refused, the command is still
logged, and the exchanges that follow are unaffected.

The check happens after the line has been parsed, so the fix bounds the
overshoot to a single response rather than removing it.

Not covered: the other unchecked allocation, the transfer command from
`SCFTPTransferCmdNew`. It is a few dozen bytes and the overshoot is bounded
to one such allocation, which is too small to distinguish from ordinary
memcap behaviour without depending on exact structure sizes.

## PCAP

Generated with flowsynth; see the Makefile.
