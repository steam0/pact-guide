# Consumer Driven Contracts implemented using Pacts

> Note: For demo see [Pact Demo Installation and Testing Guide](#pact-demo-installation-and-testing-guide)

This demo is a proof of concept of how it works and why we should implement _Consumer Driven Contracts_ (CDC) when developing microservices. In this demo CDC is implemented using the Pact framework.

## Tests

There are many different testing types. Different types of tests have different strengths and weaknesses. There is also a massive difference in the _cost_ of implementing different types of tests. This article will only mention the three types that are relevant when talking about Consumer Driven Contracts.

### Unit tests

Unit testing refers to tests that verifies the internal functionality of a given application. These tests are used to validate that the functionality of a given class or method to make sure that internal functionality does not unintentionally change.

These tests are relatively cheap to write and run as they do not require any external applications to be running. Unit tests will only be able to detect errors in internal logic.

- Bar graph containing two bars: cost, confidence

### Integration tests

Integrations testing referes to tests that verify integrations from one application to another. These are typically used by calling endpoint provided by an external application and verifying the response. Such tests are less cheap than unit tests since you rely on an external service. Integration tests will require that all external services are online for the tests to pass. This is often not the case and integration tests will often be ignored.

Integration tests can also be created such that there is no external call and the test receives a mocked response that you can run your tests with. Using mocks, reduces the cost of integration tests, but it creates other problem that will be discussed later.

- Bar graph containing two bars: cost, confidence

###  System tests

System testing refers to tests that verify that the system meets its requirement. These tests will be executed on a system of running applications and verify that actions get executed properly and that responses are valid.

Writing and executing system tests are very costly to implement and run, and will often be neglected and completely ignored.

- Bar graph containing two bars: cost, confidence

## Consumer Driven Contracts

Knowing a little bit about testing bring to light a big problem with testing software: The only types of tests that are low cost to write and run are _unit tests_ and _mocked integration tests_. Developers often forget or avoid implementing good integration or system tests because of the cost which then removes the confidence these tests provide. There is also an important problem with using mocked integration tests as a cost effective option. If and when a service changes its interface, the mocked integration test on the consumer side will fail to detect these changes and the consumer application will crash even while having green builds. Consumer Driven Contracts provides a solution to this exact problem.

CDC is a testing paradigm which let consumers of a service define a contract that the service can validate against. These tests are an alternative to the traditional integration test, but are executed on both the consumer and the service provider application.

When an application is consuming an external service, the application becomes a _consumer_ of that service. The external service is now a _provider_ of services to this consumer. The consumer is calling different endpoints on the external service and is writing integration tests based on the response. 

## APIs are contracts

> As an API is a contract from the provider to any consumer describing how to use services from the provider, a pact is a contract from a given consumer to the provider describing how the consumer uses services from the provider.

An API is a contract. The provider guarantees that if you use the API as described, they will provide responses according to the API-specification. The consumer writes integration tests to confirm that the API works as described for the consumer service. However this does not provide any guidelines for the provider to make sure that they don't break the consumer integration.

Using _Consumer Driven Contracts_ is a way for consumers to write mock integration tests, publish the contract to a broker and let the provider access it.

## Pacts

Pact is a framework for implementing Consumer Driven Contracts in your applications. The consumer is able to write a contract to the provider by creating a _Pact_. A pact is an integration test written by the consumer, that the provider will run before building (or deploying) a new version of their application.

## Why should you apply Consumer Driven Contracts to your system

> If you don't know who your consumers are you will not be able to predict how your API is being used.

When developing applications, and specifically applications in a microservice architechture, tests are written to ensure system stability and to provide a solid foundation for future development. In a microservice architecture it is not always clear which applications are using your service. When accepting this premise it becomes clear that having unit tests and integration tests would be useless for the provider of an API. If you don't know who your consumers are you will not be able to predict how your API is being used and even if you do know who your consumers are you still won't successfully predict how they use your API.

As discussed earlier will Consumer Driven Contracts solve this problem for you by providing a design pattern for consumers to apply to their tests. Since all applications *_should_* have both unit tests and mocked integration tests there is no significant code change needed on the consumer side to create a contract for the provider to validate.

Consumer Driven Contracts will give your developers the confidence to keep deploying new versions of their applications while knowing that they won't break functionality for any of their consumers.

## Objections to Consumer Driven Contracts

> Using Consumer Driven Contracts will move the responsibilty of compatibility from the consumer to the API provider

It is the API provider that decides what their contract to the world is and it is up to the consumer of an API to comply with this contract. This is one of the main objections to implementing the CDC paradigm. This is however not true. On the one hand you want to be in control of your own API, but you also want to attract users to your API. It is important to keep an open mind and assess your consumers. The Consumer Driven Contracts paradigm provide a symbiotic relationship between the API provider and the consumer much like the symbiotic relationship between the seller (API provider) and the buyer (API consumer).

> Having Consumer Driven Contracts will prevent developers from making changes as the tests will break

This is not entirely true. Using CDC will help developers by making them observant on breaking changes in their code. By using a CDC framework like Pact makes it abundantly clear who uses your service and in what way they are using it. If a change break a consumer this creates an excellent opportunity for developers to _talk_ with each other. Communication is the key to success and CDC help consumers and providers communicate.

If and when a CDC test break should developers from the provider investigate if the breaking change is intended and notify the consumer about it. If the breaking change is intended both teams of developers will need to cooperate with each other to solve the issue. If the teams cannot agree on who should fix this issue, then both teams need to grow up. Symbiotic relationships require that both sides give what the other need. A CDC test should not prohibit further development of an application so a CDC should be considered a guideline for the provider to ensure service reliability. Unless there are other legal contracts involved then the provider should just go ahead and deploy the changes. Temporary API-versioning of the specific failing tests can also be considered in extreme cases.

## API versioning

API versioning is an important tool to use in order to make microservices truly decoupled from each other. There are many different ways of implementing API versioning, but that will not be discussed here. API versioning frees an application API from all its existing contracts by defining a new version of that contract. By doing so a developer will then have to support and maintain `n+1` versions of it's API. This makes API versioning a tool that should be used carefully.

> API versioning is a great theory that might be awful to implement. When building microservices and doing multiple deploys per day, developers may risk having to create new versions of the API every day.

API versioning is not by itself how to solve decoupled releases of microservices. Not because it is impossible to implement, but because it ultimately will result in having way more code to maintain which again inhibits development speed. Having multiple versions of an API makes the codebase unneccesarily large and makes it more likely to contain errors.

> API versioning does not answer the question: When do I need to create a new API version? 

Even though API versioning makes it possible to support consumers depending on older versions of an API, it does not provide the desired confidence while maintaining the API. While correcting a mistake in an older API version the change might break the consumers of that API version. This is where using Consumer Driven Contracts will make developers more confident when making changes to the code. Developers will be able to maintain older versions of an API with confidence by validating all consumers of that specific API version.

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
- Releasing new versions with confidence
- Impossible to predict all corner cases of usages
- > API versioning is a great theory that is awful to implement. When building microservices and doing multiple deploys per day developers may risk having to support multiple different API-versions. API versioning is great while having regular scheduled system upgrades.
- Discuss negative sides. Why wont this work? 
- CDC For internal use only? Why/Why not external use? (A tool for external consumers to inform about usage/ An external CDC is not a binding contract but a guideline/information pipeline to the provider)
- My test doesnt work, whos problem is it?
- Good and bad examples of cdc-tests https://docs.pact.io/best_practices/contract_tests_not_functional_tests.html
- It is impossible to write perfect API documentation. (All contracts will be flawed)
- CDC can decrease number of api versions by telling you when you break something