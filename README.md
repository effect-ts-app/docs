# README

WIP

## Models and Resources

All data modeling is (Schema)[https://github.com/effect-ts-app/libs/tree/main/packages/schema] based.

## Data type architecture/hierarchy

Sketch..

![alt text](doc/img/data-arch.png)

## Concepts

- Applications
  - API - private business logic, endpoints with resources for UI
  - UI - views and public business logic, uses derived clients to talk to endpoints on the API
- Messages - talk between API/Worker services
- Resources - talk between UI and API - think REST resources, actions, view models.
- Models - domain core types, many others are derived from these

### Queries (read) & Commands/Mutations (write)

TBD

## Messaging Queues

TBD

## UI Events

TBD

- Published to UI via Server Sent Events

## Database

The default storage adapters and setup is for document databases.
There is the Persistence Model (how does the data look like within the data store), Encoded type (often the same as persistence model) and Model type (fully validated and strong domain types).

### Adapters

In the box are adapters for

- memory (`memdb://`): good for local development, tests, single instance, no need for migrations
- disk (`diskdb://.data`): good for local development, tests, single instance, migrations optional
- redis (`redis://`...): good for dev instances, single instance, migrations optional
- CosmosDB (`AccountEndpoint=`): production ready, multi instance support
- Mongo (`mongo://`...): production ready, multi instance support

### Transactions

While basic transaction support for ACID operations is possible, it is recommended to design mutations in non corruptive ways.

### Migrations

Data migrations are manual and explicit.
Empty values should be `null`, so omitted/`undefined` values in the data are usually considered suspicious and an error.
So if you use a persistent database adapter like diskdb, cosmos, redis or mongo, you need to make sure you migrate your data, via:

- just in time migration within the repository, when converting from Persistence Model, to Encoded type, before parsing
- implementing a migration strategy that runs at start up or at specific command request
- manually writin/executing migration queries for the particular database.

locally when the data is considered disposable, you could also increment the `STORAGE_VERSION` constant,
which will ignore previous data, and store under a new namespace.

### Concurrency

By default, optimistic concurrency is used via an `_etag` field. If the `_etag` on a record changed between reading the record, and updating the record,
an `OptimisticConcurrencyException` is thrown. You will need to handle it as appropriate for your case.

### Domain & Integration Events

During saving operations you can include events (domain or integration events), each repository decides what events to support and what to do with them.
Often an Event needs to be published on a Queue or MessageBus, after the data has been successfuly saved.
To make this even more durable, you can save the events within the same database transaction, so that if reaching a queue fails after saving or the process is terminated, you can reconsile this by replaying unprocessed events from the database.

The [`Pure`](https://github.com/effect-ts-app/libs/blob/main/packages/prelude/_src/Pure.ts) type helps you to build logic that builds or updates state,
and logs events to be picked up by the save and publish. It composes nicely.
Which is inspired by [ZPure](https://zio.github.io/zio-prelude/docs/zpure/)
