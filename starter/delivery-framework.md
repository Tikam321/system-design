In this module we will go through the structure process of how we proceed in system design itnerview step by step.

<img width="1489" height="684" alt="Screenshot 2026-04-07 at 10 09 22 PM" src="https://github.com/user-attachments/assets/1e5fc62d-9e49-4f2a-a6b0-104c1cd8b345" />

# Requirement
- The goal of the requirements section is to get a clear understanding of the system that you are being asked to design. To do this,
-  we suggest you break your requirements into two sections.

# 1. functional requirement:
- Functional requirements are your "Users/Clients should be able to..." statements.
- These are the core features of your system and should be the first thing you discuss with your interviewer.
- Users should be able to post tweets
- Users should be able to follow other users
- Users should be able to see tweets from users they follow

- # 2. Non functional requirement:
- These can be phrased as "The system should be able to..." or "The system should be..." statements.
- The system should be highly available, prioritizing availability over consistency
- The system should be able to scale to support 100M+ DAU (Daily Active Users)
- The system should be low latency, rendering feeds in under 200ms
- You'll want to identify the top 3-5 that are most relevant to your system.
1. CAP Theorem:
2. scalability
3. latancy
4. durability
5. security
6. fault tolerance
# TRICK TO REMEBER
"SCALE For Cloud DesignS"

S- Scalability
CA - CAP Theorem
L - Latency
E - Environment Constraints.
F - Fault Tolerance
C - Compliance
D - Durability
S  - Security
- # 3.  capacity estimation
- not neccesserality required but can show if requried

# core entities
- For our Twitter example, our core entities are rather simple:
- User
- Tweet
- Follow

# API DESIGN/INTERFACE
```java
POST /v1/tweets
body: {
  "text": string
}

GET /v1/tweets/{tweetId} -> Tweet

POST /v1/follows
body: {
  "followee_id": string
}

GET /v1/feed -> Tweet[]
```
# [Optional] Data Flow (~5 minutes)
- for data prcessing system required to explain the flow of data
- eg. For a web crawler, this might look like:
- Fetch seed URLs
- Parse HTML
- Extract URLs
- Store data
- Repeat

# High Level Design (~10-15 minutes)
- Now that you have a clear understanding of the requirements, entities, and API of your system, you can start to design the high-level architecture.
- This consists of drawing boxes and arrows to represent the different components of your system and how they interact.
- Components are basic building blocks like servers, databases, caches, etc.
- This can be done either in person on a whiteboard or virtually using whiteboarding software like Excalidraw.
- Don't over think this! Your primary goal is to design an architecture that satisfies the API you've designed and, thus, the requirements you've identified.
- In most cases, you can even go one-by-one through your API endpoints and build up your design sequentially to satisfy each one.

- <img width="1488" height="734" alt="Screenshot 2026-04-07 at 10 31 02 PM" src="https://github.com/user-attachments/assets/60db6d84-0f02-46d0-a79f-11bd5f3fb159" />
# Deep Dives (~10 minutes)
- Validate non-functional requirements (scale, latency, availability)
- Handle key edge cases (celebrity users, traffic spikes)
- Identify major bottlenecks (feed generation, DB load)
- Apply scaling (horizontal scaling, sharding)
- Use caching to reduce latency
- Optimize feeds (fanout-on-read vs fanout-on-write)
- Iterate and refine design based on trade-offs


