# Applied Design

A collection of applied patterns at various work projects.

- [2-Phase Commit](#2-phase-commit)
- [SAGA Pattern](#saga-pattern)
  - [Orchestrator](#orchestrator)
- [Sync Async](#sync-async)
- [Migration](#migration)
- [Queueing](#queueing)

# 2-Phase Commit

- policy service
    - `create_changeset`
        - Builds projection to validate it
        - Applies locks
    - `apply_changeset` => re-builds projection and applies it
    - `discard_changeset` => discards changeset
    

# Saga Pattern

## Orchestrator

- change-request service => `confirm_change`
    - Applies changes to data services, in order one by one
        - Rollbacks if necessary
    - Applies lock

# Sync-Async

- LTM service => `process_cycle`
    - Build discard object with more and more changes that have been applied
    - Discard all accumulated changes if any change fails to be applied


- Queue pre-emptively
    - And queue rollback if local change fails


- After local change is applied, queue change in second system


# Migration

- @babylonhealth
- consultations-migrator pulling changes from db A to db B


# Queueing

- @babylonhealth => sending to Kafka
- Commit updates to local database
    - Commit the events together with the domain data in a single transaction
- Batch job to get updates and queue them to Kafka
