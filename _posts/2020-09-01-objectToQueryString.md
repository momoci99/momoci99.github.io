---
layout: single
title: 'objectToQueryString'
tag:
  - javascript
  - 30secondsofcode
---

# 개요

> 이 글은 https://www.30secondsofcode.org/에서 작성한 글을 번역 및 확인해보는 글 입니다.

원문 : [objectToQueryString](https://www.30secondsofcode.org/js/s/object-to-query-string)

```
주어진 객체의 key-value 쌍으로 부터 생성된 query 문자열을 반환합니다.
```

`Object.entries(queryParameters)` 에 `Array.prototype.reduce()` 를 사용하여 query 문자열을 만들 수 있습니다. `queryString`의 `length`를 기반으로 `symbol`이 `?` 혹은 `&`인지 결정합니다. 그리고 `val` 이 문자열일때만 `queryString`에 이어 붙입니다. `queryParameters`이 거짓된 값(falsy)인 경우 빈 문자열을 반환합니다.

```js
const objectToQueryString = (queryParameters) => {
  return queryParameters
    ? Object.entries(queryParameters).reduce(
        (queryString, [key, val], index) => {
          const symbol = queryString.length === 0 ? '?' : '&'
          queryString += typeof val === 'string' ? `${symbol}${key}=${val}` : ''
          return queryString
        },
        ''
      )
    : ''
}

objectToQueryString({ page: '1', size: '2kg', key: undefined }) // '?page=1&size=2kg'
```
