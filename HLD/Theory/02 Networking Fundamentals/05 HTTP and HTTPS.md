Almost everything a modern system does over the network uses HTTP somewhere.

HTTP (Hypertext Transfer Protocol) is the request-response protocol behind websites, public APIs, mobile backends, and a lot of internal service-to-service calls.

HTTPS is HTTP protected by TLS (Transport Layer Security). TLS encrypts the connection and lets the client check that it is talking to the right server. In production, HTTPS is the normal default.

Plain HTTP is mostly limited to local development, tightly controlled internal health checks, and redirects that send clients to the HTTPS version of a URL. Anything user-facing or security-sensitive should use HTTPS.

The HTTP ideas stay the same across versions: methods, status codes, headers, and bodies. What changes is the transport underneath. HTTP/1.1 and HTTP/2 usually run over TCP. HTTP/3 runs over QUIC, which uses UDP.

This chapter explains how HTTP works, what HTTPS adds, and why the HTTP version matters in real systems.

TLS 1.3 Handshake, One Round Trip to Encrypted
Watch a client and server agree on keys, verify a certificate, and switch to an encrypted channel.


Client
Server
plaintext
Client
your-browser
Server
example.com:443
round trips: 0.0
Handshake state
Plaintext
Negotiated
not negotiated yet
Server certificate
not received yet
Key schedule
client key share + server key share
->
ECDHE shared secret
->
handshake keys
->
application keys
keys are derived on both sides, never sent on the wire
Idle, press Play to begin



0 / 7
1x


1. What HTTP Is
HTTP (Hypertext Transfer Protocol) is a protocol for exchanging requests and responses. A client sends a request. A server returns a response.

HTTP defines the meaning of common request and response parts:

Methods such as GET, POST, PUT, and DELETE
Status codes such as 200, 404, 429, and 503
Headers such as Content-Type, Authorization, Cache-Control, and ETag
Message bodies such as HTML, JSON, images, protobuf, or event streams
Caching, conditional requests, redirects, content negotiation, and authentication hooks
HTTP is not tied to one network transport forever. HTTP/1.1 and HTTP/2 commonly run over TCP. HTTP/3 runs over QUIC, which runs over UDP.

HTTP/1.1 messages are text-based and easy to read. HTTP/2 and HTTP/3 use binary frames, which are more efficient for machines but less pleasant to debug by eye. The core HTTP meaning still stays the same: request method, path, headers, status, and body.


2. HTTP Request and Response
A typical HTTP exchange has three parts:

A request line, or the equivalent fields in newer HTTP versions
Headers
An optional body
Request:


Response:


The request target identifies the resource, such as /v1/models. Headers carry extra information, such as the content type or authorization token. The body carries data when the method allows one, such as JSON for a POST.

HTTP/2 and HTTP/3 do not send this exact text format on the wire, but the same concepts remain: method, scheme, host, path, headers, status, and body.


3. How HTTP Works
At a high level, an HTTP request follows this path:




Connection may be reused for later requests
Resolve api.example.com
IP address
TCP connect
TLS handshake (HTTPS)
HTTP request
Route to application
HTTP response
Client
DNS Resolver
Server




8 / 8
The client resolves the hostname through DNS.
The client opens or reuses a transport connection.
For HTTPS, the client and server perform a TLS handshake.
The client sends an HTTP request.
The server routes the request to application code or another internal service.
The server sends an HTTP response.
The connection may be reused for later requests.
This simple flow often hides many production components: CDNs, WAFs, API gateways, reverse proxies, service meshes, load balancers, sidecars, and application servers.

Those components can read or change HTTP headers only after the traffic has been decrypted. That point is called TLS termination. Before that point, HTTPS traffic is protected on the wire.

Stateless Protocol, Stateful Systems
HTTP is stateless at the protocol level. That means each request should carry enough information for the server to understand it. HTTP itself does not assume that the server remembers a hidden session from the previous request.

That does not mean web systems are stateless. Real systems keep state in cookies, session stores, OAuth access tokens, JWTs, CSRF tokens, server-side caches, and databases. They may also use sticky load balancing or connection pools to keep requests near useful state.

The important design question is simple: where does the state live, and what happens if a request is retried, routed to another server, or finishes after the client has timed out?


4. Methods and Status Codes
HTTP methods tell clients, proxies, gateways, and retry logic what kind of operation is being requested.

Method	Common Use	Safe	Idempotent
GET	Read a resource	Yes	Yes
HEAD	Read response headers only	Yes	Yes
POST	Create a resource or start a command	No	No by default
PUT	Replace or create a resource at a known URL	No	Yes
PATCH	Partially update a resource	No	Not guaranteed
DELETE	Delete a resource	No	Yes
Safe means the client is asking to read, not change, server state. Idempotent means repeating the same request should have the same intended effect as sending it once.

Idempotency is not trivia. It decides whether clients can safely retry after timeouts, connection resets, and load balancer failures. POST /payments should usually require an idempotency key. GET /orders/123 should not change state.

Status codes should be specific enough for clients and operators to act on:

Range	Meaning	Examples
1xx	Informational	100 Continue, 103 Early Hints
2xx	Success	200 OK, 201 Created, 204 No Content
3xx	Redirect or alternate location	301 Moved Permanently, 302 Found, 304 Not Modified
4xx	Client-side problem	400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests
5xx	Server-side or internal dependency problem	500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout
Use status codes as part of the API contract. A vague 500 for validation errors forces clients to guess. A 200 with an error object confuses caches, metrics, SDKs, and retry policies.


5. Caching and Conditional Requests
HTTP has mature caching rules. Used well, caching makes systems faster, reduces load on origin servers, lowers cloud cost, and can keep users working during partial failures.

Important headers include:

Header	Purpose
Cache-Control	Defines who may cache and for how long
ETag	Version-like value for a cached response
If-None-Match	Client asks whether an ETag is still current
Last-Modified	Timestamp used to check whether content changed
Vary	Tells caches which request headers affect the response
Content-Encoding	Indicates compression such as gzip, br, or zstd
For example, a client can avoid downloading an unchanged response:








12345
Caching is not only for browsers. CDNs, API gateways, package registries, feature stores, model metadata endpoints, and documentation sites all benefit from correct cache headers.

For personalized or sensitive responses, be explicit. Use restrictive settings such as Cache-Control: private or no-store where appropriate.


6. What HTTPS Adds
HTTPS is HTTP over TLS. TLS is the modern security protocol. SSL is old terminology and should not be used for new systems.

HTTPS provides three main protections:

Confidentiality: People or systems in the middle cannot read the protected HTTP data.
Integrity: People or systems in the middle cannot change protected traffic without being detected.
Server authentication: The client can check that the server is allowed to use the hostname.
HTTPS does not solve every security problem:

It does not authenticate the user unless the application adds authentication.
It does not make a vulnerable API safe.
It does not hide the destination IP address.
It may still expose the hostname through DNS and, depending on deployment, SNI.
It does not prevent a compromised endpoint from reading data.
SNI is the TLS field that lets a client say which hostname it wants before the encrypted HTTP request is sent. Modern encrypted-client-hello work can hide more of this in some deployments, but you should not assume HTTPS hides every routing hint.

HTTPS is the baseline expectation. Browsers warn on plain HTTP. Many platform features require a secure context. Production APIs should assume HTTPS from the first design review.


7. How TLS Works
The secure connection in HTTPS starts with a TLS handshake.

A handshake is the setup conversation where the client and server agree on how to protect the connection.




Verify certificate chain and hostname
Both sides create shared encryption keys
ClientHello (supported TLS options, SNI, ALPN)
ServerHello (chosen TLS options)
Certificate + signature
Encrypted HTTP request
Encrypted HTTP response
Client
Server




7 / 7
A modern TLS 1.3 handshake roughly does this:

ClientHello: The client says which TLS versions and encryption options it supports. It also sends information such as SNI and ALPN.
ServerHello: The server chooses the shared settings for this connection.
Certificate: The server sends certificates that prove it is allowed to serve the hostname.
Certificate verification: The client checks the hostname, expiration date, signatures, and trusted certificate authority chain.
Key setup: Both sides create shared encryption keys without sending the final keys over the network.
Encrypted HTTP: HTTP requests and responses now flow through the encrypted connection.
Older TLS explanations often describe a client encrypting a "pre-master secret" with the server's public key. That is not the normal TLS 1.3 model.

Modern TLS uses temporary key exchange, commonly ECDHE. The practical benefit is forward secrecy: even if a server's long-term private key is stolen later, old recorded connections should still be hard to decrypt.

TLS also uses ALPN (Application-Layer Protocol Negotiation) so the client and server can agree on the application protocol, such as http/1.1 or h2. HTTP/3 uses QUIC, where TLS 1.3 is built into the QUIC handshake.

0-RTT
TLS 1.3 and QUIC can support 0-RTT data for repeat connections. This lets a client send some data very early, which can reduce latency.

The tradeoff is replay risk: an attacker may be able to replay that early data. Use 0-RTT only for operations that are safe to repeat, such as idempotent reads.


8. HTTP vs HTTPS
Feature	HTTP	HTTPS
Protection	Plaintext	Encrypted and integrity-protected with TLS
Common Port	80	443
Server Authentication	None by default	Certificate-based hostname verification
Tamper Resistance	None	Protected by TLS
Browser Treatment	Marked insecure for many contexts	Required for most production web features
Production Use	Redirects, local development, tightly controlled internal cases	Default for web apps, APIs, mobile backends, and service endpoints
Do not choose plain HTTP for performance. TLS overhead is usually small compared with application work, database calls, network distance, and payload size.

Connection reuse, TLS 1.3, HTTP/2, HTTP/3, session resumption, and hardware acceleration have made HTTPS practical as the default.


9. HTTP Versions
HTTP has evolved without changing its core request-response model.

HTTP/1.1
HTTP/1.1 made reusable connections the default and added important behavior around hostnames, caching, content negotiation, range requests, and transfer encodings.

Operationally, HTTP/1.1 is simple and widely supported. Its weakness is concurrency. A single connection handles responses in order, so one slow response can hold up later responses on that connection. Clients often open multiple connections to the same origin to work around this.

HTTP pipelining exists in the specification, but it was difficult to deploy safely and never became normal browser behavior.

HTTP/2
HTTP/2 keeps the same HTTP meaning but changes how data is sent on the wire. Instead of text messages, it uses binary frames.

It adds:

Multiple streams over one TCP connection
Header compression with HPACK
Stream priorities, though real-world support has varied
Better connection reuse than HTTP/1.1
HTTP/2 reduces head-of-line blocking at the HTTP layer because multiple requests can be active on one connection.

It does not remove TCP-level head-of-line blocking. If one TCP segment is lost, all streams on that TCP connection may wait until the missing bytes are recovered.

In plain English: one missing TCP piece can make unrelated HTTP/2 streams wait, because they all share the same TCP connection.

HTTP/2 server push was part of the original design, but it proved difficult to use well and has been removed or disabled in major browsers. Do not design new systems around server push.

HTTP/3
HTTP/3 keeps the same HTTP meaning but runs over QUIC instead of TCP. QUIC runs over UDP and includes TLS 1.3, multiple streams, loss recovery, flow control, congestion control, and connection migration.

Connection migration means a connection can survive some network changes, such as a mobile client moving from Wi-Fi to cellular.

HTTP/3 helps with:

Reducing TCP-level head-of-line blocking between streams
Faster connection setup in some cases
Better behavior when mobile clients change networks
Faster transport changes because QUIC can evolve outside the operating system TCP stack
HTTP/3 is not automatically faster for every workload. Some networks block or degrade UDP. Production systems need fallback to HTTP/2 or HTTP/1.1, plus visibility into QUIC handshake failures, HTTP/3 adoption, and performance by client network.

Version	Transport	Wire Format	Main Benefit	Main Caveat
HTTP/1.1	TCP	Textual messages	Universal support, simple debugging	Limited concurrency per connection
HTTP/2	TCP	Binary frames	Multiple streams and header compression	Still affected by TCP-level head-of-line blocking
HTTP/3	QUIC over UDP	Binary HTTP/3 frames over QUIC	Stream-level recovery and connection migration	UDP reachability and operational complexity

10. HTTP in Distributed Systems
HTTP is easy to start with and easy to misuse. Production systems need clear rules for failure behavior.

Timeouts
Every HTTP client should set timeouts for each stage of a request: DNS lookup, connection setup, the TLS handshake, writing the request, and waiting for response headers.

Also set an overall deadline for the whole call and an idle timeout for connections sitting unused in the pool.

The defaults in many libraries are unsafe for production. A missing timeout can turn one slow dependency into exhausted threads, stuck connection pools, or a larger outage.

Retries
Retries should respect HTTP method behavior and application idempotency. Retrying a GET is usually safe. Retrying a POST can create duplicates unless the API supports idempotency keys.

A retry after a timeout is ambiguous because the server may have already processed the request. Retrying too aggressively can also make an outage worse.

Use bounded retries, backoff, jitter, deadlines, and clear retry budgets. Jitter means adding a little randomness so many clients do not retry at the exact same time.

Streaming
HTTP is not only for short JSON responses. Streaming is common in modern systems: Server-Sent Events for token streaming, chunked HTTP responses for incremental output, WebSockets for two-way communication, gRPC streaming over HTTP/2, and HTTP/3 streams over QUIC.

For AI products, streaming changes both user experience and infrastructure behavior. You need cancellation, idle timeouts, backpressure, partial failure handling, and metrics for time to first token and stream completion.

Backpressure means slowing the sender when the receiver or network cannot keep up.

Proxies and Headers
Most production HTTP requests pass through proxies or load balancers. Applications need to handle forwarded request information carefully:

Host
X-Forwarded-For
X-Forwarded-Proto
Forwarded
X-Request-ID or traceparent
Authorization
Only trust forwarding headers from infrastructure you control. Public clients can fake these headers unless the edge proxy removes or rewrites them.

Observability
HTTP gives excellent operational signals:

Request rate
Latency percentiles
Status code distribution
Retry rate
Payload size
TLS handshake failures
Internal service error rate
Cache hit ratio
Time to first byte
Stream duration
Break these signals down by route, method, status class, client, region, and internal service. Averages hide the failures users actually feel.


Summary
HTTP is the request-response language of the web: methods, resources, headers, status codes, caching, and bodies. HTTPS is HTTP protected by TLS, which adds encryption, tamper detection, and server identity checks. It does not replace application security.

The practical baseline is to use HTTPS for all production traffic, treat method behavior as part of the API contract, and use precise status codes and cache headers.

The major HTTP versions keep the same core ideas but differ greatly underneath. HTTP/2 reduces but does not eliminate head-of-line blocking, because TCP can still stall every stream on a connection. HTTP/2 server push is not worth designing around.

HTTP/3 over QUIC helps where its properties fit, as long as fallback and monitoring stay in place.

HTTP looks simple because the tooling is familiar. At scale, the hard parts are the same as any distributed system: latency, retries, overload, state, security, compatibility, and failure visibility. Set timeouts, retries, deadlines, and idempotency rules deliberately.