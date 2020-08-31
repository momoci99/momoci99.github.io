---
layout: single
title: 'reducedFilter'
tag:
  - javascript
  - 30secondsofcode
---

# 개요

> 이 글은 https://www.30secondsofcode.org/에서 작성한 글을 번역 및 확인해보는 글 입니다.

원문 : [reducedFilter](https://www.30secondsofcode.org/js/s/reduced-filter)

```
조건에 따라 객체 배열을 필터링하는 동시에 지정되지 않은 키도 필터링합니다.
```

조건이 참 같은 값을 반환 한 객체를 반환하도록 `Array.prototype.filter()`를 사용하여 조건 자 `fn`을 기준으로 배열을 필터링합니다.

`Array.prototype.reduce()`를 사용하여 `keys` 인수로 제공되지 않은 키값 필터링하고 `Array.prototype.map()`을 사용해 새로운 객체를 반환합니다.

```js
const reducedFilter = (data, keys, fn) =>
  data.filter(fn).map((el) =>
    keys.reduce((acc, key) => {
      acc[key] = el[key]
      return acc
    }, {})
  )
EXAMPLES
const data = [
  {
    id: 1,
    name: 'john',
    age: 24,
  },
  {
    id: 2,
    name: 'mike',
    age: 50,
  },
]

reducedFilter(data, ['id', 'name'], (item) => item.age > 24) // [{ id: 2, name: 'mike'}]
```

## 설명

1. data.filter(fn)를 통해 `fn`의 조건을 만족하는 객체만 남는다.
2. `map()`함수에 인자로 전달된 함수가 실행된다.
3. `map()`의 인자로 전달된 `keys.reduce` 함수를 실행한다
4. filter로 걸러진 객체중 `keys`내의 요소와 일치하는 속성값으로 새로운 객체를 반환한다.
5. `map()`은 4에서 반환된 객체로 새로운 배열을 반환한다.
6. 결과 반환
