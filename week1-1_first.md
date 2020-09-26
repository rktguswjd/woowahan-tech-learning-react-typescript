# 우아한 테크러닝 1회차

## 강의목표

- 나는 잘하고 있나?
- 네트워킹
- 코드 품질
- 아키텍처
- 적정기술 -> 면접 ! 정답이 있는가
  <br/>
  <br/>

## 전달하고자 하는 것

- 도구 ( TypeScript, React ,Etc ... )
- 기타등등 : 목표, 바램, 기대
  <br/>
  <br/>

## 4동안의 키워드

- 상태 (State) : 애플리케이션은 상태를 관리하는 것이 주 목표
- 환경 (Env) : 환경에 대한 대응 방법
- 제품 (Product) : 실제로 서비스해가는 것
- 목표 (Mission) : 많은 도구를 조합해 개발하는 제품에 대한 목표 달성 및 그 도구가 원천적으로 해결하기 위한 미션을 이해하기
- 코드 (Quality) : 주관적, 코드의 퀄리티를 높일 수 있는 방법
- 상대적 (E=mc^2) : 이해의 차이 및 적용의 차이
  <br/>
  <br/>

## 도구

- [TypeScript playground](https://www.typescriptlang.org/play)
- [CodeSandbox](https://codesandbox.io/index2)
- [React 공식 문서](https://reactjs.org/)
- [Redux](https://redux.js.org/)
- [MobX](https://mobx.js.org/README.html)
- [Rdux-Saga](https://redux-saga.js.org/)
- [Blueprint](https://blueprintjs.com/)
- [Testing Library](https://testing-library.com/)
  <br/>
  <br/>

---

## 강의

### TypeScript

1. 타입 추론과 명시

```typescript
let foo: number = 10; // 명시적
let foo = 10; // foo는 number로 타입 추론
```

2. Type Alias

```typescript
type Age = number;

let age: Age = 10;
let weight: number = 72;
```

age와 weight 변수는 둘 다 number 타입을 갖지만 가독성의 차이가 존재.

3. type과 Interface (Object)

```typescript
// type
type Age = number;

type Foo = {
  age: Age;
  name: string;
};
const foo: Foo = {
  age: 23,
  name: "rktguswjd",
};
```

```typescript
type Age = number;

interface Bar {
  age: Age;
  name: string;
}
const bar: Bar = {
  age: 23,
  name: "rktguswjd",
};
```

Type Alias와 interface의 사용법은 비슷하지만 미묘한 차이가 있다.
<br/>
<br/>

### React

1. 프로젝트 생성

- CRA
  - react app을 빠르게 구성
  - 다양한 환경 대응이 어려움
  - 프로덕션용으로는 사용안하는 것을 권고

```javascript
yarn create-react-app <app_name> --template typescript
```

<br/>
<br/>

2. Babel <br/>
   최신 문법의 JavaScript를 사용하면 이전 문법으로 변환하여 실행시키는 도구

- App 컴포넌트 변환

```javascript
// App 컴포넌트 변환 전
function App() {
  return <h1 id="header">Hello Tech</h1>;
}
```

```javascript
// App 컴포넌트 변환 후
"use strict";

function App() {
  return /*#__PURE__*/ React.createElement(
    "h1",
    { id: "header" },
    "Hello Tech"
  );
}
```

- createElement

```javascript
React.createElement(type, [props], [...children]);
```

인자로 주어지는 타입에 따라 새로운 React 엘리먼트를 생성하여 반환한다. type 인자로는 태그 이름 문자열, React 컴포넌트, React Fragment 타입 중 하나가 올 수 있다.
JSX로 작성된 코드는 React.createElement()를 사용하는 형태로 변환된다. JSX를 사용할 경우 React.createElement()를 직접 호출하는 일은 거의 없다.
<br/>
<br/>
