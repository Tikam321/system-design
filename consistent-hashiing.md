### consistent hashisng
- To achieve horizontal scaling, it is important to distribute requests/data efficiently and evenly across servers. Consistent hashing is a commonly used technique to achieve this goal. But first, let us take an in-depth look at the problem.
<img width="490" height="248" alt="Screenshot 2026-03-09 at 12 05 38 AM" src="https://github.com/user-attachments/assets/94f8d75d-d7e7-4fae-abc0-cfdfb88241d3" />


 ## the rehashing problem 
- If you have n cache servers, a common way to balance the load is to use the following hash method:
- serverIndex = hash(key) % N, where N is the size of the server pool.
- This approach works well when the size of the server pool is fixed, and the data distribution is even. However, problems arise when new servers are added, or existing servers are removed.
- if server 1 goes offline, the size of the server pool becomes 3. Using the same hash function, we get the same hash value for a key. But applying modular operation gives us different server indexes because the seas of servers is reduced by 1

- <img width="561" height="290" alt="Screenshot 2026-03-09 at 12 08 29 AM" src="https://github.com/user-attachments/assets/174d3283-60a4-4645-b0e3-320bef2dac04" />
- most keys are redistributed, not just the ones originally stored in the offline server (server 1). This means that when server 1 goes offline, most cache clients will connect to the wrong servers to fetch data. This causes a storm of cache misses. Consistent hashing is an effective technique to mitigate this problem.

### consistent hashing
- "Consistent hashing is a special kind of hashing such that when a hash table is re-sized and consistent hashing is used, only k/n keys need to be remapped on average, where k is the number of keys, and n is the number of slots. In contrast, in most traditional hash tables, a change in the number of array slots causes nearly all keys to be

### Hash space and hash ring
- Now we understand the definition of consistent hashing, let us find out how it works. Assume SHA-1 is used as the hash function f, and the output range of the hash function is: x0, x1, x2, x3, ..., xn. In cryptography, SHA-1’s hash space goes from 0 to 24160 - 1. That means x0 corresponds to 0, xn corresponds to 2/160 — 1, and all the other hash values in the middle fall between 0 and 24160 - 1. Figure 5-3 shows the hash space.
- One thing worth mentioning is that hash function used here is different from the one in “the rehashing problem,” and there is no modular operation. As shown in Figure 5-6, 4 cache keys (key0, key1, key2, and key3) are hashed onto the hash ring
- <img width="546" height="352" alt="Screenshot 2026-03-09 at 12 23 09 AM" src="https://github.com/user-attachments/assets/b387a1ab-01ac-44d6-9e6a-b2c18d53cbc7" />
### ADD SERVER
- Using the logic described above, adding a new server will only require redistribution of a fraction of keys.
- In Figure 5-8, after a new server 4 is added, only key0 needs to be redistributed. k1, k2, and k3 remain on the same servers. Let us take a close look at the logic. Before server 4 is added, key0 is stored on server 0. Now, key0 will be stored on server 4 because server 4 is the first server it encounters by going clockwise from key0’s position on the ring. The other keys are not redistributed based on consistent hashing algorithm.
<img width="516" height="369" alt="Screenshot 2026-03-09 at 12 25 21 AM" src="https://github.com/user-attachments/assets/19d36ade-bb8f-4e2f-b178-d77334aeab52" />

### remove server
- When a server is removed, only a small fraction of keys require redistribution with consistent hashing. In Figure 5-9, when server 1 is removed, only key1 must be remapped to server 2. The rest of the keys are unaffected.
<img width="554" height="349" alt="Screenshot 2026-03-09 at 12 34 42 AM" src="https://github.com/user-attachments/assets/193488a6-3fff-423c-b09e-466d9f0daee9" />
