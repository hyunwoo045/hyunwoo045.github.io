---
title: "Docker - AWS EC2에 배포"
excerpt: "나의 어플리케이션을 도커 컨테이너로 만들어 클라우드 서버에 올려봅니다."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Docker

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

## AWS EC2 에 Docker 설치

[Docker - 시작하기](https://hyunwoo045.github.io/nodejs/express/docker/2021/09/22/Docker-getstarted.html) 에서 window와 macos 는 Docker 공식 홈페이지에서 내려받은 설치 파일로 쉽게 설치가 가능했지만, Linux 환경의 가상 머신은 Docker 를 설치하는 과정이 조금 까다롭습니다.

하지만 Docker 공식 문서 홈페이지에 다운로드 하는 방법이 잘 작성되어 있기 때문에 이를 바탕으로 설치해보도록 하겠습니다.

우선은 연습용 AWS EC2 인스턴스를 만들어야 겠네요.

| 유형          | 값                                         |
| ------------- | ------------------------------------------ |
| 가상 머신     | Ubuntu Server 20.04 (HVM), SSD Volumn Type |
| 인스턴스 유형 | t2.micro (free tier)                       |
| 인스턴스 구성 | default                                    |
| 스토리지      | default (8GB, 범용 SSD)                    |
| 태그          | Name: Docker                               |
| 보안 그룹     | 모든 TCP 접속 허용 추가                    |

<br/>

## 설치

[docs.docker.com](http://docs.docker.com) 의 공식 가이드에 따라 `Ubuntu Server 20.04` 에 도커를 설치합니다.

```bash
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

- Docker 의 공식 GPG 키를 내려 받습니다.

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

- 아래 커맨드를 입력하여 `stable` 한 레파지토리를 설정합니다.

```
$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

- Docker Engine 을 설치합니다.

```
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

- 설치가 되었는지 확인합니다. `hello-world` 를 실행해봅니다.

```
$ sudo docker run hello-world
```

hello-world 라는 이미지를 찾을 수 없다는 메시지가 뜨고, image 를 pull 해오는 것을 확인할 수 있습니다. 이미지를 확인해봅니다.

```
$ sudo docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   6 months ago   13.3kB
```

설치가 완료되었습니다.

<br/>

## 이미지 생성

이제 직접 만든 프로젝트를 도커 파일로 만들고 이미지를 도커 허브에 업로드 한 후에 EC2 인스턴스에서 이를 내려받아서 실행하여 마치 서버에 배포한 것처럼 동작하도록 해보겠습니다.

예제로 사용할 프로젝트는 이전에 만들었던 [JWT-tutorial](https://github.com/hyunwoo045/JWT-tutorial) 을 사용하겠습니다. 해당 프로젝트에서 Local DB를 사용했기 때문에 일부 기능이 정상적으로 동작하지 않을 것이 분명하지만, <s>(귀찮기 때문에)</s> 어디까지나 만든 프로젝트를 바탕으로 도커 파일을 생성하고 허브에 올려 외부 클라우드 서버에 올려 동작시키는 것 자체가 목적이기 때문에 예제 프로젝트를 따로 수정하지 않겠습니다.

정리하자면 클라우드 서버에 아래 요청을 보냈을 때 올바른 응답이 오는지에 대한 확인이 목적입니다.

- GET: `public IPv4:3000/auth/check` -> `NOT LOGGED IN`
- GET: `public IPv4:3000/auth/check?token=aaa` -> `INVALID_TOKEN`
- GET: `public IPv4:3000/auth/check?token=${valid_token}` :

## Node.js 웹 앱의 도커라이징

참고 문서: [Node.js 웹 앱의 도커라이징](https://nodejs.org/ko/docs/guides/nodejs-docker-webapp/)

우선 docker hub 에서 공식 node 이미지를 내려받아야 합니다. 아래 명령어를 입력합시다.

```
$ docker pull node
```

```
$ docker images
REPOSITORY                TAG       IMAGE ID       CREATED       SIZE
node                      latest    93f5457b802e   9 days ago    908MB
```

node 를 내려받았다면 프로젝트 레파지토리 루트 경로에 Dockerfile 이라는 빈 파일을 생성합니다.

```
$ touch Dockerfile
```

그리고 아래와 같이 작성합니다.

```bash
FROM node:12 # 1.

WORKDIR /usr/src/app # 2.

COPY package*.json ./ # 3.

RUN npm install # 4.

COPY . . # 5.

EXPOSE 3000 # 6.

CMD [ "node", "./bin/www" ] # 7.
```

해석해봅시다.

1. node:12 버전을 사용.
2. docker 파일 시스템 내부에 해당 위치에 설치할 것이다.
3. package\*.json (즉, package.json 과 package-lock.json 등의 의존성 파일)들을 복사해서 현재 위치에 만든다.
4. npm install 명령을 실행해라. (의존성 모듈들을 모두 설치)
5. 현재 디렉토리의 모든 것들을 컨테이너의 현재 위치에 복사해라.
6. 3000번 포트를 사용할 것이다. (express에서 직접 설정한 포트를 입력하세요.)
7. node ./bin/www 을 실행시켜서 background에서 동작하게 할 것이다.

다음으로 .dockerignore 파일을 만들고 아래와 같이 입력합니다.

```
node_modules
npm-debug.log
```

이는 Docker 이미지에 로컬 모듈과 디버깅 로그를 복사하는 것을 막아 이미지 내에서 설치한 모듈을 덮어쓰지 않게 합니다.

<br/>

## 이미지 빌드

Docker 이미지를 빌드하는 아래 명령어를 입력하겠습니다. `-t` 옵션으로 이미지에 태그를 추가하여 구분이 쉽도록 합시다. 또한 `:` 뒤에 버전을 명시하여 태그로 버전을 확인할 수 있도록 명시합니다.

```
$ docker build . -t <username>/jwt-tutorial:0.1.0
```

빌드가 완료되었다면 확인해 봅시다.

```
$ docker images
REPOSITORY                TAG       IMAGE ID       CREATED       SIZE
hyunwoo045/jwt-tutorial   0.1.0     52d49bfade00   3 hours ago   935MB
node                      latest    93f5457b802e   9 days ago    908MB
```

이미지가 잘 생성되었습니다. localhost의 40000번 포트에 연결시키고 실행하여 API요청에 대한 응답이 오는지 확인해보겠습니다

```
$ docker run -p 40000:3000 -d hyunwoo045/jwt-tutorial:0.1.0
$ docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS         PORTS                                         NAMES
5a7d01286612   hyunwoo045/jwt-tutorial:0.1.0   "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:40000->3000/tcp, :::40000->3000/tcp   competent_chebyshev
```

`localhost:40000/auth/check` -> `NOT LOGGED IN`<br/>
위 기능을 확인해 봅니다.

![docker_images_local_test](/images/2021-09-23-Docker-serve/docker_serve_1.png)
Good!

## 도커 허브에 업로드 및 내려받기

빌드한 이미지를 어디서든 내려받을 수 있도록 `도커 허브`에 업로드 하고 원하는 환경에서 이미지를 내려받은 후에 실행까지 시켜보도록 하겠습니다.

우선은 도커에 로그인을 해 줍시다.

```
$ docker login
```

로그인이 되었다면 `push` 명령을 통해 docker hub로 이미지 파일을 push 할 수 있습니다. 아래와 같이 입력합니다.

```
$ docker push <username>/jwt-tutorial:0.1.0
```

여기까지 에러 없이 잘 진행되었다면 로컬에서의 준비는 끝났습니다. 이제 AWS EC2 인스턴스에 접속하여 이미지를 내려 받도록 합니다. 그리고 실행까지 시킨 후 테스트 해보겠습니다. (3000번 포트에 포트 포워딩 합니다.)

```
$ sudo docker pull <username>/jwt-tutorial:0.1.0
$ sudo docker run --name jwt-tutorial -d -p 3000:3000 <username>/jwt-tutorial:0.1.0
$ sudo docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED       STATUS       PORTS                                       NAMES
21dee358958a   hyunwoo045/jwt-tutorial:0.1.0   "docker-entrypoint.s…"   2 hours ago   Up 2 hours   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   jwt-tutorial
```

![docker_images_local_test](/images/2021-09-23-Docker-serve/docker_serve_2.png)

GET: `public IPv4:3000/auth/check?token=aaa` -> `INVALID_TOKEN` 에 대한 올바른 응답이 돌아온 것을 볼 수 있습니다.

<br/>

## 마무리

이렇게 따로 AWS EC2 서버에 node 를 설치하지 않고 로컬에서 도커 이미지를 생성하고 업로드하고 단지 그 이미지를 내려받아 실행만 시켰을 뿐인데도 정상적으로 동작까지 하는 것을 확인하였습니다. 이와 같이 여러 기능을 수행하는 환경을 생성하여 이미지를 만들고 같은 서버에서 이미지를 내려받아 다른 포트로 연결만 해준다면 한 서버가 다양한 기능을 수행할 수 있도록 구성할 수 있겠습니다. 감사합니다.
