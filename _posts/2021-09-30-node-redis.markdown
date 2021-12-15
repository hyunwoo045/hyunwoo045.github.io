---
title: "Node + RedisDB"
excerpt: "대표적인 NoSQL Database 인 Redis 에 대해서 정리합니다."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Redis
  - Node.js

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# Node + Redis

Redis 는 NoSQL 데이터베이스 중 하나이며, 인기 있는 NoSQL 데이터베이스 입니다.

게시판 만들기 장난감 프로젝트를 할 때, 로그인 유지를 위해 세션을 추가한 적이 있었습니다. 그 때 당시 `express-session` 을 사용하였었고, 데이터를 유지하기 위한 Store로 MariaDB(AWS RDS) 를 사용했었습니다. 세션에 JWT 토큰과 같은 간단한 문자열 데이터만 저장한다면 그다지 효율적인 방법은 아니었던 것 같습니다. 데이터베이스는 커넥션을 유지하는데에 비용이 많이 들기 때문이죠.

세션 혹은 간단한 key-value 의 자료나, list, set, hash 등의 간단한 자료 구조 데이터를 유지하면서도 속도를 빠르게 사용하기 위해 생각할 수 있는 절충안이 `Redis` 가 되겠습니다. 데이터베이스 이기 때문에 데이터의 유지가 가능하고, Memory 기반이기 때문에 속도도 아주 빠릅니다.

하지만 데이터가 항상 정확하게 유지되지 않을 수 있다는 점을 체크해야 합니다. 따라서 <s>(안 날아가는게 제일 좋겠지만...)</s> 날아가도 크게 문제없는 데이터를 저장하는 것이 좋습니다. JWT 토큰과 같은, 날아가도 사용자에게 다시 로그인을 요청하면 그만인 데이터를 저장할 수 있을 것 같습니다.

<br/>

## 설치

Mac OS 에서는 아주 간단하게 설치가 가능합니다.

```
$ brew install redis
```

아래 명령어들로 Redis 를 시작/정지/재시작 할 수 있습니다.

```
$ brew services start redis
$ brew services stop redis
$ brew services restart redis
```

아래 명령어로 CLI를 사용할 수 있습니다

```
$ redis-cli
```

데이터베이스는 항상 CRUD 가 핵심이죠. 간단하게 CLI 에서 사용하는 명령어들을 표로 정리합니다.

| 명령어 | 동작           | 포맷                       | 예시                 |
| ------ | -------------- | -------------------------- | -------------------- |
| set    | key-value 저장 | set [KEY] [VALUE]          | set NAME HYUNWOO_KIM |
| get    | 키 리스트 조회 | keys [패턴]                | keys NA\*            |
| del    | key-value 삭제 | del [KEY]                  | del NAME             |
| rename | 키 이름 변경   | rename [OLD_KEY] [NEW_KEY] | rename NAME MYNAME   |

string 데이터만 다루었습니다만, 다양한 데이터 타입도 지원합니다.

<br/>

## Redis with Node.js

Node.js 에서 Redis 를 사용하는 법을 정리합니다.

- 설치 : `npm install redis`

```javascript
const redis = require("redis");
const client = redis.createClient({
  host: "127.0.0.1",
  port: 6379,
});
```

아래는 위에서 정리한 CRUD 표를 코드화 한 것입니다.

```javascript
client.set("NAME", "HYUNWOO_KIM"); // key-value 저장

client.get("NAME", (err, result) => {
  // 키 'NAME'의 값 조회
  console.log(result);
});

client.del("NAME"); // key-value 삭제

client.rename("NAME", "MYNAME"); // 키 이름 변경
```

아래서부터는 다양한 데이터 타입을 Redis 에 저장하는 방법을 기록합니다.

- hash - JS의 Object를 저장.

```javascript
client.hmset("hyunwoo", "name", "김현우", "age", 18);
client.hgetall("hyunwoo", (err, obj) => {
  console.log(obj); // { name: '김현우', age: 18 }
});
```

- list - 배열

```javascript
client.rpush("fruits", "apple", "banana", "cherry"); // Array.push 와 비슷
client.lpush("fruits", "peach", "mango"); // Array.unshift 와 비슷

// client.lrange(KEY, startIndex, lastIndex (-1 은 마지막 인덱스), cb(err, result) )
client.lrange("fruits", 0, -1, (err, list) => {
  console.log(list); // ['mango', 'peach', 'apple', 'banana', 'cherry']
});
```

- set - 집합

```javascript
client.sadd("names", "철수", "영희", "철수", "짱구", "훈이");
client.smembers("names", (err, set) => {
  console.log(set); // ['철수', '영희', '짱구', '훈이']
});
```

- sorted set

```javascript
// 정렬된 집합
client.zadd("scores", 100, "현우", 30, "짱구", 85, "맹구", 90, "철수");

// client.zrange(KEY, startIndex, lastIndex (-1 은 마지막 인덱스), cb(err, result) )
client.zrange("scores", 0, -1, (err, sorted) => {
  console.log(sorted); // ["짱구", "맹구", "철수", "현우"]
});
```

<br/>

## Express-session 과 연동

```
$ npm install express-session
$ npm install connect-redis
```

```javascript
const redis = require("redis");
const redisClient = redis.createClient({
  host: "127.0.0.1",
  port: 6379,
});

const session = require("express-session");
const connectRedis = require("connect-redis");
const RedisStore = connectRedis(session);
app.use(
  session({
    secret: "sessionSecret!",
    resave: false,
    saveUninitialized: true,
    /* 
      위 영역은 express-session 의 기본 설정이며
      store 에 사용할 저장 방식, 이번 포스트에서는 Redis 를 사용하겠다는 의미 입니다.
      client 는 redis 패키지로 생성한 client 로 지정해야 합니다.
    */
    store: new RedisStore({ client: redisClient, logErrors: true }),
  })
);
```

활성화한 서버로 접속해보세요. `localhost:3000` 으로 접속해보겠습니다.

```javascript
var express = require("express");
var router = express.Router();

router.get("/", function (req, res, next) {
  res.send("WOW");
});

module.exports = router;
```

그리고 Redis CLI 로 접속하여 `keys '*'` 명령을 입력해봅니다.

```
127.0.0.1:6379> keys '*'
1) "sess:lZSR-hYI3Sy0BB5gtr0BOpUuSXGIoIOu"
```

세션이 생성되어 저장되어 있음을 알 수 있습니다. `get` 명령을 통해 내용을 확인할 수 있습니다.

```
127.0.0.1:6379> get sess:lZSR-hYI3Sy0BB5gtr0BOpUuSXGIoIOu
"{\"cookie\":{\"originalMaxAge\":null,\"expires\":null,\"httpOnly\":true,\"path\":\"/\"}}"
```

이를 바탕으로 아래와 같은 미들웨어 함수를 작성할 수 있을 것 같습니다.

```javascript
const getSession = (req, res, next) => {
  redisClient.get(`sess:${req.session.id}`, (err, data) => {
    if (err) throw err;
    if (data) {
      // data handling...
      res.send("session ok");
    } else {
      next();
    }
  });
};
```
