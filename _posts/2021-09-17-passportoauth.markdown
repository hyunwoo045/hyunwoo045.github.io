---
title: "Passport로 소셜 로그인 구현하기"
excerpt: "구글, 카카오 등의 로그인 인증 시스템을 내 어플리케이션에 넣어봅시다!"

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - express
  - passport

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

### 설치

- passport 기본 패키지
- google, kakao Strategy 를 사용하기 위한 추가 패키지들 (`passport-google-oauth2`, `passport-kakao`)
- express 에서 session 기능을 사용하기 위한 패키지 (`express-session`)
- session 정보를 저장하기 위한 storage로 MySQL를 사용. 이를 위한 패키지 (`express-mysql-session`)

```
$ npm install passport passport-google-oauth2 passport-kakao express-session express-mysql-session
```

설치가 끝나면 `./app.js`에 기본적인 설정을 해보도록 하겠습니다.

설치한 모듈들을 가져옵니다.

```javascript
const passport = require("passport");
const GoogleStrategy = require("passport-google-oauth2").Strategy;
const KakaoStrategy = require("passport-kakao").Strategy;
```

passport 는 Strategy 라는 것을 사용합니다. 아래는 GoogleStrategy 를 작성한 것입니다. passport 에서 LocalStrategy 이외에 구글과 같은 다른 사이트의 로그인 기능을 사용하려면 `CLIENT_ID` 와 같은 인증 키를 반드시 받아야 합니다.

[Google Cloud](https://cloud.google.com/developers) <br/>
[Kakao Developers](https://developers.kakao.com/)

위 사이트들에서 키를 발급 받은 후에 Strategy 를 작성합니다.

```javascript
passport.use(
  new GoogleStrategy(
    {
      clientID: GOOGLE_CLIENT_ID,
      clientSecret: GOOGLE_CLIENT_SECRET,
      callbackURL: "http://localhost:3000/api/auth_social/google/callback",
    },
    async (token, tokenSecret, profile, done) => {
      return done(null, profile);
    }
  )
);
```

`cilentID` 등에 대한 인증이 끝나면 콜백함수가 실행이 되고, 인자로 `token, tokenSecret, profile, done`을 전달 받습니다. 그 중 `profile`에 필요한 유저의 정보가 들어있습니다. 대략적으로 아래와 같은 정보들이 들어있습니다.

```javascript
{
  provider: 'google',
  sub: '....',
  id: '....',
  displayName: '김현우',
  name: { givenName: '현우', familyName: '김' },
  language: 'ko',
  photos: '...' // .....
  // (....)
}
// 정말 있을 건 다 있군요!
```

이 정보를 바탕으로 아래와 같은 작업을 처리 할 수도 있습니다. 게시판에 유저에 대한 고유 식별번호를 생성해야 하는데 `provider`, `id`의 복합키로 생성해 보도록 하겠습니다.

```javascript
passport.use(
  new GoogleStrategy(
    {
      clientID: GOOGLE_CLIENT_ID,
      clientSecret: GOOGLE_CLIENT_SECRET,
      callbackURL: GOOGLE_CALLBACK_URL,
    },
    async (token, tokenSecret, profile, done) => {
      console.log("GOOGLE LOGIN!", profile);
      try {
        /* 
          1) provider (google || kakao), id 를 user 테이블에서 검색
          1-1) 없다면 (= 처음 로그인), provider, identifier, displayName 으로 새로운 record 생성
        */
        let id = await User.find(profile.provider, profile.id);
        if (!id) {
          await User.create(profile.provider, profile.id, profile.displayName);
          id = await User.find(profile.provider, profile.id);
        }

        // 필요한 정보만 세션에 저장하도록 한다. { db상 user_id, 받은 id, provider, displayName }
        return done(null, {
          id,
          identifier: profile.id,
          provider: profile.provider,
          displayName: `G-${profile.displayName}`,
        });
      } catch (err) {
        return done(err, null);
      }
    }
  )
);
```

마지막에 콜백함수의 네 번째 인자인 done 함수를 반환하였습니다. done 메서드는 기본적으로 2개의 인자를 담습니다.<br/>
`done(err, user)`<br/>
첫 번째 인자는 err 를 담아주고, 두 번째 인자에 유저 정보를 담아줍니다. err의 경우 보통 localStrategy 를 사용할 때 잘못된 로그인 정보를 입력받았을 경우의 error 핸들링을 하는 부분이라고 생각하면 됩니다. 저는 외부 사이트의 로그인 strategy 를 가져왔기 때문에 특별한 경우가 아니라면 error 에 대한 로직을 구현할 필요는 없었습니다. 따라서 `null` 로 할당해주었고, profile 의 내용을 모두 담기엔 제 사이트에서 사용하지 않는 불필요한 정보가 많이 있고 세션에 부담이 될 수 있으므로 필요한 정보만 객체에 담아 두 번째 인자로 할당해주었습니다.

다음으로 `serializeUser()`가 실행됩니다. `serializeUser()`와 `deserializeUser()`를 함께 살펴보게 되는데, 이 두 함수는 모두 세션에 관련한 기능을 수행합니다.

```javascript
passport.serializeUser((user, done) => {
  done(null, user);
});
passport.deserializeUser((user, done) => {
  done(null, user);
});
```

- `serializeUser`: 최초 로그인 시 (브라우저에 session이 없을 경우) 실행되는 함수입니다. 브라우저의 고유 session id 에 해당하는 session 정보를 서버에 생성하고 전달받은 user 정보를 생성하여 저장합니다.
- `deserializeUser`: 로그인 후 서버에 api 요청이 있을 때 마다 실행되는 함수 입니다. 해당 session id에 저장되어 있는 user 정보를 `req.user` 에 저장합니다.

이 두 함수가 로그인 세션 유지의 핵심적인 기능을 담당하고, 당연하게도 session 을 이용해야 합니다. 이를 활용하기 위해서 `express-session` 을 설치하였습니다. 또한 세션 정보를 MySQL에 저장하기 위해 `express-mysql-session` 또한 설치했습니다. 등록해 주겠습니다.

```javascript
const session = require("express-session");
const MySQLStore = require("express-mysql-session")(session);
// dbconfig: { host, user, password, database } 형태로 mysql server 에 접속하기 위한 설정
const sessionStore = new MySQLStore(dbconfig);
app.use(
  session({
    secret: sessionSecret.secret,
    resave: false,
    saveUninitialized: true,
    store: sessionStore,
  })
);
```

- `resave`: 변경사항이 없는 session도 저장할 것인지에 대한 설정입니다. 작동 효율을 높히기 위해 대부분 false로 설정합니다.
- `saveUninitialized`: 새 request가 들어와 해당 요청에서 새로 생성되어 아무 내용이 없는 세션을 저장할 것인지에 대한 설정입니다.
  - 사용자의 서버 방문 횟수가 높을 수록 좋다면 true로 설정할 수 있습니다.
  - empty session 객체가 서버에 쌓이는 것을 방지하여 스토리지를 아끼고 싶다면 false 로 설정합니다.

이제 준비가 끝났습니다. passport를 app에 등록해 줍니다.

```javascript
app.use(passport.initialize());
app.use(passport.session());
```

<br/>

이제 동작 흐름을 살펴보도록 하겠습니다.

1. Client에서 소셜 로그인 요청을 합니다.

```html
<!-- Vue.js -->
<a href="http://localhost:3000/auth_social/google">구글로 로그인</a>
```

2. 해당 페이지에서 로그인 페이지로 넘겨줍니다.

```javascript
router.get("/auth_social/google", passport.authenticate("google"));
```

3. 로그인이 되면 설정해둔 Strategy 가 동작합니다.

```javascript
passport.use(
  new GoogleStrategy(
    {
      clientID: GOOGLE_CLIENT_ID,
      clientSecret: GOOGLE_CLIENT_SECRET,
      callbackURL: GOOGLE_CALLBACK_URL,
    },
    async (token, tokenSecret, profile, done) => {
      console.log("GOOGLE LOGIN!", profile);
      try {
        let id = await User.find(profile.provider, profile.id);
        if (!id) {
          await User.create(profile.provider, profile.id, profile.displayName);
          id = await User.find(profile.provider, profile.id);
        }
        return done(null, {
          id,
          identifier: profile.id,
          provider: profile.provider,
          displayName: `G-${profile.displayName}`,
        });
      } catch (err) {
        return done(err, null);
      }
    }
  )
);
```

```pm
0|www    | GET /api/auth_social/google 302 1.665 ms - 0
0|www    | GOOGLE LOGIN! {
0|www    |   provider: 'google',
0|www    |   sub: ':D',
0|www    |   id: ':D',
0|www    |   displayName: '김현우',
0|www    |   name: { givenName: '현우', familyName: '김' },
0|www    |   given_name: '현우',
0|www    |   family_name: '김',
0|www    |   language: 'ko',
0|www    |   locale: undefined,
...(아래 생략)
```

로그인에 성공하였습니다!

4. 최초 로그인이므로 `serializeUser` 가 동작합니다.

```javascript
passport.serializeUser((user, done) => {
  console.log("SERIALIZED", user);
  done(null, {
    id: user.id,
    identifier: user.identifier,
    provider: user.provider,
    displayName: user.displayName,
  });
});
```

```pm
0|www    | SERIALIZED {
0|www    |   id: 11,
0|www    |   identifier: ':D',
0|www    |   provider: 'google',
0|www    |   displayName: 'G-김현우'
0|www    | }
```

Strategy 콜백함수에서 필요한 정보만 보내주었던 것이 잘 왔네요. done 함수가 호출되면서 두 번째 인자를 `passport.user` 속성에 알아서 넣어 데이터베이스에 저장할 겁니다.

![Session data on DB](/images/session_table.png) - 데이터베이스에 잘 저장되었네요!

5. 홈 페이지 화면으로 이동하면 `/api/content` 로 api 요청을 보내어 전체 글 목록을 가져오도록 되어 있습니다. `deserializeUser` 함수는 서버에 request가 들어올 때 마다 동작한다고 했었는데 확인 해보도록 하겠습니다.

```javascript
passport.deserializeUser((user, done) => {
  console.log("DESERIALIZED", user);
  done(null, user);
});
```

```pm
0|www    | DESERIALIZED {
0|www    |   id: 11,
0|www    |   identifier: ':D',
0|www    |   provider: 'google',
0|www    |   displayName: 'G-김현우'
0|www    | }
0|www    | GET /api/content 304 33.603 ms - -
```

- `deserializeUser` 함수가 잘 동작한 것을 볼 수 있습니다. `done` 함수가 실행되면 express의 `req`에 user 정보가 담깁니다. 이를 `req.user` 에서 확인할 수 있습니다. 실제로 `deserializeUser` 함수가 실행되고 `/api/content` 페이지로 이동하였고, 아래와 같이 유저 정보를 사용할 수 있습니다.

```javascript
/* 세션이 유효한지 판단하는 기능 */
const authenticateUser = (req, res, next) => {
  // req.user 정보가 있으면 요청에 대한 처리를 함
  // 없다면 세션이 만료되었다는 (혹은 없다는) 메세지를 되돌려 보냄.
  if (req.user) {
    next();
  } else {
    res.send("SESSION_EXPIRED");
  }
};

router.get("/content", authenticateUser, (req, res) => {
  res.send("글 목록!");
});
```

<br/>

지금까지 passport 의 기본 사용법을 기록하였고, 동작 흐름을 실제로 디버깅을 통해 확인해보았습니다. 아래는 클라이언트 측에서의 활용법을 조금 더 작성해보겠습니다.

첫 째로, 로그인 한 유저만 페이지의 정보를 열람할 수 있게 하고 싶습니다. 위에서 이미 서버사이드의 기능을 구현하였습니다. 세션이 유효하지 않다면 `SESSION_EXPIRED` 메시지를 보내도록 하였습니다. 그렇다면 글 목록을 요청하는 클라이언트의 코드에 아주 간단히 SESSION_EXPIRED 를 받았을 때에 대한 처리를 해주면 됩니다.

```javascript
this.$http.get(`${endpoint}/content`).then((response) => {
  if (response.data === "SESSION_EXPIRED") {
    // 세션이 만료되거나, 유효하지 않다면 로그인 페이지로 이동합니다.
    alert("로그인 후 이용해주세요");
    this.$router.push("/login");
  } else {
    this.contents = response.data;
  }
}
```

모든 API 호출마다 이와 같은 로직을 추가해준다면 사용자가 로그인을 하지 않는다면 그 어떤 페이지의 정보도 확인할 수 없게 할 수 있겠습니다.

두 번째로 웹 사이트에서 새로고침 하거나 창을 종료한 후 다시 웹 사이트에 접속하였을 때에 로그인이 유지되면서도 해당 유저의 기본 정보를 다시 받아와 사용할 수 있도록 해보겠습니다. `~/store/user.js` 에는 다음과 같은 정보들을 저장해두고 있습니다.

```
isLoggedin - 현재 유저가 로그인 상태인가 (default: false)
id - DB상의 유저 고유 식별 번호
nickname - 제공받은 displayName
(...)
```

고유 번호와 닉네임의 경우 각 글 수정/삭제 권한을 부여 받을 수 있도록하고, 본인의 닉네임이 화면에 나타나도록 하게 하기 위해 필요한 정보들이므로, 최초 사이트 접속 시 서버로부터 전달받아야 하는 정보입니다. 창 종료 후 다시 접속하였을 때에 `isLoggedin`은 항상 `false`인 특성을 활용합니다.

`vue-router` 의 `beforeEnter` 기능을 활용합니다. 이 기능이 설정된 페이지는 접속하기 전 beforeEnter 메서드가 동작하여 분기마다 어떤 처리를 할 지 결정할 수 있습니다.

```javascript
// (...import)

export default createRouter({
  routes: [
    path: "/",
    component: Home,
    name: "Home",
    beforeEnter: (to, from, next) => {
      if (!store.state.user.isLoggedIn) {
        axios.get(`${endpoint}/auth/session`).then(res => {
          if (res.data === "SESSION_EXPIRED") {
            next("/login")
          } else {
            const payload = res.data;
            store.commit('user/setState', payload);
            next();
          }
        })
      } else {
        next();
      }
    }
  ]
})
```

로그인이 안되어 있다는 플래그를 가지고 있을 때에 세션을 확인하여 유저 정보를 받아오는 구성입니다. 서버에 Request가 있을 때 마다 `deserializeUser`가 동작하고 유효한 세션은 항상 `req.user` 에 정보를 담아주는 특성을 이용하여 간단히 클라이언트 페이지로 유저 정보를 넘겨줄 수 있습니다. 서버의 코드는 아래와 같습니다.

```javascript
router.get("/auth/session", (req, res) => {
  // deserializeUser 가 실행되고 req에 user 정보가 담겨있다면 그 정보를 보내주고
  // 아니라면 세션이 만료되었다는 메시지를 전달한다.
  if (req.user) {
    const { id, identifier, provider, displayName } = req.user;
    res.send({ id, identifier, provider, displayName });
  } else {
    console.log("SESSION_EXPIRED");
    res.send("SESSION_EXPIRED");
  }
});
```
