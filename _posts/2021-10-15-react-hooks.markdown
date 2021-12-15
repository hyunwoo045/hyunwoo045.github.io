---
title: "React - Hooks"
excerpt: "함수형 컴포넌트를 강력하게 사용할 수 있도록 도와주는 Hooks 에 대해 공부해봅니다."

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

## useState

기존에 state 를 사용하기 위해서는 class component 를 이용하여 `this.state` 와 같은 방식으로 사용했었습니다. 간단한 예제 코드 보시죠.

```jsx
import React from "react";

export default class Example extends React.Component {
  state = { count: 0 };

  render() {
    const { count } = this.state;

    return (
      <div>
        <p>You Clicked {count} times</p>
        <button onClick={this.click}>Click Me!</button>
      </div>
    );
  }

  click = () => {
    this.setState({ count: this.state.count + 1 });
  };
}
```

React 에 Hooks 라는 개념이 도입되면서 `functional component` 에서도 이 state 를 사용할 수 있게 되었습니다. 위 코드를 아래와 같이 함수형으로 쓸 수 있겠습니다.

```jsx
import React from "react";

export default function Example2() {
  // [사용할 state명, state를 수정할 function] = React.useState(초기값)
  const [count, setCount] = React.useState(0);

  return (
    <div>
      <p>You Clicked {count} times</p>
      <button onClick={click}>Click Me!</button>
    </div>
  );

  function click() {
    setCount(count + 1);
  }
}
```

배열 형태의 초기화에서 두번째 index 로 들어간 `setCount` 와 같은 함수는 state 를 변경할 뿐만 아니라 수정된 state 를 반영한 component 를 다시 랜더링 해줍니다.

state 의 초기값을 객체로 만들 수도 있겠습니다. 아래와 같이요.

```jsx
import React from "react";

export default function Example3() {
  const [state, setState] = React.useState({
    count: 0,
  });

  return (
    <div>
      <p>You Clicked {state.count} times</p>
      <button onClick={click}>Click Me!</button>
    </div>
  );

  function click() {
    setState({
      count: state.count + 1,
    });
  }
}
```

위 코드의 `setState` 함수는 약간 Component 의존적입니다. 아래와 같이 `setState` 내부에서 또 다른 함수를 실행하는 방식으로 Component 에 의존적이지 않고 외부에서 다시 사용할 수 있는 방식으로도 사용이 가능하겠습니다.

```jsx
import React from "react";

export default function Example3() {
  const [state, setState] = React.useState({
    count: 0,
  });

  return (
    <div>
      <p>You Clicked {state.count} times</p>
      <button onClick={click}>Click Me!</button>
    </div>
  );

  function click() {
    // 기존에 setState 가 Example3 에 의존적이었다면,
    // 지금은 setState 의 내부 함수가 state 를 인자로 받아서 이를 수정하고 return 하기 때문에 덜 의존적이게 되었다.
    setState((state) => {
      return {
        count: state.count + 1,
      };
    });
  }
}
```

이와 같이 `useState` 를 이용하여 class component 의 state 를 대체할 수 있겠습니다.

<br/>

---

## Class Component vs Functional Component

기존에 Class Component 의 방식으로 state 를 관리하는 방식이 있었음에도 불구하고 Hooks 를 개발하여 Functional Component 를 사용하도록 한 이유가 무엇일까요?

1. 컴포넌트 사이에서 상태와 관련된 로직을 재사용하기 어렵다.
2. 복잡한 컴포넌트들은 이해하기 어렵다
3. Class 는 사람과 기계를 혼동시킨다. 컴파일 단계에서 코드를 최적화하기 어렵게 한다
4. this.state 는 로직에서 레퍼런스를 공유하기 때문에 문제가 발생할 수 있다.

<br/>

---

## useEffect

기존 class component 의 라이프 사이클을 대체할 수 있는 `useEffect` 에 대해 알아보겠습니다. 기존 class component 의 라이프 사이클을 '대체 할 수 있다' 이지 동등한 레벨이 아님을 주의해주세요. 아래 3개의 라이프 사이클 메서드를 사용 대체할 수 있습니다.

1. componentDidMount
2. componentDidUpdate
3. componentWillUnmount

기존에는 Component 가 Mount 될 때와 Update 될 때 특정한 동작을 수행하도록 하기 위해서는 class component 내부에서 `componentDidMount` 와 `componentDidupdate` 를 이용할 수 있었습니다. 아래와 같이요!

```jsx
import React from "react";

export default class Example extends React.Component {
  state = { count: 0 };

  render() {
    const { count } = this.state;

    return (
      <div>
        <p>You Clicked {count} times</p>
        <button onClick={this.click}>Click Me!</button>
      </div>
    );
  }

  componentDidMount() {
    console.log("componentdidMount", this.state.count);
  }

  componentDidUpdate() {
    console.log("componentDidUpdate", this.state.count);
  }

  click = () => {
    this.setState({ count: this.state.count + 1 });
  };
}
```

이를 functional component 에서도 사용해보겠습니다.

```jsx
import React from "react";

export default function Example5() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    console.log("componentDidMount & componentDidUpdate", count);
  });

  return (
    <div>
      <p>You Clicked {count} times</p>
      <button onClick={click}>Click Me!</button>
    </div>
  );

  function click() {
    setCount(count + 1);
  }
}
```

`useEffect()` 라는 함수 내부에 callback 함수를 정의함으로써 컴포넌트가 Mount 될 때와 Update 될 때 callback 함수가 호출될 수 있도록 하였습니다. 특이한 점은 Mount, Update 가 묶여 있다는 점입니다.

분리할 수는 없을까요? Mount 될 때 수행되는 동작 따로 있고, Update 될 때 수행되는 동작이 따로 있어야 할 상황이 있을 수도 있을 것입니다. 이 때, `useEffect()` 의 두 번째 인자인 `React.DependencyList` 를 사용할 수 있습니다.

```jsx
React.useEffect(() => {
  console.log("componentDidMount & componentDidUpdate", count);
}, []);
```

`React.DependencyList` 는 배열입니다. 빈 배열을 넣어보세요. 최초 컴포넌트가 한 번 Mount 될 때 로그가 찍힌 후에 아무리 'count' 를 바꾸어 보아도 콘솔이 찍히지 않습니다. `React.DependencyList` 는 어떤 state 에 의존하여 그 state 가 변경 될 때 `useEffect` 를 호출 할 것인지에 대한 리스트 입니다. 아무 것도 설정하지 않았다는 것은 그 어떤 변화에도 반응하지 않겠다는 의미겠죠?

```jsx
React.useEffect(() => {
  console.log("componentDidMount & componentDidUpdate", count);
}, [count]);
```

배열 안에 count 를 넣고, 버튼을 클릭해보면 로그가 잘 찍혀 나오는 것을 볼 수 있습니다.

마지막으로 `componetWillUnmount` 를 functional component 에서 사용하는 법을 알아보겠습니다.

```jsx
import React from "react";

export default function Example5() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    console.log("componentDidMount & componentDidUpdate", count);

    // componentWillUnmount 에 해당하는 부분
    return () => {
      console.log("componentWillUnmount", count);
    };
  }, [count]);

  return (
    <div>
      <p>You Clicked {count} times</p>
      <button onClick={click}>Click Me!</button>
    </div>
  );

  function click() {
    setCount(count + 1);
  }
}
```

`useEffect()` 의 콜백 함수 내부 반환 값으로 함수를 정의합니다. 이 것이 곧 `componentWillUnmount` 입니다. 버튼을 누르며 콘슬을 확인해보세요.

가장 최초에 `componentDidMount & componentDidUpdate 0` 이 나옵니다. <br/>
그리고 버튼을 클릭하면 `componentWillUnmount 0` 와 `componentDidMount & componentDidUpdate 1` 이 두 줄에 걸쳐 차례로 나오는 것을 볼 수 있습니다.

이를 통해 우리는 state 가 변경될 때마다 (1) 기존의 컴포넌트가 unmount 되고, (2) 새로운 컴포넌트로 Update 된다는 사실을 알 수 있습니다.

<br/>

---

## 나만의 Hooks

나만의 hooks 를 커스텀 해봅시다.

별개의 'hooks' 라고 하는 디렉토리에 `useWindowWidth.js` 파일을 만듭니다. 윈도우 창의 가로 크기가 변할 때 마다 그 크기를 찍어주도록 해보려고 합니다.

```jsx
import { useState, useEffect } from "react";

export default function useWindowWidth() {
  // state.width 초기화!
  const [width, setWidth] = useState(window.innerWidth);

  /*
   * 라이프 사이클 메서드
   *
   * componentWillUpdate 때마다
   * addEventListener, removeEventListener 할 필요는 없다.
   * 따라서 두 번째 인자로 빈 배열을 넣어 componentWillUpdate 메서드는 제외
   */
  useEffect(() => {
    const resize = () => {
      setWidth(window.innerWidth);
    };
    window.addEventListener("resize", resize);
    return () => {
      window.removeEventListener("resize", resize);
    };
  }, []);

  return width;
}
```

이제 `App.js` 에서 테스트 해보세요.

```jsx
import useWindowWidth from "./hooks/useWindowWidth";

function App() {
  const width = useWindowWidth();
  return <div className="App">{width}</div>;
}

export default App;
```

<br/>

---

## useReducer

state 를 변경하는 useState 의 확장판이라고 보면 되겠습니다. 다수의 하윗값을 포함하는 복잡한 정적 로직을 만드는 경우, 다음 state 가 이전 state 에 의존적인 경우 사용하면 좋습니다.

기존에 사용하던 버튼을 클릭할 시 카운트가 올라가던 예제를 계속 사용해볼께요.

```jsx
import { useState } from "react";

export default function Example6() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You Clicked {state.count} times</p>
      <button onClick={click}>Click Me!</button>
    </div>
  );

  function click() {
    setCount(count + 1);
  }
}
```

여기에 `useReducer` 를 적용해보겠습니다. 아래와 같은 코드로 변하겠네요. 설명은 주석으로 대체합니다.

```jsx
import { useReducer } from "react";

/*
 * reducer
 * 1. state, action 을 받아옴
 * 2. action.type 을 보고 분기 처리
 */
const reducer = (state, action) => {
  if (action.type === "PLUS") {
    return {
      count: state.count + 1,
    };
  }
  return state;
};

export default function Example6() {
  // const [count, setCount] = useState(0);
  /*
   * useReducer
   * 1. state 를 { count: 0 } 으로 초기화 하겠다.
   * 2. state 를 변경하는 로직으로 reducer 라는 함수를 사용할 것이다.
   * 3. dispatch 는 action 을 정의하여 reducer를 실행한다.
   */
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>You Clicked {state.count} times</p>
      <button onClick={click}>Click Me!</button>
    </div>
  );

  function click() {
    // setCount(count + 1);
    /*
     * dispatch
     * 1. reducer 를 실행할 것이다.
     * 2. 인자로 action 을 전달할건데 action.type = 'PLUS' 로 정의한다.
     */
    dispatch({ type: "PLUS" });
  }
}
```

<br/>

---

## useMemo

<br/>

---

## useRef

<br/>

---

## useCallback

<br/>

---

## useHistory

Hook 이전에 별개의 컴포넌트에서 `react-router-dom` 을 이용하여 `history` 를 사용하기 위해서는 `withRouter` 로 props 를 받아와서 props.history 를 사용해야 했었습니다. 아래와 같이 말이죠!

```jsx
import { withRouter } from "react-router-dom";

export default withRouter(function LoginButton(props) {
  function login() {
    props.history.push("/");
  }

  return <button onClick={login}> 로그인! </button>;
});
```

여기에 Hooks 를 적용해보겠습니다. 아래와 같이 수정해주세요.

```jsx
import { useHistory } from "react-router-dom";

export default function LoginButton(props) {
  const history = useHistory();
  function login() {
    history.push("/");
  }

  return <button onClick={login}> 로그인! </button>;
}
```

훨씬 편리하죠.

<br/>

---

## useParams

React Router 에서 동적으로 변하는 params 를 처리하던 방식을 다시 보겠습니다.

```jsx
import { BrowserRouter, Route } from "react-router-dom";
import Home from "./pages/Home";
import Profile from "./pages/Profile";

function App() {
  return (
    <BrowserRouter>
      <Route path="/profile/:id" component={Profile}></Route>
      <Route path="/profile" component={Profile}></Route>
      <Route path="/" exact component={Home}></Route>
    </BrowserRouter>
  );
}
```

위와 같이 `id` 가 동적으로 변하고,

```jsx
export default function Profile(props) {
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

props 를 전달받아 `props.match.params` 객체에 들어있는 `id` 값을 꺼내 쓸 수 있었습니다.

이를 Hooks 를 사용하여 수정해보겠습니다.

```jsx
import { useParams } from "react-router";

export default function Profile() {
  const params = useParams();
  const id = params.id;

  return (
    <div>
      <h2>Profile 페이지입니다.</h2>
      <p>id 는 {id} 입니다.</p>
    </div>
  );
}
```

이 역시 훨씬 편리하죠!

<br/>

---

매우 자주 사용되는 Hooks 를 위주로 선 정리를 해두었습니다. 내용이 없는 것들은 추후에 더 추가하도록 하겠습니다.

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
