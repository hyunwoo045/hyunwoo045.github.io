---
title: "NestJS 시작해보기"
excerpt: "NestJS 프로젝트를 시작해보기."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Nest.js

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# 시작하기

개발자로써 입사하고 이제 6개월차.. 당연히 많은 코드를 본 것은 아니지만 나름 현재 회사의 서버 구조는 마이크로 서비스 아키텍처를 구축해두었기 때문에 다양한 서버의 다양한 언어의 다양한 코드를 볼 수 있었다. 그 중에서도 `Node.js` 로 쓰여진 서비스가 많았는데, 각각의 프로젝트 구조는 정말 각각 다른 구조를 가지고 있었다. 어떤 것은 `Controller - Model`, 어떤 것은 `Controller - Service - DTO`, 심지어는 express의 uri를 정의하는 파일에 미들웨어를 포함한 모든 로직을 작성해놓은 구조도 있었다 (그건 신입 입장에서 봐도 안 좋은 코드였다고 생각함). 자유롭게 프로젝트의 구조를 개발자 마음대로 정할 수 있다는 것은 Node.js 의 장점이자 단점이 아닐까. Java의 Spring은 어느 정도 프로젝트의 구조가 규칙으로써 정해져 있었고, 아직 공부해본적은 없지만 Python 의 django 도 구조가 정해져 있다고 들었다. 규칙과 구조가 정해져 있는 프레임워크는 실수를 줄여준다. Node.js로 사이드 프로젝트를 하면서 구조를 설계할 때에 언급하기 부끄러울 정도의 잔 실수를 상당히 많이 했었는데 그런 실수를 사전에 예방할 수 있게 해준다는 것은 상당히 매력적이다.

`Nest.js` 를 공부해봐야 겠다라고 마음 먹기 전에 약간 이에 대해서 리서치를 해보았고 많은 포스트들이 Nest.js 의 장점들, Node.js 에서 개선한 점들을 잘 설명해주는 것을 보았다. 사실 지금 OOP, FP, FRP 등의 키워드를 모두 사용한다거나 Node.js 의 모호함을 개선했다거나 하는 것들을 상세하게 적을 생각은 없다. 이 블로그는 개인적인 공부 노트이기 때문에 직접 경험하지 못한 장점들은 적지 않는다. 미래의 무치에게 저 것들을 정리하는 걸 맡기고 일단 시작해보자.

<br/>

# 설치

NestJS 는 node.js 위에서 동작하는 프레임워크이므로 node.js 는 기본적으로 설치되어 있어야 함.

- `node.js`

이제 `@nestjs/cli` 를 설치하자.

```
$ npm i -g @nestjs/cli
```

이제 새 프로젝트의 개발 환경을 아주 쉽게 구축할 수 있다.

```
$ nest new project-name
```

이제 새로 만든 프로젝트의 루트 디렉토리에 들어가보면 아래와 같은 모습의 프로젝트를 볼 수 있다 (dist, node_modules를 제외함).

```
.
├── README.md
├── nest-cli.json
├── package-lock.json
├── package.json
├── src
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── tsconfig.build.json
└── tsconfig.json
```

<br/>

# main.ts

`main.ts` 는 NestJS 의 기본 시작 파일임. 이름을 바꾸지 않도록 주의하자. 코드는 아래와 같음. 특별할 것이 딱히 없다.

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

`app.module.ts` 파일에서 생성된 모듈로 app 을 생성하고 3000번 포트에서 listening 하고 있다. 끝.

그럼 이제 module 을 살펴보자

<br/>

# app.module.ts

```ts
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

뭔가뭔가 자바스러운 느낌의 모습.. `AppModule` 이라고 하는 클래스가 있고 내부 로직은 아무 것도 없다. 하지만 그 위에 자바의 `@`어노테이션 같은 모습을 한 녀석이 있다. "데코레이터" 라고 하며 클래스 위의 함수이자 클래스를 위해 움직이는 함수이다. 그래서 위의 코드의 가장 핵심적인 부분이라고도 할 수 있고, `imports - controllers - providers` 를 정의하는 한다.

기본적으로 프로젝트에 단 하나만 존재하는 부모 모듈이다. 지금은 각 AppController와 AppService 밖에 없지만 나중에 회원 인증에 관련된 UserController 를 만든다고 가정했을 때 이 부모 모듈의 Module 데코레이터 안에 Controllers 배열 안에 UserController 를 추가해줘야 한다. 이 말인 즉슨 프로젝트에서 어떤 컨트롤러들이 있는지, 즉, 어떤 종류의 기능들이 있는지 아주 직관적으로 파악할 수 있다고 생각하면 될 듯 하다.

<br/>

# app.controller.ts

```ts
import { Controller, Get } from "@nestjs/common";
import { AppService } from "./app.service";

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

뭔가 더더욱 자바 Spring 이 떠오르는 듯. `AppController` 라는 클래스에 데코레이터 @Controller 가 있다. 누가봐도 생성자입니다 라고 말하는 `constructor` 가 있고 아래에 `getHello()` 함수가 `@Get()` 데코레이터를 달고 있다. 이는 node.js 로 치면 아래와 같은 코드.

```js
// ...

router.get("/", getHello);

function getHello(req, res) {
  res.send("Hello World!");
}
```

당연히 @Post, @Put 등을 쓰면 POST method 나 PUT method 를 쓰는 것과 같음.

<br/>

# app.service.ts

```ts
import { Injectable } from "@nestjs/common";

@Injectable()
export class AppService {
  getHello(): string {
    return "Hello World!";
  }
}
```

`app.controller.ts` 에서 호출하던 `getHello` 가 바로 이 녀석이다. 기본적으로 그냥 "Hello World" 라는 문자열을 반환하도록 되어 있다. 이 간단한 걸 왜 그냥 Controller 에서 `return "Hello World"` 하지 않았을까?

NestJS 는 비즈니스 로직과 컨트롤러를 구분 짓는 것을 지향한다. 컨트롤러는 URL 을 가져와서 알맞는 함수를 호출하는 역할이다. 상세한 비즈니스 로직은 Service 부분에 작성한다.

<br/>

```
$ npm i class-validator class-transformer
$ npm i @nestjs/mapped-types
```
