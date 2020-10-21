# 우아한 테크러닝 8회차

## 강의목표

- [마지막 예제](https://codesandbox.io/s/navigation08live-8op00?file=/src/index.tsx)
- MobX

## 강의

### MobX

`MobX`는 `React`에 의존성이 없는 일반적인 자바스크립트 상태관리 라이브러리다. <br />
`React`와 같이 사용하기 위해서는 `mobx-react`라이브러리를 사용해야 한다.
`MobX`를 알아보기 전 `setInterval`로 값을 증가시킬 수 있다.

- `App.tsx`

```javascript
import * as React from "react";
import "./styles.css";

interface AppProps {
  data: number;
}

export default function App(props: AppProps) {
  return (
    <div className="App">
      <h1>외부 데이터: {props.data}</h1>
    </div>
  );
}
```

작성한 `App`컴포넌트는 아래와 같이 사용된다.

- `index.tsx`

```javascript
import * as React from "react";
import { render } from "react-dom";

import App from "./App";

const store = {
  data: 1,
};

const rootElement = document.getElementById("root");

render(<App data={1} />, rootElement);
```

`Redux`와 달리 `MobX`는 한번에 여러개의 상태를 관리하는 `store`를 만들 수 있다. <br />
`MobX`에서는 관찰해 관리할 수 있도록 돕는 `observable`함수를 제공한다.

```javascript
import { observable } from "mobx";

const cart = observable({
  data: 1,
});

const rootElement = document.getElementById("root");
render(<App data={cart.data} />, rootElement);
```

`MobX`에서는 `Redux` 스토어의 `subscribe`와 비슷한 기능을 하는 `autorun` 함수를 제공한다.

```javascript
import { observable, autorun } from "mobx";

const cart = observable({
  data: 1,
});

autorun(() => {
  console.log(`in autorun => ${cart.data}`);
});

const rootElement = document.getElementById("root");
render(<App data={cart.data} />, rootElement);

cart.data++;
```

위 코드에서 `autorun`에 등록된 `console.log`는 두 번 콘솔에 찍히게 된다. <br />
처음 `observable`에 값이 초기화될 때와 `cart.data++`로 값이 변경될 때이다.

```javascript
autorun(() => {
  console.log(`in autorun => ${cart.data}`);
  render(<App data={cart.data} />, rootElement);
});

setInterval(() => {
  cart.data++;
}, 1000);
```

위와 같이 `autorun`함수에 `render`를 넣게되면 변경되는 상태가 컴포넌트에 적용 된다.
`App` 컴포넌트의 속성에 `number`타입의 `counter`속성을 아래와 같이 추가한다.

- `App.tsx`

```javascript
interface AppProps {
  data: number;
  counter: number;
}

export default function App(props: AppProps) {
  return (
    <div classNamep="App">
      <h1>
        외부 데이터: {props.data} vs {props.counter}
      </h1>
    </div>
  );
}
```

마찬가지로 `observable`함수로 생성된 `cart`스토에도 `counter`속성을 추가해준다.

- `index.tsx`

```javascript
import { observable, autorun } from "mobx";

const cart = observable({
  data: 1,
  counter: 1,
});

autorun(() => {
  render(<App data={cart.data} counter={cart.counter} />, rootElement);
});

setInterval(() => {
  cart.data++;
  cart.counter += 2;
}, 1000);
```

일반적으로 `observable`로 감쌀 수 있는 것은 객체나 배열과 같은 객체 타입값이다.<br />
`number`, `string` 같은 원시타입을 감싸주기 위해서는
`observable.box`를 사용해야 한다.

```javascript
const weight = observable.box(82);
```

`observalbe.box`로 만들어진 `weight`는 객체가 되며 `getter`와 `setter`를 갖는다.
`weight`의 기존값을 기반으로 값을 변경시키기 위해서는 `get`,`set` 메서드를 사용할 수 있다.

```javascript
setInterval(() => {
  cart.data++;
  cart.counter += 2;
  weight.set(weight.get() - 1);
}, 1000);
```

위 처럼 값이 변경될 때 `autorun`내부에 로깅을 진행할 경우 3개의 값이 바뀌어 3번의 로그가 찍힌다.
이러한 상황을 방지하게 위해서는 `MobX`가 제공하는 `action`함수를 이용할 수 있다.

```javascript
import { action } from "mobx";

const myAction = action(() => {
  cart.data++;
  cart.counter += 2;
  weight.set(weight.get() - 1);
});

setInterval(() => {
  myAction();
}, 1000);
```

결론적으로 `MobX`가 제공하는 `action`함수는 작업단위를 묶어주는 역할을 한다.

### observable 클래스와 데코레이터

작성한 `cart`객체를 `action`기능과 묶어 아래와 같이 클래스로도 작성해볼 수 있다.

```javascript
class Cart {
  data: number = 1;
  cunter: number = 1;

  myAction = action(() => {
    this.data++;
    this.counter += 2;
  });
}

const cart = new Cart();

cart.myAction();
```

하지만 위의 코드로만은 `cart`인스턴스가 `observable`이 아니기 때문에 `autrun`이 실행되지 않는다.
`Cart`클래스를 `observable`로 관리하기 위해서는 데코레이터 문법을 사용해야 한다.
`tsconfig.json`의 `compileOptions`의 `experimentalDecorators`속성을 변경해주어야 한다.

```javascript
{
  "compileOptions": {
    "experimentalDecorators": true
  }
}
```

그 후 아래와 같이 `@observable`데코레이터를 `Cart`클래스 위에 붙여주면 된다.

```javascript
@observable
class Cart {
  data: number = 1;
  cunter: number = 1;

  myAction = action(() => {
    this.data++;
    this.counter += 2;
  });
}
```

클래스 자체가 아닌 클래스의 멤버 변수에 `@observable` 데코레이터를 붙일 수도 있다.

```javascript
class Cart {
  @observable data: number = 1;
  @observable cunter: number = 1;

  myAction = action(() => {
    this.data++;
    this.counter += 2;
  });
}

const cart = new Cart();

setInterval(() => {
  cart.myAction();
}, 3000);
```

위와 같이 `@observable` 데코레이터를 사용하게 되면 값이 정상적으로 바뀌며 리렌더가 된다. <br />
`observable`함수와 마찬가지로 `action` 함수도 동일하게 데코레이터로 사용할 수 있다.

```javascript
class Cart {
  @observable data: number = 1;
  @observable cunter: number = 1;

  @action
  myAction() {
    this.data++;
    this.counter += 2;
  }
}
```

이전과 동일하게 정상적으로 작동하며 ES6문법에서 사용할 수 있는 데코레이터는 함수임을 알 수 있다.

### 클래스 메소드와 this

클래스의 메소드를 사용할 때는 `this`객체가 어느것을 가리키는지 확인하는 것이 중요하다

```javascript
setInterval(() => {
  cart.myAction.call(null);
}, 3000);
```

위 처럼 `cart` 인스턴스 객체가 아닌 다른곳에 `this` 바인딩이 되어있을 경우 문제가 발생한다. `MobX`의 `action`함수는 이러한 상황을 미연에 방지하기 위해 `action.bound`기능을 제공한다.

```javascript
class Cart {
  @observable data: number = 1;
  @observable cunter: number = 1;

  @action.bound
  myAction() {
    this.data++;
    this.counter += 2;
  }

  @action
  myBindedAction = () => {
    this.data++;
    this.counter += 2;
  };
}
```

또는 `action` 데코레이터만 사용하고 클래스 메서드를 화살표 함수로 작성하면 된다.

### observable에 반응하는 함수

`autorun`과 비슷한 기능을 하는 함수는 `when`과 `reaction`이 추가적으로 존재한다.

1. when

- `when`함수 사용법

```javascript
when(pedicate: () => boolean, effect?: () => void, options?)
```

- `when`함수 예시

```javascript
class Resource {
  constructor() {
    when(
      () => !this.isVisible,
      () => this.dispose()
    );
  }

  @computed get isVisible() {}

  dispose() {}
}
```

`when` 함수는 기본적으로 두 개의 함수를 인자로 받는다. 첫 번째 인자는 `boolean`을 반환하는 함수고, 두 번째 함수는 첫 번째 함수가 `true`를 반환할 경우 실행된다.

2. reaction

- `reaction` 함수 사용법

```javascript
reaction(() => data, (data, reaction) => { sideEffect }, options?)
```

- `reaction` 함수 예시

```javascript
import { reaction } from "mobx";

const myReaction = reaction(
  () => todos.length,
  (length) =>
    console.log(`myReaction: ${todos.map((todo) => todo.title).join(", ")}`)
);
```

`reaction`은 `useEffect`와 비슷하게 생겼으며 두 개의 함수를 인자를 받는다. <br />
첫 번째 함수의 반환값은 두 번째 함수의 인자로 전달되며 반환값이 이전과 다를 경우에만 실행된다.

### mobx-react

기존의 `App.tsx`에서는 `autorun`함수를 이용해 컴포넌트를 다시 렌더링 시킨다.

```javascript
autorun(() => {
  render(<App data={cart.data} counter={cart.counter} />, rootElement);
});
```

위의 기능을 하며 `MobX`와 함께 사용할 수 있는 라이브러리는 `mobx-react`가 있다. `mobx-react`의 6버전 이전은 `Hook`을 지원하지 않으며 이후부터는 지원을 한다.
`Hook`을 지원하는 `mobx-react`라이브러리로 `mobx-react-lite`가 있다.

- `App.tsx`

```javascript
import React from 'react';
import { inject, observer } from 'mobx-rect';

interface AppProps {
  data: number;
  counter: number;
}

@observer
function App(props: AppProps) {
  return (
    <div classNamep='App'>
      <h1>
        외부 데이터: {props.data} vs {props.counter}
      </h1>
    </div>
  );
}

export default App;
```

함수 컴포넌트에 `@observer` 데코레이터를 사용하면 오류가 발생하기 때문에 클래스로 변경해야 한다.

- `App.tsx`

```javascript
import React from "react";
import { inject, observer } from "mobx-react";

interface AppProps {
  data?: number;
  counter?: number;
}

@injct("cart")
@observer
export default class App extends React.Component<AppProps> {
  render() {
    return (
      <div classNamep="App">
        <h1>
          외부 데이터: {this.props.cart.data} vs {this.props.cart.counter}
        </h1>
      </div>
    );
  }
}
```

`inject` 데코레이터를 이용하면 스토어 인스턴스를 주입할 수 있다. 또 HoC형태가 되기 때문에 `Typescript`와 사용하게 되면 `props`의 타입을 optional로 주어야 한다.

### 비동기 액션

`MobX`에서 비동기로 실행될 액션을 `Promise`형태로 실할 수 있다.

```javascript
import { action, observable } from "mobx";

class Store {
  @observable githubProjects = [];
  @observable state = "pending";

  fetchProjects() {
    this.githubProjects = [];
    this.state = "pending";
    fetchGithubProjectsSomehow().then(
      action("fetchSuccess", (projects) => {
        const filteredProjects = somePreprocessing(projects);
        this.githubProjects = filteredProjects;
        this.state = "done";
      }),
      action("fetchError", (error) => {
        this.state = "error";
      })
    );
  }
}
```

또한 `asunc - await`을 이용해 처리할 수 있다.

```javascript
import { runInAction, observable } from "mobx"

class Store {
    @observable githubProjects = []
    @observable state = "pending" // "pending", "done" or "error"

    async fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        try {
            const projects = await fetchGithubProjectsSomehow()
            const filteredProjects = somePreprocessing(projects)
            runInAction(() => {
                this.githubProjects = filteredProjects
                this.state = "done"
            })
        } catch (e) {
            runInAction(() => {
                this.state = "error"
            }
        }
    )
}
```

이 경우에는 `action`함수를 사용하지 않고 `runInAction` 함수를 사용하면 `action`을 감쌀 수 있다. <br />
`flow`함수를 이용하면 좀 더 간단하게 사용할 수 있다.

```javascript
import { flow } from "mobx";

class Store {
  githubProjects = [];
  state = "pending";

  fetchProjects = flow(function* (this: Store) {
    this.githubProjects = [];
    this.state = "pending";
    try {
      const projects = yield fetchGithubProjectsSomehow();
      const filteredProjects = somePreprocessing(projects);
      this.state = "done";
      this.githubProjects = filteredProjects;
    } catch (error) {
      this.state = "error";
    }
  });
}
```

`flow` 자체가 `action` 역할을 해주므로 따로 `@action` 데코레이터를 사용할 필요 없다.
`flow`는 매개변수로 제너레이터를 받으며 `redux-saga`와 비슷하게 로직을 작성할 수 있다.
