# 0. Overview
- Web scaling
  - Data centers
    - Users/servers bound into groups
  - Load Balancer
    - Running multiple servers
  - Stateless
    - Servers detached from sessions
  - Message Queue
    - Task server detached from web servers
- Database scaling
  - Replication
    - Master/slave to handle store/read
  - Sharding
    - Distributes workload to different dbs
- Cache Improvements
  - Generic cache.
    - Expiration, evictions, synchronizations.
  - CDN
    - Cache static data w/o web servers.
- Logging/Metrics/Automation

# 1. Single Server Setup 

## Request Flow
- Users -&gt; domain name -&gt; DNS (Domain Name System) Server.
- DNS -&gt; IP (Internet Protocol) address -&gt; Users.
- Users -&gt; HTTP (Hypertext Transfer Protocol) Request -&gt; Web Server.
- Web Server -&gt; HTML / JSON -&gt; Users.

## Traffic Source
- Web Application
  - Server side (Java, Python, etc.) to handle business logic, and storage.
  - Client side (HTML, JavaScript, etc.) for presentation.
- Mobile Application
  - HTTP is a communication protocol. 
  - JSON(JavaScript Object Notation) is response format.

# 2. Database

Separate web/mobile traffic (web tier) and database (data tier).

## Relational Database
- Relational database management system (RDBMS) or SQL database.
- MySQL, Oracle database, PostgreSQL, etc.
- Presents and stores data in tables and rows. Perform join operations across different database tables via SQL.

## Non-Relational Database
- NoSQL database.
- CouchDB, Neo4j, Cassandra, HBase, Amazon DynamoDB.
- Categories:
  - Key-value stores. 
  - Graph stores.
  - Column stores.
  - Dodument stores.
- Join operations are not supported.

## What to use
Choose relational by default. Use non-relational if:
- Application requires super-low latency.
- Unstructured data.
- Serialize and deserialize.
- Store massive amounts of data.

# 3. Vertical scaling vs Horizontal scaling
- **Vertical scaling**
  - Scale up, adding more power (CPU, RAM, etc) to your servers.
  - Pros:
    - Simplicity. Suitable when traffic is low.
  - Cons:
    - Hard limit. Cannot add unlimited power to a server.
    - No failover and redundancy. One server goes down, and the web/app goes done.
- **Horizontal scaling**
  -  Scale out, adding more servers to your pool of resources.

# 4. Load Balancer (Web tier scaling)
A load balancer evenly distributes incoming traffic among web servers that are defined in a load-balanced set.
- Users connect to the load balancer through public IP.
- Load balancer communicates with web servers through private IPs.

How it resolves problems of failover and availability:
- One server is offline, traffic goes to other servers, while a new server is being added.
- Too much traffic, add new servers, load balancer automatically sends requests to them.

# 5. Database Replication (Data tier scaling)
Master (original) / salve (copies) relationship.
- Master only supports write.
  - Data modifying operations send to master, e.g. insert, delete, update, etc.
- Slaves only supports read operations.
  - More slaves than master as higher ratio of reads than writes.

Advantages:
- Better performance: Parallel query.
- Reliability: Data is replicated across all locations (masters/slaves).
- Availability: Access data stored in another database if offline.
  - Slave goes offline.
    - One slave. Reads redirected to master.
    - Multiple slaves. Reads redirected to healthy slaves.
    - Adds new slave.
  - Master goes offline.
    - One slave was promoted to a new master. Add a new slave.
      - Slave data is not up to date.
        - Recover missing data by running data recovery scripts. 
        - Or multi-master and circular replication.

# 6. Cache (Performance Improvement)

## Cache tier
- Web server -&gt; Request -&gt; Cache hit -&gt; Web server
- Web server -&gt; Request -&gt; Cache miss -&gt; Database -&gt; Cache store -&gt; Web server (Read through)

Pros:
- Better performance,
- Reduce database workloads.
- Scale cache tier independently.

Considerations:
- **Use when data is read frequently but modified infrequently**. Cache data is stored in volatile memory, not ideal for persisting data.
- **Consistency**. Keeping data store and cache in sync. Inconsistency when not transactional. 
- **Expiration policy**. Too long -&gt; data become stale. Too short -&gt; reload from database frequently.
- **Evication policy**. LRU (least recently used). LFU (least frequently used). FIFO.
- **Migrating failures**. Multiple cache servers across different data centers to avoid a single point of failure. (SPOF) or overprovision the required memory.

# 7. Content delivery network (CDN. Cache w/o web tier)

A network of geographically dispersed servers used to deliver static content (image, videos, CSS, JavaScript. etc). (**Skipping web servers**)

Workflow:
- User access image by URL.
- CDN server requires image form origin.
- The origin returns the image to CDN server, including HTTP header Time-to-Live(TTL).
- CDN caches the image until TTL and returns.
- Following user requests get image from CDN server.

Considerations:
- Cost: CDN are run by third-party providers.
- Expiration policy.
- CDN fallback: If CDN outage, users detect problem and request origin.
- Invalidating files: 
  - Using APIs from CDN vendors. 
  - Using object versioning, by adding a parameter to URL.

# 8. Stateless web tier (Web tier scaling)

To scale web tier horizontally, move the state out of web tier by storing session data in persistent storage (SQL/NoSQL).

## Stateful architecture

All requests from the same client must be routed to the same server. Can be done with sticky sessions in load balancers, but add overhead on adding/removing servers.

## Stateless architecture

HTTPs requests from users can be sent to any web server. Fetch data from a shared data storage. Simpler, robust and scalable.

Use NoSQL as it is easy to scale.

# 9. Data Ceneters (Web/Data/Cache tier scaling)

Users are geoDNS-routed, or geo-routed, to the closest data ceneter. In the event of significant data center outage, redirect to a healthy data center.

- Server, database, Cache, are spread across data centers.
- Session data (NoSQL), is shared across data centers.

Challenges:
- **Traffic redirection**: Direct users to correct data center. GeoDNS.
- **Data synchronization**: 
  - Healthy: Different users from different regions use different local databases or caches.
  - Failover: Traffic is migrated where data is unavailable. Replicate data.
- **Test and deployment**: Test on different locations. Automated deployment tools.

# 10. Message Queue (Web tier horizontal scaling and communication)

Durable, in memory component, to support asynchronous communication. Buffer to distributes asynchronous requests.

- Producers/Publishers: Create messages, publish to queue.
- Consumers/Subscribers: Listen to queue, perform actions defined by the messages.

Decoupling:
- Producer (web server) post message when consumer is unavailable.
- Consumer (dedicated worker) read message when producer is unavailable.
- Producer and consumer can be scaled independently.

# 11. Logging, Metrics, Automation

- Logging: Mointor error.
  - Per server level.
  - Use tools to aggregate them to a centralized service.

- Metrics: Gain business insights and understand health status.
  - Host level metrics: CPU/Memory/Disk
  - Aggregated level metrics: Database tier, cache tier, etc.
  - Key business metrics: Daily active users, revenue, etc.

- Automation: CICD. Build, test, deployment.

# 12. Database Scaling

## Vertical Scaling

Scaling up. Adding more power to an existing machine. Example: Amazon Relational Database Service (RDS) 24TB of RAM. Cons:
- Hardware limits.
- SPOF.
- Cost.

## Horizontal Scaling

Scaling out/Sharding. Separates large databases into smaller, easily managed parts called shards, sharing schema but unique data.

Choose sharding key: Help to route data to correct database. Choose one that can evenly distribute data.

Challenges:
- Resharding data: Updating sharing and moving data. -&gt; Consistent sharding.
  - A single shard could no longer hold more data due to rapid growth.
  - Certian shards exprience shard exhaustion faster due to uneven distribution.
- Celebrity problem: Excessive access to a specific shard cause server overload. -&gt; Allocate shard for each celebrity, even further partition.
- Join and de-normalization: Hard to perform join operations across database shards. -&gt; De-normalize so queries can be performed in a single table.

# 13. Millions of users and beyond

- Keep web tier stateless
- Build redudancy at every tier
- Cache data as much as you can
- Support multiple data centers
- Host static assets in CDN
- Scale your data tier by sharding
- Spit tiers into individual services
- Monitor your system and use automation tools

