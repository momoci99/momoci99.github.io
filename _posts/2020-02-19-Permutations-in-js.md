---
layout: single
title:  "순열에 대하여"
tag : 
    - Algorithm
    - Permutation
---


# 참고사항

이 글은 [Implement All Permutations of a Set in JavaScript-Michael Mitrakos
](https://initjs.org/all-permutations-of-a-set-f1be174c79f8)
를 참고하고 번역한 글입니다.
(주의! 구글 번역기를 주로 사용하였으며 의역, 직역, 저의 주관이 함께 섞여있습니다. 정확한 의미는 링크를 타고 원문을 참고하여주시면 감사하겠습니다.)

## 들어가며
순열 관련 글을 보다가 우연히 간단하게 잘 정리된 글을 보게되어 번역하고 정리하려고 한다.


이 기사는 이 기사는 매우 일반적인 인터뷰 질문인 집합의 모든 순열을 다루고 있습니다.
‘주어진 문자열의 모든 순열이나 순서를 얻는 함수를 구현합니다.'
예를 들어 'abc'라는 입력이 있을때, 출력은 [‘abc’, ‘acb’, ‘cab’, ‘cba’, ‘bca’, ‘bac’] 이어야 합니다.

## 분석

많은 사람들이 순열이라는 단어를 듣고 두려워합니다. 그래서 먼저 순열이 실제로 무엇인지 다루겠습니다.

> 순열은 집합의 모든 멤버를 [sequence](https://en.wikipedia.org/wiki/Sequence) 또는 [order](https://en.wikipedia.org/wiki/List_of_order_structures_in_mathematics)로 정렬하는 작업과 관련이 있습니다.

쉽네요! 글쎄.. 아직은 아니지만, 우리는 최소한 마지막에 무엇을 리턴해야하는지 알고있습니다. 즉 'abc'가 주어졌을때 결과는 다음과 같아야 합니다.

>['abc', 'acb', 'bac', 'bca', 'cab', 'cba']

먼저 이 문제가 자연스럽게 해결 될 수있는 방법을 살펴 보겠습니다. 'abc'가 주어지면 자연스럽게 첫 글자를 분리하고 다음 두 글자의 모든 순열을 찾습니다. 따라서“abc”의 경우 :

    a bc
    a cb
    b ac
    b ca
    c ab
    c ba

이 경우 테스트 케이스에서 문자열이 얼마나 큰지 알지 못하기 때문에 마음이 재귀로 바로 넘어갑니다. 따라서 문자열의 모든 문자에 대해 문자열의 문자 하나를 잡고 해당 문자가없는 모든 순열 앞에 붙일 수 있습니다.

재귀에서 재귀 호출을 중지하려면 '기본 사례'가 필요합니다. 이 경우 문자열이 단일 문자 일 때 문자를 반환합니다.

    permutations(abc) = a +     permutations(bc)     +
                        b +     permutations    (ac) +
                        c +     permutations    (ab)

    permutations(ab) =  a +     permutations(b)      +
                        b +     permutations (a)

    permutations(a) =   a

## 의사 코드

    function getAllPermutations (string)
      define results
      if string is a single character
        add the character to results
        return results
      for each char in string
        define innerPermutations as a char of string
        set innerPermutations to getAllPermutations (without next char)
        foreach string in innerPermutations
          add defined char and innerPermutations char
      return results

    function getAllPermutations (string)
      결과를 정의함
      if 문자열이 단일 문자라면
        단일 문자를 결과에 추가함.
        return 결과

      반복문 : 문자열내의 문자를 순회
        문자열의 문자를 내부순열로 정의
        모든 순열을 내부 순열로 정의(다음 문자없이)
        반복문 : 내부순열 내의 문자열을 순회
          add defined char and innerPermutations char
          결과에 정의된 문자와 내부순열의 문자를 더해 추가함
      return 결과


## 복잡도
위 알고리즘의 시간 및 공간 복잡도는 생성 된 항목 수와 같아야합니다. 크기 n의 집합에 대한 고유 순열의 수는 n!이므로 알고리즘의 시간 및 공간 복잡도는 O (n!)입니다.



```js

//주석
function getAllPermutations(string) {
  var results = [];

  if (string.length === 1) {
    results.push(string);
    return results;
  }

  for (var i = 0; i < string.length; i++) {
    var firstChar = string[i];
    var charsLeft = string.substring(0, i) + string.substring(i + 1);
    var innerPermutations = getAllPermutations(charsLeft);
    for (var j = 0; j < innerPermutations.length; j++) {
      results.push(firstChar + innerPermutations[j]);
    }
  }
  return results;
}
```

---

여기까지가 원문에 있는 내용이다.

## 기타

C++ 에는 [next_permutation](http://www.cplusplus.com/reference/algorithm/next_permutation/), [prev_permutation](http://www.cplusplus.com/reference/algorithm/prev_permutation/) 이 존재한다. 이 함수들은 지정된 범위에 따라 각각 다음순열, 이전순열을 구하는 함수이다. 혹시 관심이 있다면 찾아보자.

