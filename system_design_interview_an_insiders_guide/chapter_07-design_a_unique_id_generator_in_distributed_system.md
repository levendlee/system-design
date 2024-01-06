# Step 1 - Understand the problem and establish design scope

- Characteristics: Unique and sortable.
- Increment: Increment by time.
- Numerical values.
- Length: 64-bit.
- Scale: 10,000 IDs per second.

# Step 2 - Propose high-level design and get buy-in

## Multi-master replication
Having *k* database servers in use, each increase its next ID by *k* through *auto_increment* feature in database.
Pros
- Scalability. Scale with number of database servers.
Cons
- Hard to scale with multiple data centers. ???
- IDs do not go up with time across multiple servers.
- Doesn't scale well when a server is added or removed.

## UUID
128-bit number. Very low probability of getting collusion. Can be used independently without coordination.
Pros
- Simple. No coordination. No synchronization.
- Easy to scale.
Cons
- IDs are 128-bit long.
- IDs do not go up with time.
- IDs could be non-numeric.

## Ticket server
Use a centralized *auto_increment* feature in a single database server (Ticket Server).
Pros
- Numeric IDs.
- Easy to implement. Works for small to medium scale applications.
Cons
- Single point of failure. Could use multiple servers, but requires sychronization.

## Twitter snowflake approach
- Sign bit: 1 bit. Reserved.
- Timestamp: 41 bits. Milliseconds (1e-3 seconds) since the epoch. 69 years.
- Datacenter ID: 5 bits.
- Machine ID: 5 bits.
- Sequence number: 12 bits. 4096 combinations.

# Step 3 - Design deep dive

(About the twitter snowflake approach)

# Step 4 - Wrap up
- Clock synchronization. Assuming ID generation servers having the same clock is problematic for multiple cores or multiple machines. Network Time Protocol is a popular solution.
- Section length tuning. Fewer sequence numbers but more timestamp bit are effective for low concurrency and long-term applications.
- High availability.
