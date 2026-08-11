# ftp memuse accounting for trimmed command lines

`CopyCommandLine` allocated `line->len + 1` bytes, then trimmed trailing
whitespace by decrementing `line->len`, and returned the post-trim length.
That value is stored as `tx->request_length` and is what `FTPTransactionFree`
credits back, so every command line carrying trailing whitespace left the
difference in `ftp.memuse` permanently. Since memuse only drifted upwards,
the effective memcap tightened over the life of the process.

The whitespace has to be inside the line rather than be the line delimiter:
`FTPGetLineForDirection` already removes the CR/LF, so ordinary traffic does
not reach the trim. The commands here carry trailing spaces and tabs ahead
of the CRLF, plus one command with nothing to trim as a control.

The last case is the boundary: `max-line-length` is lowered to 40 so that the
RETR line is truncated, which is the one path that hands `CopyCommandLine` a
line with no delimiter stripped. The cut lands inside a run of spaces, so the
trim fires on a truncated line too.

Before the fix this pcap leaves 16 bytes in `ftp.memuse` after the flow is
gone; the last stats record is written after that.

Not covered: a command line that trims away entirely. `CopyCommandLine` is
only reached once the line has matched a known command, so an all-whitespace
line is discarded before it gets there.

## PCAP

Generated with flowsynth; see the Makefile.
