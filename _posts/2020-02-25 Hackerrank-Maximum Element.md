---
layout: single
title:  "Hackerrank Array Manipulation"
tag : 
    - Array
    - Hackerrank
---

# Maximum Element

[문제 링크](https://www.hackerrank.com/challenges/maximum-element/problem?utm_campaign=challenge-recommendation&utm_medium=email&utm_source=24-hour-campaign)

# 설명

처음에 문제 설명대로 코딩을 하게되면 바로 time out으로 통과할 수 없다. 특히 javascript의 array는 `push`, `pop`연산으로 stack처럼 활용할 수 있기때문에 무턱대로 array를 사용해선 안된다. 나또한 그냥 array를 사용하였다가 timeout 으로 인해 실패하였다.

따라서 array를 사용하되 `push`, `pop` 연산을 하지 않는 방법을 사용하여 문제를 해결하였다. `Math.max()`도 사용하지 않았다. 문제 내용은 매우 간단하지만 성능을 항상 고려하는 자세를 잃지 말아야겠다.

# 코드

```js

function processData(input) {
    
    let splitByNewLine = input.split("\n");
 
    let queryLength = splitByNewLine.length;
    let stack = Array(queryLength).fill(-1);

    //array의 내장함수 push, pop 대신 top을 활용하여
    //자체 push, pop 연산을 한다.
    let top = -1;

    for(var i = 0; i<queryLength; i++){
        let splited = splitByNewLine[i].split(" ");
        
        let query = splited[0];
  
        if(query == 1){
            //push 연산
            top++;
            stack[top] = Number.parseInt(splited[1]); 

        } else if (query == 2){

            //pop 연산
            if(top > -1){
                stack[top] = -1;
                top--;
            }

        } else if (query == 3){
            let max = -2;
            
            //top 까지만 비교하여 최대값을 찾는다.
            for(let j = 0; j<= top+1; j++){
                if(max < stack[j]){
                    max = stack[j];
                }
            }
            console.log(max);
        }
    }       
} 
```