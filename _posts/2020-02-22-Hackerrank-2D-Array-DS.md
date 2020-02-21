---
layout: single
title:  "Hackerrank 2D Array - DS"
tag : 
    - Array
    - Hackerrank
---

# Hackerrank 2D Array - DS

[문제 링크](https://www.hackerrank.com/challenges/2d-array/problem)

## 설명

그렇게 어렵지 않은 문제다. 모래시계 패턴을 찾아 합을 구하고 가장 큰 합을 찾으면 된다.


참고로 `result` 에는 최소값을 주어야 비교가 가능해진다.

## 코드
```js

function hourglassSum(arr) {


    let result = Number.MIN_SAFE_INTEGER;

    for (let i = 0; i < arr.length - 2; i++) {

        for (let j = 0; j < arr.length - 2; j++) {

            let tempSum = (arr[i][j] + arr[i][j + 1] + arr[i][j + 2] +
                arr[i + 1][j + 1] +
                arr[i + 2][j] + arr[i + 2][j + 1] + arr[i + 2][j + 2]);

            if (result < tempSum) {
                
                result = tempSum;
            }
        }

    }

    return result;
}

```

