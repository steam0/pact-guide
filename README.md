# Pact Guide

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

```
Get a copy of the consumer repo: https://github.com/steam0/pact-consumer

`git clone git@github.com:steam0/pact-consumer.git`
```

2. Run tests and push pact file to broker

```
./gradlew clean build publishPact
```

## Run pact tests

1. Get a producer application

```
Get a copy of the provider application: https://github.com/steam0/pact-provider
`git clone git@github.com:steam0/pact-provider.git`
```

2. Run tests

```
./gradlew clean build
```