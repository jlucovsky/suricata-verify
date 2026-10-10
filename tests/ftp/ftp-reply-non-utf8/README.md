# Test Description

The FTP server replies in this test carry bytes that are not valid UTF-8. The
response line parser converts each line with `String::from_utf8_lossy`, which
writes a 3-byte U+FFFD for each invalid byte and each incomplete sequence, so
the `ftp.reply` buffer can be longer than the line on the wire. The EVE checks
match the logged reply and completion code for each command.

The replies cover:

- one invalid byte (Latin-1 `\xf6`) inside the message
- valid 2- and 4-byte UTF-8, which must pass through unchanged
- a message made only of invalid bytes (`\xff\xfe`, 2 bytes stored as 6)
- an incomplete 3-byte sequence (`\xe2\x82`), replaced by a single U+FFFD
- an invalid byte inside the completion code (`2\xff0`), which leaves no code
  and keeps the whole line as the reply
- a surrogate (`\xed\xa0\x80`), an overlong encoding (`\xc0\xaf`) and a sequence
  beyond U+10FFFF (`\xf4\x90\x80\x80`), replaced by 3, 2 and 4 U+FFFD
- a line of spaces, which the parser rejects, so the transaction logs neither a
  reply nor a completion code

Sid 1 checks that the `ftp.reply` buffer holds the converted bytes, using
`bsize` on the `\xff\xfe` reply. Sid 2 checks that the reply without a code
hands detection an empty `ftp.completion_code` buffer, which `bsize:0` matches.

The test does not detect the change made for ticket 8803. Those commits don't
change runtime behavior, and the test passes with and without them.

## PCAP

Handcrafted with scapy: one FTP control session
(`USER`/`PASS`/`SYST`/`PWD`/`NOOP`/`HELP`/`STAT`/`QUIT`), one server reply per
command.

## Related issues

Ticket https://redmine.openinfosecfoundation.org/issues/8803
