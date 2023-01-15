---
layout: single
title: "[TIL] Typescript - Generics"
tag:
  - Typescript
  - TIL
  - Generics
---

## 개요

제네릭이란, 클래스 혹은 함수 내부에서 데이터 타입을 정의하지 않고, 외부에서 정의하는 기법을 말합니다. Java, C#, flutter를 비롯하여 typescript에도 제네릭이 존재합니다.

## 제네릭 알아보기

우선, 인수로 무엇이 들어오던 그대로 반환하는 identity 함수를 만들어봅시다. 현재 제네릭을 사용하지 않았기에 인자의 type과 반환형의 type을 정의해야합니다.

```tsx
function identity(arg: number): number {
  return arg;
}
```

```tsx
console.log(identity(3)); //---> 3
console.log(identity("5")); //---> 컴파일 error
```

혹은 단순히 any로 할 수 있습니다. 아무 타입을 다 받을 수 있다는 점에서 제네릭으로 볼 수 있겠지만, 실제로 데이터가 반환될 때, 타입 정보가 소실되는 문제가 있습니다. 인수로 number 타입을 전달해도 항상 any 타입이 리턴된다는 정보가 끝입니다.

```tsx
function identity(arg: any): any {
  return arg;
}
```

```tsx
console.log(identity(3)); //---> 3
console.log(identity("5")); //---> '5'
```

any 대신, 타입 변수를 사용하여 인수의 타입을 캡쳐해봅시다.

```tsx
function identity<Type>(arg: Type): Type {
  return arg;
}
```

인수의 타입을 캡쳐하는 Type 변수 덕분에 typescript의 인텔리센스가 타입 정보를 추론할 수 있게됩니다. any와는 달리 어떤 정보도 잃어버리지 않습니다.

제네릭 함수를 호출하는 방법에는 두가지 방법이 있습니다.

Type을 명시적으로 꺽쇠 기호 <> 를 활용하여 전달해주는 방법입니다. 개발자 및 컴파일러가 명확하게 타입을 추론할 수 있습니다.

```tsx
let output = identity<string>("myString"); // 출력 타입은 'string'입니다.
```

컴파일러가 타입을 추론하게 하는 방법입니다. 전달하는 값에 따라 컴파일러가 알아서 타입을 추론하는 방법입니다. 좀 더 간결하지만 필요에 따라 위의 방법을 사용해야하는 경우가 있습니다.

```tsx
let output = identity("myString"); // 출력 타입은 'string'입니다.
```

## 제네릭 타입

제네릭 타입의 함수는 비 제네릭 함수와 유사합니다.

```tsx
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Type>(arg: Type) => Type = identity;
```

타입변수의 수와, 타입 변수가 사용되는 방식에 따라 다른 이름을 사용할 수 있습니다.

```tsx
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Input>(arg: Input) => Input = identity;
```

객체 리터럴 타입의 함수 호출 시그니처로 작성할 수도 있습니다.

```tsx
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: { <Type>(arg: Type): Type } = identity;
```

위에서 설명한 특성들로 제네릭 인터페이스를 작성할 수 있습니다.

```tsx
interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

제네릭 매개변수를 전체 인터페이스의 매개변수로 옮기고 싶다면, 아래 예제를 봅시다. 인터페이스의 다른 모든 멤버가 타입 매개변수를 볼 수 있습니다. 아래 예제는 `number` 타입을 전달하여 타입을 number로 제한한 상태입니다.

```tsx
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

## 제네릭 클래스

제네릭 클래스와 제네릭 인터페이스는 형태가 비슷합니다. 제네릭 클래스는 클래스 이름 뒤에 꺾쇠괄호(`<>`
) 안쪽에 제네릭 타입 매개변수 목록을 가집니다.

```tsx
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

좀 더 자유로운 타입을 사용할 수 있는 예제입니다.

```tsx
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
  return x + y;
};

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

refer

[https://www.typescriptlang.org/ko/docs/handbook/2/generics.html](https://www.typescriptlang.org/ko/docs/handbook/2/generics.html)

[https://poiemaweb.com/typescript-generic](https://poiemaweb.com/typescript-generic)
