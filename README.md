# Consumer Driven Contracts implemented using Pacts

This demo is a proof of concept of how it works and why we should implement _Consumer Driven Contracts_ (CDC) when developing microservices. In this demo CDC is implemented using the Pact framework.

## Testing

There are many different testing types but we will just talk about three of them.

### Unit Testing

Unit testing refers to tests that verifys the functionality of the different functions. These tests are used to validate that the functionality of a given class or method is as you expect.

These tests are relatively cheap as they do not require any external applications to be running. 

### Integration Testing

Integrations testing referes to tests that verify integrations from one application to another. These are typically used by calling services on an external application and verifying the response. Such tests are less cheap than unit tests since you rely on an external service.

Integration tests can also be created such that there is no external call and the test receives a mocked response that you can run your tests with. Using mocks, reduced the cost of integration tests.

###  System Testing

System testing refers to tests that verify that the system meets its requirement. These tests will run through a running system and verify that actions gets executed and that responses are valid. 

Writing and executing system tests are very costly.

## Consumer Driven Contracts

Knowing a little bit about testing, we see that the only tests that is low cost to write and run are _unit tests_ and _mocked integration tests_. Because of this developers often forget or avoid implementing good integration or system test. There is also an important problem with using mocked integration tests. What happens if the service changes what it returns? A mocked integration thest on the consumer side will not detect these changes. This is where Consumer Driven Contracts might help during development.

CDC is a testing paradigm which let consumers of a service define a contract that the service can validate against. These tests are an alternative to the traditional _Integration Test_, but are located on the service provider side.

## Creating Pacts

When an application is consuming an external service, the application becomes what Pact calls a _consumer_. The external service is now a _provider_ of services to the consumer. The consumer is calling different endpoints on the external service and is writing tests based on the response. 

The consumer can now write a contract to the provider by creating a _Pact_. A pact is an integration test written by the consumer, that the provider will run before building(or deploying) a new version of the application.

> As an API is a contract from the provider to any consumer describing how to use services from the provider, a pact is a contract from a given consumer to the provider describing how the consumer uses services from the provider.

## APIs are contracts

An API is a contract. The provider guarantees that if you use the API as described, they will provide responses according to the API-specification. The consumer writes integration tests to confirm that the API works as described for the consumer service. However this does not provide any guidelines for the provider to make sure that they don't break the consumer integration.

Using _Consumer Driven Contracts_ is a way for consumers to write mock integration tests and publish the contract (Pact) to a broker to let the provider access it.

# Pact Demo Installation and Testing Guide

## Install and configure database
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

## Run pact tests

1. Get a producer application

Get a copy of the provider application: https://github.com/steam0/pact-provider

```
git clone git@github.com:steam0/pact-provider.git
```

2. Run tests

```
./gradlew clean build
```




