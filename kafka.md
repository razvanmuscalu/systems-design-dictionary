# Kafka

A collection of notes around from Kafka [design documentation](https://kafka.apache.org/documentation/#design)

- [General](#general)
- [Producers/Consumers Sync](#producers-consumers-sync)
- [Producer](#producer)
  - [Async vs Sync](#async-vs-sync)
  - [Idempotent Delivery](#idempotent-delivery)
- [Consumer](#consumer)
  - [Push vs Pull](#push-vs-pull)
  - [Delivery Semantics](#delivery-semantics)

# General

- Supports persistence
    - Directly on filesystem
    - All topics (and all partitions within a topic) maintain order

<br />

- Supports replication
    - Master-Follower replication as all reads and writes are via leader
    - Followers are in-sync if heartbeat is ok AND if they’re not too far behind
        - A leader can also have followers which are not in-sync (will catch up eventually)
    - Master keeps list of in-sync followers
        - When leader dies, any of these in-sync followers can become leader
    - Master maintains and decides the order of the messages
        - So followers only need to persist without worrying about order

<br />

- Supports partitioning of topic
    - Brokers assign multiple consumers with same consumer group to different partitions
        - Or consumers can assign themselves to exact partition
    - Producers can send to topic or exact partition

<br />

- Supports splitting of topics into multiple topics
- Supports transformation of events on the fly


# Producers-Consumers Sync

- If producers mark messages as _consumed_ immediately
    - If consumers fail during processing then the message is lost
- Producers can mark messages as _sent_ and wait for ack from consumers to mark as _consumed_
    - If consumers fail to send ack back to producers, then messages will be consumed twice

<br />

- Producers decide level of consistency (how many replicas have to be written to followers)
    - A message is considered committed on the broker when all in-sync followers persisted the message
    - Consumers only see messages that are committed


# Producer

- Push to brokers
    - Brokers replicate the message on multiple partitions

## Async vs Sync

- Send async via batching
    - `batch.size`
    - `linger.ms` (how much to wait before sending whatever is in batch)
- Send sync and wait for broker ack
    - `max.block.ms` (how much to wait for message to be consumed)
- Leader ack, quorum ack
    - Leader vs Quorum ack means producers can choose level of eventual consistency

## Idempotent Delivery

- This achieves exactly-once delivery from producer to broker
- Producers send a sequence ID with every message
    - Producers can retry safely to deliver messages
    - Brokers can deduplicate messages when a producer sends a message that the broker fails to commit
    

# Consumer

- Pull from brokers

## Push vs Pull

- If data is pushed to consumers, then consumers can be overwhelmed
    - So publishers have to implement exponential backoff

<br />

- If consumers pull data, they could pull when there’s nothing new, thus wasting resources
    - Kafka supports long-poll so connections aren’t wasted

## Delivery Semantics

- `at-most-once`: read messages, save position in the log, process messages
    - If processing fails, message is lost as log position has already been saved

<br />

- `at-least-once`: read messages, process messages, save position in the log
    - If saving position fails, processed messages will be re-processed

<br />

- `exactly-once` with transactions
    - In Kafka Streams, when consuming from topic 1 and producing to topic 2
        - Push messages to topic 2 and save position in topic 1 in a transaction
    - When processing messages in a non-Kafka system
        - Save log position and messages in same database and use transactions