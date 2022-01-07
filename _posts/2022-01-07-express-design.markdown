---
title: "[프로젝트 일기] 메신저 웹 API 제작 일기 (1) - 프로젝트 구조"
excerpt: "내가 설계한 프로젝트 구조"

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Node.js
  - Express

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# 프로젝트 시작하기

- 서버: `Node.js v17.0.1`
- 서버 모듈: `Express ~4.16.1`
- 데이터베이스: `MySQL v8.0.27`, `Mongo v5.0.3`
- 배포 서버: `AWS EC2 Ubuntu 20.04` 공짜

이 프로젝트는 2인 협업 프로젝트며, 간단한 '메신저'를 만드는 프로젝트이다. 작성자는 API 서버 구현을 맡았고, 간단한 포스팅을 통해 개발하는 동안 기록으로 남기고 싶은 내용들을 적어본다.

언어는 자바스크립트, node.js를 선택하였다. 두 데이터베이스를 썼고, 하나는 관계형 데이터베이스인 MySQL, 다른 하나는 비관계형 데이터베이스인 Mongo 를 사용하였다.

MySQL은 사용자-친구, 사용자-방과 같이 관계가 명확하거나 회원 정보와 같이 데이터 스키마가 정형화되어 있는 데이터들을 주로 저장하는 용도로 사용하였다. 또한 관계형 데이터베이스의 경우 NoSQL 에 비해 데이터의 유실 가능성이 낮으므로 아주 중요한 데이터들을 저정하는 데에 사용하였다. Mongo 는 방의 세부 정보를 저장하는 용도로 채택했다. 방 세부 정보에는 참여하고 있는 유저, 채팅 기록이 있으며 특히 채팅 기록의 경우 배열의 크기가 아주 크고 중복이 많을 것으로 예상하여 관계형 데이터베이스에 저장하는 것은 적합하지 않다고 생각하여 Mongo 에 저장하기로 하였다.

현재 배포 서버가 존재하며 프리 티어 AWS EC2 인스턴스에서 동작 중이다. 배포는 Docker 를 사용하여 이미지를 빌드 후 컨테이너화 하는 방식을 사용하였다.

<br/>

# 프로젝트 구조

```
├── app.js
├── config
├── controller
├── models
├── package-lock.json
└── package.json
```

프로젝트 구조는 MVC (Model-View-Controller) 패턴을 표방하고 있다. API 서버이기 때문에 View 파트가 없어 . MVC 패턴을 표방한 만큼, Controller는 사용자로부터 데이터를 전달받아 검증하고, 필요한 형태로 가공 및 다시 사용자에게 데이터를 전달하는 역할을 주었고, Model은 DB와의 직접적인 상호작용을 하는 역할을 주었다. config 디렉토리는 서버 initialize 에 필요한 모듈들 (HTTP Server, Mongo, Socket, Routes)과 환경 변수를 정의하는 파일들을 모아놓았다.

<br/>

# - config

예전엔 설정 코드들을 `app.js` 에 다 때려 박았었다.

```js
const http = require("http");
const createError = require("http-errors");
const express = require("express");

const mongoose = require("mongoose");
const mysql = require("mysql");
const { Server } = require("socket.io");

// .....대충 http server를 설정하는 코드들

// .....대충 mongo db 설정하는 코드들

// .....대충 mysql pool 생성하는 코드들

// .....대충 소켓 로직 처리하는 코드들
```

예전에 혼자 게시판 만들기를 할 때만 해도 이런 식으로 했었는데, 어느 순간보니 코드들이 뒤죽박죽이 되어 있었다. 한 파일에 뭐 http 서버 코드 한 뭉탱이, session 관리하는 코드 한 뭉탱이, 뭉탱이 별로 잘 모아놨더라면 좀 덜 지저분해 보였을텐데, 그 땐 그 것도 못했고 남들이 그런 코드를 유지보수 하려 한다면 진짜 짜증이 날 것이다. (나도 내 께 짜증나는데..)

그래서 아래와 같이 바꿔보았다.

```js
// app.js
const httpServer = require("./config/initializer/httpServer");
const mongodb = require("./config/initializer/mongodb");
const websocket = require("./config/initializer/websocket");
const logger = require("./config/logger");
const path = require("path");

const runServer = async () => {
  require("dotenv").config({
    path: path.join(__dirname, `./config/env/${process.env.MODE}.env`),
  });
  try {
    await httpServer(process.env.PORT);
    await websocket(process.env.SOCKET_PORT);
    await mongodb({
      host: process.env.MONGO_HOST,
      port: process.env.MONGO_PORT,
    });
    logger.info(`[CHAT-APP] initialization SUCCESSFULLY!`);
  } catch (error) {
    logger.error(error);
  }
};

runServer();
```

HTTP Server 를 구동하는 코드들을 따로, mongo 초기화하는 코드들을 따로, 소켓 로직 정의해놓은 코드들을 따로 모아 다른 파일에 적어놓고 `app.js` 에서는 비동기적으로 하나씩 실행하기만 하면 되도록 함. 맞는 방식인지는 모르겠지만 "잘 돌아가고", "잘 분리" 됬으면 우선은 그걸로 됬다고 생각한다. 앞으로 현업에서 좋은 코드들을 많이 보는 수 밖에,,

<br/>

# - routes, controller

회원가입 기능을 수행하는 `POST /auth` 를 만든다 쳐보자. 이 역시 직전 프로젝트의 작성자는 아래와 같이 코드를 짰다.

```js
// app.js

const http = require("http");
const express = require("express");
const authRouter = require("../routes/authRoute");

const app = express();
app.use("/auth", authRouter);
```

```js
// config/routes/authRoute.js

const express = require("express");
const router = express.Router();
router.post("/", function (req, res) {
  // 1. 아이디 중복 확인.
  // 대충 mysql 에서 ID 를 select 해서 가져온다는 내용
  // 2. 비밀번호 암호화.
  // 3. 닉네임 중복 확인
  // 대충 mysql 에서 nickname 을 select 해서 가져온다는 내용
  // 4. mysql 에 insert
});
```

위에거야 다 주석으로 흐름만 적어서 와닿지 않을 수 있는데, 실제 작성자의 회원가입 코드는 100줄 가량 된다. 이 `authRoutes.js` 파일이 `POST /auth` 만 있고 끝나면 다행이겠지만, `auth` 에는 로그인, refresh token 재 발급하는 api 가 따로 정의되어 있다.

```js
// config/routes/authRoutes.js

const express = require("express");
const router = express.Router();

router.post("/", function (req, res) {
  // 대충 100줄임
});

router.post("/login", function (req, res) {
  // 대충 130줄임
});

router.get("/refresh_token", function (req, res) {
  // 대충 30줄임
});
```

이 역시 와닿지 않을 수 있지만, 한 파일에 몇 백줄 가량의 코드가 몰려 있다면 진짜 읽기 힘들고 유지보수하기도 엄청나게 어렵다. 작성자는 이 것을 보고 '틀렸다' 라고 말 할 짬이 안된다. 실제로 서비스 중인 API 서버 중에 위와 같은 형태로 작성되어져 있는 코드도 본 적이 있기 때문에 더더욱 '아 저건 잘못됬어요' 라고 말 할 수 없다. 하지만 '저 코드 보기 X같아요' 라고는 말 할 수 있다. 작성자는 아직도 자기가 짠 코드조차도 잘 못 쫓아가는 응애이므로, 프로젝트 구조를 잘 나눌 필요가 있었고 회사의 많은 코드들을 읽어보며 고민한 결과 Controller와 Model로 구분짓기로 마음 먹었다.

트리로 따지면 아래와 같고,

```bash
├── config
│   └── routes
│       ├── authRoute.js
├── controller
│   ├── auth
│   │   ├── index.js
│   │   ├── login.js
│   │   ├── refreshToken.js
│   │   └── registUser.js
└── models
    └── userModels.js
```

코드의 뼈대는 아래와 같다.

`config/routes/authRoutes.js`

```js
const app = require("express");
const router = app.Router();
const controller = require("../../controller/auth");

router.post("/login", controller.login);

router.post("/", controller.registUser);

router.get("/refresh_token", controller.refreshToken);

module.exports = router;
```

`controller/auth/index.js`

```js
const login = require("./login");
const registUser = require("./registUser");
const refreshToken = require("./refreshToken");

module.exports = {
  login,
  registUser,
  refreshToken,
};
```

`controller/auth/login.js`

```js
/* METHODS */
function getUserById(userId) {
  return Promise((resolve, reject) => {
    // 대충 mysql 에서 id로 user 정보 가져온다는 코드
  });
}

function validatePassword(password, context) {
  return Promise((resolve, reject) => {
    // 대충 복호화한 db.password랑 입력 password랑 비교한다는 코드
  });
}

function getAccessToken(payload) {
  return Promise((resolve, reject) => {
    // access token 받아오기
  });
}

function getRefreshToken(payload) {
  return Promise((resolve, reject) => {
    // refresh token 받아오기
  });
}

/* EXPORTS */
module.exports = async function (req, res) {
  const { userId, password } = req.body;
  try {
    // 비동기적으로 methods 실행
    const dbUserInfo = await getUserById(userId);
    await validatePassword(password, dbUserInfo);

    const { userId, nickname, avatarUrl, statusMessage } = dbUserInfo;
    const payload = { userId, nickname, avatarUrl, statusMessage };

    const token = await getAccessToken(payload);
    const refreshToken = await getRefreshToken(payload);

    res
      .cookies("refreshToken", refreshToken)
      .send({ user: payload, accessToken: token });
  } catch (error) {
    // 악! 에러임! 더러워!
  }
};
```

router 를 정의 하는 부분에는 어떤 기능들(endpoint)가 있는 지 깔끔하게 보이고, 보고 싶은 기능은 따로 파일을 찾아 들어가서 로직을 볼 수 있다. 자존감이 낮은 작성자는 이 것이 맞음 님들도 이렇게 하셈 이라고 말할 자신은 없지만, 확실히 코드를 보는 데에 불편함이 별로 없었다.

<br/>

## 여담

회사에서 하루는 이런 코드를 본 적이 있다. 메서드들을 비동기로 처리하기 위해 `runner.js` 라는 common function 정의해두었는데 아래 모습과 같았다.

```js
function runner(methods, context, next) {
  let last = methods.length - 1,
    asyncArr = [];

  (function run(pos) {
    methods[pos].call(context, function (err) {
      if (err) return next(err);
      else if (pos === last) return next(err);

      process.nextTick(function () {
        run(++pos);
      });
    })(0);
  });
}

module.exports = runner;
```

인자로 전달받는 `methods` 비동기로 순차 실행될 함수들을 배열의 형태로 넣어 둔 형태이고, `context`는 call 함수에서 볼 수 있듯이 `this` 다.

그래서 `runner` 를 호출하는 controller 는 아래와 같았다.

```js
const runner = require("./runner.js");

function firstTask(done) {
  this.firstTaskResult = this.req.userId;
  done();
}

function secondTask(done) {
  this.secondTaskResult = this.req.nickaname;
  done();
}

module.exports = function (req, res, next) {
  this.req = req;
  this.res = res;

  runner([firstTask, secondTask], this, next);
};
```

아기개발자 입장에서는 굉장히 있어보이는 코드였다. 그래서 실제로 조금 더 코드를 자세히 들여다보고 이 프로젝트에 적용시켜 보았는데, 코드를 리뷰하는 과정에서 아래와 같은 이슈가 있었다.

1. Javascript의 `this`는 자바나 c++에 비해 너무 불분명하다.
2. `runner` 의 핵심은 `process.nextTick()` 으로 메서드를 제어하는 것인데, `process.nextTick()`을 사용할 때는 주의해야 할 점이 있다.

사실 `this` 가 자바스크립트에서는 엄청나게 혼란스럽고 헷갈리는 부분이 많지만 사실 잘 쓰면 뭐든지 아트라고 생각하기 때문에 큰 문제는 없을 것 같았다. 하지만 실제 걱정되는 부분은 `process.nextTick()` 에 있었다.

`process.nextTick()` 는 node.js 의 비동기 API 에 속해있지만 일반적으로 비동기 API 하면 생각나는 이벤트 루프의 일부분이 아니다. `process.nextTick()` 은 이벤트 루프가 지금 뭔 작업을 처리하던지와 관계 없이 현재 작업이 완료되고 즉시 처리된다. 이 말을 난 이벤트 루프에 지속적으로 `process.nextTick()` 의 작업이 끼어든다는 것으로 이해했고, 그래서 이를 재귀적으로 호출하면 이벤트 루프가 `poll` 단계에 다다르는 것을 막아 I/O Starvation 을 야기할 수도 있다는 [Node.js 공식 문서](https://nodejs.org/ko/docs/guides/event-loop-timers-and-nexttick/)의 말을 보고서는 조금 더 잘 알아보고 사용해야 겠다는 생각에 이 구조를 갈아 엎었다. (사실 위 코드는 엄밀히 따지면 재귀는 아니긴 하지만 어쨌든 반복문을 통해 무분별하게 `process.nextTick()` 을 사용하고 있는 것은 사실이니...)

<br/>

# - model

사실 "함수의 재활용성을 높히기 위해서 함수를 모듈화하여 파일들을 다 쪼개놓았습니다!" 는 그다지 우아하지 않다. 파일이 무분별하게 많아지면 그 것도 그거 나름대로 가독성도 떨어지고 코드를 쓸 때 불편한 부분도 없지 않아 있어 누군가 디자인 패턴을 가르칠 때도 무분별하게 메서드를 분리하는 것은 좋지 않다고 했었다. 그래도 "데이터베이스 CRUD"의 경우 controller 의 여러 부분에서 같은 형태의 작업을 하는 경우가 많기 때문에 `Model` 이라는 거창한 이름을 달아 분리했구나 싶다. (근데 왜 `Model` 이지..)

이 프로젝트에서 이를 적용하였고 코드는 아래와 같다.

`models/authModels.js`

```js
let Model = function () {
  const _mysql = require("config/mysqldb");

  this.getUser = (id, done) => {
    _mysql((conn) => {
      conn.query(
        "SELECT * FROM table WHERE userId=?",
        [id],
        (error, result) => {
          if (error) done(error, null);
          else done(null, result);
        }
      );
      conn.release();
    });
  };
};

module.exports = function () {
  return new Model();
};
```

'왜 굳이 `module.exports` 함수에 또 생성자 함수를 생성해서 반환하나요?' -> 그냥요. 이렇게도 해보고 싶었어요.

`controller/auth/login.js`

```js
/* IMPORTS */
const authModel = require("../../models/authModels")();

/* METHODS */
function getUserById(userId) {
  return Promise((resolve, reject) => {
    authModel.getUser(userId, (error, result) => {
      if (error) reject(error);
      else resolve(result);
    });
  });
}

function validatePassword(password, context) {
  return Promise((resolve, reject) => {
    // 대충 복호화한 db.password랑 입력 password랑 비교한다는 코드
  });
}

// ... 이하 생략
```

예. 지금까지 코드를 나누는 얘기만 계속 해서 더 쓸 말이 없다...

<br/>

# 1번 일기 마무리

1번 일기는 프로젝트 구조를 어떻게 구축했는지, 코드의 큰 뼈대는 어떻게 세웠는지 기록해보았다. 다음 글부터 기록에 남기고 싶은 것들을 좀 자세하게 적을 건데 리스트업 하자면 아래와 같이 할라고 한다.

- Socket
- Mongo 와 mongoose
- JWT 를 조금만 더 잘 써보자.
- Docker 로 배포를 날로 먹자.

<br/>

작성자의 글은 똥입니다. 똥을 된장으로 만들어 주세요. 지적 구걸합니다. `hyunwoo045@gmail.com`. 감사합니다.
