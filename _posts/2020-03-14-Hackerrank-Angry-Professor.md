---
layout: single
title:  "Hackerrank Angry Professor"
tag : 
    - Algorithms
    - Hackerrank
---


# Angry Professor
[문제링크](https://www.hackerrank.com/challenges/angry-professor/problem)

# 설명
문제 내용이 재미있어서 풀어본 문제이다. 배열 a 의 요소를 순회하면서
요소의 크기가 0 이하인 요소의 갯수를 파악하여 k 보다 크면 `NO` 를
k값 미만이면 `YES`를 리턴한다. 

# 코드

```js
function angryProfessor(k, a) {

    let joinedStudent = 0;

    for(let i = 0; i<a.length; i++){
        
        if(a[i] <= 0){
            joinedStudent++;
        }

        if(joinedStudent >= k){
            return "NO";
        }
    }

    return "YES";
    
}

```