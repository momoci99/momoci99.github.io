---
layout: single
title:  "[npm] npm package 취약점(vulnerabilities) 해결"
tag : 
    - npm
    - audit
---

# 개요

토이 프로젝트 진행중 사용하지 않는 모듈을 정리하기 위해 `npm install`을 진행중 취약점 고나련 메시지가 출력되는것을 확인하였다. 해결하는데 큰 노력은 들지 않았지만 과정을 기록하기 위해 글을 남긴다.

# 해결과정

![npm_audit](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/dev_review_songi_stock/npm_audit.png)

해당 메시지를 구글링 해보니 내가 사용하고 있는 package중 취약점 문제가 있으니 이를 해결하라는 
일종의 경고문이였다.


우선 어떤 package에서 취약점이 발견되었는지 확인 한 결과

![npm_audit2](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/dev_review_songi_stock/npm_audit2.png)

- mem
- braces

이 두 패키지에서 각각 `DoS`, `Regular Expression Dos` 공격 취약점을 확인할 수 있었다. 이를 해결하기 위해 `audit fix` 를 실행하였고 이걸로 잘 해결될줄 알았으나 braces 패키지가 계속 취약점 문제가 있다는 메시지가 출력되었다.

![hmm](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/Etc/hmm.jpg)

~~Hmmmmmm..... 왜?~~

혹시나 braces 패키지를 참조하고 있는 babel-cli 패키지를 `update` 하면 해결되지 않을까 하여 업데이트를 실행하였더니 이번에는 에러 메시지를 확인 할 수 있었다.

> Refusing to delete node-gyp.cmd

처음에는 권한 문제인줄 알고 관리자 권한으로 시도했으나 똑같은 결과가 되풀이되었다. 이를 해결하기위해 또 구글링하여 아래의 명쾌한 해답을 얻었다.

- [npm-err-refusing-to-delete-code-eexist](https://stackoverflow.com/questions/46541371/npm-err-refusing-to-delete-code-eexist/46541654#46541654)


그렇다 밀어버리고 새로 설치하면 되는것이다. `npm install` 이 완료되었다. 그리고 위에서 진행한 과정을 다시 반복하였다.

- npm update
- audit fix

문제가 해결되었을까?.. 아니다 문제는 해결되지 않았다. 그렇다면 문제에 대한 접근 방식이 잘못되었다. `brace` 패키지를 참조하는 `babel-cli`에 대해 구글링을 진행해보니 나와 같은 문제가 있는 깃허브 이슈를 발견하였다.

- [babel/babel/issues/9578](https://github.com/babel/babel/issues/9578#issuecomment-466922413)

comment를 보면 Babel7에서는 분명 해결되었다고 하는데 나는 babel 7을 이미 사용하고 있었던 것이다. 뭔가 이상하여 `package.json`를 확인해보니 아차 싶었다.

- "babel-cli": "^6.26.0",
- ...
- "@babel/cli": "^7.5.5",

`babel-cli`와 `@babel/cli`가 함께 `package.json`에 있었던것이다. 이러니 업데이트가 안되는 6버전의 취약점이 계속 나타나는것이였다.

결국 `"babel-cli": "^6.26.0"`를 `package.json` 에서 제거하고 npm 패키지를 모두 재설치한 후 아래의 메시지를 볼수있었다.

![npm_audit3](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/dev_review_songi_stock/npm_audit3.png)



# 마무리

좀 허탈하게 해결한 감이 없지 않지만 덕분에 `package.json` 관리의 중요성 및 `audit`, `audit fix` 기능을 활용해볼 수 있었던 좋은 기회였다. 덤으로 사용하지 않는 package 를 찾기위해 [npm depcheck](https://www.npmjs.com/package/depcheck)도 알게되었다.
