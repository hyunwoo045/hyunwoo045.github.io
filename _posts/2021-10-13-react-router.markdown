---
title: "React Router"
excerpt: "React Router 에 대해 알아보겠습니다."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - React

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

## React 포스트 목록

- ### [React 시작하기](https://hyunwoo045.github.io/react/2021/10/11/react-starts.html)
- ### React Router

---

<br/>

# React Router

기존의 라우팅 방식은 경로가 바뀌면 해당 경로의 HTML 문서와 기타 필요한 resource들을 서버에서 일일이 받아왔습니다.

하지만 React 로 넘어오면서 SPA(Single Page Application) 의 개념이 도입됩니다. 서버에서 한 번에 모든 페이지들을 받아온 후 URL 에 따라서 보여주어야 할 페이지만 보여주는 방식입니다.

---

## SPA 라우팅 과정

1. 브라우저에서 최초에 '/' 경로로 요청을 하면 React Web App 을 내려줌
2. 내려받은 React App 에서 '/' 경로에 맞는 컴포넌트를 보여줌
3. React App 에서 다른 페이지로 이동하는 동작을 수행하면, 해당 경로에 맞는 컴포넌트를 보여줌

이를 도와줄 패키지로 `react-router-dom` 을 사용하겠습니다. create-react-app 에 기본 내장된 패키지가 아니어서 따로 설치가 필요합니다. (Facebook 의 공식 패키지가 아닌 것을 참고합시다.)

```
$ npm install react-router-dom
```

간단하게 3개의 페이지를 생성합니다.

```jsx
// Home.jsx
export default function Home() {
  return <div>Home 페이지입니다.</div>
}

// Profile.jsx
export default function Profile() {
  return <div>Profile 페이지입니다.</div>
}

// About.jsx
export default function About() {
  return <div>About 페이지입니다.</div>
}
```

그리고 `App.js` 파일에서 페이지들을 나열해주고, URL path 에 따라 보여줄 페이지들을 설정합니다. `<BrowserRouter>` 태그 안에 자식 요소로 `<Route>` 태그들을 나열합니다. `<Route>` 태그의 속성으로 `path`, `component` 를 정의하여 path 에 따라 어떤 component 를 랜더링 할 지를 결정할 수 있습니다.

```jsx
import { BrowserRouter, Route } from "react-router-dom";
import Home from "./pages/Home";
import Profile from "./pages/Profile";
import About from "./pages/About";

function App() {
  return (
    <BrowserRouter>
      <Route path="/" exact component={Home}></Route>
      <Route path="/profile" component={Profile}></Route>
      <Route path="/about" component={About}></Route>
    </BrowserRouter>
  );
}

export default App;
```

Home Route 태그의 component 속성 앞에는 `exact` 라는 키워드를 사용하였습니다. 해당 키워드를 사용하지 않고 `/profile` 로 이동해보면, "Home 페이지입니다" 와 "Profile 페이지입니다" 가 함께 나타나는 것을 볼 수 있습니다. 이는 `/profile` 이라는 경로를 `/`, `/profile` 로 인식하여 둘 다 랜더링 하는 것입니다. 따라서 정확히 `/` 이라는 경로를 사용할 때만 Home Component 를 보여주도록 설정할 필요가 있습니다.

<br/>

---

## 다이나믹 라우팅

URL 이 고정적이지 않고 동적으로 바뀌는 경우도 있을 것입니다. `/profile/2` 나 `/profile/react` 등으로 profile path 뒤에 알 수 없는 값이 온다고 가정해보겠습니다. `:` 기호를 앞에 붙히고 변수명을 지정해 준 후에, 해당 페이지에서 이 값을 받아 사용할 수 있겠습니다.

```jsx
import { BrowserRouter, Route } from "react-router-dom";
import Home from "./pages/Home";
import Profile from "./pages/Profile";
import About from "./pages/About";

function App() {
  return (
    <BrowserRouter>
      <Route path="/" exact component={Home}></Route>
      <Route path="/profile" component={Profile}></Route>
      <Route path="/profile/:id" component={Profile}></Route>
      <Route path="/about" component={About}></Route>
    </BrowserRouter>
  );
}

export default App;
```

`Profile.jsx` 파일에서 변수를 받아 쓰기 위해서 `props` 를 활용합니다. 컴포넌트의 인자로 `props` 를 전달하고 출력하게 되면,

![]()

위와 같은 객체가 출력되는 것을 볼 수 있고, 지정했던 `id`라는 값은 `props.match.params` 안에 들어 있는 것을 알 수 있습니다.

```jsx
export default function Profile(props) {
  console.log(props);

  const id = props.match.params.id;
  console.log("id:", id, typeof id);
  return (
    <div>
      <h2>Profile 페이지입니다.</h2>
      <p>id 는 {id} 입니다.</p>
    </div>
  );
}
```

쿼리 스트링을 사용하는 URL (ex. `/about?name=zzanggu`) 에 대해서 처리 하는 방법도 알아 보겠습니다.

두 가지 방식으로 처리할 수 있습니다. Browser 의 내장 메서드인 `URLSearchParams()` 를 사용하는 방법과 `query-string` 이라는 모듈을 설치하여 사용하는 방법입니다.

```jsx
/*
 * URLSearchParams()
 * 단점
 * 1. 메서드를 모두 기억하고 사용해야함.
 * 2. Browser 내장 객체이기 때문에 browser 마다 지원하지 않을 수 있음 (IE 지원안함)
 */

// About.jsx
export default function About(props) {
  const searchParams = props.location.search;
  const obj = new URLSearchParams(searchParams);
  console.log(obj.get("name"));

  return <div>About 페이지 입니다.</div>;
}
```

```jsx
/*
 * query-string 패키지 활용
 */
import queryString from "query-string";
export default function About(props) {
  const searchParams = props.location.search;
  const query = queryString.parse(searchParams);
  return (
    <div>
      <h2>About 페이지 입니다.</h2>
      {query.name && <p>name 은 {query.name} 입니다.</p>}
    </div>
  );
}
```

<br/>

---

## Switch 와 NotFound

- Switch
  - 여러 Route 중 순서대로 먼저 맞는 하나만 보여준다.
  - exact 를 뺄 수 있는 로직을 만들 수 있다.
  - 어떤 path 도 맞는 곳이 없으면 보여지는 컴포넌트를 설정하여 `Not Found` 페이지를 만들 수 있다.

예제 코드를 보겠습니다.

```jsx
import { BrowserRouter, Route, Switch } from "react-router-dom";
import Home from "./pages/Home";
import Profile from "./pages/Profile";
import About from "./pages/About";
import NotFound from "./pages/NotFound";

function App() {
  return (
    <BrowserRouter>
      <Switch>
        <Route path="/profile/:id" component={Profile}></Route>
        <Route path="/profile" component={Profile}></Route>
        <Route path="/about" component={About}></Route>
        <Route path="/" exact component={Home}></Route>
        <Route component={NotFound}></Route>
      </Switch>
    </BrowserRouter>
  );
}

export default App;
```

주의해야 할 점

- depth 가 더 깊은 path 를 먼저 선언한다.
  - `/profile/:id` 를 `/profile` 보다 뒤에 선언하면 `id` 값이 있는 경로더라도 항상 `/profile` 만 보여짐.
- `/` 인 루트 경로 만큼은 `exact` 키워드를 사용해야 한다. (NotFound 페이지를 설정해야 할 경우)
  - `exact` 키워드를 쓰지 않으면 설정하지 않은 path 가 들어왔을 때 (ex. `/aboutme`) `/` 경로의 경우로 인식하여 Home 페이지가 랜더링 된다.
- 가장 마지막에 path 를 지정하지 않은 `<Route>` 태그를 선언하고 NotFound 페이지를 구성한 컴포넌트를 설정해야 한다.

<br/>

---

## JSX 링크로 라우팅

일반적으로 `<a>` 태그를 이용하여 페이지를 이동합니다. `<a>` 태그는 기본적으로 SPA 의 개념과 어울리지 않는 태그입니다. `<a>` 태그로 생성된 링크를 클릭하면 페이지에 필요한 모든 resource 들을 다시 받아오고 페이지가 랜더링 됩니다. React 에 어울리는 Link 생성 방법을 알아보겠습니다.

링크를 나열하는 간단한 컴포넌트를 생성합니다.

```jsx
import { Link } from "react-router-dom";

export default function Links() {
  return (
    <ul>
      <li>
        <Link to="/">Home</Link>
      </li>
      <li>
        <Link to="/profile">Profile</Link>
      </li>
      <li>
        <Link to="/profile/1">Profile/1</Link>
      </li>
      <li>
        <Link to="/about">About</Link>
      </li>
      <li>
        <Link to="/about?name=mark">About?name=mark</Link>
      </li>
    </ul>
  );
}
```

이 컴포넌트를 `App.js` 에 import 합니다.

```jsx
import { BrowserRouter, Route, Switch } from "react-router-dom";
import Home from "./pages/Home";
import Profile from "./pages/Profile";
import About from "./pages/About";
import NotFound from "./pages/NotFound";
import Links from "./components/Links";

function App() {
  return (
    <BrowserRouter>
      <Links></Links>
      <Switch>
        <Route path="/profile/:id" component={Profile}></Route>
        <Route path="/profile" component={Profile}></Route>
        <Route path="/about" component={About}></Route>
        <Route path="/" exact component={Home}></Route>
        <Route component={NotFound}></Route>
      </Switch>
    </BrowserRouter>
  );
}

export default App;
```

<br/>

---

## 네비게이션 링크

```jsx
import { NavLink } from "react-router-dom";

export default function NavLinks() {
  const activeStyle = { color: "green" };
  return (
    <ul>
      <li>
        <NavLink to="/" exact activeStyle={activeStyle}>
          Home
        </NavLink>
      </li>
      <li>
        <NavLink to="/profile" exact activeStyle={activeStyle}>
          Profile
        </NavLink>
      </li>
      <li>
        <NavLink to="/profile/1">Profile/1</NavLink>
      </li>
      <li>
        <NavLink
          to="/about"
          activeStyle={activeStyle}
          isActive={(match, location) => {
            return match !== null && location.search === "";
          }}
        >
          About
        </NavLink>
      </li>
      <li>
        <NavLink
          to="/about?name=mark"
          activeStyle={activeStyle}
          isActive={(match, location) => {
            return match !== null && location.search === "?name=mark";
          }}
        >
          About?name=mark
        </NavLink>
      </li>
    </ul>
  );
}
```

<br/>

---

## JS 로 라우팅

`<Route>` 태그로 만들어진 페이지는 라우팅에 필요한 여러 정보들이 객체의 형태로 props 에 저장되어 있습니다. 그 중 `history` 라고 하는 옵션이 있고, 내부에 페이지 이동에 필요한 여러 메서드들을 정의하고 있습니다. 그 중 `push` 메서드를 이용하여 페이지를 이동하는 법을 알아보겠습니다.

새 컴포넌트를 생성!

```jsx
// src/pages/Login.jsx
export default function Login(props) {
  function login() {
    setTimeout(() => {
      props.history.push("/");
    }, 1000);
  }
  return (
    <div>
      <h2>Login 페이지 입니다.</h2>
      <button onClick={login}>로그인</button>
    </div>
  );
}
```

생성한 버튼을 클릭하면 1초 뒤에 홈 페이지로 이동합니다.

여기서 주의해야 할 점은 위 컴포넌트는 정의된 후 `Route` 태그로 생성된 컴포넌트라는 점입니다. 이 컴포넌트는 props 에 라우팅에 필요한 정보를 객체로 담고 있습니다. 하지만 페이지 구성이 복잡해져서 로그인 버튼 부분 또한 하위 컴포넌트로 제작하였다고 가정해보겠습니다.

```jsx
import LoginButton from "../components/LoginButton";
// src/pages/Login.jsx
export default function Login(props) {
  return (
    <div>
      <h2>Login 페이지 입니다.</h2>
      <LoginButton />
    </div>
  );
}
```

```jsx
// src/components/LoginButton.jsx
export default function LoginButton(props) {
  console.log(props); // => {}
  function login() {
    setTimeout(() => {
      props.history.push("/");
    }, 1000);
  }
  return <button onClick={login}>로그인</button>;
}
```

위 코드는 정상적으로 작동하지 않습니다. `LoginButton` 의 props 를 콘솔에 찍어보세요. 빈 객체가 출력될 것입니다. `Login.jsx` 페이지에서 `LoginButton` 을 호출할 때 아무 props 도 전달하지 않았기 때문이죠.

이를 아래와 같이 해결할 수는 있습니다.

```jsx
import LoginButton from "../components/LoginButton";
// src/pages/Login.jsx
export default function Login(props) {
  return (
    <div>
      <h2>Login 페이지 입니다.</h2>
      <LoginButton {...props} />
    </div>
  );
}
```

`LoginButton` 컴포넌트에 `Login` 페이지가 전달받은 props 를 그대로 전달하는 방법입니다. 하지만 권장되지 않는 방법입니다. 페이지 구성이 아주 복잡해져서 Route Page 의 몇 단계 아래 컴포넌트에서 페이지 이동 로직을 구현해야 한다면 props 가 여러 컴포넌트를 거쳐가야 하고, 사고로 이어질 수 있기 때문이죠. props 를 전달하지 않고 해결하기 위해서는 하위 컴포넌트에서 `react-router-dom` 의 ` withRouter` 를 받아와서 component 전체를 감싸는 방법이 있습니다.

```jsx
/* props 를 전달하지 않습니다 */

import LoginButton from "../components/LoginButton";
// src/pages/Login.jsx
export default function Login(props) {
  return (
    <div>
      <h2>Login 페이지 입니다.</h2>
      <LoginButton />
    </div>
  );
}
```

```jsx
import { withRouter } from "react-router-dom";

export default withRouter(function LoginButton(props) {
  console.log(props);
  function login() {
    setTimeout(() => {
      props.history.push("/");
    }, 1000);
  }
  return <button onClick={login}>로그인하기</button>;
});
```

이로써 위와 같이 저 밑에 있는 컴포넌트도 props 를 전달 받지 않고 history 객체를 사용할 수 있게 됩니다.

<br/>

---

## Redirect

`Redirect` 라고 하는 `Hook` 을 사용하는 방법입니다. `<Route>` 태그에서 지금까지는 설정한 path 로 진입할 경우 보여줄 component 를 지정하는 방식으로 라우팅을 처리했습니다. component 를 직접 지정하지 않고 render 라는 props 를 설정하여 페이지를 이동할 수 있습니다. 아래의 코드는 `/toprofile` 라는 경로로 접근했을 때 `/profile` 페이지로 리다이렉트 해주는 `<Route>` 태그를 추가한 코드입니다.

```jsx
import { BrowserRouter, Route, Switch, Redirect } from "react-router-dom";
import Home from "./pages/Home";
import Profile from "./pages/Profile";

function App() {
  return (
    <BrowserRouter>
      <Switch>
        <Route path="/toprofile" render={() => <Redirect to="/profile" />} />
        <Route path="/profile" component={Profile}></Route>
        <Route path="/" exact component={Home}></Route>
        <Route component={NotFound}></Route>
      </Switch>
    </BrowserRouter>
  );
}
```

render 로 함수를 선언할 수 있기 때문에 다양한 라우팅 로직을 구현할 수 있습니다.

```jsx
import { BroswerRouter, Route, Redirect } from "react-router-dom"
import Home from './pages/Home';
import Login from './pages/Login';

function App() {
  const isLogin = false;
  /*
   * 로그인이 되어 있다면 '/' 으로 리다이렉트, 아니면 Login 컴포넌트를 보여줌.
   */
  return (
    <BrowserRouter>
      <Route path="/login" render={() => (isLogin ? <Redirect to="/" /> : <Login />)}>
      <Route path="/" component={Home} />
    </BrowserRouter>
  )
}
```

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
