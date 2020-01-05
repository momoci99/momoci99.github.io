---
layout: single
title:  "예외처리 이야기"
tag : 
    - Exception
    
---

# 들어가며...

어느 소프트웨어던 예외-Exception 상황은 존재하기 마련이다. 프로그래머가 미처 예상하지 못하여 대비 못한 예외가 발생한다면 그때부터 골치아픈 일이 생긴다. 프로그램이 그냥 죽던가, 전혀 예상치 못한 동작을 한다던지, 멈춘채로 자원을 점유만 하고있다던지 등등.. 정말 끔찍한 일들이다. 누가 말하길 실제 소프트웨어 개발에 30%의 시간을 쓰고 나머지 70%에 예외처리하는데 보낸다고 할정도로 중요하다.

이번에는 예외처리를 정리하면서 평소 생각만 해둔 내용과 명확하지 않은 부분을 해소하고자 정리를 해보기로 하였다.

예제코드는 대부분 [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/try...catch#Nested_try-blocks)에서 가지고 왔다.

## 동기식 코드내에서의 기본적인 try~catch


    try {
        try_statements
    }
    [catch (exception_var_1) { 
        catch_statements_1
    }]
    ...
    [catch (exception_var_2) {
        catch_statements_2
    }]
    [finally {
        finally_statements
    }]

위에는 가장 기본적인 javascript 의예외처리 구문이다. `동기식`으로 명시한 이유는 비동기 코드는 try~catch 로 예외처리가 힘들고 다른 방법을 사용해야하기 때문이다.

### try
코드가 실행될 블럭이다. 만약 try 블럭 내에서 예외가 발생한다면 catch 블럭에서 예외를 처리할 코드가 실행된다. 


### catch 
try 블럭에서 예외가 발생하였을때 실행될 코드가 위치한다.

만약 예외가 발생하지 않는다면 catch문은 실행되지 않는다.

### finally
이 블럭의 코드는 try 블럭 실행 이후나 catch 블럭 실행 이후  **반드시** 실행되는 코드이다. 보통 이 영역에는 할당받았거나 점유한 자원을 해제시키는 코드가 위치한다.

### throw

사용자 정의 예외를 말 그대로 `던지는` 코드이며 throw문 이후의 코드 제어는 call stack 내의 첫번째 catch 블록으로 전달되며 만약, catch 블록이 없으면 프로그램은 종료된다.


예외처리문의 단어는 직관적인데 
뭔가를 시도하고(try) 에러가 발생하여 이걸 누군가 처리하길 기원하면서 던지고(throw) 던진 에러를 받아(catch) 처리한다. 마무리를 위해 finally(끝) 구문이 실행된다.


## try catch의 간단한 예제
```js

//javascript
normalException = () => {

    try {
        throw "myException";
        // 에러발생
    }
    catch (e) {
       
        console.log(e); 
        // 에러 내용 확인
    }

};

normalException(); 

//print
//myException
```

위 코드는 간단한 try~catch문 예제이다. 여기서부터 조금씩 심회시켜 나가보자

### catch문의 식별자

위 예제에서 본  catach문에 
```js
cathc(e)
```
`e` 라는 변수가 있다. 이 변수는 try 블럭에서 throw 된 상태가 저장되어있다. 이 변수를 통해 어떤 에러가 발생했는지 알수있다.

단 catach 블럭은 local이기 때문에 변수는 catch문 블럭을 벗어나면 더이상 사용할 수 없다. 아래 예제를 보자.

```js
//javascript
normalException = () => {

    try {
        throw "myException";
        // 에러발생
    }
    catch (e) {
       
        console.log(e); 
        // 에러 내용 확인
    }
    console.log(e);
};

normalException(); 


//print
//myException
//ReferenceError: e is not defined

```
### catch문의 식별자를 이용한 예제


```js
try {
    myroutine(); // 세가지 유형의 예외가 발생할 수 있는 함수
} catch (e) {
    if (e instanceof TypeError) {
        // TypeError 예외를 처리할 수 있는 선언문
    } else if (e instanceof RangeError) {
        // RangeError 예외를 처리할 수 있는 선언문
    } else if (e instanceof EvalError) {
        // EvalError 예외를 처리할 수 있는 선언문
    } else {
       // 명시되지 않은 그 외 모든 예외를 처리할 수 있는 선언문

       logMyErrors(e); //예외 객체를 커스텀 예외처리 함수에 전달
    }
}

```
`instanceof `와if else 문으로 적절하게 상황에 맞게 에러 처리가 가능해진다.

### 중첩된 try catch문1

먼저 중첩된 try catch문에 대해 설명하기전에 이 pattern은 결코 바람직하지 않다는것을 말하고 싶다. 왜냐하면..

- 코드가 지저분해진다.
- 제어의 흐름을 파악하기 힘들다.

따라서 최대한 사용을 자제하는게 바람직하지만 어쩔수 없이 사용하는 경우가 있으니 알아보도록 하자.

```js
try {
  try {
    throw new Error("oops");
  }
  finally {
    console.log("finally");
  }
}
catch (ex) {
  console.error("outer", ex.message);
}

// Output:
// "finally"
// "outer" "oops"

```

우선 내부 try-finally 구문의 실행이 종료되고 나서 그다음 외부의 catch문이 실행된다.

언뜻보면 제어 흐름이

- 내부 try 
- 외부 catch
- 내부 finally

라고 생각해볼수도 있지만 try-catch-finally는 각각 연결되어있는 하나의 덩어리로 보는게 맞다.

### 중첩된 try catch문 2

이번에는 내부에 catch문이 추가되었다. throw가 발생했을때 가장 가까운 catch문에서 이를처리하므로 외부의 catch문이 동작하지 않았다.

```js
try {
  try {
    throw new Error("oops");
  }
  catch (ex) {
    console.error("inner", ex.message);
  }
  finally {
    console.log("finally");
  }
}
catch (ex) {
  console.error("outer", ex.message);
}

// Output:
// "inner" "oops"
// "finally"

```

### 중첩된 try catch문 3

이번에는 내부의 catch문에서 다시 `throw`하는 예제이다. 내부의 catch문에서 다시 예외를 throw 하므로 외부의 catch문에서 이를 처리하게 된다.

```js
try {
  try {
    throw new Error("oops");
  }
  catch (ex) {
    console.error("inner", ex.message);
    throw ex;
  }
  finally {
    console.log("finally");
  }
}
catch (ex) {
  console.error("outer", ex.message);
}

// Output:
// "inner" "oops"
// "finally"
// "outer" "oops"

```

### 예외처리와 return
try catch finally 내부에 return문을 넣으면 어떻게 될까? 물론 일반적인 경우는 아니지만 궁금하니 정리해보자.



#### try내부의 throw 이후 return문
이 경우는 좀 쉬운 경우이다. throw문 이후 동일 블럭내의 코드는 모두 무시되므로 `return 1`은 무시된다.
```js
example = () => {

    try {
        throw "myException";
        return 1;
    }
    catch (e) {
        console.log(e);

    } finally {
        console.log("finally");
    }

    console.log("end of function");

};

console.log(example()); 

//output
//myException
//finally
//end of function
//undefined
```

#### catch내부의 return문과 finally

이 경우는 좀 주의해서 봐야하는 경우이다. catch 블럭내에 return이 존재하는 경우가 일반적인게 아니라는걸 인지하고 보는게 좋겠다.


```js
example = () => {

    try {
        throw "myException";
        return 1;
    }
    catch (e) {
        console.log(e);
        return 2;

    } finally {
        console.log("finally");
    }

    console.log("end of function");

};

console.log(example()); 

//Output
//myException
//finally
//2
```
catch블럭내의 return 때문에 finally가 무시될것같지만 finally는 `반드시` 실행된다고 약속되어있기에 실행된다.

주목할점은 catch내의 return문이 먼저 위치해 있었음에도 불구하고 `finally`가 실행 된 이후 return된것이다.

#### catch내부의 return문과 finally return문 다 있을때

이런 케이스가 없을것같지만 궁금하니 확인해보자

```js
normalException = () => {

    try {
        throw "myException";
        return 1;
        // 에러발생
    }
    catch (e) {
       
        console.log(e); 
        return 2;

    } finally{
        console.log("finally");
        return 3;
    }

};

//Output
//myException
//finally
//3
```
주목할 사항은 return되는 값이 2를 예상했지만 실제로는 3이 return되는것이다. 이는 `finally`블럭이 `무조건` 실행되기 때문에 catch블럭의 return문을 덮어버린것이다.

### 진짜 하면 안좋은 예

사실 위에서 말한 예제는 그럴수도 있겠다 싶지만 아래에서 다루는 예제는 정말 나쁜 패턴이니 정리해두고 따라하지 말자.


#### catch 문이 비어있을때

```js
try {
    //...예외가 발생할 수 있는 코드
}  
catch (e) {

}
```

이렇게 되면 예외가 발생했을 때 catch블럭이 비어 아무런 후속 조치 없이 그냥 로직이 끝나버린다. 저렇게 되면 try catch구문을 사용하는 의미가 퇴색되며 finally가 없는이상 catch문의 블럭을 빈 상태로 놔두지 말자.


## 마무리

여기까지 javascript의 동기식 예외처리 방법과 관련 이야기를 했는데 다음에는 비동기식 예외처리에 대해 정리하고싶다. 비동기 코드의 예외처리는 좀더 연구가 필요하다.

