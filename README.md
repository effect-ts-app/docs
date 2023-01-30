# README

WIP

## Models and Resources

All data modeling is Schema based.

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

### Events

TBD

- Published to UI via Server Sent Events
