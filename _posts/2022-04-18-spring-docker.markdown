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

Docker container 로 프로젝트를 실행할 때 profile 을 설정하는 방법을 우선 2가지만 정리해봄.

<br/>

## Docker build 할 때 Argumnet 설정

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

1. `FROM ...`: 각자 프로젝트에 알맞는 jdk 버전으로 빌드 (alpine 버전이 있다면 용량이 훨씬 작아 그걸로 하는게 좋음)
2. gradle 프로젝트에 알맞는 방식으로 빌드
3. `ARG ENVIRONMENT`: 빌드 시 ENVIRONMENT 라는 환경 변수를 받도록 함.
4. `SPRING_PROFILES_ACTIVE` 에 적용 시킴으로써 profile 을 설정함.
5. 네

그래서 실제로 도커 이미지 빌드 명령을 입력할 때 `ENVIRONMENT` 환경 변수를 설정해주면 된다. 아래는 예시 명령

```sh
$ docker build --build-arg ENVIRONMENT=dev -t example-auth:0.0.1 .
```

이 후에 push 하고 서버에서 pull 하고 run 시켜보면 잘 동작할 거임...

<br/>

## Docker run 할 때 환경변수 설정

위 방법대로 해도 문제는 없겠지만 profile 이 바뀔 때 마다 새로 build 를 해줘야 하는 번거로움이 있음. 그래서 `docker run` 할 때 컨테이너 환경 변수를 지정해주고 entrypoint 에서 환경 변수를 가져다가 proifle 을 지정해주는 것이 사실 내 생각엔 제일 깔끔하지 않나 생각함. 어쨌든 Dockerfile 부터 수정해보자.

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

EXPOSE 8080
ENTRYPOINT ["java", "-Dspring.profiles.active=${SERVER_MODE}", "-jar", "/app.jar"]
```

빌드할 때 `ARGUMENT` 를 받아 `SPRING_PROFILE_ACTIVE` 에 적용하는 코드를 아예 삭제함. 그리고 직접 jar 파일을 실행하는 명령어를 실행할 때 컨테이너의 환경 변수, `SERVER_MODE` 를 받아 실행하도록 함.

그럼 이제 build 할 때는 특별한 옵션 없이 빌드하면 되고,

```bash
$ docker build -t example-auth:0.0.1 .
```

run 할 때 `-e` 옵션을 이용하여 환경 변수를 전달하며 실행하도록 하자.

```bash
$ docker run --name example-auth -d -p 8080:8080 -e SERVER_MODE=dev example-auth:0.0.1
```

<br/>

# 갈무리

뭔가 쉽게 잘 되서 찝찝하다.

<br/>

## 참고자료

- [Spring-boot Profile 도커 Runtime에 적용하기](https://cubenuri.tistory.com/417)
- [Spring 환경에서 Docker run으로 jar에 argument 전달하기](https://imksh.com/94)
- [[Spring] profile로 서버 환경에 맞는 Context 적용하기(-Dspring.profiles.active)](https://velog.io/@gillog/Spring-profile%EB%A1%9C-%EC%84%9C%EB%B2%84-%ED%99%98%EA%B2%BD%EC%97%90-%EB%A7%9E%EB%8A%94-Context-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-Dspring.profiles.active)
