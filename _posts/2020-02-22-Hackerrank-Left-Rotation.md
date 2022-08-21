---
layout: single
title: "Hackerrank Left Rotation"
tag:
  - Array
  - Hackerrank
---

# Left Rotation

[문제 링크](https://www.hackerrank.com/challenges/array-left-rotation/problem)

## 설명

배열 요소를 주어진 만큼 왼쪽으로 회전시키면 된다. javascript `Array`의 내장함수로 쉽게 해결 할 수 있다.

## 코드

```js
"use strict";

process.stdin.resume();
process.stdin.setEncoding("utf-8");

let inputString = "";
let currentLine = 0;

process.stdin.on("data", (inputStdin) => {
  inputString += inputStdin;
});

process.stdin.on("end", (_) => {
  inputString = inputString
    .replace(/\s*$/, "")
    .split("\n")
    .map((str) => str.replace(/\s*$/, ""));

  main();
});

function readLine() {
  return inputString[currentLine++];
}

function rotation(arr, d) {
  let rotated = arr;

  for (let i = 0; i < d; i++) {
    rotated.push(rotated.shift());
  }

  return rotated;
}

function main() {
  const nd = readLine().split(" ");

  const n = parseInt(nd[0], 10);

  const d = parseInt(nd[1], 10);

  const a = readLine()
    .split(" ")
    .map((aTemp) => parseInt(aTemp, 10));

  const result = rotation(a, d);

  console.log(result.join(" "));
}
```
