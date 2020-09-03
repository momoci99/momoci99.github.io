---
layout: single
title: 'What is the ternary operator and how do I use it?'
tag:
  - javascript
  - 30secondsofcode
---

# 개요

> 이 글은 https://www.30secondsofcode.org/에서 작성한 글을 번역 및 확인해보는 글 입니다.

원문 : [What is the ternary operator and how do I use it?](https://www.30secondsofcode.org/blog/s/javascript-ternary-operator)

자바스크립트의 조건 연산자 (`?:`)는 conditional operator 로 불리며 가장 일반적으로 할당하는 조건문을 대체하는데 사용됩니다.

```js
// if..else 를 사용했을 때
let x
if (someCondition) {
  x = 10
} else {
  x = 20
}

// 삼항 연산자를 사용한 똑같은 코드
const x = someCondition ? 10 : 20
```

위의 예제에서 알 수 있듯이 삼항 연산자는 `if..else`문을 사용하는 것보다 짧으며 조건이 거의 한 줄이면 결과 값을 변수에 할당 할 수 있습니다. 이 유용한 결과는 암시 적 리턴이있는 화살표 함수에 삼항 연산자를 사용할 수 있다는 것입니다.

```js
// if..else 를 사용했을 때
const conditionalX = (condition, x) => {
  if (condition) return x
  else return x + 5
}

// 삼항 연산자를 사용한 똑같은 코드
const conditionalX = (condition, x) => (condition ? x : x + 5)
```

그러나 중첩 삼항 표현식은 ESLint 에서 이러한 종류의 상황에 대한 [전용 규칙](https://eslint.org/docs/rules/no-nested-ternary) 이 있는 경우에도 일반적으로 권장되지 않습니다. 또한 삼항 연산자는 JSX 코드에서 쉽게 조건부 렌더링을 허용하므로 React 개발자가 선호하는 연산자입니다.

```js
const ItemListTitle = (count) => (
  <h3>Item list{ count ? (<span> - {count} items</span>) : '(Empty)'}<h3>
);
```

마지막으로 "삼항"연산자라고하는 이유가 궁금 할 것입니다. "삼항"이라는 단어는 (n 항 단어 설정)[https://en.wikipedia.org/wiki/Arity]을 기반으로하며 세 개의 피연산자가있는 연산자를 의미합니다 (조건, 진실이면 실행할 표현식, 거짓이면 실행할 표현식).
