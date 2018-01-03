# Consumer Driven Contracts implemented using Pacts

Note: For demo see [Pact Demo Installation and Testing Guide](#pact-demo-installation-and-testing-guide)

<img src="https://github.com/steam0/pact-guide/blob/master/images/cdc_broker_provider_consumer.png?raw=true" width="60%">

To increase the velocity and reduce the cost of microservices development, it is key to be able to build and deploy new versions with confidence that you don't break any dependencies. Microservices are easy to build and run, but they quickly become a tangled web of dependencies that slow down development and result in broken dependencies. Microservice architectures can vary from only a handful of services to more than a thousand services in a distributed system. Organizations that transition from a traditional monolithic design to a microservice architecture, will soon realize that it is hard to keep track of all dependencies.  

Consumer Driven Contracts is a testing paradigm that let API-consumers communicate to the API-providers how they are using their services. This article discusses software testing, how Consumer Driven Contracts can make developers more confident, how and when to use Consumer Driven Contracts, and how to benefit from combining Consumer Driven Contracts and API-versioning.

## Tests

Testing software code is an important part of development. Testing gives you confidence that the application works as intended, but it is time consuming both to write and run tests. Different types of tests have different strengths and weaknesses and there is a big difference in the _cost_ (time spent writing and running tests) of the various types of tests.

<img src="https://github.com/steam0/pact-guide/blob/master/images/testing_pyramid.png?raw=true" width="60%">

### Unit tests

Unit tests verify the internal functionality of an application. These tests are used to validate the functionality of a class or method, and to make sure that internal functionality does not unintentionally change.

The tests do not require any external applications to be running, and they are relatively cheap to write and run. Unit tests can only detect errors in internal logic.

<img src="https://github.com/steam0/pact-guide/blob/master/images/unit_test_graph.png?raw=true" width="60%">

### Integration tests

Integration tests verify integrations between applications. They typically call endpoints provided by an external application and verify the response. Integration tests rely on an external service and are more expensive than unit tests. Integration tests require that all external services are online for the tests to pass. If an external service is not online, the test can fail without good feedback about why it failed.

Integration tests can also be created without an external call and with mocked responses that you can run your tests with. Using mocks reduces the cost of running integration tests, but it also removes some of the confidence that the integration test is supposed to gice.

<img src="https://github.com/steam0/pact-guide/blob/master/images/integration_test_graph.png?raw=true" width="60%">

###  End-to-end tests

End-to-end tests verify that the entire system meets its requirement. The tests are executed on a system of running applications and verify that actions are executed properly and that responses are valid.

System tests are very costly to write, implement and run, and are often neglected or ignored. Of all test types, end-to-end testing gives the most confidence, but it is not common for developers to perform end-to-end testing during development.

<img src="https://github.com/steam0/pact-guide/blob/master/images/system_test_graph.png?raw=true" width="60%">

## Anatomy of a microservice

<img src="https://github.com/steam0/pact-guide/blob/master/images/microservice.png?raw=true" width="60%">

This is a simplified illustration of a microservice. A microservice's responsibility is limited by a logical boundary, often called the _bounded context_. Each microservice have this single responsibility principle and systems are made by combining multiple services. Each part of the microservice should be tested with multiple testing strategies. The interface and domain parts are usually tested with unit tests, while the persistence and integrations parts are tested with integration tests. This is a very simplified explanation and there are many exceptions. End-to-end testing is used to test multiple services together, and manual testing is used when you need to test very specific parts of a system.

## Consumer Driven Contracts

<img src="https://github.com/steam0/pact-guide/blob/master/images/microservices.png?raw=true" width="70%">

When an application is consuming an external service, the application becomes a _consumer_ of that service. The external service becomes a _provider_ of services to this consumer. The consumer calls different endpoints on the external service and writes integration tests based on the response. 

Developers often avoid or forget to implement good integration or end-to-end tests because they are so much more expensive to write and run than unit tests. You can add mocked integration tests, but that exposes other problems. If a service changes its interface, the mocked integration test on the consumer side will fail to detect the changes and the consumer application will crash even if the builds are all green. Consumer Driven Contracts provides a solution to this problem.

Consumer Driven Contracts is a testing paradigm that allows consumers of a service to define a consumer contract that the service can validate against. These tests are an alternative to the traditional (mocked) integration tests, but are executed on both the consumer and the service provider application. This makes the provider application aware of how its consumers are using the APIs and make breaking changes visible to the developers before the changes are deployed.

<img src="https://github.com/steam0/pact-guide/blob/master/images/testing_pyramid_cdc.png?raw=true" width="60%">

## APIs are contracts

> An API is a contract from the provider to any consumer and describes how to use services from the provider. A pact is a contract from a consumer to the provider and describes how the consumer uses services from the provider.

An API is a contract. The provider guarantees that if you use the API as described, it will provide responses according to the API-specification. [_The OpenAPI Specification_](https://github.com/OAI/OpenAPI-Specification) is a framework for an API provider to expose APIs as a contract using JSON or YAML. Even though such frameworks help developers share information about request data and response data, it does not explain the intended usage of an endpoint and it is up to developers to document what the endpoint really does. This makes it possible for consumers to misunderstand the intended capability of an API. 

A consumer usually writes integration tests to confirm that it has interpreted the API contract correctly. The integration tests describe how the consumer uses the provided API and may show which assumptions the consumer made about the API. This does not provide any guidelines to make sure that the provider doesn't break the consumer integration when the provider application is changed.

_Consumer Driven Contracts_ let consumers write mock integration tests and create a _consumer contract_ that defines how the consumer is using the API. The integration tests are then submitted to the provider to run whenever the provider code is changed. This consumer-provider contract creates confidence for the consumer that their services will keep working even when the providers are updated.

> The sum of all consumer contract tests defines the overall service contract

When consumers connect to an external interface, they form a contract between themselves and the interface. With Consumer Driven Contracts, the consumer creates a contract and exposes this contract to the provider. As more and more services connect to the external service, they together create a set of contracts that in combination defines the overall service contract. By using frameworks for sharing consumer contracts, providers can access the contracts and verify that consumers won't be impacted when the provider service is changed.

## Pact

Pact is a framework for implementing Consumer Driven Contracts in an application. The consumer can write a consumer contract to the provider by creating a _pact_. A pact is an integration test, written by the consumer, that the provider can add to their test suite and run before building (or deploying) a new version of the application. A pact file contains information about which endpoint is called, the request data, the response data, and whatever validation of the response data that the consumer chose to perform.

## Publishing pacts

When a consumer has generated a pact file, it should publish it to the providers. You can attach the file as a part of the repository (do not do this), expose the file at a url (should not do this) or publish it to a shared _broker_. 

A pact broker is a separate application that should be a part of any system implementing pacts. The pact broker waits for pact files from any consumer and indexes the pact files based on which provider they are communicating with. When a pact is published to the broker, it becomes available to any provider that want to validate them.

<img src="https://github.com/steam0/pact-guide/blob/master/images/cdc_publishing_contracts.png?raw=true" width="60%">

## Pact-dashboard

The pact broker provides a simple web application with information about all pacts registered. The broker displays who the consumer is, which provider the pact is for and whether the pact has been verified by the provider application.

<img src="https://github.com/steam0/pact-guide/blob/master/images/pact-example.png?raw=true" width="70%">

The dashboard can also display dependency graphs based on all pacts. This gives an automatically updated view of all dependencies - very helpful when you update existing services or develop new services.

<img src="https://github.com/steam0/pact-guide/blob/master/images/simple-dependency-graph.png?raw=true" width="70%">

## Why you should apply Consumer Driven Contracts to your system

> If you don't know who your consumers are, you will not be able to predict how your API is being used.

When you develop applications, and specifically applications in a microservice architecture, you write tests to ensure system stability and to provide a solid foundation for future development. In a microservice architecture it is not always clear which applications are using your service, and unit tests and integration tests are less useful for the API-provider. If you don't know who your consumers are, you cannot predict how your API is being used. Even if you do know who your consumers are, you still can't always predict how they're using your API.

Consumer Driven Contracts solve this problem by providing a design pattern for consumers to apply while writing tests. Since all applications *should* have both unit tests and mocked integration tests, there is no significant code change needed on the consumer side to create a contract for the provider to validate.

Consumer Driven Contracts give your developers the confidence to keep deploying new versions of their applications knowing that they will not break functionality for any consumers.

> organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations.
> -- Melvin Conway

This famous quote from Melvin Conway is often used to explain why computer systems end up like they do. The qoute is even more useful to explain built on microservice architehtures since dependencies are not as obvious as they are in a monolith. Much like internal departments in an organization, microservices have separate responsibilities and concerns. When different departments or teams develop services, they often model their APIs the same way as they communicate with other departments. Implementing Consumer Driven Contracts to a computer system provides a platform for developers, teams and departments to communicate better. All parties provide contracts to each other. This helps the consumer discover information about how to use an API and the provider can see how the consumer uses their API and which assumptions the consumer made. It is as true in software development as it is in business: Communication is the key to success.


## Objections to Consumer Driven Contracts

> Consumer Driven Contracts (CDC) moves the responsibility of compatibility from the consumer to the API provider

One of the main objections to implementing the CDC paradigm, is that it is the API provider that decides what their contract to the world is, and it is up to the consumer of an API to comply with this contract. This is not necessarily true. On one hand you want to be in control of your own API, but you also want to attract users to your API. It is important to keep an open mind and assess your consumers. The Consumer Driven Contracts paradigm provides a platform to form a symbiotic relationship between the API provider and the consumer much like the symbiotic relationship between a vendor (API provider) and a customer (API consumer).

> Consumer Driven Contracts prevent developers from making changes as tests will break

This is a fair objection, but it is not entirely true. Consumer Driven Contracts help developers by making them aware of breaking changes in their code. A Consumer Driven Contracts framework like Pact clarifies who uses your service and how they are using it. If a change breaks a consumer, it is an excellent opportunity for developers to _talk_ with each other. Communication is the key to success and Consumer Driven Contracts help consumers and providers communicate.

If a Consumer Driven Contracts test breaks, developers from the provider should investigate if the breaking change is intended and notify the consumer about it. If the breaking change is intended, both teams of developers will need to cooperate with each other to resolve the issue. If the teams cannot agree on who should fix the issue, both teams need to act more maturely. Symbiotic relationships require that both sides provide what the other side needs. A CDC test should not prevent further development of an application and the consumer contract should be considered a guideline for how the provider can ensure service reliability. Unless there are other legal contracts involved, the provider should deploy the changes after informing all consumers about the change. Temporary API-versioning of the failing tests can also be considered in some cases.

## Internal vs. external usage

Consumer Driven Contracts can solve different issues that may occur while microservices are being developed in a large organization, but it is not always clear *when* to use this testing pattern or *who* you should expose it to. 

> Consumer Driven Contracts should be used internally in a development team

A development team normally consist of 3-9 people and a can responsible for hundreds of microservices within an organization. Within a team, all development should use Consumer Driven Contracts when the team creates dependent services. Consumer Driven Contracts makes it easier for developers to know which of their supported services depend on other services and helps new team members discover and understand all system dependencies.

> Consumer Driven Contracts should very often be used internally in an organization

Organizations come in different shapes and sizes, and there is no definite way to decide if an organization should use Consumer Driven Contracts or not. Ultimately, it depends on the size of the organization and the number of services that the organization supports. In most cases the decision will be "yes". Larger organizations that produce microservices, usually have multiple development teams that consume services created by other internal teams. These organizations should apply Consumer Driven Contracts. Not only will this create a great platform for teams to communicate with each other, but it will also improve the stability entire systems.

> Consumer Driven Contracts can be used for external consumers

Some services have external consumers that are known to the provider. In these cases, Consumer Driven Contracts can be exposed from the providing organization to their consumers. By letting their known customers show how they are using the API, the providing organization can make better decisions about how to improve the service without breaking customer systems. Consumer Driven Contracts from an external consumer should always be considered as guidelines only. The external consumer should always be notified if one of their contracts will be broken when a new version is deployed. 

> Consumer Driven Contracts should not be used for open APIs

Any organization that provides open APIs should not let unknown consumers provide contracts because of the redundant dependencies they will create. If open APIs were to let all their consumers provide contracts, they should only use them as guidelines. For open APIs the data will often not be useful because of the lack of implementations from consumers. It does not help a development team if only one consumer out of a hundred choose to provide a contract and will not give the developers more confidence when they update to their code.

<img src="https://github.com/steam0/pact-guide/blob/master/images/cdc_usage.png?raw=true" width="100%">

There is a big difference between internal consumers and external consumers when it comes to how strictly a provider should comply with the test results. When Consumer Driven Contracts tests from an internal consumer fails, it should be considered as breaching a binding contract. When contracts from an external consumer fails, it should only be considered as information discovery that report what APIs break and why. The guideline is there to discover faults, not to prohibit development.

## API versioning

API versioning is an important tool to decouple microservices. There are many ways of implementing API versioning, but that's another topic. API versioning frees an application API from all its existing contracts by defining a new version of that contract. With API versioning, you will then have to support and maintain `n+1` versions of your API. For that reason, you should use API versioning with care.

> API versioning is great in theory but can be very hard to implement. If you deploy microservices several times each day, you risk having to create new versions of the API daily.

API versioning is not the solution to achieve decoupled releases of microservices. Not because it is impossible to implement, but because it ultimately results in more code to maintain which again inhibits development speed. Multiple versions of an API make the codebase unnecessarily large and more likely to contain errors.

> When do I need to create a new API version? 

API versioning makes it possible to support consumers who depend on older versions of an API, but it does not provide the desired confidence in maintaining the API. If you correct a mistake in an older API version, you may break the consumers of that version. This is an example where using Consumer Driven Contracts will make developers more confident when making changes to the code. By validating all consumers of an API version, developers can maintain older versions of APIs with confidence.

## Consumer Driven API Design

You can also use Consumer Driven Contracts to drive API design. When a service requests a new API from a provider, misunderstandings will happen. If you let consumers write consumer contracts specify requirements for the new API before you start developing the new endpoint, it is much easier for developers of the providing service to know what the consuming service needs and expects.

## Summary

Consumer Driven Contracts is all about giving developers the confidence they need when they create new versions of an application. Developers can confirm all changes in an API by validating consumer contracts and be assured that their code does not brake consumers. It seems like a good idea to combine API-versioning and Consumer Driven Contracts since the versioning decouples the application from its consumers, and Consumer Driven Contracts verifies when it is necessary to create a new API-version.

Pact is a framework for writing Consumer Driven Contracts where the consumer writes mocked integration tests and uploads them to a pact broker. API providers can download pacts from this broker and validate them in the build process. Pact supports different programming languages and is very easy to start to use.

Consumer Driven Contracts can also be used to drive API-design by creating consumer contracts before the API. This allows consumers to provide accurate feedback to the provider to make it easier for providers to create the API.

# Pact Demo Installation and Testing Guide

This is a set of example implementations of Pact. First, we will set up a Pact broker on the local machine. Then we will set up a consumer, and finally a provider that will use the contract (Pact) created by the consumer and verify that the API is within the consumer contract.

## Install and configure a pact broker database
1. Start postgres db

```
docker run --name pact-db -p 6543:5432 -e POSTGRES_USER=pact -e POSTGRES_PASSWORD=password -d postgres
```

2. Connect to db

```
docker run -it --link pact-db:postgres --rm postgres sh -c 'exec psql -h "$POSTGRES_PORT_5432_TCP_ADDR" -p "$POSTGRES_PORT_5432_TCP_PORT" -U pact'
```

3. Run script

```
CREATE USER pactuser WITH PASSWORD 'password';
CREATE DATABASE pactbroker WITH OWNER pactuser;
GRANT ALL PRIVILEGES ON DATABASE pactbroker TO pactuser;
```

## Run Pact Broker

```
docker run --name pactbroker --link pact-db:postgres -e PACT_BROKER_DATABASE_USERNAME=pactuser -e PACT_BROKER_DATABASE_PASSWORD=password -e PACT_BROKER_DATABASE_HOST=postgres -e PACT_BROKER_DATABASE_NAME=pactbroker -d -p 8080:80 dius/pact-broker
```


## Generate consumer pact file

1. Get a consumer application

Get a copy of the consumer repo: https://github.com/steam0/pact-consumer

```
git clone git@github.com:steam0/pact-consumer.git
```

2. Run tests and push pact file to broker

```
./gradlew clean build
```

## Run pact tests on the provider

1. Get a producer application

Get a copy of the provider application: https://github.com/steam0/pact-provider

```
git clone git@github.com:steam0/pact-provider.git
```

2. Run tests

```
./gradlew clean build
```


# Notes for future development of this guide
- Introduction discussing what the trouble with microservices is. Why is it hard to keep your speed up as an organization and system grow? How can you release with confidence.
- Releasing new versions with confidence
- Discuss negative sides. Why won't this work? 
- CDC for internal use only? Why/why not external use? (A tool for external consumers to inform about usage/ An external CDC is not a binding contract but a guideline/information pipeline to the provider)
- My test doesn't work, who's problem is it?
- Good and bad examples of cdc-tests https://docs.pact.io/best_practices/contract_tests_not_functional_tests.html
- It is impossible to write perfect API documentation. (All contracts will be flawed)
- CDC can decrease the number of api versions by telling you when you break something
- Show how pact can auto generate dependency graphs
- Display a microservice, explain what it is, which components it contains and how/why we should test all components.