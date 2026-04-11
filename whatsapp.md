# Design WhatsApp

## Introduction

The goal is to design a large-scale chat application similar to WhatsApp. The system must support real-time messaging, offline message delivery, high availability, and persistent storage. The design should handle one hundred million daily active users efficiently and reliably.

# Functional Requirements

1. Users should be able to start group chats with multiple participants (limit 100).
2. Users should be able to send/receive messages.
3. Users should be able to receive messages sent while they are not online (up to 30 days).
4. Users should be able to send/receive media in their messages.

# Non-Functional Requirements

1. Messages should be delivered to available users with low latency, < 500ms.
2. We should guarantee deliverability of messages - they should make their way to users.
3. The system should be able to handle billions of users with high throughput (we'll estimate later).
4. Messages should be stored on centralized servers no longer than necessary.
5. The system should be resilient against failures of individual components.


#  core entities
 <img width="677" height="667" alt="Screenshot 2026-02-20 at 11 12 09 PM" src="https://github.com/user-attachments/assets/3795efbc-01a7-4218-9dbe-ae9adcf5c154" />

1. user
2. usergroup
3. group
4. message
5. messageInbox(offline user info)



# API design
1. create chat api
- // -> createChat
```java
{
    "participants": [],
    "name": ""
} -> {
    "chatId": ""
}
```
2. send message
- // -> sendMessage
```java
{
    "chatId": "",
    "message": "",
    "attachments": []
} -> {
    "status": "SUCCESS" | "FAILURE",
    "messageId": ""
}
```
3. send media
- // -> createAttachment
```java
{
    "body": ...,
    "hash": 
} -> {
    "attachmentId": ""
}
```
4. add/remove participants
// -> modifyChatParticipants
```java
{
    "chatId": "",
    "userId": "",
    "operation": "ADD" | "REMOVE"
} -> "SUCCESS" | "FAILURE"
```
# High level design
- 1) Users should be able to start group chats with multiple participants (limit 100)
<img width="738" height="214" alt="Screenshot 2026-04-10 at 12 06 36 AM" src="https://github.com/user-attachments/assets/3775b4e6-cc56-45e8-9e44-ecb57f3e1f06" />


- So, to send a message:
- Sender sends a sendMessage message to the Chat Server.
- The Chat Server looks up all participants in the chat via the ChatParticipant table.
- The Chat Server (a) writes the message to our Message table and (b) creates an entry in our Inbox table for each recipient.
- The Chat Server returns a SUCCESS or FAILURE to the sender with the final message id.
- The Chat Server looks up the websocket connection for each participant and attempts to deliver the message to each of them via newMessage.
(For connected clients) Upon receipt, the client will send an ack message to the Chat Server to indicate they've received the message. The Chat Server will then delete the message from the Inbox table.
-  For clients who aren't connected, we'll keep their messages in the Inbox table for some time. Later, when the client decides to connect, we'll:
- Look up the user's Inbox and find any undelivered message IDs.
- For each message ID, look up the message in the Message table.
- Write those messages to the client's connection via the newMessage message.
- Upon receipt, the client will send an ack message to the Chat Server to indicate they've received the message.
- The Chat Server will then delete the message from the Inbox table.

2) Users should be able to send/receive messages.
- To allow users to send/receive messages, we're going to need to start taking advantage of the websocket connection that we established. To keep things simple while we get off the ground, let's assume we have a single host for our Chat Server.
- This is obviously a terrible solution for scale (and you might say so to your interviewer to keep them from itching), but it's a good starting point that will allow us to incrementally solve those problems as we go.
- When users make Websocket connections to our Chat Server, we'll want to keep track of their connection with a simple in-memory hash map which will map a user id to a websocket connection. This way we know which users are connected and can send them messages
- To send a message:
- 1. User sends a sendMessage message to the Chat Server.
- 2. The Chat Server looks up all participants in the chat via the ChatParticipant table.
- 3. The Chat Server looks up the websocket connection for each participant in its internal hash table and sends the message via each connection.

3) Users should be able to receive messages sent while they are not online (up to 30 days).
-  1. Let's keep an "Inbox" for each user which will contain all undelivered messages. When messages are sent, we'll write them to the inbox of each recipient user.
-  2.  If they're already online, we can go ahead and try to deliver the message immediately. If they're not online, we'll store the message and wait for them to come back later.
-  3.  How much write throughput does this add? The vast majority of chats are 1:1, and the average user sends about 20 messages per day. With 200M active users, that's 4B messages/day or roughly 40K messages/second.
-  4.  For each message in a 1:1 chat, we write once to Messages and once to Inbox (for the recipient). Even accounting for group chats, we're looking at roughly 100K writes/second — well within DynamoDB's capabilities with userId as the partition key.

<img width="734" height="474" alt="Screenshot 2026-04-10 at 10 03 40 AM" src="https://github.com/user-attachments/assets/d2bc0945-98a5-4996-a90b-83d9a21c9ca1" />

So, to send a message:
- 1. Sender sends a sendMessage message to the Chat Server.
- 2. The Chat Server looks up all participants in the chat via the ChatParticipant table.
- 3. The Chat Server (a) writes the message to our Message table and (b) creates an entry in our Inbox table for each recipient.
- 4. The Chat Server returns a SUCCESS or FAILURE to the sender with the final message id.
- 5. The Chat Server looks up the websocket connection for each participant and attempts to deliver the message to each of them via newMessage.
- (For connected clients) Upon receipt, the client will send an ack message to the Chat Server to indicate they've received the message. The Chat Server will then delete the message from the Inbox table.
-  1. For clients who aren't connected, we'll keep their messages in the Inbox table for some time. Later, when the client decides to connect, we'll:
- 2. Look up the user's Inbox and find any undelivered message IDs.
- 3. For each message ID, look up the message in the Message table.
- 4. Write those messages to the client's connection via the newMessage message.
- 5. Upon receipt, the client will send an ack message to the Chat Server to indicate they've received the message.
- 6. The Chat Server will then delete the message from the Inbox table.
- Finally, we'll need to periodically clean up the old messages in the Inbox and messages tables. We can do this by setting a TTL on the items of the tables.

4) Users should be able to send/receive media in their messages.
- An ideal approach is that we give our users permission (e.g. via pre-signed URLs) to upload directly to the blob storage.
- As an example, they might send a getAttachmentTarget message to the Chat Server which returns a pre-signed URL. Once uploaded, the user will have a URL for the attachment which they can send to the Chat Server as an opaque URL.

 <img width="676" height="342" alt="Screenshot 2026-04-10 at 1 23 45 PM" src="https://github.com/user-attachments/assets/550ecb43-f1a3-4534-81f7-47bbd5a69d75" />

- challanges: Expiring attachments once they've been downloaded by all recipients isn't handled. Managing encryption and security will require extra steps.


## DEEP DIVES
1) How can we handle billions of simultaneous users?
- If we have 1b users, we might expect 200m of them to be connected at any one time. Whatsapp famously served 1-2m users per host, but this will require us to have hundreds of chat servers. That's a lot of simultaneous connections (!).
# offload to pub sub 
- Redis Pub/Sub is a good example which uses a very lightweight hashmap of socket connections to allow you to ferry messages to arbitrary destinations.
- With Pub/Sub you can create a subscription for a given user ID and then publish messages to that subscription which are received "at most once" by the subscribeOn connection:
- The difference here with Kafka is that Pub/Sub is not managing storage of messages. Kafka requires significant disk-based overhead per topic (in addition to the messages themselves) since it persists messages to disk and maintains consumer offsets.
- Redis Pub/Sub channels are much lighter — they're essentially in-memory pointers to subscriber connections with no message persistence. That said, we wouldn't run all our pub/sub through a single Redis instance.
- In practice, we'd shard across a cluster of Redis instances (partitioning channels by user ID), with each instance handling a fraction of the total channels.
# On connection:
- When users connect to our Chat Server, that server will connect to Pub/Sub to subscribe to the topic for that user ID.
Any messages received on that subscription are then forwarded on to the websocket connection for that user.
- When a message needs to be sent:
- We publish the message to the Pub/Sub topic for the relevant user ID.
- The message is received by all subscribing Chat Servers.
- Those Chat Servers then forward the message to the user's websocket connection.
- Pub/Sub is "at most once" which means it doesn't guarantee delivery. If there's no subscribers listening or Redis has a transient failure, that message is lost.

- This is acceptable because we write to the Inbox and Message tables before publishing to Pub/Sub. The write path is:
- Write message to Message table + create Inbox entries (durable)
- Return success to sender
- Publish to Pub/Sub for real-time delivery (best-effort)

<img width="687" height="343" alt="Screenshot 2026-04-10 at 6 14 43 PM" src="https://github.com/user-attachments/assets/50f5f3dd-5ff5-46ed-8645-e1528fdfc63a" />

# Should we partition by chat or by user?
- 1 user have 250 chats but each chat have 1:1 participants
- If we partition by chat the total number of channels in the system is 125 per user (250 chats / 2 since each chat is shared). But each chat server still needs to subscribe to 250 channels per connected user (one for each of that user's chats).
- When a message is sent, the server publishes to just 1 channel.
If we partition by user there's 1 channel per user.
- Each chat server subscribes to 1 channel per connected user. When a message is sent, the server publishes to 1 channel (the recipient's).
- In this scenario it should be obvious we prefer to partition by user — 1 subscription per user instead of 250.


- 2. If we partition by chat the total number of channels is 1 per 100 users (since all 100 share a single chat channel). Each chat server subscribes to 1 channel per connected user. When a message is sent, the server publishes to just 1 channel.
- If we partition by user there's 1 channel per user. Each chat server subscribes to 1 channel per connected user. When a message is sent, the server has to - publish to 99 channels (one for each other participant).
- In this scenario, we save on the number of publishes by having channels partitioned by chat.

# 2) What do we do to handle multiple clients for a given user?

- To this point we've assumed a user has a single device, but many users have multiple devices: a phone, a tablet, a desktop or laptop - maybe even a work computer.
- Imagine my phone had received the latest message but my laptop was off. When I wake it up, I want to make sure that all of the latest messages are delivered to my laptop so that it's in sync.
- We can no longer rely on the user-level "Inbox" table to keep track of delivery!

- Let's see if we can account for this with minimal changes to our design.
- We'll need to create a new Clients table to keep track of clients by user id.
- When we look up participants for a chat, we'll need to look up all of the clients for that user.
- We'll need to update our Inbox table to be per-client rather than per-user.
- When we send a message, we'll need to send it to all of the clients for that user.
- On the pub/sub side, nothing needs to change. Chat servers will continue to subscribe to a topic with the userId.

<img width="736" height="470" alt="Screenshot 2026-04-10 at 7 08 55 PM" src="https://github.com/user-attachments/assets/a5f9d1bd-a1d5-4462-bcdc-da80a9f6c016" />

# 3) What happens if the WebSocket connection fails?
- Users often sit on poor network connections. The WebSocket may technically be open, but the connection is functionally severed—we won't know until we try to send a message and it times out.
- TCP keepalives can take minutes to detect a dead connection, which is far too slow for a chat app. How can we make sure our users aren't impacted?

- The Chat Server sends periodic ping messages (every 10-30 seconds) over the WebSocket. The client must respond with a pong within a timeout (say, 5 seconds). If the client doesn't respond, the server closes the connection.
- When the connection closes, the client reconnects and syncs any missed messages from the Inbox. This catches dead connections within seconds rather than minutes.

- challenge: 
- Heartbeats add overhead—with 200M connected users and a 10-second heartbeat interval, that's 20M ping/pong exchanges per second. In practice this is fine (they're tiny messages), but it's worth noting.

# 4) What happens if Redis fails to deliver a message?
- Redis Pub/Sub is "at most once"—if there's no subscriber listening or Redis has a transient failure, the message is lost.
- We've already handled durability by writing to the Inbox before publishing to Pub/Sub (importantly this means all messages will eventually get delivered), but how do we ensure connected clients quickly receive messages that Pub/Sub dropped?
- 1.  periodic polking:
- Connected clients periodically poll the server for missed messages. Every 30-60 seconds, the client sends a "sync" request. The server checks the Inbox table for any undelivered messages and sends them down.
- challange: This adds load proportional to connected users. With 200M connected users polling every 30 seconds, that's ~7M queries/second just for sync checks. You can mitigate with longer intervals, but you're trading latency for load.

- 2. sequence number pe chat (gap detection)
- Each message gets a monotonically increasing sequence number per chat.
- Clients track the last sequence number they've seen. If they receive message #5 but last saw #3, they know they missed #4 and request a re-sync.
- We can generate these sequence numbers in a separate Redis instance with simple INCR (atomic increment) commands.
- challange: Gap detection only works when you do receive a message. If the chat goes quiet after the missed message, you won't detect the gap until the next message arrives. You still need polling as a backstop.

- 3. piggyback sequence on heartbeat:
 - Combine heartbeats (from the previous section) with sequence numbers:
- Global sequence per user: Maintain a single incrementing counter per user. Every message to that user increments their sequence.
- Include sequence in heartbeat: When the server sends a heartbeat ping, include the user's current sequence number.
- Client detects gaps: If the client's local sequence is behind the server's, it immediately requests a sync.
- This gives you fast detection of missed messages (within one heartbeat interval) with minimal additional load because you're already sending heartbeats

- challanges: The global sequence per user requires an atomic counter, which adds coordination. You can use Redis INCR commands, but it's another dependency. The benefit is that a single sequence catches all missed messages regardless of which chat they're in.



