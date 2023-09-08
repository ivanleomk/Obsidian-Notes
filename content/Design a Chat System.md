
# Questions

I came up with some questions of

- What type of chats are we supporting? (Eg. 1-on-1, group chats)
- How many users are using the app?
- How many messages are going to be sent per minute?
- Is this a mobile or web app?

Some questions that were suggested were

- What is the maximum limit for group chat?
- Does the chat app need to support end-to-end encryption
- How long do we store chat history for?
- Is there a max length for messages?
- What features are important for the chat app? Can it support attachments

# Overview

## Connection Types

![[Pasted image 20230908210859.png | 400]]

There are a few different methods for our client to remain connected to our servers

1. **Simple HTTP** : Where a single request is sent
2. **Polling** : Where we send periodic requests over to the server. We can also use long polling where the client holds a connection open until there are actually new messages available or a timeout threshold has been reached.
3. **Web sockets** : Where we maintain a persistent connection to our server

Ideally in a chat application, we have a mix of these so that we only maintain web sockets for tasks that require a persistent connection. This is because websockets require a stateful server and a persistent connection is **expensive** to maintain. We also want to reduce the use of polling since it's a rather inefficient mechanism to update our database. 

### Websockets

![[Pasted image 20230908212008.png | 400]]

Websockets are a bidirectional and persistent form of connection. They start as HTTP connections and eventually get upgraded via a TCP handshake to a WebSocket connection. WebSocket connections generally work even if a firewall is in place. This is because they use port 80 or 443 which are also used by HTTP/HTTPS connections.


# Implementation

![[Pasted image 20230908212142.png | 400]]

We ideally want to use a few different types of services as seen above

> Stateful services store and manage user session data on the server-side, while stateless services rely on clients or shared storage to maintain session data.

We keep our auth, service discovery, group management and user profile services stateless since they do not require a persistent connection. Users simply need to make a request and get some data back in return using a HTTP request.

Our chat service is stateful because we need our client to maintain a persistent network connection to a chat server.  This helps down the line when we need to implement features such as

- Determining if a user is online or not
- Syncing user chat history
- Send messages to user devices

All of these features are significantly easier to implement when using a persistent connection such as a web socket. 

## Chat Service

Chat services tend to have a high read to write ratio. They also have long tails, which means that the data they store can stretch for either very long durations or large quantities. As a result, this means that any [[Relational Databases]] will struggle when dealing with this since the computed index will be very large. 

Our data access layer also needs to support a random access of data - such as search, view mentions or even jump to specific messages. 

As a result, this makes [[Non Relational Databases]] a good choice for this use case. This is because they 

- Allow for easy [[Horizontal Scaling]]
- Provide low latency access to data - significantly faster than [[Relational Databases]]
- Have a track record of being adopted by other reliable chat applications - for instance Facebook uses [[Hbase]] while Discord uses [[Cassandra]]

We can store information on our chats by using the following table

%% TODO: Clarify how this KV Store might work exactly? %%

```
1-to-1 MESSAGE
message_id bigint
messaage_from bigint
message_to bigint
content text
created_at timestamp
```

where message_id is the primary key

```
GROUP CHAT MESSAGE
channel_id bigint
message_id bigint
user_id bigint
content text
created_at timestamp
```

where `(channel_id,message_id )` forms a composite primary key and we perform a partition using the `channel_id` since all queries in a group chat operate in a channel. We then generate group level and chat level unique message_ids that are local within a group. This means that they are only used to maintain a message_sequence within a one-on-one channel or a group channel.

## Service Discovery

We want to be able to quickly recommend users a chat server based on some predefined criteria. This can be done using a tool like [[Apache Zookeeper]] which registers all existing chat servers and then picks the best server for a client.

![[CleanShot 2023-09-08 at 22.22.50@2x.png | 500]]

In the event that a client's server goes down, we can immediately recommend a second chat server according to some predefined criteria.

## 1 on 1 Messages

We can visualise the message flow using this diagram

![[Pasted image 20230908222506.png | 400]]

When users send a message to the chat server that was assigned to them, they kickstart the process. First, we obtain a new message id from a ID generator which is local and unique to the chat. Then, the message is sent to a message sync queue, which is responsible for sending the message over to the end user. 

When we process the message, we 

- Store it in a KV Store
- If the user is online, we forward it to chat server 2 where he is connected.
- If the user is not online, then we forward it to a push notification service which will forward a push notification to the user

## Group Messages

We can extend the process in [[#1 on 1 Messages |above]] to support group messages in two possible ways

- **Queues by User** : We can allocate a single queue for each user to receive messages for all his groups

![[Pasted image 20230908224027.png | 300]]

- **Queues by Groups** : Each user in a group has a unique queue to receive messages in. This is done in Wechat.

![[Pasted image 20230908224043.png | 300]]

Generally we can use the second method initially because it's not too cumbersome to store a copy of each message in the queue for every member in the group when a new message is sent. However, once we reach enough scale, going for the second method is significantly more efficient.


## Synchronising Messages

%% TODO: Figure out whether this is max message for each indv group or  %%

We maintain a count of the most recent message id and then back propogate from there.


## Online Status

Client status is stored in a KV store with the following information

```
USER A : { status: online | offline, last_active_at: timestamp }
```

When a user comes online and establishes a websocket connection, we then update his status as online. When the user breaks the websocket connection, we update the status as offline and store the timestamp.

However, since users frequently connect and disconnect from the internet, we can use a heartbeat mechanism to solve the problem of identifying if a user is online or offline. Every 5s or so, the device will send a heartbeat event to the server. In the event that our client is disconnected and does not reconnect within x = 30 seconds, then we change his online status to offline.

### Updating other Users

We can update other users by using a [[publish-subscribe]] model whereby we update friends/small selection of a user's friends that are subscribed to his updates. This happens through a real-time websocket.

# Extensions

Some potential extensions are 

- Extend the chat app to support media files such as photos and videos. Media files are significantly larger than text in size. Compression, cloud storage, and thumbnails are interesting topics to talk about.
    
- End-to-end encryption. Whatsapp supports end-to-end encryption for messages. Only the sender and the recipient can read messages. Interested readers should refer to the article in the reference materials [9].
    
- Caching messages on the client-side is effective to reduce the data transfer between the client and server.
    
- Improve load time. Slack built a geographically distributed network to cache users’ data, channels, etc. for better load time [10].
    
- Error handling.
    
- The chat server error. There might be hundreds of thousands, or even more persistent connections to a chat server. If a chat server goes offline, service discovery (Zookeeper) will provide a new chat server for clients to establish new connections with.
    
- Message resent mechanism. Retry and queueing are common techniques for resending messages.