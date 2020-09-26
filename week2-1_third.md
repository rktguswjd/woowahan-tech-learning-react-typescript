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
      <h1>Hello!!</h1>
      <StudyList />
    </div>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

StudyList 함수는 `JSX`를 반환하고 있으며 `JSX`문법은 `javascript`를 확장한 문법으로 `HTML`처럼 태그를 사용한다. `ReactDOM`은 `render`라는 정적메서드를 가진다. 그리고 2개의 인자를 받는데 첫 번째 인자로 화면에 렌더링할 컴포넌트이고 두 번째 인자는 컴포넌트를 렝더링할 요소이다.
