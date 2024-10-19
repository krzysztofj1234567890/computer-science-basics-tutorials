# Monolith to Microservices 

## Process
Defining service boundaries is an iterative process. 
Because this process is a non-trivial amount of work, you need to __continuously evaluate the cost of decoupling against the benefits__ that you get. 
Following are factors to help you __evaluate how you approach decoupling a monolith__:
- __Avoid refactoring everything all at once.__ To __prioritize__ service decoupling, evaluate cost versus benefits.
- Start with a __Pilot__ Project
- Find the right candidates for refactoring: Services in a __microservice architecture are organized around business concerns__, and __not technical concerns__.
- When you __incrementally__ migrate services, __configure communication between services and monolith to go through well-defined API contracts__.
- __Anticorruption Layer__:  strategy used to ensure that the transition from monolith to microservices does not corrupt the business logic of your system. 
- Implement __API Gateways__ or events
The ACL acts as a barrier between the monolith and the microservices, converting data and requests between the two systems.
- Microservices require much more __automation__: think in advance about __continuous integration (CI), continuous deployment (CD), central logging, and monitoring__.
- Instrument for end-to-end __observability__

### Decouple by domain-driven design

https://cloud.google.com/architecture/microservices-architecture-refactoring-monoliths

Microservices should be designed around business capabilities, not horizontal layers such as data access or messaging.

Microservices are loosely coupled if you can change one service without requiring other services to be updated at the same time.

Domain-driven design (DDD) requires a good understanding of the domain for which the application is written. The necessary domain knowledge to create the application resides within the people who understand it—the domain experts.

You can apply the DDD approach retroactively to an existing application as follows:
- Identify a ubiquitous language—a common vocabulary that is shared between all stakeholders. As a developer, it's important to use terms in your code that a non-technical person can understand. What your code is trying to achieve should be a reflection of your company processes.
- Identify the relevant modules in the monolithic application, and then apply the common vocabulary to those modules.
- Define bounded contexts where you apply explicit boundaries to the identified modules with clearly defined responsibilities. The bounded contexts that you identify are candidates to be refactored into smaller microservices.

### Prioritize services for migration

An ideal starting point to decouple services is to identify the loosely coupled modules in your monolithic application. 
You can choose a loosely coupled module as one of the first candidates to convert to a microservice. 
To complete a dependency analysis of each module, look at the following:
- The type of the dependency: dependencies from data or other modules.
- The scale of the dependency: how a change in the identified module might impact other modules.

Migrating a module with heavy data dependencies is usually a nontrivial task. 
If you migrate features first and migrate the related data later, you might be temporarily reading from and writing data to multiple databases. 
Therefore, you must account for data integrity and synchronization challenges.

### Extract a service from a monolith

After you identify the ideal service candidate, you must identify a way for both microservice and monolithic modules to coexist. 
One way to manage this coexistence is to introduce an inter-process communication (IPC) adapter, which can help the modules work together. 
Over time, the microservice takes on the load and eliminates the monolithic component.

### Manage a monolithic database

Typically, monolithic applications have their own monolithic databases. 
One of the principles of a microservices architecture is to have one database for each microservice. 
Therefore, when you modernize your monolithic application into microservices, you must split the monolithic database based on the service boundaries that you identify.

However, splitting a monolithic database is complex because there might not be clear separation between database objects. 
You also need to consider other issues, such as data synchronization, transactional integrity, joins, and latency. 

#### Share data through an API

When you separate the core functionalities or modules into microservices, you typically use APIs to share and expose data. 
The referenced service exposes data as an API that the calling service needs.

This implementation has obvious performance issues due to additional network and database calls. 
However, sharing data through an API works well when data size is limited.

#### Replicate data

Another way to share data between two separate microservices is to replicate data in the dependent service database. 
The data replication is read-only and can be rebuilt any time. 
This pattern enables the service to be more cohesive. 

#### Static data as configuration

You can inject such static data as a configuration in a microservice.

#### Shared mutable data

Monolithic applications have a common pattern known as shared mutable state. 
In a shared mutable state configuration, multiple modules use a single table.

To migrate a shared mutable state monolith, you can develop a separate ShoppingStatus microservice to manage the ShoppingStatus database table. 
This microservice exposes APIs to manage a customer's shopping status.

#### Distributed transactions

After you isolate the service from the monolith, a local transaction in the original monolithic system might get distributed between multiple services. 
A transaction that spans multiple services is considered a distributed transaction. In the monolithic application, the database system ensures that the transactions are atomic. To handle transactions between various services in a microservice-based system, you need to create a global transaction coordinator. 
The transaction coordinator handles rollback, compensating actions, and other transactions.

#### Data consistency

Consider a multistep transaction in a microservices-based architecture. 
If any one service transaction fails, data must be reconciled by rolling back steps that have succeeded across the other services.

It can be challenging to determine when a step that implements eventual consistency has failed. 

### Design interservice communication

- One-to-one interaction: each client request is processed by exactly one service.
- One-to-many interactions: each request is processed by multiple services.

Consider whether the interaction is synchronous or asynchronous:
- Synchronous: the client expects a timely response from the service and it might block while it waits. HTTP-based REST, gRPC
- Asynchronous: the client doesn't block while waiting for a response. The response, if any, isn't necessarily sent immediately. AMQP or STOMP

Using asynchronous communication provides the following advantages:
- Loose coupling
- Failure isolation
- Responsiveness: An upstream service can reply faster if it doesn't wait on downstream services.
- Flow control: A message queue acts as a buffer, so that receivers can process messages at their own rate.

Challenges to using asynchronous messaging effectively:
- Latency: If the message broker becomes a bottleneck, end-to-end latency might become high.
- Overhead in development and testing
- Complicates error handling

## Patterns

### Strangler Fit pattern

We recommend that you incrementally refactor your monolithic application. 
When you incrementally refactor an application, you gradually build a new application that consists of microservices, and run the application along with your monolithic application. 
This approach is also known as the Strangler Fig pattern. 
Over time, the amount of functionality that is implemented by the monolithic application shrinks until either it disappears entirely or it becomes another microservice.

### Anticorruption Layer

The anticorruption layer (ACL) is a strategy used to ensure that the transition from monolith to microservices does not corrupt the business logic of your system. 
The ACL acts as a barrier between the monolith and the microservices, converting data and requests between the two systems.

## Interservice communication in a microservices setup

After you isolate the service and split it from the monolith, a local transaction in the original monolithic system is distributed among multiple services. 

In the microservice-based system that has a separate database for each service doesn't have a global transaction coordinator that spans the different databases. 
Because transactions aren't centrally coordinated, a failure in processing a payment doesn't roll back changes that were committed in the order service. 
Therefore, the system is in an inconsistent state.

The following patterns are commonly used to handle distributed transactions:
- Two-phase commit protocol (2PC): Part of a family of consensus protocols, 2PC coordinates the commit of a distributed transaction and it maintains atomicity, consistency, isolation, durability (ACID) guarantees.
- Saga: consists of running local transactions within each microservice that make up the distributed transaction. 
An event is triggered at the end of every successful or failed operation. 
All microservices involved in the distributed transaction subscribe to these events. 
If the following microservices receive a success event, they execute their operation. 
If there is a failure, the preceding microservices complete compensating actions to undo changes. 

There are various ways to implement a Saga—for example, you can use task and workflow engines such as Apache Airflow or Apache Camel.

