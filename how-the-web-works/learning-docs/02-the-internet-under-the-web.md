# Chapter 2: The Internet Under the Web — IP, Packets, TCP/UDP, Ports

## Overview

The web rides on the internet the way trucks ride on roads. This chapter is about the roads: how data physically gets from your machine to a server across the world and back. You'll learn what an IP address really identifies, why data travels as small **packets** rather than whole files, the difference between TCP and UDP (and why the web mostly chose TCP), what **ports** are, and why `localhost` is the most useful address you'll ever memorize as a developer.

Why it matters: every HTTP request in this track — every page load, every API call — is carried by the machinery in this chapter. When a site "won't connect," a port is "already in use," or a request "times out," you are debugging *this* layer, not HTML or JavaScript.

## Definitions & Explanations

### IP addresses: the internet's street addresses

Every device directly connected to the internet has an **IP address** — a number that identifies where packets should be delivered.

- **IPv4**: four numbers 0–255 separated by dots, e.g. `93.184.216.34`. Only ~4.3 billion possible addresses — we've essentially run out.
- **IPv6**: the successor, written in hex like `2606:2800:220:1:248:1893:25c8:1946`. Vastly more addresses. Both are in active use; your machine likely has both kinds.

Special addresses worth knowing:

| Address | Meaning |
|---|---|
| `127.0.0.1` | Loopback — "this machine" (IPv4). `localhost` resolves here. |
| `::1` | Loopback in IPv6. |
| `192.168.x.x`, `10.x.x.x`, `172.16-31.x.x` | **Private** ranges — only valid inside your local network (home Wi-Fi, office LAN). Not reachable from the internet. |
| `0.0.0.0` | "All interfaces" — a server listening here accepts connections from any network the machine is on. |

Your home network almost certainly uses **NAT** (Network Address Translation): every device gets a private address like `192.168.1.23`, and your router presents one shared public address to the outside world, translating back and forth. This is why "my IP" gives different answers depending on where you ask (see hands-on below).

### Packets: data travels in pieces

The internet does not open a pipe and pour your file through it. Data is chopped into **packets** — chunks of at most ~1,500 bytes, each stamped with source and destination addresses, like separately-mailed envelopes:

```
A 3 MB image becomes roughly 2,000 packets:

+-----------------------------+
| To:   93.184.216.34         |   <- header (addressing info)
| From: 203.0.113.7           |
| Part: 517 of 2000           |
+-----------------------------+
| ...1400 bytes of image...   |   <- payload (your actual data)
+-----------------------------+
```

Each packet is independently passed from **router** to router, each router looking only at the destination and forwarding it one hop closer. Packets from the same file can take *different routes*, arrive *out of order*, or get *lost entirely*. This design — "dumb network, smart endpoints" — is why the internet survives broken links: routers just route around damage.

But it creates a problem: who reassembles the pieces, notices missing ones, and asks for re-sends? That's the job of the next layer up.

### TCP vs UDP: two delivery philosophies

Both are protocols that run on top of IP and deliver packets to programs. They make opposite trade-offs:

**TCP (Transmission Control Protocol)** — *reliable, ordered, slower to start*:
- Before any data flows, client and server perform a **three-way handshake** ("Can we talk?" / "Yes, can you hear me?" / "Yes — go") to establish a **connection**.
- Every packet is acknowledged; lost packets are re-sent automatically.
- Packets are reassembled in the correct order before your program sees them.
- The receiving program gets a clean, complete, in-order stream — as if the network were perfect.

**UDP (User Datagram Protocol)** — *fast, connectionless, no guarantees*:
- No handshake, no connection. Just fling packets at the destination.
- No acknowledgements, no retransmission, no ordering. Lost packets stay lost.
- Ideal when *freshness beats completeness*: live video, voice calls, online games. A re-sent video frame from 2 seconds ago is worthless — better to skip it.

```
TCP:  "Did you get packet 4?"  "No."  "Here it is again."  (correct, eventually)
UDP:  "packet packet packet packet..."                     (fast, maybe lossy)
```

The web historically runs on TCP (HTTP/1.1 and HTTP/2), because a web page with missing chunks is garbage — reliability matters more than the handshake cost. (Modern HTTP/3 actually runs over QUIC, which is built on UDP but *re-implements* reliability smartly — proof that these are design trade-offs, not laws. Chapter 10 touches on this.)

### Ports: many programs, one address

An IP address gets packets to the right *machine* — but a machine runs many networked programs at once. **Ports** (numbers 0–65535) get the data to the right *program*. Think apartment numbers in one building:

```
            93.184.216.34  (one machine)
            +---------------------------+
  :22  ---> | SSH server                |
  :80  ---> | web server (plain HTTP)   |
  :443 ---> | web server (HTTPS)        |
  :5432 --> | PostgreSQL database       |
            +---------------------------+
```

Well-known defaults you'll see constantly:

| Port | Used for |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |
| 53 | DNS |
| 3000, 5000, 5173, 8000, 8080 | common local dev servers |

A server program **listens** on a port (claims it — only one program per port). A client is assigned a random high-numbered **ephemeral port** for its end of the conversation. A TCP connection is fully identified by the four-tuple: *(client IP, client port, server IP, server port)* — which is how one server on port 443 can talk to thousands of clients simultaneously.

`https://example.com` implicitly means port 443; `http://localhost:8000` explicitly says port 8000. The `:number` in URLs is this exact concept.

### localhost: the network to yourself

`localhost` (which resolves to `127.0.0.1`) means "this very machine." Packets sent there never touch a cable or Wi-Fi — the operating system loops them straight back. This is how you'll develop everything: run a server on a port, connect to `localhost:<port>` as the client. Same protocols, same tools, zero internet required.

## Hands-On Examples

Open PowerShell for all of these.

### 1. Meet your own addresses

```powershell
ipconfig
```

Look for `IPv4 Address` under your active adapter (e.g. `192.168.1.23`) — that's your *private* address. Now ask an outside service what address it sees:

```powershell
curl.exe https://api.ipify.org
```

Expected: a completely different address (e.g. `86.140.72.19`) — your router's *public* face. Two different answers to "what's my IP" — that's NAT in action.

### 2. Reachability and round-trip time with ping

```powershell
ping example.com
```

Expected output (times vary):

```
Pinging example.com [93.184.216.34] with 32 bytes of data:
Reply from 93.184.216.34: bytes=32 time=88ms TTL=56
Reply from 93.184.216.34: bytes=32 time=87ms TTL=56
...
```

The `time=` value is the **round-trip time (RTT)** — a physical latency floor set by distance and the speed of light. Ping something nearby (your router: `ping 192.168.1.1`, adjust to your gateway from `ipconfig`) and something far away, and compare. No amount of code optimization beats geography — this is why CDNs exist (Chapter 9).

### 3. Watch packets hop with tracert

```powershell
tracert example.com
```

Expected: a numbered list of 8–20 hops — each line is a router your packets pass through. Hop 1 is your home router; the next few are your ISP; then backbone networks; finally the destination. Lines showing `* * *` are routers that decline to answer (normal). You are literally looking at the internet's road network.

### 4. Ports in action — and a port conflict

Start a server, then try to start a second one on the same port:

```powershell
python -m http.server 8000
```

Leave that running; open a *second* PowerShell window:

```powershell
python -m http.server 8000
```

Expected: an error ending in something like `OSError: [WinError 10048] Only one usage of each socket address ... is normally permitted` — the port is taken. Now run it on `8001` instead — it works. This "address already in use" error will visit you many times in your career; now you know exactly what it means. Stop both with `Ctrl+C`.

### 5. See live connections with netstat

With your Python server still running on 8000, load `http://localhost:8000` in the browser, then:

```powershell
netstat -an | Select-String "8000"
```

Expected: lines like

```
TCP    0.0.0.0:8000       0.0.0.0:0          LISTENING
TCP    127.0.0.1:8000     127.0.0.1:52814    ESTABLISHED
```

The first is your server listening; the second is an actual TCP connection — note the browser's random ephemeral port (`52814` here) on the other end. That's the four-tuple, live on your screen.

## Common Misconceptions

- **"My computer has one IP address."** It typically has several: a loopback (`127.0.0.1`), a private LAN address (`192.168.x.x`), an IPv6 address, and it *shares* a public address via NAT. "What's my IP" depends on who's asking from where.
- **"Data travels as a continuous stream, like water in a pipe."** It travels as thousands of independent packets that may take different routes and arrive out of order. TCP creates the *illusion* of a smooth stream on top of that chaos.
- **"TCP is good, UDP is bad/unreliable junk."** They're trade-offs. UDP's "unreliability" is a *feature* for live media and games, where late data is worse than lost data. HTTP/3 runs over UDP.
- **"Ports are physical sockets on the computer."** Ports are purely software — numbers in packet headers used by the OS to route data to the right program. Nothing physical about them.
- **"localhost involves the network/my router."** Loopback traffic never leaves your machine. Your Wi-Fi can be off and `localhost` servers still work — a great isolation trick when debugging.
- **"IP addresses identify a person."** They identify a network interface, usually a shared router, often reassigned over time. Geolocation from IP is city-level guesswork at best.

## Practice Exercises

1. **Address inventory.** Using `ipconfig` and `curl.exe https://api.ipify.org`, list every IP address associated with your machine right now. Label each as loopback / private / public, and note which ones would work in a URL typed on a friend's computer at their house.
2. **Latency map.** Ping five hosts: your router, `example.com`, and three sites you believe are hosted on different continents (guess from the organization). Record average RTTs in a table and rank them. Do the numbers match geography?
3. **Trace comparison.** Run `tracert` against two different sites and compare the first four hops. Explain why they're identical (or nearly so) regardless of destination.
4. **Port detective.** Run `netstat -an | Select-String "LISTENING"` and count how many TCP ports are listening on your machine. Pick two port numbers you recognize (or look up) and note which program probably owns them. (Hint: `netstat -ab` run as Administrator shows the owning program.)
5. **Two servers, one machine.** Run two Python http servers simultaneously on ports 8000 and 8001, each in a different folder containing a different `index.html`. Verify in the browser that `localhost:8000` and `localhost:8001` serve different content, and explain in one sentence how your OS knows which server gets which request.
