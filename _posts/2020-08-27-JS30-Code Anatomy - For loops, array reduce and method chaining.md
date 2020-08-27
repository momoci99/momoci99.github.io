---
layout: single
title: '[30secondsofcode] loops, array reduce and method chaining'
tag:
  - javascript
  - 30secondsofcode
---

# 개요

> 이 글은 https://www.30secondsofcode.org/에서 작성한 글을 번역 및 확인해보는 글 입니다.

원문 : [Code Anatomy - For loops, array reduce and method chaining](https://www.30secondsofcode.org/blog/s/code-anatomy-chaining-reduce-for-loop)

## For loops

```js
const files = ['foo.txt ', '.bar', '   ', 'baz.foo']
let filePaths = []

for (let file of files) {
  const fileName = file.trim()
  if (fileName) {
    const filePath = `~/cool_app/${fileName}`
    filePaths.push(filePath)
  }
}

// filePaths = [ '~/cool_app/foo.txt', '~/cool_app/.bar', '~/cool_app/baz.foo']
```

- 어떤 `for` 루프든 사용될 수 있음.
- 오늘날에는 덜 대중적인 방법이며, 함수형 프로그래밍이 좀더 대중적.
- 이른 `return`과 같은 순회를 넘어서는 제어
- 반복문 바깥에서 결과를 저장하는 배열이 선언되어야함.
- `Array.prototype.push()` 혹은 전개연산자(`...`)를 사용하여 요소를 더함.
- `O(N)` 의 복잡도를 가짐. 각 요소는 한번만 순회됨.

## Array reduce

```js
const files = ['foo.txt ', '.bar', '   ', 'baz.foo']
const filePaths = files.reduce((acc, file) => {
  const fileName = file.trim()
  if (fileName) {
    const filePath = `~/cool_app/${fileName}`
    acc.push(filePath)
  }
  return acc
}, [])
```

- 초기값으로 빈 배열을 `Array.prototype.reduce()`와 함께 사용.
- 요즘에는 함수형 프로그래밍이 더 많이 사용되기 때문에 더 일반적임.
- 반복문에 대한 제어가 줄고 요소를 건너뛰거나 조기 `return`이 불가능함.
- 필요하다면 다른 메서드와 chain이 가능함.
- `Array.prototype.push()` 혹은 전개연산자(`...`)를 사용하여 요소를 더함.
- `O(N)` 의 복잡도를 가짐. 각 요소는 한번만 순회됨.

## Method chaining

```js
const files = ['foo.txt ', '.bar', '   ', 'baz.foo']
const filePaths = files
  .map((file) => file.trim())
  .filter(Boolean)
  .map((fileName) => `~/cool_app/${fileName}`)
// filePaths = [ '~/cool_app/foo.txt', '~/cool_app/.bar', '~/cool_app/baz.foo']
```

- `Array.prototype.map()` 와 `Array.prototype.filter()`를 사용.
- 요즘에는 함수형 프로그래밍이 더 많이 사용되기 때문에 더 일반적임.
- 반복문에 대한 제어가 줄고 요소를 건너뛰거나 조기 `return`이 불가능함.
- 선언적이고 읽고 리팩토링하기 더 쉬워졌으며 필요하다면 결과에 chain이 가능함.
- `Array.prototype.push()` 혹은 전개연산자(`...`)를 사용하지 않음.
- `O(cN)` 복잡도를 가짐. `c`는 요소당 반복횟수를 말함
