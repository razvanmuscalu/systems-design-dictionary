# Domain Driven Design

- Same language shared between engineering and business
    - Same terminology, same domain model, etc.


- `Domain` is the smallest unit and consists of
    - `Entity` (e.g. agreement, policy)
    - `Value Object` (e.g. start-date, cycle number)
    - `Services` (e.g. MTA Processor, Cycle Processor)
- `Bounded Contexts` are mappings of multiple domains into a larger area
  https://martinfowler.com/bliki/BoundedContext.html
- `Aggregate` is combining multiple domains into a single unit (e.g. issuance, subscription)
    - An “imported” entity in one bounded context can be an “entire” aggregate in another bounded context
    - For example, a policy is an Entity within the LTM Bounded Context
    - But a policy is an Aggregate (of other entities: changeset, event) in Policy Bounded Context