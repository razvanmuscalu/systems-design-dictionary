# Database

Notes from a few Grokking the Systems Design Interview.

- [Tiny URL](#tiny-url)
- [API Rate Limiter](#api-rate-limiter)
- [Instagram/Twitter](#instagram-twitter)

# Tiny URL

- Use Mongo
    - There are not lots of relationships
    - System will be read heavy and needs to be highly available


- Generate on-demand: Encode URL
    - Problems
        - Can result in key collisions (MD5 produces 21 characters, but we’ll end up choosing first 6)
        - Multiple users can enter same URL => resulting in a single short URL
    - Solutions
        - Append increasing sequence to long URL and then encode
        - Append user ID to long URL and then encode


- Generate offline: KGS (Key Generation Service)
    - KGS continuously generates unique keys
    - Keep 2 tables: one for unused and one for used keys
    - Application servers can cache sets of short keys
        - When db gives these keys to application, it moves them to used keys table
        - If application server dies it’s ok as we only lose a few keys but we have billions at our disposal
        - Access to cache needs to be synchronised so two requests don’t get the same key



# API Rate Limiter

- Use Redis for distributed cache
    - Can persist too if necessary


Below assumes rate limiting per 1 minute


- Fixed window
    - UserID -> {Count, StartTime}
    - When request comes, increment Count
    - When request comes and 1 minute passed => reset StartTime and Count


- Sliding window
    - UserID -> SortedSet<Request.Time>
    - When request comes, add it to set
    - When request comes, remove requests older than 1 minute


- Sliding window with time slicing
    - UserID -> Map<Time, Count>
    - When request comes, increment count of the time window it falls under
    - When request comes, remove map entries where the key (time) is older than 1 minute
      (This uses a lot less storage then sliding window algo)



# Instagram-Twitter

- We need to keep relationships: users that a user follows, pages that a user follows, etc.
    - Use SQL (and use many-to-many tables)
    - Or use Cassandra (and each user that a user follows is stored in a new column)


- Sharding
    - Per User ID
        - Photo ID is a simple increment
        - Not good because of hot users
    - Per Photo ID
        - Have to pre-generate photo IDs
            - Because we need to know the ID first to find the shard
            - Can use KGS (Key Generation Service) to generate the IDs
        - Append epoch to key so that news generation queries can be more efficient


- News Feed Generation
    - On-demand is not good because it would require too much computing per each request
    - Solution is to pre-generate feeds continuously for every user
    - Sending to clients
        - Pull: not good for users with few followers as lots of pulls will be when there’s nothing new
        - Push: not good for users with many followers as they’d receive too many pushes
        - Hybrid: combine pull & push


- This is very similar to designing Twitter
    - Just replace photos with tweets