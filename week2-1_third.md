# 우아한 테크러닝 3회차

## 강의목표

- React 만들기
- React 이해하기

---

## 강의

### 간단한 React

```javascript
const list = [
  { title: "React에 대해 알아봅시다." },
  { title: "Redux에 대해 알아봅시다." },
  { title: "Typescript에 대해 알아봅시다." },
];

const rootElement = document.getElementById("root");

function app(items) {
  rootElement.innerHTML = `
    <ul>
        ${items.map((item) => `<li>${item.title}</li>`).join("")}
    </ul>
    `;
}

app(list);
```

list를 app으로 rootElement의 `innerHTML`을 이용해 화면을 그려주고 있는데, app함수는 `innerHTML`과 같이 직접 `DOM`에 접근한다. `DOM`에 직접 접근하는 것은 권장하지 않는다. 규모가 커지게되면 복잡해질 수 있기 때문이다. 이러한 이유로 `Virtual DOM` 형태의 프레임워크들이 개발되었고 그 중 하나가 Facebook의 `React`이다.
<br />
<br />

### DOM과 Virtual DOM

```javascript
import React from "react";
import ReactDOM from "react-dom";
function App() {
  return (
    <div>
      <h1>Hello?</h1>
      <ul>
        <li>React</li>
        <li>Redux</li>
        <li>MobX</li>
        <li>Typescript</li>
      </ul>
    </div>
  );
}
ReactDOM.render(<App />, document.getElementById("root"));
```

App 함수는 `JSX`를 반환하고 있으며 `JSX`문법은 `javascript`를 확장한 문법으로 `HTML`처럼 태그를 사용한다. `ReactDOM`은 `render`라는 정적메서드를 가진다. 그리고 2개의 인자를 받는데 첫 번째 인자로 화면에 렌더링할 컴포넌트이고 두 번째 인자는 컴포넌트를 렝더링할 요소이다.

```javascript
"use strict";
var _react = _interopRequireDefault(require("react"));
var _reactDom = _interopRequireDefault(require("react-dom"));
function _interopRequireDefault(obj) {
  return obj && obj.__esModule ? obj : { default: obj };
}
function App() {
  return /*#__PURE__*/ _react.default.createElement(
    "div",
    null,
    /*#__PURE__*/ _react.default.createElement("h1", null, "Hello?"),
    /*#__PURE__*/ _react.default.createElement(
      "ul",
      null,
      /*#__PURE__*/ _react.default.createElement("li", null, "React"),
      /*#__PURE__*/ _react.default.createElement("li", null, "Redux"),
      /*#__PURE__*/ _react.default.createElement("li", null, "MobX"),
      /*#__PURE__*/ _react.default.createElement("li", null, "Typescript")
    )
  );
}
_reactDom.default.render(
  /*#__PURE__*/ _react.default.createElement(App, null),
  document.getElementById("root")
);
```

위 코드는 `Babel`에서 `JSX`를 `javascript`로 변환한 결과다.

```javascript
import React from "react";
import ReactDOM from "react-dom";
function StudyList() {
  return (
      <ul>
          <li>React</li>
          <li>Redux</li>
          <li>MobX</li>
          <li>Typescript</li>
      </ul>
  );
}
function App() {
  return (
      <div>
          <h1>Hello?</h1>
          <StudyList />
      </div>
  );
}
ReactDOM.render(<App />, document.getElementById("root"));
}
```

App함수 내부에 있던 `ul > li`컴포넌트들을 StudyList함수로 만들어 분리했다.
컴포넌트를 분리하면서 이름을 갖는 함수가 생기며 코드를 관리하기에 더 쉬워졌다.
위 StudyList 함수에서 반환하는 `JSX`엘리먼트들을 Babel로 변환되면 아래와 같은 형태이다.

```javascript
function StudyList() {
  return /*#__PURE__*/ _react.default.createElement(
    "ul",
    null,
    /*#__PURE__*/ _react.default.createElement(
      "li",
      {
        className: "item",
      },
      "React"
    ),
    /*#__PURE__*/ _react.default.createElement(
      "li",
      {
        className: "item",
      },
      "Redux"
    ),
    /*#__PURE__*/ _react.default.createElement(
      "li",
      {
        className: "item",
      },
      "MobX"
    ),
    /*#__PURE__*/ _react.default.createElement(
      "li",
      {
        className: "item",
      },
      "Typescript"
    )
  );
}
```

변환된 상태를 참고해 객체로 바꾸면 아래의 `vdom`과 같은 형태를 가지게된다.

```javascript
const vdom = {
  type: "ul",
  props: {},
  children: [
    { type: "li", props: { className: "item" }, children: "React" },
    { type: "li", props: { className: "item" }, children: "Redux" },
    { type: "li", props: { className: "item" }, children: "Typescript" },
    { type: "li", props: { className: "item" }, children: "MobX" },
  ],
};
```

`vdom`을 보고 왜 `React`가 상위 단 하나의 최상위 부모 컴포넌트가 있는지 알 수 있다. 이렇게 `React`는 `Virtual DOM`이 단 하나다.
<br />
<br />

### React만들기

위의 코드에서 `JSX`가 변환된 모습을 보면 `createElement`는 아래와 같이 구성되어 있다.

```javascript
function createElement(type, props = {}, ...children) {
  return { type, props, children };
}
```

```javascript
const vdom = createElement("ul", {}, createElement("li", {}, "React"));
```

`createElement`를 사용하여 `Virtual DOM`을 만들 수 있다.
<img width="483" alt="1" src="https://user-images.githubusercontent.com/52924202/94428257-62a68080-01cb-11eb-9fc3-829fd2a542b7.png">

`vdom`과 동일하게 객체가 구성된다. `createElement`는 `vdom`객체를 만드는 함수이고 `render`를 할 때 `render`객체는 계속 쫒아가면서 realdom으로 변환한다.

- `App.js`

```javascript
/* @jsx createElement */
import { createElement, render } from "./tiny-react";

function Hello(props) {
  return <li className="item">{props.label}</li>;
}

function App() {
  return (
    <div>
      <h1>Hello World</h1>
      <ul className="board" onClick={() => null}>
        <Hello label="Hello" />
        <Hello label="World" />
        <Hello label="React" />
      </ul>
    </div>
  );
}

render(<App />, document.getElementById("root"));
```

- `tiny-react.js`

```javascript
function renderElement(node) {
  if (typeof node === "string") {
    return document.createTextNode(node);
  }

  const el = document.createElement(node.type);

  node.childern.map(renderElement).forEach((element) => {
    el.appendChild(element);
  });
  return el;
}

export function render(vdom, container) {
  container.appendChild(renderElement(vdom));
}

export function createElement(type, props, ...children) {
  if (typeof type === "function") {
    return type.apply(null, [props, ...children]);
  }
  return { type, props, children };
}
```

<br />
<br />

### 클래스 컴포넌트와 함수 컴포넌트(Hooks)

- 클래스 컴포넌트
  클래스 컴포넌트는 라이프사이클 API를 사용하여 state를 생성자에서 초기화 할 수 있다.

```javascript
class Hello extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      count: 1,
    };
  }

  componentDidMount() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return <p>안녕하세요!</p>;
  }
}
```

상태를 변경하려면 `React.Component`에 존재하는 `setState`메서드를 사용한다.
클래스 컴포넌트의 상태는 객체의 생성자에 초기화되어 `React`가 지우지 않는 이상 유지된다.

- 함수 컴포넌트
  함수 컴포넌트는 초창기에 상태는 함수가 호출할 때 마다 생성되었기 때문에 유지될 수 없어 상태를 관리하지 못한다고 여겨져 사용하지 않았다.

```javascript
function App() {
  let x = 10;
  return (
    <div>
      <h1>상태</h1>
      <Hello />
    </div>
  );
}
```

위의 App컴포넌트의 x와 같은 함수 내부의 변수가 상태처럼 여겨졌다. 하지만 Hooks의 등장으로 함수 컴포넌트에서도 상태를 관리할 수 있게 되었다.

```javascript
import React, { useState } from "react";

function App() {
  const [count, setCount] = useState(1);

  return (
    <div>
      <h1>상태</h1>
      <Hello />
    </div>
  );
}
```

클래스 컴포넌트처럼 상태를 관리하기 위해서 `useState`를 사용한다. `useState`의 반환 배열의 첫번째는 상태값, 두번째는 `dispatcher`로 사용되는 함수다.

```javascript
import React, { useState } from "react";

function App() {
  const [count, setCount] = useState(1);

  return (
    <div>
      <h1 onClick={() => setCounter(count + 1)}>상태 {count}</h1>
      <Hello />
    </div>
  );
}
```

위와 같이 h1태그에 onClick 이벤트를 주어 `dispatcher`를 사용해 상태를 변경 할 수 있다.

`Hooks`는 항상 최상위(at the Top Level)에서만 `Hooks`를 호출해야 한다.
최상위가 아닌 부분에서 호출 될 경우 전역 배열에 원하지 않는 값을 반환하여 문제가 발생 할 경우가 생길 수 있다.
