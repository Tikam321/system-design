Bit.ly System Design (Beginner-Friendly)
Understanding the Problem
What is Bit.ly?
Bit.ly is a URL shortening service that converts long URLs into short, manageable links. It can also provide analytics for shortened URLs.

Designing a URL shortener is a classic beginner system design interview question. This walkthrough focuses on strong fundamentals over unnecessary complexity.

### Functional Requirements
In a system design interview, first clarify what the system must do.

### Core Requirements
Users can submit a long URL and receive a shortened URL.
Users can optionally specify a custom alias (e.g., short.ly/my-custom-alias).
Users can optionally specify an expiration date.
Users can access the original URL using the shortened URL.
Out of Scope (Below the Line)
User authentication and account management.
Link analytics (click counts, geography, etc.).
Non-Functional Requirements
Define how well the system should perform.

### Core Requirements
Short codes must be unique (1 short code -> exactly 1 long URL).
Redirect latency should be low (< 100ms).
High availability (99.99%), prioritizing availability over consistency.
Scale to 1B shortened URLs and 100M daily active users.
Out of Scope (Below the Line)
Real-time consistency for analytics.
Advanced security/spam/malicious URL filtering.
Read/Write Asymmetry
URL shorteners are heavily read-dominant (e.g., 1000 (line 1) reads to writes).
This affects:

### Caching strategy
Database design
Overall architecture
Setup
Core Entities
Start with high-level entities before detailing schema:

Original URL: long URL provided by user
Short URL: generated short link/code
User: creator of short URL (optional in MVP if auth is out of scope)
API Design
Map APIs to functional requirements.

Create Short URL
POST /urls
Content-Type: application/json

{
  "long_url": "https://www.example.com/some/very/long/url",
  "custom_alias": "optional_custom_alias",
  "expiration_date": "optional_expiration_date"
}
Response:

{
  "short_url": "http://short.ly/abc123"
}
Redirect to Original URL
GET /{short_code}
Response:

302 Found with Location: <long_url> (recommended)
301 Moved Permanently is possible but often less flexible
High-Level Design
1) Create a Short URL
Components:

Client (web/mobile)
Primary Server (validation, code generation, business logic)
Database (short_code -> long_url mapping, alias, expiry)
Flow:

Client sends POST /urls.
Server validates URL format.
Optionally checks deduplication policy.
Generates short code (or validates custom alias).
Stores mapping in DB.
Returns shortened URL.
Notes:

Most systems allow multiple short URLs per same long URL.
Prevent alias collision with generated codes using namespaces or prefix rules.
2) Redirect to Original URL
Flow:

Browser requests GET /abc123.
Server looks up short code in DB/cache.
Server checks expiration.
If active, returns redirect to long URL.
If expired, return 410 Gone.
Cleanup:

Periodic job can delete expired records (or keep with expiry metadata).
Cache TTL should not exceed URL expiration.
Redirect Status Codes
301 Permanent Redirect: browser may cache aggressively.
302 Found: better control for updates/deletes/expiry and click tracking.
Example:

HTTP/1.1 302 Found
Location: https://www.original-long-url.com
Deep Dive 1: Ensuring Unique Short Codes
Key constraints:

Uniqueness
Short length
Efficient generation
Option A: Hash-Based Codes
Approach:

Canonicalize URL
Hash (SHA-256, etc.)
Base62 encode
Take first N chars (e.g., 8)
Pseudo-code:

input_url = "https://www.example.com/some/very/long/url"
canonical_url = canonicalize(input_url)
hash_code = hash_function(canonical_url)
short_code_encoded = base62_encode(hash_code)
short_code = short_code_encoded[:8]
Why Base62:

Uses a-zA-Z0-9
Avoids + and / from Base64 (URL/path issues)
Challenges:

Truncation can still collide.
Need DB UNIQUE constraint + retry logic (3-5 retries).
Deterministic hash implies same input -> same output unless salted.
Option B: Global Counter + Base62 (Common Choice)
Approach:

Use atomic counter (e.g., Redis INCR)
Encode counter value in Base62
Pros:

Guaranteed uniqueness without collision checks.
Very fast and simple.
Easy to scale writes with centralized counter service.
Challenges:

Single global counter coordination in distributed systems.
Sequential IDs are predictable (possible enumeration).
Code length increases over time (slowly).
Capacity example:

1,000,000,000 in base62 ≈ 6 chars (15ftgG)
6 chars covers ~56B values (62^6)
7 chars covers ~3.5T values (62^7)
Deep Dive 2: Fast Redirects
Pattern: Scale reads aggressively.

Use Cache + CDN + Edge
Cache short_code -> long_url mappings
Serve through CDN PoPs globally
Optionally run redirect logic at edge (Cloudflare Workers, Lambda@Edge)
Benefits:

Lower latency (closer to user)
Popular links may not hit origin server
Tradeoffs:

Cache invalidation complexity
Edge runtime limits
Higher cost
Distributed debugging/monitoring complexity
Deep Dive 3: Scaling to 1B URLs and 100M DAU
Storage Estimate
Approx per row:

short code: ~8 bytes
long URL: ~100 bytes
created time: ~8 bytes
alias (optional): ~100 bytes
expiry: ~8 bytes
plus metadata overhead
Use rough estimate: ~500 bytes/row

For 1B rows:

500 * 1B = 500GB
This is manageable with modern storage.

Database Choice
Given low write rate and cached reads, many DBs fit:

Postgres
MySQL
DynamoDB
If unsure in interview: choose Postgres.

Availability
DB replication + failover
Periodic backups + restore plan
Service Scaling
Split services:

Write Service: short URL creation
Read Service: redirects
Scale each horizontally based on load.

Counter Service at Scale
With multiple write instances, use shared counter source:

Centralized Redis (atomic increments)
Counter Batching Optimization
To reduce Redis calls:

Write instance requests batch (e.g., 1000 IDs)
Redis increments global counter by batch size
Instance uses local batch
Refill when exhausted
Benefits:

Fewer network round trips
Lower Redis load
Better throughput
HA + Multi-Region
Redis Sentinel/Cluster with failover
Region-specific counter ranges to avoid cross-region coordination
Region A: 0 - 1B
Region B: 1B - 2B
DB UNIQUE(short_code) remains final safety net
Note:

Losing a few counter values during failure is acceptable (uniqueness matters, not continuity).
Final Architecture Summary
POST /urls handled by Write Service
GET /{short_code} handled by Read Service
Redis counter (with batching) for unique ID generation
Base62 encoding for compact short codes
Cache + CDN/Edge for low-latency redirects
Relational DB for durable mappings
Replication/failover for high availability
410 Gone for expired links
302 Found as default redirect status for control and flexibility
