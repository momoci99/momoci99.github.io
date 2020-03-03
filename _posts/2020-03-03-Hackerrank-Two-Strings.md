---
layout: single
title:  "Hackerrank Two Strings"
tag : 
    - Dictionaries and Hashmaps
    - Hackerrank
---


# Two Strings
[문제링크](https://www.hackerrank.com/challenges/two-strings/problem?h_l=interview&playlist_slugs%5B%5D=interview-preparation-kit&playlist_slugs%5B%5D=dictionaries-hashmaps)

# 설명
주어진 두 문자열을 비교하여 서로 일치하는 **단일 문자** 가 있다면 "YES", 그렇지 않다면 "NO"를 리턴하면 되는 문제이다.

푸는 방법은 여러 방법이 있지만 나는 js의 `Set`을 사용하여 교집합을 구하는것으로 풀었다.

두 문자열의 집합을 구하여 교집합이 하나라도 있다면 "YES"를 리턴하고 그렇지 않다면 "NO"를 리턴하게 하였다.

# 코드

```js
Set.prototype.intersection = function(setB) {
    var intersection = new Set();
    for (var elem of setB) {
        if (this.has(elem)) {
            intersection.add(elem);
        }
    }
    return intersection;
}

//using set
function twoStrings(s1, s2) {

    let splited1 = s1.split("");
    let splited2 = s2.split("");

    let set1 = new Set(splited1);
    let set2 = new Set(splited2);

    let result = set1.intersection(set2);

    if(result.size > 0){
        return "YES";
    } else {
        return "NO";
    }
    
}

```