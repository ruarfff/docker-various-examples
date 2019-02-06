# Generator: a very basic docker image with just enough to pull down a generated java project and unpack it.
FROM alpine as generator

WORKDIR /usr/gen/

RUN apk update
RUN apk add curl
RUN apk add unzip

RUN curl 'https://start.spring.io/starter.zip?type=maven-project&language=java&bootVersion=2.1.2.RELEASE&baseDir=demo&groupId=com.example&artifactId=demo&name=demo&description=Demo+project+for+Spring+Boot&packageName=com.example.demo&packaging=jar&javaVersion=1.8&autocomplete=&generate-project=&style=devtools&style=web' --compressed --output demo.zip
RUN unzip demo.zip
RUN rm demo.zip

# Builder: Comes with maven to allow us to build and run the application code copied over from generator
FROM openjdk:13-alpine as builder

RUN mkdir /usr/app
WORKDIR /usr/app

COPY --from=generator /usr/gen/demo .

RUN ./mvnw clean install

