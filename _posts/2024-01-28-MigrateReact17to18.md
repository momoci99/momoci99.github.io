---
layout: single
title: "React 17 to 18 migration"
tag:
  - React
---

현재 회사에서 react 17 → react 18 버전으로 마이그레이션을 준비하고 있어 관련 내용을 정리합니다

다루는것

- 클라이언트 사이드에서 마이그레이션시 검토해야하는 부분

다루지 않는것

- 서버사이드 렌더링
- react native

## roots api 변경

- root render 모듈이 react-dom → react-dom/client로 변경
- react 18에서 제공하는 creeatRoot에서 렌더링 성능이 개선되었음.
- creactRoot는 batch 렌더링을 지원.

```jsx
// Before
import { render } from "react-dom";
const container = document.getElementById("app");
render(<App tab="home" />, container);

// After
import { createRoot } from "react-dom/client";
const container = document.getElementById("app");
const root = createRoot(container); // createRoot(container!) if you use TypeScript
root.render(<App tab="home" />);
```

## unmountComponentAtNode → root.unmount 로 변경

- dom에서 react 컴포넌트를 제거하는 api

```jsx
// Before
unmountComponentAtNode(container);

// After
root.unmount();
```

## render함수에 대한 callback 제거

- suspense 동작에 이슈가 있어 변경됨.

```jsx
// Before
const container = document.getElementById("app");
render(<App tab="home" />, container, () => {
  console.log("rendered");
});

// After
function AppWithCallbackAfterRender() {
  useEffect(() => {
    console.log("rendered");
  });

  return <App tab="home" />;
}

const container = document.getElementById("app");
const root = createRoot(container);
root.render(<AppWithCallbackAfterRender />);
```

## Strict Mode 동작 변경

이전보다 더 강력하게 동작이 변경됨

**react 17**

변경 발생시

- 변경된 컴포넌트에 대한 마운트만 다시 수행

```jsx
* React mounts the component.
    * Layout effects are created.
    * Effect effects are created.
```

**react 18**

변경 발생시

- 컴포넌트 mounte
- 컴포넌트 unmount 시뮬레이트
- 이전 상태와 함께 컴포넌트 마운팅

```jsx
* React mounts the component.
    * Layout effects are created.
    * Effect effects are created.
* React simulates unmounting the component.
    * Layout effects are destroyed.
    * Effects are destroyed.
* React simulates mounting the component with the previous state.
    * Layout effect setup code runs
    * Effect setup code runs
```

## 타입스크립트 정의 업데이트

- @types/reac, @types/react-dom 라이브러리에 대한 최신 버전 업데이트가 필요함.
- (그런데 병원 웹은 사용하지 않고 있고, 파트너에는 이미 최신 메이저 버전(18)을 사용중임)
- props 정의시 children에 대한 props를 명시적으로 지정해주어야함.

```jsx
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

## 자동 batching

**react 17**

- react 이벤트 핸들러 내부에서만 일괄 처리 수행
- Promise, setTimeout, 기본 이벤트 핸들러 or 기타 이벤트 내부에서의 상태 업데이트는 react에서 일괄처리하지 않음

```jsx
// Before React 18 only React events were batched

function handleClick() {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // setState는 2번 정의되어있으나 한번만 re-render 수행(일괄처리)
}

setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // re-redenr를 두번 수행(일괄처리 안됨)
}, 1000);
```

**react 18**

- createRoot 내부에서 발생하는 모든 상태 업데이트는 일괄처리됨.

```jsx
// react 18 및 그 이후버전 부터는 timeouts, promises, native event를 비롯한
// 모든 이벤트들은 일괄처리됩니다.

function handleClick() {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // setState는 2번 정의되어있으나 한번만 re-render 수행(일괄처리)
}

setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  //  setState는 2번 정의되어있으나 한번만 re-render 수행(일괄처리)
}, 1000);
```

## 새로운 api

**useSyncExternalStore**

- 외부 스토어를 연동하기 위함 hook. 그러나 대부분의 상태 관리 라이브러리에서는 이미 사용중

**useInsertionEffect**

- css-in-js에서 렌더링 도중에 스타일이 업데이트 되는 경우 리엑트에서 렌더링을 멈추고 브라우저가 레이아웃을 다시 계산할 수 있도록 함.
- styled-component를 위한 전용 hook

**IE 를 더이상 지원하지 않음**

- react 17까지만 IE를 지원.

Deprecations

- `react-dom`: `ReactDOM.render` has been deprecated. Using it will warn and run your app in React 17 mode.
- `react-dom`: `ReactDOM.hydrate` has been deprecated. Using it will warn and run your app in React 17 mode.
- `react-dom`: `ReactDOM.unmountComponentAtNode` has been deprecated.
- `react-dom`: `ReactDOM.renderSubtreeIntoContainer` has been deprecated.
- `react-dom/server`: `ReactDOMServer.renderToNodeStream` has been deprecated.

다른 Breaking Changes

- 컴포넌트에서 undefined를 반환해도 warn이 노출되지 않고 렌더링 될 수 있도록 변경
- react-testing-library에서 e2e테스트를 실행할때 act warning이 더이상 발생하지 않음. 필요할때만 발생시키도록 환경변수 제공
- unmounted된 컴포넌트에서 setState호출시 warning 미출력으로 변경.
  - React 17까지는 unmounted된 컴포넌트에서 setState 호출시 메모리 누수에 대해 경고를 하였으나 불필요한것으로 판단하여 제거됨
- Strict모드에서 console.log가 두번씩 호출되는부분을 React 17에서는 한번씩으로 억제하는 기능이 있었으나 혼란을 야기한다는 의견을 바탕으로 해당기능을 제거함 (console.log가 두번씩 노출 in strict모드)
- mounted 해제시 더 많은 내부 필드를 제거하여 잠재적인 메모리 누수 감소.

참고 자료

- https://react.dev/blog/2022/03/08/react-18-upgrade-guide
- [https://velog.io/@code-bebop/react-17에서-18으로-마이그레이션](https://velog.io/@code-bebop/react-17%EC%97%90%EC%84%9C-18%EC%9C%BC%EB%A1%9C-%EB%A7%88%EC%9D%B4%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%85%98)
- https://react.dev/blog/2022/03/08/react-18-upgrade-guide#updates-to-strict-mode
- https://github.com/reactwg/react-18/discussions/5
