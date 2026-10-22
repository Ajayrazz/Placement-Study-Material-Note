# Computer Networks (CN) — The SDE-1 & SWE Intern Interview Handbook

> **Target Audience:** SWE Interns & SDE-1 Candidates  
> **Objective:** Master networking models, protocols, low-level mechanics, real-world analogies, and exact interview talking points.

---

## Table of Contents
1. [Network Models: OSI vs. TCP/IP](#1-network-models-osi-vs-tcpip)
   - [The 7-Layer OSI Model Deep Dive](#the-7-layer-osi-model-deep-dive)
   - [TCP/IP 4-Layer Architecture](#tcpip-4-layer-architecture)
   - [Packet Encapsulation & Decapsulation](#packet-encapsulation--decapsulation)
2. [Application Layer Protocols](#2-application-layer-protocols)
   - [DNS (Domain Name System) Deep Dive](#dns-domain-name-system-deep-dive)
   - [HTTP Evolution: HTTP/1.1 vs. HTTP/2 vs. HTTP/3 (QUIC)](#http-evolution-http11-vs-http2-vs-http3-quic)
   - [HTTPS & The TLS 1.3 Handshake Sequence](#https--the-tls-13-handshake-sequence)
   - [Real-Time Communication: WebSockets vs. SSE vs. Long Polling](#real-time-communication-websockets-vs-sse-vs-long-polling)
3. [Transport Layer (High-Frequency Interview Topic)](#3-transport-layer-high-frequency-interview-topic)
   - [TCP vs. UDP: Comprehensive Face-Off](#tcp-vs-udp-comprehensive-face-off)
   - [TCP 3-Way Handshake (Connection Establishment)](#tcp-3-way-handshake-connection-establishment)
   - [TCP 4-Way Teardown & The TIME_WAIT State](#tcp-4-way-teardown--the-time_wait-state)
   - [TCP Flow Control vs. Congestion Control](#tcp-flow-control-vs-congestion-control)
   - [TCP Congestion Control Algorithms (AIMD, Slow Start, Fast Retransmit)](#tcp-congestion-control-algorithms)
4. [Network Layer (IP & Routing)](#4-network-layer-ip--routing)
   - [IPv4 vs. IPv6](#ipv4-vs-ipv6)
   - [Subnetting, CIDR & IP Classes](#subnetting-cidr--ip-classes)
   - [NAT (Network Address Translation) & Private IP Ranges](#nat-network-address-translation--private-ip-ranges)
   - [Routing Protocols (Distance Vector vs. Link State, BGP, OSPF)](#routing-protocols)
   - [ICMP (Ping & Traceroute Mechanics)](#icmp-ping--traceroute-mechanics)
   - [ARP (Address Resolution Protocol) & ARP Poisoning](#arp-address-resolution-protocol--arp-poisoning)
5. [Data Link & Physical Layer Essentials](#5-data-link--physical-layer-essentials)
   - [MAC Address vs. IP Address](#mac-address-vs-ip-address)
   - [Network Devices: Hub vs. Switch vs. Router vs. Gateway](#network-devices-hub-vs-switch-vs-router-vs-gateway)
   - [Collision Domains vs. Broadcast Domains](#collision-domains-vs-broadcast-domains)
6. [The Ultimate Interview Question: "What happens when you type google.com?"](#6-the-ultimate-interview-question-what-happens-when-you-type-googlecom)
7. [Network Security Fundamentals](#7-network-security-fundamentals)
   - [Symmetric vs. Asymmetric Encryption](#symmetric-vs-asymmetric-encryption)
   - [Hashing vs. Encryption vs. Encoding](#hashing-vs-encryption-vs-encoding)
   - [CORS (Cross-Origin Resource Sharing)](#cors-cross-origin-resource-sharing)
8. [Top 20 SDE-1 / Intern Interview Questions & Model Answers](#8-top-20-sde-1--intern-interview-questions--model-answers)

---

## 1. Network Models: OSI vs. TCP/IP

### The 7-Layer OSI Model Deep Dive
The Open Systems Interconnection (OSI) model is a conceptual 7-layer framework that standardizes network communication functions.

```
Mnemonic: "Please Do Not Throw Sausage Pizza Away" (Layers 1 -> 7)
Mnemonic: "All People Seem To Need Data Processing" (Layers 7 -> 1)

+---+--------------------+------------------------------------------+-----------------------+
| # | Layer              | Primary Responsibility                   | Protocols / Hardware  |
+---+--------------------+------------------------------------------+-----------------------+
| 7 | Application        | Network services for user applications   | HTTP, HTTPS, DNS, FTP |
| 6 | Presentation       | Data translation, encryption, compression| TLS, SSL, JPEG, JSON  |
| 5 | Session            | Establishes, manages, & ends sessions    | RPC, NetBIOS, SOCKS   |
| 4 | Transport          | End-to-end delivery, reliability, ports  | TCP, UDP              |
| 3 | Network            | Logical addressing & path routing        | IP, ICMP, ARP, OSPF   |
| 2 | Data Link          | Node-to-node framing & MAC addressing    | Ethernet, Wi-Fi, Switch|
| 1 | Physical           | Raw binary bit transmission over media   | Cables, Fiber, Hub, RF|
+---+--------------------+------------------------------------------+-----------------------+
```

---

### TCP/IP 4-Layer Architecture
The TCP/IP model is the **pragmatic, real-world implementation** used on the internet. It combines OSI layers 5, 6, and 7 into a single Application layer, and layers 1 and 2 into the Network Access (Link) layer.

```
OSI 7-LAYER MODEL                       TCP/IP 4-LAYER MODEL
+-----------------------+               +-----------------------+
| 7. Application        | \             |                       |
| 6. Presentation       |  +----------> | 4. Application Layer  | (Data)
| 5. Session            | /             |                       |
+-----------------------+               +-----------------------+
| 4. Transport          | ------------> | 3. Transport Layer    | (Segment / Datagram)
+-----------------------+               +-----------------------+
| 3. Network            | ------------> | 2. Internet Layer     | (Packet)
+-----------------------+               +-----------------------+
| 2. Data Link          | \             | 1. Network Access     | (Frame)
| 1. Physical           |  +----------> |    (Link) Layer       | (Bits)
+-----------------------+               +-----------------------+
```

---

### Packet Encapsulation & Decapsulation

When data moves **down** the sender's stack, each layer wraps the payload with its own protocol header (**Encapsulation**). When data moves **up** the receiver's stack, each layer strips its header (**Decapsulation**).

```
Sender (Encapsulation: Downwards)            Receiver (Decapsulation: Upwards)
Application:  [ Data ]                                    ^
                     |                                    | Strips TCP Header
Transport:    [ TCP Header | Data ] = SEGMENT             |
                     |                                    | Strips IP Header
Internet:     [ IP Header | TCP | Data ] = PACKET         |
                     |                                    | Strips MAC & Trailer
Network Access:[ MAC Header | IP | TCP | Data | FCS ]     |
                     | (FRAME)                            |
Physical:     01011101001010101011010001010010 ---------->
```

**Protocol Data Units (PDUs):**
- **Application Layer:** Message / Data
- **Transport Layer:** Segment (TCP) / Datagram (UDP)
- **Network Layer:** Packet (Logical addressing: IP to IP)
- **Data Link Layer:** Frame (Physical addressing: MAC to MAC)
- **Physical Layer:** Bits (Voltage / Light Pulses)

---

## 2. Application Layer Protocols

### DNS (Domain Name System) Deep Dive
DNS is the "Phonebook of the Internet." It resolves human-readable hostnames (e.g., `api.uber.com`) into machine-routable IP addresses (e.g., `104.18.22.45`).

#### DNS Query Flow (Recursive vs. Iterative)
> **Analogy:** Looking up a book in a massive hierarchical library.
> You ask the Front Desk (Recursive Resolver). The front desk asks the Section Guide (Root), who directs them to the Hallway (TLD), who points to the exact Shelf (Authoritative Server).

```
[ User Browser ]
       | 1. "What is www.google.com?"
       v
[ Local Recursive DNS Resolver ] (ISP or 8.8.8.8 / 1.1.1.1)
       | 2. Check local DNS Cache -> Miss!
       |
       | 3. Query Root DNS Server (".")
       |<-- 4. Returns TLD Server IP (".com")
       |
       | 5. Query TLD DNS Server (".com")
       |<-- 6. Returns Authoritative DNS Server IP ("google.com")
       |
       | 7. Query Authoritative DNS Server ("ns1.google.com")
       |<-- 8. Returns A Record: 142.250.190.46 (TTL: 300s)
       |
       | 9. Caches record & returns IP to Browser
       v
[ Browser initiates TCP connection to 142.250.190.46 ]
```

#### Core DNS Record Types:
- **A Record:** Maps hostname to IPv4 address (`example.com -> 93.184.216.34`).
- **AAAA Record:** Maps hostname to IPv6 address (`example.com -> 2606:2800:220:1:248:1893:25c8:1946`).
- **CNAME (Canonical Name):** Maps an alias hostname to another hostname (`www.example.com -> example.com`).
- **MX (Mail Exchange):** Directs domain email to mail servers (`mail.google.com`).
- **TXT Record:** Stores arbitrary text; critical for domain ownership validation, **SPF** (Sender Policy Framework), and **DKIM** to prevent email spoofing.
- **NS (Name Server):** Delegates a DNS zone to authoritative servers.
- **TTL (Time-To-Live):** The duration (in seconds) that resolvers are permitted to cache the DNS response.

---

### HTTP Evolution: HTTP/1.1 vs. HTTP/2 vs. HTTP/3 (QUIC)

```
+-------------------------------------------------------------------------------------------------+
|                                 HTTP EVOLUTION COMPARISON                                       |
+-------------------+------------------------------+--------------------+-------------------------+
| Feature           | HTTP/1.1 (1997)              | HTTP/2 (2015)      | HTTP/3 (2022)           |
+-------------------+------------------------------+--------------------+-------------------------+
| Underlying Protocol| TCP                          | TCP                | **QUIC (over UDP)**     |
| Transport Format  | Plaintext / ASCII            | Binary Framing     | Binary Framing          |
| Multiplexing      | No (Head-of-Line Blocking at | **Yes** (Streams   | **Yes** (Independent    |
|                   | application layer)           | over single TCP)   | streams over QUIC)      |
| TCP HoL Blocking  | Yes                          | **Yes** (Single packet| **No** (Packet loss on|
|                   |                              | loss stalls all)   | stream 1 doesn't block 2|
| Header Compression| None (Heavy redundant text)  | **HPACK**          | **QPACK**               |
| Server Push       | No                           | Yes                | Yes                     |
| Connection Handshake| TCP Handshake + TLS (2-3 RTT)| TCP + TLS (2-3 RTT)| **0-RTT or 1-RTT**      |
| Connection Migration| Breaks on IP change (WiFi->4G)| Breaks on IP change| **Connection ID based** |
|                   |                              |                    | (Survives IP changes)   |
+-------------------+------------------------------+--------------------+-------------------------+
```

#### Understanding Head-of-Line (HoL) Blocking:
- **HTTP/1.1 Application HoL Blocking:** Browsers can only send one request per TCP connection at a time. Workaround: Browsers open 6 parallel TCP connections per domain.
- **HTTP/2 Transport HoL Blocking:** HTTP/2 multiplexes multiple requests into logical streams over **a single TCP connection**. However, if **one single TCP packet is dropped**, TCP halts all subsequent packets until the missing packet is retransmitted. Thus, all concurrent HTTP streams stall!
- **HTTP/3 Solution:** Runs on top of **QUIC over UDP**. Each HTTP stream is tracked independently. A lost packet on Stream A **never** delays Stream B.

---

### HTTPS & The TLS 1.3 Handshake Sequence
HTTPS is HTTP encrypted inside a **TLS (Transport Layer Security)** tunnel. It provides three guarantees:
1. **Confidentiality:** Data cannot be read by eavesdroppers.
2. **Integrity:** Data cannot be modified or corrupted in transit.
3. **Authentication:** Proves the client is communicating with the genuine server, not an impostor.

```
TLS 1.3 HANDSHAKE (1 Round Trip Time [1-RTT]):

Client                                                    Server
  |                                                         |
  | -------- 1. ClientHello ------------------------------> |
  |   - Supported TLS versions (e.g., TLS 1.3)              |
  |   - Supported Cipher Suites                             |
  |   - Client Random                                       |
  |   - Key Share (Diffie-Hellman public key)               |
  |                                                         |
  | <------- 2. ServerHello ------------------------------- |
  |   - Chosen Cipher Suite                                 |
  |   - Server Random                                       |
  |   - Server Key Share (Diffie-Hellman public key)        |
  |   - Digital Certificate (X.509 signed by CA)            |
  |   - Finished (HMAC verifying handshake integrity)       |
  |                                                         |
  | [Client verifies Certificate using Root CA public keys] |
  | [Both independently compute Symmetric Session Key]      |
  |                                                         |
  | <====== 3. ENCRYPTED APPLICATION DATA TRAFFIC =========> |
```

> **Why Asymmetric + Symmetric?**  
> Asymmetric encryption (RSA / ECC) is computationally expensive. It is used **only during the initial handshake** to authenticate identity and safely agree upon a shared secret. Once established, all high-throughput payload data is encrypted using high-speed **Symmetric encryption** (AES-256-GCM / ChaCha20).

---

### Real-Time Communication: WebSockets vs. SSE vs. Long Polling

| Feature | Short Polling | Long Polling | Server-Sent Events (SSE) | WebSockets |
| :--- | :--- | :--- | :--- | :--- |
| **Direction** | Client $\rightarrow$ Server | Client $\rightarrow$ Server | Server $\rightarrow$ Client | **Full Duplex** ($\leftrightarrow$) |
| **Protocol** | Standard HTTP | Standard HTTP | Standard HTTP (`text/event-stream`) | `ws://` or `wss://` (TCP) |
| **Handshake** | Repeated HTTP requests | Repeated HTTP requests | Single HTTP connection | HTTP Upgrade handshake |
| **Connection** | Opens and closes repeatedly | Server holds connection until data arrives | Long-lived persistent HTTP | Single persistent TCP socket |
| **Overhead** | Very High (repeated headers) | High (new headers per response) | Very Low | Minimal (2-byte frame header) |
| **Best For** | Infrequently updated dashboards | Legacy chat systems | Stock tickers, Live sports scores | Online gaming, Collaborative editing (Google Docs), Crypto trading |

---

## 3. Transport Layer (High-Frequency Interview Topic)

### TCP vs. UDP: Comprehensive Face-Off

```
+--------------------------------------------------------------------------+
|                            TCP vs. UDP COMPARISON                        |
+-----------------------+-------------------------+------------------------+
| Feature               | TCP (Transmission Control| UDP (User Datagram     |
|                       | Protocol)               | Protocol)              |
+-----------------------+-------------------------+------------------------+
| Connection Type       | Connection-Oriented     | Connectionless         |
|                       | (Requires 3-way handshake) (Send & Forget)        |
| Reliability           | 100% Guaranteed delivery| Unreliable (Best-effort|
|                       | (ACKs + Retransmission) | packets can drop/dupe) |
| Ordering              | Strictly Ordered        | No order guarantee     |
|                       | (Sequence Numbers)      | (Handled by app layer) |
| Header Size           | 20 - 60 Bytes           | Fixed 8 Bytes          |
| Speed                 | Slower (Overhead + acks)| Blazing Fast (Minimal) |
| Flow Control          | Yes (Sliding Window)    | No                     |
| Congestion Control    | Yes (AIMD, Slow Start)  | No                     |
| Transmission Model    | Byte Stream             | Datagram / Message     |
| Real-World Analogy    | Registered Postal Mail  | Walkie-talkie / Radio  |
| Use Cases             | Web (HTTP/HTTPS), SSH,  | Live Video Streaming,  |
|                       | Email (SMTP), File (FTP)| VoIP, DNS, Gaming (FPS)|
+-----------------------+-------------------------+------------------------+
```

---

### TCP 3-Way Handshake (Connection Establishment)

Before any data is exchanged, client and server must synchronize sequence numbers and allocate transmission buffers.

```
Client (Port: 54321)                              Server (Port: 80/443)
  [ CLOSED ]                                           [ LISTEN ]
      |                                                     |
      | -------- 1. SYN (seq = X) ------------------------> | [ SYN_RCVD ]
  [ SYN_SENT ]                                              |
      |                                                     |
      | <------- 2. SYN-ACK (seq = Y, ack = X + 1) -------- |
  [ ESTABLISHED ]                                           |
      |                                                     |
      | -------- 3. ACK (seq = X + 1, ack = Y + 1) -------> | [ ESTABLISHED ]
      |                                                     |
  [ DATA TRANSMISSION BEGINS ]                              |
```

1. **Step 1 (SYN):** Client generates a random initial sequence number $X$ and sends a packet with flag `SYN=1`. Client enters `SYN_SENT`.
2. **Step 2 (SYN-ACK):** Server acknowledges receipt by setting `ACK=1, ack_num = X + 1`. Server generates its own random sequence number $Y$, sets `SYN=1`, and sends back. Server enters `SYN_RCVD`.
3. **Step 3 (ACK):** Client acknowledges server's sequence number by setting `ACK=1, ack_num = Y + 1`. Both sockets enter `ESTABLISHED`. *(Client can piggyback HTTP request data inside this 3rd packet!)*

> **Interview Trap: Why initial sequence numbers (ISN) are random?**  
> If ISNs always started at 0, old delayed duplicate packets from a previous closed connection could arrive and corrupt the new connection. Randomization also prevents **TCP sequence prediction attacks** (TCP Spoofing).

---

### TCP 4-Way Teardown & The TIME_WAIT State

Because TCP is **Full-Duplex** (both sides can independently send and receive), closing a connection requires each direction to be terminated separately.

```
Client (Active Closer)                            Server (Passive Closer)
  [ ESTABLISHED ]                                      [ ESTABLISHED ]
      |                                                     |
      | -------- 1. FIN (seq = u) ------------------------> | [ CLOSE_WAIT ]
  [ FIN_WAIT_1 ]                                            |
      |                                                     |
      | <------- 2. ACK (ack = u + 1) --------------------- |
  [ FIN_WAIT_2 ]                                            |
      |                                                     | (Server finishes
      |                                                     |  flushing pending data)
      |                                                     |
      | <------- 3. FIN (seq = v) ------------------------- | [ LAST_ACK ]
      |                                                     |
      | -------- 4. ACK (ack = v + 1) --------------------> |
  [ TIME_WAIT ]                                        [ CLOSED ]
      | (Waits 2 * MSL)                                     |
  [ CLOSED ]                                                |
```

#### The Critical Interview Concept: Why does `TIME_WAIT` exist?
The active closer (usually client) does not close immediately; it waits for $2 \times \text{MSL}$ (Maximum Segment Lifetime, typically 60 to 120 seconds).
1. **Ensure Final ACK is received:** If the 4th packet (`ACK`) is lost in the network, the server will retransmit its `FIN`. If the client were already `CLOSED`, it would respond with an `RST` (error), causing the server to terminate abnormally instead of cleanly.
2. **Drain lingering duplicate packets:** Ensures that any delayed packets belonging to this connection expire in the internet router buffers before a new connection reuses the same port number.

---

### TCP Flow Control vs. Congestion Control

```
+--------------------------------------------------------------------------+
|                 FLOW CONTROL vs. CONGESTION CONTROL                      |
+-------------------+------------------------------+-----------------------+
| Dimension         | Flow Control                 | Congestion Control    |
+-------------------+------------------------------+-----------------------+
| Objective         | Prevent **Sender from        | Prevent **Sender from |
|                   | overwhelming the Receiver**  | overwhelming the intermediate|
|                   | (Buffer overflow)            | Network Routers**     |
| Mechanism         | **Receive Window (rwnd)**    | **Congestion Window   |
|                   | advertised in TCP header     | (cwnd)** maintained in|
|                   | by receiver.                 | sender's memory.      |
| Controlling Entity| The Receiver.                | The Sender.           |
| Formula           | Max data allowed in flight = min(rwnd, cwnd)         |
+-------------------+------------------------------+-----------------------+
```

---

### TCP Congestion Control Algorithms

```
Congestion Window (cwnd)
  ^
  |                                       / (Additive Increase)
  |                  Threshold (ssthresh)/
  |                     +---------------+
  |                    /
  |                   / (Slow Start: Exponential)
  |                  /
  |                 /  <-- Packet Loss Occurs!
  |                /       (Drops to 1 or ssthresh / 2)
  |               /
  |  +-----------+
  +-----------------------------------------------------> Time
```

1. **Slow Start:**
   * Starts with a small `cwnd` (e.g., 10 MSS).
   * For every ACK received, `cwnd` increases by 1 MSS $\rightarrow$ `cwnd` **doubles every RTT (Exponential growth)**.
   * Continues until `cwnd` hits the Slow Start Threshold (`ssthresh`).
2. **Congestion Avoidance (Additive Increase):**
   * Once `cwnd >= ssthresh`, growth switches from exponential to linear.
   * Increases `cwnd` by 1 MSS per RTT ($\text{AIMD}$ - Additive Increase, Multiplicative Decrease).
3. **Loss Event Detection:**
   * **Case A: Triple Duplicate ACKs (Fast Retransmit):**
     * Indicates 1 packet was lost, but subsequent packets arrived safely.
     * Retransmit the missing packet immediately without waiting for timeout.
     * Cut `ssthresh = cwnd / 2`, set `cwnd = ssthresh` (Fast Recovery - TCP Reno).
   * **Case B: Retransmission Timeout (RTO):**
     * Indicates severe network collapse.
     * Set `ssthresh = cwnd / 2`, collapse `cwnd = 1 MSS`, and drop back to Slow Start!

---

## 4. Network Layer (IP & Routing)

### IPv4 vs. IPv6

| Dimension | IPv4 | IPv6 |
| :--- | :--- | :--- |
| **Address Length** | 32 bits (4 bytes) | 128 bits (16 bytes) |
| **Address Space** | $\approx 4.3 \times 10^9$ ($4.3$ billion addresses) | $\approx 3.4 \times 10^{38}$ (Undepletable) |
| **Format** | Dotted Decimal: `192.168.1.1` | Hexadecimal: `2001:0db8:85a3::8a2e:0370:7334` |
| **Header Size** | 20 to 60 bytes (Variable) | Fixed 40 bytes (Faster router parsing) |
| **Checksum in Header** | Yes (Routers recompute on every hop) | Removed (Offloaded to L2 & L4 for speed) |
| **NAT Required?** | Yes (Workaround for address exhaustion)| No (Every device gets a global IP) |

---

### Subnetting, CIDR & IP Classes

#### Classful Addressing (Legacy):
- **Class A:** `0.0.0.0` - `127.255.255.255` (Subnet: `/8`, $16.7\text{M}$ hosts/net)
- **Class B:** `128.0.0.0` - `191.255.255.255` (Subnet: `/16`, $65\text{K}$ hosts/net)
- **Class C:** `192.0.0.0` - `223.255.255.255` (Subnet: `/24`, $254$ hosts/net)
- **Class D:** `224.0.0.0` - `239.255.255.255` (Multicast)
- **Class E:** `240.0.0.0` - `255.255.255.255` (Experimental)

#### Classless Inter-Domain Routing (CIDR):
Replaced classful addressing by allowing arbitrary prefix lengths denoted by `/n`.
- Address: `192.168.1.0/26`
- **Network Bits:** 26 bits. **Host Bits:** $32 - 26 = 6$ bits.
- **Total IP Addresses:** $2^6 = 64$ addresses.
- **Usable Host Addresses:** $2^6 - 2 = 62$ hosts *(First IP is Network ID; Last IP is Broadcast Address)*.

---

### NAT (Network Address Translation) & Private IP Ranges

Because IPv4 addresses are exhausted, RFC 1918 reserved 3 non-routable **Private IP blocks** for local networks:
1. `10.0.0.0` to `10.255.255.255` (`10.0.0.0/8`)
2. `172.16.0.0` to `172.31.255.255` (`172.16.0.0/12`)
3. `192.168.0.0` to `192.168.255.255` (`192.168.0.0/16`)

#### How PAT (Port Address Translation) Works:
Your home router has **1 public IP** assigned by your ISP (`203.0.113.5`).
- When Laptop (`192.168.1.10:51234`) requests Google, the router rewrites the source IP and port to its public IP and a unique port: `203.0.113.5:60001`.
- Router records this translation in its **NAT Translation Table**.
- When Google replies to `203.0.113.5:60001`, the router consults the table, translates it back to `192.168.1.10:51234`, and forwards it to the laptop.

---

### Routing Protocols

```
                          ROUTING PROTOCOLS
                                  |
        +-------------------------+-------------------------+
        |                                                   |
   Interior Gateway Protocols (IGP)               Exterior Gateway Protocols (EGP)
   (Inside an Autonomous System)                  (Between Autonomous Systems / ISPs)
        |                                                   |
   +----+--------------------+                              v
   |                         |                             BGP
Distance Vector         Link State                     (Path Vector Protocol;
(e.g., RIP)             (e.g., OSPF)                    Powers global Internet)
- Bellman-Ford Alg      - Dijkstra's SPF Alg
- Sends table to nbrs   - Floods link state to all
```

---

### ICMP (Ping & Traceroute Mechanics)
**ICMP (Internet Control Message Protocol)** is a network-layer protocol used by network devices for diagnostic reporting and error signaling.

#### How `ping` works:
Sends an `ICMP Echo Request (Type 8)` to target. Target responds with `ICMP Echo Reply (Type 0)`. Calculates round-trip time.

#### How `traceroute` works (Classic Interview Question):
Traceroute discovers all intermediate routers between source and destination using the **IP TTL (Time-To-Live)** field:
1. Sends packet with `TTL = 1`. First router decrements TTL to 0, drops packet, and sends back `ICMP Time Exceeded (Type 11)`. Sender learns Router 1's IP.
2. Sends packet with `TTL = 2`. Second router drops packet and sends back `ICMP Time Exceeded`. Sender learns Router 2's IP.
3. Repeats with incrementing TTLs until destination is reached (which returns `ICMP Port Unreachable` or Echo Reply).

---

### ARP (Address Resolution Protocol)
ARP bridges Layer 3 (IP) and Layer 2 (MAC). It answers: *"I know the destination IP address is 192.168.1.1, but what is its hardware MAC address?"*

```
1. Host A broadcasts ARP Request: "Who has 192.168.1.1? Tell 192.168.1.5"
   (Destination MAC: FF:FF:FF:FF:FF:FF - Broadcast to all switch ports)
2. Target Host B recognizes its IP, unicasts ARP Reply: "192.168.1.1 is at 00:1A:2B:3C:4D:5E"
3. Host A stores mapping in its local ARP Cache for future packets.
```

---

## 5. Data Link & Physical Layer Essentials

### MAC Address vs. IP Address

| Feature | MAC Address (Media Access Control) | IP Address (Internet Protocol) |
| :--- | :--- | :--- |
| **Layer** | Data Link Layer (Layer 2) | Network Layer (Layer 3) |
| **Nature** | Physical / Hardware address burned into NIC. | Logical / Software-assigned address. |
| **Portability** | Permanent (does not change when moving cities).| Dynamic (changes based on network connection).|
| **Scope** | Local Area Network (hop-to-hop routing). | Global End-to-End delivery. |
| **Structure** | 48 bits (6 bytes): `00:1A:2B:3C:4D:5E` | 32 bits (IPv4) or 128 bits (IPv6) |

---

### Network Devices: Hub vs. Switch vs. Router vs. Gateway

```
+-------------------------------------------------------------------------------------------------+
|                                 NETWORK DEVICES COMPARISON                                      |
+-------------------+-------------+-------------------+--------------------+----------------------+
| Device            | OSI Layer   | Operating Address | Collision Domain   | Broadcast Domain     |
+-------------------+-------------+-------------------+--------------------+----------------------+
| **Hub**           | Layer 1     | None (Raw bits)   | 1 Shared (Severe   | 1 Shared             |
|                   | (Physical)  |                   | collisions!)       |                      |
| **Switch**        | Layer 2     | MAC Address       | **Each port is an  | 1 Shared             |
|                   | (Data Link) | (MAC Table)       | isolated domain**  |                      |
| **Router**        | Layer 3     | IP Address        | Each port is an    | **Each port is an    |
|                   | (Network)   | (Routing Table)   | isolated domain    | isolated domain**    |
| **Gateway**       | Layers 4-7  | Protocol Transl-  | N/A                | N/A                  |
|                   | (App/Trans) | ation (e.g. VoIP) |                    |                      |
+-------------------+-------------+-------------------+--------------------+----------------------+
```

> **Interview Golden Rule:**
> - Switches break up **Collision Domains**, but forward broadcasts.
> - Routers break up **both Collision Domains and Broadcast Domains**.

---

## 6. The Ultimate Interview Question: "What happens when you type `https://www.google.com` in your browser and press Enter?"

*This is the single most common holistic interview question for SDE-1 and SWE Intern roles. Here is the complete, high-scoring 10-step sequence:*

```
[ BROWSER ] 
    | 1. URL Parsing & HSTS Check (forces HTTPS)
    | 2. DNS Resolution:
    |    Browser Cache -> OS Cache -> /etc/hosts -> Router Cache -> ISP Resolver -> Root -> TLD -> Authoritative
    v 
[ IP: 142.250.190.46 ]
    | 3. ARP Request (Find Gateway Router MAC address on local Wi-Fi)
    | 4. TCP 3-Way Handshake (SYN -> SYN-ACK -> ACK) to Port 443
    | 5. TLS 1.3 Handshake (Cert validation + Session Key generation)
    | 6. HTTP GET Request sent (Encrypted via symmetric key)
    v
[ INTERNET ROUTING (BGP / OSPF / NAT traversal across optical fiber) ]
    v
[ GOOGLE INFRASTRUCTURE ]
    | 7. Anycast IP routes packet to nearest Google Edge Data Center
    | 8. Reverse Proxy / Load Balancer (Nginx / Envoy / Maglev) terminates TLS
    | 9. Web Server processes request -> Queries databases/caches -> Generates HTML
    v
[ BROWSER RENDERING ]
    | 10. Browser receives HTTP 200 OK
    | 11. Parses HTML -> Constructs DOM Tree
    | 12. Parses CSS -> Constructs CSSOM Tree -> Combines into Render Tree
    | 13. Layout & Painting -> Fetches external assets (JS, images, fonts)
```

---

## 7. Network Security Fundamentals

### Symmetric vs. Asymmetric Encryption

- **Symmetric Encryption (Single Secret Key):**
  * Same key encrypts and decrypts.
  * *Algorithms:* AES-128, AES-256, ChaCha20.
  * *Pros:* Extremely fast; minimal CPU overhead.
  * *Con:* Key Distribution Problem (How do you share the key across the internet safely?).
- **Asymmetric Encryption (Public/Private Key Pair):**
  * Public key encrypts; Private key decrypts.
  * *Algorithms:* RSA, ECC (Elliptic Curve Cryptography), Diffie-Hellman.
  * *Pros:* Eliminates key distribution problem.
  * *Con:* 100x - 1000x slower computationally than symmetric.

---

### Hashing vs. Encryption vs. Encoding

```
+--------------------------------------------------------------------------+
|                  HASHING vs. ENCRYPTION vs. ENCODING                     |
+-------------------+----------------+----------------+--------------------+
| Dimension         | Hashing        | Encryption     | Encoding           |
+-------------------+----------------+----------------+--------------------+
| Reversibility     | **One-Way**    | **Two-Way**    | **Two-Way**        |
|                   | (Irreversible) | (Reversible    | (Reversible by     |
|                   |                | with key)      | anyone without key)|
| Purpose           | Data Integrity,| Confidentiality| Data formatting &  |
|                   | Password verify| & Privacy      | transmission       |
| Key Required?     | No             | Yes            | No                 |
| Output Length     | Fixed length   | Proportional to| Proportional to    |
|                   | (SHA-256: 256b)| input length   | input length       |
| Examples          | SHA-256, bcrypt| AES, RSA       | Base64, URL-encode |
+-------------------+----------------+----------------+--------------------+
```

---

### CORS (Cross-Origin Resource Sharing)
A browser-enforced security mechanism based on the **Same-Origin Policy (SOP)**.
- **Origin Definition:** Scheme + Domain + Port (e.g., `https://api.example.com:443`).
- If web app on `http://localhost:3000` fetches from `https://api.mybackend.com`:
  1. Browser intercepts request and sends a **Preflight Request (`OPTIONS`)** with header `Origin: http://localhost:3000`.
  2. Server responds with `Access-Control-Allow-Origin: http://localhost:3000` (or `*`).
  3. If missing or mismatched, **the browser blocks the response** from client JavaScript!

---

## 8. Top 20 SDE-1 / Intern Interview Questions & Model Answers

### Q1: Why does TCP use a 3-way handshake instead of a 2-way handshake?
**Answer:** A 2-way handshake is insufficient because it only verifies the client's ability to communicate to the server and the server's ability to receive. It cannot confirm that the client is able to receive communications from the server, nor can it ensure that both sides have agreed on the initial sequence numbers. Furthermore, delayed duplicate SYN packets from old, dead connections could arrive at the server; in a 2-way model, the server would open a bogus connection, wasting resources.

---

### Q2: Why does TCP teardown require 4 steps instead of 3?
**Answer:** TCP is a full-duplex protocol. When the client sends a `FIN`, it indicates that the client has no more data to send, but it can still receive data. The server acknowledges this (`ACK`), but the server may still have pending buffered data to flush. Only when the server has finished transmitting all its remaining data does it send its own `FIN`, which the client acknowledges (`ACK`).

---

### Q3: What is the `TIME_WAIT` state, and why is it set to $2 \times \text{MSL}$?
**Answer:** `TIME_WAIT` is entered by the active closer of a TCP connection. It waits for twice the Maximum Segment Lifetime ($2 \times \text{MSL}$, typically 1–2 minutes) for two reasons: (1) To ensure the final `ACK` reached the peer; if lost, the peer retransmits `FIN`, which can be re-acknowledged. (2) To allow all stray, delayed packets from this connection to expire in internet router queues so they do not corrupt a future connection reusing the same local/remote IP and port tuple.

---

### Q4: How does HTTPS establish a secure connection?
**Answer:** HTTPS uses TLS. The client sends a `ClientHello` with supported cipher suites. The server responds with `ServerHello`, its chosen cipher suite, and its digital certificate. The client verifies the certificate against trusted Root Certificate Authorities. Using asymmetric cryptography (Diffie-Hellman), both sides negotiate a shared secret without transmitting the key over the wire. They then use this secret to derive symmetric encryption keys (AES) for all subsequent application data.

---

### Q5: What is the difference between Flow Control and Congestion Control in TCP?
**Answer:** Flow Control prevents a fast sender from overwhelming a slow **receiver**; it is governed by the receiver-advertised `rwnd` (receive window) in the TCP header. Congestion Control prevents a fast sender from overwhelming the **intermediate network routers**; it is governed by `cwnd` (congestion window), calculated dynamically by the sender using algorithms like Slow Start and AIMD. The actual transmission rate is $\min(\text{rwnd}, \text{cwnd})$.

---

### Q6: What is Head-of-Line (HoL) Blocking, and how does HTTP/3 solve it?
**Answer:** In HTTP/1.1, HoL blocking occurs because only one request/response can use a TCP connection at a time. HTTP/2 solved this at the application layer with multiplexing, but retained **Transport-layer HoL blocking**: because all streams share one TCP connection, a single dropped packet causes TCP to pause delivery of all streams. HTTP/3 solves this by switching from TCP to **QUIC over UDP**, where each stream is tracked independently; packet loss on one stream does not stall other streams.

---

### Q7: Why does DNS primarily use UDP instead of TCP?
**Answer:** DNS queries are lightweight (a single request and response packet). UDP avoids the latency overhead of a 3-way TCP handshake and teardown, making lookups much faster. However, DNS falls back to TCP when the response exceeds 512 bytes (or EDNS0 limits) or during **DNS Zone Transfers** between primary and secondary name servers where reliable byte-stream transfer is required.

---

### Q8: What is a SYN Flood attack, and how is it mitigated?
**Answer:** An attacker floods a server with TCP `SYN` packets from spoofed IPs and ignores the `SYN-ACK` replies. The server allocates TCB (Transmission Control Block) memory in its `SYN_RCVD` queue for each half-open connection until memory is exhausted, denying legitimate users. It is mitigated using **SYN Cookies**, where the server does not allocate memory on receiving `SYN`; instead, it encodes connection parameters into the sequence number ($Y$) sent in `SYN-ACK` and only allocates resources when the final `ACK` arrives with the valid hash.

---

### Q9: What is the purpose of the Subnet Mask?
**Answer:** A Subnet Mask is a 32-bit number used to divide an IP address into its **Network component** and **Host component**. By performing a bitwise AND operation between an IP address and its subnet mask (`IP & SubnetMask`), a device determines whether the destination IP is on the same local subnet (deliver via Layer 2 ARP/switch) or on an external network (forward to default gateway router).

---

### Q10: How does `traceroute` discover the path to a destination?
**Answer:** Traceroute sends packets with deliberately small TTL (Time-To-Live) values starting at 1. Each intermediate router decrements TTL by 1. When a router decrements TTL to 0, it drops the packet and sends back an `ICMP Time Exceeded` packet, revealing the router's IP address. Traceroute increments TTL by 1 successively until reaching the destination.

---

### Q11: What is the difference between a Switch and a Router?
**Answer:** A Switch operates at Layer 2 (Data Link) using MAC addresses to forward frames within a single Local Area Network; each port is its own collision domain, but all ports share a single broadcast domain. A Router operates at Layer 3 (Network) using IP addresses to route packets between different networks; each router interface separates both collision domains and broadcast domains.

---

### Q12: What is the difference between asymmetric and symmetric encryption? Why do we need both in TLS?
**Answer:** Symmetric encryption uses the same key for encryption and decryption; it is computationally fast but suffers from the key distribution problem. Asymmetric encryption uses a public/private key pair; it solves key distribution but is computationally expensive. TLS combines both: asymmetric cryptography is used during the handshake to authenticate identity and establish a shared session key, after which symmetric encryption is used for high-speed data transmission.

---

### Q13: What is the difference between WebSockets and HTTP Long Polling?
**Answer:** HTTP Long Polling keeps an HTTP request hanging until new data is available on the server; once data is sent, the connection closes and a new HTTP request must be initiated with fresh headers. WebSockets start with an HTTP upgrade handshake, but transition into a persistent, bidirectional, full-duplex TCP connection with a minimal 2-byte framing overhead, allowing simultaneous real-time communication.

---

### Q14: What is an Inverted Page Table vs. Translation Lookaside Buffer?
*(Note: Common cross-subject check)*
**Answer:** A TLB is a CPU hardware cache storing recent virtual-to-physical address mappings. An Inverted Page Table is a memory management data structure indexed by physical frame number rather than virtual page number, keeping only one entry per physical frame in RAM to minimize page table memory overhead in 64-bit systems.

---

### Q15: What is the difference between Private and Public IP addresses?
**Answer:** Public IP addresses are globally unique and routable across the public internet, assigned by IANA/ISPs. Private IP addresses (defined in RFC 1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) are non-routable on the internet and are used inside private LANs. Devices with private IPs communicate with the internet through Network Address Translation (NAT) on a border router.

---

### Q16: What is ARP and what problem does it solve?
**Answer:** Address Resolution Protocol (ARP) translates a Layer 3 logical IP address into a Layer 2 physical MAC address. When a host needs to deliver a frame to an IP on the local subnet, it broadcasts an ARP request (`"Who has IP X?"`); the matching device unicasts an ARP reply with its MAC address, which the sender caches in its ARP table.

---

### Q17: What is the difference between HTTP Status Codes 301 and 302?
**Answer:**
- **301 Moved Permanently:** Target resource has permanently moved to a new URI. Browsers and search engines cache this redirection and update bookmarks/SEO rankings to point to the new URL.
- **302 Found (Temporary Redirect):** Target resource resides temporarily at a different URI. Clients follow the redirect, but search engines and browsers do not cache the redirect long-term.

---

### Q18: What is MTU (Maximum Transmission Unit) and MSS (Maximum Segment Size)?
**Answer:** MTU is the largest packet size (in bytes) that can be transmitted over a physical network medium without fragmentation (standard Ethernet MTU is 1500 bytes). MSS is the maximum amount of pure TCP application data (payload) that can be packed into a single segment; typically $\text{MSS} = \text{MTU} - (\text{IP Header: } 20\text{B} + \text{TCP Header: } 20\text{B}) = 1460$ bytes.

---

### Q19: What is the difference between Gateway and Router?
**Answer:** A Router is a Layer 3 device that routes packets between distinct networks sharing the same network protocols. A Gateway is a broader concept that can operate across all layers (Layers 4 to 7), actively converting and translating between fundamentally incompatible network architectures and communication protocols (e.g., converting a cellular voice protocol to VoIP SIP).

---

### Q20: What is the difference between cookies, localStorage, and sessionStorage?
**Answer:**
- **Cookies:** Small ($\approx 4\text{KB}$) storage automatically sent with every HTTP request to the server in the `Cookie` header; supports `HttpOnly` and `Secure` flags for auth tokens.
- **localStorage:** Larger ($\approx 5–10\text{MB}$) client-side key-value storage that persists indefinitely until cleared explicitly via JavaScript or user action; never sent automatically in HTTP headers.
- **sessionStorage:** Same API as localStorage, but data is scoped strictly to the current browser tab and is cleared immediately when the tab or window is closed.
