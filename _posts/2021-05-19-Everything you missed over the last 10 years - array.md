---
layout: single
title: "잊고 지냈던 js array 메서드"
tag:
  - CleanCode
---

이 글은 Modern Javascript: Everything you missed over the last 10 years 에서 영감을 받은 글입니다.

# 개요

얼마전 GeekNews에 [최신 JavaScript: 지난 10년간 놓친 모든 것](https://news.hada.io/topic?id=4270)이라는 글이 올라왔습니다. 아니 이런것도 있었나?!하는 항목중에 (!) Array 관련 메서드를 간단한게 정리해보았습니다.

## [Array.from()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/from)

유사 배열 객체(array-like object)혹은 반복 가능 객체(iterable object)를 얕게 복사하여 새로운 Array 객체를 만드는 메서드입니다.

### String에서 배열을 만들어 봅시다.

Array.from 사용

```js
Array.from("foo");
// ["f", "o", "o"]
```

Array.from을 사용하지 않고 동일한 결과

```js
"foo".split("");
// ["f", "o", "o"]
```

### 그럼 이번에는 `Set`을 배열로 만들어 봅시다.

Array.from 사용

```js
const s = new Set(["foo", window]);
Array.from(s);
// ["foo", window]
```

Array.from을 사용하지 않고 동일한 결과

```js

const s = new Set(["foo", window]);
const a = [..s]
// ["foo", window]
```

### Array.map 처럼 사용할 수 있습니다.

Array.from 사용

```js
Array.from([1, 2, 3], (x) => x + x);
// [2, 4, 6]
```

Array.from을 사용하지 않고 동일한 결과

```js
[1, 2, 3].map((x) => x + x);
```

## [Array.reduceRight()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/ReduceRight)

`Array.reduce()`와 비슷하지만 오른쪽에서 왼쪽으로 연산을 수행합니다.

예를 들어 배열의 요소를 단일 배열로 전개한다고 생각해봅시다.

Array.reduceRight()는 맨 오른쪽 요소부터 연산을 수행하기 때문에 4,5 부터 시작하게 됩니다.

```js
const array1 = [
  [0, 1],
  [2, 3],
  [4, 5],
].reduceRight((accumulator, currentValue) => accumulator.concat(currentValue));

//output: Array [4, 5, 2, 3, 0, 1]
```

반면 Array.reduce()는 맨 왼쪽 요소부터 연산을 수행하기 때문에 0,1 부터 시작하게 되네요.

```js
const array1 = [
  [0, 1],
  [2, 3],
  [4, 5],
].reduce((accumulator, currentValue) => accumulator.concat(currentValue));
//output : Array [0, 1, 2, 3, 4, 5]
```

## [Array.copyWithin()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/copyWithin)

원문 글에는 없었지만 MDN을 뒤져보다 발견한 메서드입니다. 이 메서드는 배열 일부를 얕게 복사하여 동일한 배열의 다른 위치에 덮어쓰고(?!) 배열을 반환합니다. 이때 크기는 바뀌지 않습니다.

```js
const array1 = ["a", "b", "c", "d", "e"];

// 3번째 요소를 0번째 위치에 복사
console.log(array1.copyWithin(0, 3, 4));
// output: Array ["d", "b", "c", "d", "e"]

// 3번째 요소부터 끝까지 모든 요소를 ​​인덱스 1에 복사
console.log(array1.copyWithin(1, 3));
// output: Array ["d", "d", "e", "d", "e"]
```

# 링크

[Modern Javascript: Everything you missed over the last 10 years](https://turriate.com/articles/modern-javascript-everything-you-missed-over-10-years)
