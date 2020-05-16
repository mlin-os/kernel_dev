### Questions

1. How does `struct sk_buff` work?

2. What's New API (NAPI)?

3. netlink module

[RFC 3549](https://datatracker.ietf.org/doc/rfc3549/)

[RFC 3549 from ieft](https://tools.ietf.org/html/rfc3549)

[Wiki](https://en.wikipedia.org/wiki/Netlink)

### FAQ

1. How to fetch L2, L3 and L4 header?

```C
skb_mac_header()       // L2 header
skb_network_header()   // L3 header
skb_transport_header() // L4 header
```
