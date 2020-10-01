# 우아한 테크러닝 5회차

## 강의목표

- javascript로 만든 Redux 리뷰
- Redux 비동기
- Redux 미들웨어

## 강의

### Redux 리뷰

2회차 때 만든 `redux`를 사용하여 `store`와 `reducer`를 생성한다.

- `index.js`

```javascript
import { createStore } from "./redux";

function reducer(state = { counter: 0 }, action) {
  switch (action.type) {
    case "INC":
      return {
        ...state,
        counter: state.counter + 1,
      };
    default:
      return { ...state };
  }
}

const store = createStore(reducer);

store.subscribe(() => {
  console.log(store.getState());
});

store.dispatch({
  type: "inc",
});

store.dispatch({
  type: "inc",
});
```

- `redux.js`

```javascript
export function createStore(reducer) {
  let state;
  const listeners = [];
  const getState = () => ({ ...state });

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach((fn) => fn());
  };

  const subscribe = (fn) => {
    listeners.push(fn);
  };

  return {
    getState,
    dispatch,
    subscribe,
  };
}
```

`index.js`에서 이루어지는 모든 동작은 각각 따로 일어나는데 실제로 모든 동작은 동기적으로 작동한다. 그렇기 때문에 순서가 완전히 보장된다.
그래서 `reducer`는 순수함수 여야만 한다.

> 순수함수란 외부와 아무런 `dependency`없이 내부적으로 아무런 `side effect`없이 작동하는 함수<br />
> 멱등성이란 연산을 여러 번 적용하더라도 결과가 달리지지 않는 성질<br />

실행될 때 마다 결과가 일정하지 않은 작업들을 순수하지 않은 작업이라고 한다. 대표적으로 비동기 작업이고 그 중 API호출이다. 그렇기 때문에 결과 예측이 안된다.
현재 상태에서 API를 다루는 것은 불가능하다

- `index.js`

```javascript
function api(url, cb) {
  setTimeout(() => {
    cb({ type: "응답이야", data: [] });
  }, 2000);
}
...
function reducer(state = { counter: 0 }, action) {
  switch (action.type) {
    case "inc":
      return {
        ...state,
        counter: state.counter + 1
      };
    case "fetch-user":
      api("/api/v1/users/1", (users) => {
        return { ...state, ...users };
      });
      break;
    default:
      return { ...state };
  }
}
...
store.dispatch({
  type: "fetch-user"
})
```

`redux`는 모든 로직을 동기적으로 처리하기 때문에 값을 기다리지 않는다. 그렇기 때문에 `reducer`가 호출되고 `api`가 호출되고 반환하지만 현재 상태에서는 반환한 상태가 없다. 그렇기 때문에 `store`의 상태를 바꾸는 것이 불가능하다.

따라서 redux는 미동기 작업을 할 때 미들웨어를 사용한다.

### Redux 미들웨어

```javascript
const myMiddleware = (store) => (dispatch) => (action) => {
  dispatch(action);
};

function yourMiddleware(store) {
  return function (dispatch) {
    return function (action) {
      dispatch(action);
    };
  };
}

function ourMiddleware(store, dispatch, action) {
  dispatch(action);
}
```

myMiddleware와 youtMiddleware는 문법만 다르고 같은 형태이며 ourMiddleware은 함수가 하나다. 이 세 함수의 공통점은 실행하는 코드가 같으며 차이점은 중첩되어 있는 지점이 다르다는 것이다.
인자가 n개인 함수를 인자를 각각 하나씩 쪼갠 함수를 분리하는 기법을 함수형 프로그래밍에서는 커링이라고 한다.

```javascript
myMiddleware(store)(store.dispatch)({ type: "inc" }); // Object {counter: 1}
yourMiddleware(store)(store.dispatch)({ type: "inc" }); // Object {counter: 2}
ourMiddleware(store, store.dispatch, { type: "inc" }); // Object {counter: 3}
```
