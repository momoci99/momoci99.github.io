---
layout: single
title:  "Hackerrank Minimum Swaps 2"
tag : 
    - Array
    - Hackerrank
---


# Minimum Swaps 2

[문제링크](https://www.hackerrank.com/challenges/minimum-swaps-2/problem?h_r=profile)

# 설명

배열에 있는 요소들을 최소한의 swap 횟수로 오름차순 정렬하는 문제이다.

최초에는 퀵정렬, 삽입정렬을 고려하였으나 두 방법 모두 비교 횟수를 최소화 하는데에는 부적절한 방법이었다.

배열의 요소가 가지고 있는 특징(이번 문제는 중복없는 숫자)을 고려하지 않고 일반적인 방법으로 접근하였기에 시간이 더 소요된 감이 있다.

결국 구글링을 통해 문제를 해결 할 수 있었다.

[https://chickenpaella.tistory.com/83](https://chickenpaella.tistory.com/83)


이 문제는 아래의 특징을 가지고 접근해야한다.

- 배열의 모든 요소가 `정수`
- 오름차순 정렬
- 요소는 1 부터 시작

배열을 순회하는 인덱스 `i`에 위치한 값이 i+1 (정렬후 위치할 장소)가 아니면 본래 있어야할 위치와 교환하게 되면 교환횟수를 최소화하고 원하는 결과를 얻을 수 있다.

사족이지만 복잡하게 생각하지 않고 조금 가볍게 생각했다면 좀더 쉽게 풀렸을 문제였다고 생각된다.

# 코드

```js

function minimumSwaps(arr) {

    let swapCount = 0;

    for(let i = 0; i<arr.length; i++){
        while(arr[i] !== i+1){
            let temp = arr[i];
            arr[i] = arr[temp - 1];
            arr[temp-1] = temp;
            swapCount++;
        }
    }
    
    return swapCount;
}
```



