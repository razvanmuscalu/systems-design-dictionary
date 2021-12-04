# Messaging

A collection of notes around messaging.

- [Event Sourcing](#event-sourcing)
- [CQRS](#cqrs)
- [Event Bus](#event-bus)
- [Queue](#queue)

# Event Sourcing


- e.g. a database has a change stream with all the changes ever made
  - Equivalent to a change stream of all write operations ever made

<br />

- System does not store domain model
- System instead stores the changes to that model as a series of events
- Models for reading are built from events and stored as domain models in a separate database

<br />

Benefit: rich clients
- Clients consume raw data instead of transformed data (which loses information in the process)
- And therefore can build richer transformations themselves


#CQRS


- Traditionally, domains mapped to a single CRUD flow
  - Same domain model used for reading and writing

<br />

- CQRS is about separating read models from write models
  https://www.eventstore.com/cqrs-pattern
  https://martinfowler.com/bliki/CQRS.html
  - This is done as application complexity increases
  - Separation is at application/flow level (under-the-hood can separate databases into read and write databases but it’s not a pre-requisite)

<br />

- CQRS works well with Event Sourcing (https://microservices.io/patterns/data/cqrs.html)
  - Multiple sources publish their write-events on individual streams
  - One consumer reads the events and builds a read-model


# Event Bus


- Systems publish domain events
  - These are not raw events (like the write operations in a database change stream)
  - Instead they are transformed domain objects
- All systems publish on a single bus
- All systems consume from a single bus
  - Each consumes the events they’re interested in

<br />

Problems
- This breaks away from clear boundaries between domains/services
- Producers can break consumers by pushing data they cannot handle
- It’s hard to achieve immutability of events once published
  - The correct way is to publish new event types which refer back to existing data on previous events
  - But this means there will be lots of event types and lots of events
  - And this makes consuming quite difficult
- Opacity
  - Systems will do some actions in an event-driven way
  - And some other actions in standard ways (HTTP calls)
  - Together, these two means it will become increasingly difficult to reason about the flow of data within the system
- Frameworks to connect (publish/consume) tend to be heavy
- Hard to deal with events that are deprecated but have already been written
  - Either compact the log and remove them (not easy with many consumers)
  - Or keep them around


### Event Sourcing & Event Bus


- There can be a common trap to mix these two
  - Many systems doing their own event sourcing (publishing) on a common stream (event bus)


# Queue


- It CAN be similar to an event bus
- But it’s usually a 1-to-1 mapping
  - One publisher pushes messages consumed by a single consumer
  - So there are many queues for all the 1-to-1 communications in a system
