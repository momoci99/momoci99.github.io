---
layout: single
title:  "Hackerrank Queue using Two Stacks"
tag : 
    - Queue
    - Hackerrank
---

# Queue using Two Stacks
[문제 링크](https://www.hackerrank.com/challenges/queue-using-two-stacks/problem?utm_campaign=challenge-recommendation&utm_medium=email&utm_source=24-hour-campaign)

# 설명
위 문제는 2개의 stack을 이용하여 큐를 구현하는 문제이다. stack의 특성상 요소를 모두 집어넣고 빼면 순서가 반대로 뒤집힌다는 특성을 이용하여  1번 stack의 모든 요소를 2번stack에 집어넣고 queue의 삭제 혹은 맨 앞 요소 조회(peek)연산후 다시 2번 -> 1번 순으로 넣어주면 된다.

다만 위 방법을 실제로 구현하면 성능이 나오지 않으므로 개선된 방법으로 구현해야한다.

- 개선된 방법

stack1에는 새 원소가 맨 위에 있는 스택으로 활용하고, stack2에는 새 원소가 맨 아래에 있는 스택으로 활용한다. 큐에서 제거할때는 오래된 원소부터 제거해야하므로 stack2에서 원소를 제거한다. stack2가 비어 있을때에는 stack1의 내용을 stack2로 모두 넣는다. 새로운 원소를 삽입할때는 stack1에 삽입하여 새로운 원소가 스택 최상단에 유지되도록 한다.


# 코드

```js
function processData(input) {

    let line = input.split("\n");
    
    let stackNew = []; //stack1
    let stackOld = []; //stack2

    line.forEach(element => {
        let splited = element.split(" ");
        let query = splited[0];

        if (query == 1) {
            stackNew.push(splited[1]);
        } else if (query == 2) {

            if(stackOld.length === 0){
                while(stackNew.length !== 0){
                    stackOld.push(stackNew.pop());
                }
            }
            
            stackOld.pop();

        } else if (query == 3) {

            if(stackOld.length === 0){
                while(stackNew.length !== 0){
                    stackOld.push(stackNew.pop());
                }
            }

            console.log(stackOld[stackOld.length-1]);
        }
    });
}

```