# README

WIP

## Migrations

Data migrations are manual and explicit (empty values should be `null`, so omitted/`undefined` values in the data are usually considered suspicious and considered 
an error) currently.
So if you use a persistent database adapter like diskdb, cosmos, redis or mongo, you need to make sure you migrate your data, via:

- just in time migration within the repository, when converting from Persistence Model, to Encoded type, before parsing
- implementing a migration strategy that runs at start up or at specific command request
- manually writin/executing migration queries for the particular database.

locally when the data is considered disposable, you could also increment the `STORAGE_VERSION` constant.
