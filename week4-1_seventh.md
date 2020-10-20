# 우아한 테크러닝 7회차

## 강의목표

- 컴포넌트 분리
- React & TypeScript 예제 훑어보기

## 강의

### 컴포넌트 분리

컴포넌트가 **외부 상태에 의존적인 것**과 **의존적이지 않은 것**을 분리하는 것은 `react`의 원칙적인 것이다. 외부상태는 컴포넌트의 상태와 관련된 **비즈니스 로직**을 의미한다.

```javascript
export const Maybe: React.FC<IMaybeProps> = ({ test, children }) => (
  <React.Fragment>{test ? children : null}</React.Fragment>
);
```

위와 같이 컴포넌트 `props`만 받아 사용하는 비즈니스 로직이 벗는 컴포넌트이다. 상태가 존재하지 않는 컴포넌트는 `components`폴더에 주로 작성한다.

```javascript
class OrderStatus extends React.Component<OrderStatusProps> {
  state = {
    errorRate: 0,
  };

  componentDidUpdate(prevProps) {
    if (
      prevProps.success !== this.props.success ||
      prevProps.failure !== this.props.failure
    ) {
      this.setState({
        errorRate:
          this.props.failure > 0
            ? Number((this.props.failure / this.props.success) * 100).toFixed(2)
            : 0,
      });
    }
  }

  render() {
    return <MonitorCard>{/* ... 생략 ... */}</MonitorCard>;
  }
}

export const OrderStatusContiner = connect(mapStateToProps)(OrderStatus);
```

위와 같이 상태를 가지거나 전역 상태와 연결되는 컴포넌트는 `containers`폴더에 주로 작성한다. <br />
컴포넌트는 애플리케이션의 크기에 따라 분할하며 컴포넌트 분할에 너무 집착할 필요는 없다.

### React & TS Boilerplate 예제

아래 `App` 컴포넌트에는 3개의 컨테이너 컴포넌트로 이루어져 있다.

```javascript
import * as React from "react";

import {
  NotificationContainer,
  OrderStatusContiner,
  MonitorControllerContainer,
} from "./containers";
import { Typography } from "antd";

import "antd/dist/antd.css";
import "./sass/main.scss";

export default class App extends React.PureComponent {
  render() {
    return (
      <div>
        <NotificationContainer />
        <header>
          <Typography.Title>React & TS Boilerplate</Typography.Title>
        </header>
        <main>
          <OrderStatusContiner />
          <MonitorControllerContainer />
        </main>
      </div>
    );
  }
}
```

`containers`에는 `index.ts`파일을 가지고 있고, `components`폴더에는 `dex.ts`파일을 가지고 있다.

```javascript
xport * from "./OrderStatus";
export * from "./MonitorController";
export * from "./Notification";
```

위처럼 `index.ts`파일을 구성하게 되면서 컴포넌트 모듈을 편하게 사용하고 관리 할 수 있다. 또한 내부의 정보를 외부에서 감출 수 있다. (캠슐화)

```javascript
import {
  NotificationContainer,
  OrderStatusContiner,
  MonitorControllerContainer,
} from "./containers";
```

### React에서 TypeScript

컴포넌트 `props`의 타입을 지정하는 경우 아래와 같은 형식으로 작성된다.

```javascript
export interface OrderStatusProps {
  showTimeline: boolean;
  success: number;
  failure: number;
  successTimeline: ITimelineItem[];
  failureTimeline: ITimelineItem[];
}

class OrderStatus extends React.Component<OrderStatusProps> {
  // ...
}
```

위 코드는 `interface`에서 논쟁이 있다. 그렇다면 `interface`와 `typealias`중 무엇을 사용할까?

### Interface와 Type Alias

`typescript`에서 타입을 지정할 때 `type`을 이용한 타입 별칭이나 `interface`를 이용할 수 있다.

```javascript
type Person = {
  name: string,
  age: number,
  job?: [],
};

interface Human {
  name: string;
  age: number;
  job?: [];
}
```

위와 같은 `type`이나 `interface`는 컴파일 타임에 적용되는 스펙이다. <br />
아래는 `type`과 `interface`를 트랜스 파일한 결과다.

```javascript
"user strict";

```

트랜스 파일링한 결과 아무것도 나오지 않는다.

```javascript
type Person = {
  name: string,
  age: number,
  job?: [],
};

const p: Person = JSON.parse(prompt("객체를 입력해주세요"));
```

위 코드는 사용자가 어떤 값을 입력할지 예측할 수 없다.
런타임에서 사용자의 입력에 따라 오류가 발생해 문제가 생길 수 있으므로 오류 처리가 필요하다.

### type과 interface의 차이점

`type` 키워드로 선언한 타입 별칭은 아래의 **유니온 타입**을 지원한다.

```javascript
type box = number | string;
let b: box = 1;
b = "box";
```

`interface`는 실제로 **상속을 지원**하지만 유니온 타입은 지원하지 않는다.

```javascript
interface Cat extends Animal {
  // ...
}
```

### 제레릭

[제네릭 공식문서 참고](https://www.typescriptlang.org/docs/handbook/generics.html) <br />
`제네릭`은 동적으로 타입을 지정하는 기능이다.

```javascript
function identity<T>(arg: T): T {
  return arg;
}
```

위의 `identity`함수는 `T`타입의 값이 매개변수를 받아 `T`타입을 반환한다.

```javascript
let stringOutput = identity < string > "string";
let numberOutput = identity < number > 1;

console.log(stringOutput); // string
console.log(numberOutput); // 1
```

TypeScript의 제네릭은 `type`과 `interface`동일하게 컴파일 타임에 작동하는 코드다. <br />
React의 컴포넌트의 제네릭은 아래와 같이 구성되어 있다.

```javascript
React.Component<P, S, SS>
```

`P`는 컴포넌트의 `props`,`S`는 컴포넌트의 `state`, `SS`는 `snapshot`을 의미한다.

### redux와 컴포넌트 연결하기

기존의 `react-redux`에서는 컴포넌트와 `redux`를 연결하기 위해 `component`함수를 사용했다.

```javascript
const mapStateToProps = (state: StoreState) => {
  return {
    monitoring: state.monitoring,
  };
};

const mapDispatchToProps = (dispatch: Dispatch) => ({
  onStart: () => {
    dispatch(startMonitoring());
  },
  onStop: () => {
    dispatch(stopMonitoring());
  },
});

export const MonitorControllerContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(MonitorController);
```

`connect`함수는 `Hook`형태로 작성되어 있으며 2개의 인자를 받는다. 첫 번째 인자는 `mapStateToProps`이며 두 번째 인자는 `mapDispatchToProps`를 받는다. <br />
`mapStateToProps`는 `store`에 존재하는 상태에서 필요한 데이터를 `props`로 전달 받는 함수다.<br />
`mapDispatchToProps`는 `dispatch`할 액션을 반환해 `props`로 전달 받는 함수다. <br />
현재는 `react-redux`에서도 함수형 컴포넌트를 사용하기 위해 `Hooks`를 지원해 쉽게 사용 가능하다.

```javascript
import { useDispatch, useSelector } from "react-redux";

function MonitorController() {
    const dispatch = useDispatch();
    const monitoring  = useSelector(state = > state.monitoring);

    function onStart() {
        dispatch(startMonitoring());
    }

    function onStop() {
        dispatch(stopMonitoring());
    }

    // ...
}

export default MonitorController;
```

위와 같이 `useDispatch`와 `useSelector`를 이용해 기존의 `connect`함수를 대체할 수 있다.

### API 통신

`axios`패키지를 사용하고 있으며 기본적인 인터페이스 정의는 아래와 같이 되어있다.

```javascript
interface IApiSuccessMessage {
  status: string;
}

interface IApiError {
  status: string;
  statusCode: number;
  errorMessage: string;
}

export class ApiError implements IApiError {
  status: string = "";
  statusCode: number = 0;
  errorMessage: string = "";

  constructor(err: AxiosError) {
    this.status = err.response.data.status;
    this.statusCode = err.response.status;
    this.errorMessage = err.response.data.errorMessage;
  }
}

interface INumberOfSuccessfulOrderResponse extends IApiSuccessMessage {
  result: {
    success: number,
  };
}

interface INumberOfFailedOrderResponse extends IApiSuccessMessage {
  result: {
    failure: number,
  };
}

interface IOrderTimelineResponse extends IApiSuccessMessage {
  results: {
    successTimeline: [],
    failureTimeline: [],
  };
}
```

실제 API통신을 진행하는 함수는 아래와 같이 `Promise`형태로 작성했다.

```javascript
export function fetchNumberOfSuccessfulOrder(): Promise<INumberOfSuccessfulOrderResponse> {
  return new Promise((resolve, reject) => {
    axios
      .get(endpoint.orders.request.success({ error: true }))
      .then((resp: AxiosResponse) => resolve(resp.data))
      .catch((err: AxiosError) => reject(new ApiError(err)));
  });
}

export function fetchNumberOfFailedOrder(): Promise<INumberOfFailedOrderResponse> {
  return new Promise((resolve, reject) => {
    axios
      .get(endpoint.orders.request.failure())
      .then((resp: AxiosResponse) => resolve(resp.data))
      .catch((err: AxiosError) => reject(new ApiError(err)));
  });
}

export function fetchOrderTimeline(
  date: string
): Promise<IOrderTimelineResponse> {
  return new Promise((resolve, reject) => {
    axios
      .get(endpoint.orders.request.timeline(date))
      .then((resp: AxiosResponse) => resolve(resp.data))
      .catch((err: AxiosError) => reject(new ApiError(err)));
  });
}
```

API 요청은 모두 `GET`메소드를 사용하고 있으며 요청 정보는 다른 파일에서 관리한다. <br />
요청 정보에 관한 타입은 아래와 같이 `interface`로 정의되어 있다.

```javascript
interface Config {
  orders: {
    request: {
      success(options: { error?: boolean }): string,
      failure(): string,
      timeline(date: string): string,
    },
  };
}

const config: Config = {
  orders: {
    request: {
      success: ({ error = false }) =>
        `${SERVER}/${API_PREFIX}/orders/request/success${
          error ? "?error=random" : ""
        }`,
      failure: () => `${SERVER}/${API_PREFIX}/orders/request/failure`,
      timeline: (date) => `${SERVER}/${API_PREFIX}/orders/request/all/${date}`,
    },
  },
};
```

`orders`객체의 `request`필드는 `success`, `failue`, `timeline`함수를 갖는다. 위 함수는 API 요청을 보낼 URL을 반환하게 된다.
