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
Using parent image postgres, and then copy the table initilization files into "/docker-entry-initdb.d/", using ```docker image build --tag=<name> <file_location>``` to build the image and then run it. 

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
