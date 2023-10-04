### Abbreviations

* SCTP - Stream Control Transmission Protocol
* DCCP - Datagram Congestion Control Protocol
* LTE - Long Term Evolution

### Questions

1. How does `struct sk_buff` work?

2. What's New API (NAPI)?

3. netlink module

[RFC 3549](https://datatracker.ietf.org/doc/rfc3549/)

[RFC 3549 from ieft](https://tools.ietf.org/html/rfc3549)

[Wiki](https://en.wikipedia.org/wiki/Netlink)

### Internet Control Message Protocol (ICMP)

> ICMP is part of the Internet protocol suite as defined in RFC 782.

### FAQ

1. How to fetch L2, L3 and L4 header?

```C
skb_mac_header()       // L2 header
skb_network_header()   // L3 header
skb_transport_header() // L4 header
```

2. How to find MAC address?

It's done by the neighboring subsystem. In IPv4, neighbor discovery is handled by the ARP protocol, which relies on sending broadcast requests. On the contrary, In IPv6, it's handled by NDISC protocol by sending ICMPv6 requests which are in fact multicast packets.
