---
title: "React 시작하기"
excerpt: "React 에 대해 기초부터 간단한 프로젝트까지 진행해보겠습니다."

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

- ### React 시작하기
- ### [React Router](https://hyunwoo045.github.io/react/2021/10/12/react-router.html)

---

<br/>

# React

공식 홈페이지에서 이를 소개하기를 `사용자 인터페이스를 만들기 위한 JavaScript 라이브러리` 라고 소개하고 있습니다. 사용자에게 보여지는 View 그 자체와 그 안에서 동적으로 변화하는 상태 데이터들을 쉽게 관리하기 위한 라이브러리 라고 생각하시면 되겠습니다.

<br/>

## Component

다른 언어에서 여러 정의로 컴포넌트를 정의합니다만, React 에서는 간단하게 아래와 같이 정의할 수 있을 것 같습니다.

<h3> ` 문서(HTML), 스타일(CSS), 동작(JS)를 합쳐서 자신이 만든 일종의 태그 ` </h3>

각각의 컴포넌트는 각 컴포넌트를 정의하는 개별적인 문서와 스타일, 동작을 정의하고 있고, 각 컴포넌트는 개별적으로 동작합니다.

<br/>

## React CSR

Client Side Rendering

1. 서버에서 Browser 로 Response 를 보냄
2. Browser 가 JS 를 다운로드 받음
3. Browser 가 React 를 실행함
4. 페이지가 보여지고 상호 작용이 가능하게 됨

## React SSR

Server Side Rendering

1. 이미 만들어진 뷰를 내려받음
2. Browser 가 JS 를 다운로드 받음 (이 때, 이미 사용자는 화면을 볼 수 있음. 브라우저가 1. 에서 다운받은 뷰를 랜더링 하고 있기 때문)
3. Browser 가 React 를 실행함
4. 페이지가 상호 작용이 가능하게 됨.

- CSR
  - JS 가 전부 다운로드 되어 리액트 앱이 정상 실행되기 전까지는 화면이 보이지 않음
- SSR
  - JS 가 다 다운되지 않아도 화면은 보이지만 유저가 사용할 수 없음.
  - JS 가 다운로드 되어 리액트앱이 정상 실행된 후 유저가 사용이 가능함.

<br/>

## React 가 하는 일

핵심 모듈 2개가 있습니다.

- import ReactDOM from 'react-dom'
  - '만들어진 React Component' 를 실제 HTML Element 에 연결할 때 ReactDOM 라이브러리를 사용.
- import React from 'react'
  - 기타 React Component 를 생성하고 내부의 상태 제어하는 등의 메서드들을 정의하고 있는 모듈.

<br/>

## Use React, ReactDOM Library with CDN

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Hello React!</title>
  </head>
  <body>
    <script
      crossorigin
      src="https://unpkg.com/react@17/umd/react.development.js"
    ></script>
    <script
      crossorigin
      src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"
    ></script>
  </body>
</html>
```

<br/>

## ReactDOM.render

```javascript
ReactDOM.render(el or component, parent element)
```

## React.createElement

```javascript
React.createElement(React Component, props, innerHTML)
```

<br/>

## React Component 만드는 법

```javascript
// 정의 (class component)
import React from "react";

class ClassComponent extends React.Component {
  render() {
    return <div>Hello</div>;
  }
}

// 정의 (function component)
// 정말 특별해 보일 것 없지만 진짜 이게 전부다.
function FunctionComponent() {
  return <div>Hello</div>;
}

// 정의 2 (function component)
// 정말 특별해 보일 것 없지만 진짜 이게 전부다.
const FunctionComponentArrow = () => {
  return <div>Hello</div>;
};
```

<br/>

## React.createElement 로 컴포넌트 만들기

```javascript
React.createElement(
  type, // 태그 이름 문자열 | 리액트 컴포넌트 | React.Fragment
  [props], // 리액트 컴포넌트에 넣어주는 데이터 객체
  [...children] // 자식으로 넣어주는 요소들
);

// 1. 태그 이름 문자열 type
ReactDOM.render(
  React.createElement("h1", null, `예제 1번`),
  document.querySelector("#root")
);

// 2. React Component type
const Component = () => {
  return React.createElement("p", null, `예제 2번`);
};
ReactDOM.render(
  React.createElement(Component, null, null),
  document.querySelector("#root")
);

// 3. React.Fragment
// 태그 부분 없이 그냥 3번째 인자인 innerHTML 이 나옴
ReactDOM.render(
  React.createElement(React.Fragment, null, `예제 3번`),
  document.querySelector("#root")
);
```

<br/>

## JSX

React Element 를 생성하는 문법입니다. jsx 로 작성된 파일은 webpack 등으로 별도의 번역 작업을 거쳐 browser 가 인식 가능한 언어로 변환됩니다. 이 과정에서 느슨한 문법을 적용하게 되면 번역이 어려우니 엄격한 문법을 적용하게 됩니다.

왜 jsx 을 사용할까요?

1. 가독성
2. 문법 오류 인지하기 쉬움

아래서부터는 jsx 의 문법입니다.

1. 최상위 요소가 하나여야 한다.
2. 최상위 요소를 리턴하는 경우, () 로 감싸야 한다.
3. 자식들을 바로 랜더링하고 싶으면 <>자식들</> 를 사용한다.

```js
// 1번 문법을 어기지 않으면서 여러 최상위 요소를 랜더링 하고 싶으면 아래와 같이 하자
React.DOM.render(
  <>
    <div id="first-child"> 첫째야! </div>
    <div id="second-child"> 둘쨰야! </div>
  </>
);
```

4. 자바스크립트 표현식을 사용하려면, {표현식} 을 사용한다.

```js
// 변수의 값을 출력하고 싶어서 자바스크립트 표현식을 활용한 예제
const title = "제목 :D";
ReactDOM.render(
  <div id="root">
    <div>
      <h1>{title}</h1>
    </div>
  </div>
);
```

5. if 문을 사용할 수 없다.

- 삼항 연산자 혹은 && 를 사용한다.

6. `style` 을 이용하여 인라인 스타일링이 가능하다
7. class 대신 className 을 사용해 class 를 적용할 수 있다.
8. 자식요소가 있으면, 꼭 닫아야 하고, 자식요소가 없으면 열면서 닫아야 한다.

```html
<p>에에에~</p>
<br />
```

<br/>

## Props 와 State

- Props : 컴포넌트 외부에서 컴포넌트에게 주는 데이터.
- State : 컴포넌트 내부에서 변경할 수 있는 데이터.

둘 다 변경이 발생하면 다시 `render` 될 수 있다.

- Render 함수 : Props 와 State 를 바탕으로 컴포넌트를 그리는 함수.
  - Props와 State가 변경되면 다시 호출된다.
  - 컴포넌트를 그리는 방법을 기술하는 함수이다.

<br/>

## Component Lifecycle (16.3 버전 이전)

리액트 컴포넌트의 생성부터 삭제까지 여러 지점에서 개발자가 작업이 가능하도록 메서드를 오버라이딩 할 수 있게 해줍니다.

Initialization

- setup props and state

Mounting

- ComponentWillMount
- render
- ComponentDidMount

Updation

- props, state 의 변경
- componentWillReceiveProps (여기서는 state 변경에 반응하지 않음)
- shouldComponentUpdate (return 값을 true로 하면 re-render, false 하면 state 변경 후 아무것도 하지 않는다.)
- componentWillUpdate
- render
- componentDidUpdate

Unmounting

- componentWillUnmount

<br/>

## Component Lifecycle (16.3 버전 이후)

| 변경 전                   | 변경 후                  |
| ------------------------- | ------------------------ |
| componentWillMount        | getDerivedStateFromProps |
| componentWillReceiveProps | getDerivedStateFromProps |
| componentWillUpdate       | getSnapshotBeforeUpdate  |

<br/>

## Create React App

```
$ npx create-react-app [project name]
```

<br/>

## ESLint

```
$ npm install eslint -D
$ npx eslint --init

# 이 후 eslint 의 설정
1. 어떻게 ESLint 를 사용할 것인가? (문법 체크? 문법 체크와 문제 찾기? code style 강제까지 모두)
2. 어떤 type 의 모듈?
3. framework? (react? ...)
4. typescript 사용 여부 (yes or no)
5. 어떤 런타임에서 사용? (node, browser)
6. config 파일의 타입? (js, yaml, json)
```

```js
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: "eslint:recommended",
  parserOptions: {
    ecmaVersion: 13,
    sourceType: "module",
  },
  rules: {},
};
```

설정을 마치고 나면 위와 같은 파일이 자동으로 생성되고, 룰을 지정할 수 있는 곳이 나타난다. eslint 홈페이지에 가면 rules 를 잘 정의해두었으니 참고하도록 합시다.

링크: [ESLint 공식홈페이지](https://eslint.org/)

create react app 으로 프로젝트를 생성했다면 eslint 가 알아서 설치되어 있는 것을 알 수 있고, `package.json` 에 아래와 같은 항목을 볼 수 있습니다.

```
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
```

바로 위에 `extends: "eslint:recommended"` 와 같은 속성인 `extends` 가 있는 것을 볼 수 있습니다. 즉, 우리는 `package.json` 을 수정하여 이전에 .eslintrc.js 을 수정하여 만든 룰을 적용할 수 있다는 것을 알 수 있습니다.

```json
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ],
    "rules": {
      "semi": ["error", "always"]
    }
  },
```

<br/>

## Prettier

코드를 이쁘게 만들어주는 모듈입니다. extensions 에서 따로 설치할 수 있습니다. 이미 많은 사람들이 이에 대해서 포스팅을 했으니 자세한 내용을 다루진 않겠습니다만, `create-react-app` 으로 프로젝트를 생성한 경우 주의해야 하는 부분이 있습니다.

`create-react-app` 으로 프로젝트를 생성한 경우 eslint 가 자동으로 설치되어 있습니다. 이에 prettier 까지 설치하는 경우, eslint 와 prettier 가 충돌할 수 있습니다. 아래와 같이 설정하여 충돌을 방지할 수 있겠습니다.

```json
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest",
      "prettier"
    ]
  },
```

<br/>

## Husky

프로젝트에 git을 사용하는 경우 commit이 되기 직전에 실행할 명령어들을 정의해 줄 수 있게 해주는 모듈입니다.

```
$ npm install husky -D
```

husky에서 git hook을 사용할 수 있게 해주려면

```
$ npx husky install
```

```json
"scripts": {
  "prepare": "husky install",
  "test": "echo \"Error: no test specified\"...
}
```

<br/>

## lint-staged

eslint, prettier 와 husky 를 연결해서 lint-staged 까지 사용하면 커밋을 하기 직전에 staged 된 파일들을 포맷을 점검하고, 문법을 확인하고 고친 후에 커밋을 할 수 있게 됩니다. 간단히 사용하는 방법을 알아봅니다.

```
$ npm install husky
$ npx husky install
```

`package.json` 에 "script" 속성에 prepare 항목을 추가합니다.

```json
  "scripts": {
    "prepare": "husky install",
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
```

`lint-staged` 를 설치.

```
$ npm install lint-staged -D
$ npx husky add .husky/pre-commit "lint-staged"
```

"lint-staged" 라는 명령은 아직 설정하지 않았음. `packaged.json` 에 아래와 같은 속성을 추가한다.

```json
  "lint-staged": {
    "**/*.js": [
      "eslint --fix",
      "prettier --write",
      "git add ."
    ]
  },
```

위 코드는 아래와 같은 말을 합니다.

1. eslint 로 fix 하고
2. prettier 로 포매팅 된 것을 파일에 write 한 후
3. git 에 모두 다시 add 한다.

.husky/pre-commit 파일에 맨 아래줄에 lint-staged 라고만 되어 있을 것입니다. 터미널은 'lint-staged' 이 뭔지 모르니 아래와 같이 수정합시다.

```
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

설정을 마쳤습니다. git commit 해보세요!

```
$ git add .
$ git commit -m "test"
```

<br/>

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
