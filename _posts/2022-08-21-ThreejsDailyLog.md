---
layout: single
title: "React에서 threejs 사용 및 삽질"
tag:
  - threejs
---

React + Typescript 환경에서 Threejs 사용시 활용했던 라이브러리 및 삽질했던 내용들을 간단하게 정리하였습니다.

**Threejs**

[https://github.com/mrdoob/three.js/](https://github.com/mrdoob/three.js/)

웹 브라우저에서 3d 그래픽을 처리할 수 있게 해주는 라이브러리 입니다. 웹 브라우저에서 3d 관련 라이브러리 작업에는 거의 필수로 사용되고 있습니다. 물론 바닐라 상태로 사용해도 됩니다만, react에서 좀 더 편리하게 사용하기 위한 wrapper 라이브러리를 사용하는게 좋습니다.

**React Three Fiber**

[https://github.com/pmndrs/react-three-fiber](https://github.com/pmndrs/react-three-fiber)

react에서 threejs를 사용하기 위한 renderer입니다. threejs의 기본 컴포넌트를 react JSX.Element로 제공해주며, Custom Hook을 통해 보다 쉽게 threejs Object를 다룰 수 있게 해줍니다.

**Drei**

https://github.com/pmndrs/drei

React Three Fiber를 좀 더 쉽게 사용하게 해주는 Helper 라이브러리입니다. (React에서 threejs 하나 쓰는데 벌써 라이브러리를 2개 가져다 썼습니다😱)

## 삽질 기록

**React Custom Hook**

React, React Three Fiber, Drei에서 제공하는 Hook, Custom Hook은 반드시 컴포넌트 안에서 호출해야합니다. 외부에서 호출시 react에서 아래의 에러를 뱉습니다. (react 초보는 당황스러울 뿐입니다)

```plaintext
React Hook "useThree" must be called in function "xxx" that is neither a React component nor a custom React hook function.
```

[https://ko.reactjs.org/docs/hooks-rules.html](https://ko.reactjs.org/docs/hooks-rules.html)

**Three.js-object의 크기 측정하기**

```jsx
const box = new Box3().setFromObject(object);
const vc = new Vector3();
const size = box.getSize(vc);
```

- threejs의 object 크기를 구하기 위해서 box3 object를 생성하고 크기를 구하려는 object에 대해 setFromObject를 호출하여야합니다.
- 그후 Vector3 객체를 통해 사이즈를 구하게 됩니다. (box.getSize의 파라미터로 전달한 Vector3 객체에 사이즈 정보가 저장되는 구조)

Three.js-object의 속성 변경하기

- threejs는 object의 속성을 그냥 변경할 수 없는 경우가 종종 있습니다.
- 객체를 새로 생성하던가, 객체의 setter를 호출하던가 해야합니다.
- [https://threejs.org/docs/#manual/en/introduction/How-to-update-things](https://threejs.org/docs/#manual/en/introduction/How-to-update-things)

**Three.js-material의 이미지 texture 변경하기**

```jsx
const Box = () => {
  const mesh = useRef();
  const material = useRef();
  const { textureUrl } = useContext(AppContext);
  useEffect(() => {
    if (textureUrl) {
      const textureLoader = new TextureLoader();
      textureLoader.load(textureUrl, (t) => {
        material.current.map = t;
        material.current.needsUpdate = true;
      });
    }
  }, [textureUrl]);
  return (
    <mesh ref={mesh}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial ref={material} />
    </mesh>
  );
};
```

- material의 texture를 업데이트 할 때 TextureLoader의 callback에서 처리합니다.
- material.current.map에 texture를 직접 할당했다가 업데이트가 안되서 애먹었습니다 😭
- [https://github.com/pmndrs/react-three-fiber/discussions/1383](https://github.com/pmndrs/react-three-fiber/discussions/1383)
- [https://codesandbox.io/s/elated-fire-198px](https://codesandbox.io/s/elated-fire-198px)

Threejs-Helper 사용

- drei의 `useHelper`를 사용합니다.
- [https://codesandbox.io/s/r3f-use-helper-ly6kw?from-embed=&file=/src/App.js:1421-1437](https://codesandbox.io/s/r3f-use-helper-ly6kw?from-embed=&file=/src/App.js:1421-1437)
