---
layout: single
title: 'palindrome'
tag:
  - javascript
  - 30secondsofcode
---

# 개요

> 이 글은 https://www.30secondsofcode.org/에서 작성한 글을 번역 및 확인해보는 글 입니다.

원문 : [palindrome](https://www.30secondsofcode.org/js/s/palindrome)

```
주어진 문자열이 회문 일때 `true`를 반환하고 그렇지 않으면 `false`를 반환합니다.
```

문자열을 `String.prototype.toLowerCase()`로 변환하고 `String.prototype.replace()`로 영숫자가 아닌 문자열을 제거합니다. 그리고 나서 전개 연산자(`...`)로 개별문자인 `Array.prototype.reverse()`, `String.prototype.join('')` 으로 분할하고 이를 원래 문자열과 비교한 다음 `String.prototype.toLowerCase()`로 변환합니다.

```js
const palindrome = (str) => {
  const s = str.toLowerCase().replace(/[\W_]/g, '')
  return s === [...s].reverse().join('')
}

palindrome('taco cat') // true
```

## 설명

번역을 매우 많이 이상하게 했는데 설명하면 아래와 같습니다.

1. 주어진 문자열을 소문자로 변환합니다.
2. 1에서 영문자 숫자가 아닌 문자(ex : 공백)를 모두 제거합니다.
3. 2에서 만들어진 문자를 전개연산자로 문자 배열로 변환합니다.
4. 문자 배열을 `reverse()`로 반전시킵니다.
5. 4의 반환된 문자 배열을 하나의 문자열로 합칩니다.
6. 2에서 만들어진 문자열과 5에서 만들어진 문자열을 비교합니다.
