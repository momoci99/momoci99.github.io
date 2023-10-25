---
layout: single
title: "[Typescript] Effective Typescript 3장 - 타입 추론"
tag:
  - Effective Typescript
---

이펙티브 타입스크립트 3장 내용을 정리하였습니다.

# 3. 타입 추론

타입 추론에서 발생할 수 있는 몇가지 문제와 그 해법을 안내.

숙련자와 초보자의 차이는 타입 구문의 수에서 차이를 보임. (숙련자일수록 필요한 부분에만 사용)

## 19. 추론 가능한 타입을 사용해 장황한 코드 방지하기

타입스크립트의 많은 구문은 사실 불필요함.

다음과 같이 코드의 모든 변수에 타입을 선언하는 것은 비생산적이며 형편없는 스타일로 여겨짐

```tsx
let x: number = 12;
```

```tsx
//이정도로 하여도 충분함
let x = 12;
```

- 편집기를 통해 확인하면 이미 타입이 number로 추론되어 있음.

```tsx
const person: {
  name: string;
  born: {
    where: string;
    when: string;
  };
  died: {
    where: string;
    when: string;
  } = {
    name: "Sojourner Truth",
    born: {
      where: "Swartekill, NY",
      when: "c.1797",
    },
    died: {
      where: "Battle Creek, MI",
      when: "Nov. 26, 1883",
    },
  };
};
```

- 타입 선언이 위와 같을때 타입을 생략하고 작성해도 충분함.

```tsx
//타입 생략 버전
const person = {
  name: "Sojourner Truth",
  born: {
    where: "Swartekill, NY",
    when: "c.1797",
  },
  died: {
    where: "Battle Creek, MI",
    when: "Nov. 26, 1883",
  },
};
```

- 두 예제에서 Person의 타입은 동일함.
- 값에도 추가로 타입을 작성하는 것은 거추장스러움

```tsx
function square(nums: number[]) {
  return nums.map((x) => x * x);
}
const squares = square([1, 2, 3, 4]); // 타입은 number[]
```

- 배열의 경우도 객체와 마찬가지로, 타입스크립트는 입력받아 연산을 하는 함수가 어떤 타입을 반환하는지 정확히 알고 있음.

```tsx
const axis1: string = "x"; //타입은 string
const axis2 = "y"; //타입은 "y"
```

- axis2 변수를 string으로 예상하기 쉽지만 타입스크립트가 추론한 “y”가 더 정확한 타입.

```tsx
interface Product {
  id: number;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id;
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

- 타입 추론이되면 리팩터링 역시 하기 쉬움

```tsx
interface Product {
  id: string;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id;
  //Type 'string' is not assignable to type 'number'.
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

- id에 문자도 들어올 수 있음을 나중에 알게되었다고 가정.
- 그래서 Product내의 id의 타입을 변경. logProduct내의 id 변수 선언에 있는 타입과 맞지 않기 때문에 오류가 발생
- 만약 logProduct 함수 내의 명시적 타입 구분이 없었다면, 코드는 아무런 수정 없이도 타입 체커를 통과했을것.

```tsx
function logProduct(product: Product) {
  const { id, name, price } = product;
  console.log(id, name, price);
}
```

- logProduct는 비구조화 할당문을 사용해 구현하는게 나음.

```tsx
function logProduct(product: Product) {
  const { id, name, price }: { id: string; name: string; price: number } =
    product;
  console.log(id, name, price);
}
```

- 여기에 추가적으로 명시적 타입 구문을 넣는다면 불필요한 타입 선언으로 인해 코드가 번잡해짐.

- 정보가 부족하여 타입스크립트가 스스로 타입을 판단하기 어려운 상황도 일부 있음.
- 그럴 때는 명시적 타입 구문이 필요함.
- logProduct 함수에서 매개변수 타입을 product로 명시한 경우가 그 예

- 어떤 언어들은 매개변수의 최종 사용처까지 참고하여 타입을 추론.
- 타입스크립트는 최종 사용처까지 고려하지 않음.
- 타입스크립트에서 변수의 타입은 일반적으로 처음 변수가 등장할 때 결정.

- 이상적인 타입스크립트 코드는 함수/메서드 시그니처에 타입 구문을 포함
- 함수 내에서 생성된 지역 변수에는 타입 구문을 넣지 않음.
- 타입 구문을 생략하여 방해되는 것들을 최소화하고 코드를 읽는 사람이 구현 로직에 집중할 수 있게 함.

e.g : 간혹 함수 매개변수에 타입 구문을 생략하는 경우

```tsx
function parseNumber(str: string, base = 10) {
  //...
}
```

- 여기서 기본값이 10이므로 타입은 number로 추론됨.

```tsx
//anti-pattern
app.get("/health", (request: express.Request, response: express.Response) => {
  response.send("OK");
});

// good-pattern
app.get("/health", (request, response) => {
  response.send("OK");
});
```

- 보통 타입 정보가 있는 라이브러리에서, 콜백 함수의 매개변수 타입은 자동으로 추론됨.
- request, response의 타입 선언은 필요하지 않음.

```tsx
const elmo: Product = {
  name: "Tickle Me Elmo",
  id: "048188 627152",
  price: 28.99,
};
```

- 타입이 추론될 수 있음에도 여전히 타입을 명시하고 싶은 몇가지 상황 중 하나는 객체 리터럴을 정의할 때.
- 이런 정의에 타입을 명시하면, 잉여 속성 체크가 동작
- 잉여 속성 체크는 선택적 속성이 있는 타입의 오타 같은 오류를 잡는 데 효과적.
- 변수가 사용되는 순간이 아닌 할당하는 시점에 오류가 표시되도록 해줌.

```tsx
const furby = {
  name: "Furby",
  id: 635045681526,
  price: 28.99,
};
logProduct(furby);
//error
/*
Argument of type '{ name: string; id: number; price: number; }' is not assignable to parameter of type 'Product'.
  Types of property 'id' are incompatible.
    Type 'number' is not assignable to type 'string'.
*/
```

- 만약 타입 구문을 제거한다면 잉여 속성 체크가 동작하지 않고, 객체를 선언한 곳이 아니라 객체가 사용되는 곳에서 타입 오류가 발생

- 함수의 반환에도 타입을 명시하여 오류를 방지할 수 있음.
- 타입 추론이 가능할지라도 구현상의 오류가 함수를 호출한 곳 까지 영향을 미치지 않도록 하기 위해 타입 구문을 명시하는게 좋음.

```tsx
//주식 시세를 조회하는 함수를 작성했다고 가정
function getQuote(ticker: string) {
  return fetch(`https://quotes.example.com/?q=${ticker}`).then((response) =>
    response.json()
  );
}
```

```tsx
//이미 조회한 종목을 다시 요청하지 않도록 캐시를 추가함.
const cache: { [ticker: string]: number } = {};
function getQuote(ticker: string) {
  if (ticker in cache) {
    return cache[ticker];
  }
  return fetch(`https://quotes.example.com/?q=${ticker}`)
    .then((response) => response.json())
    .then((quote) => {
      cache[ticker] = quote;
      return quote;
    });
}
```

- 이 코드에는 오류가 있음.
- getQuote는 항상 Promise를 반환하므로 if 구문에는 cache[ticker]가 아니라 Promise.resolve(cache[ticker])가 반환되도록 해야함.

```tsx
getQuote("MSFT").then(considerBuying);
//error
//Property 'then' does not exist on type 'number | Promise<any>'.
//  Property 'then' does not exist on type 'number'
```

- 실행해 보면 오류는 getQuote 내부가 아닌 getQuote를 호출한 코드에서 발생.

```tsx
const cache: { [ticker: string]: number } = {};
function getQuote(ticker: string): Promise<number> {
  if (ticker in cache) {
    return cache[ticker];
    //error
    //Type 'number' is not assignable to type 'Promise<number>'.
  }
  return fetch(`https://quotes.example.com/?q=${ticker}`)
    .then((response) => response.json())
    .then((quote) => {
      cache[ticker] = quote;
      return quote;
    });
}
```

- 의도된 반환 타입(Promise<number>)을 명시한다면, 정확한 위치에 오류가 표시됨.

- 반환 타입을 명시하면, 구현상의 오류가 사용자 코드의 오류로 표시되지 않음
- Promise와 관련된 특정 오류를 피하는 데는 async 함수가 효과적

반환 타입을 명시해야하는 이유

- 반환 타입을 명시하면 함수에 대해 더욱 명확하게 알 수 있기 때문
- 반환 타입을 명시하려면 구현하기 전에 입력 타입과 출력 타입이 무엇인지 알아야함.
- 추후에 코드가 조금 변경되어도 그 함수의 시그니처는 쉽게 바뀌지 않음.
- 미리 타입을 명시하는 방법은, 함수를 구현하기 전에 테스트를 먼저 작성하는 테스트 주도 개발(TDD)와 비슷함.
- 전체 타입 시그니처를 먼저 작성하면 구현에 맞추어 주먹구구식으로 시그니처가 작성되는 것을 방지하고 제대로 원하는 모양을 얻게됨.
- 반환 타입을 명시하면 더욱 직관적인 표현이 됨.

```tsx
interface Vector2D {
  x: number;
  y: number;
}
function add(a: Vector2D, b: Vector2D) {
  return { x: a.x + b.x, y: a.y + b.y };
}
```

- 타입 스크립트는 해당 코드의 반환 타입을 {x: number, y:number} 로 추론.
- 이 경우 Vector2D와 호환되지만, 입력이 Vector2D인데 반해 출력은 Vector2D가 아니기 때문에 유저 입장에서는 당황스러움
- 이때 반환 타입을 명시하면 더욱 직관적인 표현이 됨.
- 반환 값을 별도의 타입으로 정의하면 타입에 대한 주석을 잓어할 수 있어서 더욱 자세한 설명이 가능.
- 린터를 사용하고 있다면 eslint 규칙 중 no-inferrable-types 사용하여 모든 타입 구문이 정말로 필요한건지 확인 가능

요약

- 타입스크립트가 타입을 추론할 수 잇다면 타입 구문을 작성하지 않는게 좋음.
- 이상적인 경우 함수/메서드의 시그니처에는 타입 구문이 있지만, 함수 내의 지역 변수에는 타입 구문이 없음.
- 추론될 수 있는 경우라도 객체 리터럴과 함수 반환 타입에는 타입 명시를 고려해야함. 이는 내부 구현의 오류가 사용자 코드 위치에 나타나는 것을 방지함.

## 20. 다른 타입에는 다른 변수 사용하기

자바스크립트에서는 한 변수를 다른 목적을 가지는 다른 타입으로 재사용해도 됨.

```tsx
let id = "12-34-56";
fetchProduct(id); // string으로 사용
id = 123456;
fetchProductBySerialNumber(id); // number로 사용
```

```tsx
//typescript 코드
let id = "12-34-56";
fetchProduct(id);

//Error - Type 'number' is not assignable to type 'string'.
id = 123456;
fetchProductBySerialNumber(id);
```

- 타입스크립트는 “12-34-56”이라는 값을 보고 id의 타입을 string으로 추론. string타입에는 number 타입을 할당할 수 없으므로 오류 발생
- 변수의 값은 바뀔 수 있지만, 그 타입은 보통 바뀌지 않는다는 관점을 알 수 있음.
- 타입을 바꿀 수 있는 한가지 방법은 범위를 좁히는 것. 새로운 변수값을 포함하도록 확장하는 것이 아니라 타입을 더 적게 제한하는것.

```tsx
//에러를 수정한 코드
let id: string | number = "12-34-56";
fetchProduct(id);
id = 123456; //정상
fetchProductBySerialNumber(id); //정상
```

- 타입스크립트는 첫 번째 함수 호출에서 id는 string으로, 두 번째 호출에서는 number라고 제대로 판단.
- 할당문에서 유니온 타입으로 범위가 좁혀졌기 때문.

```tsx
const id = "12-34-56";
fetchProduct(id);

const serial = 123456; //정상
fetchProductBySerialNumber(serial); //정상
```

- 유니온 타입으로 코드는 동작하기는 하겠지만, 더 많은 문제가 생길 수 있으.ㅁ
- id를 사용할 때 마다 값이 어떤 타입인지 확인해야 하기 때문에 유니온 타입은 string, number같은 간단한 타입에 비해 다루기 더 어려움.
- 차라리 별도의 변수를 도입하는것이 더 나음.

다른 타입에는 별도의 변수를 상용하는게 바람직한 이유

- 서로 관련이 없는 두 개의 값을 분리(id와 serial).
- 변수명을 더 구체적으로 지을 수 있음.
- 타입 추론을 향상시키며, 타입 구문이 불필요해짐.
- 타입이 좀 더 간결해짐(string|number 대신 string과 number를 사용).
- let 대신 const로 변수를 선언하게 됨. 코드가 간결해지고 타입 체커가 타입을 추론하기에도 좋음.

e.g 가려지는 변수(shadowed)

```tsx
const id = "12-34-56";
fetchProduct(id);

{
  const id = 123456; //정상
  fetchProductBySerialNumber(id); //정상
}
```

- 여기서는 두 id는 이름이 값지만 실제로는 서로 아무 관련이 없음.
- 타입스크립트 코드는 잘 동작하겠으나, 사람에게 혼란을 줄 수 있음.
- 목적이 다른 곳에는 별도의 변수명을 사용하기 바람.
- 대다수의 개발팀이 린터 규칙을 통해 ‘가려지는’ 변수를 사용하지 못하도록함.

요약

- 변수의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않음.
- 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록함.

## 21. 타입 넓히기

타입 스크립트가 작성된 코드를 체크하는 정적 분석 시점에, 변수는 가능한 값들의 집합인 타입을 가짐.

상수를 사용해서 변수를 초기화 할 때 타입을 명시하지 않으면 타입 체커는 타입을 결정해야함.

지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야함.

타입 스크립트에서는 이러한 과정을 넓히기(widening)이라고 부름.

→ 넓히기의 과정을 이해한다면 오류의 원인을 파악하고 타입 구문을 더 효과적으로 사용할 수 있음.

```tsx
//벡터를 다루는 라이브러리 코드 가정
interface Vector3 {
  x: number;
  y: number;
  z: number;
}
function getComponent(vector: Vector3, axis: "x" | "y" | "z") {
  return vector[axis];
}
```

- Vector3함수를 사용한 다음 코드는 런타임에 오류 없이 실행

```tsx
let x = "x";
let vec = { x: 10, y: 20, z: 30 };
getComponent(vec, x);
//Error - Argument of type 'string' is not assignable to parameter of type '"x" | "y" | "z"'.
```

- 그러나 편집기에서 오류 표시
- getComponent함수는 두 번째 매개변수에 “x” | “y” | “z” 을 기대했지만 x의 타입은 할당 시점에 넓히기가 동작해서 string으로 추론.
- string 타입은 “x” | “y” | “z” 타입에 할당이 불가능하므로 오류가 된 것.
- 타입 넓히기가 진행될 때, 주어진 값으로 추론 가능한 타입이 여러 개이기 때문에 과정이 상당히 모호.

```tsx
const mixed = ["x", 1];
```

- mixed 의 타입이 될 수 있는 후보군
  - (’x’ | 1)[]
  - [’x’, 1]
  - [string, number]
  - readonly [string, number]
  - (string|number)[]
  - readonly (string|number)[]
  - [any, any]
  - any[]
- 정보가 충분하지 않다면 mixed가 어떤 타입으로 추론되어야 하는지 알 수 없음.
- 그러므로 타입 스크립트는 작성자의 의도를 추측(이 경우 (string|number)[]으로 추측)
- 그러나 타입스크립트가 아무리 영리하더라도 사람의 마음까지 읽ㅇ르 수 없고, 따라서 추측한 답이 항상 옳을 수도 없음.

```tsx
let x = "x";
x = "a";
x = "Four asdf asdfewer";
```

- 타입 스크립트는 다음 예제와 같은 코드를 예상했기 때문에 x의 타입을 string으로 추론.

```tsx
let x = "x";
x = /x|y|z/;
x = ["x", "y", "z"];
```

- 자바스크립트는 위와 같이 작성해도 유효함.
- 타입스크립트는 x의 타입을 string으로 추론할 때, 명확성과 유연성 사이의 균형을 유지하려고함.
- 일반적인 규칙은 변수가 선언된 후로는 타입이 바뀌지 않아야하므로 string|RegExp나 string|string[] 이나 any보다는 string을 사용하는게 나음.

```tsx
//오류 해결
const x = "x"; //타입이 "x"
let vec = { x: 10, y: 20, z: 30 };
getComponent(vec, x); //정상
```

- 타입스크립트는 넓히기의 과정을 제어할 수 있도록 몇가지 방법을 제공
- 넓히기 과정을 제어할 수 있는 첫 번째 방법은 const
- 만약 let 대신 const로 변수를 선언하면 더 좁은 타입이 됨.
- 이제 x는 재할당될 수 없으므로, 타입스크립트는 의심의 여지 없이 더 좁은 타입(”x”)으로 추론할 수 있음.
- 문자 리터럴 타입 “x”는 “x”|”y”|”z”에 할당 가능하므로 코드가 타입 체커를 통과함.

- 그러나 const는 만능이 아님. 객체와 배열의 경우에는 여전히 문제가 있음.
- 아이템 초반에 있는 mixed 예제(const mixed = [’x’, 1];)는 배열에 대한 문제를 보여줌
- 튜플 타입을 추론해야할 지, 요소들은 어떤 타입으로 추론해야 할지 알 수 없음.
- 비슷한 문제가 객체에도 발생.

```tsx
//자바스크립트는 정상으로 처리
const v = {
  x: 1,
};
v.x = 3;
v.x = "3";
v.y = 4;
v.name = "Pythagoras";
```

- v의 타입은 구체적인 정도에 따라 다양한 모습으로 추론될 수 있음
- 가장 구체적인 경우라면 {readonly x:1}
- 추상적으로는 {x:number}, {[key: string] number} 또는 object

```tsx
const v = {
  x: 1,
};

v.x = 3;
v.x = "3";
//Error - Type 'string' is not assignable to type 'number'
v.y = 4;
//Error - Property 'y' does not exist on type '{ x: number; }'.
v.name = "Pythagoras";
//Error - Property 'name' does not exist on type '{ x: number; }'.
```

- 객체의 경우 타입스크립트의 넓히기 알고리즘은 각 요소를 let으로 할당된 것처럼 다룸.
- 그래서 v의 타입은 {x:number}가 됨. 덕분에 v.x를 다른 숫자로 재할당 할 수 있게 되지만 string으로는 안됨.
- 그리고 다른 속성을 추가할 수도 없음.

- 타입스크립트는 명확성과 유연성 사이의 균형을 유지하려고 함.
- 오류를 잡기 위해서는 충분히 구체적으로 타입을 추론해야 하지만, 잘못된 추론(false positive)을 할 정도로 구체적으로 수행하지는 않음.
- 예를 들어 1과 같은 값으로 초기화되는 속성을 적당히 number의 타입으로 추론함.

타입의 추론 강도를 직접 제어하려면 타입스크립트의 기본 동작을 재정의 해야하는데 세가지 방법이 있음.

1. 명시적 타입 구문을 제공

```tsx
const v: { x: 1 | 3 | 5 } = { x: 1 }; //타입이 {x: 1|3|5};}
```

1. 타입 체커에 추가적인 문맥을 제공 (함수의 매개변수로 값을 전달)
2. const 단언문을 사용하는것
   - const 단언문과 변수 선언에 쓰이는 let이나 const와 혼동해서는 안됨.
   - const 단언문은 온전히 타입 공간의 기법.

```tsx
const v1 = {
	x: 1,
	y: 2,
}; //타입은 {x:number; y:number}

const v2 = {
	x: 1 as const;
	y: 2,
}; //타입은 { x: 1; y:number;}

const v3 = {
	x:1,
	x:2,
} as const; //타입은 {readonly x: 1; readonly y: 2;}
```

- 값 뒤에 as const를 작성하면 타입스크립트는 최대한 좁은 타입으로 추론함.
- v3에서는 넓히기가 동작하지 않았음. v3가 진짜 상수라면, 주석에 보이는 추론된 타입이 실제로 원하는 형태일것.
- 또한 배열을 튜플 타입으로 추론할 때에도 as const를 사용할 수 있음.

```tsx
const a1 = [1, 2, 3]; //타입이 number[]
const a2 = [1, 2, 3] as const; //타입이 readonly [1,2,3]
```

- 넓히기로 인해 오류가 발생한다고 생각되면, 명시적 타입 구문 또는 const 단언문을 추가하는 것을 고려해야함.
- 단언문으로 인해 추론이 어떻게 변화하는지 편집기에서 주기적으로 타입을 살펴볼 것.

### 요약

- 타입스크립트가 넓히기를 통해 상수의 타입을 추론하는 법을 이해해야함.
- 동작에 영향을 줄 수 있는 방법인 const, 타입 구문, 문맥, as const에 익숙해야함.

## 22. 타입 좁히기

- 타입 넓히기의 반대는 타입 좁히기.
- 타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말함.
- 가장 일반적인 예시는 null 체크

```tsx
const el = document.getElementById("foo"); //타입이 HTMLElement | null
if (el) {
  el; //타입이 HTMLElement
  el.innerHTML = "Party Time".blink();
} else {
  el; //타입이 null
  alert("No element #foo");
}
```

- 만약 el이 null이라면, 분기문의 첫 번째 블록이 실행되지 않음. 즉 첫 번째 블록에서 HTMLElement | null 타입의 null을 제외하므로, 더 좁은 타입이 되어 작업이 훨씬 쉬워짐.
- 타입 체커는 일반적으로 이러한 조건문에서 타입 좁히기를 잘 해내지만, 타입 별칭이 있다면 그러지 못할 수도 있음.

```tsx
const el = document.getElementById("foo"); //타입이 HTMLElement | null
if (!el) throw new Error("Unable to find #foo");
el; //이제 타입은 HTMLElement
el.innerHTML = "Party Time".blink();
```

- 분기문에서 예외를 던지거나 함수를 반환하여 블록의 나머지 부분에서 변수의 타입을 좁힐 수 있음.

```tsx
function contains(text: string, search: string | RegExp) {
  if (search instanceof RegExp) {
    search; //타입이 RegExp
    return !!search.exec(text);
  }
  search; //타입이 string
  return text.includes(search);
}
```

- instanceof를 사용해서 타입을 좁히는 예제

```tsx
interface A {
  a: number;
}
interface B {
  b: number;
}
function pickAB(ab: A | B) {
  if ("a" in ab) {
    ab; //타입이 A
  } else {
    ab; //타입이 B
  }
  ab; //타입이 A | B
}
```

- 속성 체크로도 타입을 좁힐 수 있음.

```tsx
function contains(text: string, terms: string | string[]) {
  const termList = Array.isArray(terms) ? terms : [terms];
  termList; //타입이 string[]
  //...
}
```

- Array.isArray같은 일부 내장 함수로도 타입을 좁힐 수 있음.

- 타입스크립트는 일반적으로 조건문에서 타입을 좁히는 데 매우 능숙함.
- 그러나 타입을 섣불리 판단하는 실수를 저지르기 쉬우므로 다시 한번 꼼곰히 따져봐야함.

```tsx
const el = document.getElementById("foo"); //타입이 HTMLElement | null
if (typeof el === "object") {
  el; //타입이 HTMLElement | null
}
```

- 해당 예제는 유니온 타입에서 null을 제외하기 위해 잘못된 방법을 사용
- 자바스크립트에서 typeof null이 “object” 이므로 if 구문에서 null이 제외되지 않음.

```tsx
function foo(x?: number | string | null) {
  if (!x) {
    x; //타입이 string|number|null|undefined
  }
}
```

- 또한 기본형 값이 잘못되어도 비슷한 사례가 발생함.
- 빈 문자열 ‘’과 0 모두 false가 되기 때문에, 타입은 전혀 좁혀지지 않았고 x는 여전히 블록 내에서 string또는 number가 됨.

```tsx
interface UploadEvent { type: 'upload'; filename: string; contents: string}
interface DownloadEvent { type: 'download'; filename: string;}
type AppEvent = UploadEvent | DownloadEvent;
function handleEvent(e: AppEvent) {
	switch (e.type){
		case 'download':
			e //타입이 DownloadEvent
			break;
		case 'upload';
			e //타입이 UploadEvent
			break;
	}

}
```

- 또다른 일반적인 방법은 명시적인 태그를 붙이는것.
- 이 패턴은 태그된 유니온(tagged union) 또는 구별된 유니온(discriminated union)이라고 불리며 타입스크립트 어디에서나 찾아 볼 수 있음.

e.g 만약 타입스크립트가 타입을 식별하지 못한다면, 식별을 돕기 위해 커스텀 함수를 도입할 수 있음.

```tsx
function isInputElemnt(el: HTMLElement): el is HTMLInputElement {
  return "value" in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el; //타입이 HTMLInputElement
    return el.value;
  }
  el; //타입이 HTMLElement
  return el.textContent;
}
```

- 이러한 기법을 `사용자 정의 타입 가드` 라고 함.
- 반환 타입의 el is HTMLInputElement는 함수의 반환이 true인 경우 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알려줌.

```tsx
const jackson5 = ["jackie", "Titi", "Jermaine", "Marlon", "Michael"];
const members = ["Janet", "Michael"].map((who) =>
  jackson5.find((n) => n === who)
); //타입이 (string | undefined)[]
```

- 어떤 함수들은 타입 가드를 사용하여 배열과 객체의 타입 좁히기를 할 수 있음.
- 예를 들어, 배열에서 어떤 탐색을 수행할 때 undefined가 될 수 있는 타입을 사용할 수 있음.

```tsx
const members = ["Janet", "Michael"]
  .map((who) => jackson5.find((n) => n === who))
  .filter((who) => who !== undefined); //타입이 (string | undefined)[]
```

- filter함수를 사용해 undefined를 걸러내려고 해도 잘 동작하지 않음.

```tsx
function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined;
}

const members = ["Janet", "Michael"]
  .map((who) => jackson5.find((n) => n === who))
  .filter(isDefined);
```

- 이럴 때 타입 가드를 사용하면 타입을 좁힐 수 있음

- 편집기에서 타입을 조사하는 습관을 가지면, 타입 좁히기가 어떻게 동작하는지 자연스럽게 익힐 수 있음.
- 타입스크립트에서 타입이 어떻게 좁혀지는지 이해한다면 타입 추론에 대한 개념을 잡을 수 있고, 오류 발생의 원인을 알 수 있으며, 타입 체커를 더 효율적으로 이용할 수 있음.

### 요약

- 분기문 외에도 여러 종류의 제어 흐름을 살펴보며 타입스크립트가 타입을 좁히는 과정을 이해해야함.
- 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원활하게 만들 수 있음.

## 23 한꺼번에 객체 생성하기

앞선 내용에서 다룬 것 처럼 변수의 값은 변경될 수 있지만, 타입 스크립트의 타입은 일반적으로 변경되지 않음.

이러한 특성으로 인해 일부 자바스크립트 패턴을 타입스크립트로 모델링하는게 쉬워짐.

객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입 추론에 유리함.

```jsx
//자바스크립트에서 2차원 점을 표현하는 객체를 생성하는 방법.
const pt = {};
pt.x = 3;
pt.y = 4;
```

```tsx
const pt = {};
pt.x = 3; //Error - Property 'x' does not exist on type '{}'.
pt.y = 4; //Error - Property 'y' does not exist on type '{}'.
```

- 타입스크립트에서는 각 할당문에 오류가 발생함.
- 첫번째 줄의 pt타입은 {} 값을 기준으로 추론되기 때문.
- 존재하지 않는 속성을 추가할 수는 없음.

e.g : 만약 Point인터페이스를 정의하는 경우

```tsx
interface Point {
  x: number;
  y: number;
}
const pt: Point = {};
//Error - Type '{}' is missing the following properties from type 'Point': x, y
pt.x = 3;
pt.y = 4;
```

- Point 인터페이스를 정의한다면 오류가 변경됨.

```tsx
const pt = {
  x: 3,
  y: 4,
}; //정상
```

- 객체를 한번에 정의하면 해결할 수 있음.

```tsx
const pt = {} as Point;
pt.x = 3;
pt.y = 4; //정상
```

- 객체를 반드시 제각각 나눠서 만들어야한다면, 타입 단언문(as)을 사용해 타입 체커를 통과하게 할 수 있음.

```tsx
const pt: Point = {
  x: 3,
  y: 4,
};
```

- 이 경우에도 선언할 때 객체를 한꺼번에 만드는게 더 나음.

```tsx
const pt = { x: 3, y: 4 };
const id = { name: "Pythagoras" };
const namedPoint = {};
Object.assign(namedPoint, pt, id);
namedPoint.name;
//Error - Property 'name' does not exist on type '{}'.
```

- 작은 객체들을 조합해서 큰 객체를 만들어야 하는 경우에도 여러 단계를 거치는 것은 좋지 않은 생각.

```tsx
const namedPoint = {...pt, ...id};
namedPoint.name = //정상, 타입이 string
```

- 객체 전개 연산자 (…) 를 사용하면 큰 객체를 한꺼번에 만들어 낼 수 있음.
- 이때 모든 업데이트마다 새 변수를 사용하여 각각 새로운 타입을 얻도록 하는게 중요함.

```tsx
const pt0 = {};
const pt1 = { ...pt0, x: 3 };
const pt: Point = { ...pt1, y: 4 }; //정상
```

- 간단한 객체를 만들기 위해 우회하기는 했지만, 객체에 속성을 추가하고 타입스크립트가 새로운 타입을 추론할 수 있게 해 유용함.

```tsx
declare let hasMiddle: boolean;
const firstLast = { first: "Harry", last: "Truman" };
const president = { ...firstLast, ...(hasMiddle ? { middle: "S" } : {}) };
```

- 타입에 안전한 방식으로 조건부 속성을 추가하려면, 속성을 추가하지 않는 null 또는 {} 으로 객체 전개를 사용하면 됨.

```tsx
const president: {
  middle?: string;
  first: string;
  last: string;
};
```

- 타입스크립트는 president 의 타입을 선택적 속성을 가진것으로 추론함.

```tsx
declare let hasDates: boolean;
const nameTitle = { name: "Khufu", title: "Pharaoh" };
const pharaoh = {
  ...nameTitle,
  ...(hasDates ? { start: -2589, end: -2566 } : {}),
};
```

- 전개 연산자로 한꺼번에 여러 속성을 추가할 수도 있음.

```tsx
const pharaoh:
  | {
      start: number;
      end: number;
      name: string;
      title: string;
    }
  | {
      name: string;
      title: string;
    };
```

- 타입스크립트는 pharaoh를 유니온으로 추론함.
- start와 end가 선택적 필드이기를 원했다면 이런 결과가 당황스러울 수 있음.
- 이 타입에서는 start를 읽을 수 없음.

e.g 선택적 필드 방식으로 표현하기 위한 헬퍼 함수 사용

```tsx
function addOptional<T extends object, U extends object>(
  a: T,
  b: U | null
): T & Partial<U> {
  return { ...a, ...b };
}

const pharaoh = addOptional(
  nameTitle,
  hasDates ? { start: -2589, end: -2566 } : null
);
pharaoh.start; //정상
```

- 간혹 객체나 배열을 변환해서 새로운 객체나 배열을 생성하고 싶을 수 있음.
- 이런 경우 루프 대신 내장된 함수형 기법 또는 로대시(Lodash)같은 유틸리티 라이브러리를 사용하는것이 한꺼번에 객체 생성하기 관점에서 보면 옳음.

## 요약

- 속성을 제각기 추가하지 말고 한꺼번에 객체로 만들어야함. 안전한 타입으로 속성을 추가하려면 객체 전개({…a, …b})를 사용하면됨.
- 객체에 조건부로 속성을 추가하는 방법을 익힐것.

e.g: 어떤 값에 새 이름을 할당하는 예제

```tsx
const borough = { name: "Brooklyn", location: [40, 688, -73.979] };
const loc = borough.location;
```

- brough.location 배열에 loc이라는 별칭(alias)를 생성.

```tsx
> loc[0] = 0;
> brough.location
[0, -73.979]
```

- 별칭의 값을 변경하면 원래 속성값에서도 변경됨.
- 그런데 별칭을 남발해서 사용하면 제어 흐름을 분석하기가 어려움.
- 모든 언어의 컴파일러 개발자들은 무분별한 별칭 사용으로 골치를 썩고 있음.
- 타입스크립트에서도 마찬가지로 별칭을 신중하게 사용해야함.
- 그래야 코드를 잘 이해할 수 있고, 오류도 쉽게 찾을 수 있음.

e.g 다각형을 표현하는 자료구조를 가정

```tsx
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[][];
  bbox?: BoundingBox;
}
```

- 다각형의 기하학적 구조는 exterior와 holes속성으로 정의됨.
- bbox는 필수가 아닌 최적화 속성.
- bbox 속성을 사용하면 어떤 점이 다각형에 포함되는지 빠르게 체크할 수 있음.

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  if (polygon.bbox) {
    if (
      pt.x < polygon.bbox.x[0] ||
      pt.x > polygon.bbox.x[1] ||
      pt.y < polygon.bbox.y[0] ||
      pt.y > polygon.bbox.y[1]
    ) {
      return false;
    }
  }
  //...
}
```

- 코드는 잘 동작(타입 체크 포함)하지만, 반복되는 부분이 존재함.
- 특히 polygon.bbox는 3줄에 걸쳐 5번이나 등장함.
- 다음 코드는 중복을 줄이기 위해 임시 변수를 뽑아낸 모습.

```tsx
//아래 코드는 strictNullChecks를 활성화 했다고 가정
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.box;
  if (polygon.box) {
    if (
      pt.x < box.x[0] ||
      pt.x > box.x[1] ||
      //Error 'box' is possibly 'undefined'.
      pt.y < box.y[0] ||
      pt.y > box.y[1]
    )
      //Error 'box' is possibly 'undefined'.
      return false;
  }
}
```

- 이 코드는 동작하지만 편집기에서 오류로 표시.
- 그 이유는 polygon.bbox를 별도의 box라는 별칭으로 만들었고, 첫 번째 예제에서는 잘 동작했던 제어 흐름 분석을 방해했기 때문

e.g : 어떤 동작이 이루어졌는지 box와 polygon.bbox의 타입을 조사.

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  polygon.bbox; //타입이 BoundingBox | undefined
  const box = polygon.bbox;
  box; //타입이 BoundingBox | undefined
  if (polygon.bbox) {
    polygon.bbox; //타입이 BoundingBox
    box; //타입이 BoundingBox | undefined
  }
}
```

- 속성 체크는 polygon.bbox의 타입을 정제했지만 box는 그렇지 않았기 때문에 오류가 발생.
- 이러한 오류는 “별칭은 일관성 있게 사용한다”는 기본 원칙(golden rule)을 지키면 방지할 수 있음.

e.g : 속성 체크에 box를 사용하도록 코드를 변경

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if (box) {
    if (
      pt.x < box.x[0] ||
      pt.x > box.x[1] ||
      pt.y < box.y[0] ||
      pt.y > box.y[1]
    ) {
      return false; //정상
    }
  }
}
```

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const { bbox } = polygon;
  if (bbox) {
    const { x, y } = bbox;
    if (
      pt.x < box.x[0] ||
      pt.x > box.x[1] ||
      pt.y < box.y[0] ||
      pt.y > box.y[1]
    ) {
      return false; //정상
    }
  }
  //...
}
```

- 객체 비 구조화를 이용하여 보다 간결한 문법으로 일관된 이름을 사용.
- 배열과 중첩된 구조에서도 역시 사용 가능.

객체 비 구조화 사용시 주의사항

- 전체 bbox속성이 아니라 x, y가 선택적 속성일 경우에 속성 체크가 더 필요함.
- 따라서 타입의 경계에 null 값을 추가하는 것이 좋음.
- bbox에는 선택적 속성이 적합했지만 holes는 그렇지 않음. holes가 선택적이라면 값이 없거나 빈 배열([])이었을 것. 차이가 없는데 이름을 구별한것
- 빈 배열은 ‘holes 없음’을 나타내는 좋은 방법

e.g : 별칭은 타입 체커뿐만 아니라 런타임에도 혼동을 야기할 수 있음

```tsx
const { bbox } = polygon;
if (!bbox) {
  calculatePolygonBbox(polygon); //polygon.bbox가 채워짐
  //이제 polygon.bbox와 bbox는 다른 값을 참조!!
}
```

e.g : 타입스크립트의 제어 흐름 분석시 객체 속성의 경우

```tsx
function fn(p: Polygon) {
  /* ... */
}

polygon.bbox; //타입이 BoundingBox | undefined
if (polygon.bbox) {
  polygon.bbox; //타입이 BoundingBox
  fn(polygon);
  polygon.bbox; //타입이 BoundingBox
}
```

- fn(polygon) 호출은 polygon.bbox를 제거할 가능성이 있으므로 타입을 BoundingBox | undefined로 되돌리는 것이 안전함.
- 그러나 함수를 호출할 때 마다 속성 체크를 반복해야 하기 때문에 좋지 않음.
- 그래서 타입스크립트는 함수가 타입 정제를 무효화하지 않는다고 가정. 그러나 실제로는 무효화될 가능성이 있음.
- polygon.bbox로 사용하는 대신 bbox지역 변수로 뽑아내서 사용하면 bbox의 타입은 정확히 유지되지만, polygon.bbox의 값과 같게 유지되지 않을 수 있음.

## 요약

- 별칭은 타입스크립트가 타입을 좁히는 것을 방해함. 따라서 변수에 별칭을 사용할 때는 일관되게 사용해야함.
- 비구조화 문법을 사용해서 일관된 이름을 사용하는것이 좋음.
- 함수 호출이 객체 속성의 타입 정제를 무효화 할 수 있다는 점을 주의해야함. 속성보다 지역 변수를 사용하면 타입 정제를 믿을 수 있음.

## 25. 비동기 코드에는 콜백 대신 async 함수 사용하기

- 과거의 자바스크립트에서는 비동기 동작을 모델링하기 위해 콜백을 사용.
- 그렇기 때문에 악명 높은 콜백 지옥(callback hell)을 필연적으로 마주할 수 밖에 없었음.

e.g : callback hell 의 예제

```tsx
fetchURL(url1, function (response1) {
  fetchURL(url2, function (response2) {
    fetchURL(url3, function (response3) {
      //...
      console.log(1);
    });
    console.log(2);
  });
  console.log(3);
});
console.log(4);

//로그
//4
//3
//2
//1
```

- 로그에 따라 실행의 순서는 코드의 순서와 반대. 이러한 콜백이 중첩된 코드는 직관적으로 이해하기 어려움.
- 요청들을 병렬로 실행하거나 오류상황을 빠져나오고 싶다면 더욱 혼란스러워짐.

- ES2015는 콜백 지옥을 극복하기 위해 프로미스(promise)개념을 도입.
- 프로미스는 미래에 가능해질 어떤 것을 나타냄(future라고도 부름)

e.g : 프로미스를 사용하여 코드를 개선

```tsx
const page1Promise = fetch(url1);
page1Promise
  .then((response1) => {
    return fetch(url2);
  })
  .then((response2) => {
    return fetch(url3);
  })
  .then((response3) => {
    // ...
  })
  .catch((error) => {
    // ...
  });
```

- 코드의 중첩이 적어졌고, 실행 순서도 코드 순서와 같아졌음.
- 또한 오류를 처리하기도. Promise.all 같은 고급 기법을 사용하기도 쉬워짐.

e.g : ES2017에서 도입된 async & await 키워드를 도입하여 콜백지옥을 더 간단하게 개선

```tsx
async function fetchPages() {
  const response1 = await fetch(url1);
  const response2 = await fetch(url2);
  const response3 = await fetch(url3);
  //...
}
```

- await 키워드는 각각의 프로미스가 처리(resolve)될 때까지 fetchPages 함수의 실행을 멈춤
- async 함수 내에서 await 중인 프로미스가 거절(reject)되면 예외를 throw함.

e.g : 일반적인 try/catch 구문을 사용.

```tsx
async function fetchPages() {
  try {
    const response1 = await fetch(url1);
    const response2 = await fetch(url2);
    const response3 = await fetch(url3);
    //...
  } catch (e) {
    //...
  }
}
```

- es5 또는 더 이전 버전을 대상으로 할 때, 타입스크립트 컴파일러는 async와 await가 동작하도록 정교한 변환을 수행.
- 다시 말해 타입스크립트는 런타임과 관계 없이 async/await를 사용할 수 있음.

콜백보다는 프로미스나 async/await를 사용해야하는 이유는 다음과 같음.

- 콜백보다는 프로미스가 코드를 작성하기 쉬움
- 콜백보다는 프로미스가 타입을 추론하기 쉬움

e.g : 병렬로 페이지를 로드하기 위해 Promise.all을 사용해서 프로미스를 조합

```tsx
async function fetchPages() {
  const [response1, response2, response3] = await Promise.all([
    fetch(url1),
    fetch(url2),
    fetch(url3),
  ]);
  // ...
}
```

- 이런 경우 await와 구조 분해할당이 매우 좋음.
- 타입스크립트는 세 가지 response 변수 각각의 타입을 Response로 추론함. 그러나 콜백 스타일로 동일한 코드를 작성하려면 더 많은 코드와 타입 구문이 필요함.

```tsx
function fetchPagesCB() {
	let numDone = 0;
	const responses: string[] = [];
	const done = () => {
		const [response1, response2, response3] = responses;
		//...
	}
};
const urls = [url1, url2, url3];
urls.forEach((url, i) => {
	fetchURL(url, r => {
		responses[i] = url;
		numDone++;
		if(numDone === urls.length) done();
		});
	});
}
```

- 이 코드에는 오류 처리를 포함하거나 Promise.all 같은 일반적인 코드로 확장하는 것은 쉽지 않음.

e.g : Promise.race를 사용한 예제

```tsx
function timeout(millis: number): Promise<nerver> {
  return new Promise((resolve, reject) => {
    setTimeout(() => reject("timeout"), millis);
  });
}

async function fetchWithTimeout(url: string, ms: number) {
  return Promise.race([fetch(url), timeout(ms)]);
}
```

- 한편 입력된 프로미스들 중 첫번째가 처리될 때 완료되는 [Promise.race](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)가 타입추론과 잘 맞음.
- Promise.race를 사용하여 프로미스에 타임아웃을 추가하는 방법은 흔하게 사용되는 패턴.

- 타입 구문이 없어도 fetchWithTimeout의 반환 타입은 Promise<Response>로 추론됨.
- 추론이 동작하는 이유를 살펴보면 흥미로운점이 있음.
- Promise.race의 반환 타입은 입력 타입들의 유니온. 이번 경우는 Promise<response | never>가 됨.
- 그러나 never(공집합)와의 유니온은 아무런 효과가 없으므로 결과가 Promise<Response>로 간단해짐.
- 프로미스를 사용하면 타입스크립트의 모든 타입 추론이 제대로 동작하게 됨.

- 가끔 프로미스를 직접 생성해야할 때, 특히 setTimeout과 같은 콜백 API를 래핑할 경우가 있음.
- 그러나 선택의 여지가 있다면 일반적으로는 프로미스를 생성하기 보다는 async/await를 사용 권장

async/await를 사용해야하는 이유

- 일반적으로 더 간결하고 직관적인 코드가 됨.
- async 함수는 항상 프로미스를 반환하도록 강제함.

```tsx
//function getNumber():Promise<number>
async function getNumber() {
  return 42;
}
```

e.g : async 화살표 함수를 만들 수 있음.

```tsx
const getNumber = async () => 42; //타입이 () => Promise<number>
```

e.g : 프로미스를 직접 생성

```tsx
const getNumber = () => Promise.resolve(42); //타입이 () => Promise<number>;
```

- 즉시 사용가능한 값에도 프로미스를 반환하는게 이상하게 보일 수 있지만 실제로 비동기 함수로 통일하도록 강제하는 데 도움이 됨.
- 함수는 항상 동기 또는 비동기로 실행되어야 하며, 절대 혼용해서는 안됨.

e.g : 안티 패턴

```tsx
const _cache: { [url: string]: string } = {};
function fetchWithCache(url: string, callback: (text: string) => void) {
  if (url in _cache) {
    callback(_cache[url]);
  } else {
    fetchURL(url, (text) => {
      _cache[url] = text;
      callback(text);
    });
  }
}
```

- 코드가 최적화된 것 처럼 보일지 몰라도, 캐시된 경우 콜백 함수가 동기로 호출되므로 fetchWithCache 함수는 사용하기 매우 어려워짐

```tsx
let requestStatus: "loading" | "success" | "error";
function getUser(userId: string) {
  fetchWithCache("/user/${userId}", (profile) => {
    requestStatus = "success";
  });
  requestStatus = "loading";
}
```

- getUser를 호출한 후에 requestStatus의 값은 온전히 profile이 캐시되었는지 여부에 따라 달림.
- 캐시되어있지 않다면, requestStatus는 조만간 ‘success’가 됨.
- 캐시되어있다면 ‘success’가 되고 나서 바로 ‘loading’으로 다시 되돌아가버림.

e.g:async를 두함수에 사용하여 일관적인 동작을 강제

```tsx
const _cache: { [url: string]: string } = {};
async function fetchWithCache(url: string) {
  if (url in _cache) {
    callback(_cache[url]);
  }

  const response = await fetch(url);
  const text = await response.text();
  _cache[url] = text;
  return text;
}

let requestStatus: "loading" | "success" | "error";
async function getUser(userId: string) {
  requestStatus = "loading";
  const profile = await fetchWithCache("/user/${userId}");
  requestStatus = "success";
}
```

- 이제 requestStatus가 ‘success’로 끝나는것이 명백해짐.
- 콜백이나 프로미스를 사용하면 실수로 반(half)동기 코드를 작성할 수 있지만, async를 사용하면 항상 비동기 코드를 작성하는 셈.
- async 함수에서 프로미스를 반환하면 또 다른 프로미스로 래핑되지 않음.
- 반환 타입은 Promise<Promise<T>>가 아닌 Promise<T>가 됨.
- 타입스크립트를 사용하면 타입 정보가 명확히 드러나기 때문에 비동기 코드의 개념을 잡는 데 도움이 됨.

```tsx
//function getJSON(url:string):Promise<any>
async function getJSON(url: string) {
  const response = await fetch(url);
  const jsonPromise = response.json(); //타입이 Promise<any>
  return jsonPromise;
}
```

## 요약

- 콜백보다는 프로미스를 사용하는게 코드 작성과 타입 추론 면에서 유리함.
- 가능하면 프로미스를 생성하기 보다는 async와 await를 사용하는것이 좋음. 간결하고 직관적인 코드를 작정할 수 있고 모든 종류의 오류를 제거할 수있음.
- 어떤 함수가 프로미스를 반환한다면 async로 선언하는것이 좋음

## 26. 타입 추론에 문맥이 어떻게 사용되는지 이해하기

- 타입스크립트는 타입을 추론할 때 단순히 값만 고려하지 않음.
- 값이 존재하는 곳의 문맥까지도 살핌.
- 그런데 문맥을 고려해 타입을 추론하면 가끔 이상한 결과가 나옴.
- 이때 타입 추론에 문맥이 어떻게 사용되는지 이해하고 있다면 제대로 대처할 수 있음

- 자바스크립트는 코드의 동작과 실행 순서를 바꾸지 않으면서 표현식을 상수로 분리 가능

```tsx
//아래 두 코드는 동일

// 인라인 형태
setLanguage("JavaScript");

// 참조 형태
let language = "JavaScript";
setLanguage(language);
```

e.g : 타입스크립트에서는 다음 리팩터링이 여전히 동작함.

```tsx
function setLanguage(language: string) {
  /* ... */
}

setLanguage("JavaScript"); //정상

let language = "JavaScript";
setLanguage(language); //정상
```

e.g: 문자열 타입을 더 특정하여 문자열 리터럴 타입의 유니온으로 바꾼다고 가정

```tsx
type Language = "JavaScript" | "TypeScript" | "Python";
function setLanguage(language: Language) {
  /* ... */
}

setLanguage("JavaScript"); //정상

let language = "JavaScript";
setLanguage(language);
// Argument of type 'string' is not assignable to parameter of type 'Language'.
```

- 인라인(inline) 형태에서 타입 스크립트는 함수 선언을 통해 매개변수가 language 타입이어야하는 걸 알고있음.
- 해당 타입에 문자열 리터럴 ‘JavaScript’는 할당 가능하므로 정상.
- 그러나 이 값을 변수로 분리해 내면, 타입스크립트는 할당 시점에 타입을 추론.
- 이번 경우는 string으로 추론했고, Language타입으로 할당이 불가능하므로 오류가 발생.

e.g : language의 가능한 값을 타입 선언으로 제한하기

```tsx
let language: Language = "JavaScript";
setLanguage(language); //정상
```

- 만약 language의 값에 ‘Typescript’ (원래 대문자 ‘S’ 여야함) 같은 오타가 있다면 오류를 표시해주는 장점도 있음.

e.g : language를 상수로 만드는 것.

```tsx
const language = "JavaScript";
setLanguage(language); //정상
```

- const를 사용하여 타입 체커에게 language는 변경할 수 없다고 알려줌.
- 따라서 타입스크립트는 language에 대해서 더 정확한 타입인 문자열 리터럴 “JavaScript”로 추론가능.
- “JavaScript”는 Language에 할당수 있으므로 타입 체크를 통과. 단, language를 재할당 해야한다면 타입 선언이 필요.

- 해당 과정에서 사용되는 문맥으로부터 값을 분리하였음. 문맥과 값을 분리하면 추후에 근본적인 문제를 발생시킬 수 있음.

**튜플 사용시 주의점**

- 문자열 리터럴 타입과 마찬가지로 튜플 타입에서도 문맥과 값을 분리시 문제가 발생함.

e.g : 이동이 가능한 지도를 보여주는 프로그램을 작성

```tsx
//매개변수는 (latitude, longitude) 쌍.
function panTo(where: [number, number]) {
  /* ... */
}

panTo([10, 20]); //정상

const loc = [10, 20];
panTo(loc);
//Error - Argument of type 'number[]' is not assignable to parameter of type '[number, number]'.
//  Target requires 2 element(s) but source may have fewer.
```

- 여기서도 문맥과 값을 분리.
- 첫 번째 경우는 [10, 20]이 튜플 타입 [number, number]에 할당이 가능함.
- 두 번째 경우는 타입스크립트가 loc의 타입을 number[]로 추론. (즉 길이를 알 수 없는 숫자의 배열)
- 많은 배열이 이와 맞지 않는 수의 요소를 가지므로 튜플 타입에 할당 할 수 없음.

e.g : 타입 스크립트가 의도를 정확히 파악할 수 있도록 타입 선언을 제공하는 방법 시도.

```tsx
const loc: [number, number] = [10, 20];
panTo(loc);
정상;
```

e.g : 상수 문맥을 제공 하는 방법 시도.

- const는 단지 값이 가리키는 참조가 변하지 않는 얕은(shallow)상수.
- 반면 as const는 그 값이 내부까지 (deeply)상수라는 사실을 타입스크립트에게 알려줌

```tsx
const loc = [10, 20] as const;
panTo(loc);
// Error - Argument of type 'readonly [10, 20]' is not assignable to parameter of type '[number, number]'.
//  The type 'readonly [10, 20]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.
```

- loc의 타입은 이제 number[]가 아니라 readonly[10, 20]으로 추론됨.
- 그러나 이 추론은 너무 과하게 정확함.
- panTo의 타입 시그니처는 where의 내용이 불변이라고 보장하지 않음.
- 즉 loc의 매개변수가 readonly 타입이므로 동작하지 않음.

e.g: panTo함수에 readonly구문을 추가

```tsx
function panTo(where: readonly [number, number]) {
  /* ... */
}
const loc = [10, 20] as const;
panTo(loc); //정상
```

- 타입 시그니처를 수정할 수 없는 경우라면 타입 구문을 사용해야함.

as const의 단점

- 문맥 손실과 관련된 문제를 깔끔하게 해결할 수 있지만, 한가지 단점을 가지고 있음.
- 만약 타입 정의에 실수가 있다면(예를 들어, 튜플에 세 번째 요소를 추가한다면) 오류는 타입 정의가 아니라 호출되는 곳에서 발생.
- 특히 여러 겹 중첩된 객체에서 오류가 발생한다면 근본적인 원인을 파악하기 여러움.

e.g : as const의 오류 예제

```tsx
const loc = [10, 20, 30] as const; //실제 오류는 여기서 발생.
panTo(loc);
//Error - Argument of type 'readonly [10, 20]' is not assignable to parameter of type '[number, number]'.
//  The type 'readonly [10, 20]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.
```

**객체 사용시 주의점**

- 문맥에서 값을 분리하는 문제는 문자열 리터럴이나 튜플을 포함하는 큰 객체에서 상수를 뽑아낼 때도 발생.

```tsx
type Language = "JavaScript" | "TypeScript" | "Python";
interface GovernedLanguage {
  language: Language;
  organization: string;
}

function complain(language: GovernedLanguage) {
  /* ... */
}

complain({ language: "TypeScript", organization: "Microsoft" }); //정상

const ts = {
  language: "TypeScript",
  organization: "Microsoft",
};

complain(ts);
//Error - Argument of type '{ language: string; organization: string; }' is not assignable to parameter of type 'GovernedLanguage'.
//  Types of property 'language' are incompatible.
//    Type 'string' is not assignable to type 'Language'.
```

- ts 객체에서 language의 타입은 string으로 추론됨.
- 이 문제는 타입 선언을 추가하거나(const ts: GovernedLanguage = …) 상수 단언(as const)를 사용해서 해결.

**콜백 사용시 주의점**

- 콜백을 다른 함수로 전달할 때, 타입스크립트는 매개변수 타입을 추론하기 위해 문맥을 사용

```tsx
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
  fn(Math.random(), Math.random());
}

callWithRandomNumbers((a, b) => {
  a; //타입이 number
  b; //타입이 number
  console.log(a + b);
});
```

- callWithRandom의 타입 선언으로 인해 a와 b의 타입이 number로 추론됨.

```tsx
const fn = (a, b) => {
  //Error
  console.log(a + b);
};
callWithRandomNumber(fn);
```

- 콜백을 상수로 뽑아내면 문맥이 소실되고 noImplicityAny오류가 발생하게됨.
- 이 경우 매개 변수에 타입을 추가하여 해결

```tsx
const fn = (a: number, b: number) => {
  console.log(a + b);
};
callWithRandomNumber(fn);
```

- 또는 가능할 경우 전체 함수 표현식에 타입 선언을 적용하는것.

## 요약

- 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴봐야함.
- 변수를 뽑아서 별도로 선언했을 때 오류가 발생한다면 타입 선언을 추가해야함.
- 변수가 정말로 상수라면 상수 단언(as const)을 사용해야함. 그러나 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생. 그러므로 주의가 필요함.

## 27. 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 파이썬, C, 자바 등에서 볼 수 있는 표준 라이브러리가 자바스크립트에는 포함되어있지 않음.
- 표준 라이브러리의 빈 공백을 대체하고자 여러 라이브러리가 존재하며, 라이브러리의 여러 기법들을 타입스크립트와 조합하면 시너지 효과를 얻을 수 있음.
- 타입 정보가 그대로 유지되면서 타입 흐름(flow)이 계속 전달되도록 하기 때문.
- 반면 직접 루프를 구현한다면 타입 체크에 대한 관리도 직접 해야함.

e.g : CSV 데이터를 파싱한다고 가정.

```tsx
const csvData = "...";
const rawRows = csvData.split("\n");
const headers = rawRows[0].split(",");

const rows = rawRows.slice(1).map((rowStr) => {
  const row = {};
  rowStr.split(",").forEach((val, j) => {
    row[headers[j]] = val;
  });
  return row;
});
```

- 순수 자바스크립트로 절차형(imperative) 프로그래밍 형태로 구현할 수 있음.

e.g: reduce를 사용해서 행 객체를 만듬 (함수형)

```tsx
const rows = rawRows
  .slice(1)
  .map((nowStr) =>
    rowStr
      .split(",")
      .reduce((row, val, i) => ((row[headers[i]] = val), row), {})
  );
```

- 절차형 코드에 비해 세줄을 절약했으나 보는사람에 따라 더 복잡하게 느껴질 수 있음.
- 키와 값 배열로 취합(zipping)해서 객체로 만들어주는 로대시의 zipObject 함수를 이용하면 코드를 더욱 짧게 만들 수 있음.

```tsx
import _ from "lodash";
const rows = rawRows
  .slice(1)
  .map((rowStr) => _.zipObject(headers, rowStr.split(",")));
```

- 코드가 많이 개선되었으나, 자바스크립트에서 프로젝트에 서드파티 라이브러리 종속성을 추가할 때 마다 신중해야함.
- 단 같은 코드를 타입스크립트로 작성하면서 서드파티 라이브러리를 사용하는 것이 더 유리함. 타입 정보를 참고하며 작업할 수 있기 때문에 작업 시간이 훨씬 단축됨.
- 추가사항
  - lodash는 이슈 파산을 한 부분이 있음.
  - https://github.com/lodash/lodash
  - https://news.hada.io/topic?id=10920
  - https://twitter.com/danielcroe/status/1703127430523703432

e.g : 위에서 작성한 CSV파서의 절차형 버전과 함수형 버전 모두 오류 발생.

```tsx
//...
row[headers[j]] = val;
//Error - Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{}'.
//  No index signature with a parameter of type 'string' was found on type '{}'.
//...
```

- 두 버전 모두 {}의 타입으로 {[column:string]: string} 또는 Record<string, string>을 제공하면 오류가 해결.

e.g: 로대시 버전은 별도의 수정 없이도 타입 체커 통과

```tsx
import _ from "lodash";
const rows = rawRows
  .slice(1)
  .map((rowStr) => _.zipObject(headers, rowStr.split(",")));
// 타입이 _.Dictionary<string>[]
```

- Dictionary는 로대시의 타입 별칭
- Dictionary<string>은 {[key:string]: string} 또는 Record<string, string>과 동일
- 타입 구문이 없어도 rows의 타입이 정확하다는 것.
- 데이터의 가공(munging)이 정교해질수록 장점은 더욱 분명해짐

e.g : 모든 NBA 팀의 선수 명단을 가지고 있다고 가정

```tsx
interface BasketballPlayer {
  name: string;
  team: string;
  salary: number;
}
declare const rosters: { [team: string]: BasketballPlayer[] };
```

e.g : 루프를 사용해 단순 (flat)목록을 만들려면 배열에 concat을 사용해야함.

```tsx
let allPlayers = [];
// Error - Variable 'allPlayers' implicitly has type 'any[]' in some locations where its type cannot be determined.

for (const players of Object.values(rosters)) {
  allPlayers = allPlayers.concat(players);
  // Error - Variable 'allPlayers' implicitly has an 'any[]' type.
}
```

e.g : 해당 오류를 고치기 위해 allPlayers에 타입 구문을 추가해야함.

```tsx
let allPlayers: BasketballPlayer[] = [];
for (const players of Object.values(rosters)) {
  allPlayers = allPlayers.concat(players); //정상
}
```

e.g : 더 나은 해법을 위해 [Array.prototype.flat](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/flat)을 사용.

```tsx
const allPlayers = Object.values(rosters).flat();
//정상, 타입이 BasketballPlayer[]
```

- flat 메서드는 다차원 배열을 평탄화(flattern)해줌.
- 타입 시그니처는 T[][] ⇒ T[] 같은 형태.
- 이 버전이 가장 간결하고 타입 구문도 필요가 없음.
- 또한 allPlayers 변수가 향후에 변경되지 않도록 let 대신 const를 사용할 수 있음.

e.g: allPlayers를 가지고 각 팀별로 연봉 순으로 정렬해서 최고 연봉 선수의 명단을 만든다고 가정.

```tsx
//lodash를 사용하지 않는 방법.
const teamToPlayers: { [team: string]: BasketballPlayer[] } = {};
for (const player of allPlayers) {
  const { team } = player;
  teamToPlayers[team] = teamToPlayers[team] || [];
  teamToPlayers[team].push(player);
}

for (const players of Object.values(teamToPlayers)) {
  players.sort((a, b) => b.salary - a.salary);
}

const bestPaid = Object.values(teamToPlayers).map((players) => players[0]);
bestPaid.sort((playerA, playerB) => playerB.salary - playerA.salary);
console.log(bestPaid);
```

결과는 다음과 같음

```tsx
[
	{ team: 'GSW', salary:37457154, name: 'Stephen Curry'}
	{ team: 'HOU', salary:35654150, name: 'Chris Paul'}
	{ team: 'LAL', salary:3564150,  name: 'LeBron James'}
	{ team: 'OKC', salary:3564150,  name: 'Russell Westbrook'}
	{ team: 'DET', salary:3208893, name: 'Blake Griffin'}
]
```

e.g : lodash를 사용해서 동일한 작업을 하는 코드를 구현

```tsx
const bestPaid = _(allPlayers)
  .groupBy((player) => player.team)
  .mapValues((players) => _.maxBy(players, (p) => p.salary)!)
  .values()
  .sortBy((p) => -p.salary)
  .value(); //타입이 BasketballPlayer[]
```

- 길이가 절반으로 감소, 가독성이 향상되었음.
- null 아님 단언문(!)을 딱 한번만 사용
- lodash와 언더스코어의 개념인 ‘체인’을 사용했기 때문에 더 자연스러운 순서로 일련의 연산 작성 가능.

e.g : 체인을 사용않는 경우의 연산 과정

```tsx
_.c(_.b(_.a(v)));
```

- 뒤에서부터 연산이 수행됨

e.g : 체인 사용시 연산자의 등장 순서와 실행 순서가 동일하게 됨.

```tsx
_(v).a().b().c().value();
```

- \_(v)는 값을 래핑(wrap)하고 .vaue()는 언래핑 함.
- 래핑된 값의 타입을 보기 위해 체인의 각 함수 호출을 조사할 수 있고 결과는 항상 정확함.

### 요약

- 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기 보다는 내장된 함수형 기법과 lodash같은 유틸리티 라이브러리 사용 권장… 이게 맞는건지? 의문.
