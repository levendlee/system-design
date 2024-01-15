# Chapter 12: Design a Chat System

## Step 1 - Understand the problem and establish design scope
- Chat Type
    - 1-1 and group caht.
- Mobile/Web
    - Both
- Scale
    - 50M DAU
- Group chat limit
    - 100 people maximum
- Features
    - 1-1 chat. Group chat. Online indicator.
- Message size limit
    - < 100K characters.
- End to end encryption required?
    - Not required.
- Chat history?
    - Forever.

## Step 2 - Propose high-level design and get buy-in
Clients communicates with each other through **Chat service**, which supports:
- *Receive messages*.
- *Find recipients*.
- *Determine online status*, send if online, hold if offline.

### Network protocols

Client connects with chat service with one or more network protocols.

#### Requests (Senders connection)
Sender->Chat service. Client-initiated. 
- Using time-tested HTTP protocol.
    - Opens a HTTP connection with chat service and sends message.
    - Keep-alive header to:
        - Maintain a persistent connection.
        - Reduce the number of TCP handshakes.

#### Response (Receive connection)
Chat service->Receiver. Server-initiated.

##### *Polling*
Client periodically asks the server if message available. 
- Client query periodically.
- Server response immediately.

Cons:
- Costly.

##### *Long Polling*
Client holds connection open until there are actually new messages available or timeout.
- Client query immediately.
- Server response delayed until new message.

Cons:
- Sender and receiver may not connect to same chat server + HTTP based servers are stateless.
- A server has no good way to tell is a client is disconnected.
- Inefficient. Periodic connections after timeouts and keep connects alive.

##### ***Websocket***
Most common solution for sending asynchronous updates from server to client.
- Initiated by client.
- (Client sent) HTTP Handshake -> (Server sent) Acknowledgement -> Birectional messages.
- Birectional and persistent.
- Still works with firewall, port 80 or 443.

Persistent and efficient, critical on server-side. Use both for sending and receiving. Simplifies.

### High-level design

#### Stateless
- User -> Load Balancer -> Service discovery/Authentication service/Group management/User profile

Traditional public-facing request/response services. Login/SignUp/UserProfile.

#### Stateful
- User 1 -> (Websocket) -> Chat service/Presence service -> (Websocket) -> User 2

Each client maintains a persistent network connection to a chat server. Doesn't switch as long as server is still available.

#### Third Party
Push notification

### Scalability
Start with a single server. But must mention it is a starting point and we will scale.

- Chat servers: Messaging sending/receiving.
- Presence servers: Manage online/offline status.
- API servers: Login/SignUp/Profile.
- Notification servers: Send push notifications.
- K-V storage: Store chat history, offline users can receive message when they become online.

### Storage
#### Generic data
User profile, settings, friends list. 
- Stored in robust and reliable relational databases.
- Replication and sharding are common techniques to meet availability and scalability requirenments.

#### Chat history
Data characteristics
- Amount of dta is enormous.
- Only recent chats are accessed frequently.
- Support random access. Search, view mentionds, jump to message.
- Read to write ratio is 1:1 for 1 on 1 chats.

Select ***K-V Store***:
- K-V allows easy horizontal sharding.
- K-V provides low latency to access.
- Relational doesn't handle long tail of data well. Indexes grow large, random access is expensive.
- K-V adopted by other reliable chat applications.
    - Facebook messager: HBase.
    - Discord: Cassandra.

#### Data Models

- 1 on 1 chat: `message_id` (primary key, determine sequence), `message_from`, `message_to`, `content`, `created_at`.

- Group chat: `channel_id` (primary key), `message_id`(primary key), `user_id`, `content`, `created_at`.

##### Message ID
Requirement:
- Unique.
- Sortable by time.

Implementation:
- `auto_increment` in MySQL. (NoSQL doesn't provide).
- Global 64-bit sequence number. 
- Local seuquence number generator.
    - Sufficient for one-to-one channel or a group channel.

## Step 3 - Design deep dive

### Service discovery
Recommend the best chat server for a client based on critias:
- Geographical location.
- Server capacity.

Apache Zookeeper is a popular open-source solution. Register all chat servers and pick based on predefined criteria.

- User -> Load Balancer -> API server -> Service discovery -> Recommend chat server

### Message flows
- User A -> Chat Server 1.
- Chat server 1 obtains ID from ID generator.
- Chat server 1 sends messages to message sync queue.
- Message stored in K-V store.
- If user B online:
    - Message forward to Chat server 2 where B is connected.
    - Chat server 2 forwards the message to B. (w/ persistent WebSocket connection.)
- If user B offline:
    - Push notification send to third_party servers.

### Message synchronization across multiple devices
Different devices connect to the same chat server w/ WebSocket.

Each device maintains a variable called *cur_max_message_id*, keeps track of the latest message ID. Login and read messages from K-V store. New messages as:
- As recipient.
- Message ID in the k-v store is larger than *cur_max_message_id*.

### Small group chat flow
Sender
- Keeps track of other members in group.
- Sends messages to dedicated message sync queue to other members.

Receiver
- Checks its message sync queue populated by other members.

Okay for small but not okay for large group chat to duplicate message for each member.

### Online presence

#### User login
User's online status and *last_active_at* timestamp are saved in KV store.

#### User Logout
User's online status updated.

#### User disconnection
Avoid updating online status frequently due to small variations.

##### Heartbeat mechanism
Periodically sends a heartbeat event to presence servers. Considered offline if not received in *X* seconds.

##### Online status fanout
Publish-subscribe model. Presence servicers -> Channels -> Other users.

Okay for small but not okay for large group chat. Large group chat only update when enter or manually refresh.

## Step 4 - Wrap Up

### Recap

- Websocket
- Components
    - Chat servers
    - Presence servers
    - Push notification servers
    - Key-value stores: chat history
    - API servers

### Additions

- Extend to support Media
    - Compression. Cloud storage. Thumbnails.
- E2E encryption
- Caching messages on client
- Improve load time
    - Geographically distributed network.
- Error handling
    - Chat server error.
        - Service discovery provides a new chat server to establish new connections with.
    - Message resent mechanism. 
        - Retry and queueing are common for resending messages.