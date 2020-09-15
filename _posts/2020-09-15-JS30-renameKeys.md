---
layout: single
title: 'renameKeys'
tag:
  - javascript
  - 30secondsofcode
---

# 개요

> 이 글은 https://www.30secondsofcode.org/에서 작성한 글을 번역 및 확인해보는 글 입니다.

원문 : [renameKeys](https://www.30secondsofcode.org/js/s/rename-keys)

```
주어진 값으로 객체의 키값명을 변경합니다.
```

`Object.keys()`를 `Array.prototype.reduce()` 및 전개 연산자 (`...`)와 함께 사용하여 객체의 키를 가져오고 keysMap에 따라 이름을 바꿉니다.

```js
const renameKeys = (keysMap, obj) =>
  Object.keys(obj).reduce(
    (acc, key) => ({
      ...acc,
      ...{ [keysMap[key] || key]: obj[key] },
    }),
    {}
  )

const obj = { name: 'Bobo', job: 'Front-End Master', shoeSize: 100 }
renameKeys({ name: 'firstName', job: 'passion' }, obj)
// { firstName: 'Bobo', passion: 'Front-End Master', shoeSize: 100 }
```

참고링크 : https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR
