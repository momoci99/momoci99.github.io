---
layout: single
title: 'Understanding higher-order functions in JavaScript'
tag:
  - javascript
  - 30secondsofcode
---

# 개요

> 이 글은 https://www.30secondsofcode.org/에서 작성한 글을 번역 및 확인해보는 글 입니다.

원문 : [Understanding higher-order functions in JavaScript](https://www.30secondsofcode.org/blog/s/javascript-higher-order-functions)

고차 함수는 인수로 취하거나 결과로 반환하여 다른 함수에서 작동하는 함수입니다. 이를 통해 값뿐만 아니라 작업에 대한 추상화 계층을 만들 수 있습니다.

JavaScript에서 고차 함수를 작성할 수있는 이유는 함수가 값이기 때문에 변수에 할당되고 값으로 전달 될 수 있기 때문입니다. 고차 함수에서 호출하기 때문에 인수로 전달되는 함수를 참조 할 때 콜백 이라는 용어를들을 수도 있습니다 . 이는 이벤트 처리, 비동기 코드 및 배열 작업이 콜백 개념에 크게 의존하는 JavaScript에서 특히 일반적입니다.

이 기술의 주요 장점은 앞서 언급했듯이, 우리가 생성 할 수있는 추상화 계층과 구성 및 더 재사용 가능하고 읽기 쉬운 코드입니다. 30 초의 코드 조각 대부분은 더 복잡한 논리를 만들기 위해 쉽게 구성 할 수 있고 재사용 가능성이 높은 작고 쉽게 소화 할 수있는 함수를 촉진하기 때문에 고차 함수의 아이디어를 기반으로합니다.

즉, 매우 간단한 기능을 활용하여 예제를 살펴볼 수 있습니다.

```js
const add = (a, b) => a + b
const isEven = (num) => num % 2 === 0

const data = [2, 3, 1, 5, 4, 6]

const evenValues = data.filter(isEven) // [2, 4, 6]
const evenSum = data.filter(isEven).reduce(add) // 12
```

HOF와 관련하여 잘 정리된 글의 링크를 추가합니다

https://dev-momo.tistory.com/entry/HigherOrder-Function-%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80
