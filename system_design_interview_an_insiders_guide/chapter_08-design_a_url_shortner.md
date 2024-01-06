# Step 1 - Understand the problem and establish design scope
- Functionality. Volume. Length. Characters. URL be deleted or updated.
## Back of the envelop estimation
- Write operation: 100 million URLs are generated per day. Per second: 100 million / 24 / 3600 = 1160.
- Read operation: Assuming read to write ratio is 10:1. Per second: 1160 * 10 - 11,600
- Assuming URL shortener service will run for 10 years, it means support 100 million * 365 * 10 = 365 billion records, 365TB storage assuming average URL length is 100.

# Step 2 - Propose high-level design and get buy-in
## API Endpoints
[RESTful API](https://aws.amazon.com/what-is/restful-api/#:~:text=RESTful%20API%20is%20an%20interface,applications%20to%20perform%20various%20tasks.)

- URL shortening. Sends a POST request.
```text
POST api/v1/data/shorten
- request parameter: {longUrl: longURLString}
- return shortURL
```

- URL redirecting. Sends a GET request.
```text
GET api/v1/ShortURL
- return longURL for HTTP redirction.
```

## URL redirecting
Server receives a tinyURL request, it changes the short URL to the long URL with 301/302 redirect. Use hash tables to implement storage.

- **301 redirect**: Permanent and cached in browser.
  - Use to reduce the server load.
- **302 redirect**: Temporary and always sent to server first.
  - Use to get analytics.

## URL shortening
Find a function that maps a long URL to the hash value. Hash function must (1-1 mapping):
- Each *longURL* must be hashed to one *hashValue*.
- Each *hashValue* can be mapped back to the *longURL*.

# Step 3 - Design deep dive
## Data model
Store short2long mapping in a relational database with in memory cache.

## Hash function
### Hash value length.
If use alphanumeric chars, then have 10 + 26 + 26 = 62 chars to use. Need to figure out 62^n &gt;= 365 billion, which indicates n = 7 (3.5 trillion).

### Hash + collusion resolution
Use well known hash functions (CRC32/MD5/SHA-1) to hash as 7 char strings. However the hash value is too long.
- Collect first 7 characters of a hash value. Recursively append a new predefined string and rehash until no more collision is discovered. 
- Problem: Expensive to query database to detect collision.
- Solution: Can use [bloom filters](https://brilliant.org/wiki/bloom-filter/#:~:text=A%20bloom%20filter%20is%20a,is%20added%20to%20the%20set.) to improve performance. In memory testing before hitting disk based hash table.

### Base 62 conversion
Converts unique ID into a URL using base 62.

### Comparison
| Hash + collision resolution | Base 62 conversion |
| --- | --- |
| Fixed short URL length. | Unfixed short URL length. Goes up with ID. |
| No need a unique ID generator. | Need a unique ID generator. |
| Collision is possible and need to resolve. | Collusion is impossible. |
| Possible to figure out next available URL. | Easy to figure out next available URL. Security risk. |

### URL shortening deep dive
- Input longURL.
- Check longURL in database?
  - Yes. Return shortURL.
  - No. 
    - Generate a new ID.
    - Convert ID to shortURL.
    - Save ID, shortURL, longURL in database.

### URL redirecting deep dive
- User requests shortURL.
- Load balancer forwards to web servers.
- If shortURL in cache, then return longURL.
- If not in cache, fetch the longYRL from the database, cache, return.

# Step 4 - Wrap Up
- Rate Limiter. Filter based on IP address to avoid malicious users.
- Web server scaling. Stateless. Easy to scale.
- Database scaling. Replica and sharding.
- Analytics: How many users clicking? When do they click? etc.
- Availability, consistency and reliability. 
