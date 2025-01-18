---
layout: single
title: "Prioritized Task Scheduling API 훑어보기"
tag:
  - Web API
  - Prioritized Task Scheduling API
---

`Prioritized Task Scheduling API`는 브라우저에서 태스크 실행을 더 효율적으로 관리할 수 있도록 돕는 Web API입니다.

특징

- 개발자는 태스크를 실행할 때 각 태스크에 우선순위를 지정할 수 있습니다.
- 예를 들어, 사용자 인터페이스 업데이트와 같은 중요한 태스크는 높은 우선순위[`user-blocking`](https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API#user-blocking)를 부여하고, 백그라운드 데이터 동기화는 낮은 우선순위를 부여할 수 있습니다.
- 브라우저는 높은 우선순위의 태스크를 먼저 실행함으로써 사용자 인터페이스의 응답성을 유지합니다.
- 스크롤, 클릭, 입력 등의 사용자 인터랙션이 빠르게 처리되도록 보장합니다.
- 낮은 우선순위 태스크[`background`](https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API#background)는 브라우저가 유휴 상태일 때 실행됩니다.
- 예를 들어, 애니메이션이 끝난 후나 사용자 입력이 없는 상태에서 태스크가 처리됩니다.
- webworker에서도 사용이 가능합니다.

### 다른 스케쥴링 관련 API와의 비교

| API                          | 사용 목적                                | 우선순위 제어      | 대표적 사용 사례                       |
| ---------------------------- | ---------------------------------------- | ------------------ | -------------------------------------- |
| `setTimeout` / `setInterval` | 지정된 시간 후(혹은 간격으로) 작업 실행  | 없음               | 간단한 작업 딜레이, 주기적 작업        |
| `requestAnimationFrame`      | 다음 화면 리프레시 전에 작업 실행        | 렌더링 우선        | 애니메이션 업데이트                    |
| `queueMicrotask`             | 현재 실행 중인 작업 완료 후 실행         | 마이크로태스크     | DOM 업데이트 후 작은 작업 처리         |
| `scheduler.postTask`         | 작업의 우선순위를 명시적으로 설정해 실행 | 우선순위 지정 가능 | 대규모 작업 스케줄링, 비동기 작업 관리 |

# 주요한 api들

- [**`scheduler.yield()`**](https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API#scheduler.yield)
- [**`scheduler.postTask()`**](https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API#scheduler.posttask)
- [\*\*`TaskController`](https://developer.mozilla.org/en-US/docs/Web/API/TaskController)\*\*
- [**`TaskSignal`**](https://developer.mozilla.org/en-US/docs/Web/API/TaskSignal)

위 api는 아래의 세가지 우선순위중 하나를 설정할 수 있습니다.

- `user-blocking`: 가장 높은 우선순위로 즉시 실행되어 사용자 경험에 중요한 태스크에 적합.
- `user-visible`: 일반적인 사용자 인터페이스 업데이트에 적합. (대부분 기본값)
- `background`: 백그라운드에서 실행되어도 되는 태스크에 적합.

### 주의사항

- Prioritized Task Scheduling 관련 API는 일부 브라우저에서 지원합니다. 지원 여부를 확인해야 합니다.

## [**`scheduler.yield()`**](https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API#scheduler.yield)

- 현재 실행 중인 태스크가 다른 작업(특히 우선순위가 높은 작업)에게 실행 시간을 양보하는 Web API
- 브라우저는 다른 작업을 수행 한 후 `scheduler.yield()` 이후 코드를 실행

### 주요 특징

- **작업 양보**:
  - `scheduler.yield()`는 현재 태스크를 일시 중단하고, 다음 실행 가능한 태스크가 먼저 실행되도록 합니다.
  - 브라우저가 다른 작업(우선순위가 높은 태스크 또는 렌더링 작업)을 처리한 후, 남은 태스크를 다시 이어서 실행합니다.
- **`Promise` 기반**:
  - `scheduler.yield()`는 `Promise`를 반환합니다. 이 `Promise`는 브라우저가 태스크를 다시 실행할 준비가 되면 `resolved` 로 변경됩니다.
- **긴 작업 분할**:
  - 긴 작업을 여러 작은 작업으로 분할하여 브라우저의 응답성을 유지하는 데 유용합니다.

### 문법

```jsx
scheduler.yield(options);
```

**`options`** (선택 사항):

- `priority`: 양보한 후 태스크가 다시 실행될 때의 우선순위
  - `user-blocking`: 가장 높은 우선순위로 즉시 실행되어 사용자 경험에 중요한 태스크에 적합.
  - `user-visible`: 일반적인 사용자 인터페이스 업데이트에 적합. **(기본값)**
  - `background`: 백그라운드에서 실행되어도 되는 태스크에 적합.

### 사용 예제

- 일정 주기마다 브라우저에게 제어권을 양도하는 코드 - 응답성을 유지

```jsx
async function performLongTask() {
  for (let i = 0; i < 100000000; i++) {
    // 작업의 일부 수행
    if (i % 1000 === 0) {
      // 1000번마다 제어권을 양도 - 브라우저가 완전히 멈추는것을 방지
      await scheduler.yield();
    }
  }
  console.log("작업 완료");
}

performLongTask();
```

- 우선순위와 함께 사용하는 예제

```jsx
async function performTask() {
  console.log("작업 시작...");
  await scheduler.yield({ priority: "background" }); // 우선순위가 낮은 작업
  console.log("브라우저의 메인 thread 작업 이후 진행...");
}

performTask();
```

### setTimeout과의 차이

- **`setTimeout()`**:

  - 명시적으로 **일정 시간 후에 실행**해야 하는 작업이 있을 때.
  - 작업의 우선순위를 신경 쓰지 않고, 일정한 간격으로 작업을 실행할 때.
  - 지정한 시간 이후 callback에 전달했던 작업 실행
  - 반환값 : 해당 이벤트와 연결된 id

- **`scheduler.yield()`**:
  - 긴 작업을 효율적으로 나누면서 브라우저 응답성을 유지하고 싶을 때.
  - 사용자 입력이나 애니메이션 프레임 등의 **우선순위 높은 작업**을 방해하지 않도록 할 때.
  - 브라우저가 필요할 때 즉시 이후 작업을 재개
  - 반환값 : Promise

## [**`scheduler.postTask()`**](https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API#scheduler.posttask)

- 우선순위에 따라 예약할 작업을 추가하는데 사용되는 WebAPI

### 주요 특징

- **태스크 스케줄링**:
  - `scheduler.postTask()`를 사용하면 특정 태스크를 브라우저 작업 큐에 추가할 수 있습니다.
  - 기본적으로 태스크는 비동기로 실행됩니다.
- **우선순위 지정**:
  - `priority` 옵션을 통해 태스크의 우선순위를 설정할 수 있습니다.
  - 가능한 우선순위 값:
    - `user-blocking`: 가장 높은 우선순위로 즉시 실행되어 사용자 경험에 중요한 태스크에 적합.
    - `user-visible`: 일반적인 사용자 인터페이스 업데이트에 적합. (**기본값**)
    - `background`: 백그라운드에서 실행되어도 되는 태스크에 적합.
- **동작 설정**:
  - 태스크에 대한 실행 시점을 제어하기 위해 `delay`를 설정할 수 있습니다.
  - 태스크 실행 시 발생할 수 있는 에러를 처리할 수 있도록 `Promise`를 반환합니다.

### 문법

```jsx
scheduler.postTask(callback, options);
```

- `callback`: 실행할 태스크 함수.
- `options` (선택 사항):
  - `priority`: 태스크의 우선순위 (`user-blocking`, `user-visible`, `background`).
  - `delay`: 태스크 실행 전 대기 시간(밀리초).
  - `signal`: `AbortSignal` 객체로 태스크를 취소하는 데 사용.

### 사용 예제

```jsx
scheduler.postTask(
  () => {
    console.log("높은 우선순위의 작업 실행");
  },
  {
    priority: "user-blocking",
  }
);

scheduler.postTask(
  () => {
    console.log("Background 작업 실");
  },
  {
    priority: "background",
    delay: 2000, // 2초 후 실행
  }
);
```

## [\*\*`TaskController`](https://developer.mozilla.org/en-US/docs/Web/API/TaskController)\*\*

- 태스크의 우선순위를 동적으로 변경하거나, 태스크를 취소하는 기능을 제공합니다.
- `scheduler.postTask()`와 함께 사용되어 실행 중이거나 대기 중인 태스크를 제어할 수 있습니다

### 주요 특징

1. **우선순위 동적 변경**:
   - 실행 대기 중인 태스크의 우선순위를 변경할 수 있습니다.
   - 우선순위 변경은 태스크가 실행되기 전에 반영됩니다.
2. **태스크 취소**:
   - `TaskController`의 `signal`을 통해 특정 태스크를 취소할 수 있습니다.
   - 취소된 태스크는 실행되지 않고, `Promise`가 `AbortError`와 함께 거부됩니다.
3. **태스크 그룹 관리**:
   - 여러 태스크를 하나의 컨트롤러로 관리할 수 있어, 태스크 그룹의 우선순위나 상태를 제어하기에 적합합니다.

### 예제 코드

```jsx
const controller = new TaskController({ priority: "user-visible" });
```

**`priority`** (선택 사항)

- 초기 태스크 우선순위를 지정합니다. 기본값은 `"user-visible"`입니다.
- 가능한 값: `"user-blocking"`, `"user-visible"`, `"background"`.

### 주요 속성과 메서드

1. **속성**
   - `signal`: `AbortSignal` 객체로, 태스크 취소를 관리합니다.
     - 이를 `scheduler.postTask()`의 옵션으로 전달하여 태스크를 취소할 수 있습니다.
2. **메서드**
   - `setPriority(priority)`: 현재 태스크의 우선순위를 변경합니다.
     - `priority`: `"user-blocking"`, `"user-visible"`, `"background"` 중 하나.

### 예제 코드 2

우선순위를 올리는 예제 코드

```jsx
// TaskController 생성
const controller = new TaskController({ priority: "background" });

// 태스크 실행
scheduler.postTask(
  () => {
    console.log("태스크가 실행되었습니다.");
    //코드 실행중..
  },
  { signal: controller.signal }
);

// 우선순위 변경
controller.setPriority("user-blocking"); // background -> user-blocking 우선순위 상승
```

태스크를 취소하는 코드

```jsx
// TaskController 생성
const controller = new TaskController({ priority: "user-visible" });

scheduler.postTask(
  () => {
    console.log("취소되면 이 코드는실행되지 않습니다.");
  },
  { signal: controller.signal }
);

// 태스크 취소 이벤트 리스너
controller.signal.addEventListener("abort", () => {
  console.log("태스크가 취소되었습니다.");
});

// Abort the task
controller.abort();
```

여러 태스크를 관리하는 코드

```jsx
const controller = new TaskController({ priority: "background" });

scheduler.postTask(() => console.log("Task 1"), { signal: controller.signal });
scheduler.postTask(() => console.log("Task 2"), { signal: controller.signal });

// 우선순위 변경
controller.setPriority("user-blocking");
```

### 주요 장점

1. **동적 우선순위 관리**:
   - 대기 중인 태스크의 우선순위를 실행 상황에 맞게 변경 가능.
2. **태스크 취소**:
   - 필요하지 않은 태스크를 중단하여 리소스를 효율적으로 관리.
3. **그룹 제어**:
   - 여러 태스크를 하나의 컨트롤러로 관리하여 작업을 체계적으로 제어.

## [**`TaskSignal`**](https://developer.mozilla.org/en-US/docs/Web/API/TaskSignal)

- 태스크의 상태와 제어 정보를 제공하는 객체
- 태스크를 취소하거나 상태를 모니터링하는 데 사용

### 주요 특징

1. **태스크 상태 관리**:
   - 태스크의 실행 상태(예: 실행 중, 취소됨 등)를 나타냅니다.
   - 태스크가 취소되었는지 여부를 확인할 수 있습니다.
2. **태스크 취소 통신**:
   - `TaskSignal`은 태스크와 `TaskController` 간의 커뮤니케이션을 위한 매개체 역할을 합니다.
   - 태스크가 취소되었을 때 이를 감지하고 적절한 처리를 수행할 수 있습니다.
3. **이벤트 기반**:
   - `abort` 이벤트를 지원하여, 태스크가 취소될 때 콜백을 실행할 수 있습니다.

### 생성 방법

- `TaskSignal` 객체는 직접 생성되지 않으며, `TaskController`의 `signal` 속성을 통해 얻습니다.

```jsx
const controller = new TaskController();
const signal = controller.signal;
```

### 주요 속성과 이벤트

### 속성

1. **`aborted`**:
   - `true`일 경우 태스크가 취소된 상태.
   - `false`일 경우 태스크가 아직 활성 상태.

### 이벤트

1. **`abort`**:
   - 태스크가 취소될 때 트리거되는 이벤트.
   - 태스크 취소 시 후속 작업(예: 정리 작업)을 처리하는 데 사용.

### 사용 예제

태스크 상태 확인

```jsx
const controller = new TaskController();
const signal = controller.signal;

scheduler.postTask(
  () => {
    if (signal.aborted) {
      console.log("태스크가 시작하기 전에 취소되었습니다."); //이 log는 출력되지 않습니다.
      return;
    }
    console.log("태스크가 실행되었습니다.");
  },
  { signal }
);

// 태스크 취소 - 에러 throw
controller.abort();
```

태스크 취소 이벤트 처리

```jsx
const controller = new TaskController();
const signal = controller.signal;

signal.addEventListener("abort", () => {
  console.log("태스크가 취소되었습니다.");
});

scheduler.postTask(
  () => {
    console.log("태스크가 실행되었습니다.");
  },
  { signal }
);

// 태스크 취소
controller.abort();
```

복잡한 태스크 관리

```jsx
function myLog(message) {
  console.log(`[${new Date().toISOString()}] ${message}`);
}

// 딜레이 함수
function delay(ms, signal) {
  return new Promise((resolve, reject) => {
    const timeout = setTimeout(resolve, ms);
    if (signal.aborted) {
      clearTimeout(timeout);
      reject(new DOMException("태스크가 취소되었습니다", "AbortError"));
    }
    signal.addEventListener("abort", () => {
      clearTimeout(timeout);
      reject(new DOMException("태스크가 취소되었습니다", "AbortError"));
    });
  });
}

// TaskController 생성 (외부에 위치)
const abortTaskController = new TaskController();

async function example() {
  // 작업 예약
  const task1 = scheduler.postTask(
    async () => {
      myLog("태스크 1이 실행중입니다...");
      await delay(1000, abortTaskController.signal); // 1초 딜레이
      myLog("태스크 1이 완료되었습니다");
    },
    { signal: abortTaskController.signal }
  );

  const task2 = scheduler.postTask(
    async () => {
      myLog("태스크 2이 실행중입니다...");
      await delay(1500, abortTaskController.signal); // 1.5초 딜레이
      myLog("태스크 2이 완료되었습니다");
    },
    { signal: abortTaskController.signal }
  );

  // 작업 처리
  task1
    .then(() => myLog("태스크 1이 성공적으로 완료되었습니다")) // 로그는 출력되지 않음
    .catch((error) => myLog(`태스크 1이 실패하였습니다: ${error.message}`));

  task2
    .then(() => myLog("태스크 2가 성공적으로 완료되었습니다")) //로그는 출력되지 않음
    .catch((error) => myLog(`태스크 2이 실패하였습니다: ${error.message}`));
}

// 실행
await example();

// 300ms 후 작업 취소
setTimeout(() => {
  myLog("모든 태스크를 취소합니다...");
  abortTaskController.abort(); // 외부에서 생성된 컨트롤러 사용
}, 300);
```

### 주요 장점

1. **태스크 취소 감지**:
   - 태스크가 취소되었는지 확인하고, 필요한 경우 실행을 중단할 수 있음.
2. **태스크 정리 용이**:
   - `abort` 이벤트를 활용해 정리 작업(예: 메모리 해제, UI 업데이트 등)을 수행 가능.
3. **효율적인 작업 흐름**:
   - 불필요한 태스크 실행을 방지해 리소스를 절약.

- **취소 후 Promise 거부**:
  - 취소된 태스크의 `Promise`는 `AbortError`와 함께 거부되므로 에러 처리를 추가해야 합니다.
- **브라우저 호환성**:
  - 최신 브라우저에서만 사용 가능하므로 지원 여부를 확인해야 합니다.
- **이벤트 리스너 관리**:
  - `abort` 이벤트 리스너를 등록한 경우, 불필요한 리스너가 남아 있지 않도록 관리해야 합니다.

---

## 기타 사항

### 변경 가능한 task 우선순위와 변경 불가능한 task 우선순위

- `Scheduler.postTask()` 의 인수 (`options.priority`) 로 우선순위가 전달되면 태스크 우선순위는 도중에 변경될 수 없습니다.
- 우선순위를 도중에 변경하고싶다면 사전에 options.priority를 전달하지 않고, TaskSignal을 통해 연결되어있어야합니다.

### 태스크 우선순위 상속

- Scheduler.postTask() 작업 내의 Scheduler.yield() 호출은 작업의 우선순위를 상속합니다.
- 예를 들어 우선순위가 낮은 `background` 작업 내에서 Scheduler.yield() 이후의 작업은 기본적으로 `background` 로 예약됩니다(그러나 다시 부스트된 "백그라운드" 우선순위 대기열에 삽입되므로 "백그라운드" 이전에 실행됨) " postTask() 작업)

참고했던 자료

- MDN 문서
  - https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API
- W3C 커뮤니티 그룹 자료 (표준이 아님)
  - https://wicg.github.io/scheduling-apis/
- 다른 설명 자료
  - https://velog.io/@sehyunny/javascript-scheduler-api
