---
layout: single
title: "[Javascript] Javascript this에 대하여"
tag:
  - Javascript
  - this
---

자바스크립트 this 키워드에 대해서 mdn 문서를 기반으로 어떤 상황에서 어떻게 동작하고 정의되는지 정리하였습니다.

자바스크립트의 `this` 키워드는 다른 언어와 조금 다르게 동작합니다. 또한 [엄격 모드](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Strict_mode) 와 비엄격 모드에서도 일부 차이가 있습니다. 이번 글에서는 this가 각 문맥에서 어떻게 동작하는지 살펴보겠습니다.

- 대부분의 경우 `this`의 값은 함수를 호출한 방법에 의해 결정됩니다.
- 실행중에는 할당으로 설정할 수 없고 함수를 호출할 때마다 다를 수 있습니다.
- ES5는 함수를 어떻게 호출했는지 상관하지 않고 `this` 값을 설정할 수 있는 [bind](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 메서드가 도입되었습니다.
- ES2015는 `this` 바인딩을 제공하지 않는 화살표 함수를 추가하였습니다. (lexcial context의 `this` 값을 유지합니다.)

- Lexcial context? Lexcial Scope?
  javascript는 Lexical scope (as strict scope - 정적 스코프)를 따르고 있습니다. Lexical scope는 함수를 어디서 선언하였는지에 따라 상위 스코프를 결정하는 방식입니다. 반대의 개념으로 함수나 변수의 호출 시점에 스코프를 결정하는 Dynamic Scope가 있습니다.
  또한, 클로저의 핵심 개념 중 하나로, 함수가 어디에서 정의되었는지에 따라 스코프가 결정되는 개념입니다. 클로저는 이 Lexcial Scope를 기반으로 동작하며, 함수가 생성될 때의 스코프를 "기억"하여 나중에 그 스코프에 접근할 수 있게 합니다.

## 함수 문맥에서의 this

함수 내부에서 this의 값은 함수를 호출한 방법에 의해 좌우됩니다.

```jsx
//엄격 모드가 아님에 주의!
function f1() {
  return this;
}

// 브라우저
f1() === window; // true

// Node.js
f1() === global; // true
```

반면에 엄격 모드에서 this 값은 실행 문맥에 진입하여 설정되는 값을 유지합니다.

```jsx
function f2() {
  "use strict"; // 엄격 모드 참고
  return this;
}

f2() === undefined; // true
```

this의 값을 한 문맥(context)에서 다른 문맥으로 넘기려면 [call()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/call), [apply()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)를 사용합니다.

예시 1

```jsx
// call 또는 apply의 첫 번째 인자로 객체가 전달될 수 있으며 this가 그 객체에 묶임
var obj = { a: "Custom" };

// 변수를 선언하고 변수에 프로퍼티로 전역 window를 할당
// window.a = "Global" 과 동일
var a = "Global";

function whatsThis() {
  return this.a; // 함수 호출 방식에 따라 값이 달라짐
}

whatsThis(); // this는 'Global'. 함수 내에서 설정되지 않았으므로 global/window 객체로 초기값을 설정한다.
whatsThis.call(obj); // this는 'Custom'. 함수 내에서 obj로 설정한다.
whatsThis.apply(obj); // this는 'Custom'. 함수 내에서 obj로 설정한다.
```

`*var` 는 전역 변수를 정의합니다 (전역 객체에 프로퍼티로 할당됨) - 현 시점에는 거의 사용하지 않습니다.\*

예시 2

```jsx
// 엄격 모드 아님
function add(c, d) {
  return this.a + this.b + c + d;
}

var o = { a: 1, b: 3 };

// 첫 번째 인자는 'this'로 사용할 객체이고,
// 이어지는 인자들은 함수 호출에서 인수로 전달된다.
add.call(o, 5, 7); // 16

// 첫 번째 인자는 'this'로 사용할 객체이고,
// 두 번째 인자는 함수 호출에서 인수로 사용될 멤버들이 위치한 배열이다.
add.apply(o, [10, 20]); // 34
```

**비 엄격모드에서 this로 전달된 값이 객체가 아닌경우**

- call 과 apply는 이를 객체로 변환하기 위한 시도를 합니다. (Boxed)
- null과 undefined 값은 전역 객체가 됩니다.
- 7 (number) 나 “foo” (string) 와 같은 원시값은 관련된 생성자를 사용해 객체로 변환됩니다.
- 원시 숫자 7은 new Number(7) 에 의해 객체로 변환, 문자열 “foo” 는 new String(’foo’) 에 의해 객체로 변환됩니다.

```jsx
function bar() {
  console.log(Object.prototype.toString.call(this));
}

bar.call(7); // [object Number]
bar.call("foo"); // [object String]
bar.call(undefined); // [object global]
```

**엄격모드에서 this로 전달된 값이 객체가 아닌경우**

- call 과 apply는 이를 객체로 변환하기 위한 시도를 하지 않습니다.
- null과 undefined 값은 그대로 null 과 undefined 로 유지됩니다.

## bind 메서드

- ECMAScript 5 (ES5)는 [Function.prototype.bind](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 를 도입했습니다.
- f.bind(someObject)를 호출하면 f 와 같은 본문 (코드)과 범위를 가졌지만 this는 원본 함수를 가진 새로운 함수를 생성합니다.
- 새 함수의 this는 호출 방식과 상관없이 영구적으로 bind()의 첫 번째 매개변수로 고정됩니다.

```jsx
//엄격 모드 아님
function f() {
  return this.a;
}

var g = f.bind({ a: "azerty" });
console.log(g()); // azerty

var h = g.bind({ a: "yoo" }); // bind는 한 번만 동작함!
console.log(h()); // azerty

var o = { a: 37, f: f, g: g, h: h };
console.log(o.a, o.f(), o.g(), o.h()); // 37, 37, azerty, azerty
```

질문

Q: call, apply, bind를 사용하면 스코프가 변경되는게 아닌가요?

A: 아닙니다. 렉시컬 스코프는 유지됩니다. call, apply, bind는 this 키워드가 가르키는 대상을 변경할 뿐입니다.

## 화살표 함수

화살표 함수에서 this는 자신을 감싼 정적 범위입니다. 전역 코드에서는 전역 객체를 가르킵니다.

```jsx
//엄격 모드 아님
var globalObject = this;
var foo = () => this;
console.log(foo() === globalObject); // true
```

_단! 엄격 모드에서 화살표함수 내에서 `this`를 호출하면 `undefined` 가 됩니다._

참고!

- 화살표 함수를 call(), bind(), apply()를 사용해 호출할 때 this의 값을 정해주더라도 무시합니다.
- 사용할 매개변수를 정해주는 건 문제없지만, 첫 번째 매개변수(thisArg)는 null을 지정해야합니다.

```jsx
// 엄격 모드 아님
// 객체로서 메서드 호출
var obj = { func: foo };
console.log(obj.func() === globalObject); // true

// call을 사용한 this 설정 시도
console.log(foo.call(obj) === globalObject); // true

// bind를 사용한 this 설정 시도
foo = foo.bind(obj);
console.log(foo() === globalObject); // true
```

- 어떤 방법을 사용하던 foo의 this는 생성 시점의 것으로 설정됩니다(위 코드는 global 객체)
- 다른 함수 내에서 생성된 화살표 함수에도 동일하게 적용됩니다.
- this는 쌓여진 렉시컬 컨텍스트의 것으로 유지됩니다.

```jsx
// 엄격 모드 아님
// this를 반환하는 메소드 bar를 갖는 obj를 생성합니다.
// 반환된 함수는 화살표 함수로 생성되었으므로,
// this는 감싸진 함수의 this로 영구적으로 묶입니다.
// bar의 값은 호출에서 설정될 수 있으며, 이는 반환된 함수의 값을 설정하는 것입니다.
var obj = {
  bar: function () {
    var x = () => this;
    return x;
  },
};

// obj의 메소드로써 bar를 호출하고, this를 obj로 설정
// 반환된 함수로의 참조를 fn에 할당
var fn = obj.bar();

// this 설정 없이 fn을 호출하면,
// 기본값으로 global 객체 또는 엄격 모드에서는 undefined
console.log(fn() === obj); // true

// 호출 없이 obj의 메소드를 참조한다면 주의하세요.
var fn2 = obj.bar;
// 화살표 함수의 this를 bar 메소드 내부에서 호출하면
// fn2의 this를 따르므로 window를 반환할것입니다.
console.log(fn2()() == window); // true
```

- 위 예시에서, obj.bar에 할당된 함수(익명함수 A)는 화살표 함수로 생성된 다른 함수(익명함수 B)를 반환합니다.
- 결과로써 함수 B가 호출될 때 B의 this는 영구적으로 obj.bar(함수 A)의 this로 설정됩니다. (화살표 함수 이므로)
- 반환된 함수(함수 B)가 호출될 때, this는 항상 초기에 설정된 값 입니다.
- 위 코드 예시에서 함수 B의 `this`는 함수 A의 `this`인 obj로 설정되므로, 일반적으로 `this`를 undefined나 global 객체로 설정하는 방식으로 호출할 때도(또는 이전 예시에서 처럼 global 실행 컨텍스트에서 다른 방법을 사용할 때에도) obj설정은 유지됩니다.

## 생성자로서

- 함수를 new 키워드와 함께 생성자로 사용하면 this는 새로 생긴 객체에 묶입니다.

```jsx
//엄격 모드 아님
function C() {
  this.a = 37;
}

let o = new C();
console.log(o.a); // 37

function C2() {
  this.a = 37;
  return { a: 38 };
}

o = new C2();
console.log(o.a); // 38
```

- 함수가 생성자로 사용될 때(`new` 키워드를 사용하여), 생성자 함수가 어떤 객체에 접근하든 간에 이 함수는 생성자가 새로운 객체에 묶입니다.
- 생성자가 다른 비원시적 값을 반환하지 않는 한 이 값은 새로운 표현식의 값이 됩니다.

---

참고

- 자바스크립트 스코프 - https://poiemaweb.com/js-scope
- Lexical Scope, Closure 정리 - [https://medium.com/sjk5766/lexical-scope-closure-정리-41f5d1c928e4](https://medium.com/sjk5766/lexical-scope-closure-%EC%A0%95%EB%A6%AC-41f5d1c928e4)
- \***\*Master the JavaScript Interview: What is a Closure? -\*\*** https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36

여담

- 아는 내용이라고 생각했는데, 기술 면접에서 부족하거나 잘못 설명한 부분이 있어 이번에 바로 잡고자 정리합니다.
