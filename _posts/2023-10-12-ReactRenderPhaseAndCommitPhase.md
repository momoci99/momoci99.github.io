---
layout: single
title: "[React] React Render Phase & Commit Phase"
tag:
  - React
  - Translate
---

React는 구성요소가 화면에 실제 표시되기 전에 react에 의해 렌더링되는 과정이 필요합니다. 해당 프로세스의 과정을 이해하면 코드가 어떻게 실행되고 동작을 설명하는지 생각하는데 도움이 됩니다.

## 렌더링 과정을 주방과 요리사로 표현

컴포넌트가 주방에서 요리사가 되어 재료로 맛있는 요리를 만들고 있다고 상상해봅시다. 이 시나리오에서 React는 고객의 요청을 접수하고 주문을 전달하는 웨이터 입니다. UI를 요청하고 제공하는 프로세스는 세 단계로 구성됩니다.

```markdown
1. 렌더링 트리거 (손님의 주문을 주방으로 전달)
2. 컴포넌트 렌더링 (주방에서 주문 준비)
3. DOM에 커밋 (테이블 위에 주문 배치)
```

## 1단계 : 렌더링 트리거

컴포넌트가 렌더링 되는 데는 두 가지 이유가 있습니다.

1. 컴포넌트의 초기 렌더링일때.
2. 컴포넌트(혹은 상위 항목 중 하나)의 상태가 업데이트 되었을때.

### 초기 렌더링

앱이 시작되면 초기 렌더링을 트리거 해야합니다. 프레임워크와 샌드박스는 때때로 이 코드를 숨기지만, 이는 대상 DOM노드로 createRoot를 호출한 다음, 구성요소로 render 메서드를 호출하여 수행됩니다.

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-12-ReactRenderPhaseAndCommitPhase/Untitled.png?raw=true)

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-12-ReactRenderPhaseAndCommitPhase/Untitled%201.png?raw=true)

root.render 코드를 주석 처리하면 컴포넌트가 사라지게 됩니다.

### 상태(state)가 업데이트 되었을 때의 Re-render

컴포넌트가 처음 렌더링 되면, 상태를 set 함수를 통해 추가적인 렌더링을 트리거 할 수 있습니다. 구성 요소의 상태를 업데이트하면 렌더링이 자동으로 대기열에 추가됩니다. (식당 손님이 갈증이나 배고픈 상태에 따라 차, 디저트 등을 먼저 주문한 후 주문하는 모습을 상상해 보세요)

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-12-ReactRenderPhaseAndCommitPhase/Untitled%202.png?raw=true)

### 2단계:React가 구성 요소를 렌더링 합니다.

렌더링을 트리거한 후 React는 컴포넌트를 호출하여 화면에 표시할 내용을 파악합니다. “렌더링”은 React가 구성 요소를 호출하는 것입니다.

- 초기 렌더링시 React는 루트 구성 요소를 호출합니다.
- 후속 렌더링의 경우 React는 상태 업데이트가 렌더링을 트리거한 함수 구성 요소를 호출합니다.

이 프로세스는 재귀적입니다. 업데이트된 컴포넌트가 다른 컴포넌트 요소를 반환하면 React는 해당 컴포넌트를 다음에 렌더링하고 해당 컴포넌트도 무언가를 반환하면 해당 컴포넌트를 다음에 렌더링하는 식입니다. 더 이상 중첩된 구성 요소가 없고 React가 화면에 표시해야 할 내용을 정확히 알 때까지 프로세스가 계속됩니다.

다음 예제에서는 React는 `Gallery()` 와 `Image()` 를 여러번 호출합니다.

Gallery.js

```jsx
export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

index.js

```jsx
import Gallery from "./Gallery.js";
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(<Gallery />);
```

result

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-12-ReactRenderPhaseAndCommitPhase/Untitled%203.png?raw=true)

- **초기 렌더링** 중에는 React는 `<section>`, `<h1>`, `<img>` tag에 대한 [DOM 노드들을 생성](https://developer.mozilla.org/ko/docs/Web/API/Document/createElement)합니다.
- 리렌더링 과정중에는 React는 이전 렌더링 이후 변경된 속성이 있는지 계산합니다. 다음 단계인 커밋 단계까지 해당 정보로 아무 작업도 수행하지 않습니다.

> **함정(Pitfall)**

동일한 입력, 동일한 출력 (**Same inputs, same output)**

동일한 입력이 주어지면 컴포넌트는 항상 동일한 JSX를 반환해야 합니다. (토마토 샐러드를 주문할 때와 양파 샐러드를 받아서는 안됩니다)

컴포넌트는 컴포넌트 자신의 것만 생각합니다 (**It minds its own business)**

렌더링 전에 존재했던 객체나 변수를 변경해서는 안됩니다 (하나의 주문이 다른사람의 주문을 변경해서는 안됩니다.)

그렇지 않으면 코드베이스가 복잡해지면서 혼란스러운 버그와 예측할 수 없는 동작이 발생할 수 있습니다. “[엄격 모드](https://react.dev/reference/react/StrictMode)” 로 개발할 때 React는 각 구성 요소의 함수를 두 번 호출하여 불순한 함수로 인한 실수를 표면화하는데 도움이 될 수 있습니다.

## 3단계 : React는 DOM에 변경 사항을 커밋합니다.

컴포넌트를 렌더링(호출)한 후 React는 DOM을 수정합니다.

- 초기 렌더링의 경우 React는 [appendChild()](https://developer.mozilla.org/ko/docs/Web/API/Node/appendChild) DOM API를 사용하여 생성된 모든 DOM 노드를 화면에 배치합니다.
- 재렌더링의 경우 React는 DOM이 최신 렌더링 출력과 일치하도록 최소한의 필수 작업(렌더링 중에 계산됨!) 을 적용합니다.

React는 렌더링 간에 차이가 있는 경우에만 DOM 노드를 변경합니다. 예를 들어 매초 부모로부터 전달된 다른 props로 다시 렌더링하는 구성요소가 있습니다. 텍스트를 `<input>` 에 추가하여 값을 업데이트할 수 있지만 구성 요소가 다시 렌더링 될 때 텍스트가 사라지지 않는 방법에 주목하세요.

Clock.js

```jsx
export default function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>
      <input />
    </>
  );
}
```

이는 마지막 단계에서 React가 `<h1>` 의 내용을 새 시간으로 업데이트만 하기 때문에 동작합니다. JSX에 `<input>`이 지난번과 같은 위치에 표시되므로 React는 `<input>`이나 그 값을 건드리지 않습니다.

## 에필로그 : 브라우저 paint

렌더링이 완료되고 React가 DOM을 업데이트한 후 브라우저는 화면을 다시 칠합니다. 이 프로세스를 “브라우저 렌더링(browser rendering)”이라고 하지만 문서 전체에서 혼동을 피하기 위해 “페인팅(painting)”이라고 부르겠습니다.

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-12-ReactRenderPhaseAndCommitPhase/Untitled%204.png?raw=true)

## 요약

- React 앱의 모든 화면 업데이트는 세 단계로 이루어집니다.
  1. 트리거 (Trigger) - 렌더링을 유발
  2. 렌더 (Render) - 컴포넌트 렌더링
  3. 커밋 (Commit) - DOM에 변경사항을 반영
- 엄격 모드를 사용하여 구성 요소의 실수를 찾을 수 있습니다.
- 렌더링 결과가 지난번과 같으면 React는 DOM을 건드리지 않습니다.

참고 문서

---

- **Render and Commit** https://react.dev/learn/render-and-commit

여담

- 기술 면접에서 react render phase와 commit phase를 반대로 설명했더군요 🤯 다음에는 이런 실수를 하지 않기 위해 정리합니다.📝
