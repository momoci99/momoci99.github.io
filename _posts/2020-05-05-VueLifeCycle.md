---
layout: single
title: "Vue Life Cycle"
tag:
  - Vue
  - LifeCycle
---

# Vue Life Cycle

## 개요

Vue는 React, Angular와 마찬가지로 LifeCycle hook을 제공합니다. 이 Life Cycle을 통해 원하는 시점에 필요한 처리를 할 수 있습니다. 최근 life cycle 관련으로 삽질을 하게되어 이번 기회에 정리해보고자 합니다.

## Vue의 life cycle

Vue의 전체적인 life cycle은 아래 그림과 같이 동작합니다.

![VueLifeCycle](https://kr.vuejs.org/images/lifecycle.png)

### beforeCreate

Vue 에서 호출되는 첫 번째 lifecycle hook 이며 Vue 인스턴스가 초기화 된 직후에 호출됩니다. 이 상태에서는 `computed`, `watch`,`data`, 및 이벤트 관련 항목이 아직 초기화 되지 않습니다.

```js
export default {
  components: {
    Child,
  },
  data() {
    return {
      showChild: false,
    };
  },
  methods: {
    toggleChild() {
      this.showChild = !this.showChild;
    },
  },
  beforeCreate() {
    console.log(this.showChild);
    console.log("Parent::beforeCreate");
  },
};

//실행 결과
//undefined
//Parent::beforeCreate
```

따라서 위의 코드처럼 beforeCreate hook에서 data에 정의된 `showChild` 변수에 접근하여도 초기화가 아직 안되었기에 `undefined`가 리턴됩니다.

### created

created는 beforeCreated hook 바로 다음에 호출되는 두 번째 lifecycle hook 입니다. 이 시점에서 Vue 인스턴스의 `computed`, `watch`,`data`, 및 이벤트 관련 항목의 초기화가 시작된 상태입니다. 따라서 관련 속성에 정의된 변수에 접근이 가능해집니다. 다만 인스턴스는 초기화 되었지만 마운트는 완료되지 않았으므로 DOM을 조작할 수 는 없습니다. 보통 외부 데이터의 fetch 작업이 이 hook에서 이루어집니다.

```js
  computed: {
    squreNumber () {
      return this.number * 2
    }
  },
  data () {
    return {
      showChild: false,
      number: 2
    }
  },
  created () {
    console.log(this.showChild)
    console.log(this.squreNumber)
    console.log('Parent::created')
  },


//실행 결과
//false
//4
//Parent::created

```

위 코드의 실행 결과를 보았을 때 created hook 내에서 `data`, `computed` 속성에 접근하여 정상적으로 결과를 확인 할 수 있습니다.

### beforeMount

beforeMount는 인스턴스가 DOM에 적재되기 바로 직전 상태의 hook 입니다. 이 상태에서 template과 scpoed style이 컴파일 됩니다. 물론 DOM에 적재되지 않았으니 DOM에 접근할 수 없습니다.

### mounted

beforeMount 바로 다음에 호출되는 life cycle hook 입니다. 여기서부터 앱 구성요소 및 프로젝트의 다른 구성 요소가 동작하여 접근 및 사용이 가능합니다. `data`가 템플릿에 적재되고 DOM 요소는 data가 반영된 DOM 요소로 대체됩니다. 물론 DOM 요소에 접근도 가능해집니다.

### beforeUpdate

beforeUpdate는 mounted hook 이 호출된 이후 DOM 이 업데이트 되기 전에 호출되는 hook 입니다. 이 hook에서는 이벤트 리스너를 제거하거나 데이터가 변경되기 전에 필요한 처리들을 수행하기 좋습니다.

### beforeDestory

beforeDestory는 Vue 인스턴스가 파괴되기 직전에 호출됩니다. 파괴되기 직전이기 때문에 인스턴스와 모든 기능은 그대로 유지되고 있습니다. 리소스 관리, 변수 삭제, 구성 요소를 정리할 수 있는 단계입니다.

### destroyed

destroyed는 모든 하위 Vue 인스턴스가 파괴된 Vue life cycle의 가장 마지막 hook이며 이벤트 리스터, 지시문이 이 단계에서 해제됩니다.

## Parent 와 Child간의 Life Cycle

부모와 자식간의 life cycle이 어떻게 진행되는지 아래 코드로 살펴보겠습니다.

```js
//Parent.vue
<template>
    <div>
        I'Parent
        <button @click="toggleChild">toggle child</button>
        <child v-if="showChild"></child>

    </div>
</template>
<script>
import Child from './Child'
export default {
  components: {
    Child
  },
  computed: {
    squreNumber () {
      return this.number * 2
    }
  },
  data () {
    return {
      showChild: false,
      number: 2
    }
  },
  methods: {
    toggleChild () {

      this.showChild = !this.showChild
      console.log('toggled!!')
    }
  },
  beforeCreate () {
    console.log('Parent::beforeCreate')
  },
  created () {
    console.log('Parent::created')
  },
  beforeMount () {
    console.log('Parent::beforeMount')
  },
  mounted () {
    console.log('Parent::mounted')
  },
  beforeUpdate () {
    console.log('Parent::beforeUpdate')
  },
  updated () {
    console.log('Parent::updated')
  },
  beforeDestroy () {
    console.log('Parent::beforeDestroy')
  },
  destroyed () {
    console.log('Parent::destroyed')
  }
}
</script>
<style scoped>

</style>
```

```js
//Child.vue
<template>
    <div>
        I'm Child
    </div>
</template>
<script>
export default {
  beforeCreate () {
    console.log('Child::beforeCreate')
  },
  created () {
    console.log('Child::created')
  },
  beforeMount () {
    console.log('Child::beforeMount')
  },
  mounted () {
    console.log('Child::mounted')
  },
  beforeUpdate () {
    console.log('Child::beforeUpdate')
  },
  updated () {
    console.log('Child::updated')
  },
  beforeDestroy () {
    console.log('Child::beforeDestroy')
  },
  destroyed () {
    console.log('Child::destroyed')
  }
}
</script>
<style scoped>

</style>

```

### toggle을 통해 child component 생성

    Parent::beforeCreate
    Parent::created
    Parent::beforeMount
    Parent::mounted
    toggled!!
    Parent::beforeUpdate
    Child::beforeCreate
    Child::created
    Child::beforeMount
    Child::mounted
    Parent::updated

Parent 컴포넌트가 적재된 이후 toggle 버튼을 통해 Child 컴포넌트를 표시하게 만들었습니다. toggle 버튼으로 인해 Child 컴포넌트가 적재되는 과정에서 DOM이 업데이트 되기 때문에 Parent의 `beforeUpdate`가 먼저 실행되고 이후 Child 컴포넌트의 life cycle hook이 호출됩니다.

child 컴포넌의가 완전히 적재된 이후 다시 Parent 컴포넌트의 updated hook 이 호출됩니다.

### toggle을 통해 child component 제거

    toggled!!
    Parent::beforeUpdate
    Child::beforeDestroy
    Child::destroyed
    Parent::updated

toggle을 통해 child 컴포넌트를 제거하는 경우 생성과 마찬가지로 DOM에 변화가 일어나기 떄문에 Parent의 `beforeUpdate` hook이 호출되고 Child 컴포넌트가 제거된 후 Parent의 `updated` hook이 호출됩니다.

### toggle 없는 일반적인 Parent-Child의 경우

이번에는 단순히 toggle없이 일반적인 Parent-Child 구조에서 life cycle이 어떻게 호출되는지 살펴보겠습니다.

    Parent::beforeCreate
    Parent::created
    Parent::beforeMount
    Child::beforeCreate
    Child::created
    Child::beforeMount
    Child::mounted
    Parent::mounted

Parent가 마운트되기전에 Child의 life cycle hook이 호출되고 Child컴포넌트가 마운트 된 후에 Parent 컴포넌트의 `mounted` hook이 호출됩니다.

## 정리

vue의 lifecycle hook에 대해 정리해보았습니다. 이 hook들을 통해 필요한 로직을 사용하여 좀더 강력한 App을 만들 수 있겠습니다.
