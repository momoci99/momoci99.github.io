---
layout: single
title: "vue에서 jest 써보기"
tag:
  - vue
---

이번 글은 아주 아주 간단한 증감 연산을 수행하는 웹 앱을 vue를 통해 만들고 jest 를 활용하여 unit-test를 해보는 글입니다.

# Jest?

- [https://jestjs.io/](https://jestjs.io/)

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled.png)

FaceBook(지금은 Meta)에서 만든 테스팅 프레임워크입니다. 프레임워크라고 불릴 정도로 내부에 많은 기능이 있습니다. Test runner, Test matcher, coverage test, mocking 등등.. 덕분에 jest 하나면 react던 vue던 수월하게 테스트를 작성하고 사용할 수 있습니다.

# VTU - Vue Test Utils

- [https://vue-test-utils.vuejs.org/](https://vue-test-utils.vuejs.org/)

Jest 자체로도 강력한 도구를 제공합니다만, Vue Test Utils을 사용하면 `shallowMount()`, `createLocalVue()` 와 같은 API를 활용하여 좀더 편리하게 테스트 할 수 있습니다.

# 만들어볼 App의 기능 정의

Unit Test를 작성하며 만들어 볼 app의 기능 정의는 다음과 같습니다.

1. 웹 페이지에 숫자, 버튼이 표현된다.
2. - 버튼을 누르면 숫자가 + 1 된다.
3. - 버튼을 누르면 숫자가 -1 된다.

## 1. App.vue, Button.vue 컴포넌트 생성

우선 두개의 컴포넌트를 만들었습니다.

App.vue - 숫자와 버튼을 표시하고 버튼의 이벤트를 처리

Button.vue - 클릭시 정의한 이벤트를 처리

App.vue

```jsx
<template>
	<div id="app">
		<h1>{{ number }}</h1>
		<Button actionType="add" text="+" @event="updateNumber"></Button>
		<Button actionType="sub" text="-" @event="updateNumber"></Button>
	</div>
</template>

<script>
import Button from "./components/button";

export default {
	name: "App",
	components: {
		Button,
	},
	data() {
		return {
			number: 0,
		};
	},
	methods: {
		updateNumber(actionType) {
			if (actionType === "add") {
				this.number++;
			} else if (actionType === "sub") {
				this.number--;
			}
		},
	},
};
</script>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}
</style>
```

Button.vue

```jsx
<template>
	<button class="btn" @click="handleClick">
		{{ text }}
	</button>
</template>

<script>
export default {
	name: "Button",
	props: {
		text: {
			type: String,
			default: "",
		},
		actionType: {
			type: String,
			required: true,
			validate: function (value) {
				return ["add", "sub"].indexOf(value) !== -1;
			},
		},
	},
	methods: {
		handleClick() {
			this.$emit("event", this.actionType);
		},
	},
};
</script>

<style>
</style>
```

동작 결과

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled.gif)

## 2. Button.vue의 테스트 코드 작성

Button.vue 컴포넌트의 테스트 코드를 작성해봅시다.

- shallowMount를 사용하여 Button.vue 컴포넌트를 mount하고 mount할때 propsData속성을 활용하여 text, actionType을 전달합니다.
- shallowMount호출시 반환되는 Wrapper 객체를 통해 컴포넌트에 이벤트를 발생시키고, emit된 이벤트를확인할 수 있습니다.

button.spec.js- 테스트 코드 작성

```jsx
import { shallowMount } from "@vue/test-utils";
import Button from "@/components/Button.vue";

describe("Button.vue", () => {
  it("text props가 렌더링 되는지 확인", () => {
    const text = "+";
    const actionType = "add";
    const wrapper = shallowMount(Button, {
      propsData: { text, actionType },
    });
    expect(wrapper.text()).toMatch(text);
  });

  it("button을 클릭했을 때 props에 전달한 actionType이 emit되는지 확인", () => {
    const text = "+";
    const actionType = "add";
    const wrapper = shallowMount(Button, {
      propsData: { text, actionType },
    });

    wrapper.find("button").trigger("click");
    const add = wrapper.emitted().event[0][0];
    expect(add).toBe(actionType);
  });
});
```

테스트 결과

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled%201.png)

## 3. 기능 추가 - 부호에 따른 css style 변경

아래의 기능을 추가해보겠습니다.

- 숫자가 양수일 때 초록색
- 숫자가 음수일 때 빨간색

숫자에 대한 기능이 추가되면서 App.vue 컴포넌트에서 표시되던 숫자를 Count.vue 컴포넌트로 분리했습니다.

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled%202.png)

Count.vue 컴포넌트

```jsx
<template>
	<h1 :class="countClasses">{{ count }}</h1>
</template>

<script>
export default {
	name: "Count",
	props: {
		count: {
			type: Number,
			required: true,
		},
	},
	computed: {
		countClasses: function () {
			return {
				positive: this.count > 0,
				negative: this.count < 0,
				zero: this.count === 0,
			};
		},
	},
};
</script>

<style>
.positive {
	color: #3150d8;
}
.negative {
	color: #b03f3f;
}
.zero {
	color: #808080;
}
</style>
```

테스트 코드 작성 - count.spec.js

- Wrapper api를 활용하여 컴포넌트의 class를 체크합니다

```jsx
import { shallowMount } from "@vue/test-utils";
import Count from "@/components/Count";

describe("Count.vue", () => {
  it("count 10 표시", () => {
    const count = 10;
    const wrapper = shallowMount(Count, {
      propsData: { count },
    });
    const currentCount = Number.parseInt(wrapper.text(), 10);
    expect(currentCount).toStrictEqual(count);
  });

  it("count가 양수일 때 class가 positive", () => {
    const count = 10;
    const wrapper = shallowMount(Count, {
      propsData: { count },
    });

    const classes = wrapper
      .find("h1")
      .classes()
      .find((className) => {
        return className === "positive";
      });
    expect(classes).toStrictEqual("positive");
  });

  it("count가 음수일 때 class가 negative", () => {
    const count = -10;
    const wrapper = shallowMount(Count, {
      propsData: { count },
    });

    const classes = wrapper
      .find("h1")
      .classes()
      .find((className) => {
        return className === "negative";
      });
    expect(classes).toStrictEqual("negative");
  });

  it("edge case - MAX", () => {
    const count = Number.MAX_SAFE_INTEGER + 1;
    const wrapper = shallowMount(Count, {
      propsData: { count },
    });

    const classes = wrapper
      .find("h1")
      .classes()
      .find((className) => {
        return className === "positive";
      });

    expect(classes).toStrictEqual("positive");
  });

  it("edge case - MIN", () => {
    const count = Number.MIN_SAFE_INTEGER - 1;
    const wrapper = shallowMount(Count, {
      propsData: { count },
    });

    const classes = wrapper
      .find("h1")
      .classes()
      .find((className) => {
        return className === "negative";
      });
    expect(classes).toStrictEqual("negative");
  });
});
```

동작 결과

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled%201.gif)

테스트 결과

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled%203.png)

# 4. 기능추가 - reset

- 버튼 클릭시 숫자를 0으로 리셋하는 기능을 추가해봅시다.

App.vue 컴포넌트에 reset 역할을 해줄 Button 컴포넌트를 추가합니다.

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled%204.png)

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled%205.png)

테스트 코드 작성 - app.spec.js

```jsx
import { mount } from "@vue/test-utils";
import App from "@/app";

describe("App.vue", () => {
  it("app 컴포넌트 mount", () => {
    const wrapper = mount(App, {});
    expect(wrapper.isVisible()).toBe(true);
  });

  it("count를 10으로 셋팅한 뒤 reset", () => {
    const wrapper = mount(App, {});
    wrapper.setData({ count: 10 });
    const buttons = wrapper.findAll("button");
    const buttonLength = buttons.length;

    let resetButton;
    for (let i = 0; i < buttonLength; i++) {
      if (buttons.at(i).props().actionType === "reset") {
        resetButton = buttons.at(i);
      }
    }

    resetButton.trigger("click");
    expect(wrapper.vm.count).toBe(0);
  });
});
```

![Untitled](vue%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20jest%20%E1%84%8A%E1%85%A5%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%200be3c6cace4549d6a7933102c806670e/Untitled%202.gif)

# 정리

이번 글에서는 간단한 앱을 만들면서 unit test 코드를 작성하는 과정을 다뤄보았습니다. vue는 VTU라는 유용한 도구를 제공하므로 jest와 VTU를 활용하면 좀 더 수월하게 unit test를 작성할 수 있습니다.

# 참고했던 글

- [https://heropy.blog/2020/05/20/vue-test-with-jest/](https://heropy.blog/2020/05/20/vue-test-with-jest/)
- [https://www.daleseo.com/jest-basic/](https://www.daleseo.com/jest-basic/)
