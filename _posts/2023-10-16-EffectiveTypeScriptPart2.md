---
layout: single
title: "[Typescript] Effective Typescript 2장 - 타입스크립트의 타입시스템"
tag:
  - Effective Typescript
---

이펙티브 타입스크립트 2장 내용을 정리하였습니다.

## 편집기를 사용하여 타입시스템 탐색

- 편집기(VS Code or Visual Studio)에서 타입 스크립트 언어를 적극 활용
- 편집기 사용시 타입시스템, 타입스크립트가 타입을 추론하는 개념을 알 수 있음.
- 타입스크립트가 동작을 어떻게 모델링하는지 알기위해 타입 선언 파일을 찾아보는 방법 숙지.

## 타입이 값들의 집합이라고 생각하기

- 변수에는 다양한 종류의 값을 할당할 수 있음
- 타입스크립트는 오류를 체크하는 순간에는 타입을 가지고 있음.
- 타입을 `할당 가능한 값들의 집합(혹은 범위)` 이라고 생각하면 됨.

e.g

- 42와 37.5는 `number` 타입.
- ‘Canada’ 는 `string` 타입.
- null과 undefined는 `strictNullChecks` 플래그 사용 여부에 따라 number에 해당 될 수도, 아닐 수도 있음.

**공집합 타입 never**

- 타입스크립트의 가장 작은 집합은 `never` 이며 아무 값도 포함하지 않는 공집합.

**유닛(unit) 타입 혹은 리터릴(literal)타입**

- never 다음으로 작은 타입. 한가지 값만 포함하고 있음.

```typescript
type A = "A";
type B = "B";
type Twelve = 12;
```

**두개 혹은 세개로 묶으려면 유니온(union)타입을 사용**

```typescript
type AB = "A" | "B";
type AB12 = "A" | "B" | 12;
```

- 세개 이상의 타입을 묶을때로 동일하게 | 로 이어주면 됨.

e.g

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6a9964a5-a4b3-478c-a26c-202e04bf6b13/ae66a753-db64-4b26-9c4c-db3d7e846810/Untitled.png)

- ‘C’는 유닛 타입으로서, 범위는 단일값 `"C"` 구성. 따라서 AB(”A”와 “B”로 이루어진) 부분집합이 아니므로 오류 발생
- 집합의 관점에서 타입체커의 역할은 하나의 집합이 다른 집합의 부분집합인지 검사하는것.
- declare 설명 : https://www.typescriptlang.org/ko/docs/handbook/declaration-files/by-example.html

e.g

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-16-EffectiveTypescriptPart2/Untitled%201.png?raw=true)

- AB12는 AB의 부분집합이 될 수 없으므로 에러 발생.

e.g

```ts
type Int = 1 | 2 | 3 | 4 | 5; // | ..

interface Identified {
  id: string;
}
```

- 범위가 무한대인 경우 원소를 일일이 추가(첫번째)하는 방법도 있겠으나 원소를 서술(두번째)
- 어떤 객체가 string으로 할당가능한 id 속성을 가지고 있다면 그 객체는 Identified 타입으로 볼 수 있음.
- 구조적 타이핑 규칙들은 어떠한 값이 다른 속성을 가질 수 있음을 의미.
- 함수 호출의 매개변수에서도 다른 속성을 가질 수 있음.
- 특정 상황에서만 추가 속성을 허용하지 않는 잉여 속성 체크(express property checking : [링크](https://dev.to/this-is-learning/understanding-excess-property-checking-in-typescript-ook))만 생각하다보면 간과하기 쉬움

### & 연산자 - 인터섹션(intersection)

e.g

```typescript
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan;
```

- `&` 연산자는 두 타입의 인터섹션(intersection, 교집합)을 계산함.
- 겉으로 보았을 때, Person과 Lifespan 인터페이스는 공통 속성이 없기 때문에 PersonSpan 타입을 공집합(never 타입)으로 예상하기 쉬움.
- 그러나 타입 연산자는 인터페이스의 속성이 아닌 값의 집합(타입의 범위)에 적용.

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-16-EffectiveTypescriptPart2/Untitled%202.png?raw=true)

- 그래서 Person과 Lifespan을 둘다 가지는 값은 인터섹션 타입에 속하게됨.
- 인터섹션 타입의 값은 각 타입 내의 모든 속성을 포함하는 것이 일반적.

### **| 연산자 - 유니온(Union)**

e.g

```ts
type AB = "A" | "B";
type AB12 = "A" | "B" | 12;
```

- 값들의 합집합을 뜻함.
- 세개 이상의 타입을 묶을때도 `|` 로 이어줄 수 있음.

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-16-EffectiveTypescriptPart2/Untitled%203.png?raw=true)

- 두 인터페이스 Person 과 Lifespan의 유니온은 `never` .

e.g

- 인터섹션과 유니온 연산에 대해서는 아래의 수식이 성립

```ts
keyof (A & B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

_여기서 keyof 는 오브젝트(타입, 인터페이스 포함)의 키값을 추출하여 union형태로 반환하는 연산자_

e.g

- 일반적으로 위에서 정의한 `PersonSpan` 을 정의할때는 `extends` 사용

```ts
interface Person {
  name: string;
}
interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

- 타입이 집합이라는 관점하에 `extends`의 의미는 “~에 할당가능한 과 비슷하게” ,” ~의 부분집합이라는 의미로 받아들일 수 있음.
- `PersonSapn` 타입의 모든 값은 문자열 name 속성을 가져야함.
- 그리고 birth 속성을 가져야 제대로 된 부분 집합이 됨.

e.g

```ts
interface Vector1D {
  x: number;
}
interface Vector2D extends Vector1D {
  y: number;
}
interface Vector3D extends Vector2D {
  z: number;
}
```

- 어떤 집합이 다른 집합의 부분 집합일 때 ‘서브 집합’ 이라고 칭함.
- Vector3D는 Vector2D의 서브타입.
- Vector2D는 Vector1D의 서브타입.

e.g

extends키워드 없이 동일하게 인터페이스를 작성한다면 아래와 같음

```ts
interface Vector1D {
  x: number;
}
interface Vector2D {
  x: number;
  y: number;
}
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
```

- 집합은 바뀌지 않음.
- 두 스타일 모두 객체 타입에 대해서 잘 동작하나, 리터럴 타입과 유니온 타입을 생각해 본다면, 집합 스타일이 더 직관적.
- extends 키워드는 제너릭 타입에서 한정자로도 쓰임 (~의 부분집합의 의미)

```ts
function getKey<K extends string>(val: any, key: K) {
  //...
}
```

```ts
interface Point {
  x: number;
  y: number;
}

type PointKeys = keyof Point; //타입은 "x" | "y"

function sortBy<K extends keyof T, T>(vals: T[], key: K): T[] {
  //...
}

const pts: Point[] = [
  { x: 1, y: 1 },
  { x: 2, y: 0 },
];
sortBy(pts, "x"); //정상, 'x'는 'x' | 'y'를 상속 (즉 keyof T)
sortBy(pts, "y"); //정상, 'y'는 'x' | 'y' 를 상속
sortBy(pts, Math.random() < 0.5 ? "x" : "y"); //정상, 'x' | 'y' 는 'x' | 'y' 를 상속
sortBy(pts, "z"); //error : "z" 형식의 인수는 'x' | 'y' 형식의 매개변수에 할당될 수 없음.
```

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-16-EffectiveTypescriptPart2/Untitled%204.png?raw=true)

- 타입들이 엄격한 상속 관계가 아닐 때는 집합 스타일이 더 바람직함.
- string | Number와 string | Date 의 인터섹션 (`&`) 은 공집합이 아니며 (이때는 string) 서로의 부분 집합도 아님.
- 타입들이 엄격한 상속 관계가 아니더라도 범위에 대한 관계는 명확함 (그림참조)

e.g

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-16-EffectiveTypescriptPart2/Untitled%205.png?raw=true)

- 타입이 집합이라는 관점은 위의 예를 통해 명확히 알 수 있음.
- 위 코드에서 숫자 배열을 숫자들의 쌍(pair)라고는 할 수 없음. 빈 리스트와 [1]이 그 반례
- number[]는 [number, number]의 부분 집합이 아니므로 할당할 수 없음(반대로 할당시 동작함)

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-16-EffectiveTypescriptPart2/Untitled%206.png?raw=true)

- 동일한 이유로 위의 코드는 동작하지 않음.
- 타입이 값의 집합이라는 의미는 동일한 값의 집합을 가지는 두 타입은 같다는 의미.
- 만약, 두 타입이 의미적으로 다르고 우연히 같은 범위를 가진다 하여도 같은 타입을 두번 정의할 필요는 없음.

e.g

```tsx
// 타입은 Date
// string | Date 와 string | number의 인터섹션 타입중 string을 string | Date 에서 제외
// 그러므로 타입은 Date
type T = Exclude<string | Date, string | number>;

//타입은 여전히 number
type NonZeroNums = Exclude<number, 0>;
```

- 타입 스크립트 타입이 되지 못하는 값의 집합들이 있다는것을 기억해야함.
- 예제처럼 Exclude를 사용해서 일부 타입을 제외할 수는 있으나, 그 결과가 적절한 타입스크립트 타입일 때만 유효함

| 타입 스크립트 용어           | 집합 용어                                        |     |
| ---------------------------- | ------------------------------------------------ | --- |
| never                        | Ø(공집합)                                        |
| 리터럴 타입                  | 원소가 1개인 집합                                |
| 값이 T에 할당 가능           | 값 ∈ T (값이 T의 원소)                           |
| T1이 T2에 할당가능           | T1 ⊆ T2 (T1이 T2의 부분 집합)                    |
| T1이 T2를 상속               | T1 ⊆ T2 (T1이 T2의 부분 집합)                    |
| T1                           | T2 (T1과 T2의 유니온) T1 ∪ T2 (T1과 T2의 합집합) |
| T1 & T2 (T1과 T2의 인터섹션) | T1 ∩ T2 (T1과 T2의 교집합)                       |
| unknown                      | 전체(universal) 집합                             |

요약

- 타입을 값의 집합으로 생각하면 이해하기 쉬움(타입의 범위)
- 이 집합은 유한(boolean 또는 리터럴)하거나 무한(number 또는 string)함.
- 타입스크립트는 엄격한 상속관계가 아니라 겹쳐지는 집합(밴 다이어 그램)으로 표현.
- 두 타입은 서로 서브타입이 아니면서도 겹쳐질 수 있음.
- 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있음.
- 타입 연산은 집합의 범위에 적용. A와 B의 인터섹션(&) 은 A와 B의 범위의 인터섹션.
- 객체타입에서는 A & B인 값이 A와 B의 속성을 모두 가짐을 의미함.
- ‘A는 B를 상속’, ‘A는 B에 할당 가능’, ‘A는 B의 서브타입’ 은 ‘A는 B의 부분 집합’ 과 같은 의미.

## 타입 공간과 값을 공간의 심벌 구분하기.

e.g

```tsx
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });
```

- interface Cylinder에서 Cylinder는 타입으로 쓰임.
- const Cylinder에서 Cylinder와 이름은 같으나 값으로 쓰이며, 서로 아무 관련도 없음.
- 상황에 따라 Cylinder는 타입으로 쓰이거나, 값으로 쓰일 수 있음.
- 이런 부분이 가끔 오류를 야기함.

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-16-EffectiveTypescriptPart2/Untitled%207.png?raw=true)

- instanceof를 이용하여 shape가 Cylinder 타입인지 체크를 시도.
- 그러나 instanceof는 런타임 연산자이며 값에 대한 연산만 수행.
- 따라서 instanceof Cylinder는 타입이 아니라 함수를 참조.

e.g

```tsx
type T1 = "string literal";
type T2 = 123;
const v1 = "string literal";
const v2 = 123;
```

- 일반적으로 type이나 interface 다음에 나오는 심벌은 타입.
- const, let 다음에 나오는 심벌은 값.

e.g

```tsx
//type T1 = 'string literal';
//type T2 = 123;
//const v1 = 'string literal';
//const v2 = 123;

//타입 선언은 컴파일 후 사라진다.
"use strict";
const v1 = "string literal";
const v2 = 123;
```

- 컴파일 후 심벌이 사라진다면 타입에 해당됨.

e.g

```tsx
class Cylinder {
  radius = 1;
  height = 1;
}

function caculatorVolume(shape: unknow) {
  if (shape instanceof Cylinder) {
    shape; //정상. 타입은 Cylider
    shape.radius; //정상 타입은 number
  }
}
```

- class와 enum은 상황에 따라 타입과 값 두가지 모두 가능한 예약어
- 클래스가 타입으로 쓰일 때는 형태(속성과 메서드)가 사용됨.
- 반면 값으로 쓰일때는 생성자가 사용됨.

```tsx
interface Person {
  first: string;
  last: string;
}

const p: Person = { first: "Jane", last: "Jacobs" };

function email(p: Person, subject: string, body: string): Response {}

type T1 = typeof p; //타입은 Person
type T2 = typeof email; //타입은 (p: Person, subject:string, body:string):Response

const v1 = typeof p; // 값은 "object"
const v2 = typeof email; //값은 "function"
```

- 타입의 관점에서 typeof는 값을 읽어서 타입스크립트 타입을 반환.
- 타입의 공간의 typeof는 보다 큰 타입의 일부분으로 사용할 수 있음.
- 혹은 type 구문으로 이름을 붙이는 용도로 사용할 수 있음.

javascript의 typeof

https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/typeof

- 값의 관점에서 typeof는 자바스크립트 런타임의 typeof연산자가 됨.
- 값 공간의 typeof는 대상 심벌의 런타임 타입을 가리키는 문자열을 반환.
- 타입스크립트와는 다름.
- 자바스크립트의 런타임 타입 시스템은 타입스크립트의 정적 타입시스템보다 훨씬 간단하게 동작.

타입스크립트 타입의 종류에 비해 자바스크립트는 지금까지 단 7개가 존재(책에서는 6개지만 현재 js스펙이 올라가면서 7개로 변경됨) (string, number, boolean, undefined, object, function, symbol)

- 자바스크립트의 기본 타입
- https://developer.mozilla.org/ko/docs/Web/JavaScript/Data_structures

- 타입스크립트에서의 typeof 설명

- class키워드는 값과 타입 두가지로 모두 사용됨.
- 따라서 클래에 대한 typeof는 상황에 따라 다르게 동작함.

```ts
const v = typeof Cylinder; //값이 "function"
type T = typeof Cylinder; //타입이 typeof Cylinder
```

- 클래스는 자바스크립트에서 실제 함수로 구현되므로 첫번째 줄의 값은 “function”이 됨.
- Cylinder가 인스턴스의 타입이 아니라는 점에 주목.

```ts
declare let fn: T;
const c = new fn(); //타입이 Cylinder
```

- 실제로는 new 키워드를 사용할 때 볼 수 있는 생성자 함수.

```ts
type C = InstanceType<typeof Cylinder>; //타입이 Cylinder
```

- InstanceType 제너릭을 사용해 생성자 타입과 인스턴스 타입을 전환가능.

```ts
const first: Person["first"] = p["first"]; //또는 p.first
```

- 속성 접근자인 []는 타입으로 쓰일 때에도 동일하게 동작함.
- 그러나 obj[’field’] 와 obj.field는 값이 동일하더라도 타입은 다를 수 있음.
- 따라서 타입의 속성을 얻을 때에는 반드시 첫 번째 방법(obj[’field’])를 사용해야함.
- Person[’first’]는 여기서 타입 맥락으로 사용되었으므로 타입.
- 인덱스 위치에는 유니온 타입과 기본형 타입을 포함한 어떤 타입이든 사용할 수 있음.

```ts
type PersonEl = Person["first" | "last"]; //타입은 string
type Tuple = [string, number, Date];
type TupleEl = Tuple[number]; // 타입 string | number | Date
```

속성 접근자에 대한 정리

**this**

- 값으로 쓰이는 this는 자바스크립트의 this의 키워드.
- 타입으로 쓰이는 this는 일명 다형성(polymorphic) this라고 불리는 this의 타입스크립트 타입.
- 서브클래스의 메서드 체인을 구현할 때 유용.

**& |**

- 값에서 & 와 | 는 AND 와 OR 비트연산자로 사용
- 타입에서는 인터섹션과 유니온

const

- const는 새 변수를 선언.
- as const는 리터럴 또는 리터럴 표현식의 추론된 타입을 변경.

extends

- extends는 서브클래스(class A extends B)또는 서브타입(interface A extends B) 또는 제너릭 한정자(Generic<T extends number>)를 정의.

in

- 루프(for (key in object))또는 매핑된(mapped)타입에 등장.

타입 공간과 값 공간을 혼동하여 발생할 수 있는 케이스

```tsx
function email(options: { person: Person; subject: string; body: string }) {
  // ...
}
```

- 자바스크립트에서 객체 내 각 속성을 로컬 변수로 만들어주는 구조 분해 할당을 사용하여 위의 코드를 아래와 같이 변경

```tsx
function email ({person, subject, body}) //in javascript

//타입 스크립트에서는 아래 코드에 에러가 발생
function email({
	person:Person,
	subject:string,
	body: string}

}){
	//...
}
```

- 값의 관점에서 Person과 string이 해석되었으므로 에러가 발생하게됨.
- Person이라는 변수명과 string이라는 이름을 가지는 두개의 변수를 생성하려고 시도한것.
- 타입과 값을 구분하여 해결가능

```tsx
function email(
{person, subject, body} : {person: Person, subject:string, body:string} {
	//...
}
```

- 코드가 장황ㅎ하여도 매개변수에 명명된 타입을 사용하거나 문맥에서 추론하도록 잘 동작.

## 타입 단언보다는 타입 선언을 사용하기

```tsx
interface Person {
  name: string;
}

const alice: Person = { name: "Alice" }; //타입은 Person
const bob = { name: "Bob" } as Person; //타입은 Person
```

- 이 두가지 방법은 결과가 같아보이나 전혀 그렇지 않음.
- 첫번째 alice:Person은 변수에 ‘타입 선언’을 붙여서 그 값이 선언된 타입을 명시
- 두번째 as Person은 ‘타입 단언’ 을 수행. 타입스크립트가 이미 추론한 타입이 있더라도 Person 타입으로 간주함.
- 타입 단언보다 타입 선언을 사용하는게 더 나음.

```tsx
const alice: Person = {};
//~~~ 'Person' 유형에 필요한 'name' 속성이 '{}' 유형에 없습니다.
const bob = {} as Person; //오류 없음.
```

- 타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사함.
- 이전의 예제는 그러지 못하였으므로 (alice 케이스) 타입스크립트가 오류를 표시.
- 타입 단언(bob 케이스)은 강제로 타입을 지정하였으므로 타입 체커에게 오류를 무시하라고 하는것.

```tsx
const alice: Person = {
  name: "Alice",
  occupation: "Typescript developer",
};

const bob = {
  name: "bob",
  occupation: "Javascript developer",
} as Person;
```

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-10-16-EffectiveTypescriptPart2/Untitled%208.png?raw=true)

- 타입 선언문에는 잉여 속성 체크가 동작했으나, 단언문에는 적용되지 않음.
- 타입 단언이 꼭 필요한 경우가 아니라면 안전성 체크도 되는 타입 선언을 사용 권장

화살표 함수의 타입 선언은 추론된 타입이 모호할 때가 있음.

```tsx
const people = ["alice", "bob", "jan"].map((name) => ({ name }));
//Person[]을 원했으나 결과는 {name:string;}[]...

const people = ["alice", "bob", "jan"].map((name) => ({ name } as Person));
//타입은 Person[] 이겠으나 런타임에 문제발생
```

- 타입 단언을 사용하여 앞선 예제들처럼 런타임에 문제가 발생

```tsx
const people = ["alice", "bob", "jan"].map((name) => ({} as Person));
//오류 없음
```

개선 버전 1

```tsx
const people = ["alice", "bob", "jan"].map((name) => {
  const person: Person = { name };
  return person;
}); //타입은 Person[]
```

개선 버전 2

```tsx
const people = ["alice", "bob", "jan"].map((name): Person => ({ name }));
//타입은 Person[]
```

- (name):Person은 name의 타입이 없고, 반환 타입이 Person임을 명시.
- 그러나 (name: Person)은 name의 타입이 Person임을 명시하고 반환 타입이 없으므로 오류가 발생

개선 버전 3

```tsx
const people: Person[] = ["alice", "bob", "jan"].map(
  (name): Person => ({ name })
);
```

- 최종적으로 원하는 타입을 명시
- 타입스크립트가 할당문의 유효성을 검사하게함
- 함수 호출 체이닝이 연속되는 곳에서는 체이닝 시작에서부터 명명된 타입을 가져야함. 그래야 오류 표기가 정확해짐

타입 단언이 반드시 필요한 경우

```ts
document.querySelector("#myButton").addEventListener("click", (e) => {
  e.currentTarget; //타입은 EventTarget
  const button = e.currentTarget as HTMLButtonElement;
  button; //타입은 HTMLButtonElement;
});
```

- 타입스크립트는 DOM에 접근할 수 없으므로 #myButton이 버튼 엘리먼트인지 알지 못함.
- 이벤트의 currentTarget이 같은 버튼이여야하는것도 알지 못함.
- 타입스크립트가 알지 못하는 정보를 가지고 있으므로 타입 단언문을 사용하는것이 타당함.

타입 단정문 `!` - null이 아님을 단언하는 경우.

```tsx
const elNull = document.getElementById("foo"); //타입은 HTMLElement | null
const el = document.getElementBy("foo")!; //타입은 HTMLElement
```

- 변수의 접두사로 쓰인 !는 boolean의 부정문
- 그러나 접미사로 쓰인 !는 그 값이 null이 아니라는 단언문으로 해석됨.
- !를 일반적인 단언문처럼 생각해야하며 컴파일중에 제거됨.
- 타입 체커는 알기 어려우나, 그 값이 null이 아님을 확신할 수 있을 때 사용해야함.
- 그렇지 않은 경우 반드시 null인 경우를 체크하는 조건문을 사용해야함.

타입 단언문의 한계

```tsx
interface Person {
  name: string;
}
const body = document.body;
const el = body as Person;
//에러 발생
//Conversion of type 'HTMLElement' to type 'Person' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first.
//Property 'name' is missing in type 'HTMLElement' but required in type 'Person'.
```

- 타입 단언문으로 임의의 타입 간에 변환을 할 수 는 없음.
- A가 B의 부분집합인 경우에 타입 단언문을 사용해 변환할 수 있음.
- Person과 HTMLElement는 서로의 서브타입이 아니므로 변환이 불가능.
- 이 에러를 해결하고 싶다면 unknown을 활용해야함.

```tsx
const el = document.body as unknown as Person; //정상
```

- 모든 타입은 unknown의 서브타입이므로 unknown이 포함된 단언문은 항상 동작.
- unknown을 사용한 이상 적어도 개발자에게 위험한 동작을 하고 있음을 알림.

요약

- 타입 단언(as Type)보다 타입 선언(:Type)을 사용.
- 화살표 함수의 반환 타입을 명시하는 방법을 터득해야함.
- 타입스크립트보다 타입 정보를 더 잘 알고있는 상황에서는 타입 단언문 혹은 null 아님 단정문(!) 사용.

## 객체 래퍼 타입 피하기

- 자바스크립트 기본 타입들은 불변이며 메서드를 가지지 않는다는 점에서 객체와 구분됨
- 그러나 기본형인 string인 경우 메서드를 가지고 있는것 처럼 보임.

```tsx
"primitive".charAt(3);
//"m":
```

- 사실 charAt은 string의 메서드가 아니며 string을 사용할 때 자바스크립트 내부에서 변환 과정이 일어남.
- string 기본형에는 메서드가 없고, 메서드를 가지는 String ‘객체’ 타입이 정의되어있음.
- 자바스크립트는 기본형과 객체 타입을 서로 자유롭게 변환.
- string 기본형에 charAt 같은 메서드를 사용할 때, 자바스크립트는 기본형을 String 객체로 래핑(wrap)하고, 메서드를 호출하고, 마지막에 래핑한 객체를 버림.

String.prototype을 몽키 패치하여 위의 설명을 관찰가능.

```tsx
const originalCharAt = String.prototype.charAt;
String.prototype.charAt = function (pos) {
  console.log(this, typeof this, pos);
  return originalCharAt.call(this, pos);
};
console.log("primitive".charAt(3));

//출력
//String {'primitive'} 'object' 3
//m
```

- 메서드 내의 this는 string기본형이 아닌 String객체 래퍼.
- String객체를 직접 생성 할 수 있으며, string 기본형처럼 동작.
- 그러나 string기본형과 String객체 래퍼가 항상 동일하게 동작하는것은 아님.

```tsx
"hello" === new String("hello");
false;
new String("hello") === new String("hello");
false;
```

- String객체는 오직 자기 자신만 동일하게 취급.

```tsx
let x = "hello";
x.language = "English";
("English");
x.language;
undefined;
```

- 객체 래퍼 타입의 자동 변환은 종종 당황스러운 동작을 보일 때가 있음.
- 어떤 속성을 기본형에 할당하면 해당 속성은 사라짐.
- 실제로는 x가 String객체로 변환 된 후 language속성이 추가되었고, language속성이 추가된 객체는 버려진것.

타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링

- string / String
- number / Number
- boolean / Boolean
- symbol / Symbol
- bigint / BigInt

- 그러나 string을 사용할 때는 특히 유의.
- string을 String이라고 잘못 타이핑 하기 쉬우며, 실수를 하더라도 처음에는 잘 동작하는것으로 보임.

```ts
function getStringLen(foo: String) {
  return foo.length;
}

getStringLen("hello");
getStringLen(new String("hello"));
//둘 다 정상 동작
```

- 그러나 string을 매개변수로 받는 메서드에 String 객체를 전달하는 순간 문제 발생

```tsx
function isGreeting(phrase: String) {
  return ["hello", "good day"].includes(phrase);
}

/*
 Argument of type 'String' is not assignable to parameter of type 'string'.
 'string' is a primitive, but 'String' is a wrapper object. Prefer using 'string' when possible.
*/
```

- string은 String에 할당할 수 있지만 String은 string에 할당할 수 없음.
- 오류 메세지대로 string타입을 사용해야함.
- 대부분의 라이브러리와 마찬가지로 타입스크립트가 제공하는 타입 선언은 전부 기본형 타입으로 되어있음.

```ts
const s: String = "primitive";
const n: Number = 12;
const b: Boolean = true;
```

- 래퍼 객체는 타입 구문의 첫 글자를 대문자로 표기하는 방법으로도 사용할 수 있음.
- 당연히 런타임의 값은 객체가 아니고 기본형.
- 그러나 기본형 타입은 객체 래퍼에 할당할 수 있기 때문에 타입스크립트는 기본형 타입을 객체 래퍼에 할당하는 선언을 허용.
- 단, 기본형 타입을 객체 래퍼에 할당하는 구문은 오해하기 쉽고, 굳이 그럴 필요도 없으므로 기본형 타입을 사용하는게 나음.

new 없이 BigInt와 Symbol를 호출하는 경우는 기본형을 생성하기 때문에 사용해도 문제없음.

```ts
typeof BigInt(1234);
("bigint");
typeof Symbol("sym");
("symbol");
```

- BigInt와 Symbol ‘값’ 이지만, 타입스크립트 타입은 아님.
- 위 코드의 결과값은 bigint와 symbol타입의 값이 됨.

요약

- 기본형 값에 메서드를 제공하기 위해 객체 래퍼타입이 어떻게 쓰이는지 이해해야함.
- 객체 래퍼 타입을 직접 사용하거나 인스턴스를 생성하는것은 피해야함.
- 타입 스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용해야함.
- String대신 string, Number대신 number, Boolean 대신 boolean, Symbol 대신 symbol, BigInt 대신 bigint를 사용해야함.

## 잉여 속성 체크의 한계 인지하기

타입이 명시된 변수에 객체 리터럴을 할당할 때 타입스크립트는 해당 타입의 속성이 있는지, 그리고 그 외 속성은 없는지 확인.

예제 1

```tsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
/*
Type '{ numDoors: number; ceilingHeightFt: number; elephant: string; }' is not assignable to type 'Room'.
  Object literal may only specify known properties, and 'elephant' does not exist in type 'Room'.
*/
```

- Room 타입에 생뚱맞은 elephant속성이 있는것이 어색하나, 구조적 타이핑 관점으로 생각했을때 오류가 발생하지 않아야함.
- 임시 변수를 도입하여 테스트 해보면 알 수 있음.

예제 2

```tsx
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
const r: Room = obj; //정상
```

- obj의 타입은 {numDoors:number; ceilingHeightFt:number; elephant:string} 으로 추론됨.
- obj 타입은 Room 타입의 부분 집합을 포함하므로, Rom에 할당가능하며, 타입 체커도 통과함.

두 예제 비교

- 예제 1에서는 구조적 타입 시스템에서 발생할 수 있는 오류를 잡을 수 있도록 ‘잉여 속성 체크’ 과정이 수행됨.
- 그러나 잉여 속성 체크 역시 조건에 따라 동작하지 않는다는 한계가 존재.
- 통상적인 할당가능 검사와 함께 쓰이면 구조적 타이핑이 무엇인지 혼란스러움.
- 잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는것을 인지해야함.

타입스크립트는 의도와 다르게 작성된 코드도 탐지시도함.

```tsx
interface Options {
  title: string;
  darkMode?: boolean;
}
function createWindow(options: Options) {
  if (options.darkMode) {
    // setDarkMode();
  }
}
createWindow({
  title: "Spider Solitaire",
  darkmode: true,
});

/**
Argument of type '{ title: string; darkmode: boolean; }' is not assignable to parameter of type 'Options'.
  Object literal may only specify known properties, but 'darkmode' does not exist in type 'Options'. Did you mean to write 'darkMode'?
*/
```

- 위 코드는 런타임에는 아무 에러가 없음.
- 그러나 타입스크립트에서 알려주는 오류 메시지처럼 의도한 대로 동작하지 않을 가능성이 존재.
- 오류 내용은 darkmode가 아닌 darkMode(대문자M)이어야 함.

```tsx
const o1: Options = document; //정상
const o2: Options = new HTMLAnchorElement(); //정상
```

- Options 타입은 범위가 매우 넓으므로, 순수한 구조적 타입 체커는 이런 종류의 에러를 찾지 못함.
- document와 HTMLAnchorElement의 인스턴스 모두 string 타입인 title속성을 가지고 있어 할당문은 정상처리.

```tsx
const o: Options = { darkmode: true, title: "Ski Free" };
//Options 형식에 darkmode가 없음.
```

- 위 구문은 에러 발생

```tsx
const intermediate = { darkmode: true, title: "Ski Free" };
const o: Options = intermediate; //정상
```

- 타입 구문이 없는 임시변수를 사용하여 테스트
- 첫 번째 라인의 오른쪽은 객체 리터럴이지만, 두 번째 줄의 오른쪽(intermediate)은 객체 리터럴이 아님.
- 따라서 잉여 속성 체크는 적용되지 않고 오류가 사라짐.

```tsx
const o = { darkmode: true, title: "Ski Free" } as Options; //정상
```

- 잉여 속성 체크는 단언문을 사용할때도 적용되지 않음.

```tsx
interface Options {
  darkMode?: boolean;
  [otherOptions: string]: unknown;
}
const o: Options = { darkmode: true }; //정상
```

- 잉여 속성 체크를 원치 않는다면, 인덱스 시그니처를 사용해서 타입스크립트가 추가적인 속성을 예상하도록 할 수 있음.

```tsx
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}
const opts = { logScale: true };
const o: LineChartOptions = opts;
// Type '{ logScale: boolean; }' has no properties in common with type 'LineChartOptions'.
```

- 구조적 관점에서 LineChartOptions 타입은 모든 속성이 선택적이므로 모든 객체를 포함 가능.
- 이런 약한(Weak) 타입에 대해서 타입스크립트는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행.
- 공통 속성 체크는 잉여 속성 체크와 마찬가지로 오타를 잡는데 효과적이며 구조적으로 엄격하지 않음.
- 그러나 잉여 속성 체크와는 달리, 약한 타입과 관련된 할당문 마다 수행됨.
- 임시 변수를 제거하여도 공통 속성 체크는 여전히 동작

**요약**

- 객체 리터럴을 변수에 할당하거나, 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행됨.
- 잉여 속성 체크는 오류를 찾는 효과적인 방법이지만, 타입스크립트 타입 체커가 수행하는 일반적인 구조적 할당 가능성 체크와 역할이 다름
- 할당의 개념을 정확히 알아야 잉여 속성 체크와 일반적인 구조 할당 가능성 체크를 구분할 수 있음.
- 잉여 속성 체크에는 한계가 있음. 임시 변수를 도입하면 잉여 속성 체크를 건너 뛸 수 있음.

## 함수 표현식에 타입 적용하기

```tsx
function rollDice1(sides:number): number {/*...*/} //문장
const rollDice2 = function (sides:number):number {/*...*/} ********//표현식
const rollDice3 = function (sides:number):number => {/*...*/} //표현식********
```

- 자바스크립트 및 타입스크립트는 함수 문장과 표현식을 다르게 인식.
- 타입스크립트에서는 함수 표현식 사용을 권장
- 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용 할 수 있음.

```tsx
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = (sides) => {
  /* ... */
};
```

- 타입스크립트에서 sides를 이미 number로 인식.

```tsx
function add(a: number, b: number) {
  return a + b;
}
function sub(a: number, b: number) {
  return a - b;
}
function mul(a: number, b: number) {
  return a * b;
}
function div(a: number, b: number) {
  return a / b;
}
```

```tsx
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

- 반복되는 함수 시그니처를 하나의 함수 타입으로 통합할 수 있음.
- 라이브러리는 공통 함수 시그니처를 타입으로 제공하기도 함.
- 예를 들어 리액트는 MouseEvent 타입 대신 함수 전체에 적용할 수 있는 MouseEventHandler 타입을 제공.
- 라이브러리를 직접 만들고 있다면, 공통 콜백 함수를 위한 타입 선언을 제공하는게 좋음.

시그니처가 일치하는 다른 함수가 있을 때도 함수 표현식에 타입을 적용해 볼 만함.

```tsx
//http 요청 보냄
const responseP = fetch("/quote/?by=Mark+Twain"); //타입이 Promise<Response>
```

```tsx
async function getQuote() {
  const response = await fetch("/quote?by=Mark+Twain");
  const quote = await response.json();

  return quote;
}
//response
//{
// "quote" : "If you tell ... bla bla",
// "source" : "notebook",
// "date" : "1894"
//}
```

- 위 코드에는 드러나지 않은 버그가 있음.
- `/quote` 가 존재하지 않는 API라면, ‘404 Not Found’ 가 포함된 내용을 응답함.
- 따라서 응답은 JSON 형식이 아닐 수 있음.
- response.json()은 JSON 형식이 아니라는 새로운 오류 메시지를 담아 거절된 (rejected) 프로미스를 반환.
- 호출한 곳에서는 새로운 오류 메시지가 전달되어 실제 오류인 404가 감추어짐.
- 또한 fetch가 실패하면 거절된 프로미스를 응답하지 않는다는 점을 간과하기 쉬움.

상태 체크를 수행해줄 checkedFetch 함수 작성

```tsx
//lib.dom.d.ts에 정의된 fetch의 타입선언
declare function fetch(
  input: RequestInfo,
  init?: RequestInit
): Promise<Response>;
```

checkedFetch 정의

```tsx
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if (!response.ok) {
    //비동기 함수 내에서 거절된 프로미스로 반환
    throw new Error("Request failed: " + response.status);
  }
  return response;
}
```

```tsx
//checkedFetch를 보다 간결하게 표현
const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error("Request failed: " + response.status);
  }
  return response;
};
```

- 간결한 버전에서는 `typeof fetch` 이 적용되어 input과 init의 타입 추론을 할 수 있게 해줌.
- 타입 구문은 checkedFetch의 반환 타입을 보장하며, fetch와 동일함.
- 예를 들어 throw 대신 return을 사용했다면 타입 스크립트는 그 실수를 잡아냄.

```tsx
const checkedFetch: typeof fetch = async (input, init) => {
  //에러
  //Type '(input: RequestInfo | URL, init: RequestInit | undefined) => Promise<Response | Error>' is not assignable to type '(input: RequestInfo | URL, init?: RequestInit | undefined) => Promise<Response>'.
  //Type 'Promise<Response | Error>' is not assignable to type 'Promise<Response>'.
  //Type 'Response | Error' is not assignable to type 'Response'.
  //Type 'Error' is missing the following properties from type 'Response': headers, ok, redirected, status, and 11 more.

  const response = await fetch(input, init);
  if (!response.ok) {
    return new Error("Request failed: " + response.status);
  }
  return response;
};
```

- checkedFetch를 함수 문장으로 작성한 예시에서도 throw가 아니라 return을 사용한 경우 동일한 오류 발생(단 오류 위치는 함수를 호출한 위치에서 발생함)

정리

- 함수의 매개변수나 반환값에 타입을 명시하기 보다는 함수 표현식 전체에 타입 구문을 적용하는것이 좋음.
- 만약 같은 타입 시그니처를 반복적으로 작성한 코드가 있다면 함수 타입을 분리해 내거나 이미 존재하는 타입을 찾아볼것.
- 라이브러리를 직접 만든다면 공통 콜백에 타입을 제공해야함.
- 다른 함수의 시그니처를 참조하려면 typeof fn을 사용하면 됨.

## 타입과 인터페이스의 차이점 알기

타입스크립트에서 명명된 타입(named type)을 정의하는 방법은 두가지가 있음.

```tsx
//type 사용
type TState = {
  name: string;
  capital: string;
};

//interface 사용
interface IState {
  name: string;
  capital: string;
}
```

- 대부분의 경우에는 타입을 사용해도 되고 인터페이스를 사용해도 무관.
- 그러나 타입과 인터페이스의 사이에 존재하는 차이를 분명하게 알고, 같은 상황에서는 동일한 방법으로 명명된 타입을 정의해 일관성을 유지해야함.
- 그러기 위해서는 하나의 타입에 대해 두 가지 방법을 모두 사용해서 정의할 줄 알아야함.
- (인터페이스 명에 대문자 I 로 시작하는 부분은 더이상 사용되지 않는 스타일. 예시를 위해 사용)

**인터페이스 선언과 타입 선언의 비슷한점**

```tsx
const wyoming: TState = {
  name: "Wyoming",
  capital: "Cheyenne",
  population: 500_000,
};

//오류
//Type '{ name: string; capital: string; population: number; }' is not assignable to type 'TState'.
//  Object literal may only specify known properties, and 'population' does not exist in type 'TState'.
```

- 명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없음.
- TState 대신 IState를 사용해도 동일한 오류 발생

```tsx
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}
```

- 인덱스 시그니처는 인터페이스와 타입 모두 사용가능

```tsx
type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}

const toStrT: TFn = (x) => "" + x; //정상
const toStrI: IFn = (x) => "" + x; //정상
```

- 또한 함수 타입도 인터페이스나 타입으로 정의할 수 있음.

```tsx
type TFnWithProperties = {
  (x: number): number;
  props: string;
};

interface IFnWithProperties {
  (x: number): number;
  props: string;
}
```

- 단순한 함수 타입에는 타입 별칭 (alias)가 더 나을 수 있으나, 함수 타입에 추가적인 속성이 있다면 타입이나 인터페이스나 차이가 없음.

```tsx
type TPair<T> = {
  first: T;
  second: T;
};
interface IPair<T> {
  first: T;
  second: T;
}
```

- 자바스크립트에서 함수는 호출 가능한 객체임을 상기하면 이해 가능한 코드.
- 타입 별칭과 인터페이스 모두 제너릭으로 사용가능.

```tsx
interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop = IState & { population: number };
```

- IStateWithPop과 TStateWithPop은 동일.
- 주의 : 인터페이스는 유니온 타입과 같이 복잡한 타입을 확장하지 못함.
- 복잡한 타입을 확장하고 싶다면 타입과 &를 사용해야함.

```tsx
class StateT implements TState {
  name: string = "";
  capital: string = "";
}

class StateI implements IState {
  name: string = "";
  captital: string = "";
}
```

- class를 구현(implements)할 때는, 타입(TState)과 인터페이스(IState) 둘 다 사용할 수 있음.

타입과 인터페이스의 차이점

```tsx
type AorB = "a" | "b";
```

- 유니온 타입 은 있으나 유니온 인터페이스는 없음.

```tsx
type Input = {
  /* ... */
};
type Output = {
  /* ... */
};
interface VariableMap {
  [name: string]: Input | Output;
}
```

- 인터페이스는 타입을 확장할 수 있으나, 유니온은 할 수 없음.
- 그러나 유니온 타입을 확장하는게 필요할 때가 있는데, 위의 코드가 그 예.

```tsx
type NameVariable = (Input | Output) & { name: string };
```

- 혹은 유니온 타입에 name 속성을 붙인 타입을 만들 수도 있음.
- 이 타입은 인터페이스로 표현이 불가능함.
- type 키워드는 일반적으로 interface보다 쓰임새가 많음.
- type 키워드는 유니온이 될 수 있고, 매핑된 타입 또는 조건부 타입 같은 고급 기능에 활용

```tsx
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];
```

- 튜플과 배열 타입도 type 키워드를 이용해 더 간결하게 표현

```tsx
interface Tuple {
  0: number;
  1: number;
  length: 2;
}
const t: Tuple = [10, 20]; //정상
```

- 인터페이스로 튜플과 비슷하게 구현이 가능하나, 튜플에서 사용할 수 있는 concat 같은 메서드를 사용할 수 없음.
- 그러므로 튜플은 type 키워드로 구현하는것이 나음.

```tsx
interface IState {
  name: string;
  capitial: string;
}

interface IState {
  population: number;
}

//IState가 두번 정의됨

const wyoming: IState = {
  name: "Wyoming",
  capital: "Cheyenne",
  population: 500_000,
}; //정상
```

- 인터페이스에는 타입에는 없는 몇가지 기능이 존재.
- 그 중 하나가 보강(argument)이 가능하다는것.
- 위 예제처럼 속성을 확장하는것을 선언 병합(declaration merging)이라고 함
- 선언 병합은 주로 타입 선언 파일에 사용됨.
- 따라서 타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 반드시 인터페이스를 사용해야 하며 표준을 따라야함.
- 타입 선언에는 사용자가 채워야하는 빈틈이 있을 수 있는데, 바로 이 선언 병합이 그러함.

타입 병합에 대한 보다 자세한 내용

- 타입스크립트는 여러 버전의 자바스크립트 표준 라이브러리에서 여러 타입을 모아 병합함.
- 예를 들어 Array 인터페이스는 lib.es5.d.ts에 정의되어있고, 기본적으로는 lib.es5.d.ts에 선언된 인터페이스가 사용됨.
- 그러나 tsconfig.json의 lib 목록에 ES2015를 추가하면 타입스크립트는 lib.es2015.d.ts에 선언된 인터페이스를 병합함.
- 여기에는 es2015에 추가된 또 다른 Array 선언의 find같은 메서드가 포함됨.
- 병합은 선언과 마찬가지로 일반적인 코드에서도 지원되므로 언제 병합이 가능한지 알고 있어야함.
- 타입은 기존 타입에 추가적인 보강이 없는 경우에만 사용해야함.

결론

- 복잡한 타입이라면 → 타입 별칭 사용
- 그러나 타입과 인터페이스 두가지 방법으로 모두 표현할 수 있는 간단한 객체 타입이라면 일관성과 보강의 관점에서 고려할것.
- 일관되게 인터페이스를 사용하는 코드베이스에서 작업하고 있다면 인터페이스를 사용하고, 일관되게 타입을 사용 중이라면 타입을 사용.
- 아직 스타일이 확립되지 않은 프로젝트라면 향후에 보강의 가능성이 있을지 생각할것.
- 어떤 API가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합할 수 있어 유용하기 때문.
- 그러나 프로젝트 내부적으로 사용되는 타입에 선언 병합이 발생하는 것은 잘못된 설계. 따라서 이럴 때는 타입 사용.

요약

- 타입과 인터페이스의 차이점과 비슷한점을 이해해야함.
- 한 타입을 type과 interface 두 가지 문법을 사용해서 작성하는 방법을 터득해야함.
- 프로젝트에서 어떤 문법을 사용할지 결정할 때 한가지 일관된 스타일을 확립하고, 보강 기업이 필요한지 고려.

## 타입 연산과 제너릭 사용으로 반복 줄이기

```jsx
//원기둥의 반지름과 높이, 표면적, 부피를 출력하는 코드
console.log(
  "Cylinder 1 x 1",
  "Surface area:",
  6.283185 * 1 + 6.283185 * 1 * 1,
  "Volume:",
  3.14159 * 1 * 1 * 1
);
console.log(
  "Cylinder 1 x 1",
  "Surface area:",
  6.283185 * 1 + 6.283185 * 2 * 1,
  "Volume:",
  3.14159 * 1 * 2 * 1
);
console.log(
  "Cylinder 1 x 1",
  "Surface area:",
  6.283185 * 2 + 6.283185 * 1 * 1,
  "Volume:",
  3.14159 * 2 * 2 * 1
);
```

- 비슷한 코드가 반복되어 있어 보기 불편함.
- 값과 상수가 반복되어 드러나지 않은 오류도 가지고 있음.

```tsx
const surfaceArea = (r, h) => 2 * Math.PI * r * (r+h);
const volume = (r, h) => Math.PI * r* r * h;
for (const [r,h] of [[1, 1], [1,2], [2,1]){
	console.log(
		`Cylinder ${r} x ${h}`,
		'Surface area: ${surfaceArea(r, h)}`,
		`Volume: ${volume(r, h)}`);
}
```

- 같은 코드를 반복하지 말라는 DRY(dont’t repeat yourself) 원칙
- 소프트웨어 개발자라면 어느 분야에서든 들어 봤을 조언.
- 그런데 반복된 코드를 열심히 제거하며 DRY 원칙을 지켜왔던 개발자라도 타입에 대해서는 간과했을지도 모름

```tsx
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate {
  firstName: string;
  lastName: string;
  birth: Date;
}
```

- 타입 중복은 코드 중복만큼 많은 문제를 발생시킴.
- 여기서 선택적인 필드인 middleName을 Person에 추가한다고 가정. 그러면 Person과 PersonWithBirthDate는 다른 타입이됨.
- 타입에서 중복이 더 흔한 이유는 하나는 공유된 패턴을 제거하는 메커니즘이 기존 코드에서 하던 것과 비교해 덜 익숙하기 때문.
- 헬퍼 함수 중복 제거와 동일한 활동이 타입 시스템에서는 어떤 것에 해당할지 상상이 잘 되지 않음.
- 그러나 타입 간에 매핑하는 방법을 익히면, 타입 정의에서도 DRY의 장점을 적용할 수 있음.
- 반복을 줄이는 가장 간단한 방법은 타입에 이름을 붙이는 것.

```tsx
//다음 예제의 거리 계산 함수에는 타입이 반복적으로 등장
function distance(a: {x:number, y:number), b: {x:number, y:number}) {
	return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y,2));
}
```

```tsx
interface Point2D {
  x: number;
  y: number;
}
function distance(a: Point2D, b: Point2D) {
  /*...*/
}
```

- 코드를 수정하여 타입에 이름을 부여
- 상수를 사용해서 반복을 줄이는 기법을 동일하게 타입 시스템에 적용.
- 중복된 타입을 찾는것은 항상 쉬운 일은 아님. 중복된 타입은 종종 문법에 의해서 가려질 때가 있음.

```tsx
//같은 시그니처를 공유하는 함수
function get(url: string, opts: Options): Promise<Response> {
  /* ... */
}
function post(url: string, opts: Options): Promise<Response> {
  /* ... */
}
```

```tsx
//중복되는 시그니처를 명명된 타입으로 분리
type HTTPFunction = (url: string, opts: Options) => Promise<Response>;
const get: HTTPFunction = (url, opts) => {
  /* ... */
};
```

```tsx
//이전에 언급되었던 Person/PersonWithBirthDate
interface PersonWithBirthDate extends Person {
  birth: Date;
}
```

- 한 인터페이스가 다른 인터페이스를 확장하게 하여 반복을 제거
- 두 인터페이스가 필드의 부분 집합을 공유한다면, 공통 필드만 골라서 기반 클래스로 분리해 낼 수 있음.
- 코드 중복의 경우와 비교하면, 3.141593과 6.283185대신에 PI와 2 \* PI로 작성하는 것 과 비슷한 이치

```tsx
type PersonWithBirthDate = Person & { birth: Date };
```

- 이미 존재하는 타입을 확장하는 경우에, 일반적이지는 않지만 인터섹션 연산자(&)를 사용할 수도 있음
- 이런 기법은 유니온 타입(확장할 수 없는)에 속성을 추가하려고 할 때 특히 유용함.

```tsx
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
```

- 전체 애플리케이션의 상태를 표현하는 State타입과 단지 부분만 표현하는 TopNavState가 있는 경우
- TopNavState를 확장하여 State를 구성하기보다, State의 부분 집합으로 TopNavState를 정의하는 것이 바람직해 보임.
- 이 방법이 전체 앱의 상태를 하나의 인터페이스로 유지할 수 있게 해줌.

```tsx
type TopNavState = {
  userId: State["userId"];
  pageTitle: State["pageTitle"];
  recentFiles: State["recentFiles"];
};
```

- State를 인덱싱하여 속성의 타입에서 중복을 제거할 수 있음.
- State내의 pageTitle의 타입이 바뀌면 TopNavState에도 반영됨.
- 그러나 여전히 반복되는 코드가 존재함.

```tsx
type TopNavState = {
  [k in "userId" | "pageTitle" | "recentFiles"]: State[k];
};
```

- `매핑된 타입` 을 활용하여 좀 더 개선.
- 매핑된 타입은 배열의 필드를 루프 도는 것과 같은 방식.

```tsx
type Pick<T, K> = { [k in K]: T[k] };
```

- 매핑된 타입 패턴은 표준 라이브러리에서도 찾을 수 있으며 `Pick`이라고 함. 링크 : [Typescript Pick](https://www.typescriptlang.org/docs/handbook/utility-types.html#picktype-keys)

```tsx
type TopNavState = Pick<State, "userId" | "pageTitle" | "recentFiles">;
```

- 정의가 완전하지는 않으나 위와 같이 사용할 수 있음.
- 여기서 Pick은 `제너릭` 타입.
- Pick을 사용하는 것은 함수를 호출하는 것 과 마찬가지.
- 함수에서 두개의 매개변수 값을 받아서 결괏값을 반환하는 것 처럼, Pick은 T와 K 두 가지 타입을 받아서 결과 타입을 반환.

```tsx
interface SaveAction {
  type: "save";
  // ...
}

interface LoadAction {
  type: "load";
  // ...
}
type Action = SaveAction | LoadAction;
type ActionType = "save" | "load"; //타입의 반복
```

- 태그된 유니온에서도 다른 형태의 중복이 발생할 수 있음.

```tsx
type ActionType = Action["type"]; //타입은 "save" | "load"
```

- Action 유니온을 인덱싱하면 타입 반복 없이 ActionType을 정의할 수 있음.
- Action 유니온에 타입을 더 추가하면 ActionType은 자동적으로 그 타입을 포함.

```tsx
type ActionRec = Pick<Action, "type">; // {type: "save" | "load"}
```

- ActionType은 Pick을 사용하여 얻게 되는 type 속성을 가지는 인터페이스와는 다름.

```tsx
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
interface OptionsUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}
class UIWidget {
  constructor(int: Options) {
    /* ... */
  }
  update(options: OptionsUpdate) {
    /* ... */
  }
}
```

- 생성하고 난 다음에 업데이트가 되는 클래스를 정의.
- update 메서드 매개변수의 타입은 생성자와 동일한 매개변수이면서, 타입은 대부분 선택적인 필드가 됨.

```tsx
type OptionsUpdate = { [k in keyof Options]?: Options[k] };
```

- 매핑된 타입과 keyof를 사용하면 Options로 부터 OptionsUpdate를 만들 수 있음.

```tsx
type OptionsKeys = keyof Options;
//타입이 "width" | "height" | "color" | "label"
```

- keyof는 타입을 받아서 속성 타입의 유니온을 반환.
- 매핑된 타입([k in keyof Options])은 순회하며 Options내 k 값에 해당하는 속성이 있는지 찾음.
- ?는 각 속성을 선택적으로 만듬.
- 이 패턴 역시 일반적이며 표준 라이브러리의 Partial이라는 이름으로 포함되어있음
- [Partial 링크](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype)

```tsx
class UIWidget {
  constructor(init: Options) {
    /* .. */
  }
  update(options: Partial<Options>) {
    /* ... */
  }
}
```

- Partial을 사용하여 정리

e.g : 값의 형태에 해당하는 타입을 정의하고 싶을 때도 있음.

```tsx
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: "#00FF00",
  label: "VGA",
};

interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
```

- 이런 경우 typeof를 사용.

```tsx
type Options = typeof INIT_OPTIONS;
```

- 자바스크립트의 런타임 연산자 typeof를 사용한 것처럼 보임.
- 그러나 실제로는 타입스크립트 단계에서 연산되며 훨씬 더 정확하게 타입을 표현함.
- 주의 사항 : 값으로부터 타입을 만들어 낼 때는 선언의 순서에 주의해야함.
- 타입 정의를 먼저 하고 값이 그 타입에 할당 가능하다고 선언하는 것이 좋음.
- 그렇게 해야 타입이 더 명확해지고, 예상하기 어려운 타입 변동을 방지할 수 있음.

함수나 메서드의 반환 값에 명명된 타입을 만드는 경우

```tsx
function getUserInfo(userId: string) {
  //...
  return {
    userId,
    name,
    age,
    height,
    weight,
    favoriteColor,
  };
}
//추론된 반환 타입은 {userId: string; name: string; age:number, ...}
```

- 이때는 조건부 타입이 필요함.
- 그러나 앞에서 보았던 것 처럼 표준 라이브러리에 정의된 제너릭 타입인 ReturnType 사용

```tsx
type userInfo = ReturnType<typeof getUserInfo>;
```

- ReturnType은 함수의 ‘값’ 인 getUserInfo가 아니라 함수의 `타입` 인 typeof getUserInfo에 적용되었음.
- typeof와 마찬가지로 이런 기법은 신중하게 사용할것.
- 적용 대상이 값인지 정확히 알고, 구분해서 처리해야함.

```tsx
interface Name {
	first: string;
	last: string;
}
type DancingDuo<T extends Name> = [T, T];

const couple1: DancingDuo<Name> = [
	{first: 'Fred', last: 'Astaire'},
	{first: 'Ginger', last: 'Rogers'}
] //OK

const couple2: DancingDuo<first: string}> = [
	{first:'Sonny'},
	{first:'Cher'}
];

//Type '{ first: string; }' does not satisfy the constraint 'Name'.
//  Property 'last' is missing in type '{ first: string; }' but required in type 'Name'.
```

- 제너릭 타입은 타입을 위한 함수과 같음. 그리고 그 함수는 코드에 대한 DRY 원칙을 지킬 때 유용하게 사용됨.
- 따라서 타입에 대한 DRY원칙의 핵심이 제너릭이라는 것은 당연함.
- 그러나 함수에서 매개변수로 매핑할 수 있는 값을 제한하기 위해 타입 시스템을 사용하는 것 처럼 제네릭 타입에서 매개변수를 제한할 수 있는 방법이 필요함.
- 제너릭 타입에서 매개변수를 제한할 수 있는 방법은 extends를 사용하는것.
- extends를 사용하면 제너릭 매개변수가 특정 타입을 확장한다고 선언할 수 있음. (위 코드 예제 참조)
- {first: string} 은 Name을 확장하지 않기 때문에 오류가 발생.

e.g : 앞에 나온 Pick의 정의를 extends를 활용하여 완성하기

```tsx
type Pick<T, K> = {
  [k in K]: T[k];
};
//Type 'K' is not assignable to type 'string | number | symbol'.
```

- K는 T타입과 무관하고 범위가 너무 넓음.
- K는 인덱스로 사용될 수 있는 `string` | `number` | `symbol` 이 되어야함.
- 실제로는 범위를 더 좁힐 수 있는데 K는 실제로 T의 키의 부분 집합, 즉 keyof T가 되어야함.

```tsx
type Pick<T, K extends keyof T> = {
  [k in K]: T[k];
}; //정상
```

- 타입이 값의 집합이라는 관점에서 생각하면 extends를 확장이 아니라 부분집합이라는걸 이해

요약

- DRY(don’t repeat yourself) 원칙을 타입에도 최대한 적용해야함.
- 타입에 이름을 붙여서 반복을 피해야함. extends를 사용해서 인터페이스 필드의 반복을 피해야함.
- 타입들 간의 매핑을 위해 타입스크립트가 제공한 도구들을 공부할것. (keyof, typeof, 인덱싱, 매핑된 타입이 포함)
- 제네릭 타입은 타입을 위한 함수와 같음. 타입을 반복하는 대신 제네릭 타입을 사용하여 타입들 간에 매핑을 하는것이 좋음.
- 제네릭 타입을 제한하려면 extends를 사용하면 됨.
- 표준 라이브러리에 정의된 Pick, Partial, ReturnType같은 제너릭 타입에 익숙해질것

## 동적 데이터에 인덱스 시그니처 사용하기

```tsx
const rocket = {
  name: "Falcon 9",
  variant: "Block 5",
  thrust: "7,607 kN",
};
```

- 자바스크립트 객체는 문자열 키를 타입에 관계없이 매핑.
- 타입스크립트에서는 타입에 ‘인덱스 시그니처’를 명시하여 유연하게 매핑을 표현할 수 있음.

```tsx
type Rocket = { [property: string]: string };
const rocket = {
  name: "Falcon 9",
  variant: "Block 5",
  thrust: "7,607 kN",
}; //정상
```

- `[property: string]: string` 이 인덱스 시그니처이며 아래의 의미가 있음.
  - 키의 이름 : 키의 위치만 표시하는 용도. 타입체커에서는 사용하지 않음.
  - 키의 타입 : string이나 number또는 symbol의 조합이어야하나, 보통은 string을 사용.
  - 값의 타입: 어떤 것이든 될 수 있음.
- 이런 형식의 타입 체크는 여러 단점이 있음.
  - 잘못된 키를 포함해 모든 키를 허용함. name 대신 Name으로 작성해도 유효한 Rocket타입이 됨.
  - 특정 키가 필요하지 않음. {}도 유효한 Rocket 타입.
  - 키마다 다른 타입을 가질 수 없음. thrust는 string이 아니라 number여야 할 수도 있음.
  - 타입스크립트 언어 서비스가 제대로 동작하지 않게 됨. name: 입력시 자동 완성 기능이 동작하지 않음.

인덱스 시그니처는 부정확하므로 더 다은 방법을 찾아야함.

```tsx
interface Rocket {
  name: string;
  variant: string;
  thrust_kN: number;
}
const falconHeavy: Rocket = {
  name: "Falcon Heavy",
  variant: "v1",
  thrust_kN: 15_200,
};
```

- thrust_kN은 number 타입이며, 타입스크립트는 모든 필수 필드가 존재하는지 확인.
- 타입스크립트에서 제공하는 언어 서비스를 모두 사용가능하게함.

```tsx
function parseCSV(input: string): { [columnName: string]: string }[] {
  const lines = input.split("\n");
  const [header, ...rows] = lines;
  const headerColumns = header.split(",");
  return rows.map((rowStr) => {
    const row: { [columnName: string]: string } = {};
    rowStr.split(",").forEach((cell, i) => {
      row[headerColumns[i]] = cell;
    });
    return row;
  });
}
```

- 인덱스 시그니처는 동적 데이트를 표현할 때 사용.
- 예제처럼 CSV 파일처럼 헤더 행(row)에 열(column)이름이 있고, 데이터 행을 열 이름과 같은 값으로 매핑하는 객체로 나타내고 싶은 경우.

```tsx
interface ProductRow {
  productId: string;
  name: string;
  price: string;
}

declare let csvData: string;
const products = parseCSV(csvData) as unknown as ProductRow[];
```

- 일반적인 상황에서 열 이름이 무엇인지 알 방법이 없음.
- 이럴 때 인덱스 시그니처를 사용.
- 반면에 열 이름을 알고있는 특정한 경우에 parseCSV가 사용된다면 미리 선언해둔 타입을 활용.

```tsx
function safeParseCSV(
  input: string
): { [columnName: string]: string | undefined }[] {
  return parseCSV(input);
}
```

- 선언해 둔 열들이 런타임에 실제로 일치한다는 보장이 없음.
- 이 부분을 해결한다면 값 타입에 undefined를 추가할 수 있음.

```tsx
const rows = parseCSV(csvData);
const prices: { [product: string]: number } = {};
for (const row of rows) {
  prices[row.productId] = Number(row.price);
}

const safeRows = safeParseCSV(csvData);
for (const row of safeRows) {
  prices[row.productId] = Number(row.price);
  //error - Type 'undefined' cannot be used as an index type.
}
```

- 이제 모든 열의 undefined 여부를 체크해야함.
- 체크가 추가되는건 작업이 번거로울 수 있으니 undefined를 타입에 추가할지 여부는 상황에 맞게 판단.

```tsx
interface Row1 {
  [column: string]: number;
}
interface Row2 {
  a: number;
  b?: number;
  c?: number;
  d?: number;
}
type Row3 =
  | { a: number }
  | { a: number; b: number }
  | { a: number; b: number; c: number }
  | { a: number; b: number; c: number; d: number };
```

- 어떤 타입에 가능한 필드가 제한되어 있는 경우라면 인덱스 시그니처로 모델링 하지 말아야함.
- 예를 들어 데이터 A,B,C,D에 같은 키가 있지만, 얼마나 많이 있는지 모른다면 선택적 필드 혹은 유니온 타입으로 모델링 하면 됨.
- 마지막 형태 (Row3)가 가장 정확하지만 사용하기에는 번거로움.

string 타입이 너무 광범위하여 인덱스 시그니처를 사용하는 데 문제가 있으며 이를 해결하는데 두가지 다른 대안을 생각해 볼 수 있음.

```tsx
//Record사용
type Vec3D = Record<"x" | "y" | "z", number>;
// Type Vec3D = {
//	x:number;
//	y:number;
//	z:number;
//}
```

- 첫 번째, Record를 사용하는 방법.
- Record는 키 타입에 유연성을 제공하는 제너릭 타입.
- 특히 string의 부분 집합을 사용할 수 있음.

```tsx
//매핑된 타입 사용
type Vec3D = { [k in "x" | "y" | "z"]: number };
// Type Vec3D = {
//	x:number;
//	y:number;
//	z:number;
//}
type ABC = { [k in "a" | "b" | "c"]: k extends "b" ? string : numbe };
// Type ABC = {
//   a:number;
//   b:string;
//   c:number;
//}
```

- 두 번째, 매핑된 타입을 사용.
- 매핑된 타입은 키마다 별도의 타입을 사용하게 해줌

요약

- 런타임 때 까지 객체의 속성을 알 수 없을 경우에만(예를 들어 CSV 파일에서 로드하는 경우) 인덱스 시그니처를 사용하도록 할 것.
- 안전한 접근을 위해 인덱스 시그니처의 값 타입에 undefined를 추가하는 것을 고려해야함.
- 가능하다면 인터페이스, Record, 매핑된 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하는게 좋음.

## number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

- 자바스크립트는 이상하게 동작하기로 유명한 언어.
- 그중 가장 악명 높은 부분은 타입 강제와 관련된 부분

```tsx
> "0" == 0
true
```

- 암시적 타입 강제 관련 문제는 대부분 === 와 !== 를 사용해서 해결 가능.
- 자바스크립트 객체 모델에도 이상한 부분이 많아 자바스크립트의 객체 모델을 잘 이해해야함.
- 이번 챕터에서는 자바스크립트의 이상한 부분을 다룸.

```tsx
x = {}
> {}
x[[1,2,3]] = 2
> 2
x
> {1,2,3: 2}
```

- 파이썬이나 자바에서 볼 수 있는 해시 가능 객체라는 표현이 자바스크립트에는 없음.

```tsx
{1:2, 3:4}
> {1: 2, 3: 4}
```

- 특히 숫자는 키로 사용할 수 없음.
- 만약 속성 이름으로 숫자를 사용하려고 하면, 자바스크립트 런타임은 문자열로 변환해버림

```tsx
> typeof []
'object'
```

- 배열은 분명히 객체로 취급

```tsx
> x = [1,2,3]
[1,2,3]
> x[0]
1
```

- 그러므로 숫자 인덱스를 사용하는것이 당연함.

```tsx
> x['1']
2
```

- 이상하게 보일지도 모르겠으나, 앞의 인덱스들은 문자열로 변환되어 사용됨.

```tsx
> Object.keys(x)
['0', '1', '2']
```

- Object.keys를 이용해 배열의 키를 나열해 보면 키가 문자열로 출력됨.

```tsx
interface Array<T> {
  //...
  [n: number]: T;
}
```

- 타입 스크립트는 이런 혼란을 바로잡기 위해 숫자 키를 허용하고, 문자열 키와 다른것으로 인식함.
- Array에 대한 타입 선언은 lib.es5.d.ts에서 확인 가능.

```tsx
const xs = [1, 2, 3];
const x0 = xs[0];
const x1 = xs["1"];

function get<T>(array: T[], k: string): T {
  return array[k];
}

//error - Element implicitly has an 'any' type because index expression is not of type 'number'.
```

- 런타임에는 ECMAScript 표준이 서술하는 것 처럼 문자열 키로 인식.
- 완전히 가상이라고 할 수 있으나 타입 체크시점에 오류를 잡을 수 있어 유용함.

```tsx
const keys = Object.keys(xs); //타입이 string[]
for (const key in xs) {
  key; //타입이 string
  const x = xs[key]; //타입이 number
}
```

- Object.keys 같은 구문은 여전히 문자열로 반환됨.
- string이 number에 할당될 수 없으므로, 예제의 마지막 줄이 동작하는것이 이상하게 보일 수 있음.
- 배열을 순회하는 코드 스타일에 대한 실용적인 허용
- 배열을 순회하기에 좋은 방법은 아님

```tsx
for (const x of xs) {
  x; //타입이 number
}
```

- 인덱스에 신경 쓰지 않는다면, for-of를 사용하는게 더 좋음.

```tsx
xs.forEach((x, i) => {
	i; //타입이 Number
	x: //타입이 number
});

```

- 인덱스의 타입이 중요하다면, number 타입을 제공해 줄 Array.prototype.forEach를 사용.

```tsx
for (let i = 0; i < xs.length; i++) {
  const x = xs[i];
  if (x < 0) break;
}
```

- 루프 중간에 멈춰야 한다면, C 스타일인 for(;;)루프를 사용하는 것이 좋음.
- (forEach는 break를 지원하지 않음)
- 타입이 불확실 하다면 (대부분의 브라우저와 자바스크립트 엔진에서 ) for-in 루프는 for-of 또는 C스타일 for 루프에 비해 몇배나 느림

- 인덱스 시그니처가 number로 표현되어 있다면 입력한 값이 number여야 한다는 것을 의미하지만 (for-in 루프는 확실히 제외), 실제 런타임에 사용되는 키는 string.
- 이 부분이 혼란스럽게 느껴질 수 있으나, 일반적으로 string대신 number를 타입의 인덱스 시그니처로 사용할 이유는 많지 않음.
- 만약 숫자를 사용하여 인덱스 할 항목을 지정한다면 Array 또는 튜플 타입을 대신 사용하게 될 것.
- number를 인덱스 타입으로 사용하면 숫자 속성이 어떤 특별한 의미를 지닌다는 오해를 불러 일으 킬 수 있음

```tsx
function checkAccess<T>(xs: ArrayLike<T>, i: number): T {
  if (i < xs.length) {
    return xs[i];
  }
  throw new Error(`배열의 끝을 지나서 ${i}를 접근하려고 했습니다.`);
}
```

- 어떤 길이를 가지는 배열과 비슷한 형태의 튜플을 사용하고 싶다면 타입스크립트에 있는 ArrayLike타입을 사용할 수 있음.

```tsx
const tupleLike: ArrayLike<string> = {
  "0": "A",
  "1": "B",
  length: 2,
  //정상
};
```

- 길이와 숫자 인덱스 시그니처면 있음.
- 이런 경우가 실제로는 드물지만 필요하다면 ArrayLike를 사용.
- 그러나 ArrayLike를 사용하더라도 키는 여전히 문자열임을 명심.

요약

- 배열은 객체이므로 키는 숫자가 아니라 문자열
- 인덱스 시그니처로 사용된 number 타입은 버그를 잡기위한 순수 타입스크립트 코드.
- 인덱스 시그니처에 number를 사용하기 보다 Array나 튜플, 또는 ArrayLike타입을 사용하는 것이 좋음.

## 변경 관련된 오류 방지를 위해 readonly 사용하기

```tsx
function printTriangles(n: number) {
  const nums = [];
  for (let i = 0; i < n; i++) {
    nums.push(i);
    console.log(arraySum(nums));
  }
}
```

- 삼각수를 출력하는 코드(1, 1+2, 1+2+3…)
- 실제 실행시 문제 발생. arraySum이 nums를 변경하지 않는다고 간주.

```tsx
function arraySum(arr: number[]) {
  let sum = 0,
    num;
  while ((num = arr.pop()) !== undefined) {
    sum = sum + num;
  }
}
```

- 이 함수는 배열 안의 숫자들을 모두 합침.
- 그런데 원래 계산이 끝나면 원래 배열이 전부 비게됨.
- 자바스크립트 배열은 내용을 변경할 수 있기 때문에 타입스크립에서도 역시 오류없이 통과됨 (불변성이 깨짐)

```tsx
function arraySum(arr: readonly number[]) {
  let sum = 0,
    num;
  while ((num = arr.pop()) !== undefined) {
    //error - Property 'pop' does not exist on type 'readonly number[]'.
    sum = sum + num;
  }

  return sum;
}
```

- 오류의 범위를 줄이기 위해 arraySum이 배열을 변경하지 않는다는 선언 사용.
- readonly 접근 제어자 사용
- 에러가 발생됨을 확인

readonly number[] 는 타입 이고 number[]와 구분되는 몇 가지 특징이 있음.

- 배열의 요소는 읽을 수 있지만, 쓸 수는 없음
- length를 읽을 수 있지만, 바꿀 수는 없음. (배열을 변경함)
- 배열을 변경하는 pop을 비롯한 다른 메서드를 호출 할 수 없음.

```tsx
const a: number[] = [1, 2, 3];
const b: readonly number[] = a;
const c: number[] = b;
// The type 'readonly number[]' is 'readonly' and cannot be assigned to the mutable type 'number[]'.
```

- number[]는 readonly number[]보다 기능이 많기 때문에 , readonly number[]의 서브 타입이 됨.
- 따라서 변경 가능한 배열을 readonly 배열에 할당할 수 있음. 그러나 그 반대는 불가능

타입 단언문 없이 readonly 접근 제어자를 제거할 수 있다면 readonly는 쓸모없을 것이므로 여기서 오류가 발생하는게 맞는 방향.

매개 변수를 readonly로 선언하면 다다음과 같은 일이 생김.

- 타입 스크립트는 매개 변수가 함수 내에서 변경이 일어나는지 체크
- 호출하는 쪽에서 함수가 매개 변수를 변경하지 않는다는 보장을 받게됨.
- 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있음.

- 자바스크립트에서는(타입스크립트에서도 마찬가지) 명시적으로 언급하지 않는 한, 함수가 매개 변수를 변경하지 않는다는 가정.
- 그러나 이런 암묵적인 방법은 타입 체크에 문제를 일으 킬 수 있음(차후 재언급 예정)
- 명시적인 방법을 사용하는 것이 컴파일러와 사람에게 모두 좋음.

```tsx
function array(arr: readonly number[]) {
  let sum = 0;
  for (const num of arr) {
    sum = sum + num;
  }
  return sum;
}
```

- 배열을 변경하지 않게 arraySum을 수정

- 만약 함수가 매개변수를 변경하지 않는다면 readonly로 선언해야함.
- 더 넓은 타입으로 호출 할 수 있고, 의도치 않은 변경은 방지됨.
- 이로인한 단점은 상대적으로 적음.

readonly 사용시의 고려사항

- 매개변수가 readonly로 선언되지 않은 함수를 호출해야할 경우가 있음.
- 만약 함수가 매개 변수를 변경하지 않고도 제어가 가능하다면 readonly로 선언하면 됨.
- 그런데 어떤 함수를 readonly로 만들면, 그 함수를 호출하는 다른 함수도 모두 readonly로 만들어야함.
- 그러면 인터페이스를 명확히 하고, 타입 안전성을 높일 수 있기 때문에 꼭 단점이라고 볼 순 없음.
- 단, 다른 라이브러리에 있는 함수를 호출하는 경우라면 타입 선언을 바꿀 수 없으므로 단언문을 사용(as number[])

e.g readonly를 사용하면 지역 변수와 관련된 모든 종류의 변경 오류를 방지 할 수 있음.

- 소설에 다양한 처리를 하는 프로그램을 만든다고 가정.
- 연속된 행을 가져와서 빈 줄을 기준으로 구분되는 단락으로 나누는 기능을 하는 프로그램.

예제 텍스트

```
Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsu
```

```tsx
//실제로는 이런 긴 코드 보다 lines.join(’\n’).split(/\n\n+/)으로 작성하면 됨.
function parseTaggedText(lines: string[]): string[][] {
  const paragraphs: string[][] = [];
  const currPara: string[] = [];

  const addParagraph = () => {
    if (currPara.length) {
      paragraphs.push(currPara);
      currPara.length = 0; //배열을 비움
    }
  };

  for (const line of lines) {
    if (!line) {
      addParagraph();
    } else {
      currPara.push(line);
    }
  }
  addParagraph();
  return paragraphs;
}
```

- 해당 코드 실행시 [ [],[],[] ] 출력. 이는 잘못됨.

```tsx
paragraphs.push(currPara);
currPara.length = 0;
```

- currPara의 내용이 삽입되지 않고 배열의 참조가 삽입됨.
- currPara에 새 값을 채우거나 지운다면 동일한 객체를 참조하고 있는 paragraphs요소에도 변경이 반영.
- 새 단락을 paragraphs에 삽입하고 바로 지워버림.
- currPara를 readonly로 선언하여 이런 동작을 방지할 수 있음

```tsx
//실제로는 이런 긴 코드 보다 lines.join(’\n’).split(/\n\n+/)으로 작성하면 됨.
function parseTaggedText(lines: string[]): string[][] {
  const currPara: readonly string[] = [];
  const paragraphs: string[][] = [];

  const addParagraph = () => {
    if (currPara.length) {
      paragraphs.push(currPara);
      currPara.length = 0; //배열을 비움
      //error
      /*
			Argument of type 'readonly string[]' is not assignable to parameter of type 'string[]'.
			  The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
			Cannot assign to 'length' because it is a read-only property.
			Property 'push' does not exist on type 'readonly string[]'.
			*/
    }
  };

  for (const line of lines) {
    if (!line) {
      addParagraph();
    } else {
      currPara.push(line);
    }
  }
  addParagraph();
  return paragraphs;
}
```

- 선언을 바꾸는 즉시 코드 내에서 몇 가지 오류가 발생.
- currPara을 let으로 선언하고 변환이 없는 메서드를 사용함으로써 두개의 오류를 수정

```tsx
let currPara: readonly string[] = [];
// ..
currPara = [];
// ..
currPara = currPara.concat([line]);
```

- push와 달리 concat은 원본을 수정하지 않고 새 배열을 반환.
- 선언부를 const에서 let으로 바꾸고 readonly를 추가함으로써 한쪽의 변경 가능성을 또 다른 쪽으로 옮긴 것.
- currPara변수는 이제 가리키는 배열을 자유롭게 변경할 수 있지만, 원본 배열 자체는 변경이 불가.

여전히 paragraphs에 대한 오류는 남아있으므로, 이 오류를 바로 잡기위해 아래의 방법 고려

1. currPara의 복사본 생성

```tsx
paragraphs.push([...currPara]);
```

- currPara는 readonly로 유지되지만, 복사본은 원하는 대로 변경이 가능하기 때문에 오류는 사라짐.

1. paragraphs(그리고 함수의 반환 타입)를 readonly string[]의 배열로 변경하는 방법.

```tsx
const paragraphs: (readonly string[])[] = [];
```

- 여기서 괄호의 의미는 readonly string[][]는 readonly 배열의 변경 가능한 배열이 아니라, 변경가능한 배열의 readonly 배열을 뜻함.
- 앞의 코드는 동작은 하겠으나 parseTaggedText의 사용자에게는 조금 불친절하게 느껴질 것.
- 이미 함수가 반환한 값에 대해 영향을 끼치는 것이 맞는 방법인지 고민해 봐야함.

1. 세번째, 배열의 readonly 속성을 제거하기 위해 단언문을 사용.

```tsx
paragraphs.push(currPara as string[]);
```

- readonly는 얕게(shallow) 동작하는 것에 유의하며 사용해야함.

```tsx
const dates: readonly Date[] = [new Date()];
dates.push(new Date());
//error - readonly Date[] 형식에 push 속성이 없음
dates[0].setFullYear(2037); //정상
```

- 앞에서 이미 readonly string[][] 사레를 보았음. 만약 객체의 readonly 배열이 있다면 그 객체 자체는 readonly가 아님.

```tsx
interface Outer {
  inner: {
    x: number;
  };
}

const o: Readonly<Outer> = { inner: { x: 0 } };
o.inner = { x: 1 };
//error - Cannot assign to 'inner' because it is a read-only property.
o.inner.x = 1; //정상
```

- 비슷한 경우로는 Readonly 제너릭에도 해당됨.

```tsx
type T = Readonly<Outer>;
```

- 이 경우 readonly 접근제어자는 inner에만 적용되고 x는 아님.
- 현재 시점에는 깊은(deep) readonly 타입이 기본으로 지원되지 않지만, 제너릭을 만들면 깊은 readonly 타입을 사용할 수 있음. → 현재 논의중 https://github.com/microsoft/TypeScript/issues/13923
- 그러나 제너릭은 만들기 까다롭기 때문에 라이브러리를 사용하는것을 권장. (https://github.com/ts-essentials/ts-essentials/tree/master/lib/deep-readonly)

```tsx
let obj: { readonly [k: string]: number } = {};
//또는 Readonly<{[k:string]:number}
obj.hi = 45;
//error
//Index signature in type '{ readonly [k: string]: number; }' only permits reading.
obj = { ...obj, hi: 12 }; //정상
obj = { ...obj, bye: 34 }; //정상
```

- 인덱스 시그니처에도 readonly를 쓸 수 있음.
- 읽기는 허용하되 쓰기를 방지하는 효과가 있음.

요약

- 만약 함수가 매개변수를 수정하지 않는다면 readonly로 선언하는것이 좋음.
- readonly 매개 변수는 인터페이스를 명확하게 하며, 매개 변수가 변경되는것을 방지함.
- readonly를 사용하면 변경하면서 발생하는 오류를 방지할 수 있고, 변경이 발생하는 코드를 쉽게 찾을 수 있음.
- const와 readonly의 차이를 이해해야함.
- readonly는 얕게 (shallow) 동작한다는것을 명심.

## 매핑된 타입을 사용하여 값을 동기화하기

산점도를 그리기위한 UI 컴포넌트를 작성한다고 가정.

디스플레이와 동작을 제어하기 위한 몇 가지 다른 타입의 속성이 포함됨.

```tsx
interface ScatterProps {
  // The data
  xs: number[];
  ys: number[];

  // Display
  xRange: [number, number];
  yRange: [number, number];
  color: string;

  // Events
  onClick: (x: number, y: number, index: number) => void;
}
```

- 불필요한 작업을 피하기 위해, 필요한 때에만 차트를 다시 그릴 수 있음.
- 데이터나 디스플레이 속성이 변경되면 다시 그려야 하지만, 이벤트 핸들러가 변경되면 그럴 필요가 없음.
- 이런 종류의 최적화는 리액트 컴포넌트에서는 일반적인 일인데, 렌더링 할 때마다 이벤트 핸들러 Props의 새 화살표 함수로 설정됨.

**최적화를 두가지 방법으로 구현**

첫번째 방법

```tsx
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== "onClick") return true;
    }
  }
  return false;
}
```

- 만약 새로운 속성이 추가되면 shouldUpdate 함수는 값이 변경될 때 마다 차트를 다시 그릴 것.
- 이렇게 처리하는 것을 보수적 접근법(conservative) 혹은 실패에 닫힌(fail close) 접근법 이라고 함.
- 이 접근법은 차트는 정확하지만 너무 자주 그려질 가능성이 있음.

두번째 방법

```tsx
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // (no check for onClick)
  );
}
```

- 이 코드는 차트를 불필요하게 다시 그리는 단점을 해결.
- 하지만 실제로 차트를 다시 그려할 경우에 누락되는 일이 생길 수 있음.
- 이는 히포크라테스 전집에 나오는 원칙 중 하나인 ‘우선 망치지 말것(first, do not harm)을 어기기 때문에, 일반적인 방법은 아님.

앞선 두가지 방법 모두 이상적이지 않음. 새로운 속성이 추가될 때 직접 shouldUpdate를 고치는게 나음.

```tsx
interface ScatterProps {
  xs: number[];
  ys: number[];
  onClick: (x: number, y: number, index: number) => void;
  //참고 : 여기에 속성을 추가하려면, shouldUpdate를 고칠것
}
```

- 이 방법 역시 최선이 아니며, 타입 체커가 대신할 수 있게 하는것이 좋음.

```tsx
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

- `[k in keyof ScatterProps]` 은 타입 체커에게 REQUIRES_UPDATE가 ScatterProps과 동일한 속성을 가져야 한다는 정보를 제공

```tsx
interface ScatterProps {
  //...
  onDoubleClick: () => void;
}
```

- 나중에 ScatterProps에 새로운 속성을 추가하는 경우 위와 같은 코드 형태를 가짐
- 그리고 REQUIRES_UPDATE의 정의에 오류가 발생.

```tsx
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  //error - 'onDoubleClick' 속성이 타입에 없음.
};
```

- 이런 방식은 오류를 정확하게 잡아냄. 속성을 삭제하거나, 이름을 바꾸어도 비슷한 오류가 발생.
- 여기서 boolean값을 가진 객체를 사용했다는 점이 중요함.

```tsx
const PROPS_REQUIRING_UPDATE: (key of ScatterProps)[] = [
		'xs',
		'ys',
		// ...
];
```

- 배열을 사용했다면 위와 같은 코드가 됨.

요약

- 실패에 열린 방법을 선택할 지, 닫힌 방법을 선택할 지 정해야함.
- 매핑된 타입은 한객체가 또 다른 객체와 정확히 같은 속성을 가지게 할 때 이상적임.
- 이번 예제와 같이 매핑된 타입을 사용해 타입 스크립트가 코드에 제약을 강제하도록 할 수 있음.
- 매핑된 타입을 사용해서 관련된 값과 타입을 동기화함.
- 인터페이스에 새로운 속성을 추가할 때, 선택을 강제하도록 매핑된 타입을 고려해야함.

---

기타 링크 :

- type과 interface와의 차이 설명 [https://medium.com/humanscape-tech/type-vs-interface-언제-어떻게-f36499b0de50](https://medium.com/humanscape-tech/type-vs-interface-%EC%96%B8%EC%A0%9C-%EC%96%B4%EB%96%BB%EA%B2%8C-f36499b0de50)
- typescript의 Excess Property Checking in Typescript 이해하기 : https://dev.to/this-is-learning/understanding-excess-property-checking-in-typescript-ook
- keyof 에 대한 설명 : https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- InstanceType에 대한 설명 : https://www.typescriptlang.org/docs/handbook/utility-types.html#instancetypetype
- ReturnType에 대한 설명 : https://www.typescriptlang.org/ko/docs/handbook/utility-types.html#returntypetype
- Typescript의 tuple에 대한 설명 : [https://velog.io/@from_numpy/TypeScript-Tuple튜플](https://velog.io/@from_numpy/TypeScript-Tuple%ED%8A%9C%ED%94%8C)
