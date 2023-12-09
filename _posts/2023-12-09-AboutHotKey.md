---
layout: single
title: "단축키 관련 라이브러리"
tag:
  - Web
  - React
  - HotKey
---

최근에 같은 팀원으로부터 많은 단축키를 사용하는 상황에서 어떻게 하는게 좋을 지 질문을 받았었는데, 선뜻 답이 떠오르지 않아 두루뭉술하게 대답을 했었습니다. 이번 기회에 어떤 라이브러리가 있는지 알아보고 어떤 구조가 좋을지 알아보고자 합니다.

# 단축키 라이브러리

### **hotkeys-js**

- GitHub : https://github.com/jaywcjlove/hotkeys-js
- 단축키 지정을 위한 js 라이브러리.

**example code**

- hotkeys를 사용하여 윈도우에서 f5 동작을 막고 등록한 콜백함수를 동작시키는 함수.

```jsx
import hotkeys from "hotkeys-js";

hotkeys("f5", function (event, handler) {
  event.preventDefault();
  alert("you pressed F5!");
});
```

- 여러 키를 조합한 경우를 처리하는 예제 코드

```jsx
hotkeys("ctrl+a,ctrl+b,r,f", function (event, handler) {
  switch (handler.key) {
    case "ctrl+a":
      alert("you pressed ctrl+a!");
      break;
    case "ctrl+b":
      alert("you pressed ctrl+b!");
      break;
    case "r":
      alert("you pressed r!");
      break;
    case "f":
      alert("you pressed f!");
      break;
    default:
      alert(event);
  }
});
```

## \***\*react-hot-keys\*\***

- GitHub : https://www.npmjs.com/package/react-hot-keys
- 명시적으로 컴포넌트 내에서 키보드 단축키를 사용하기 위한 react 라이브러리
- https://github.com/jaywcjlove/hotkeys-js의 fork버전에서 시작

**example code**

- react class component를 사용한 예제
- `<Hotkeys>` hoc로 wrapping하고 `keyName`으로 모니터링할 단축키 이름 지정

```jsx
export default class HotkeysDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      output: "Hello, I am a component that listens to keydown and keyup of a",
    };
  }
  onKeyUp(keyName, e, handle) {
    console.log("test:onKeyUp", e, handle);
    this.setState({
      output: `onKeyUp ${keyName}`,
    });
  }
  onKeyDown(keyName, e, handle) {
    console.log("test:onKeyDown", keyName, e, handle);
    this.setState({
      output: `onKeyDown ${keyName}`,
    });
  }
  render() {
    return (
      <Hotkeys
        keyName="shift+a,alt+s"
        onKeyDown={this.onKeyDown.bind(this)}
        onKeyUp={this.onKeyUp.bind(this)}
      >
        <div style={{ padding: "50px" }}>{this.state.output}</div>
      </Hotkeys>
    );
  }
}
```

## **react-hotkeys-hook:**

- GitHub: https://github.com/JohannesKlauss/react-hotkeys-hook
- react-hot-keys의 react hook 버전

**example code**

- 컴포넌트에 ctrl+k 키를 바인딩하는 예제

```jsx
import { useHotkeys } from "react-hotkeys-hook";

export const ExampleComponent = () => {
  const [count, setCount] = useState(0);
  useHotkeys("ctrl+k", () => setCount(count + 1), [count]);

  return <p>Pressed {count} times.</p>;
};
```

# focus trap

- 특정 DOM에 foucs를 가두는 라이브러리
- 라이브러리 : https://github.com/focus-trap/focus-trap-react
- 특이사항 : FocusTrap hoc의 자식 컴포넌트는 ref를 가질 수 있어야함.

**example code**

- modal 컴포넌트에 hoc로 wrapping하여 포커스를 가두는 예

```jsx
<FocusTrap>
  <div id="modal-dialog" className="modal">
    <button>Ok</button>
    <button>Cancel</button>
  </div>
</FocusTrap>
```

# 동일한 단축키가 컴포넌트별로 다른 동작을 해야할때에 대한 고민

단축키 관련 기능을 개발하다보면 동일한 단축키가 상황에 따라 다르게 사용되어야하는 경우가 종종 있습니다. 🤔

1. 기본 화면에서 F를 눌렀을때 캐릭터 F키에 할당된 스킬을 사용해야합니다.
2. 아이템 창이 열려 있는 상태에 F키를 눌렀을 때는 아이템 찾기 기능의 필터를 오픈해주어야합니다.

웹 브라우저에서 이를 구현하기에는 고민되는 부분이 있습니다. `document.addEventListener` 로 키 바인딩을 하고나면 두 key event가 동시에 트리거 됩니다.

실제 코드로 예를 들어보면 아래 두 컴포넌트를 마운트하게 되면 keydown할 때 마다 각 컴포넌트에 추가된 리스너가 각각 동작하게 됩니다.

```tsx
//BaseComponent.tsx
const BaseComponent = () => {
  useEffect(() => {
    document.addEventListener("keydown", () => {
      console.log("key press in Base");
    });

    return () => {
      console.log("unmounted");

      document.removeEventListener("keydown", () => {
        console.log("key press in Base");
      });
    };
  }, []);

  return <div>BaseComponent</div>;
};
```

```jsx
//Modal.tsx
const Modal = ({ title, content, onClose, onConfirm }: ModalProps) => {
  useEffect(() => {
    document.addEventListener("keydown", (e) => {
      console.log("key press in modal");
    });

    return () => {
      document.removeEventListener("keydown", (e) => {
        console.log("key press in modal");
      });
    };
  });

  return (
    <ModalWrapper>
      <ModalInner>
        <ModalTitle>{title}</ModalTitle>
        <ModalContent>{content}</ModalContent>
        <ModalButtonWrapper>
          <ModalButton onClick={onClose}>취소</ModalButton>
          <ModalButton onClick={onConfirm}>확인</ModalButton>
        </ModalButtonWrapper>
      </ModalInner>
    </ModalWrapper>
  );
};
```

바로 이렇게요

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6a9964a5-a4b3-478c-a26c-202e04bf6b13/78d8eeda-5819-4366-868b-ba32c16d4bf6/Untitled.png)

이때 **react-hotkeys-hook** 에서 제공하는 Scoping hotkeys to components 를 사용하여 해당 컴포넌트가 포커스되었을 때만 등록한 키가 동작하도록 할 수 있습니다.

useHotkeys에서 반환되는 ref를 포커스 대상 컴포넌트에 넘겨주면 해당 컴포넌트가 포커스 되었는지를 확인하여 핫키가 동작할 수 있게 만들어 줍니다. [예제 링크](https://react-hotkeys-hook.vercel.app/docs/documentation/useHotkeys/scoping-hotkeys)

```jsx
function ScopedHotkey() {
  const [count, setCount] = useState(0);
  const ref = useHotkeys("shift+a", () =>
    setCount((prevCount) => prevCount + 1)
  );

  return (
    <>
      <p>
        The count is {count}. Click anywhere except for the button to disable
        the hotkey.
      </p>
      <button ref={ref}>Click me to enable the hotkey</button>
    </>
  );
}

function SecondScopedHotkey() {
  const [count, setCount] = useState(0);
  const ref = useHotkeys("shift+a", () =>
    setCount((prevCount) => prevCount + 1)
  );

  return (
    <>
      <p>
        The count is {count}. Click anywhere except for the button to disable
        the hotkey.
      </p>
      <button ref={ref}>Click me to enable the hotkey</button>
    </>
  );
}

render(
  <div>
    <ScopedHotkey />
    <hr />
    <SecondScopedHotkey />
  </div>
);
```

경우에 따라 tabIndex에 -1을 부여하여 tab키로 접근은 불가능하지만 포커스는 가능하게 만들어 주어야하는 경우도 있습니다. (ex : 모달)

# 참고 자료

---

\***\*React 앱에 단축키 적용하기\*\*** https://velog.io/@juno7803/React-hotkeys

**React-hotkeys-hook document** https://react-hotkeys-hook.vercel.app/

tabIndex 관련 내용 : https://developer.mozilla.org/ko/docs/Web/HTML/Global_attributes/tabindex
