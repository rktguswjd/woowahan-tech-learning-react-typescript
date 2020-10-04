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
<br/>
<br/>

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

myMiddleware와 yourMiddleware는 문법만 다르고 같은 형태이며 ourMiddleware은 함수가 하나다. 이 세 함수의 공통점은 실행하는 코드가 같으며 차이점은 중첩되어 있는 지점이 다르다는 것이다.
인자가 n개인 함수를 인자를 각각 하나씩 쪼갠 함수를 분리하는 기법을 함수형 프로그래밍에서는 커링이라고 한다.

```javascript
myMiddleware(store)(store.dispatch)({ type: "inc" }); // Object {counter: 1}
yourMiddleware(store)(store.dispatch)({ type: "inc" }); // Object {counter: 2}
ourMiddleware(store, store.dispatch, { type: "inc" }); // Object {counter: 3}
```

`inc` 액션이 동기적으로 처리되며 `store`의 `counter`가 1씩 증가된다. 그렇다면 왜 커링을 사용해 미들웨어를 작성했을까?
`reducer`에서 로깅을 하려고 할 때 `reducer`를 직접 접근할 수 없을 때 아래와 같이 로깅할 수 있다.

```javascript
console.log('action -> { type: "inc" }');
store.dispatch({ type: "inc" });

console.log('action -> { type: "inc" }');
store.dispatch({ type: "inc" });
```

액션이 `dispatch`되는 것은 로깅할 수 있지만 모든 액션마다 직접 로깅해야한다는 번거로움이 있다. 이 번거로움을 해결하는 방법으로 액션을 함수로 감싸 `dispatch`할 수 있다.

```javascript
function dispatchAndLog(store, action) {
  console.log("dispatching", action);
  store.dispatch(action);
  console.log("next state", store.getState());
}

dispatchAndLog(store, { type: "inc" });
dispatchAndLog(store, { type: "inc" });
```

각각의 액션에 로깅을 직접하지 않고도 간단하게 실행된 액션을 로깅할 수 있다.
<br />
<br />

다음은 디스패치를 몽키패칭 하는 예시이다.

```javascript
let next = store.dispatch;

store.dispatch = function dispatchAndLog(action) {
  console.log("dispatching", action);

  let result = next(action);
  console.log("next state", store.getState());
  return result;
};

store.dispatch({ type: "inc" });
store.dispatch({ type: "inc" });
```

위와 같이 `dispatchAndLog`함수를 사용하지 않고도 `dispatch`를 할 수 있게 된다.
하지만 `dispatch`에 이런 함수변환을 두 개 이상 적용해야 할 때가 생길 수도 있다.
로깅 이외의 오류가 발생할시 콘솔에 값을 출력하는 로직을 생각할 수 있다.

```javascript
function patchStoreToAddLogging(store) {
  let next = store.dispatch;

  store.dispatch = function dispatchAndLog(action) {
    console.log("dispatching", action);
    let result = next(action);
    console.log("next state", store.getState());
    return result;
  };
}

function patchStoreToAddCrashReporting(store) {
  let next = store.dispatch;

  store.dispatch = function dispatchAndReportErrors(action) {
    try {
      return next(action);
    } catch (err) {
      console.error("Caught an exception!", err);
      throw err;
    }
  };
}
```

위와 같이 두 개의 함수는 구조적으로 함수를 반환하는 함수형태로 작성된 것을 확인할 수 있다.

```javascript
patchStoreToAddLogging(store);
patchStoreToAddCrashReporting(store);
```

`store`의 `dispatch`함수에 대해 2개의 함수 변환을 진행할 수 있지만 깔끔하지 않다.
위 코드는 `store`의 `dispatch`메서드를 변경하지만 아래와 같이 함수를 반환해 적용할 수 있다.

```javascript
function logger(store) {
  let next = store.dispatch;

  return function dispatchAndLog(action) {
    console.log("dispatching", action);
    let result = next(action);
    console.log("next state", store.getState());
    return result;
  };
}
```

각각의 미들웨어를 적용시키는 `applyMiddlewareByMonkeypatching`함수를 아래와 같이 작성할 수 있다.

```javascript
function applyMiddlewareByMonkeypatching(store, middlewares) {
  middlewares = middlewares.slice();
  middlewares.reverse();

  middlewares.forEach((middleware) => (store.dispatch = middleware(store)));
}

applyMiddlewareByMonkeypatching(store, [logger, crashReporter]);
```

하지만 위 코드는 몽키패칭을 숨기기만 할 뿐 아직 사용중이다. 몽키패칭을 제거하기 위해 아래와 같이 작성할 수 있다.

```javascript
function logger(store) {
  return function wrapDispatchToAddLogging(next) {
    return function dispatchAndLog(action) {
      console.log("dispatching", action);
      let result = next(action);
      console.log("next state", store.getState());
      return result;
    };
  };
}
```

미들웨어가 `next()`디스패치 함수를 `store`인스턴스에서 읽어오는 대신 매개변수로 받을 수 있다.
화살표 함수를 이용해 커링을 간단하게 나타낼 수 있다.

```javascript
const logger = (store) => (next) => (action) => {
  console.log("dispatching", action);
  let result = next(action);
  console.log("next state", store.getState());
  return result;
};
```

`Redux`의 미들웨어는 위와 같은 모양으로 커링을 사용해 작성한다.
<br/>
<br/>

### Redux에 미들웨어 추가하기

아래와 같이 2개의 미들웨어를 추가한다.

```javascript
const logger = (store) => (next) => (action) => {
  console.log("logger:", action.type);
  next(action);
};

const monitor = (store) => (next) => (action) => {
  setTimeout(() => {
    console.log("monitor:", action.type);
    next(action);
  }, 2000);
};
```

`logger`미들웨어는 동기적으로 실행되지만 `monitor`미들웨어는 비동기적으로 실행된다.
작성한 두 개의 미들웨어를 적용하기 위해 `applyMiddleware`함수를 작성할 수 있다.

```javascript
export function applyMiddleware(store, middlewares = []) {
  middlewares = middlewares.slice();
  middlewares.reverse();

  let dispatch = store.dispatch;
  middlewares.forEach((middleware) => (dispatch = middleware(store)(dispatch)));

  return Object.assign({}, store, { dispatch });
}
```

`middlewares`매개변수를 받으며 기본값을 비어있는 배열이다. 또한 미들웨어를 적용하기 위해 `middlewares`인자로 받은 미들웨어 배열을 `reverse`한다.

```javascript
middlewares.reverse();
```

이는 미들웨어가 최종적으로 실행하는 `dispatch`는 `store`의 `dispatch`함수여야 하기 때문이다. `middleware(store)(lastDispatch)` 형태로 호출되며 반환값은 계속해서 앞에 미들웨어에 전달된다. 그래서 `reverse`로 뒤집어 가장 뒤인 안쪽부터 바깥쪽으로 `dispatch`가 전달된다. 그렇기 때문에 최종 실행되는 `dispatch`함수는 기존 맨 처음의 `sotre`의 `dispatch`함수가 된다.
