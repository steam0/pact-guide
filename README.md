# Consumer Driven Contracts implemented using Pacts

Note: For demo see [Pact Demo Installation and Testing Guide](#pact-demo-installation-and-testing-guide)

In the world of microservices change can often result in broken dependencies. Even though microservices are easy to build and run they can often become a messy pile of dependencies that slow down development. Being able to confidently build and deploy new versions is key to increase speed and reduce the cost of development. Organizations that are transitioning from a traditional monolithic design both can and will realize that it is hard to keep track of all dependencies in a complicated system. Microservice architechture can vary from only a few services to more than a thousand services that compose a complete system. Consumer Driven Contracts is a testing paradigm that let API-consumers tell the API-providers how they are using their services. This article will discuss software testing, how Consumer Driven Contracts can make developers more confident, how and when to use Consumer Driven Contracts and how to benefit from combining Consumer Driven Contracts and API-versioning.

## Anatomy of a microservice

- Display a microservice, explain what it is, which components it contains and how/why we should test all components.

## Tests

Testing code is an important part of software development. Testing code gives the developer confidence that the application works as intended but both writing and running tests is time consuming. Different types of tests have different strengths and weaknesses and there is a massive difference in the _cost_ (time spent writing and running tests) of these different types of tests. This article will give a brief introduction to three common types of tests and compare them to writing Consumer Driven Contracts.

<img src="https://github.com/steam0/pact-guide/blob/master/images/testing_pyramid.png?raw=true" width="60%">

### Unit tests

Unit testing refers to tests that verifies the internal functionality of a given application. These tests are used to validate the functionality of a given class or method and to make sure that internal functionality does not unintentionally change.

These tests are relatively cheap to write and run as they do not require any external applications to be running. Unit tests will only be able to detect errors in internal logic.

<img src="https://github.com/steam0/pact-guide/blob/master/images/unit_test_graph.png?raw=true" width="60%">

### Integration tests

Integration testing referes to tests that verify integrations from one application to another. These are typically used by calling endpoints provided by an external application and verifying the response. Such tests are less cheap than unit tests since you rely on an external service. Integration tests will require that all external services are online for the tests to pass. This is often not the case and integration tests will often be ignored.

Integration tests can also be created such that there is no external call and the test receives a mocked response that you can run your tests with. Using mocks reduces the cost of running integration tests, but it also removes some of the confidence that integration test is supposed to give.

<img src="https://github.com/steam0/pact-guide/blob/master/images/integration_test_graph.png?raw=true" width="60%">

###  End-to-end tests

End-to-end testing refers to tests that verify that the system as a whole meets its requirement. These tests will be executed on a system of running applications and verify that actions get executed properly and that responses are valid.

Writing and executing system tests are very costly to implement and run, and will often be neglected and completely ignored.

<img src="https://github.com/steam0/pact-guide/blob/master/images/system_test_graph.png?raw=true" width="60%">

## Consumer Driven Contracts

<img src="https://github.com/steam0/pact-guide/blob/master/images/cdc_steps_1.png?raw=true" width="50%">

When an application is consuming an external service, the application becomes a _consumer_ of that service. The external service is now a _provider_ of services to this consumer. The consumer is calling different endpoints on the external service and is writing integration tests based on the response. 

The only types of tests that are low cost to write and run are _unit tests_ and _mocked integration tests_. Developers often forget or avoid implementing good integration or system tests because of the cost which then removes the confidence these tests provide. There is also an issue with using mocked integration tests as a cost effective option. If and when a service changes its interface, the mocked integration test on the consumer side will fail to detect these changes and the consumer application will crash even while having green builds. Consumer Driven Contracts provides a solution to this exact problem.

Consumer Driven Contracts is a testing paradigm which let consumers of a service define a contract that the service can validate against. These tests are an alternative to the traditional (mocked) integration test, but are executed on both the consumer and the service provider application.

<img src="https://github.com/steam0/pact-guide/blob/master/images/testing_pyramid_cdc.png?raw=true" width="60%">

## APIs are contracts

> As an API is a contract from the provider to any consumer describing how to use services from the provider, a pact is a contract from a given consumer to the provider describing how the consumer uses services from the provider.

An API is a contract. The provider guarantees that if you use the API as described, they will provide responses according to the API-specification. [_The OpenAPI Specification_](https://github.com/OAI/OpenAPI-Specification) is a framework for an API provider to expose APIs as a contract using json or yaml. Even though such frameworks help developers share information about request data and response data, it does not explain the intended usage of an endpoint and it is up to the developer to write what the endpoint really do. This allows for consumers to misunderstand the intended usage. 

A consumer will usually write integration tests to confirm that they have interpreted the API contract correctly. These integration tests will describe how the consumer use the provided API and may show which assumptions the consumer made about the API. However this does not provide any guidelines for the provider to make sure that they don't break the consumer integration while making changes to the provider application.

_Consumer Driven Contracts_ let consumers to write mock integration tests and create a _consumer contract_ that defines how the consumer is using the API. These integration test are then given to the provider to run whenever they modify their code. Having this consumer-provider contract relationship creates confidence for the consumer that their services will keep working even when providers upgrade theirs.

## Pact

Pact is a framework for implementing Consumer Driven Contracts in an application. The consumer is able to write a contract to the provider by creating a _pact_. A pact is an integration test written by the consumer, that the provider can add to their test suite and run before building (or deploying) a new version of the application. A pact file will contain information about which endpoint is called, request data, response data and whatever validation of the response data that the consumer chose to perform.

## Publishing pacts

When a consumer have generated a pact file it should publish this to the providers. This can be done by attaching the file as a part of the repository (do not do this), by exposing the file at a url (should not do this) or by publishing it to a shared _broker_. 

A pact broker is a separate application which should be a part of any system implementing pacts. The pact broker will be waiting for pact files from any consumer and is indexing them based on which provider they are communicating with. When a pact is published to the broker it becomes available to any provider that want to validate them.

<img src="https://github.com/steam0/pact-guide/blob/master/images/cdc_contracts.png?raw=true" width="60%">

## Why you should apply Consumer Driven Contracts to your system

> If you don't know who your consumers are you will not be able to predict how your API is being used.

When developing applications, and specifically applications in a microservice architechture, tests are written to ensure system stability and to provide a solid foundation for future development. In a microservice architecture it is not always clear which applications are using your service. When accepting this premise it becomes clear that having unit tests and integration tests would be useless for the provider of an API. If you don't know who your consumers are you will not be able to predict how your API is being used and even if you do know who your consumers are you still won't successfully predict how they use your API.

Consumer Driven Contracts will as discussed earlier provide a solution to this problem by providing a design pattern for consumers to apply while writing tests. Since all applications *should* have both unit tests and mocked integration tests there is no significant code change needed on the consumer side to create a contract for the provider to validate.

Consumer Driven Contracts will give your developers the confidence to keep deploying new versions of their applications while knowing that they will not break functionality for any consumers.

> organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations.
> -- Melvin Conway

This famous quote from Melvin Conway is often used to explain why computer systems end up like they do. The qoute gets even more useful when trying to explain systems using microservice architechture since dependencies are not as obvious as they are in a monolith. Much like internal departments in an oragnization, microservices have seperate responsibilities and concerns. When different departments or teams develop services they will often model their APIs the same way as they communicate with other departments. Implementing Consumer Driven Contracts to a computer system will generate a perfect platform for developers, teams and departments to communicate better. Both parties will provide contracts to each other which will make the consumer discover information about how to use an API and the provider will discover how the consumer uses their API and which assumptions the consumer made. It is as true in software development as it is in business: Communication is the key to success.


## Objections to Consumer Driven Contracts

> Using Consumer Driven Contracts will move the responsibilty of compatibility from the consumer to the API provider

It is the API provider that decides what their contract to the world is and it is up to the consumer of an API to comply with this contract. This is one of the main objections to implementing the CDC paradigm. This is however not true. On one hand you want to be in control of your own API, but you also want to attract users to your API. It is important to keep an open mind and assess your consumers. The Consumer Driven Contracts paradigm provide a symbiotic relationship between the API provider and the consumer much like the symbiotic relationship between the seller (API provider) and the buyer (API consumer).

> Having Consumer Driven Contracts will prevent developers from making changes as the tests will break

This is not entirely true. Using Consumer Driven Contracts will help developers by making them observant on breaking changes in their code. By using a CDC framework like Pact makes it abundantly clear who uses your service and how they are using it. If a change break a consumer this creates an excellent opportunity for developers to _talk_ with each other. Communication is the key to success and CDC help consumers and providers communicate.

If and/or when a Consumer Driven Contracts test break, developers from the provider should investigate if the breaking change is intended and notify the consumer about it. If the breaking change is intended both teams of developers will need to cooperate with each other to solve the issue. If the teams cannot agree on who should fix this issue, then both teams need to grow up. Symbiotic relationships require that both sides give what the other need. A consumer contract test should not prohibit further development of an application. A consumer contract should be considered a guideline for the provider to ensure service reliability. Unless there are other legal contracts involved then the provider should just go ahead and deploy the changes. Temporary API-versioning of the specific failing tests can also be considered in extreme cases.

## Internal vs. external usage

Consumer Driven Contracts will be able to solve a lot of different issues that may occur while developing microservices in a large organization, but it is not always clear when to use this testing pattern or who you should expose this testing paradigm to. 

> Consumer Driven Contracts should definitely be used internally in a development team

A development team should usually consist of 3-9 people and a can have resposibility for houndreds of different microservices within an organization. All development within a team should definitely use Consumer Driven Contracts when creating dependent services. Even though all members of the development team should know which services talk to each other but as the number of services grow the system gets more unclear. Using Consumer Driven Contracts will make it easier for developers to know which of their supported services are dependent on other services, and it will make it much easier for new team members to disover and understand all system dependencies.

> Consumer Driven Contracts should almost definitely be used internally in an organization

Organizations come in many different shapes and sizes. Because of that, there is no definite way to decide if an organization should use Consumer Driven Contracts or not. It ultimately depends on the size of the organization and the number of services that the organization support and inn most cases the answer will be yes. Larger organizations that produce microservices will usually have multiple development teams that consume services created by other internal teams and these organizations should apply Consumer Driven Contracts to their development. Not only will this create a great platform for teams to communicate with each other but it will improve stability of the entire communications.

> Consumer Driven Contracts can be used for external consumers

Some services will have external consumers that are known to the provider. In these cases can Consumer Driven Contracts be exposed from the providing organization to their consumers. By letting their known customers show how they are using the API will let the providing organization make better decisions on what needs to be improved and how they can do it without breaking customer systems. Consumer Driven Contracts from an external consumer should always be described as guidelines and not absolute rules. The external consumer should always be notified if and when onfe of their contracts will get broken. 

> Consumer Driven Contracts should not be used for open APIs

Any organization provides open APIs should not let unknown consumers provide contracts to the providing API due to the unneccesary dependencies that will be created. In the case where open APIs let all their consumers provide contracts they should always use it as a guideline. For open APIs this data will often not be useful due to the lack of implementation on the consumer side. It will not help a development team if only one consumer out of a houndred chose to provide a contract.

<img src="https://github.com/steam0/pact-guide/blob/master/images/cdc_usage.png?raw=true" width="100%">

There is a big difference between internal consumers and external consumers when considering how strictly a provider should comply with the test results. When Consumer Driven Contracts tests from an internal consumer fails it should be considered as breach of a binding contract. When contracts from an external consumer fails it should only be considered as information discovery. Report what APIs break and why. A guideline is purely there to discover faults, not to prohibit development.

## API versioning

API versioning is an important tool to use in order to make microservices truly decoupled from each other. There are many different ways of implementing API versioning, but that will not be discussed here. API versioning frees an application API from all its existing contracts by defining a new version of that contract. By doing so a developer will then have to support and maintain `n+1` versions of it's API. This makes API versioning a tool that should be used carefully.

> API versioning is a great theory that might be awful to implement. When building microservices and doing multiple deploys per day, developers may risk having to create new versions of the API every day.

API versioning is not by itself how to solve decoupled releases of microservices. Not because it is impossible to implement, but because it ultimately will result in having way more code to maintain which again inhibits development speed. Having multiple versions of an API makes the codebase unneccesarily large and makes it more likely to contain errors.

> API versioning does not answer the question: When do I need to create a new API version? 

Even though API versioning makes it possible to support consumers depending on older versions of an API, it does not provide the desired confidence while maintaining the API. While correcting a mistake in an older API version the change might break the consumers of that API version. This is where using Consumer Driven Contracts will make developers more confident when making changes to the code. Developers will be able to maintain older versions of an API with confidence by validating all consumers of that specific API version.

## Summary

* Different testing paradigms.
* WHy CDC matter
* Confidence
* Combining CDC and API-versioning

# Pact Demo Installation and Testing Guide

This is a set of example implementations of Pact. First we will set up a Pact broker on the local machine. Then we will set up a consumer, and finally a provider that will use the contract (Pact) created by the consumer and verify that the API is within the consumer contract.

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
- Discuss negative sides. Why wont this work? 
- CDC For internal use only? Why/Why not external use? (A tool for external consumers to inform about usage/ An external CDC is not a binding contract but a guideline/information pipeline to the provider)
- My test doesnt work, whos problem is it?
- Good and bad examples of cdc-tests https://docs.pact.io/best_practices/contract_tests_not_functional_tests.html
- It is impossible to write perfect API documentation. (All contracts will be flawed)
- CDC can decrease number of api versions by telling you when you break something
- Show how pact can auto generate dependency graphs