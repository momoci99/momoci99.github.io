---
layout: single
title:  "TypeScript 5분만에 살펴보기"
tag : 
    - TypeScript
---

이 글은 [TypeScript in 5 minutes](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)을 참고하였습니다.


## 들어가며...

새해 목표중 하나인 TypeScript를 배워보고자 TypeScript를 가볍게 살펴보려고 한다.

## 설치

> npm install -g typescript

로컬로 설치하고 싶다면 `-g` 를 빼고 설치하면 된다. 물론 나는 글로벌 설치를 하기 위해 그대로 두었다.

## TypeScript file 컴파일

TypeScript로 작성된 파일은 `.ts` 확장자를 가지며 javascript 로 컴파일이 가능하다. 

> tsc {컴파일하려는파일명}.ts

예
![exam1](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/typescript/exam1.PNG?raw=true)


```ts
function greeter(person: string) {
    return "Hello, " + person;
}

let user = "Jane User";

document.body.textContent = greeter(user);
```
위와 같은 코드를 `tsc` 명령어로 컴파일하면 아래와 같이 변경된다.


```js
function greeter(person) {
    return "Hello, " + person;
}
var user = "Jane User";
document.body.textContent = greeter(user);
```

## Type annotations

javascirpt와 Typescript간의 두드러지는 차이중 하나가 이 타입에 대한 명시이다. 아래 예를 통해 살펴보도록 하자

```ts
function greeter(person: string) {
    return "Hello, " + person;
}

let user = [0, 1, 2];

document.body.textContent = greeter(user);
```
`greeter` 함수의 person 파라미터 형식이 `string`으로 지정되어 있다.

이 상태에서 Array인 `user` 변수를 인자로 넘기는 코드이다. 즉 잘못된 함수 호출방식이며 컴파일 시도시 에러를 발생시킨다.

![exam2](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/typescript/exam2.PNG?raw=true)

> error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.

단, 컴파일 과정에서 에러가 발생하였다 하더라도 컴파일 결과물인 .js파일은 만들어지는것에 주의하자.

## Interfaces
과거 javascript에서는 인터페이스를 지원하는 명세가 없어 이를 구현하기 위해 prototype을 조합한 다른 방법을 사용해야했다.

TypeScript는 인터페이스를 명시적으로 지원한다.


- interface 예시
```ts
interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person: Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = { firstName: "Jane", lastName: "User" };

document.body.textContent = greeter(user);

```
typescript의 interface에 대해 더 많은 내용이 있지만 이번에는 가볍게 다루고 넘어가기 위해 여기까지만 정리하고 다음기회에 제대로 다뤄야겠다.


## Class

typescript에서는 `Class`도 지원한다.

```ts
class Student {
    fullName: string;
    constructor(public firstName: string, public middleInitial: string, public lastName: string) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
}

interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person: Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", "User");

document.body.textContent = greeter(user);
```

아래는 위 코드를 컴파일 한 결과이다.

```js
var Student = /** @class */ (function () {
    function Student(firstName, middleInitial, lastName) {
        this.firstName = firstName;
        this.middleInitial = middleInitial;
        this.lastName = lastName;
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
    return Student;
}());
function greeter(person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}
var user = new Student("Jane", "M.", "User");
document.body.textContent = greeter(user);

```

컴파일 된 .js 코드는 기존 javascript파일 사용하듯 그대로 사용하면 된다.

```html
<!DOCTYPE html>
<html>
    <head><title>TypeScript Greeter</title></head>
    <body>
        <script src="greeter.js"></script>
    </body>
</html>
```


## 마무리

정말 간단하게 TypeScript의 요소를 살펴보았다. 확실히 기존 정적 타입 언어처럼 TypeChecking이 가능해 보이고 Class, Interface 또한 사용이 가능하다. 앞으로 하나씩 TypeScript에 대해 다뤄볼 계획이다.