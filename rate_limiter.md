# functional requirement
- can limit the request with ip/api key/userid(jwt token)
- error 429 with error message also show timestamp when the user can send request

# non functional requirement
- low latancy (< 10ms)
- durable(rate limit for each user/aikey/ip should be durable)
- high available (eventual consistent)
- system should handle 1M request/second

# Entity (db table)

entity ->
1. Rules ->  for different users we have different limit 
2. client ->  differetn clienst like IP/userid/ip/api key
3. Request -> requiredUrl, clientid, ruleid

rule
rule_id(pk)
rate_limit_count
exipiry_time(time_limit)
createtimestamp

client
user_type (IP,USERID,API_KEY .ETC)(pk)
rule_id(fk)
createtimestamp

# API DESIGN(INTERFACE DESIGN)
# Token Bucket Rate Limiting with Redis (Summary)

```java
get  api/ratelimit?client_id={}
 respoonse : {
 ratelimit_count: 23,
 reset_time: time1,
 }
```
 
- in response header ->  It also provides information for response headers like 
-  X-RateLimit-Remaining and 
-  X-RateLimit-Reset.

# hogh level design
## Where should we place the rate limiter?

1. In process (place rate limiter at the each microservice level)
<img width="678" height="411" alt="Screenshot 2026-04-07 at 11 31 31 PM" src="https://github.com/user-attachments/assets/4ec7bba0-defe-4043-a714-594cd4010afb" />

- issue -> the main issue is at each microservice level only knowth rate limit about the request it priocess but what about the request a at other micrroservice 

2. dedicated server
<img width="683" height="584" alt="Screenshot 2026-04-07 at 11 35 04 PM" src="https://github.com/user-attachments/assets/f65a8c66-048b-4ab2-9e6d-6541d6189214" />

-  when rate limiter become it own dedicated microservices, for each request at each m,icroserlevel we have to check the rate limithing by request to rate limiter service, so it become the extra network round which increase latency.

- You've also introduced another point of failure. If your rate limiting service goes down, you need to decide: do you fail open (allow all requests through, risking overload) or fail closed (reject all requests, essentially taking your API offline)? Neither option is great.

3. Api gateway
<img width="669" height="473" alt="Screenshot 2026-04-07 at 11 48 43 PM" src="https://github.com/user-attachments/assets/a6be7ef9-908a-4ed2-a5a9-86d38b46e6ea" />

- we have applied the ratelimimter check at the edge of api gateway.
- every request either go to downstream server or either give 429 status with response header x-RateLimit-Remaining  x-RateLimit-Reset
- this is the standard production approach used in the industry.
- challenge -> we cannot apply the rate limit baseed on the different category user like premium user as we  haev aces to request header as we don't have the access at applicatoin level.

# how to identify theclients
- user id: jwt token (base64 encryption) -> decode it
- ip address -> X-faorward-For
- API Key-> X-API-KEY

# rate limiting methods 
1. Fixed Window Counter
<img width="742" height="393" alt="Screenshot 2026-04-08 at 12 02 08 AM" src="https://github.com/user-attachments/assets/c87d9eb7-c415-40f8-b4b1-feb1278cb51b" />

- The simplest approach divides time into fixed windows (like 1-minute buckets) and counts requests in each window.
- For each user, we'd maintain a counter that resets to zero at the start of each new window.
- If the counter exceeds the limit during a window, reject new requests until the window resets.
```java
  {
  "alice:12:00:00": 100,
  "alice:12:01:00": 5,
  "bob:12:00:00": 20,
  "charlie:12:00:00": 0,
  "dave:12:00:00": 54,
  "eve:12:00:00": 0,
  "frank:12:00:00": 12,
}
  ```
- challange(Bouncry issue) -> The main challenges are boundary effects: a user could make 100 requests at 12:00:59, then immediately make another 100 requests at 12:01:00.

2. Sliding Window Log
<img width="729" height="443" alt="Screenshot 2026-04-08 at 12 14 14 AM" src="https://github.com/user-attachments/assets/ffa1371e-aa9a-4132-8b44-ed3f1e6f8aec" />
- this approach keeps timestamp log for the individual ,when when new request arrived then it remove the older timestamp which is not in current time window.
- now unfare burst allowed.
- challange ->  memory usage(to keep track of timestamp for each user)

Token Bucket -> 
- Each client has a bucket of tokens (burst capacity)
- Tokens are added at a fixed refill rate
- Each request consumes 1 token
- If no tokens → request is rejected (rate limited)
- Supports:
- Bursts → via bucket capacity
- Sustained rate → via refill rate
- Example:
- Capacity = 100 tokens
- Refill = 10 tokens/min
→ 100 instant requests, then limited to 10/min
- Implementation:
##  Track (tokens, last_refill_time) per client
- Challenges:
- Choosing capacity & refill rate
- Handling cold start (idle users with full buckets)

<img width="752" height="377" alt="Screenshot 2026-04-08 at 12 23 48 AM" src="https://github.com/user-attachments/assets/b09aefe5-df1b-4331-93de-a27090a6efe2" />

- For our system, we'll go with the Token Bucket algorithm. It strikes the best balance between simplicity, memory efficiency, and handling real-world traffic patterns.

- we will use Redis for the keep track of tokencount and last_refile_timestamp for each request
- Redis can become our central source of truth for all token bucket state. When any gateway needs to check or update a user's rate limit, it talks to Redis.
- HMGET alice:bucket tokens last_refill (to fetch the token for alice user fro redis)
- The HMGET command retrieves the values of multiple keys from a Redis hash. In this case, we're getting the current tokens count and the last_refill timestamp for Alice's bucket.
- the redis will calculte teh token from the last_refill time and then it add to the current token count upto max capacity.
- Finally, the gateway makes the final decision based on Alice's updated token count. If she has at least 1 token available, the gateway allows the request and decrements her token count by 1. If she has no tokens remaining, the gateway rejects the request.
- for race conditions redis wil update tokena and refill_timestamp 
```java
MULTI(transaction)
HSET alice:bucket tokens <new_token_count>
HSET alice:bucket last_refill <current_timestamp>
EXPIRE alice:bucket 3600
EXEC(end transaction)
```

but wait we have applied the transaction t avoid race conditoon but waht about the read operatoin let's say alie has only 1 token and  he called two request simultaneuuosly then both the request will read the same time so means 

# give response on 429 
```java
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640995200
Retry-After: 60
Content-Type: application/json

{
  "error": "Rate limit exceeded",
  "message": "You have exceeded the rate limit of 100 requests per minute. Try again in 60 seconds."
}
```
# 1) How do we scale to handle 1M requests/second?
- so at small load our api gateway can use the single redis server for token info (status) but for 1 million request/second we need the multiple redis iunstance
- as single instance of redis can handle upto 100000-200000 request per seconds so we need multiple redis server to handle the massive load .
- so we need to partition our redis server in that way so tha each request know which redis instance to check the token status.
- so we need the distributed partition algorithm like consitent hashing so we can create create the hash of the particular key and teh we will find the moduleo of it and then we will fetch that redis server for token info.
- For anonymous users, we hash their IP address. For API key requests, we hash the API key. This ensures each client's rate limiting state lives on exactly one shard, while distributing the load evenly across all shards.

<img width="740" height="390" alt="Screenshot 2026-04-08 at 9 56 10 PM" src="https://github.com/user-attachments/assets/59e68a6a-5a4a-49d9-8544-42c6a3cde05a" />
- With 10 Redis shards, each handling ~100k operations/second, we should be able to handle our 1M request/second target.


# 2) How do we ensure high availability and fault tolerance?

- When a Redis shard becomes unavailable, we face a fundamental decision about our failure mode. We have two options:
- 1. Fail closed
- Approach -> When the rate limiter can't reach Redis, reject all requests with HTTP 503 "Service Unavailable" or HTTP 429 responses. This is the most restrictive option. No requests get through that we can't verify are within limits.
- challanges -> This will effectively take your API offline during Redis outages. Users see failed requests even if your backend services are healthy. 
- actual use case ->  Financial systems processing payments might prefer to reject transactions rather than risk processing them without rate limits.
- 2. Fail-Over -> When the rate limiter can't reach Redis, allow all requests to proceed as if rate limiting was disabled.
- The API gateway skips rate limit checks and forwards requests directly to backend services. This keeps your API available even when the rate limiting infrastructure has issues.
- Challenges -> The obvious downside is temporarily losing rate limit protection. During Redis outages, malicious users could potentially overwhelm your backend services with requests. More critically, this can cause cascade failures 

use case -For social media platforms, this is especially dangerous during viral events when traffic spikes are already stressing the system. Failing open could turn a rate limiter outage into a complete platform failure.

<img width="734" height="626" alt="Screenshot 2026-04-08 at 10 27 00 PM" src="https://github.com/user-attachments/assets/6560ba74-114a-422d-97c1-2facced878af" />

# how to handle the hot keys
- For Legitimate High-Volume Clients:
- Client-side rate limiting: Encourage well-behaved clients to implement their own rate limiting to smooth traffic patterns. This prevents legitimate users from -
-accidentally creating hot shards while reducing server load. Many API SDKs include built-in client-side rate limiting that respects server response headers.
- Request queuing/batching: Allow clients to batch multiple operations into single requests, reducing the total number of rate limit checks needed.
- Premium tiers: Offer higher rate limits for power users who need them, potentially with dedicated infrastructure.
- For Abusive Traffic:
- Automatic blocking: When a client hits rate limits consistently (say, 10 times in a minute), temporarily block their IP/API key entirely by adding them to a blocklist. = The list can be kept in one of the Redis shards and only checked in case of cache misses.
- DDoS protection: Use services like Cloudflare or AWS Shield that can detect and block malicious traffic before it reaches your rate limiter.

# 5) How do we handle dynamic rule configuration?
- 1. poll based configuration: Store rule configuration in a database or dedicated configuration service. 
- Your API gateways periodically poll for configuration changes (say, every 30 seconds) and update their rate limiting logic accordingly.
- This is the most common approach because it's straightforward to implement and handles the majority of use cases.
- The configuration service can be as simple as a database table with columns for client_type, endpoint, requests_per_minute, etc.
- Gateways query this table on a schedule and cache the results locally.
- challenge -> The main downside is update delay. There's always a window between when you change a rule and when it takes effect across all gateways.
- 2. push based configuration ->
 - Use a push-based system where configuration changes are immediately sent to all API gateways.
- This is exactly what ZooKeeper was designed for - distributed configuration management with real-time notifications.
- ZooKeeper maintains configuration data and notifies all connected clients (your API gateways) immediately when any configuration changes.
- Other options include Redis pub/sub or custom configuration services that maintain persistent connections to gateways.
- When an operator changes a rate limit rule, the configuration service immediately notifies all connected gateways, which update their rules within seconds.
<img width="680" height="540" alt="Screenshot 2026-04-09 at 10 47 00 PM" src="https://github.com/user-attachments/assets/f25cadd9-95c3-477d-a977-a3638a592320" />











