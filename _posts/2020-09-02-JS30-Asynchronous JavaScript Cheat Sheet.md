---
layout: single
title: 'Asynchronous JavaScript Cheat Sheet'
tag:
  - javascript
  - 30secondsofcode
---

# 개요

> 이 글은 https://www.30secondsofcode.org/에서 작성한 글을 번역 및 확인해보는 글 입니다.

원문 : [Asynchronous JavaScript Cheat Sheet](https://www.30secondsofcode.org/blog/s/async-javascript-cheatsheet)

## Promise 기초

- **Promises**는 fullfiled(이행) 혹은 rejected(거부) 되지 않은 **pending state(보류 상태)** 에서 시작합니다.

- 명령이 완료되면 promise는 값과 함께 fullfiled(이행) 상태가 됩니다.

- 명령이 실패하면 promise는 에러와 함께 rejected(거부) 상태가 됩니다.

## Promise 생성

- `new Promise`에 전달된 함수는 동기적으로 실행됩니다.
- `resolve()` 혹은 `reject()`를 사용하여 값으로 부터 promise를 생성합니다.
- `Promise.resolve(val)`는 promise를 `val`과 함께 이행 됩니다.
- `Promise.reject(err)`는 promise를 `err`과 함께 거부 됩니다.
- 이행된 promise를 다른 이행된 promise에 넣으면 하나로 합쳐집니다.

```js
// 값(val)로 결정(resolve)되고, 에러(err)로 거부됩니다.
new Promise((resolve, reject) => {
  performOperation((err, val) => {
    if (err) reject(err)
    else resolve(val)
  })
})

// 값(value)로 결정되지 않는다면, 거부될 필요가 없습니다.
const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms))
```

## Promise 다루기

- `Promise.prototype.then()`는 두개의 부가적인 인수를 받습니다.(`onFulfilled`, `onRejected`)

- `Promise.prototype.then()`는 promise가 이행되었을때 `onFulfilled` 를 호출 합니다.

- `Promise.prototype.then()`는 promise가 거부 되었을때 `onRejected`를 호출 합니다.

- `Promise.prototype.then()`는 `onRejected`가 정의되지 않았을때(undefined) 에러를 전달합니다.

- `Promise.prototype.catch()`는 하나의 인수만을 받아들입니다. (`onRejected`)

- `Promise.prototype.catch()`는 `onFulfilled`가 생략되었을 때 `Promise.prototype.then()`처럼 동작합니다.

- `Promise.prototype.catch()`는 이행된 값을 전달합니다.

- `Promise.prototype.finally()`는 하나의 인수를 받아들입니다. (`onFinally`)

- `Promise.prototype.finally()`는 어떤 결과가 나오던 아무 인수 없이 `onFinally`를 한번 호출 합니다.

- `Promise.prototype.finally()`는 입력된 promise를 전달합니다.

```js
promisedOperation()
  .then(
    val => value + 1,   // promise가 이행 되었을때 호출됩니다.
    err => {            // promise가 거부 되었을때 호출됩니다.
      if (err === someKnownErr) return defaultVal;
      else throw err;
    }
  )
  .catch(
    err => console.log(err); // promise가 거부 되었을때 호출됩니다.
  )
  .finally(
    () => console.log('Done'); // 결과가 어떻든 호출됩니다.
  );
```

- 위 3개의 메서드는 promise가 이미 결과를 가지고 있어도 최소한 다음 tick 까지는 실행되지 않습니다.

## promise 조합

- `Promise.all()`는 promises로 이루어진 배열을 배열의 promise로 변환합니다.
- promise가 거절되면 에러가 전달 됩니다.
- `Promise.race()`는 첫번째로 결정된 promise를 반환합니다.

```js
Promise.all([p1, p2, p3]).then(([v1, v2, v3]) => {
  // 값들은 항상 promise의 순서에 대응됩니다.
  // 결정된 순서대로가 가닙니다.(예 : v1 은 p1에 대응)
})

Promise.race([p1, p2, p3]).then((val) => {
  // 값은 첫번째로 결정된 promise가 할당됩니다.
})
```

## async/await

- `async` 함수를 호출하면 항상 promise를 반환합니다.
- `(async () => value)()`는 `value`를 결정합니다.
- `(async () => throw err)()`는 에러와 함께 거절됩니다.
- `await`는 promise가 이행되고 값을 반환할때 까지 기다립니다.
- `await`는 오직 `async`함수 내에서만 사용할 수 있습니다.
- `await`는 non-promise 값도 받아들입니다.
- `await`는 이미 이행 된 promise 또는 non-promise 값을 기다리는 경우에도 결정 전 다음 틱까지 항상 기다립니다.

```js
;async () => {
  try {
    let val = await promisedValue()
    // 로직 수행
  } catch (err) {
    // 에러 처리
  }
}
```
