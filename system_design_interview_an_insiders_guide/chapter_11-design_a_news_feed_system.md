# Chapter 11: Design A News Feed System
## Step 1 - Understand the problem and establish design scope
- Interface
    - Mobile and Web.
- Features
    - Push post and see posts.
- Order
    - Sorted reverse chronolohical order.
- How many friends
    - 5k
- Traffic volume
    - 10M DAU
- Content Type
    - Media (video/images).

## Step 2 - Propose high-level design and get buy-in
Flows:
- Feed publishing
- New feed building

### Newsfeed APIs
#### Feed publishing API
HTTP 
*POST /v1/me/feed*
- content
- auth_token

#### Newsfeed rertieval API
HTTP
*Get /v1/me/feed*
- auth_token

### Feed publishing
- User: Web/Mobile. Post.
- Load balancer: Distribute traffic.
- Web servers: Redirect traffic.
- Internal services
    - Post service: Persist post in database and cache.
    - Fanout service: Push new content to friend's news feed. Newsfeed data is in cache for fast retrieval.
    - Notification service: Inform friends that new content is available.

### Newsfeed building
- User: Web/Mobile. View.
- Load balancer.
- Web servers.
- Internal service:
    - Newsfeed service: Fetch news feed from the cache.
    - Newsfeed cache: Store news feed IDs needed to render the news feed.

## Step 3 - Design deep dive

### Feed publishing deep dive
#### Web servers
***Authentication and rate-limiting***.

#### Fanout service
##### ***Fanout on write (Push model)***

Newsfeed is pre-computed during write time. Delivered to friends' cache immediately after it is published.

Pros:
- Realtime. Pushed to friends immediately.
- Fetching newsfeed is fast.

Cons:
- Hotkey problem: Users with many friends is slow to fetching the friend list and generating news feeds.
- Inactive users: Waste of compute resources.

##### ***Fanout on read (Pull model)***

Newsfeed is generated on-demand during read time. Recent posts are pulled when a user loads her home page.

Pros:
- Inactive users: No waste.
- Not pushed so no hotkey problem.

Cons:
- Fetching the news is slow.

##### Hybrid approach

- Use a push model for majority of user. 
- Use a pull model for or celebrities posting or inactive users receiving.
- Consistent hashing to distribute requests/data more evenly.

##### Fanout service workflow
- 1. Fetch friend IDs from graph database.
- 2. Get friend info from the user cache. Filter out friends based on settings. (muted, close friends, etc.)
- 3. Send friends list and new post ID to the message queue.
- 4. Fanout workers fet ch data from message queue and store news feed data in cache.
    - Store ids in cache, not original content. *<post_id, user_id>*.
    - Set configurable limit to evit old items from cache.
- 5. Store *<post_id, user_id>* in newsfeed cache.

### Newsfeed retrieval deep dive

CDN to store media content.

#### Workflow
- User and retrieve request. *Get /v1/me/feed*.
- Loadbalancer.
- Webserver.
- Newsfeed service
    - Get a list post IDs from the *newsfeed cache*.
    - Fetch the complete user and post object from *user cache* and *post cahce* to construct the fully hydrated newsfeed.
    - Return fully hydrated new feed in JSON format.

#### Cache architecture
- Newsfeed: news feed cache.
- Content: hot cache. normal.
- Social graph: follower. following.
- Action: liked. replied. others.
- Counters: likes. replies. others.

## Step 4 - Wrap Up

### Scalability
- Vertical sharding vs Horizontal sharding.
- SQL vs NoSQL.
- Master-slave replication.
- Consistency models.
- Database sharding.

### Other talking points
- Web tier stateless
- Cache as much as possible
- Multiple datacenters
- Lose components with message queues
- Monitor key metrics: QPS during peak hours and latency while refreshing newsfeed.
