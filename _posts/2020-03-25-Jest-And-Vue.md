---
layout: single
title:  "Jest And Vue"
---

# Jest And Vue

> NodeJS가 이미 설치되어 있고 npm 혹은 yarn을 쓸수 있는 환경이라는 가정하에 작성되었습니다.

## 개요
`Jest`는 페이스북에서 만든 javascript테스트 라이브러리입니다. 이 글을 쓰고있는 현재(2020-03-24) 30.2k의 github star 및 4.3k의 Fork횟수를 자랑하는 그야말로 아주 잘 나가는 Testing 라이브러리 입니다.

![사진1](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/vue-jest/framework_ranking.PNG?raw=true)

[작년 javascript 개발자 설문](https://2019.stateofjs.com/testing/#testing_experience_ranking)에서는 1위를 차지할 정도로 널리 알려져있습니다. 오늘은 이 Jest와 Jest를 사용하여 Vue.js의 컴포넌트를 테스트하는 과정을 다루려고 합니다.

## 테스트 코드가 필요한 이유
테스트 코드가 왜 필요할까요? 테스트 코드를 작성하는것은 매우 귀찮은 일이라고도 볼수있습니다. 개발자는 본래 자신이 작성해야하는 코드와 더불어 테스트코드까지 작성해야하니 일이 두배가 되는 꼴이니까요. 하지만! 테스트 코드를 작성하면 아래와 같은 이점이 생깁니다.

1. 끊임없이 발생할 버그에 대해 걱정하지 않고, 일이 언제 끝날지 알수 있게됩니다.

2. 제품의 안전성이 높아집니다. 고객의 만족도도 함께 높아집니다.

3. 코드에 대한 자신감이 생깁니다.

4. 전체적으로 보았을때 개발 시간을 감소시킵니다.

5. 행복해집니다.

이 외에도 여러 장점들이 있습니다. 심지어 테스트 주도 개발 이라는 책을 쓰신 켄트 백님은 작성하는동안 기분이 좋다고 까지하네요. 그러니 우리 모두 행복하기 위해 테스트 코드를 작성해봅시다.

## Vue CLI로 손쉽게 환경 구성하기

### 1.설치하기

Vue.js 는 Vue CLI를 통해 어렵지 않게 유닛 테스트를 진행하기 위한 옵션이 존재합니다. 이 옵션을 통해 프로젝트를 생성하게 되면 자동으로 기본적인 유닛 테스트를 위한 환경이 적용됩니다.

우선 Vue CLI 를 설치하기 위해 아래의 명령어를 입력합시다.

    npm install -g @vue/cli
    # OR
    yarn global add @vue/cli

설치가 완료되었으면 제대로 되었는지 확인해봅시다

    vue --version

혹시 window powershell 에서 vue 입력시 권한 에러가 발생한다면 아래와 같이 입력해봅시다.

    vue.cmd --version

Vue CLI가 정상적으로 설치되었다면 버전이 정상적으로 출력됩니다.

Vue에서 제공하는 [템플릿](https://github.com/vuejs-templates)을 사용하기 위해 `vue/cli-init`을 설치합시다.

    npm install @vue/cli-init -g

완료되었다면 아래의 명령어로 webpack 템플릿을 적용한 프로젝트를 생성합니다.

    vue init webpack [프로젝트명]


프로젝트를 생성하는 과정에서 아래처럼 커맨드 라인을 통해 설정을 진행할 수 있게 표시됩니다. 입맛에 맛게 진행하되 test runner부분에서는 Jest를 선택합시다. 물론 평소에는 선호하는 테스트 라이브러리를 사용하셔도 됩니다만 이번 글에는 Jest로 설명하기 때문에 Jest로 진행하겠습니다.

```
    ? Project name vue-jest-test
    ? Project description A Vue.js project
    ? Author tester
    ? Vue build standalone      
    ? Install vue-router? Yes
    ? Use ESLint to lint your code? Yes
    ? Pick an ESLint preset Standard
    ? Set up unit tests Yes
    ? Pick a test runner (Use arrow keys)
    > Jest
    Karma and Mocha
    none (configure it yourself)
```

선택이 끝나면 관련된 package를 설치하기 때문에 시간이 다소 소요될 수 있습니다.

### 2. 생성 완성된 프로젝트 살펴보기

생성이 완료되었다면 프로젝트는 아래의 구조를 가지게 됩니다.

    +---vue-jest-test
    |   +---build
    |   +---config
    |   +---node_modules
    |   +---src
    |   +---static
    |   +---test


 components 디렉토리 내부에는 HelloWorld.vue 파일이 이미 생성되어 있습니다.

![사진](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/vue-jest/d2.png?raw=true)



이뿐만 아니라 test 디렉토리에는 jest 테스트 및 e2e 테스트를 위한 파일과 설정파일이 준비되어있습니다.

![사진](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/vue-jest/d1.png?raw=true)


아래는 HelloWorld.spec.js 파일에 정의된 테스트 코드입니다. Jest는 디렉토리를 재귀적으로 탐색하여 모든 test파일을 탐색합니다.
```js
import Vue from 'vue'
import HelloWorld from '@/components/HelloWorld'

describe('HelloWorld.vue', () => {
  it('should render correct contents', () => {
    const Constructor = Vue.extend(HelloWorld)
    const vm = new Constructor().$mount()
    expect(vm.$el.querySelector('.hello h1').textContent)
      .toEqual('Welcome to Your Vue.js App')
  })
})


```


package.json 에도 테스트를 위한 명령어가 세팅되어있습니다.

```json
  "scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "unit": "jest --config test/unit/jest.conf.js --coverage",
    "e2e": "node test/e2e/runner.js",
    "test": "npm run unit && npm run e2e",
    "lint": "eslint --ext .js,.vue src test/unit test/e2e/specs",
    "build": "node build/build.js"
  },
```

이 상태에서 아래의 명령어로 테스트를 실행하면

    npm run unit

아래의 결과를 얻으실 수 있습니다.

![사진](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/vue-jest/test_result1.png?raw=true)


만약 `localstorage` 관련 에러가 난다면 이 [링크](https://stackoverflow.com/questions/51554366/jest-securityerror-localstorage-is-not-available-for-opaque-origins)를 참고하여 jest.conf.js 파일에 아래의 내용을 추가해줍시다.


```js
  ...
  verbose: true,
  testURL: "http://localhost/",
  ...
```


이로서 Vue CLI를 통해 생성된 프로젝트를 가볍게 살펴보았습니다.

## 간단한 예제를 통한 테스트

### 사칙연산을 수행하는 간단한 코드

> 아래 코드는 브랜디 랩스의 이경희 대리님의 코드[링크](http://labs.brandi.co.kr/2020/01/02/leekh.html)를 가지고 왔습니다.


사칙연산을 수행하는 간단한 코드입니다. 
```vue

<template>
  <div>
    <h1>{{ message }}</h1>
    <ul>
      <li><input type="text" v-model="frist" /></li>
      <li><input type="text" v-model="second" /></li>
      <li> = {{ total }} </li>
    </ul>
    <ul>
      <li><input type="button" value="더하기" @click="plus(frist, second)" /></li>
      <li><input type="button" value="빼기" @click="minus(frist, second)" /></li>
      <li><input type="button" value="곱하기" @click="multiply(frist, second)" /></li>
      <li><input type="button" value="나누기(반올림)" @click="divide(frist, second)" /></li>
    </ul>
    <h2></h2>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      message: 'Unit Test',
      frist: 7,
      second: 2,
      total: 0
    }
  },
  methods: {
    plus (frist, second) {
      this.total = parseInt(frist) + parseInt(second)
      return this.total
    },
    minus (frist, second) {
      this.total = parseInt(frist) - parseInt(second)
      return this.total
    },
    multiply (frist, second) {
      if (parseInt(frist) === 0 || parseInt(second) === 0) {
        return false
      }
      this.total = parseInt(frist) * parseInt(second)
      return this.total
    },
    divide (frist, second) {
      if (parseInt(frist) === 0 || parseInt(second) === 0) {
        return false
      }
      this.total = parseInt(frist) / parseInt(second)

      // 소숫점 반올림
      this.total = Math.round(this.total)
      return this.total
    }
  }
}
</script>
```

아래는 HelloWold.spec.js 파일입니다. jest가 재귀적으로 테스트 파일을 찾아 아래 테스트 코드를 실행하고 그 결과를 반환합니다.
```js
import Vue from 'vue'
import HelloWorld from '@/components/HelloWorld'

let Constructor
let vm
let frist
let second

// 초기화
beforeEach(() => {
  Constructor = Vue.extend(HelloWorld)
  vm = new Constructor().$mount()
  frist = vm._data.frist
  second = vm._data.second
})

describe('HelloWorld.vue', () => {
  describe('유효성 검사', () => {
    it('숫자가 아닌 Null 또는 문자열을 입력할 경우 결과값이 에러(NaN)인지 확인한다.', () => {
      second = '가'
      // NaN값인 경우
      expect(vm.plus(frist, second)).toBeNaN()
      expect(vm.minus(frist, second)).toBeNaN()
      expect(vm.multiply(frist, second)).toBeNaN()
      expect(vm.divide(frist, second)).toBeNaN()
    })
    it('0으로 곱하기 또는 나누기 하려고 하는지 확인한다.', () => {
      second = 0
      // toBeFalsy : if 문의 리턴값이 false 인 경우
      expect(vm.multiply(frist, 0)).toBeFalsy()
      expect(vm.divide(frist, 0)).toBeFalsy()
    })
  })
  describe('사칙연산 테스트', () => {
    it('더하기의 결과값을 비교하여 일치한지 확인한다.', () => {
      // toBe : 기본값을 비교할때 사용
      expect(vm.plus(frist, second)).toBe(9)
    })
    it('뺴기 결과값을 비교하여 정수인지 음수인지 확인한다.', () => {
      expect(vm.minus(frist, second)).toBe(5)
    })
    it('곱하기의 결과값을 비교하여 일치한지 확인한다.', () => {
      expect(vm.multiply(frist, second)).toBe(14)
    })
    it('나누기의 결과값을 비교하여 소스점 반올림이 되었는지 확인한다.', () => {
      // toBeCloseTo : 소수점 다음에 확인할 자리 수를 제어하는데 사용
      expect(vm.divide(frist, second)).toBeCloseTo(4, 5)
    })
  })
})
```
### 테스트 결과
![사진](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/vue-jest/test_result2.PNG?raw=true)

각 테스트 코드를 실행한 결과를 출력해줍니다. 위 사진은 테스트를 모두 정상적으로 통과하였기에 100% 통과율을 보였지만, 아래 예제처럼 소스코드 내에 테스트 코드로 처리가 안된 부분이 있다면 해당 소스코드를 Uncoverd Line로 지정하여 파일명 및 위치를 출력합니다.

![사진](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/vue-jest/test_result3.PNG?raw=true)

## 마무리

간단하게 Vue CLI를 이용한 Vue와 Jest의 사용법을 다루어보았습니다. 시간이 허락한다면 다음에는 jest가 제공하는 테스트 메서드와 mock 객체에 대해 다루어보려고 합니다.

## 참고한 글

- http://labs.brandi.co.kr/2020/01/02/leekh.html
- https://medium.com/witinweb/vue-cli-%EB%A1%9C-vue-js-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-browserify-webpack-22582202cd52
- https://stackoverflow.com/questions/51554366/jest-securityerror-localstorage-is-not-available-for-opaque-origins

## 더 읽으면 좋은 글

- [바닥부터 시작하는 Vue 컴포넌트 테스트](https://tech.kakao.com/2019/11/27/kakao-business-vue-component-test/)

- [E2E 테스트와 나이트왓치](https://blog.coderifleman.com/2016/06/17/e2e-test-and-nightwatch/)