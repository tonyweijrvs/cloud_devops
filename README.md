# cloud_Devops

## Introduction

Cloud-Devops is a demo project aim to deploy the trading-app(https://github.com/tonyweijrvs/trading_app_demo.git) develped in previous project. The trading-app is a microsoft application designed to simulate stock trading platform with technology like Springboot, PostgresSQL. This project will deploy the trading-app into an microsever using docker and psql command line to build and connect components together.

## Docker Architecture Diagram
![Docker Diagram](https://github.com/tonyweijrvs/cloud_devops/blob/master/assets/Untitled%20Diagram.jpg)

## dockerfile for building hte database image
```
FROM postgres
COPY ./sql_ddl/schema.sql /docker-entrypoint-initdb.d/
```
Using parent image postgres, and then copy the table initilization files into "/docker-entry-initdb.d/", using ```docker image build --tag=<name> <file_location>``` to build the image. 

```sudo docker network create --driver bridge trading-net```
create the network bridge to connect the trading-app container and postgres database container.

```
sudo docker run --name jrvs-psql \
--restart   unless-stopped \
-e POSTGRES_PASSWORD=password \
-e POSTGRES_DB=jrvstrading \
-e POSTGRES_USER=postgres \
--network trading-net \
-d -p 5432:5432 jrvs-psql
```
run the image, and set the environmental viriables for the database container and name the container jrvs-psql.


## dockerfile for building the app image
```
#
# Build stage
#
FROM maven:3.6-jdk-8-slim AS build
COPY src /build/src
COPY pom.xml /build/
RUN mvn -f /build/pom.xml clean package -DskipTests

#
# Package stage
#
FROM openjdk:8-alpine
COPY --from=build /build/target/trading-1.0-SNAPSHOT.jar /usr/local/app/trading/lib/trading_app.jar
ENTRYPOINT ["java","-jar","/usr/local/app/trading/lib/trading_app.jar"]
```
using parent maven to build a trading-app jar file image, then use the openjdk parent image to package the trading-app jar file image, later ```docker image build --tag=<name> <file_location>``` to build the trading-app image. 

```
sudo docker run \
--restart   unless-stopped \
-e "PSQL_URL=jdbc:postgresql://jrvs-psql:5432/jrvstrading" \
-e "PSQL_USER=postgres" \
-e 'PSQL_PASSWORD=password' \
-e "IEX_PUB_TOKEN=YOUR_TOKEN" \
--network trading-net \
-p 5000:5000 -t trading-app
```
run the image and set the environmental viriables, name the container trading-app, open a external port 5000 to connect to its internal port 5000
