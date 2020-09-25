# 우아한 테크러닝 2회차

## 강의목표

- JavaScript A-Z
- 간단한 Redux 구현
  <br />
  <br />

---

## 강의

### JavaScript 변수

```javascript
var a = 10;
let b = 10;
const c = 10;

function foo() {}
let funcFoo = foo;
```

JavaScript의 타입은 원시타입과 객체타입으로 구분된다. 선언과 동시에 값을 할당할 수 있다.
<br />
<br />

### JavaScript 함수

> 함수는 값으로 취급하여 변수에 대입 가능 <br />
> 모든 함수는 값을 반환

- 함수 정의문

```javascript
function foo() {}
```

- 함수 정의식

```javascript
const bar = function bar() {};
bar();

const bar2 = function () {}; // 이름 생략 가능
```

- IIFE (즉시 실행 함수)

```javascript
(function () {})();
```

- 함수가 함수를 리턴

```javascript
function foo(x){
    x();
    retrun function (){};
}
const foo2 =  foo(function (){});
```

함수는 값으로 취급한다. 따라서 매개변수 x의 형태로 함수를 전달할 수 있다. 또 foo 함수는 반환값으로 함수가 반환될 수 있다.
함수를 매개변수로 넘겨주는 것을 콜백함수라고 하며 이는 호출을 위임하는 역할을 한다.
함수를 인자로 받고 함수를 반환하는 함수를 일급함수라고 한다.

- ES6에서의 함수

```javascript
/* Arrow function (ES6) */

const bar = (x) => {
  return x * 2;
};
const bar2 = (x) => {
  return x * 2;
}; // 단일 인자인 경우 괄호 생략 가능
const bar3 = (x) => x * 2; // 한줄로 구성된 경우 return 구문 생략 가능
```

ES6 이후, 함수식을 만들 때 `=>`를 사용해 화살표 함수로 만들 수 있다. 화살표 함수는 람다식 한 줄 함수로 하나의 식을 반환하는 경우에 자주 사용된다.
<br />
<br />

### 식과 문

```javascript
const a = 10 + 20;
const foo = () => {};
const b = "rktguswjd";
foo();
```

```javascript
if(a){}
while(){}
for(){}
```

`세미콜론(;)`의 작성 여부에 따라 식과 문으로 판단된다.
식은 끝에 세미콜론을 붙이고 문은 붙이지 않으며 값, 연산식, 리터럴 객체, 함수 호출(값 또는 undefined를 return)등이 있다. 문에는 반복문이나 조건문 등이 있다.
<br />
<br />

### new 연산자와 인스턴스 객체

```javascript
function foo() {
  this.age = 23;
}

const ogj = new foo();
console.log(obj);
// foo {age:23, constructor: Object}

if (obj instanceof foo) {
}
```

`new`연산자는 빈 객체를 만들어 함수에게 전달하고 인스턴스 객체를 return한다. `instanceof`로 인스턴스 객체의 원형을 알 수 있다.
<br />
<br />

### ES6이후 Class

```javascript
function Foo() {
  this.name = "rktguswjd";
}

class Bar {
  constructor() {
    this.name = "현정";
  }
}

const foo = new Foo();
const bar = new Bar();
```

함수와 class의 차이점은 명시적인 부분이다. Foo함수는 new연산자 없이 호출이 가능하지만 Bar는 반드시 new연산자와 함께 사용 가능하다. new연산자를 강제하기 원한다면 class로 만드는 것이 좋다.
<br />
<br />

### this와 실행 컨텍스트

```javascript
const person = {
  name: "rktguswjd",
  getName() {
    return this.name;
  },
};

console.log(person.getName()); // rktguswjd

const man = person.getName;

console.log(man()); // TypeError
```

man이 호출되는 시점의 실행 컨텍스트는 전역 객체이므로 name이 존재하지 않는다. this를 고정하기 위해서는 `call`, `apply`, `bind`등의 함수를 사용해야 한다.
<br />
<br />

### 클로저

```javascript
function foo(x) {
  return function bar() {
    return x;
  };
}

const f = foo(10);

console.log(f()); // 10
```

bar 함수를 담은 f는 f()와 같이 호출할 수 있고 10을 반환한다. bar함수에서 접근하는 x변수는 bar함수 스코프의 외부 범위이다. foo함수 호출이 완료된 시점에서 foo의 스코프는 사라지지만 반환된 bar에서는 x를 기억하게 된다.
위와 같이 반환된 함수가 외부의 스코프를 기억하고 있는 상태를 클로저라고 한다.

```javascript
function makePerson() {
  let age = 23;
  return {
    getAge() {
      return age;
    },
    setAge(x) {
      age = x > 1 && x < 130 ? x : age;
    },
  };
}
p = makePerson();
console.log(p.getAge()); // 10
p.setAge(500);
console.log(p.getAge()); // 10
p.setAge(20);
console.log(p.getAge()); // 20
```

클로저를 사용하면 p.age와 같이 변수에 직접 접근할 수 없지만 p.getAge 함수로 값을 받을 수 있다.
<br />
<br />

### 비동기와 Promise

```javascript
setTimeout(function (x) {
  console.log("호출했니?");
}, 1000);

setTimeout(
  function (x) {
    console.log(x);
    console.log("호출했니?");
  },
  2000,
  "X 인자"
);

/*
1초후 : "호출했니?"
2초후 : "X 인자"
       "호출했니?"
*/
```

위처럼 `setTimeout`이나 `API`를 호출하는 경우 비동기적으로 코드가 실행된다.

```javascript
setTimeout(function () {
  console.log("foo");
  setTimeout(function () {
    console.log("bar");
  }, 2000);
}, 1000);
```

비동기 로직를 사용하고, 로직의 깊이가 깊어질 때 `콜백지옥`이 발생한다. 이를 해결하기 위해 나온 것이 `Promise`.

```javascript
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("응답1");
  }, 1000);

  reject();
});

const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("응답2");
  }, 1000);

  reject();
});

p1.then(p2())
  .then(function (r) {
    console.log(r);
  })
  .catch(function (err) {
    console.err(err);
  });
```

`Propmise`에서 `then`의 콜백함수는 `resolve`로 호출되고 `catch`는 `reject`로 호출된다.
<br />
<br />

### async와 await

`Promise`를 더 간결하고 가독성이 좋게 만들어 주는 패턴이 `async-await`이다.

```javascript
onst delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

async function main() {
    console.log("1");
    try {
        const result = await delay(3000);
    } catch (e) {
        console.error(e);
    }
    console.log("2");
}

main();
```

main 함수가 실행 되면, 1을 출력한 뒤 3초 뒤에 2가 출력된다. `await`dms `async`키워드가 붙은 함수에서 사용이 가능하다. `Promise`에서 `reject`는 `try-catch`구문의 `catch`를 이용해 사용할 수 있다.
<br />
<br />

### Rdux

`redux`는 상태 관리 라이브러리다.
`redux`의 `store`에 저장된 데이터는 컴포넌트가 직접적으로 데이터를 바꿀 수 없다. 상태를 직접적으로 바꿀 수 없도록 하기위해 클로저를 이용해 `state`를 은닉화 한다.
`createStore`를 이용해 `store`를 생성할 수 있고 `getState`를 이용해 상태를 볼 수 있다. `store`에 저장된 값은 `reducer`함수가 호출되어 변경되어야 한다. `reducer`는 반드시 `action`을 인자로 받는 `dispatch`를 이용해 호출되어야 한다.
`reducer`는 호출될 때 기존 저장된 상태와 실행할 액션 객체를 매개변수로 전달 받는다. 액션 객체의 타입에 `type`에 따라 기존 `state`의 값을 변경시켜 `store`에 저장된다.
`subscribe`함수는 `store`를 구독하는 `listener`배열에 받은 `fn`를 저장해 `dispatch`시 마다 실행한다.

- reudx.js

```javascript
export function createStore(reducer) {
  let state;
  const listeners = [];

  const publish = () => {
    listeners.forEach(({ subscriber, context }) => {
      subscriber.call(context);
    });
  };

  const dispatch = (action) => {
    state = reducer(state, action);
    publish();
  };

  const subscribe = (subscriber, context = null) => {
    listeners.push({
      subscriber,
      context,
    });
  };

  return {
    dispatch,
    getState,
    subscribe,
  };
}

export function actionCreator(type, payload = {}) {
  return {
    type,
    payload: { ...payload },
  };
}
```

- index.js
  > 실제 Redux의 reducer 작성

```javascript
import { createStore, actionCreator } from "./redux";

const INIT = "init";
const INCREMENT = "increment";
const DECREMENT = "decrement";
const RESET = "reset";

function reducer(state = {}, { type, payload }) {
  switch (type) {
    case INIT:
      return {
        ...state,
        count: payload.count,
      };

    case INCREMENT:
      return {
        ...state,
        count: state.count ? state.count + 1 : 1,
      };

    case DECREMENT:
      return {
        ...state,
        count: state.count ? state.count - 1 : 0,
      };

    case RESET:
      return {
        ...state,
        count: 0,
      };

    default:
      return {
        ...state,
      };
  }
}
```

> - createStore 함수로 store 생성 <br/>
> - getState 함수를 사용해 값을 출력하는 update함수 작성<br/>
> - store의 subscribe함수를 이용해 update함수로 구독<br />

```javascript
const store = createStore(reducer);

function update() {
  console.log(store.getState());
}

store.subscribe(update);
```

> action을 조금 더 쉽게 dispatch 하기 위해 함수들 작성

```javascript
function init(count) {
  store.dispatch(actionCreator(INIT, { count }));
}

function increment() {
  store.dispatch(actionCreator(INCREMENT));
}

function decrement() {
  store.dispatch(actionCreator(DECREMENT));
}

function reset() {
  store.dispatch(actionCreator(RESET));
}
```

> - type과 payload를 포함해 action을 dispatch <br/>
> - actionCreator를 사용해 action을 반환한 후 dispatch<br/>
> - payload를 전달받아 dispatch 해주는 함수 사용<br/>

```javascript
store.dispatch({ type: INIT, payload: { count: 0 } });
store.dispatch({ type: INCREMENT });
store.dispatch({ type: DECREMENT });
store.dispatch({ type: RESET });

store.dispatch(actionCreator(INIT, { count: 0 }));
store.dispatch(actionCreator(INCREMENT));
store.dispatch(actionCreator(INCREMENT));
store.dispatch(actionCreator(RESET));

init(5);
increment();
decrement();
reset(0);
```
