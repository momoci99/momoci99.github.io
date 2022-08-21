---
layout: single
title: "Hackerrank Arrays DS"
tag:
  - Array
  - Hackerrank
---

# Arrays DS

[문제 링크](https://www.hackerrank.com/challenges/arrays-ds/problem)

# 설명

배열 요소의 순서를 뒤집으면 된다. stack의 성질을 이용하여 주어진 배열의 마지막 요소부터 첫 요소까지 순서대로 stack에 집어넣는다.

# 코드

```js
function reverseArray(a) {
  if (Array.isArray(a)) {
    let reversed = [];
    let length = a.length;
    for (let i = length - 1; i >= 0; i--) {
      reversed.push(a[i]);
    }
    return reversed;
  }
}
```
