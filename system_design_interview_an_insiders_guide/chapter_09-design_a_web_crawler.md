# Chapter 9: Design A Web Crawler

- Purposes
    - Search engine indexing
    - Web archiving
    - Web mining
    - Web monitoring

## Step 1 - Understand the problem and establish design scope

- Basic algorithm
    - Given URLs, download all web pages addressed.
    - Extract URLs from these web pages.
    - Add new URLs to the list. Repeat.

- Functionalities
    - Purpose?
    - Volume?
    - Types?
    - How long Storage?
    - Consider newly added pages?
    - Duplicated content?

- Charactertics
    - Scalability
    - Robutness
    - Politness
        -   Not make too many request to a webseite within short time
    - Extensibility

### Back of the envelope estimation

-   1B web pages per month.
-   QPS: 1e9 / (30 * 24 * 2600) ~ 1e9 / 2.6e6 ~ 400 pages per second.
-   Peak QPS: 2 * QPS = 800
-   Avg web page size: 500 kBytes
-   Storage per month: 1e9 * 500e3 = 500e12 = 500 TBytes
-   Storage 5 years: 500e12 * 12 * 5 = 30e15 = 30 PBytes

## Step 2 - Propose high-level design and get buy-in

### Workflow
- 1. Seed URLs
- 2. URL Froniter -> HTML Downloader (DNS Resolver) -> Content Parser
- 3. Content Storage (If not seen)
- 4. Link Extractor -> URL Filter
- 5. URL Storage (If not seen)
- 6. Repeat.

### Components

#### Seed URLs

Utilize to traverse as much as possible. Divide the entire URL space into smaller ones.
    -   Divide based on localities.
    -   Divide based on topics.

#### URL Frontier

A FIFO queue storing URLs to be downloaded.

#### HTML Downloader

#### DNS Resolver
HTML Downloader uses it the resolve IP address.

#### Content Parser
Parse and validate web page to prevent malformed web pages.

#### Content Seen
Eliminate data redundancy and shorten processing time. Compare hash values.

#### Content Storage
Types of storage depends on data type, data size, access frequency, life spand, etc. Popular content on memory, others on disk.

#### URL Extractor
Relative paths are converted to absolute ones.

#### URL Filter
Excludes certrain content types, file extensions, denylists.

#### URL Seen
Avoid adding repeated URLs as it can increase server workload and potential infinite loop. 
Boom filter and hash table can be applied here.

## Step 3 - Design Deep Dive
### DFS vs BFS
DFS is not a good choice as the depth of DFS can be very deep.

BFS is used and implememnted by FIFO queue.
- Impolite: Most links are linking back to the same host. Crawler busy processing URLs from the same host and servers got flooed with requests with parallel processing.
- Priorities: Web is large and prioritize based on ranks, tracffic, update frequency, etc.

### URL Frontier
Important component to ensure *politness, priorization and freshness*.

#### Politeness
Enforcing politness by downloading one page at a time from the same host using a separate FIFO queue for each hostname. Add delay between two tasks.

- Workflow
    -   Queue router -> Mapping Table.
        -   Dispatch URL to different queues based on host.
    -   Queues
        -   Queues with associated hosts.
    -   Queue selector
        -   Dispatch different queues to work threads.
    -   Threads

#### Priority
Prioritize based on usefulness, PageRank, traffic, update frequency, etc.

- Workflow
    -   Prioriter
        -   Computes priority of the URL and dispatch to queues base don it.
    -   Queues
        -   Queues with associated priorities.
    -   Queue selector
        -   Randomly choose a queue with a bias towards queue with higher priority.

-   ***Combined Workflow***
    -   Prioritizer
    -   Front queues (w/ priorities).
    -   Front queue selector.
    -   Back queue router -> Mapping table.
    -   Back queues (w/ hosts).
    -   Back queue selector.

#### Freshness 
Recraw as webs are being updated.
- Recrawl based on web pages' update history.
- Prioritize URLs and recrawl important pages first and more frequently.

### Storage for URL Frontier
Number of URLs in frontier could be hundreds of millions. Memory is neither durable or scalable. Need to use disk.

Majority of URLs are stored on disk. Maintain buffers in memory for enqueue/dequeue operations. Data in buffer periodically written to disk.

### HTML Downloader

#### Robots.txt (Robots Exclusion Protocol)
Check its corresponding `robots.txt` first and follow its rules. Cache and update this file periodically.

#### Performance Optimization
- Distributed crawl
    - URL is partitioned into smaller pieces. Each downloader is responsible for a subset.
- Cache DNS Resolver
    - DNS response time ranges from 10ms to 200ms.
    - Keeps domain name to IP address mapping and is updated periodically by cron jobs.
- Locality
    - Closer to host server, shorter response time.
    - Crawl servers, cache, queue, storage, etc.
- Short timeout
    - Some pages respond slowing or not respond at all.

#### Robutness
- Consistent hashing
- Save crawl states and data.
    - Disrupted crawl can be restarted by loading saved states and data.
- Exception handling
- Data validation
    - Prevent system errors. ???

#### Extensibility
Extendable modules in parallel with link extractor. Image downloader. Web Monitor (copyright and trademark).

#### Detect and avoid problematic content
Redundant, meaningless, or harmful content.
- Redundant: Hashes or checksums.
- Spider traps: Infinite deep directory structure. Set maximal length for URL.
- Data noise: Filter out ads, code snippets, spam URLs, etc.

## Step 4 - Wrap Up
- Server-side rendering: User scripts to generate links on the fly.
- Filter out unwanted pages: Anti-spam.
- Database replication and sharidng.
- Horizontal scaling. (Server-tier)
- Availability, consistency and reliability.
- Analytics
