# Database

Notes from a few Grokking the Systems Design Interview.

- [Tiny URL](#tiny-url)
- [API Rate Limiter](#api-rate-limiter)
- [Instagram/Twitter](#instagram-twitter)

# Tiny URL

### Storage

- Use NoSQL
    - There are not lots of relationships
    - System will be read heavy and needs to be highly available
    - Objects stored are small

<br />

### Logic

- Generate on-demand: Encode URL
    - Problems
        - Can result in key collisions (MD5 produces 21 characters, but we’ll end up choosing a subset)
        - Multiple users can enter same long URL but they'll get the same short URL
    - Solutions
        - Append increasing sequence to long URL and then encode
        - Append user ID to long URL and then encode
          - User would have to be signed in to create short URLs

<br />

- Generate offline: KGS (Key Generation Service)
    - KGS continuously generates unique keys
    - Keep 2 tables: one for unused and one for used keys
    - Application servers can cache sets of short keys
        - When db gives these keys to application, it moves them to used keys table
          - This needs to be transactional and have strong consistency (so that same keys aren't given to multiple application servers) 
        - If application server dies it’s ok as we only lose a few keys but we have billions at our disposal
        - Access to application cache needs to be synchronised so two requests don’t get the same key

<br />

### Partitioning

- range-based (e.g. based on first letter of the long URL)
  - this will have skewed servers (e.g. more URLs start with letter "E" than with letter "Z")
- hash-based (based on the short URL)
  - short URLs are random so there will not be skewed servers


# API Rate Limiter

### Storage

- Use Redis for distributed cache
    - Can persist too if necessary

<br />

### Logic

Below assumes rate limiting per 1 minute

<br />

- Fixed window
    - UserID -> {Count, StartTime}
    - When request comes, increment Count
    - When request comes and 1 minute passed => reset StartTime and Count

<br />

- Sliding window
    - UserID -> SortedSet<Request.Time>
    - When request comes, add it to set
    - When request comes, remove requests older than 1 minute

<br />

- Sliding window with time slicing
    - UserID -> Map<Time, Count>
    - When request comes, increment count of the time window it falls under
    - When request comes, remove map entries where the key (time) is older than 1 minute
      (This uses a lot less storage then sliding window algo)


# Instagram-Twitter

### Storage

- We need to keep relationships: users and pages that a user follows, photos belonging to users, etc.
    - Use SQL (and use many-to-many tables)
    - Or use NoSQL/Cassandra
      - _Photo_: PhotoID -> Object (Location, Timestamp, etc.)
      - _USerPhoto_: each photo ID that a user uploads, is stored in a new column
      - _UserFollow_: each user ID that a user follows, is stored in a new column

<br />

### Scalability

- Separate into read and write servers
- So that writes don't starve connections and affect reads

<br />

### Sharding

Sharding per User ID
- Photo ID is a simple increment in each database shard
  - Because all data about one user is on a single shard
- Not good because of hot users (with lots of followers)
  - As this will create lots of traffic to only a few shards
- Some users might have lots of data that doesn't fit on a single shard

<br />

Sharding Per Photo ID
- Have to pre-generate photo IDs
  - Because we need to know the ID first in order to find the shard
  - Can use KGS (Key Generation Service) to generate the IDs
    - Can have multiple KGS (e.g. one supplies even numbers and one supplies odd numbers) and round-robin on them

Sharding Per Creation Time
- Solves problem of database queries on individual shards being slow
  - As there needs to be a sort by time, but photos already sorted and indexed by time
- Problem is that there would be only a subset of shards getting most of the read and write traffic

Sharding Per Photo ID and Creation Time
- Append creation time to photo ID
  - ID will have two parts: creation time + incrementing sequence from KGS
- Traffic is evenly distributed across all shards
- Each shard is efficient since there's no need for secondary index on creation time

<br />

### News Feed Generation

- On-demand is not good because it would require too much computing per each request (so it'd be slow)
- Solution is to pre-generate feeds continuously for every user
    - First query to get last time the feed was generated (when the linked hash map was updated)
    - Then generate new data from that time onwards and push into the linked hash map
    - Users query the linked hash map by a feed item id and x items from that item will be returned

```
User -> Struct {
            LinkedHashMap<FeedItemID, FeedItem> feedItems;
            DateTime lastGenerated;
        }
```

- Need to decide on how big the in-memory linked hash map has to be.
- Since the linked hash map has fixed size for every user, we CAN shard this structure by User ID.

### Sending to Clients

- Pull: not good for users with few followers as lots of pulls will be when there’s nothing new
- Push: not good for users with many followers as they’d receive too many pushes
- Hybrid: combine pull & push

<br />

- This is very similar to designing Twitter
    - Just replace photos with tweets