---
layout: single
title: "리팩토링 - 코드에서 나는 악취를 찾아서"
tag:
  - Refactoring
---


# 개요

저는 일하기 귀찮을때 가끔씩 제가 개발했던 코드를 한번씩 훑어보곤 합니다. 그때마다 악취가 올라오는 코드를 발견하면 지체없이 리팩터링 2판을 끼고 이 부분을 어떻게 개선하면 좋을 지 즐거운(?) 고민을 하곤 합니다. 

이번 글에는 리팩터링2판 내용중 냄새나는 코드의 특징 및 연관된 내용을 담았습니다.

```jsx
냄새나면 갈아라 - 켄트 백 할머니의 육아 원칙
```

# 기이한 이름 (Mysterious Name)

- 코드는 항상 단순하고 명료해야합니다.
- 함수, 모듈, 변수, 클래스등 이름만 보고도 용도를 파악할 수 있어야 합니다.
- 물론, 이름 짓기는 프로그래밍에서 가장 어려운 일 중 하나입니다.

![meme.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/Refactoring-With-Smell/meme.png)

정말.. 어렵습니다😩

**Example - meme** 

```jsx
const zoneBer = (time) => {
	//time만큼 실행을 대기하는 함수
}
```

'존버' 라는 밈을 모르면 알 수 없습니다. 재미는 있겠네요.

**Example - 너무 추상적**

```jsx
const processPermission = (user) => {
    //user의 권한을 확인
		//user의 권한이 없으면 권한을 요청함.
    //user의 권한이 있으면 true 반환.
}
```

process라는 말로는 어떤 행위를 하는지 명확하지 않습니다.

**Example - 모호함**

```jsx
const zoneIn = (user) => {
	//user를 특정 구역에 진입처리 로직
}
```

`zoomIn` 으로 착각할 수 있습니다.

### 어떻게 고치나요?

- 함수 선언 바꾸기
- 필드 선언 바꾸기
- 변수 선언 바꾸기

# 중복 코드 (Duplicated Code)

- 똑같은 코드 구조가 반복되면 통합 할 수 있습니다.
- 중복이 많은 코드는 수정하기가 어렵습니다. 각각 비교 해야하니까요.

**Example - 비슷한 패턴이 반복**

```jsx
var findManager = function(){
  employees.forEach(function(person){
    if (person.type == 'Manager') {
      console.log(person.name);
    }
  })
}
var findCleaner = function(){
  employees.forEach(function(person){
    if (person.type == 'Cleaner') {
      console.log(person.name);
    }
  })
}
var findDeveloper = function(){
  employees.forEach(function(person){
    if (person.type == 'Developer') {
      console.log(person.name);
    }
  })
}
```

**Example - 동일한 코드가 반복**

```jsx
const array_a = [];
const array_b = [];
 
let sum_a = 0;

for (let i = 0; i < 4; i++)
   sum_a += array_a[i];

let average_a = sum_a / 4;
 
let sum_b = 0;

for (let j = 0; j < 4; i++)
   sum_b += array_b[j];

let average_b = sum_b / 4;
```

### 어떻게 고치나요?

- 함수 추출하기
- 문장 슬라이드하기
- 메서드 올리기

# 긴 함수 (Long Function)

- 긴 함수는 한번에 이해하기 어렵습니다.
- Example - 정신이 혼미해질 정도로 긴 함수(JAVA)

    ![long_java_1.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/Refactoring-With-Smell/long_java_1.png)

    ![long_java_2.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/Refactoring-With-Smell/long_java_2.png)

### 어떻게 고치나요?

- 함수 추출하기
- 임시변수를 질의 함수로 바꾸기
- 매개변수 객체 만들기
- 객체 통째로 넘기기
- 함수를 명령으로 바꾸기
- 조건문 분해하기
- 조건문 다형성으로 바꾸기
- 반복문 쪼개기

# 긴 매개변수 목록 (Long Parameter List)

- 매개변수가 길어지면 그 자체로 이해하기 어렵습니다.

**Example - 긴 매개 변수**

```jsx
const createUser = (name, address, hobby, income, car, length, mailAddress, weight, friendLength, job, assets) => {
	//User 객체를 생성하는 함수
}
```

### 어떻게 고치나요?

- 매개변수를 질의 함수로 바꾸기
- 객체 통째로 넘기기
- 매개변수 객체로 만들기
- 플래그 인수 제거하기
- 여러 함수를 클래스로 묶기

# 전역 데이터 (Global Data)

- 쓰지마세요
- 쓰면 벌받습니다. 특히 javascript에서는요!
- 전역 변수가 많아지면 감당할 수 없습니다.

**Example - const, let, var 없는 전역 변수 사용 😡**

```jsx
globalFlag = true //const, let 심지어 var도 없는 끔찍한 전역 변수

if(globalFlag){
	//do bad thing
} else {
	//do hell thing
}
```

### 어떻게 고치나요?

- 변수 캡슐화하기

# 가변 데이터 (Mutable Data)

- 데이터가 변경되면 예상치 못한 결과나 버그로 이어질 수 있습니다.

**Example - 변수가 계속 갱신되어 의미가 바뀌는 경우**

```jsx
let temp = 2 * (height + width); 
console.log(temp)

temp = height * width;
console.log(temp)
```

### 어떻게 고치나요?

- 변수 캡슐화하기
- 변수 쪼개기
- 문장 슬라이드하기
- 함수 추출하기
- 질의 함수와 변경 함수 분리하기
- setter 제거하기
- 패상 변수를 질의 함수로 바꾸기
- 여러 함수를 클래스로 묶기
- 여러 함수를 변환 함수로 묶기
- 참조를 값으로 바꾸기

# 뒤엉킨 변경 (Divergent Change)

- 여러 기능을 추가 할때 하나의 모듈이 계속 변경되는경우를 말합니다.
- 단일 책임 원칙이 지켜지지 않고, 각각 독립된 코드로 분리되어야하나 그렇지 못한경우 발생합니다.

### 어떻게 고치나요?

- 단계 쪼개기
- 함수 옮기기
- 함수 추출하기
- 클래스 추출하기

# 산탄총 수술 (Shotgun Surgery)

- 뒤엉킨 변경과 비슷하지만 정 반대의 개념입니다.
- 하나의 기능을 변경하기 위해 소스코드의 여러 부분을 수정해야하는 경우를 말합니다.

### 어떻게 고치나요?

- 함수 옮기기
- 필드 옮기기
- 여러 함수를 클래스로 묶기
- 여러 함수를 변환 함수로 묶기
- 단계 쪼개기
- 함수 인라인하기
- 클래스 인라인하기

# 기능 편애 (Feature Envy)

- 메소드가 자신이 속한 클래스보다 다른 클래스에 더 깊게 관여되어 있는 경우입니다.
- 이로 인해 두 클래스의 의존성이 발생합니다.

### 어떻게 고치나요?

- 함수 옮기기
- 함수 추출하기

# 데이터 뭉치 (Data Clumps)

- 하나의 클래스에 너무 많은 필드가 존재하는 경우입니다.
- Example - 너무 많은 필드

    ```tsx
    class Animal {
      name: string;
    	addresss : string;
      color: string;
      food : string;
      sleeepingTime : string;
      age : number;
      roomNumber : number;
      manager : string;
      zoo : string;
      //그외 동물을 나타내는 간접적인 클래스 필드들..
    	//
    	//
    	//
    	//
    	//
    	//
    	//
    	//
    	//..

      constructor(theName: string) {
        this.name = theName;
      }
      move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
      }
    }
    ```

### 어떻게 고치나요?

- 클래스 추출하기
- 매개변수 객체 만들기
- 객체 통째로 넘기기

# 기본형 집착 (Primitive Obsession)

- 기본형에만 너무 집착하는 경우를 말합니다.
- 화폐단위, 물리단위, 시간 단위를 오직 string, number로만 처리하려는게 그 예입니다.
- 객체지향을 활용하여 세련된 코드로 작성 할 수 있습니다.

### 어떻게 고치나요?

- 기본형을 객체로 바꾸기
- 타입 코드를 서브 클래스로 바꾸기
- 조건부 로직을 다형성으로 바꾸기
- 클래스 추출하기
- 매개변수 객체 만들기

# 반복되는 switch문 (Repeated Switches)

- 모든 switch - case 문이 나쁜건 아닙니다.
- 계속 반복되는 switch - case 문은 변경을 어렵게 합니다.
- 객체지향의 다형성 or Key-Value 방식으로 개선 할 수 있습니다.

- Example -  긴 switch - case 문

    ```jsx
    const getFoodCategory = (food) => {
      let category = null
    	switch(food) {
    	case 'coke':
    		category = 'drink'
    	  break;
    	case 'pizza':
    	  category = 'food';
    	  break;
    	case 'orange':
    	case 'strawberry':
    	case 'apple':
    		category = 'fruit'
    		break;
    	case 'icecreame':
    		category = 'indulgence'
    		break;
    	case 'vegetable':
    		category = 'bean'
    		category = 'tomato'
    		break;
    	case 'bread':
    		category = 'bread'
    		break;
    	default:
    	  category = 'Unknown food!';
    	}
    }
    console.log(getFoodCategory('pizza'));
    ```

### 어떻게 고치나요?

- 조건부 로직을 다형성으로 바꾸기

# 반목문 (Loops)

- 단순 for, while loop는 파이프라인으로 처리 할 수 있습니다.
- reduce, map, filter 등등..

### 어떻게 고치나요?

- 반복문을 파이프라인으로 바꾸기

# 성의 없는 요소 (Lazy Element)

- 불필요한 클래스, 인터페이스, 메서드 사용을 한 경우입니다.

### 어떻게 고치나요?

- 함수 인라인하기
- 클래스 인라인하기
- 계층 합치기

# 추측성 일반화 (Speculative Generality)

- 언젠가 사용될 것이라고 추측하여 작성하였지만, 단 한번도 쓰이지 않는 경우를 말합니다.

### 어떻게 고치나요?

- 계층 합치기
- 함수 인라인하기
- 클래스 인라인하기
- 함수 선언 바꾸기
- 죽은 코드 제거하기

# 임시 필드 (Temporary Field)

- 코드 내에 특정한 상황에서만 의미가 부여되는 임시 필드가 존재하는 경우입니다.

### 어떻게 고치나요?

- 클래스 추출하기
- 함수 옮기기
- 특이 케이스 추가하기

# 메시지 체인 (Message Chains)

- 객체가 연쇄적으로 나열되는 경우를 말합니다.
- 메시지 체인의 중간 단계가 수정되면 이후 모든 단계에 영향을 미칩니다.

- Example

    ```jsx
    //memberName을 얻기 위해 총 3개의 객체에 접근하는 경우
    const memberName = aPerson.department.manager.name;
    ```

### 어떻게 고치나요?

- 위임 숨기기
- 함수 추출하기
- 함수 옮기기

# 중개자 (Middle Man)

- 현재 클래스가 가지고 있는 메서드 절반 이상이 다른 클래스의 메서드를 호출하는 경우를 말합니다.
- 즉, 지나치게 위임을 하는 경우를 말합니다.

### 어떻게 고치나요?

- 중개자 제거하기

# 내부자 거래 (Insider Trading)

- 자식클래스가 부모클래스에 대해 필요 이상으로 접근하려는 경우를 말합니다.

### 어떻게 고치나요?

- 함수 옮기기
- 필드 옮기기
- 위임 숨기기
- 서브클래스를 위임으로 바꾸기
- 슈퍼클래스를 위임으로 바꾸기

# 거대한 클래스 (Large Class)

- 클래스 하나가 너무 거대한 경우를 말합니다.

### 어떻게 고치나요?

- 클래스 추출하기
- 슈퍼클래스 추출하기
- 타입 코드를 서브클래스로 바꾸기

# 주석 (Comments)

- 주석은 나쁜게 아닙니다.
- 다만, 주석이 필요 없을 수준으로 코드를 작성하는게 더 좋습니다.

### 어떻게 고치나요?

- 함수 추출하기
- 함수 선언 바꾸기
- 어서션 추가하기

### 언급은 되었지만 좀더 생각해 봐야하는 냄새들

- 서로 다른 인터페이스의 대안 클래스들 (Alternative Classes with Different Interfaces)
- 데이터 클래스(Data Class)
- 상속 포기(Refused Badquest)

# 참고했던 글

좋은 이름, 나쁜 이름, 이상한 이름, NDC2018 - 전현규

[https://www.slideshare.net/devcatpublications/ndc2018](https://www.slideshare.net/devcatpublications/ndc2018)

Replacing switch statements with Object literals - by Todd Motto

[https://ultimatecourses.com/blog/deprecating-the-switch-statement-for-object-literals](https://ultimatecourses.com/blog/deprecating-the-switch-statement-for-object-literals)