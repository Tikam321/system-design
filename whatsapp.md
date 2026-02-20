# Design WhatsApp

## Introduction

The goal is to design a large-scale chat application similar to WhatsApp. The system must support real-time messaging, offline message delivery, high availability, and persistent storage. The design should handle one hundred million daily active users efficiently and reliably.

# Functional Requirements

1. A user can send and receive text messages in real time.
2. When a user comes online, she must receive all messages that were sent to her while she was offline.
3. By default, once a message is successfully delivered to the recipient, it is stored only on the recipient’s device. The server does not permanently store delivered messages.
4. Users may optionally choose to store their messages on the server for up to ninety days.
5. The system must ensure that users do not see duplicate messages.

# Non-Functional Requirements

1. The system must support one hundred million daily active users.
2. Each user sends an average of twenty messages per day.
3. The maximum length of a message is one thousand English characters.
4. Undelivered messages must be stored for up to one year.
5. The system must provide high availability.
6. When the server responds that a message was successfully sent, the message must already be stored durably.
7. The system must scale horizontally.

# Capacity Estimation

## Message Size

Assume each message, including metadata such as sender identifier, receiver identifier, timestamp, and message identifier, is approximately two hundred bytes.

## Queries Per Second Calculation

Each user sends an average of twenty messages per day.

Total messages per day:

One hundred million users multiplied by twenty messages per user equals two billion messages per day.

Number of seconds in one day:

Twenty-four multiplied by three thousand six hundred equals eighty-six thousand four hundred seconds. 
Average messages per second:
Two billion divided by eighty-six thousand four hundred is approximately twenty-three thousand messages per second.
To handle peak traffic, assume peak load is ten times the average load.
Therefore, the system should handle approximately two hundred thirty thousand requests per second during peak hours.
## Storage Estimation
We do not need to store all messages for one year.
We only need to store:
1. Undelivered messages.
2. Messages for users who enable server-side storage for up to ninety days.
Since most messages are delivered in real time, we assume the average storage duration is four months.

Storage calculation:

One hundred million users multiplied by twenty messages per day multiplied by two hundred bytes multiplied by thirty-one days multiplied by four months equals approximately fifty terabytes of storage.
Therefore, the system must support at least fifty terabytes of storage capacity.

# High-Level Architecture
The system consists of the following main components:
1. Client applications (mobile or web).
2. Load balancer.
3. WebSocket servers.
4. Message service.
5. Persistent storage (database).
6. Redis for real-time message routing.
7. Inbox service for offline messages.

# Real-Time Messaging Flow (Online User)
1. The sender sends a message through a WebSocket connection to the WebSocket server.
2. The WebSocket server forwards the message to the message service.
3. The message service writes the message to the persistent database.
4. Once the message is durably stored, the server sends an acknowledgment to the sender.
5. The message service publishes the message to Redis.
6. The WebSocket server that holds the recipient’s active connection receives the message from Redis.
7. The WebSocket server forwards the message to the recipient’s device in real time.

# Offline Message Flow

1. The sender sends a message.
2. The message is written to the persistent database.
3. An entry is created in the recipient’s inbox table.
4. If the recipient is offline, the message is not delivered immediately.
5. When the recipient reconnects, the client calls a synchronization application programming interface.
6. The server retrieves all undelivered messages from the inbox table.
7. The messages are delivered to the recipient.
8. The inbox entries are marked as delivered.

# Database Design

## Message Table
* Message identifier
* Sender identifier
* Receiver identifier
* Message content
* Timestamp
* Delivery status

The message table should be sharded by user identifier to allow horizontal scaling.

## Inbox Table
* User identifier
* Message identifier
* Delivery status
* Timestamp

This table tracks undelivered messages.
# Real-Time Routing with Redis

Each WebSocket server subscribes only to the Redis channels of users who are currently connected to that server.

When a message needs to be delivered:

1. The system publishes the message to the Redis channel corresponding to the recipient’s user identifier.
2. Redis forwards the message to the subscribed WebSocket server.
3. The WebSocket server forwards the message to the user’s device.

Redis is used only for real-time routing. The database remains the source of truth.


# Handling Duplicate Messages
To prevent duplicate messages:

1. Each message has a globally unique identifier.
2. The client ignores messages with identifiers that were already processed.
3. Database writes are idempotent.
4. Delivery acknowledgments are tracked using message identifiers.

# High Availability Strategy

1. Deploy services across multiple availability zones.
2. Use replication for databases.
3. Use Redis clusters instead of a single instance.
4. Use stateless WebSocket servers to allow horizontal scaling.
5. Implement automatic failover mechanisms.

# Scaling WebSocket Connections

Assume ten percent of users are online simultaneously.Ten percent of one hundred million users equals ten million concurrent connections.
 If one server can handle one million WebSocket connections, then ten WebSocket servers are required. These servers should be placed behind a load balancer.

# Data Durability

The system must write the message to persistent storage before acknowledging success to the sender. This guarantees that messages are not lost even if a server crashes after responding.

### API Endpoint Design
- Messages are sent using WebSocket because it allows real-time, bidirectional communication after a single persistent connection is established. Unlike HTTP long polling, WebSocket removes repeated request-response overhead.
 <img width="677" height="667" alt="Screenshot 2026-02-20 at 11 12 09 PM" src="https://github.com/user-attachments/assets/3795efbc-01a7-4218-9dbe-ae9adcf5c154" />

 1. create group
  - post chat/app/createGroup/
    request  { groupname: "", userId: "", members: []}
    response : 201, {groupId: ""}
 2. send message
  - post chat/app/send
  - requestPayload: {userId: "", content: "", crratedat: "", groupId: "", attachments: []}
  - response: success/fail

 3. add/remove members from group
  - post chat/app/add (remove)
  - requestPayload: {userId: [], groupId: ""}
  - response: success/fail
    

<img width="2090" height="764" alt="image" src="https://github.com/user-attachments/assets/24c77882-7288-4181-a76a-1c6772900106" />
1. Client A sends a message to the Chat Service
```java
{
  "action": "send_message";
  "receiver_id": string;
  "content": string; // Length <= 1000 English char.
  "message_id_by_client": number; // Incremental ID generated on the client side, used to identify messages in the server's confirmation message.
  "timestamp": number;
}
```
2. Chat Service confirms it received Client A's message
```java
{
  "action": "message_received";
  "message_id": string; // Globally unique id generated by the Chat Sever
  "message_id_by_client"; // ID from the send_message message used to match messages on the client side
  "timestamp": number;
}
```
3. Chat Service sends the message to Client B
```java
{
  "type": "incoming_message";
  "message_id": string;
  "timestamp": number;
}
```
4. Client B sends a message to the Chat Service to confirm the message has been successfully delivered
```json
{
  "type": "message_delivered",
  "message_id": string,
  "timestamp": number,
}
```
5. Chat Service sends a message to Client A notifying it that Client B has received the message
```java
{
  "type": "message_delivered",
  "message_id": string;
  "timestamp": number;
}
```

```java
// 1. Client A sends a message to the Chat Service
{
  "action": "send_message",
  "receiver_id": "user_2",
  "content": "Hello",
  "message_id_by_client": 101,
  "timestamp": 1710000000
}

// 2. Chat Service confirms it received Client A's message
{
  "action": "message_received",
  "message_id": "uuid-12345",
  "message_id_by_client": 101,
  "timestamp": 1710000001
}

// 3. Chat Service sends the message to Client B
{
  "type": "incoming_message",
  "message_id": "uuid-12345",
  "sender_id": "user_1",
  "content": "Hello",
  "timestamp": 1710000001
}

// 4. Client B confirms delivery to the Chat Service
{
  "type": "message_delivered",
  "message_id": "uuid-12345",
  "receiver_id": "user_2",
  "timestamp": 1710000002
}

// 5. Chat Service notifies Client A that the message was delivered
{
  "type": "message_delivered",
  "message_id": "uuid-12345",
  "timestamp": 1710000002
}
```

