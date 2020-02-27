---
layout: single
title:  "Javascript Array 가볍게 정리"
tag : 
    - Array
    - Javascript
---

# 들어가며..

javascript의 array는 다른 정적 언어에 비해 재미있는 특성을 가졌다. 보통 배열이라고 하면 보통 정해진 길이를 가지고 있고 동일한 형식의 요소의 연속적인 모음을 의미한다. 하지만 javascript에서는 좀 다르다. 길이도 자유롭게 변경되고 요소의 형식은 구별되지 않는다. 이런점에서 javascript의 array만의 특징과 재미있는점을 정리한다.


# 1. 무엇이든 환영

다른 정적 언어에서는 배열을 선언할때 보통 배열의 크기, 저장할 요소의 형식이 미리 지정되어 있어야한다. 그렇지 않으면 컴파일러에서 에러를 뿜어내지만 javascript는 아래의 예제처럼 모두 자유다.


```js
//ok
let a = [];

//ok
let b = ["foo","bar"];

//ok
let c = [1,"foo", 3.5];

//Nan, null, undefined도 저장할 수 있다. ok
let whyThisWorking = [NaN, null, undefined];

//ok
let objectArray = [{aa:11},{},1,2,"asdf"}];

console.log(a); 
//[]

console.log(b); 
//["foo", "bar"]

console.log(c); 
//[1, "foo", 3.5]

console.log(whyThisWorking); 
//[NaN, null, undefined]

console.log(objectArray); 
//[{…}, {…}, 1, 2, "asdf"] 객체 전개시 객체 내용이 출력된다.


```

다만 `TypedArray` 라고 하는 형식이 지정된 배열이 존재한다. 이 TypedArray는 추후에 따로 다뤄보도록 하겠다. [링크](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)


작은 팁이지만 간혹 배열의 크기를 지정하여 지정된 값으로 선언해야하는 경우가 종종 발생한다. 이때는 아래의 방법으로 배열을 만들어보자.

```js
//아래 코드는 es6를 지원하는 환경에서 동작함.
let filledArray = Array(3).fill(0);

console.log(filledArray);
// [0, 0, 0]

```




# 2. 길이도 마음대로
js의 array는 길이또한 마음대로 늘였다 줄였다 할 수 있다. array의 크기는 `array.length`에서 확인할 수 있다. 이 length는 배열의 길이를 나타내며 임의의 정수(0 이상 2^32 미만의 값)를 대입하면 배열의 길이를 조절 할 수 있다.


**배열의 길이를 늘리는 예**
```js
let a = [1,2,3];

console.log(a.length);
//3

//length 값을 수정
a.length = 5;

console.log(a)
//[1, 2, 3, empty × 2]

```

**배열의 길이를 줄이는 예**
```Js
let a = [1,2,3];


console.log(a.length);
//3

//lenth를 줄이면 요소가 삭제됨
a.length = 2;

console.log(a);
//[1, 2]

```


참고로 length에 올바르지 않은 값을 대입하면 아래의 에러가 발생한다.
> Uncaught RangeError: Invalid array length

## 2-1. length는 요소의 갯수가 아니다.

array.length에 대한 이야기를 좀 더 하자면  `array.length`는 배열의 길이를 의미하지 요소 갯수를 의미하는것은 아니다. javascript의 배열 특성상 빈 요소가 들어갈 수 있기 때문이다. 아래 예제를 살펴보자

[코드 참조-length 와 숫자형 속성의 관계](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array#length_%EC%99%80_%EC%88%AB%EC%9E%90%ED%98%95_%EC%86%8D%EC%84%B1%EC%9D%98_%EA%B4%80%EA%B3%84)


```js

const fruits = [];
fruits.push('banana', 'apple', 'peach');

console.log(fruits.length);
// 3

fruits[5] = 'mango';
console.log(fruits[5]);           // 'mango'
console.log(Object.keys(fruits)); // ['0', '1', '2', '5']
console.log(fruits.length);       // 6

```

## 2-2. index의 유효성을 검사하지 않는다.

일반적인 정적 언어에서는 범위를 벗어나거나 올바르지 않은 index로 array에 접근하면 대번에 에러를 발생시킨다. 하지만 javascript는 좀 다르다. 아래의 예시를 보면 에러가 날것같지만 에러가 발생하지 않는다.

```js

let a = [1,2,3];

//배열의 크기보다 더 큰 index로 배열에 접근한다.
a[6] = 4;

console.log(a);
//[1, 2, 3, empty × 3, 4]

//잘못된 index로 배열에 접근한다.
a[-1];
//아무일도 일어나지 않는다.

console.log(a);
//[1, 2, 3, empty × 3, 4]

```

# 3. 배열의 요소를 추가하거나 제거하는 메서드 push(), pop(), shift(), unshift()

javascript의 요소를 다루는데에 여러 메서드들이 있지만 우선 주로 사용하는 4가지를 살펴보자.


**Array.prototype.push()**

`push()`메서드는 배열의 **끝**에 하나 이상의 요소를 추가하고 배열의 새로운 길이를 반환하는 메서드이다.


예제코드 참조
[MDN-Array.prototype.push()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/push)
```js
const animals = ['pigs', 'goats', 'sheep'];

const count = animals.push('cows');
console.log(count);
// expected output: 4
console.log(animals);
// expected output: Array ["pigs", "goats", "sheep", "cows"]

animals.push('chickens', 'cats', 'dogs');
console.log(animals);
// expected output: Array ["pigs", "goats", "sheep", "cows", "chickens", "cats", "dogs
```
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/concat

`push()`메서드를 이용하여 두 배열을 하나로 만드는 경우 아래의 예제를 참고해보자
[MDN-Array.prototype.push()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/push)
```js

var vegetables = ['설탕당근', '감자'];
var moreVegs = ['셀러리', '홍당무'];

// 첫번째 배열에 두번째 배열을 합친다. 
// vegetables.push('셀러리', '홍당무'); 하는 것과 동일하다.
Array.prototype.push.apply(vegetables, moreVegs);

console.log(vegetables); // ['설탕당근', '감자', '셀러리', '홍당무']

```
위 예제에 쓰인 `apply()` 메서드를 사용하지 않고 전개 연산자를 사용할 수 있다.

```js
var vegetables = ['설탕당근', '감자'];
var moreVegs = ['셀러리', '홍당무'];

// 첫번째 배열에 두번째 배열을 합친다. 
// vegetables.push('셀러리', '홍당무'); 하는 것과 동일하다.
vegetables.push(...moreVegs);

console.log(vegetables); // ['설탕당근', '감자', '셀러리', '홍당무']
```

`concat()`과 무슨 차이가 있느냐고 할수 있는데 concat()는 두 배열을 이어 붙혀 **새로운 배열**을 반환환다. 반면 `push()`는 기존 배열을 수정한다. 


**Array.prototype.pop()**

`pop` 메서드는 배열의 마지막 요소를 제거하고 그 요소를 반환한다.


예제코드 참조
[MDN-Array.prototype.pop()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/pop)

```js
const plants = ['broccoli', 'cauliflower', 'cabbage', 'kale', 'tomato'];

console.log(plants.pop());
// expected output: "tomato"

console.log(plants);
// expected output: Array ["broccoli", "cauliflower", "cabbage", "kale"]

plants.pop();

console.log(plants);
// expected output: Array ["broccoli", "cauliflower", "cabbage"]


```

만약 요소가 없는 빈 배열일때는 `undefined`를 리턴한다.


**push & pop**

push와 pop을 활용하면 array를 자료구조 `stack` 처럼 사용하는것이 가능하다. `stack`의 특징인 먼저 들어온 요소가 나중에 빠져나간다는 특성을 이용한것이다.


**Array.prototype.shift()**

`shift()` 메서드는 배열에서 첫 번째 요소를 제거하고, 제거된 요소를 반환한다. 이 메서드 또한 원본 배열을 수정한다.



예제코드 참조
[MDN-Array.prototype.shift()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/shift)
```js
const array1 = [1, 2, 3];

const firstElement = array1.shift();

console.log(array1);
// expected output: Array [2, 3]

console.log(firstElement);
// expected output: 1
```

만약 요소가 없는 빈 배열일때는 `undefined`를 리턴한다.


**Array.prototype.unshift()**

`unshift()` 메서드는 새로운 요소를 배열의 맨 앞쪽에 추가하고, 새로운 길이를 반환한다.

```js

const array1 = [1, 2, 3];

console.log(array1.unshift(4, 5));
// expected output: 5

console.log(array1);
// expected output: Array [4, 5, 1, 2, 3]


```

**unshift & pop**
unshift와 pop을 활용하면 array를 자료구조 `Queue` 처럼 사용하는것이 가능하다.
`Queue`의 특징인 먼저 들어온 요소가 먼저 빠져나간다는 특성을 이용한것이다.



# 마무리

정적 언어와 비교하여 javascript의 array는 형식이 자유로울 뿐만아니라 크기도 원하는만큼 변경이 가능하다. 어떤 측면에서는 자유로움을 더해주지만 반대로 생각하면 그만큼 형식에 대해 깐깐하게 검사해야하고 자칫 잘못하면 성능이 안나오는 문제가 있다. 그만큼 프로그래머에게 신중함을 요구한다고 보인다.


