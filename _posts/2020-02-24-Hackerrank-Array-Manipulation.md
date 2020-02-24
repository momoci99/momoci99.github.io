---
layout: single
title:  "Hackerrank Array Manipulation"
tag : 
    - Array
    - Hackerrank
---


# Array Manipulation

[문제 링크](https://www.hackerrank.com/challenges/crush/problem)

## 설명

주어진 입력에 따라 일정 범위의 배열에 k 값을 더하면 된다. 하지만 이 문제는 시간 복잡도를 최적화해야하는 문제가 있다. 시간 복잡도가 n 을 초과하면 실패하므로 단순히 코딩만 해서는 통과할 수 없다.

### 첫번째 시도
문제의 설명을 그대로 따라 작성한 코드이다. 

- n 크기의 배열 생성
- querires를 순회하면서 위에서 생성한 배열의 a 부터 b까지 k 값을 더한다.

결과는 시간 초과로 실패하였다.

```js
function arrayManipulation(n, queries) {
    
    let array = Array(n).fill(0);
    let queriesLength = queries.length;

    for(let i = 0; i< queriesLength; i++){
        let a = queries[i][0];
        let b = queries[i][1];
        let k = queries[i][2];

        for(let j = a - 1; j<= b -1; j++){
            array[j] = array[j] + k;
        }

    }

    return Math.max(...array);

}

```

도저히 답이 나오지 않아 구글링한 결과, 문제를 다른 방법으로 접근하는 내용을 찾을 수 있었다.

[Logic used behind Array Manipulation of HackerRank
](https://stackoverflow.com/a/48162313/3836836) 


- a 에서 b 까지 k 만큼 증가시키지만 그 다음 요소는 증가하지 않음.
- 이걸 다르게 해석하면 b 다음 요소는 k 만큼 빼고 있다고 볼 수 있음.
- 이 증감을 통해 최종 값을 저장 할 수 있음.
- 마지막으로 왼쪽에서 오른쪽 방향으로 모든 요소를 더함. 이는 한 요소가 이전 요소보다 얼마나 큰지를 저장하는것임.

example

배열의 크기가 5 이고 첫번재 쿼리가 1 3 3 일때.

원래 방식
> 3 3 3 0 0

다른 관점의 방식
> 3 0 0 -3 0

이는 아래와 같은 의미를 가지고 있다.

    첫번째 element    is 3 은 0보다 크다.
    두번째      ->       0 은 인덱스 1의 요소 보다 크다.
    세번째      ->       0 은 인덱스 2의 요소 보다 크다.
    네번째      ->      -3 은 인덱스 3의 요소 보다 크다.
    다섯번째    ->       0 은 인덱스 4의 요소 보다 크다.

두번째 쿼리가 2 4 4 일때

원래 방식
> 3 7 7 4 0

다른 관점의 방식
> 3 4 0 -3 -4


마지막으로

    0+(3) 0+3+(4) 0+3+4+(0) 0+3+4+0+(-3) 0+3+4+0-3+(-4)

    3     7       7         4            0  

이전 방식의 연산과 결과가 동일하다.

이 다른 방식의 결과를 코드로 풀이하면 아래와 같다.


## 코드

```js

function arrayManipulation(n, queries) {
    //n보다 하나 더 큰 배열을 생성한다.
    let array = Array(n+1).fill(0);
    let arrayLength = array.length;
    let queriesLength = queries.length;
    let maxValue = Number.MIN_SAFE_INTEGER;

    for (let i = 0; i < queriesLength; i++) {
        let a = queries[i][0] - 1;
        let b = queries[i][1];
        let k = queries[i][2];

        array[a] = array[a] + k;
        array[b] = array[b] + (k * -1);
      
    }

    let temp = 0;
    //배열 요소의 합을 구하면서
    //최대값을 찾는다.
    for(let j = 0; j<arrayLength; j++){
        temp = temp + array[j];
        
        if(maxValue < temp){
            maxValue = temp;
        }
    }


    return maxValue;

}

```
