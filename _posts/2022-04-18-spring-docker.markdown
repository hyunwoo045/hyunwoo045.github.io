---
title: "Spring Boot - 환경 분리와 Docker 이미지 빌드"
excerpt: Spring boot 프로젝트를 Docker 이미지로 빌드하며 환경 설정해보자.

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Spring Boot
  - Docker

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# 환경의 분리

서비스를 하는 곳마다 다르겠지만 일반적으로 개발 환경은 기본적으로 로컬 개발 환경, 개발 환경, 운영 환경으로 나뉘고 추가로 스테이징 환경 등으로 더 나눌 수 있음. 그럼 각 환경마다의 설정이 당연히 달라짐 (예를 들어 바라보고 있는 데이터베이스의 호스트 라던가.. 운영 환경에서는 ddl query가 동작하지 않게 하고 sql query log 를 찍지 않는다던가...)

가장 원초적인 방법으로다가 코드를 서버에 올린 후 서버에 직접 들어가서 환경 설정 파일(ex. `application.yml`)을 직접 수정할 수 있지만 위험하기도 하고 뭐니뭐니해도 엄청나게 귀찮음. 그래서 Spring Boot (gradle 프로젝트) 는 어떻게 환경을 분리하는지, 그리고 도커로 이미지 빌드 하는 법과 빌드하면서 환경을 설정하는 법을 정리함.

<br/>

# Gradle 프로젝트 환경 분리 (with IntelliJ)

Spring boot 2.4 버전 이 전에 사용하던 방식이 있었는데 deprecated 되었다고 하니 삼고명하고 새로운(?) 방식을 정리함.

간단하다. `application.yml` 파일을 `application-${}.yml` 와 같은 형식으로 나누면 된다. 예를 들어서 `application-local.yml`, `application-dev.yml`, `application-prod.yml` 정도로 나눌 수 있음.

나눈 상태에서 어떤 환경을 적용시킬 건지는 아래와 같이 하면 됨.

![](/images/2022-04-18-spring-docker/1.png)

![](/images/2022-04-18-spring-docker/2.png)

짱 쉽다.

<br/>

# Spring Boot (Gradle) Docker 이미지 빌드

쉽게 해보자. 우선 Dockerfile 생성

```docker
FROM openjdk:11 AS builder
COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .
COPY src src
RUN chmod +x ./gradlew
RUN ./gradlew bootJar

FROM openjdk:11
COPY --from=builder build/libs/*.jar app.jar

ARG ENVIRONMENT
ENV SPRING_PROFILES_ACTIVE=${ENVIRONMENT}

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```
