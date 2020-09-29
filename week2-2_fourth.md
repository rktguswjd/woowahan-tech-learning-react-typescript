# 우아한 테크러닝 4회차

## 강의목표

- Redux 상태디자인
- 비동기와 제너레이터

---

## 강의

### React 컴포넌트의 상태와 관리

다음과 같은 `React` 애플리케이션을 만든다.

- `index.js`

```javascript
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById("root")
);
```

- `App.js`

```javascript
import React from "react";

const App = () => {
  return (
    <div>
      <header>
        <h1>React and TypeScript</h1>
      </header>
      <ul>
        <li>1회차: Overview</li>
        <li>2회차: Redux 만들기</li>
        <li>3회차: React 만들기</li>
        <li>4회차: 컴포넌트 디자인 및 비동기</li>
      </ul>
    </div>
  );
};

export default App;
```

위와 같은 형태로 되어있는 것을 `li`태그 데이터(상태)를 객체로 만들어 `index.js`에서 `App` 컴포넌트 값으로 넘겨 준다.

- `index.js`

```javascript
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

const rootElement = document.getElementById("root");

const sessionList = [
  { id: 1, title: "1회차: Overview" },
  { id: 2, title: "2회차: Redux 만들기" },
  { id: 3, title: "3회차: React 만들기" },
  { id: 4, title: "4회차: 컴포넌트 디자인 및 비동기" },
];

ReactDOM.render(<React.StrictMode></React.StrictMode>);
```

- `App.js`

```javascript
import React from "react";
const App = (props) => {
  const { sessionList } = props.store;
  return (
    <div>
      <header>
        <h1>React and TypeScript</h1>
      </header>
      <ul>
        {sessionList.map((session) => (
          <li>{session.title}</li>
        ))}
      </ul>
    </div>
  );
};

export default App;
```

`li`태그를 `map`을 사용해 생성할 수 있는데 이는 태그를 바노한하는 코드가 많이 질수록 가독성이 떨어지기 때문에 컴포넌트로 한번 더 분리하는 것이 좋다.

- `App.js`

```javascript
import React from "react";

const SessionItem = ({ title }) => <li>{title}</li>;

const App = (props) => {
  const { sessionList } = props.store;
  return (
    <div>
      <header>
        <h1>React and TypeScript</h1>
      </header>
      <ul>
        {sessionList.map((session) => (
          <SessionItem title={session.title} />
        ))}
      </ul>
    </div>
  );
};

export default App;
```

위와 같이 분리하여 가독성이 좋게 만들었다. 하지만 SessionItem과 App컴포넌트는 상태를 가지고 있지 않다. 버튼을 누르면 오름차순, 내림차순으로 바뀌는 상태를 가지도록 컴포넌트를 만든다.

- `App.js`

```javascript
import React from "react";

const App = (props) => {
  let displayOrder = "ASC";
  const { sessionList } = props.store;

  const orderedSessionList = sessionList.map((session, i) => ({
    ...session,
    order: i,
  }));

  const toggleDisplayOrder = () => {
    displayOrder = displayOrder === "ASC" ? "DESC" : "ASC";
  };

  return (
    <div>
      <header>
        <h1>React and TypeScript</h1>
      </header>
      <button onClick={toggleDisplayOrder}>재정렬</button>
      <ul>
        {orderedSessionList.map((session) => (
          <SessionItem title={session.title} key={session.id} />
        ))}
      </ul>
    </div>
  );
};

export default App;
```

위와 같이 버튼을 클릭해도 `displayOrder`의 값은 변경되지 않고 다시 렌더링되지 않는다. 리액트는 `App`이 다시 호출되어 `Virtual DOM`이 다시 만들어지게 되고 DOM에 업데이트를 해야하지만 안에서 함수가 이미 끝났기 때문에 다시 호출해야한다는 신호를 함수 바깥으로 보낼수가 없다. 따라서 초창기에는 클래스 컴포넌트를 사용할 수 밖에 없었다.
하지만 `React`의 함수 컴포넌트에서 상태를 가질 수 있게 되었는데 이것이 `Hooks`다.
`App.js`가 상태를 가질 수 있도록 `useState`를 이용해 `Hooks`를 사용해보자.

- `App.js`

```javascript
import React, { useState } from "react";

const SessionItem = ({ title }) => <li>{title}</li>;

const App = (props) => {
  const [displayOrder, setDisplayOrder] = useState("ASC");
  const { sessionList } = props.store;
  const orderedSessionList = sessionList.map((session, i) => ({
    ...session,
    order: i,
  }));

  const toggleDisplayOrder = () => {
    setDisplayOrder(displayOrder === "ASC" ? "DESC" : "ASC");
  };

  console.log(displayOrder);

  return (
    <div>
      <header>
        <h1>React and TypeScript</h1>
      </header>
      <p>전체 세션 갯수: {displayOrder}개</p>
      <button onClick={toggleDisplayOrder}>재정렬</button>
      <ul>
        {orderedSessionList.map((session) => (
          <SessionItem key={session.id} title={session.title} />
        ))}
      </ul>
    </div>
  );
};

export default App;
```

`useState`의 인자는 상태 초기값이고 [상태, 상태를 변경하는 함수]형태의 배열을 반환한다.
<br />
<br />

### 비동기와 제너레이터

- 제너레이터

```javascript
function* foo() {}
```

제너레이터는 위와 같이 `function*`으로 선언한다.

- 비동기 함수

```javascript
async function bar() {}
```

비동기 함수는 `function`앞에 `async`키워드를 붙여 사용한다.
<br />

아래 코드는 동기적으로 실행되는 `javascript`코드이다.

```javascript
const x = 10;
const y = x * 10;
```

`y`변수에 `x`의 의존성이 있기 때문에 `x`가 없이 `y`가 생길 수 없다.
함수인 경우에는 ?

```javascript
const x = () => 10;
const y = x() * 10;
```

`x`의 값이 확정되는 순간에 `y`의 호출 시점이 되는데, 함수를 반환할 수 있다는 특성으로 `x`와 같이 지연호출이 가능하다.
`Promise`도 지연호출과 비슷하다.

```javascript
const p = new Promise(function (resolve, reject) {
  setTimeout(() => {
    resolve("1");
  }, 1000);
});

p.then(function (r) {
  console.log(r);
});
```

- 코루틴
  제너레이터는 제너레이터 객체를 반환하는데 제너레이터 객체는 코루틴이라는 함수의 구현체이다.
  > 코루틴 <br />
  > 함수는 인자를 받으면 계산을 하고 리턴을 한다. 그리고 한 번 호출하고 결과를 받거나 그 다음 스템을 실행한다. 이런 상황을 폭 넓게 넓혀서 호출자한테 리턴을 여러 번 할 수 있게 해주는 것이다.
  > 리턴을 여러 번 한다는 것은 함수가 다시 호출될 때 처음부터 시작되는 것이 아닌 마지막 리턴한 지점부터 시작한다는 것이고 함수를 조금 더 일반화한 컨셉으로 확장해서 기능을 추가해 놓은 것이 코루틴이다. 자바스크립트에서 코루틴의 개념을 차용해 제너레이터라고 명명했다.

```javascript
function* make() {
  return 1;
}

console.log(make());
```

위 코드에서 일반적인 함수인 경우에 make함수의 반환값을 1일 것이다. 하지만 위 `make`함수는 `제너레이터`이기 때문에 객체가 반환된다.
`제너레이터`는 `yield`와 함께 사용한다.

```javascript
function* makeNumber() {
  let num = 1;

  while (true) {
    yield num++;
  }
}
```

아래에 `makeNumber()`와 같이 제너레이터를 생성해도 값이 반환되지 않고 객체를 넘겨준다.

```javascript
const g = makeNumber();
```

실제 `제너레이터`의 값을 가져오기 위해서 제너레이터 객체의 `next`를 사용한다.

```javascript
console.log(g.next());
```

![다운로드](https://user-images.githubusercontent.com/52924202/94557953-3dcd0e80-029a-11eb-8d69-1e37162be798.png)
<br/>
콘솔을 확인해 보면 `value`는 `yiede`로 반환된 값이고, `done`은 `true`가 될 경우 제너레이터가 종료된다. 즉, `yield`나 `return`이 나올 때 까지 실행하는 것이다.
제너레이터의 `next`메서드는 값을 받을 수 있다.

```javascript
function* makeNumber() {
  let num = 1;

  while (true) {
    const x = yield num++;
    console.log(x);
  }
}

const g = makeNumber();

console.log(g.next());
console.log(g.next("a"));
```

![다운로드 (1)](https://user-images.githubusercontent.com/52924202/94558305-c055ce00-029a-11eb-858a-b70537d47bde.png)
위와 같이 `next`에 매개변수로 넘겨준 `a`가 `yield`의 반환값으로 담겨 출력된다.
제너레이터를 이용해 비동기함수를 동기적으로 나타낼 수 있다.

```javascript
const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

function* main() {
  console.log("시작");
  yield delay(3000);
  console.log("3초 뒤");
}

const it = main();

const { value } = it.next();

value.then(() => {
  it.next();
});
```

코드를 실행시키면 "시작"이 출력되고 3초가 지난 뒤에 "3초 뒤"라고 출력된다.
`async / await`를 사용해 비슷하게 사용할 수 있다.

```javascript
async function main2() {
  console.log("시작");
  await delay(3000);
  console.log("3초뒤");
}
main2();
```

`async / await`은 `Promise`에 최적화 되어있지만 제너레이터를 이용하는 경우에 `Promise`형태의 값이 반환되지 않아도 사용 가능하다. 따라서 더 다양한 상황에서 응용이 가능하다.
