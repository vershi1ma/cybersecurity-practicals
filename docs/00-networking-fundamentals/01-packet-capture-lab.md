# Phase 0: Watching a Web Request Travel

**Goal:** see what really happens on the network when a computer visits a website, by recording the packets myself.

**Setup:** an AWS EC2 instance reached through Session Manager (no SSH, port 22 closed). Tools: `dig`, `curl`, `tcpdump`.

## What I found

**1. DNS (`dig example.com`)**
- The name resolved to two IPs from the AWS resolver at `172.31.0.2`, over UDP port 53, with a 291 s cache lifetime (TTL).
- `tcpdump` showed just two packets, a question and an answer matched by the same query ID, in about 4 ms.
- Classic DNS is unencrypted and unauthenticated, which is why DNS spoofing exists and why DNSSEC and DoH were created.

**2. Plain HTTP**
- Saw the TCP three-way handshake (SYN, SYN-ACK, ACK), then `GET / HTTP/1.1` and `200 OK`, all in readable text.
- Anyone on the network path can read this, including passwords typed into a site.

**3. HTTPS**
- Same conversation on port 443: the content was scrambled by TLS encryption.
- Still visible: IPs, ports, packet sizes, timing, and the site name (`example.com`, sent in the clear as SNI). Encryption hides what was said, not who is talking to whom.

## Problem hit
My first HTTPS capture was full of unexpected traffic. Session Manager itself runs over port 443, so `tcpdump` was recording my own terminal. Filtering to only the two example.com IPs fixed it: a capture records everything that matches, so filters must be specific.

## OSI mapping

| Layer | Seen in this lab |
|---|---|
| 7 Application | DNS question, `GET / HTTP/1.1` |
| 6 Presentation | TLS encryption |
| 4 Transport | TCP or UDP; ports 53, 80, 443 |
| 3 Network | IP addresses |
| 2 Data Link | Interface `ens5` |
