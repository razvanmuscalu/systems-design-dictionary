# Database

A collection of notes around databases.

- [Pessimistic Locking](#pessimistic-locking)
- [Optimistic Locking](#optimistic-locking)
- [Connection Pooling](#connection-pooling)
- [Mongo](#mongo)

# Pessimistic Locking

- Standard database transaction locking
- Isolation levels
    - Read Uncommitted
        - Allows dirty reads (reading data before a write is committed)
        - Write is safe (therefore write-only apps can use this)
    - Read Committed
        - Guards against dirty reads
    - Repeatable Read
        - Exclusive lock on record being changed and read locks on referenced records
        - Guards against writes affecting a set of records
    - Serialisable
        - Locks entire table

# Optimistic Locking

- Data has a version next to it
- Read the current version => Prepare the change => Re-read the version before committing (to see it hasn’t changed)
    - Fail if version changed by another write
    - This can lead to many failed transactions so retry mechanism becomes important

# Connection Pooling

- When read/write is done, does not close connection
    - This way it can be re-used
- Nr. of threads close to nr. of cores
    - (2 x core_count) + effective_spindle_count
        - Effective spindle count relates to the fact I/O blocks while disks wait until next time the spinning plate is in right position and data can be accessed again
    - Other I/O blocking operation is network
        - While the app buffers/unbuffers sends and receives
        - This is more negligible
    - Use separate pools for long-running queries/tasks vs short-running queries/tasks
        - To avoid pool starvation

# Mongo

- Indexes
    - Single field
    - Compound (multiple fields)
        - Groups by first field
        - Then index and sorts by second field
    - TTL indexes to auto-remove data after some time
- Supports change streams
    - Clients subscribe to database changes
    - Clients can filter on the stream and read only what they are interested in
        - Reading is via aggregation pipeline so data can also be aggregated on the fly
- Supports replication
- Supports sharding
- Recently supports transactions
- Recently supports aggregation pipelines (basically similar to Java streaming but done inside database)
    - Aggregation works on sharded clusters