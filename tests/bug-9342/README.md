# Description

Check that TCP segment memory accounting stays balanced when pcap-log
runs with `conditional: alerts`. In that mode each segment carries a
pcap header, and its 88 bytes (on x86_64) were added to
`tcp.reassembly_memuse` twice but subtracted once when the segment was
freed.

`segment-prealloc: 0` makes the segment pool free every segment it gets
back, so all 725 segments it allocates for this pcap go through the
free path.
On x86_64, with the double count the final `tcp.reassembly_memuse` is
76040 bytes. With the fix it is 12240.

# Ticket

https://redmine.openinfosecfoundation.org/issues/9342

# Pcap

Reused from nfs3-lookup-overclaim-skip.
