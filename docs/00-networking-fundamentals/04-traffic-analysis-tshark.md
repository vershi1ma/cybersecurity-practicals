# Identifying Unknown Traffic in a Packet Capture (tshark)

**Goal:** isolate one conversation from a busy packet capture, then work out who an unexpected visitor really is using evidence I can verify, not assumptions.

**Setup:** a 50-packet capture (`tcpdump -w`) from my web server's network interface, read back with `tshark` (Wireshark's command-line version) and checked with `dig`.

## What I found

**Unexpected traffic.** The capture was meant to hold only my own test requests. Instead it caught an outside address, `216.180.246.194`, connecting to the server's HTTPS port three times. Every attempt had the same shape: a TLS Client Hello, the server answering with its certificate, then the visitor cutting the connection with a TCP reset (`RST`). No HTTP request was ever sent. The third attempt was still in progress when the capture ended.

**Isolating it with display filters.** A display filter changes what is shown, never the capture file itself.

| Filter | What it showed |
|---|---|
| `ip.addr == 216.180.246.194` | every packet to or from that one address |
| `ip.addr == 216.180.246.194 && tcp.flags.reset == 1` | only its hang-ups: two resets, ending attempts 1 and 2 |
| `tls.handshake.type == 1` | only Client Hello packets: three, all from this one address |

**Reading the behaviour.**
- The Client Hello named my server by its bare IP address (`SNI=13.63.229.50`) instead of a website name. The visitor found the server by sweeping address ranges, not by looking for my site.
- The three hellos differed in size (486, 486 and 407 bytes), consistent with probing which TLS settings the server accepts.
- A repeated SYN one second after the first was an ordinary retransmission, not hostile behaviour.

**Who is it.** A reverse DNS lookup (`dig -x`) returned `crawler194.deepfield.net`. On its own that proves nothing, because whoever controls an address can publish any name for it. So I ran the check in the other direction: the name resolves back to the same IP. This is forward-confirmed reverse DNS, and it shows the name is controlled by the same party as the address. Deepfield is a network-measurement company, so the visit fits an internet survey crawler reading my certificate.

**Verdict:** harmless measurement crawler, no action needed.

## Why this matters

- A visit that never sends an HTTP request leaves nothing in Apache's access log, so it is only visible at packet level. Logs and captures answer different questions, and an analyst needs both.
- Scanner versus real visitor at packet level: a scanner shows handshake then `RST`. A real visitor shows handshake, then encrypted `Application Data`, then an orderly close. Over HTTPS the request itself is hidden inside the encryption, so the shape of the conversation is the evidence.

## Limits of this analysis

- The identification rests on DNS evidence. It was not cross-checked against a registry (WHOIS) lookup or the operator's published address ranges, so it is strong but not absolute.
- The capture was only 50 packets, so the pattern is based on three attempts and the last one is incomplete.

## Terms used

**pcap:** a saved recording of network packets. **tshark:** reads and decodes a pcap on the command line. **Display filter:** a rule that limits which packets are shown. **Client Hello / Server Hello:** the first messages of a TLS handshake. **SNI:** the site name a client announces during the handshake. **RST:** an abrupt connection reset. **Reverse DNS:** looking up the name registered for an IP address. **Forward-confirmed reverse DNS:** checking that name resolves back to the same IP.
