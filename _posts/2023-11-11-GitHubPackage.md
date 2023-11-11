---
layout: single
title: "Github Package"
tag:
  - github
---

회사 소스코드의 레거시를 개선해보고자 github package에 대해서 알아보고 실제 예제를 통해 어떻게 동작하는지, 삽질⚒️했던 부분도 함께 살펴보겠습니다.

# Github Package?

- [https://github.com/features/packages](https://github.com/features/packages)
- npm과 마찬가지로 js package(module, lib..)를 저장하고 관리할 수 있는 저장소입니다.
- npm은 기본적으로 public으로 공개되는것과 다르게, public, organization 단위로 관리할 수 있습니다.
- private npm을 구현하기위해 별도의 저장소(ex : verdaccio)구축이 필요하지 않습니다.

# 요금 체계

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled.png?raw=true)

github packages는 2023년 11월 11일 기준으로 다음과 같은 요금 체계를 가지고 있습니다.

- Free : Storage 500MB / github action외부 네트워크 트래픽 1GB 제한
- Team : Storage 2GB / github action외부 네트워크 트래픽 10B 제한
- Enterprise : Storage 500GB / github action외부 네트워크 트래픽 100GB 제한

# 실제로 해보기

- 이번 예제는 react typescript로 간단한 counter 컴포넌트를 생성하고 github action을 통해 package를 publish합니다
- publish한 컴포넌트를 다른 repo에서 참조하는 과정을 알아보겠습니다.

사용될 Repo

- test-common-ui : github packages로 업로드될 모듈을 관리하는 repo
- test-import-project : test-common-ui repo에서 생성한 github package를 설치하여 구동하는 repo

# Package를 관리할 Repo setup (test-common-ui)

(github repo 셋팅은 생략하겠습니다)

1. npm package를 위한 초기 설정을 진행합니다.

```powershell
npm init
```

- package.json의 상세 설정은 더 아래 단계에서 다루겠습니다.

1. react, react-dom, typescript 설치

```powershell
npm install --save-dev react react-dom typescript @types/react
```

(보일러 플레이트를 사용해도 상관은 없지만 약간의 추가 작업이 필요할 수 있습니다)

1. tsconfig.json을 설정합니다.

```json
{
  "include": ["src"],
  "exclude": ["dist", "node_modules"],
  "compilerOptions": {
    "module": "esnext",
    "lib": ["dom", "esnext"],
    "importHelpers": true,
    "declaration": true,
    "sourceMap": true,
    "rootDir": "./src",
    "outDir": "./dist/esm",
    "strict": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "moduleResolution": "node",
    "jsx": "react",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

1. 공통 컴포넌트를 작성합니다.

컴포넌트 위치 : root 디렉터리/src/components/MyCounter.tsx

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_1.png?raw=true)

```tsx
import React, { useState } from "react";

type Props = {
  value?: number;
};
const MyCounter = ({ value = 0 }: Props) => {
  const [counter, setCounter] = useState(value);

  const onMinus = () => {
    setCounter((prev) => prev - 1);
  };

  const onPlus = () => {
    setCounter((prev) => prev + 1);
  };

  return (
    <div>
      <h1>Counter: {counter}</h1>
      <button onClick={onMinus}>-</button>
      <button onClick={onPlus}>+</button>
    </div>
  );
};

export default MyCounter;
```

1. 공통 컴포넌트를 Export하기 위한 index.ts파일을 작성합니다.

index.ts의 위치 : root 디렉터리/src/index.ts

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_2.png?raw=true)

```tsx
import MyCounter from "./components/MyCounter";

export { MyCounter };
```

1. publish를 위해 npm script를 수정합니다.

package.json의 script 영역에 아래의 스크립트를 추가해주세요.

```tsx
"build": "npm run build:esm && npm run build:cjs",
"build:esm": "tsc",
"build:cjs": "tsc --module commonjs --outDir dist/cjs",
"prepare": "npm run build" //package가 pack 되기 전에 실행할 명령어입니다.
```

추가되었다면 아래와 같을것입니다. (추가적인 스크립트는 임의로 작성할 수 있습니다)

```tsx
"scripts": {
    "build:tsc": "tsc", //임의로 추가한 스크립트
    "build": "npm run build:esm && npm run build:cjs",
    "build:esm": "tsc",
    "build:cjs": "tsc --module commonjs --outDir dist/cjs",
    "prepare": "npm run build"
  },
```

1. entrypoint를 지정하기 위해 package.json을 수정합니다

```tsx
"main": "./dist/cjs/index.js",
"module": "./dist/esm/index.js",
"types": "./dist/esm/index.d.ts",
```

1. package.json에 repository 주소를 설정합니다.

```tsx
"repository": {
    "type": "git",
    "url": "git+.githubRepoUrlgit" //example https://github.com/aaaa/reponame.git"
  },
```

1. package.json에 해당 패키지의 최소 요구 의존성 패키지를 명시합니다. (최소한 react 16 버전 이상을 요구합니다)

```tsx
"peerDependencies": {
  "react": ">=16"
},
```

1. package.json에 package에 포함될 파일을 지정합니다

```tsx
"files": [
  "dist",
  "LICENSE",
  "README.md"
],
```

1. package.json에 author를 수정합니다.

주의: author에는 package의 base가 되는 계정 명을 명시해야합니다. 그렇지 않으면 에러의 원인이 됩니다.

```tsx
"author": "momoci99"
```

1. package.json에 저장소 정보를 추가하기 위해 publishConfig를 추가합니다.

```tsx
"publishConfig": {
    "@momoci99:registry": "https://npm.pkg.github.com"
 },
```

1. 완성된 package.json 스크립트는 다음과 비슷한 형태를 가지게 됩니다.

```json
{
  "name": "@momoci99/test-common-ui",
  "version": "1.1.4", //버전은 필요에 따라 수정해주세요.
  "description": "",
  "main": "./dist/cjs/index.js",
  "module": "./dist/esm/index.js",
  "types": "./dist/esm/index.d.ts",
  "scripts": {
    "build:tsc": "tsc",
    "build": "npm run build:esm && npm run build:cjs",
    "build:esm": "tsc",
    "build:cjs": "tsc --module commonjs --outDir dist/cjs",
    "prepare": "npm run build"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/momoci99/test-common-ui.git"
  },
  "author": "momoci99",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/momoci99/test-common-ui/issues"
  },
  "homepage": "https://github.com/momoci99/test-common-ui#readme",
  "peerDependencies": {
    "react": ">=16"
  },
  "devDependencies": {
    "@types/react": "^18.2.37",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.2.2"
  },
  "publishConfig": {
    "@momoci99:registry": "https://npm.pkg.github.com"
  },
  "files": ["dist", "LICENSE", "README.md"]
}
```

1. 루트 디렉터리에 .npmrc 파일을 만들고 github package저장소를 사용할것을 알려줍니다.

```
@momoci99:registry=https://npm.pkg.github.com
```

# Github Action을 통해 publish하기 위한 setup하기

Github Action을 사용하여 release될 때마다 publish하기 위해 action 스크립트를 추가하겠습니다.

루트 디렉터리/.github/workflows/release-package.yml

```yaml
name: Node.js Package

on:
  release:
    types: [created]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v3
        with:
          node-version: 16
      - run: npm ci

  publish-gpr:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v3
        with:
          node-version: 16
          registry-url: https://npm.pkg.github.com/
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

- 여기서는 토큰이 따로 필요 없습니다. github action내에서 secrets.GITHUB_TOKEN은 [자동 토큰 인증](https://docs.github.com/ko/actions/security-guides/automatic-token-authentication)을 사용합니다.

# Tagging & Release

1. 현재까지의 변경사항을 commit하고 원격 저장소에 push까지 완료한 후 tagging을 수행합니다.

```bash
git tag v1.0.0
```

1. tag한 후 원격 저장소에 push합니다

```bash
git push origin v1.0.0
```

1. Releases 메뉴를 활용하여 tag된 v1.0.0을 릴리즈합니다.

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_3.png?raw=true)

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_4.png?raw=true)

1. Github Action이 수행되면서 package의 publish가 수행됩니다.

(꼭 github action을 사용하지 않고 수동으로 publish할 수 있습니다)

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_5.png?raw=true)

1. package가 정상적으로 업로드 된것을 확인할 수 있습니다.

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_6.png?raw=true)

# 실제 package 사용하기

참고 : Github Package를 public repo에 업로드 하였더라도 사용하기 위해서는 PAT(Private Access Token)이 필요합니다.

[PAT 토큰 발행](https://docs.github.com/ko/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) (권한 지정시 package:read 권한만 부여해주세요.)

1. publish된 Github Package를 사용하기위해 간단한 react application을 셋팅 및 .npmrc에 제가 올린 package 저장소 정보를 추가합니다.

```bash
@momoci99:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken={PAT_TOKEN}
```

- 주의 : **Token은 원격 저장소에 push하면 안됩니다.** (실제로 push하게되면 github에서 강제로 수정합니다)
- 환경변수로 관리하는걸 권장합니다.

1. package를 설치합니다.

```bash
npm install @momoci99/test-common-ui@1.0.0
```

1. 설치를 확인하고 해당 컴포넌트를 import 하여 사용합니다.

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_7.png?raw=true)

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_8.png?raw=true)

![Untitled](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/2023-11-11-GitHubPackage/Untitled_9.png?raw=true)

# Summary

- Github Packages를 사용하여 react ui 컴포넌트를 패키징하고 github action을 통해 publish 하였습니다.
- 다른 프로젝트에서 이를 참조하여 실제로 사용해 보았습니다.

사용한 repo

- [https://github.com/momoci99/test-import-project](https://github.com/momoci99/test-import-project)
- [https://github.com/momoci99/test-common-ui](https://github.com/momoci99/test-common-ui)

# Etc

- 사내 Design System구현시 github package을 활용한다면 매우 유용하게 쓰일 것 같습니다.

---

참고 글

[https://betterprogramming.pub/how-to-create-and-publish-react-typescript-npm-package-with-demo-and-automated-build-80c40ec28aca](https://betterprogramming.pub/how-to-create-and-publish-react-typescript-npm-package-with-demo-and-automated-build-80c40ec28aca)
