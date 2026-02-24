requirements:
1. search functionality
2. train ticket booking
3. waiting list logic
4. concurrenc control


<img width="1243" height="682" alt="Screenshot 2026-02-24 at 11 10 18 PM" src="https://github.com/user-attachments/assets/0bac0c81-22e8-44f7-989e-0decd182f6b3" />

 ### partial booking
 
For train 12002 on route 293 with stations A -> B -> C -> D, seat availability is tracked by adjacent segments only: [AB, BC, CD].
So each seat has a 3-bit occupied_mask:
bit 0 -> AB
bit 1 -> BC
bit 2 -> CD
Initially, for a free seat: occupied_mask = 000.
If a user wants to travel from A -> C, the requested segments are AB and BC, so request_mask = 011.

Availability check:
(occupied_mask & request_mask) == 0

If true, no overlap exists, so the seat is available for that journey.
If false, at least one segment overlaps, so booking must be rejected for that seat.
When available, reserve using:

occupied_mask = occupied_mask | request_mask

Example:

before: 000
request: 011
after reserve: 011
To handle concurrency safely, perform this as an atomic conditional update (with version/CAS) so two users cannot reserve overlapping segments at the same time.

For payment flow, use:
HOLD seat segments temporarily (with TTL),
process payment,
on success -> CONFIRM booking,
on failure/timeout -> release hold bits.
This supports partial bookings correctly (for example, B->D and A->B can coexist on the same seat, but A->C conflicts with B->D).

### waitinglist logic
for waiting list logic we will use Redis distributed List
<key> : queue
so key will be. {tripid}:{startindex}:{endindex} -> queue 
so users with same trip id having same start and end index will be push to same queue in oreder so taht we can pop the userid fro
Redis List is an ordered sequence of strings, like a queue.

Core behavior
Preserves insertion order.
Supports push/pop from both ends.
Good for FIFO waitlist.
Commands you’ll use
LPUSH key v -> add left
RPUSH key v -> add right
LPOP key -> remove/read left
RPOP key -> remove/read right
LRANGE key 0 -1 -> read all
LLEN key -> size
BLPOP key timeout -> blocking pop (worker waits until item exists)
Queue pattern (FIFO)
Producer adds: RPUSH waitlist req_101
Consumer takes: LPOP waitlist
Oldest request gets served first.
Example
RPUSH wl:q:TRIP123:1:3 req_101 req_102 req_103
LRANGE wl:q:TRIP123:1:3 0 -1
# => req_101 req_102 req_103

LPOP wl:q:TRIP123:1:3
# => req_101


### search functinalaity:
- search bytrain name and code name
- SELECT station_id, code, name, city
FROM station
WHERE LOWER(name) LIKE LOWER(:q || '%')
   OR LOWER(code) LIKE LOWER(:q || '%')
ORDER BY name
LIMIT 10;

search by src,dest and date
SELECT t.trip_id, t.train_id, t.journey_date, t.departure_time, t.arrival_time
FROM trip t
JOIN route_stop rs_src ON rs_src.route_id = t.route_id AND rs_src.station_id = :src_station_id
JOIN route_stop rs_dst ON rs_dst.route_id = t.route_id AND rs_dst.station_id = :dst_station_id
WHERE t.journey_date = :journey_date
  AND rs_src.stop_index < rs_dst.stop_index
  AND t.status = 'OPEN'
ORDER BY t.departure_time ASC
LIMIT :limit OFFSET :offset;

###  concurrency habdling 
For your ticket booking system
Optimistic concurrency (version/CAS) on trip_seat_inventory
Use when conflicts are possible but not constant, and you need high throughput.
Atomic update with condition:
no segment overlap
version matches
If update fails, retry next seat.
Short HOLD + TTL
Use before payment to prevent double booking.
Reserve seat temporarily (HOLD)
Confirm after payment
Auto-release on timeout.
Idempotency key on booking/payment APIs
Use for client retries/network timeouts.
Same request key -> same result
Prevent duplicate bookings/charges.
FIFO queue for waitlist allocation (Redis List/Stream)
Use when a seat is freed and many users compete.
Single allocator worker per trip partition
Deterministic assignment order.
Pessimistic lock (FOR UPDATE SKIP LOCKED)
Use only when contention is very high on small seat pool.
Stronger locking, less retry churn
Can reduce throughput if overused.
Quick rule of thumb
Normal booking traffic: Optimistic CAS + HOLD TTL + idempotency
Extreme hotspot contention: add pessimistic row locking selectively
Waitlist promotions: queue + single allocator consumer
If you want, I can give exact SQL/Redis commands for each step.
   
