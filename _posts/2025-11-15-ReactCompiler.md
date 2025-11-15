---
layout: single
title: "[React] React Compiler 가볍게 살펴보기"
tag:
  - React
  - React Compiler
---

# React Compiler

- 컴포넌트와 훅을 자동으로 최적화(메모이제이션 Memoization)
- `useMemo`, `useCallback`, `memo` 사용에 대해 더이상 신경쓰지 않아도 됨.
- 선택적으로 컴파일이 가능합니다.

# 준비

react compiler를 사용하기 위해 react 버전 확인, eslint 플러그인, babel 플러그인 설정이 필요.

### 최소 지원 버전

- react compiler는 17버전부터 지원하며, 19버전에서 가장 잘 동작.

### eslint-plugin-react-hooks

- react의 기본적인 rule 위반 사항을 체크하는 lint
- react compiler는 규칙을 위반한 컴포넌트는 컴파일 대상에서 제외.
- npm [링크](https://www.npmjs.com/package/eslint-plugin-react-hooks)

### eslint-plugin-react-compiler

- react-compiler 기준으로 rule 위반사항을 체크하는 lint (기본 react rule보다 더 강하게 체크)
- 마찬가지로 react compiler는 규칙을 위반한 컴포넌트는 컴파일 대상에서 제외.
- `eslint-disable` 구문이 있는 컴포넌트도 제외.
- npm [링크](https://www.npmjs.com/package/eslint-plugin-react-compiler/v/19.1.0-rc.2)

![image.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/refs/heads/master/assets/img/2025-11-15-ReactCompiler/image.png)

### babel-plugin-react-compiler

- react-compiler가 “컴포넌트나 훅이 어떻게 렌더되고 언제 재렌더되는가”를 정적 분석하기 위한 플러그인.
- npm [링크](https://www.npmjs.com/package/babel-plugin-react-compiler)

# 예시

매우 무거운 연산을 실행하는 컴포넌트

```tsx
export default function SlowComponent({ value }: { value: number }) {
  console.log("SlowComponent rendered with", value);
  // 매우 느린 계산 시뮬레이션
  let sum = 0;
  for (let i = 0; i < 100000000; i++) {
    sum += i * value;
  }
  return <div>Computed: {sum}</div>;
}
```

`SlowComponent` 를 렌더링하는 부모 컴포넌트

```tsx
import { useState } from "react";
import SlowComponent from "./SlowComponent";

export default function SlowComponentWrapperWithCompiler() {
  const [count, setCount] = useState(0);
  const [value, setValue] = useState(1);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount((c) => c + 1)}>
        Increment count (SlowComponent에 영향을 주지 않는 상태 변화)
      </button>

      <h1>Value: {value}</h1>
      <button onClick={() => setValue((v) => v + 1)}>
        Increment value (SlowComponent에 영향을 주는 상태 변화(props 전달)
      </button>

      <SlowComponent value={value} />
    </div>
  );
}
```

`SlowComponent` 를 렌더링 하지만, `"use no memo"` 지시자를 사용하여 컴파일에서 제외된 부모 컴포넌트

```tsx
"use no memo"; // 컴포넌트가 React 컴파일러에 의해 컴파일되지 않도록 제외합니다.

import { useState } from "react";
import SlowComponent from "./SlowComponent";

export default function NoCompilerSlowComponentWrapper() {
  const [count, setCount] = useState(0);
  const [value, setValue] = useState(1);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount((c) => c + 1)}>
        Increment count (SlowComponent에 영향을 주지 않는 상태 변화)
      </button>

      <h1>Value: {value}</h1>
      <button onClick={() => setValue((v) => v + 1)}>
        Increment value (SlowComponent에 영향을 주는 상태 변화(props 전달)
      </button>

      <SlowComponent value={value} />
    </div>
  );
}
```

## 컴파일 결과

- 컴파일을 통해 자동 최적화에 성공한 컴포넌트는 react dev tool에 “Memo ✨” 아이콘 표시

![image.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/refs/heads/master/assets/img/2025-11-15-ReactCompiler/image%201.png)

## 진짜 잘 메모이제이션 되었는가?

### 최적화된 컴포넌트의 상태변화

![image.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/refs/heads/master/assets/img/2025-11-15-ReactCompiler/image%202.png)

![image.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/refs/heads/master/assets/img/2025-11-15-ReactCompiler/image%203.png)

- `const [count, setCount] = useState(0);` 의 count가 0 → 1로 변경되는 과정에서 자식 컴포넌트 (SlowComponent) 는 리렌더링 되지 않았음. (최적화됨)

### 최적화안된 컴포넌트의 상태변화

![image.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/refs/heads/master/assets/img/2025-11-15-ReactCompiler/image%204.png)

![image.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/refs/heads/master/assets/img/2025-11-15-ReactCompiler/image%205.png)

![image.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/refs/heads/master/assets/img/2025-11-15-ReactCompiler/image%206.png)

- 부모가 가진 상태 `const [count, setCount] = useState(0);` 가 0 → 1로 변화하면서 자식 컴포넌트가 리렌더링 되는것을 볼 수 있다.

# 지시자

- `use memo` : 해당 지시자가 있는 함수, 컴포넌트는 일단 컴파일 시도
- `use no memo` : 해당 지시자가 있는 함수, 컴포넌트는 어떤 컴파일 모드에서든 컴파일 시도 X
  - 최적화시 의도하지 않은 문제가 예상되는 컴포넌트에 사용.
- 상세 설명 : https://react.dev/reference/react-compiler/directives

# 컴파일모드

- react-compiler는 여러 모드를 지원.
- 문서 : [compilationmode](https://react.dev/reference/react-compiler/compilationMode#compilationmode)

### 타입

`'**infer**' | 'syntax' | 'annotation' | 'all'`

### 기본값

`'infer'`

### 옵션별 설명

- `'infer'` (**기본값**)
  - 컴파일러가 스마트한 휴리스틱(heuristics)을 이용해 컴포넌트나 훅을 식별.
    - `"use memo"` 지시자가 붙은 함수들.
    - PascalCase(첫 글자 대문자) 형태 이름 + JSX를 반환하거나 훅(Hooks)을 호출하는 함수.
- `'annotation'`
  - `"use memo"` 지시자가 **명시적으로 붙은** 함수만 컴파일.
  - 점진적(migration)으로 적용할 때 적합.
- `'syntax'`
  - Flow 문법을 사용하는 코드베이스일 때만 컴파일.
  - 예컨대 Flow의 `component`/`hook` 구문을 사용한 경우.
- `'all'`
  - 사용을 권장하지 않음.
  - 최상위(top-level) 함수 전부를 컴파일 대상에 포함.
  - **주의**: 유틸 함수 같은 일반 함수까지 컴파일되어 성능에 부정적 영향을 줄 수 있음.
- 컴파일 옵션 적용 예시
  ```tsx
  //vite.config.ts
  export default defineConfig({
    plugins: [
      react({
        babel: {
          plugins: [
            [
              "babel-plugin-react-compiler", // 다른 플러그인보다 먼저 와야함.
              {
                compilationMode: "infer",
              },
            ],
          ],
        },
      }),
    ],
  });
  ```

# 성능이 개선되는가?

- 그럴수도 있고 아닐수도 있음.
- 이미 메모이제이션이 잘 적용된 프로젝트는 성능 개선이 미비함.
- 메모이제이션이 잘 되지 않은 프로젝트에서는 성능 개선 효과를 볼 수 있음.

# 반드시 사용해야하나?

- react 공식 문서에도 점진적인 도입을 권장

  ```
  자바스크립트의 유연한 특성으로 인해 컴파일러가 가능한 모든 위반 사항을 잡아내지는 못하며,
  가끔 거짓 음성False Negatives으로 컴파일할 수 있습니다.
  즉, 컴파일러는 React의 규칙을 위반하는 컴포넌트나 Hook을 실수로 컴파일할 수 있어
  정의되지 않은 동작으로 이어질 수 있습니다.

  따라서 기존 프로젝트에서 컴파일러를 성공적으로 도입하려면,
  먼저 제품 코드의 작은 디렉토리에서 실행해 보는 것이 좋습니다.
  이를 위해 컴파일러를 특정 디렉토리 집합Set에서만 실행하도록 구성할 수 있습니다.
  ```

- 지금은 아니더라도 점진적으로 도입은 해야한다고 생각.
- 당장 준비가 어렵다면 eslint 플러그인으로 규칙 위반부터 하나씩 수정해나가는것도 방법

참고한문서

- (번역) 리액트 컴파일러 v1.0 **:** [링크](https://ykss.netlify.app/translation/react_compiler/?utm_source=substack&utm_medium=email)
- 리액트 컴파일러 : [링크](https://ko.react.dev/learn/react-compiler)

---
