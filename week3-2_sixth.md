# 우아한 테크러닝 6회차

## 강의목표

- webpack 이해
- React & Typescript 살펴보기
- Redux-saga 사용하기

## 강의

### Webpack

`webpack`은 일반적으로 `webpack.config.js`라는 이름을 갖는 설정파일을 갖는다. 이름이 변경될 수 있지만 관례적으로 사용된다. `webpack`은 `node`에서 실행되기 때문에 `require`를 사용한다.

```javascript
const webpack = require("webpack");
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const { TsconfigPathsPlugin } = require("tsconfig-paths-webpack-plugin");
```

설정 값을 담은 객체를 생성하기 위해서는 `config`객체를 생성하고 모듈로 내보낸다.

```javascript
const config = {
...
}
module.exports = config;
```

`webpack`의 설정의 `entry`는 `weboacj`실행 시 진입점을 가리키는 파일을 작성하게 되는데, `core-js`는 polyfill라이브러리이며 하위 브라우저의 문법과 관련 호환성을 위해 사용한다.

```javascript
const config = {
  // ...
  entry: {
    main: ["core-js", "./src/index.tsx"],
  },
  // ...
};
```

자세한 속성들은 [webpack 공식문서](https://webpack.js.org/concepts/)를 참조하자.

### Loader

`loader`는 리덕스의 미들웨어와 같은 역할을 한다고 생각하면 된다.
`entry`를 통해 읽어들인 파일은 `loader`로 넘어가게 되며 각각의 `loader`의 역할을 수행한다.<br />
`loader`는 설정 파일의 `module`속성에 작성되고 `rules`배열안에 객체로 작성된다.

```javascript
const config = {
  // ...
  module: {
    rules: [
      {
        test: /\.(ts|js)x?$/,
        include: path.resolve("src"),
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
        },
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        loader: "file-loader",
        options: {
          name: "[name].[ext]",
          outputPath: "./images/",
        },
      },
    ],
  },
  // ...
};
```

### Plugin

`plugin`은 `loader`보다 훨씬 복잡하다. `webpack`의 안쪽에 있는 Low-level의 기능들이 `plugin`한테 노출을 시켜주기 때문에 `plugin`이 훨씬 더 많은 일을 할 수 있다.

```javascript
const config = {
  // ...
  plugins: [
    new webpack.SourceMapDevToolPlugin({}),
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, "public/index.html"),
    }),
    new webpack.DefinePlugin({
      "process.env.NODE_ENV": JSON.stringify(process.env.NODE_ENV),
    }),
  ],
  // ...
};
```

보통 `plugin`은 `loader`가 모두 실행된 후 실하게 된다. 그렇기 때문에 `loader`는 컨버팅이고 `plugin`은 후처리라고 생각하면 된다.

### React & Typescript 살펴보기

`redux`는 특정 프레임워크나 라이브러리에 종속되어 있는 상태관리자가 아니다. 그래서 `vanilla.js`에서도 사용 가능하고 원한다면 `Vue.js`,`Angular.js`에서도 사용 가능하다.<br />
아래와 같이 앞에서 구혀냏 보았던 함수들과 같은 기능을 함수로 가져와 사용할 수 있다.

```javascript
import { createStore, applyMiddleware } from "redux";
```

`redux`를 `React`에서 직접 사용하려면 `react-redux`를 사용할 수 있다.

```javascript
import { Provider } from "react-redux";
// ...
render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
);
```

`react-redux`는 `ContextAPI`형식으로 구성된 `Provider`를 제공하며 위와 같이 사용할 수 있다.
`reducer`는 초기 상태값을 나타낼 인터페이스이다.

```javascript
interface StoreState {
  monitoring: boolean;
  success: number;
  failure: number;
}

const initialState: StoreState = {
  monitoring: false,
  success: 0,
  failure: 0,
};
```

### typesafe-actions

`typesafe-actions`라이브러리의 `createAction`을 사용해 `actions`를 생성할 수 있다.

```javascript
import { createAction } from "typesafe-actions";

export const startMonitoring = createAtion(
  "@command/monitoring/start",
  (resolve) => {
    return () => resolve();
  }
);

export const stopMonitoring = createAtion(
  "@command/monitoring/stop",
  (resolve) => {
    return () => resolve();
  }
);

export const fetchSuccess = createAction("@fetch/success", (resolve) => {
  return () => resolve();
});

export const fetchFailure = createAction("@fetch/failure", (resolve) => {
  return () => resolve();
});
```

`typesafe-actions`를 사용하게 되면서 상수를 직접 만들지 않고 액션을 정의할 수 있게 되었다.

```javascript
import { ActionType, getType } from "typesafe-actions";
import * as Actions from "../actions";

// ...
// 상태 인터페이스 및 상태 선언
// ...

export default (
  state: StoreState = initializeState,
  action: ActionType<typeof Actions>
) => {
  switch (action.type) {
    case getType(Actions.fetchSuccess):
      return {
        ...state,
        success: state.success + Math.floor(Math.random() * (100 - 1) + 1),
      };
    case getType(Actions.fetchFailure):
      return {
        ...state,
        failure: state.failure + Math.floor(Math.random() * (2 - 0)),
      };
    default:
      console.log(action.type);
      return Object.assign({}, state);
  }
};
```

`reducer`를 작성하기 위해 `getType`함수를 이용해 가져온 액션의 타입을 가져올 수 있다.

### Redux-saga

현재 미들웨어로 `redux-saga`를 사용하고 있다.

```javascript
import * as React from "react";
import { render } from "react-dom";
import { Provider } from "react-redux";
import { createStore, applyMiddleware } from "redux";
import { StoreState } from "./types";
import reducer from "./reducers";
import createSagaMiddleware from "redux-saga";
import rootSaga from "./sagas";
import App from "./App";

const sagaMiddleware = createSagaMiddleware();

const store: StoreState = createStore(reducer, applyMiddleware(sagaMiddleware));
const rootElement: HTMLElement = document.getElementById("root");

sagaMiddleware.run(rootSaga);

render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
);
```

`redux-saga`는 비동기 처리를 위해 제너레이터를 사용한다.

```javascript
export default function* () {
  yield fork(monitoringWorkflow);
}
```

`redux-saga/effects`패키지에 있는 헬퍼 함수들이다.

```javascript
import { fork, all, take, race, delay, put } from "redux-saga/effects";
```

헬퍼함수들의 결과는 객체다.
각각의 함수들은 특정 역할을 가지고 있으며 `redux-saga` 스펙에 맞게 사용하면 된다.

```javascript
function* monitoringWorkflow() {
  while (true) {
    yield take(getType(Actions.startMonitoring));

    let loop = true;

    while (loop) {
      yield all([
        put({ type: getType(Actions.fetchSuccess) }),
        put({ type: getType(Actions.fetchFailure) }),
      ]);

      const { stoped } = yield race({
        waiting: delay(200),
        stoped: take(getType(Actions.stopMonitoring)),
      });

      if (stoped) {
        loop = false;
      }
    }
  }
}
```

`fork`함수에 들어간 `monitoringWorkflow`제너레이터는 위와 같이 작성되어 있다. <br />
`put`함수는 액션을 `dispatch`하는 것을 `redux-saga`에게 시키고 `all`을 사용해 여러개의 액션을 `dispatch`한다.<br />
`take`는 리덕스의 액션(문자열)을 주고 이 문자열에 액션이 들어오면 알려달라고 `next`를 던져준다.<br />
`delay`는 `Promise`와 같이 사용되며 특정 시간을 기다린 후 다시 코드가 실행되게 해준다.<br />
`race`는 인자 객체의 값중 먼저 응답이 온 값을 반환하는 역할을 한다.

```javascript
const { stoped } = yield race({
  waitting: delay(200),
  stoped: take(getType(Actions.stopMonitoring))
});
```
