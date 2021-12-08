# Design Principles

- [DDD](#ddd)
- [SOLID](#solid)
- [REST HATEOAS HAL](#rest)

# DDD

- Same language shared between engineering and business
    - Same terminology, same domain model, etc.

<br />

- `Domain` is the smallest unit and consists of
    - `Entity` (e.g. agreement, policy)
    - `Value Object` (e.g. start-date, cycle number)
    - `Services` (e.g. MTA Processor, Cycle Processor)
- `Bounded Contexts` are mappings of multiple domains into a larger area
  https://martinfowler.com/bliki/BoundedContext.html
- `Aggregate` is combining multiple entities into a single unit (e.g. issuance, subscription)
    - An “imported” `Entity` in one `Bounded Context` can be an “entire” `Aggregate` in another `Bounded Context`
    - For example, a policy is an `Entity` within the LTM `Bounded Context`
    - But a policy is an `Aggregate` (of other entities: changeset, event) in Policy `Bounded Context`

# SOLID

- `Single Responsibility` - every class has one responsibility 
- `Open/Closed` - open for extension, closed for modification
- `Liskov Substitution` - users of base classes must be able to use derived classes without knowing it
- `Interface Segregation` - favour many client-specific interfaces over one general-purpose interface
- `Dependency Inversion` - depend on abstractions (interfaces) not concretes (implementations)

# REST

- Representational State Transfer
- Architectural style forcing clients to consume a server-driven data model

- REST levels
  - Level 1: identify resources (same URI for all CRUD operations)
  - Level 2: use HTTP verbs for different CRUD operations (e.g. GET/POST/PUT/DELETE)
  - Level 3: use HTTP status codes (e.g. 200, 201, 401, 404, 500, 504, etc.)
  - Level 4: `HATEOAS`
    - Hypermedia As The Engine Of Application State
    - Server sends data AND LINKS to clients
    - So clients only know entrypoint and do NOT hardcode resource URIs
    - `HAL` (Hypertext Application Language) is a draft RPC for providing a uniform way of achieving HATEOAS

```json
{
  "links": {
    "self": { "href":  "https://api.com/items" },
    "item": [
      { "href":  "https://api.com/items/1" },
      { "href":  "https://api.com/items/2" }
    ]
  },
  "data": [
    { "itemName": "a" },
    { "itemName": "b" }
  ]
}
```