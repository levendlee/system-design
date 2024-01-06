# Goal
- Prevent resource starvation caused by Denial of Service (DoS) attack.
- Reduce cost.
- Prevent servers from being overloaded.

# Design
## Step 1 - Understand the problem and establish design scope
- Typical questions: Client-side/Server-side? IP/ID/... based? Scale? Distributed? Separate service/Application code? Info users?
- Typical requirements:
  - Accurate.
  - Low latency.
  - Distributed. Shared across multiple servers or processes.
  - Exception handling. Show users exceptions that their requests are throttled.
  - High fault tolerance. Doesn't impact system if down.

## Step 2 - Propose high-level design and get buy-in
### Where to put rate limiter?
- Client-side: Not reliable.
- Server-side: Doable.
- Middleware: Recommended. (Return status code 429 then throttled.)
  - API getaway (cloud micro-service): Fully managed service supports rate limiting, SSL termination, authentication, IP whitelisting, servicing static content, etc.
- Server-side vs Middleware:
  - (Pro-middleware) Evaluate current tech stack and make sure your current programming language is efficient to implement rate limiting.
  - (Pro-server) Identify the rate limiting algorithm fits business needs. Implement everything on server-side has full control.
  - (Pro-middleware) If already have mirco-service w/ API getaway, add rate limiter there is easy.
  - (Pro-middleware) Build own rate limiting service take time.

### Algorithm for rate limiting
#### Token bucket algorithm
  - Algorithm
    - (Amazon, Stripe)
    - A token bucket is a container with pre-defined capacity filled with pre-defined rate periodically.
    - When token is full, no more tokens are added.
    - Each request consumes one token.
    - When token is empty, request is dropped.
  - Parameters
    - Bucket size: Maximum number of tokens allowed.
    - Refill rate: number of tokens put in bucket each second.
  - Requires different buckets for different rules (APIs/IPs/global/etc.).
  - Pros
    - Easy.
    - Memory efficient.
    - Allows a burst of traffic for short period.
  - Cons
    - Challenging to tune parameters.
#### Leaking bucket algorithm
  - Algorithm
    - (Shopify)
    - FIFO.
    - When request arrives, system checks if queue if full.
      - If not full, request added to queue.
      - Otherwise, request dropped.
    - Requests are pulled from the queue and processed at regular intervals.
  - Parameters
    - Bucket size: Queue size.
    - Outflow rate: How many requests to be process in seconds.
  - Pros
    - Memory efficient given the limited queue size.
    - Requests are processed at fixed rate and suitable for cases that stable outflow is needed.
  - Cons:
    - A burst of traffic fills old requests and blocks new requests.
    - Challenging to tune parameters.
#### Fixed window counter algorithm
  - Algorithm
    - Divides timeline into fix-sized time windows and assign a counter for each window.
    - Each request increments the counter by one. Got dropped if the counter reaches predefined threshold.
  - Pros
    - Easy.
    - Memory efficient.
    - Resetting available quota at the end of unit time window fits certain use cases.
  - Cons
    - Spike in traffic at edges of window could cause more requests than allowed quota to go through.
#### Sliding window log algorithm
  - Algorithm
    - Keep track of request timestamps. Usually in cache.
    - When new request in, remove all outdated timestamps.
    - Add timestamp of new request to log.
    - Accept if log size is smaller than threshold. Reject otherwise.
  - Pros:
    - Accurate.
    - Consumes lots of memory, even if a request if rejected, its time still stored in memory.
#### Sliding window counter algorithm
  - Algorithm:
    - Use Request in current window + request in previous window * overlap percentage of rolling window and previous window.
  - Pros
    - Smooths outs spikes in traffic because rate is based on avg rate of prev window.
    - Memory efficient.
  - Cons
    - Approximation.

### High-level architecture

Use cache to store counters. Redis:
  - INCR: Increases the stored counter by 1.
  - EXPIRE: Sets a timeout for the counter with auto delete.
User flow: Client -&gt; Rate Limiter Middleware -&gt; API Server
                      Redis

## Step 3 - Design deep dive

### Rate limiting rules

Rules are generally written in configuration files and saved on disk.

### Exceeding the rate limit

API returns a HTTP response code 429. Might enqueue the rate-limited requests to be processed later (e.g. orders).

#### Rate limiter headers

HTTP headers:
- X-Ratelimit-Remaining: Remaining number of allowed requests in window.
- X-Ratelimit-Limit: Total allowed requests in window.
- X-Ratelimit-Retry-After: Number of seconds to wait until without being throttled. (When dropped. Send with 429 error code).

#### Detailed design

- Rules stored on disk. Workers pull rules from disk and stores in cahce.
- Client send request to rate limiter middleware first.
- Rate limiter middleware loads rules from cache, counters and timestamps from Redis cache.
  - If request is not limited, forward to API servers.
  - If request is limited, return 429 to client. Drop the request or forward to queue.

#### Rate limiter in a distributed environment

- Race condition
  - Lock. Slows down.
  - Lua script. ???
  - [Sorted sets data structure in Redis](https://medium.com/analytics-vidhya/redis-sorted-sets-explained-2d8b6302525).

- Synchronization issue.
  - Use sticky session. (Not recommended)
  - Use centralized data storage. Redis.

- Performance optimization.
  - Multi-data center. Route request to closest edge server to reduce latency.
  - Synchronize data with an eventual consistency model.

#### Monitoring

Gather analytics data to check algorithms effective and rules are effective.

- Requests are dropped unexpectedly?
- Ineffective with sudden increase in traffic?

## Step 4 - Wrap up

- Algorithms
- Architecture
- Hard vs soft limit.
- Rate limit at different level. This chapter talks about application level (layer 7). But can also do other levels, like Iptables (layer 3). [OSI model](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
- Avoids being limited.
  - Client cache.
  - Understand the limit and don't send out too many requests.
  - Include code to catch exceptions and recover.
  - Add sufficient back off time to retry logic.

