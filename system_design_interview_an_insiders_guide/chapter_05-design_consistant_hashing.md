# The rehashing problem

Static hashing causes massive cache misses when adding/removing servers and rehasing.

# Consistent hasing

Consistent hashing is a special kind of hashing such that when a hash table is resized, only k/n keys need to be remapped on average, where k is the number of keys and n is the number of slots.

## Terminologies and Concepts

- Hash space: The range of possible hash values.
- Hash ring: Connecting the smallest and largest hash values as a ring.
- Hash servers: Map servers based on IP/name onto the hash ring.
- Hash keys: No modular function. Map onto the hash ring as well.

## Basic approach
- Steps
  - Server lookup: Go clockwise from the key position on the ring until a server is found.
  - Add a server: Only require redistribution a fraction of keys, moving from the next old server to the new aded server.
  - Remove a server: Only require redistribution a fraction of keys, moving from the new removed server to the next old server.
- Overall
  - Map servers ad keys on the ring using a uniformly distributed hash function.
  - To find out which server a key is mapped to, go clockwise from key position until first server is found,
- Problems
  - Impossible to keep same size of partitions on the ring with additions and removals.
  - Possible to have a non-uniform key distribution on the ring.

## Virtual nodes
- Concept
  - Represents each server with multiple virtual nodes on the ring. Lookup virtual nodes and then find server.
  - Pros: More virtual nodes -&gt; more balanced distribution of keys as standard deviation is smaller.
  - Cons: More virtual nodes -&gt; more memory consumption.

## Find affected keys
- Add a node
  - Find the previous node anti-clockwise. All keys from previous node to current node need to be moved to new server.
- Remove a node
  - Find the previous node anti-clockwise. All keys from previous node to current node need to be moved to the next server (clockwise).

# Wrap up

Benefits
- Minimized keys redistribution when servers added/removed.
- Easy to scale horizontally because data are evenly distributed.
- Mitigate hotspot key problem by redistributing the data more evenly.

