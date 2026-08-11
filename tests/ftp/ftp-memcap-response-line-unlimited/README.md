# ftp response line with the memcap unlimited

The companion to ftp-memcap-response-line, run over the same pcap with
`app-layer.protocols.ftp.memcap` set to 0. A memcap of 0 means unlimited, so
the ~3000 byte response that the other test sees refused is recorded here in
full and the memcap counter stays at 0.

This is the boundary the new check in `FTPResponseWrapperAlloc` has to get
right: `FTPCheckMemcap` returns 1 for any size when the limit is 0, so adding
the check must not start dropping responses on the default configuration.
