Software is usually made of many smaller pieces. Those pieces need a safe, predictable way to talk to each other.

An API (Application Programming Interface) is that agreed way of talking. It tells one piece of software what it can ask for, what data it must send, what it will get back, and what errors can happen.

You can think of an API as a front desk for a system. A caller does not walk into the database, inspect the source code, or reach into internal services. The caller sends a request to the API, and the API decides what to do.

APIs come in many forms. A weather API returns a forecast. A payment API charges a card. An AI API starts a model run or streams tokens. A library API exposes functions you can call from code. Different shape, same idea: the caller depends on a clear agreement, not on hidden implementation details.

A good API also protects the system behind it. It checks input, verifies who is calling, checks what the caller is allowed to do, limits abusive traffic, and keeps internal services from being exposed directly.

This chapter explains what an API is, what belongs in an API contract, and what a well-designed API boundary should handle.


1. What an API Contract Defines
An API contract is the agreement between the caller and the provider. A useful contract answers these questions:

Question	Example
What can the caller do?	GET /v1/orders/ord_123
What must the caller send?	Path values, query values, headers, and request body
What comes back?	JSON response, file, stream, or event
What can go wrong?	Status codes, error codes, and whether retrying is safe
Who is calling?	API key, OAuth token, session cookie, or mTLS certificate
What can this caller access?	Which user, tenant, team, or resource is allowed
What are the limits?	Rate limits, request size, timeout, and pagination
How does it change safely?	Versions, support windows, and optional fields
For a network API, the contract may live in documentation, an OpenAPI file, Protocol Buffers, a GraphQL schema, AsyncAPI, or a provider-specific format. For a library API, it may live in function signatures, types, docstrings, and tests. These are all ways to write down the same basic agreement.

An API Contract in Practice
Here is a simple API request to start a model run. It shows the method and path, an auth token, the content type, an idempotency key, and a JSON body. An idempotency key is a duplicate-protection key; we will come back to it later.

Plaintext







12345
The body carries the actual input for the model run:

Json







1234567
The response should be easy for the caller to predict and parse:

Json







12345
The caller should also get a clear error when something is wrong:

Json







1234567
That is the difference between "the server returned some data" and "clients can safely build on this API."


2. APIs Are Boundaries
APIs are boundaries. They separate what callers are allowed to use from how the system works internally.

This matters because the server should be able to change its database, queue, model server, cache, or internal service layout without forcing every client to change. As long as the API still behaves the same from the caller's point of view, the inside can evolve.




Implementation Can Change

Clients

Web App

Mobile App

Partner System

Internal Worker

Stable API Contract

Application Service

Cache

Database

Queue

Model Server

This boundary is valuable because it gives the system a clean edge:

Hiding internals: Callers use behavior without knowing how it is implemented.
Safer changes: Old clients keep working while the server changes carefully.
Security: Callers get controlled access instead of direct database or system access.
Ownership: Teams can own clear service boundaries and publish clear contracts.
Reuse: Web apps, mobile apps, partners, and workers can use the same capability.
Debugging: Requests can be logged, measured, traced, and audited at the API edge.
APIs also create responsibility. If a public API breaks, customers may break. If an internal API has no clear owner, every change becomes a meeting. If a model API hides its latency, cost, or safety behavior, the systems calling it become harder to debug.


3. Types of APIs
APIs can be grouped by who uses them and how they are called.

Public APIs
Public APIs are used by external developers, customers, partners, or third-party systems.

Because people outside your team depend on them, they need clear documentation, stable behavior, authentication, authorization, rate limits, usage quotas, safe error responses, monitoring, and a plan for changing old behavior without surprising callers.

Examples include payment APIs, maps APIs, messaging APIs, cloud APIs, and AI platform APIs.

Partner APIs
Partner APIs are used by selected external organizations. They are not open to everyone, but they still cross company boundaries.

They often need extra care: contract review, access per tenant or partner, audit logs, data-sharing rules, planned change windows, and stronger support expectations.

Internal APIs
Internal APIs are used inside one organization. For example, a checkout service may call inventory, payment, fraud, fulfillment, notification, and analytics services.

Internal does not mean informal. Internal APIs still need owners, documentation, timeouts, stable request and response shapes, and clear behavior when something fails.

The risk can be large because internal APIs often sit on important paths such as checkout, login, payment, search, or model inference.

Library APIs
Library APIs are called inside the same running program. A standard library, web framework, database client, or ML library exposes functions, classes, and types.

Python







123456
The caller does not need to know how Python stores list capacity internally. The API exposes useful behavior and hides the details.


4. Network APIs
Most system design discussions focus on network APIs. A client sends data over a network to a server. The server responds right away, streams results, or starts work that can be checked later.

Request Parts
An HTTP API request usually has:

Part	Purpose	Example
Method	What kind of action this is	GET, POST, PATCH, DELETE
Path	Target resource or action	/v1/orders/ord_123
Query string	Filtering, pagination, options	?limit=50&cursor=abc
Headers	Extra information and credentials	Authorization, Accept, Content-Type
Body	Structured input	JSON request body
Those parts combine into a request on the wire:

Plaintext







1234
Response Parts
An HTTP API response usually has:

Part	Purpose	Example
Status code	Outcome at the HTTP level	200, 201, 400, 401, 404, 429, 500
Headers	Extra information about the response	Content-Type, Cache-Control, RateLimit-Remaining
Body	The returned data or error	JSON object, file bytes, stream
A successful response to the request above carries the matching body:

Json







12345678910
The body should not be the only place that says whether the request worked. A failed request usually should not return 200 OK with { "success": false }.

Status codes matter because clients, gateways, proxies, SDKs, and monitoring tools use them to understand what happened.


5. Common API Styles
Different API styles use different contracts.

Style	How the API is shaped	Common Use
REST	Resources and standard HTTP behavior	Public web APIs, product backends
GraphQL	Typed graph and client queries	Flexible product screens over connected data
gRPC	Service methods and Protocol Buffers	Internal service-to-service APIs
WebSocket	Long-lived two-way connection	Chat, collaboration, trading, games
Server-Sent Events	One-way HTTP event stream	Progress updates, notifications, token streaming
Webhooks	HTTP callbacks	Provider-to-server event notifications
Message-driven APIs	Events on a broker	Background workflows and data pipelines
SOAP	XML envelopes and WSDL	Legacy enterprise integrations
The first API in a small product is often REST because it is easy to inspect, easy to document, and works across browsers, mobile apps, scripts, and backend services.

As the system grows, other styles may appear for specific needs: GraphQL when screens need flexible data, gRPC for strongly typed internal calls, SSE for token streaming, WebSocket for two-way live sessions, and events for background workflows.


6. Authentication, Authorization, and Limits
An API usually needs to answer three basic questions:

Who is calling?
What are they allowed to do?
How much traffic are they allowed to send?
Authentication
Authentication means identifying the caller. In plain language: "Who are you?"

Common methods:

API keys: Common for server-to-server access and developer platforms.
OAuth 2.0 access tokens: Common when a user or app grants limited API access.
Session cookies: Common for browser applications.
mTLS: Common for internal services and high-trust enterprise integrations.
Do not put API keys or bearer tokens in query parameters. URLs can end up in browser history, proxy logs, analytics tools, referrer headers, and monitoring systems. Put secrets in headers such as Authorization.

Authorization
Authorization means deciding what the caller is allowed to do. In plain language: "Are you allowed to do this?"

Authentication is not enough. A signed-in user may still be forbidden from reading another tenant's invoice, deleting another team's dataset, or using a model their plan does not include.

Good authorization checks both the caller and the specific resource being requested.

Rate Limits and Quotas
Rate limits protect the API from overload and abuse. Quotas enforce product or tenant usage limits.

For AI APIs, counting requests is often not enough. One tiny prompt and one huge document-processing request may cost very different amounts. Token count, file size, model type, number of parallel requests, and queue time may all matter.


7. API Reliability
Production APIs must be clear about failure. Clients need to know when to wait, when to retry, and when to stop and show an error.

Timeouts
Every API call should have a timeout. Without one, callers can wait forever. Threads and connections can pile up, and one slow dependency can make the whole system feel stuck.

Retries
Retries help only when repeating the request is safe, or when the API has a way to recognize duplicate attempts.

Retrying GET /orders/ord_123 is usually fine because it is just a read. Retrying POST /payments without an idempotency key can charge the customer twice.

Idempotency
Idempotency means that sending the same request more than once has the same intended effect as sending it once.

Payment, order creation, model-run creation, and job-submission APIs often use idempotency keys. The key tells the server, "If you have already processed this exact operation, do not do it again."

Plaintext







12345
Debugging and Observability
APIs should record enough information to debug production issues. At minimum, that usually means:

A request ID or trace ID.
The caller or tenant, when safe to log.
The route and method.
The status code.
How long the request took.
The request and response size.
Whether the request was rate limited.
Time spent calling downstream services.
Logs must not casually store secrets, credentials, raw access tokens, sensitive personal data, model prompts, or private documents. If a system logs sensitive data, it needs explicit controls for who can access it and how long it is kept.


8. How Engineers Use an API
A practical API integration flow looks like this:

Read the official documentation or machine-readable API contract.
Identify authentication, required scopes, limits, and pricing.
Test a request with curl, an API client, or a generated SDK.
Validate success and error responses.
Add timeouts, retries, and idempotency where they are needed.
Handle rate limits and pagination.
Log request IDs and important failure details.
Keep credentials out of source code, URLs, logs, and browser bundles.
Example curl request:

Shell







123
Example Python request:

Python







123456789101112131415161718192021
This example keeps the token in an environment variable, sends it in a header, uses query parameters only for non-secret input, and sets a timeout.


Summary
An API is an agreement between software systems. It defines what callers can do, what they must send, what they get back, what errors mean, who can access what, and what limits apply.

It is also a boundary. Clients should use the API instead of reaching into databases, source code, queues, model servers, or internal services directly.

Public, partner, internal, and library APIs serve different audiences, but they all benefit from clear behavior and predictable contracts.

Network APIs need careful handling of methods, paths, headers, bodies, status codes, authentication, authorization, rate limits, timeouts, retries, and debugging information.

A strong API makes a system easier to use, change, secure, and operate. A weak API leaks internal details and forces every client to learn special rules.