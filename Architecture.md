This document provides a very generic outline in architecture's deriving from the `effect-ts-app` boilerplate and it's components.

# Architecture

By the term architecture we refer here to the _abstract structure_ or _design_ of a software designsystem and it's components.

The general idea of the architecture adopted in the `effect-ts-app` boilerplate has several goals, namely:

- **Independence from implementation details through layering**. The architecture is not reliant on specific software to work, in fact it's generic enough that it can be used any kind of application from CLIs, browser or native front ends, backends or any other kind of application. It is also independent of data storage specifics such as databases or file systems.
- **Testability**. By not directly depending on a specific set of software details, tests can focus on verifying business logic and behavior.
- **Reusability and abstraction**. Layering the different domains of responsibility allows for code reuse in different contexts, independently from its specifics.
  
The Architecture is made of several abstract layers of responsibility, often also called Architecture's components.
The following section will attempt to give those components specific definitions in an attempt to lay ground for a common language.

Note that most of these terms derive of have equivalents in _Clean Code_ and _Domain Driven Development_ fields which inspire a large part of the design choices behind the Architecture.

## Business Domain Object

A **business domain object** is an abstraction of data and behaviour of that data specific to a certain domain.

An _ecommerce_ application may deal with data involving users, orders, shipping, dates, accounting that is general enough to not be specific to the actual application leveraging it.
E.g. An ecommerce will often deploy several applications in the same domain such as:
- a customer-facing browser-based application where users can find items and manage their orders
- another customer-facing native application, where users can do the same but in store or internet-downloaded application
- an internal-facing application to help customers help manage their orders
- another internal-facing application that handles items' storage such as in a warehouse
- a partners-facing application where sellers can create listings, run discount or marketing campaigns, etc
- an internal application for translating content
- an application for carriers to manage shippings
- an application for data and marketing teams to analyze user's behaviour to improve the various services
- applications that can be embedded in third-party websites
- a large number of backend services involving full fledged applications or tiny-scoped microservices
- ...and many others

All of those applications will have to interact and work under the same business domain and one of the core ideas of several domain-centric software designs is to separate the business logic related to the domain (common to all these applications) separately from the actual applications consuming it.

This avoids one common pitfall of many enterprise software where business (or `Core`) logic is first shoved in a specific application and then requires to be refactored away or more commonly cloned across different projects with an exponential increase in software maintenance and velocity of adaptation of software to business needs.

## Entity

An `Entity` encapsulates and abstracts a _business domain object_.

In `effect-ts-app` `Entity`s are defined through `Schema`s defined in `types` packages.

Behaviour, the _what_, _how_, and _when_ we can operate on and with this data is defined in `Core` modules in `api` packages. Behaviour encodes the rules of the business.

An important distinction between `Entity`s and other _business domain objects_ is that `Entity`s **alwyas** have a unique identifier which allows to distinguish each of those from any other `Entity`s of the same type.

`Entity`s can contain and reference other `Entity`s or `Value-Object`s.

An example of an `Entity` can be an `Order` entity in an e-commerce.

By defining `Entity`s encapsulations we can define what `Order`s are (their `Schema`) and how, when and what can be done with them (through their `Core` logic). The business rules encoded in `Core` such as the inability to place `Order`s with a delivery date set in the past, are general and independent from the class of applications that may interface with the `Order`s.

## Value-Object

Similar to `Entity`s, `Value-Object`s also abstract a _business domain object_. The core difference between `Entity`s and `Value-Object`s is that `Value-Object`s do not have a unique identifier.
`Value-Object`s too are defined by their data shape and their business logic, and those too are defined through `Schema` and `Core` logic inthe same packages where `Entity`s are defined.

An example of a `Value-Object` might be an email or a geographical `Address`. An `Order` might have a delivery `Address` but this `Address` is only defined by its content. `Value-Object`s are generally attached to `Entity`s and modifications of `Value-Object`s are indistinguishable from replacing the `Value-Object`s as they do not hold any unique identifier and their identity is defined by their content only.

## Schema

A `Schema` is a blueprint for generating:

- encoders and decoders for the `Model`. The `Model` defines the data shape of the _business domain object_
- `Api`s. Utilities to derive other `Schema`s from a base `Model`
- interpreters like `arbitraty` (functions able to generate fake data based on the `Model` definition.)

Should be noted that `Schema`s can encode more than basic data type information such as specific constraints. E.g. a `DateString` can encode not only that the underlying data is a `string` but whether _any_ given `string` can be encoded or decoded to valid `Date` object representation.

## UseCase

`UseCase`s describe the specific application-layer capabilities.

Each capacity is implemented through a single `Controller`.

An example might be that two different applications may both interact with `Order`s but they may not necessarily have the same capabilities. A user-facing application, such as our usual ecommerce example, may give the capabilities to create and review `Order`s and `Item`s.

Another application used in a backoffice might also interact with `Order`s but witha different scope such as handling updates, communication and payments but it may not create or review orders.

Another application used in a warehouse may handle the inventory, packaging and operations with the carrier. But it may neither have the capabilities to create or review orders, nor to handle the payments or communication with the customer.

## Controller

A `Controller` is a piece of application-specific logic. A `Conteoller` defines a single capability of the application such as creating `Order`s. `Controller`s use the logic defined in `Core` and use injected dependencies (through inversion of control) to handle communication with external `Service`s.

## Service

A `Service` provides a dependency needed by a `Controller`.

Examples of `Service` are `Repository`s or database pool connections.

## Repository

A `Repository` is a `Service` that provides APIs for CRUD operations for some `Entity` while hiding the actual implementation and driver-level details.

An example of a `Repository` is a for our `Order` `Entity` would be an `OrderRepository` which would provide the necessary APIs to perform CRUD operations involving `Order` `Entity`s.

## Resource

A `Resource` provides specific `Request/Response` messages that `Controller`s take as input or return as output.

## View

A `View` represents specific types of data representation taht can be derived from `Model`s. They are essentially information included in `Response` messages.

An `Order` may have very different types of such representations such as:

- `CustomerOrderView` that would contain `Order` information specific tgo the `Customer` combined with information about other `Entity`s such as the `Item`s and their quantities.
- `WarehouseOrderView` that would contain `Order` information specific to warehouse workers, such as `Item` location, quantity, dactory, manufacturer and carrier data but may not contain the private information of the customer or the billing.