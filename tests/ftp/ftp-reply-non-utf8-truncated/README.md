# Test Description

The FTP server replies in this test are made of invalid UTF-8 and sit at the
4096-byte line limit. The limit applies to the raw line, and the response line
parser converts the line with `String::from_utf8_lossy` afterward, so each
invalid byte becomes a 3-byte U+FFFD and the `ftp.reply` buffer can reach three
times the limit.

The limit counts the line's CRLF. The replies cover:

- 4093 bytes of `\xff`: 4095 with the CRLF, under the limit, not truncated,
  stored as 12279 bytes
- 4094 bytes of `\xff`: 4096 with the CRLF, flagged truncated although the
  parser keeps all 4094 bytes, stored as 12282 bytes
- 4100 bytes of `\xff`: cut to 4096 bytes, stored as 12288 bytes
- 4095 bytes of `a` followed by `\xc3\xa9`: the cut keeps only `\xc3`, which
  becomes U+FFFD, stored as 4098 bytes

The EVE checks match `reply_truncated` for each command and the three
`response_command_too_long` events. The rules check each stored length with
`bsize`.

The test does not detect the change made for ticket 8803. Those commits don't
change runtime behavior, and the test passes with and without them.

## PCAP

Handcrafted with scapy: one FTP control session
(`USER`/`PASS`/`SYST`/`PWD`/`QUIT`), one server reply per command, each reply in
one TCP segment.

## Related issues

Ticket https://redmine.openinfosecfoundation.org/issues/8803
