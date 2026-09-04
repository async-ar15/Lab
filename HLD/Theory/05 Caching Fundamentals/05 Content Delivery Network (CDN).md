When users are spread across the world, distance starts to matter. A request from Sydney to a server in Virginia is slow even if the server itself is very fast.

A Content Delivery Network (CDN) helps by placing servers closer to users. These nearby servers are called edge servers.

Instead of sending every request to one central server, the CDN can serve cached content from an edge server near the user. If the edge does not have the content yet, it fetches it from the origin server, stores it if allowed, and uses that copy for later users.

A good CDN setup does more than make pages faster. It also reduces load on your origin, helps during traffic spikes, and can block some unwanted traffic before it reaches your application.

Content Delivery Network

Hits: 0
Misses: 0
Hit Rate: 0%
Cache Hit: Served from edge (fast).Cache Miss: Fetched from origin, then cached.


New York

London

Tokyo
US-East
0 cached
EU-West
0 cached
AP-NE
0 cached
Origin
Server
Click a user to request content. First request caches at edge, subsequent requests are faster.

Fullscreen
In this chapter, we cover what a CDN does, how request routing works, what CDNs cache, what can go wrong, and how to choose a provider.


1. What a CDN Does
A CDN is a network of edge locations, often called points of presence (PoPs). Each edge location runs servers that can accept user requests, store cached responses, and forward requests to the origin when needed.




CDN Edge Placement
Users (EU)
Users (JP)
Users (BR)
Origin (US-East)
CDN EU
CDN Asia
CDN SA
CDN AUS
CDN US-West
The origin is the real source for the content. It might be an object storage bucket, web server, load balancer, API gateway, or application service.




Request

Cache hit

Cache miss

Response

Return response
and store if allowed

User

CDN Edge

Origin Server

Without a CDN, every user request travels to the origin. With a CDN, many requests stop at the edge.

That changes the scaling problem. The origin no longer needs to serve every image, JavaScript file, video segment, or cacheable API response. It mostly handles misses, dynamic requests, writes, and content that should not be cached.


2. How CDN Routing Works
A CDN does not always choose the edge server that is closest on a map. It also considers network speed, edge capacity, server health, routing rules, and CDN configuration.

You may hear terms like DNS routing, anycast, and peering when people discuss CDN routing. The beginner-friendly version is this: the CDN tries to send each user to an edge that is nearby, healthy, and fast from the network's point of view.

From the user's point of view, the flow usually looks like this:

The user requests https://static.example.com/app.js.
DNS resolves the hostname to the CDN, not directly to the origin.
The user's request reaches an edge location chosen by the CDN's routing system.
The edge checks whether it has a fresh cached response for that request.
On a cache hit, the edge returns the response immediately.
On a cache miss, the edge fetches the response from the origin.
If the response may be cached, the edge stores it for future requests.



alt
[Cache hit]
[Cache miss]
Resolve static.example.com
Return CDN address
GET /app.js
Return cached response
Fetch /app.js
Return response
Store if allowed
Return response
Browser
DNS
CDN Edge
Origin
Browser
DNS
CDN Edge
Origin




8 / 8
The first request for an object in a region may still be slow because it has to go back to the origin. The payoff comes when later requests are served directly from the edge.


3. What CDNs Cache
CDNs are best known for caching static assets: images, CSS files, JavaScript bundles, fonts, video segments, audio files, documents, software installers, and game patches.

They can also cache some dynamic responses, such as rendered HTML pages, search results, catalog pages, or API responses. This is safe only when everyone who shares the same cache key should receive the same response.

Do not cache private user data in a shared CDN cache unless the design has been reviewed carefully. A small cache-key mistake can show one user's data to another user.

A good fit for CDN caching is content that is read often, does not change on every request, can be shared by many users, can be a little stale for a known amount of time, and is large or expensive enough that caching is worth it.

Poor fits include account pages, shopping carts, payment flows, admin pages, and responses that depend heavily on private cookies or login state.


4. Cache Keys
A CDN cache is usually a large key-value store. The cache key decides whether two requests should use the same cached response.

A basic CDN cache key includes the scheme, host, path, and sometimes the query string:

Scheme: https
Host: static.example.com
Path: /images/logo.png
Query string: ?w=800
Many CDNs can also include selected headers or cookies in the cache key. That gives you more control, but it also makes mistakes easier.

If the key is too broad, different requests get treated as the same request. For example, if the CDN ignores a language header, a user asking for Spanish content may receive English content.

If the key is too narrow, too many requests look unique. For example, if the CDN varies on every cookie, most requests miss the cache and the CDN stops helping.

Common Cache Key Choices
The examples below separate paths that are usually safe to cache from personalized paths that should usually skip shared caching.








12345678910
For public content, keep the key stable and small. For personalized content, skip shared CDN cache unless you have a carefully designed plan.


5. HTTP Caching Headers
CDNs usually follow HTTP caching headers from the origin, although CDN-specific rules can override them.

Common headers include:

Plaintext







123456
Important directives:

public: Shared caches such as CDNs may store the response.
private: The response is meant for one user and should not be stored in a shared cache.
max-age: How long caches can treat the response as fresh.
s-maxage: Like max-age, but only for shared caches such as CDNs.
no-store: Do not store the response at all.
immutable: The asset will not change while it is fresh.
stale-while-revalidate: Serve a stale response briefly while refreshing it in the background, if supported.
ETag and Last-Modified help with validation. When cached content expires, the CDN can ask the origin, "Did this object change?" If it did not change, the origin can return 304 Not Modified instead of sending the full object again.


6. TTLs and Purging
A TTL controls how long a CDN can treat cached content as fresh.

Long TTLs usually mean better performance and less origin traffic. Short TTLs reduce stale content, but they send more requests back to the origin.

The cleanest pattern for static assets is versioned URLs:








123
When the file changes, its URL changes. The old file can stay cached for a long time because new pages no longer point to it.

For content that keeps the same URL, you need a plan for keeping it fresh:

Use a short TTL.
Purge the URL when content changes.
Purge by cache tag if the CDN supports it. This lets you delete a group of related objects without listing every URL.
Use ETag or Last-Modified so the CDN can ask whether the content changed before downloading it again.
Use stale-while-revalidate for content that can be briefly stale.
Avoid designing a system that needs constant global purges. Purges can take time, have provider limits, and are easy to get wrong during incidents.


7. Benefits of a CDN
A CDN does more than move bytes closer to users.

Lower Latency
Users often connect to an edge location that is faster to reach than the origin. This reduces round-trip time for secure connections, HTTP requests, and large downloads.

Less Origin Load
Cache hits do not reach the origin. This can greatly reduce bandwidth, CPU usage, storage reads, and database pressure.

Better Availability
If one edge location has trouble, CDN routing can move traffic elsewhere. Some CDNs can also serve stale cached content when the origin is temporarily unavailable.

Traffic Spike Absorption
CDNs are built for moments when many users request the same content at once. They are useful for product launches, breaking news, live events, sales, and software releases.

Security at the Edge
Many CDN platforms include DDoS protection, web application firewall rules, bot filtering, TLS management, rate limits, and geo rules. These features do not replace application security, but they reduce how much unwanted traffic reaches the origin.

Protocol and Media Optimizations
Depending on the provider, CDNs can support HTTP/2, HTTP/3, compression, image optimization, video packaging, small edge functions, and request routing rules.


8. Trade-offs and Things That Can Go Wrong
CDNs make systems faster, but they also add another layer to understand and operate.

Stale Content
The edge may serve old content until the TTL expires or a purge finishes. This is fine for many images and product listings, but dangerous for prices, permissions, legal text, and account data.

Cache Key Bugs
A wrong cache key can either ruin the hit ratio or serve one user's response to another user. Review cookie, header, and query string behavior carefully.

Origin Overload on Misses
When a popular object expires everywhere at the same time, many edges may fetch it from the origin at once.

Several techniques reduce this risk:

Origin shielding: Send cache misses through a small number of shield locations before they reach the origin.
Request collapsing: Let one origin fetch satisfy multiple simultaneous misses for the same object.
Jittered TTLs: Add small random variation to expiration times so many objects do not expire together.
Stale-while-revalidate: Serve slightly stale content while refreshing it in the background.
Debugging Complexity
The response may depend on edge location, cache state, headers, cookies, routing rules, and origin behavior. Good logs and debug headers are essential.

Useful debug headers include:

Plaintext







1234
Cost
CDN pricing often includes bandwidth, request count, cache fill traffic, purges, logs, security features, image processing, and edge compute. Video delivery and large downloads can become expensive.

Regional and Privacy Requirements
Some systems must control where data is processed or cached. This matters for regulated data, contracts, and regional privacy rules.


9. Common Use Cases
CDNs fit content that many users request and that is safe to serve from a shared cache.

Static Web Assets
The most common use case is serving JavaScript, CSS, fonts, and images from the edge. Versioned filenames and long TTLs work well here.

Video and Audio Streaming
Streaming systems split media into many small segments. CDNs cache and deliver those segments close to users, reducing buffering and origin bandwidth.

Software and Game Downloads
Installers, mobile app assets, game patches, and operating system updates are large and often requested by many users at the same time. CDN caching is a strong fit.

Public API and HTML Caching
Some APIs and rendered pages can be cached at the edge, especially catalog pages, documentation pages, search results, and anonymous home pages. The key is to cache only responses that are safe to share.

Security Front Door
Many teams put a CDN in front of public applications to handle TLS, apply WAF rules, block abusive traffic, and route requests before they reach the application stack.


10. Production Practices
Use these defaults unless your application has a good reason to do something different:

Put immutable static assets on versioned URLs.
Give immutable assets long TTLs.
Use short TTLs or explicit purges for content that changes in place.
Do not cache authenticated or personalized responses in a shared cache by default.
Keep cookies out of the cache key unless they are required.
Normalize query strings so equivalent requests use the same key.
Use origin shielding or request collapsing for expensive objects.
Allow stale content briefly for pages where freshness is not critical.
Add response headers that expose cache status and request IDs.
Monitor hit ratio per route, not just globally.
Useful metrics to watch include cache hit ratio, origin request rate, origin bandwidth, edge latency, origin latency, 4xx and 5xx rates, cache fill errors, purge volume, purge latency, and the top paths by bandwidth and request count.

A high global hit ratio can hide a bad path. Inspect the endpoints that matter most to users and to your origin cost.


11. CDN Providers
Major CDN providers include:

Akamai
Cloudflare
Fastly
Amazon CloudFront
Google Cloud CDN and Media CDN
Azure Front Door
Choose a provider based on your system's needs, not on a generic ranking.

Useful selection criteria include user coverage, cloud integration, cache rule flexibility, purge speed, how targeted purges can be, origin shielding, request collapsing, metrics and logs, security features, edge compute support, media delivery support, pricing for your traffic pattern, and how comfortable your team is with the platform.

For example, a team already running on AWS may prefer CloudFront because it fits naturally with AWS. A media company may care more about video delivery features. A platform team serving dynamic web applications may care about edge logic, fast purges, and detailed logs.

Also check product lifecycle. For example, Microsoft has been moving customers from classic Azure CDN offerings toward Azure Front Door as its modern CDN platform.


Summary
A CDN is an edge layer that improves performance and reliability by serving cacheable content closer to users.

Most of the design work goes beyond turning it on. You need to decide what can be cached, how the cache key is built, how freshness is controlled, how content is purged, and how the origin is protected during misses.

For static assets, use versioned URLs and long TTLs. For dynamic or semi-dynamic content, be explicit about cache keys, headers, and stale content. For private data, bypass shared caching unless the design has been reviewed carefully.

A well-configured CDN makes a global system faster, cheaper, and more reliable. A careless CDN setup can hide stale data, leak personalized responses, or turn cache misses into origin incidents.