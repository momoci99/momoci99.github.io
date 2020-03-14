---
layout: single
title:  "Hackerrank Jesse and Cookies"
tag : 
    - Heap
    - Hackerrank
---


# Two Strings
[문제링크](https://www.hackerrank.com/challenges/jesse-and-cookies/problem)

# 설명

우선순위 큐를 활용하는 문제이다. 요소의 순위를 계속 유지해야하므로 이럴 때에는 우선순위 큐를 이용하자.


# 코드

```java
//PriorityQueue가 구현되어있는 java를 활용함
static int cookies(int k, int[] A) {

    int calCount = 0;
    PriorityQueue<Integer> cookies = new PriorityQueue<Integer>();

    for(int i = 0; i < A.length; i++){
        cookies.offer(A[i]);
    }

    //주어진 조건을 만족하여 연산이 필요없는경우
    if(cookies.peek() >= k)
    {
        return calCount;
    }

    while(cookies.size() > 1 && cookies.peek() < k){
        calCount++;

        int leastSweetCookie = cookies.poll();
        int secondSweetCookie = cookies.poll();
        int combinedCookie = 1 * leastSweetCookie + 2 * secondSweetCookie;

        cookies.offer(combinedCookie);

        if(cookies.peek() >= k){
            return calCount;
        }
    }
    return -1;
}

```