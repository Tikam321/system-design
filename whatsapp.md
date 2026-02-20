Design WhatsApp
Requirements
Design a chat app like WhatsApp.

Functional Requirements
A user can send and receive text messages in real time.
When the user goes online, she receives all the messages that were addressed to her during her offline time.
By default, once a message is delivered to a user, it will be stored on the user's device and we no longer need to store them on the server (like WhatsApp and WeChat). However, users have the option to modify their history storage policy. They can choose to have their messages stored on the server for up to 90 days.
The users shouldn't see duplicate messages.
Non-Functional Requirements
100M DAU
20 average daily messages per user
The maximum length of a message is 1000 English characters.
Store the undelivered chats for 1 year.
The system provides high availability.
Data must be persistent when we respond that the write is successful.
Resource Estimation
Assume each user sends 20 messages per day, assume each message, with metadata, is roughly 200 bytes.

Send/Receive QPS

100M * 20 / (3600 * 24 ) ~= 23K

To plan for the peak hour, we should aim for 230K RPS (Assuming ten times the average value).

Storage

The requirement to store for 1 year is not for all data, but only for data that has not been delivered to the recipient. For a real-time chat app, most messages will be received by the recipient's client shortly after they are sent. Additionally, users have the option to store chat records for 90 days, but not all users will choose this option. Taking these factors into account, we can assume that the average storage duration for messages is 4 months. Therefore, the storage space only needs to accommodate the amount of data for 4 months.

100M * 20 * 200 * 31 * 4 ~= 50 TB

machines

Modern web servers can handle a large number of WebSocket connections. WhatsApp's fine-tuned Linux machine could already handle up to 2 million simultaneous WebSocket connections in 2012. Let's assume 10% of the 100MM users will be online at the same time, and 1 machine can handle 1MM connections. We will need 10 machines to handle all the WebSocket connections.
