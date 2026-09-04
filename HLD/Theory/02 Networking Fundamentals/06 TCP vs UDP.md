Every networked application has to move data from one program to another.

Most of that data moves through one of two transport protocols: TCP or UDP.

Both sit above IP. IP gets packets to the right machine. TCP and UDP help get the data to the right program on that machine by using ports.

The difference is in what help they provide:

TCP gives you a reliable, ordered stream of bytes.
UDP gives you small independent messages called datagrams, but it does not promise delivery or order.
So the real design question is:

What should the application do when data is lost, late, duplicated, or arrives out of order?

Modern systems add one more twist: QUIC, the transport behind HTTP/3, runs on top of UDP but adds reliability, encryption, and congestion control itself. We will cover that too.


1. The Transport Layer
IP gets packets toward the right machine. The transport layer gets data to the right program on that machine.

Transport protocols may provide:

Ports: Tell the operating system which program should receive the data. For example, HTTPS commonly uses port 443, and PostgreSQL commonly uses port 5432.
Segmentation: Split large data into smaller pieces that fit on the network.
Reassembly: Put received pieces back together.
Reliability: Notice when data is missing and resend it.
Ordering: Give data to the application in the right order.
Flow control: Stop a fast sender from overwhelming a slow receiver.
Congestion control: Slow down when the network path looks overloaded.
TCP provides most of this automatically.

UDP provides ports, length, and checksums. It leaves delivery, ordering, pacing, and recovery to the application or to another protocol built on top of UDP.




Application Protocol
HTTP, DNS, gRPC, WebRTC

TCP
Reliable ordered byte stream

UDP
Independent datagrams

IP
Addressing and routing

Network

Picking TCP or UDP decides what the operating system handles for you and what your application must handle itself.


2. TCP
TCP (Transmission Control Protocol) is connection-oriented.

Before normal application data flows, the client and server first agree to open a connection. After that, TCP gives the application a stream of bytes.

TCP is a stream, not a message protocol.

If an application writes three messages, the receiver may read them as one combined chunk, three chunks, or several partial chunks. TCP keeps the bytes in order, but it does not remember your message boundaries.

That means the application protocol must define where one message ends and the next begins. Common approaches include length prefixes, delimiters, or structured formats.

TCP is a good fit when the application needs all bytes delivered in order and would rather wait than process stale or incomplete data.

What TCP Provides
TCP provides:

Connection setup: A three-way handshake opens the connection and agrees on sequence numbers.
Ordered delivery: Bytes are delivered to the application in sequence.
Retransmission: Lost segments are resent.
Duplicate suppression: Duplicate data is detected through sequence numbers.
Flow control: The receiver advertises how much data it can accept.
Congestion control: The sender adjusts its rate to avoid overloading the network.
Backpressure: A slow receiver or congested path eventually slows the sender.
TCP does not guarantee that a business operation succeeded.

It only guarantees reliable, ordered delivery of bytes while the connection stays healthy. If a server commits a database transaction but the connection breaks before the client receives the response, the client still does not know what happened.

That is why real systems still need timeouts, retries, idempotency keys, and duplicate handling.

Connection Setup
TCP uses a three-way handshake:




TCP connection established
SYN (seq=x)
SYN-ACK (seq=y, ack=x+1)
ACK (seq=x+1, ack=y+1)
Client
Server




4 / 4
SYN: The client asks to open a connection and sends an initial sequence number.
SYN-ACK: The server acknowledges the client sequence number and sends its own.
ACK: The client acknowledges the server sequence number.
After that, both sides can exchange data.

The handshake costs at least one round trip before normal application data flows.

In practice, systems reduce that cost with connection reuse, TLS 1.3, TCP Fast Open in limited environments, or QUIC.

Data Transfer
During transfer, TCP tracks byte positions with sequence numbers. The receiver confirms what it has received. If something is missing, the sender sends it again.




Receiver delivers bytes to the application in order
Segment seq=1000 len=500
Segment seq=1500 len=500
ACK 2000
Segment seq=2000 len=500
Client
Server




5 / 5
The cost of this reliability is waiting.

If one TCP segment is lost, later bytes may already be sitting in the receiver's buffer. But the application cannot receive those later bytes until the missing earlier bytes arrive.

This is called head-of-line blocking at the TCP stream level.

For many systems, that waiting is exactly what you want. A SQL result, an HTTP response body, or a file download is usually useless if bytes are missing or out of order.

Connection Close
TCP connections can close gracefully with FIN packets or abruptly with RST packets.




Graceful close with FIN
Connection closed
Abrupt close with RST
Connection torn down immediately
FIN
ACK
FIN
ACK
RST
Client
Server




9 / 9
A graceful close means each side has finished sending bytes. It does not mean the business operation succeeded.

Applications still need clear success responses and safe state handling.

Where TCP Fits
TCP is the default for:

HTTP/1.1 and HTTP/2
Traditional HTTPS
SSH
SMTP, IMAP, and many mail flows
Database connections such as PostgreSQL and MySQL
Most internal RPC systems, including standard gRPC over HTTP/2
Message brokers and queues that require ordered byte streams
For request-response APIs, admin tools, database protocols, and most service-to-service calls, TCP is still the boring and correct choice.


3. UDP
UDP (User Datagram Protocol) is connectionless.

It sends independent datagrams without opening a transport connection first.

UDP does not provide reliable delivery, ordering, retransmission, flow control, or congestion control by itself.

A UDP datagram may arrive, arrive late, arrive twice, arrive out of order, or never arrive.

That tradeoff is intentional. For real-time systems, waiting for old data can be worse than dropping it and moving on.

What UDP Provides
UDP provides:

Ports: Source and destination program identifiers.
Datagram boundaries: One send maps to one datagram at the UDP layer.
Length: The receiver knows the datagram size.
Checksum: Corruption detection. The UDP checksum is mandatory in IPv6 and optional in IPv4, though commonly used.
No connection setup: The sender can transmit immediately.
UDP's header is small: 8 bytes. TCP's base header is 20 bytes before options.

Header size can matter, but the bigger difference is behavior: UDP does not make the sender wait for acknowledgments or retransmissions.

How UDP Works
An application creates a datagram and sends it to a destination IP and port.

If a program is listening there and the network delivers the datagram, the receiver can process it. If the datagram is lost, UDP does not recover it.




UDP does not retransmit or reorder
Datagram 1
Datagram 2
Datagram 4
Sender
Receiver
Sender
Receiver




4 / 4
Applications that use UDP responsibly usually add the pieces they need:

Sequence numbers to detect loss or reordering
Timestamps to discard stale data
Application-level acknowledgments for important messages
Forward error correction for media
Rate control so they do not flood the network
Encryption through DTLS, SRTP, or QUIC
UDP is not permission to ignore the network.

A high-volume UDP system that sends faster than the network can carry will cause packet loss, hurt other traffic, and usually hurt itself too.

Where UDP Fits
UDP is commonly used for:

DNS queries
Real-time voice and video
Online games
WebRTC media
QUIC and HTTP/3
Service discovery and local network protocols
Telemetry where occasional loss is acceptable
UDP is a good fit when fresh data matters more than complete delivery, or when a higher-level protocol adds the missing behavior itself.


4. TCP vs UDP
TCP vs UDP, Reliability Tradeoff
25% loss. TCP retransmits and buffers. UDP leaves gaps.

Stable
Lossy
Reordered
Congested

TCP
reliable, ordered, ACKed
sent 0
retries 0
unacked 0
goodput 0%
Sender
unacked window
empty
ACK
Receiver
buffer
empty
delivered
none
UDP
best-effort, unordered, no retry
sent 0
lost 0
loss 0%
Sender
no window
fire & forget
Receiver
arrival order
none
no packets yet
Idle, press Play to begin



0 / 18
1x

Feature	TCP	UDP
Basic model	Ordered byte stream	Independent datagrams
Connection setup	Three-way handshake	No transport handshake
Reliability	Retransmits lost data while connection is healthy	No built-in retransmission
Ordering	Delivers bytes in order	No ordering guarantee
Message boundaries	Not preserved	Preserved per datagram
Flow control	Built in	Not built in
Congestion control	Built in	Not built in
Head-of-line blocking	Yes, within the TCP stream	Not at UDP layer
Typical protocols	HTTP/1.1, HTTP/2, SSH, databases, SMTP	DNS, QUIC, WebRTC media, games
The simplest rule:

Use TCP when the application needs a complete ordered stream.

Use UDP when the application can tolerate some loss, needs low-latency datagrams, or uses a protocol such as QUIC that adds reliability and congestion control on top.

Avoid the shortcut "TCP is slow and UDP is fast."

TCP can be very fast on healthy networks. UDP can perform badly if the application handles packet loss, pacing, or packet size poorly.


5. QUIC and HTTP/3
Modern TCP-vs-UDP discussions need to include QUIC.

QUIC is a transport protocol that runs inside UDP datagrams. It was originally developed at Google and later standardized by the IETF.

HTTP/3 is HTTP running over QUIC.

QUIC uses UDP because UDP is already supported by most networks. It also lets QUIC implement its own rules for reliability, ordering, and congestion control instead of depending on the operating system's TCP stack.

QUIC provides:

Connection setup with built-in TLS 1.3
Encryption by default
Reliability and retransmission
Congestion control
Flow control
Multiplexed streams
Connection migration when a client changes networks, such as moving from WiFi to cellular



HTTP/3

QUIC
streams, TLS, recovery, congestion control

UDP

IP

QUIC avoids one important TCP limitation for protocols that carry many streams at once.

In HTTP/2 over TCP, many streams share one TCP connection. If one TCP segment is lost, all streams behind that missing byte can be blocked at the TCP layer.

QUIC has independent streams, so loss on one stream does not block unrelated streams in the same way.

QUIC is not always better. Some networks block or degrade UDP. Some older network devices are harder to use with QUIC. Teams still need monitoring, fallback to TCP-based HTTP, and a careful rollout.


6. Choosing Between TCP, UDP, and QUIC
Start with what the application needs, not with protocol fashion.

Choose TCP When
TCP is usually right when every byte matters and the data must be processed in order.

It is also the safer default when the application protocol is already built around streams, or when you want mature behavior through firewalls, proxies, and company networks.

Use TCP for common protocols such as HTTP/1.1, HTTP/2, SSH, PostgreSQL, MySQL, SMTP, or standard gRPC.

This describes most day-to-day backend traffic: payment API calls, database transactions, file uploads, internal gRPC calls, and admin SSH sessions.

Choose UDP When
UDP is usually right when fresh data is more valuable than complete old data, and the application can tolerate loss or repair it itself.

It also fits when you need datagram boundaries, when you are using an existing UDP-based protocol, or when your application controls pacing, retries, and congestion behavior.

Real-time voice and video, game state updates, DNS lookups, local discovery protocols, and telemetry where occasional loss is acceptable all match this pattern.

Choose QUIC or HTTP/3 When
QUIC or HTTP/3 can be a strong fit when you want HTTP over a modern encrypted transport and connection setup time matters.

It also helps when clients move between networks, such as mobile users switching from WiFi to cellular, or when many streams suffer from TCP head-of-line blocking.

The practical requirement is that you can run UDP on port 443 reliably and fall back to TCP-based HTTP when needed.

These properties can pay off for large web platforms, mobile APIs, streaming AI responses, latency-sensitive edge APIs, and systems that benefit from HTTP/3 but can fall back to HTTP/2.


7. Production Design Considerations
Protocol choice affects reliability, capacity, debugging, and operations.

Timeouts and Retries
TCP resends missing bytes. It does not retry application operations.

A client that times out after sending a request may not know whether the server processed it.

This is why APIs that change state need idempotency keys, request IDs, deduplication, or clear retry rules.

For UDP systems, retry behavior must be designed by the application. DNS retries are different from game updates, and both are different from media recovery.

Packet Size and MTU
MTU means the largest packet size a network path can carry without splitting it up.

Large packets are more likely to be split into fragments or dropped.

Fragmentation is especially painful for UDP because losing one fragment loses the whole datagram.

Practical guidance:

Keep UDP datagrams comfortably below common path MTUs unless the protocol handles discovery and fragmentation carefully.
Do not assume jumbo frames exist outside controlled networks.
For TCP, the operating system handles segmentation, but path MTU problems can still cause stalls.
Load Balancing
TCP load balancers usually assign a connection to a backend and keep that connection stable.

UDP is trickier because there may be no real connection at the transport layer.

Load balancers often group UDP traffic using source IP, source port, destination IP, destination port, and protocol. NAT changes, mobile network changes, and short-lived datagrams can affect where traffic goes.

QUIC has connection IDs that help keep a connection together even when the client IP or port changes, if the infrastructure supports it correctly.

Monitoring
TCP gives operators familiar signals: connection counts, resets, retransmits, SYN backlog, accept queue, connection duration, and socket errors.

UDP systems need protocol-specific metrics:

Datagrams sent and received
Estimated loss
Jitter, which means delay variation between packets
Reordering, which means packets arriving in a different order than they were sent
Application-level acknowledgments
Dropped packets at socket buffers
Rate limiting and pacing behavior
For QUIC and HTTP/3, expose handshake failures, fallback rates, stream resets, congestion metrics, and UDP reachability.

Security
Neither TCP nor UDP automatically makes an application secure.

TCP applications commonly use TLS.
UDP applications can use DTLS, SRTP, WireGuard-style protocols, or QUIC's built-in TLS 1.3.
UDP services need care around spoofing and amplification attacks.
Public UDP endpoints should enforce rate limits and validate clients before sending large responses.
Security comes from the full protocol stack and operational controls, not from choosing TCP or UDP alone.


8. Real-World Examples
The choice between TCP and UDP shows up in many different systems.

The same question applies each time: what should happen when data is late or lost?

Two systems can make opposite choices from that question. A delayed database response may be a small stall. A delayed audio packet may be useless.

Web and APIs
HTTP/1.1 and HTTP/2 commonly run over TCP with TLS. HTTP/3 runs over QUIC over UDP.

A mature web platform often supports HTTP/2 and HTTP/3, measures performance, and falls back cleanly when UDP is blocked.

Databases
Databases generally use TCP because queries, results, transactions, and replication streams need reliable ordered bytes.

The harder design problems are connection pooling, timeouts, transaction retries, and backpressure.

DNS
DNS traditionally uses UDP for small queries because it is simple and low latency.

DNS can also use TCP. Modern encrypted DNS options include DNS over TLS, DNS over HTTPS, and DNS over QUIC. The right choice depends on response size, privacy needs, deployment environment, and resolver support.

Real-Time Media
Voice and video systems usually prefer timely delivery over perfect delivery. A late audio packet is often useless.

These systems use jitter buffers, codecs, packet loss concealment, forward error correction, and congestion control to keep the session usable.

Online Games
Games often send frequent state updates over UDP.

If a player position update is lost, the next update may replace it. Critical events, such as inventory changes or purchases, still need reliable application-level handling or a separate reliable channel.

AI Systems
Most AI APIs use TCP-based HTTPS because request correctness, authentication, and broad compatibility matter.

Streaming tokens can use HTTP chunking, Server-Sent Events, WebSockets, gRPC streaming, or HTTP/3 depending on the client and platform.

Internal AI infrastructure may use TCP or gRPC for control-plane calls and model metadata. It may use specialized streaming or UDP-based protocols for real-time media input, low-latency interactive experiences, or telemetry where occasional loss is acceptable.


Summary
TCP and UDP are transport-layer tools with different promises.

TCP gives a reliable, ordered byte stream. It keeps bytes in order, but it does not preserve application message boundaries, and it does not prove that a business operation succeeded.

UDP gives independent datagrams. It preserves datagram boundaries, but it does not promise delivery or order.

TCP includes flow control and congestion control. UDP applications must add their own pacing, reliability, and recovery when they need those features.

QUIC runs over UDP and adds encryption, streams, recovery, congestion control, and connection migration.

The best design question is not "Which one is faster?" It is:

What should the application do when data is late, lost, duplicated, reordered, or processed twice?