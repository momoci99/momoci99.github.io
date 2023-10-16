---
layout: single
title: "[Typescript] Effective Typescript 1장 - 타입스크립트 알아보기"
tag:
  - Effective Typescript
---

이펙티브 타입스크립트 1장 내용을 정리하였습니다.

# 타입스크립트와 자바스크립트의 관계

- 타입스크립트는 자바스크립트의 슈퍼셋 언어
- 모든 자바스크립트 코드는 타입스크립트에 속한다고 볼 수 있음.
- 그러나, 일부 자바스크립트만이 타입 체커를 통과함.

![Untitled](https://github.com/momoci99/WIL/blob/main/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8/%EB%A6%AC%EC%86%8C%EC%8A%A4/1%EC%9E%A5/pic1.png?raw=true)

타입스크립트는 자바스크립트의 동작을 모델링함.

```tsx
const x = 2 + "3"; //정상
const y = "2" + 3; //정상
```

위 코드는 C, C++, Java에서는 런타임 오류 발생. 그러나 타입 스크립트의 타입 체커는 정상으로 인식.

```tsx
const a = null + 7;
const b = [] + 12;

alert("Hello", "Typescript");
```

런타임에는 문제가 없으나 타입 스크립트의 타입 체커는 에러 표시

![Untitled](https://github.com/momoci99/WIL/blob/main/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8/%EB%A6%AC%EC%86%8C%EC%8A%A4/1%EC%9E%A5/pic2.png?raw=true)

- 타입스크립트의 타입 체크는 런타임 오류까지 잡아낼 수 없음.

```tsx
const names = ["Alice", "Bob"];
console.log(names[2].toUpperCase()); //런타임에서 배열의 크기를 벗어난 인덱스 참조.
```

![Untitled](https://github.com/momoci99/WIL/blob/main/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8/%EB%A6%AC%EC%86%8C%EC%8A%A4/1%EC%9E%A5/pic3.png?raw=true)

# 타입스크립트 설정 이해하기

- 타입스크립트는 설정(tsconfig)을 통해 타입 체크 규칙을 설정할 수 있음
- 대표적인 설정에는

  - `noImplicitAny`
    - 암묵적인 any 타입을 허용하지 않음. 기본설정 true 권장.
    - https://www.typescriptlang.org/tsconfig#noImplicitAny
  - `strictNullChecks`
    - null과 undefined가 모든 타입에 할당될 수 있는지 여부를 체크. 기본설정 true 권장

- `noImplicitAny` 과 `strictNullChecks` 는 자바스크립트 에서 타입스크립트로 코드 마이그레이션 시 어려움을 겪게 만드는 원인 중 하나.
- 그러나 신규 타입스크립트 프로젝트를 시작한다면 기본값으로 활성화 시킬것을 권장.

# 코드 생성과 타입은 관계없음을 이해하기.

타입 스크립트 컴파일러는 아래의 두가지 역할을 수행.

1. 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구 버전 자바스크립트로 트랜스파일(transpile)
2. 코드의 타입 오류를 체크.

- 이 두가지 역할은 완벽히 독립적으로 수행.
- 타입 체크 에러가 코드 변환에 영향을 미치지 않음.
- 마찬가지로 자바스크립트의 실행 시점에도 타입은 영향을 주지 않음.

따라서 타입 오류가 있는 코드도 컴파일이 가능하며 실행도 가능.

![Untitled](https://github.com/momoci99/WIL/blob/main/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8/%EB%A6%AC%EC%86%8C%EC%8A%A4/1%EC%9E%A5/pic4.png?raw=true)

- 단, 트랜스파일 과정에서 타입 체크 에러 발생시 컴파일을 하지 않으려면 `noEmitOnError` 를 설정하거나 빌드 도구에서 동일하게 설정.
- 현업에서는 보통 `noEmitOnError` 를 활성화시켜 사용함.

런타임에는 타입 체크가 불가능.

![Untitled](https://github.com/momoci99/WIL/blob/main/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8/%EB%A6%AC%EC%86%8C%EC%8A%A4/1%EC%9E%A5/pic5.png?raw=true)

- instanceof 체크는 런타임에 수행. 그러나 Rectangle은 타입이므로 런타임 시점에 아무것도 할 수 없음.
- 자바스크립트로 컴파일 되는 과정에서 삭제됨. 따라서 런타임에 `Rectangle is not defined` 에러 발생

위에 언급된 shape 타입을 명확히 하기 위해서 런타임에 타입정보를 유지하는 방법이 필요.

![Untitled](https://github.com/momoci99/WIL/blob/main/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8/%EB%A6%AC%EC%86%8C%EC%8A%A4/1%EC%9E%A5/pic6.png?raw=true)

- Square 타입에만 존재하는 height 속성이 존재하는지 검사. 에러를 해결함.

혹은 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 태그 기법 활용

```tsx
interface Square {
  kind: "square";
  width: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape.kind === "rectangle") {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}

const shape: Square = {
  kind: "square",
  width: 10,
};

console.log(calculateArea(shape));
```

타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법.

```tsx
class Square {
  constructor(public width: number) {}
}

class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width);
  }
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.width;
  } else {
    return shape.width * shape.width;
  }
}
```

- 인터페이스는 타입으로만 사용가능하지만, Rectangle을 클래스로 선언하면 타입과 값으로 모두 사용가능.

타입 연산은 런타임에 영향을 주지 않음.

- 위 코드는 string 혹은 number 타입인 값을 항상 number로 정제하는 경우를 가정.

```tsx
function asNumber(val: number | string): number {
  return val as Number;
}
```

- js로 변환된 코드

```jsx
function asNumber(val) {
  return val;
}
```

- 실제 변환된 코드에 아무 정제과정이 없음. (타입은 변환시 휘발되므로)
- 값을 정제하고 싶다면 런타임의 타입을 체크하고 자바스크립트 연산을 통해 변환 수행

```tsx
function asNumber(val: number | string): number {
  return typeof val === "string" ? Number(val) : val;
}
```

런타임 타입은 선언된 타입과 다를 수 있음.

```tsx
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      berak;
    default:
      console.log("실행되지 않을까봐 걱정됩니다");
  }
}
```

- value가 boolean값이라 `default` 에 정의된 콘솔이 실행되지 않을 것 같지만 실제 런타임에는 타입을 체크하지 않으므로 다른 값(문자열)이 들어온다면 default가 실행 될 수 있음.
- 타입스크립트는 런타임 타입과 선언된 타입이 맞지 않을 수 있다는 것을 명심.

**타입스크립트 타입으로는 함수를 오버로드 할 수 없음.**

- 타입스크립트에서는 타입과 런타임의 동작이 무관하므로 함수 오버로딩은 불가능함.

![Untitled](https://github.com/momoci99/WIL/blob/main/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8/%EB%A6%AC%EC%86%8C%EC%8A%A4/1%EC%9E%A5/pic7.png?raw=true)

함수 오버로딩 기능은 지원하기는 하지만, 온전히 타입 수준에서만 동작함.

```tsx
function add(a: number, b: number): number;
function add(a: string, b: string): string;

function add(a, b) {
  return a + b;
}

const three = add(1, 2); //타입이 number
const twelve = add("1", "2"); //타입이 string
```

- add 함수 선언문은 타입스크립트가 자바스크립트로 변환되면서 제거됨.

타입스크립트 타입은 런타임 성능에 영향을 주지 않음. (타입은 컴파일시 제거되므로)

- 런타임 오버헤드가 없는 대신 빌드 타임 오버헤드가 존재.
- 일반적인 타입스크립트 컴파일 성능은 매우 좋은편.
- 호환성(eg: es5 타겟 컴파일)과 런타임 성능은 타입과 무관.

# 구조적 타이밍에 익숙해지기

- 타입스크립트는 본질적으로 [덕 타이핑 기반](https://ko.wikipedia.org/wiki/%EB%8D%95_%ED%83%80%EC%9D%B4%ED%95%91)
- 구조적 타이핑을 제대로 이해한다면 오류인 경우와 아닌 경우의 차이를 잘 알 수 있음.

물리 라이브러리와 2D 벡터 타입을 다루는 경우를 가정.

```tsx
interface Vector2D {
  x: number;
  y: number;
}
```

벡터의 길이계산 함수 정의

```tsx
function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
```

이름 속성이 추가된 벡터를 정의

```tsx
interface NamedVerctor {
  name: string;
  x: number;
  y: number;
}
```

NamedVector는 number 타입의 x, y 속성이 있으므로 calculateLength 함수를 호출 할 수 있음.

```tsx
const v: NamedVerctor = { x: 3, y: 4, name: "Zee" };
calculateLength(v); // 정상, 결과는 5
```

- Vector2D와 NamedVector의 관계를 전혀 선언하지 않았음에도 타입스크립트는 구조가 호환되기 때문에 NamedVector는 calculateLength 함수를 호출 가능

다만 구조적 타이핑으로 문제가 발생하는 경우도 있음.

```tsx
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
```

벡터의 길이를 1로 만드는 정규화 함수를 작성.

```tsx
function normalize(v: Vector3D) {
  const length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}
```

```tsx
> normalize({x: 3, y:4, z:5})
//{
//  "x": 0.6,
//  "y": 0.8,
//  "z": 1
//}
```

- z 값이 무시되어 기대하지 않은 결과값이 반환됨.
- calculateLength는 Vector2D 을 받게 되어있으나 구조적으로 Vector3D와 호환되어 타입 체커가 문제로 인식하지 않음.
- 함수 작성시 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만을 가질거라 생각하기 쉬움. (봉인된 혹은 정확한 - sealed or precise 타입으로 불림)
- 그러나 타입스크립트 타입 시스템에서는 이를 제대로 표현할 수 없음. 이로인해 예상하지 못한 결과가 발생

```tsx
function calculateLengthL1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis]; //에러 발생
    length = length + Math.abs(record);
  }
}
```

```tsx
Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Vector3D'.
  No index signature with a parameter of type 'string' was found on type 'Vector3D'.
```

- 파라미터 v는 Vector3D로 타입핑 되어있으나 실제로는 어떤 속성이든 가질 수 있음.
- 따라서 타입스크립트는 올바르게 에러를 잡아낸것.

```tsx
function calculateLengthL1(v: Vector3D) {
  return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z);
}
```

- 이 경우 루프로 각 속성을 순회하는 것 보다 각 속성을 더하는 구현이 나은 방향

```tsx
class C {
  foo: string;
  constructor(foo: string) {
    this.foo = foo;
  }
}

const c = new C("instance of C");
const d: C = { foo: "object literal" }; //정상 처리
```

- 변수 d가 C 타입에 할당되는 이유는 d가 foo 속성을 가지고 있으므로 구조적 타이핑에 의거. 문제가 없음.
- 단 C의 생성자에 단순 할당이 아닌 연산로직이 존재한다면 a의 경우 생성자를 실행하지 않으므로 문제 발생.

테스트를 작성할 때도 구조적 타이핑이 유리함.

```tsx
interface Author {
  first: string;
  last: string;
}
function getAutors(database: PostgresDB): Author[] {
  const authorRows = database.runQuery("SELECT FIRST, LAST FROM AUTHORS");
  return authorsRows.map((row) => ({ first: row[0], last: row[1] }));
}
```

```tsx
interface DB {
  runQuery: (sql: string) => any[];
}
function getAuthors(database: DB): Author[] {
  const authorRow = database.runQuery(`SELECT FIRST, LAST FROM AUTHORS`);
  return authorsRows.map((row) => ({ first: row[0], last: row[1] }));
}
```

- DB 인터페이스에 runQuery 메서드가 있으므로 실제 환경에서도 getAuthors에 PostgressDB를 사용할 수 있음.
- 따라서 PostgresDB가 DB 인터페이스를 구현하는지 명확히 선언할 필요가 없음.

```tsx
test('getAuthors', () => {
 const authors = getAuthors({
	 runQuery(sql:string) {
		return [['Toni', 'Morrison'], ['Maya', 'Angelou']];
  }
 expect(authors).toEqual([
		{first:'Toni', last:'Morrison'},
		{first:'Maya', last:'Angelou'},
	]);
});
```

# any 타입 지양하기

- 타입스크립트의 타입 시스템은 점진적이고 선택적으로 사용할 수 있음.
- 코드에 타입을 조금씩 추가할 수 있으며, 언제든지 타입을 해제 할 수 있음.
- `any` 타입을 사용함으로써 위 기능들을 사용 가능.

```tsx
let age: number;
age = "12"; //type error!

age = "12" as any; //ok
```

- as any를 사용하여 오류를 해결하 수 있으나 일부 특별한 경우를 제외하고는 any 사용은 지양해야함

## any 타입에는 안정성이 없음

```tsx
age = age + 1; //런타임에는 에러가 없어도 age는 문자열이므로 '121'
```

- age는 위에서 number로 선언되어있으나 as any사용으로 인해 string으로 할당을 할 수 있게됨.
- 타입 체커는 선언에따라 number로 판단. 이로인해 혼돈이 발생

## any는 함수 시그니처를 무시

- 함수를 작성할때는 시그니처를 명시. 호출하는 쪽은 약속된 타입의 입력을 제공. 함수는 약속된 타입의 출력을 반환.
- 그러나 any 타입은 이 모든 약속을 무시할 수 있음

```tsx
function calculateAge(birthDate: Date): number {
  //...
}

let birthDate: any = "1990-01-19"; //정상 처리
calculateAge(birthDate);
```

- birthDate 매개변수는 string이 아닌 Date 타입이어야함.
- 그러나 any 타입 사용으로 calculateAge의 시그니처를 무시.
- 특히 자바스크립트는 암시적으로 타입이 변환되므로 문제가 될 수 있음.

## any타입 사용시 intellisense가 동작하지 않음.

- any타입을 사용하게 되면 intellisense가 타입, 속성을 추론 할 수 없어 코드 작성시 알 수가 없음.

## any 타입은 코드 리팩터링시 버그를 감춤.

```tsx
interface ComponentProps {
  onSelectItem: (item: any) => void;
}
```

```tsx
function renderSelector(props: ComponentProps) {
  /* ... */
}
let selectedId: number = 0;

function handleSelectItem(item: any) {
  selectedId = item.id;
}

renderSelector({ onSelectItem: handleSelectItem });
```

- 코드가 위와 같을 때, ComponentPros의 시그너처를 다음과 같이 변경

```tsx
interface ComponentPros {
  onSelectItem: (id: number) => void;
}
```

- 타입 체크는 모두 통과했겠으나, handleSelectItem은 any 타입의 매개변수를 받고있으므로 id를 전달 받으면 런타임에서, 존재하지 않는 속성을 참조하고 있다는 에러가 발생.
- any가 아니라 구체적인 타입을 사용했다면 타입 체커가 사전에 에러를 발견했을것.

## any는 타입 설계를 감춤.

- 어플리케이션 상태 같은 객체를 정의할 때 복잡성을 가짐.
- 그러나 any탑을 사용하면 간단하게 정의하게 됨.
- 그렇게되면 설계를 알 수 없게 되어 다른 동료가 코드 검토시 알 수있는 방법이 없음.

## any는 타입시스템의 신뢰도를 떨어트림.

- 타입스크립트의 타입 체커는 타입을 통해 실수를 바로잡고 신뢰도를 향상시킴.
- 그러나 any타입을 사용한다면 런타임에서 발생할 오류를 미리 잡기가 어려워지고 신뢰도가 그만큼 하락함.
- 어쩔 수 없이 any를 사용해야하는 경우가 있으나 사용을 자제해야함.
