---
title: "React Styled Component"
excerpt: "React 에서 scoped css 를 구현하기 위한 방법을 살펴봅니다."

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

## React Shadow

CSS 파일을 사용하다보면 중복된 className 을 사용한 요소에 원치 않은 스타일이 적용되는 경우가 더러 있습니다. Component 별로 scope style 을 적용하는 방법을 알아보겠습니다. 이를 구현하기 위해 `React Shadow` 를 사용하겠습니다.

- 설치

```
$ npm i react-shadow
```

scoped style 을 적용하고 싶은 component 를 `<root.div>` 태그로 감쌉니다. 감싸진 부분은 외부와 연결이 끊어진 독립된 공간입니다.

```jsx
import root from "react-shadow";

function ReactShadow() {
  return (
    <root.div>
      <div class="hi">
        <button>Hello</button>
      </div>
    </root.div>
  );
}
```

스타일을 적용하는 방법입니다.

```jsx
import root from "react-shadow";

const style = `
  button {
    width: 100px;
    height: 80px;
    background-color: magenta;
  }
`;
function ReactShadow() {
  return (
    <root.div>
      <div class="hi">
        <button>Hello</button>
      </div>
      <style type="text/css">{style}</style>
    </root.div>
  );
}
```

단점

1. 전역 공간에서 사용하는 스타일은 계속 반복적으로 넣어줘야 한다.
2. 외부와 내부가 차단되어 있기 때문에 외부에서 가져올 값을 가져오는 데에 제약이 있다.

<br/>

---

## Ant Design

```
$ npm i antd
```

`index.js` 파일에 antd 의 css 파일을 import 하여 전역에 스타일이 적용되도록 해야 한다.

```js
import React from "react";
import ReactDOM from "react-dom";
import "antd/dist/antd.css"; // <- 추가!
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById("root")
);
reportWebVitals();
```

[Ant Design 공식 홈페이지](https://ant.design/) 에서 Components 에 맘에 드는 것을 가져와 보겠습니다.

```jsx
import { Calendar } from "antd";

function onPanelChange(value, mode) {
  console.log(value.format("YYYY-MM-DD"), mode);
}

ReactDOM.render(<Calendar onPanelChange={onPanelChange} />, mountNode);
```

홈페이지에서 제공하는 코드입니다. 일단 단순히 Calander 만 보이도록 수정해보겠습니다.

```jsx
import { Calendar } from "antd";

function App() {
  return <Calendar />;
}

export default App;
```

커스터마이징을 할 수 있으니 무엇인가 필요할 때마다 잘 찾아서 사용해보도록 합시다.

<br/>

---

## Ant Design Icon

[Ant Design icon 공식 홈페이지](https://ant.design/components/icon/)

```
$ npm install --save @ant-design/icons
```

홈페이지에서 마음에 드는 아이콘을 하나 가져와봅시다.

```jsx
import { Calendar } from "antd";
import { GithubOutlined } from "ant-design/icons";

function App() {
  return (
    <>
      <Calendar />
      <GithubOutlined />
    </>
  );
}

export default App;
```

Ant Design 을 사용하는 법을 살펴보았는데요, 이러한 외부 컴포넌트 라이브러리를 사용할 때에는 `props` 를 잘 파악하는 것이 중요합니다.

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
