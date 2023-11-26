---
layout: single
title: "eslint-plugin-import"
tag:
  - eslint
---

최근 회사 프로젝트에 eslint-plugin-import 도입 검토를 하기 위해 관련 내용을 정리하였습니다.

# eslint-plugin : \***\*eslint-plugin-import\*\***

[https://www.npmjs.com/package/eslint-plugin-import](https://www.npmjs.com/package/eslint-plugin-import)

장점

- 다양한 옵션들을 지원합니다

단점

- 다양한 옵션을 지원하는 만큼 설정이 복잡할 수 있습니다.
- 타 플러그인에 무겁습니다. unpacked size(**1.21 MB)**

import/sort rule은 require()/import 구문들 정렬하는데 필요한 강력한 옵션들을 제공합니다.

## groups:[array]

- import 하는 유형에 따라 그룹별로 정렬하는 옵션입니다.
- 기본값은 `["builtin", "external", "parent", "sibling", "index"]` 입니다.
- example
  ```jsx
  // 1. node "builtin" modules
  import fs from 'fs';
  import path from 'path';
  // 2. "external" modules
  import _ from 'lodash';
  import chalk from 'chalk';
  // 3. "internal" modules
  // (if you have configured your path or webpack to handle your internal paths differently)
  import foo from 'src/foo';
  // 4. modules from a "parent" directory
  import foo from '../foo';
  import qux from '../../foo/qux';
  // 5. "sibling" modules from the same or a sibling's directory
  import bar from './bar';
  import baz from './bar/baz';
  // 6. "index" of the current directory
  import main from './';
  // 7. "object"-imports (only available in TypeScript)
  import log = console.log;
  // 8. "type" imports (only available in Flow and TypeScript)
  import type { Foo } from 'foo';
  ```
  - builtin : node 기본 module
  - external : 외부 모듈 (like npm package)
  - internal : 내부 모듈 (like component)
  - parent : 디렉터리 경로가 상위인 경우
  - sibling: 디렉터리 경로가 형제 인
  - index: 디렉터리 경로가 현재 파일의 경로인 경우
  - object-imports : (typescript 한정) “object” import
  - type :(타입스크립트 한정) type import

# eslint-plugin : simple-import-sort

[https://www.npmjs.com/package/eslint-plugin-simple-import-sort](https://www.npmjs.com/package/eslint-plugin-simple-import-sort)

장점

- 복잡한 설정이 필요하지 않고 다른 의존성이 없습니다.
- 가볍습니다. unpacked size (**37.3 kB)**

단점

- 상세한 옵션이 부족합니다.
- 그룹별 정렬을 지원하지 않습니다.

설정 예제

```tsx
{
  "parserOptions": {
    "sourceType": "module"
  },
  "env": { "es6": true },
  "plugins": ["simple-import-sort", "import"],
  "rules": {
    "simple-import-sort/imports": "error",
    "simple-import-sort/exports": "error",
    "import/first": "error",
    "import/newline-after-import": "error",
    "import/no-duplicates": "error"
  }
}
```

eslint의 autofix 기능 사용을 위한 셋팅

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

# 적용을 위한 셋업

아래의 패키지 설치가 필요

- "@typescript-eslint/parser": "^5.62.0"
  - typescript 코드 parser
- "eslint-import-resolver-alias": "^1.1.2",
  - alias(별칭)을 인식하기 위한 패키지
- "eslint-import-resolver-typescript": "^3.6.1",
  - typescript alias를 인식하기 위한 패키지
- "eslint-plugin-import": "^2.29.0",
  - 핵심 패키지

## Example

### fail

```jsx
import _ from "lodash";
import path from "path"; // `path` 모듈은 `lodash` 보다 먼저 import 되어야함.

// -----

var _ = require("lodash");
var path = require("path"); // `path` `모듈은 'lodash` 보다 먼저 import 되어야함.

// -----

var path = require("path");
import foo from "./foo"; // `import` 구문은 `require` 구문 보다 먼저 와야함.
```

### pass

```jsx
import path from "path";
import _ from "lodash";

// -----

var path = require("path");
var _ = require("lodash");

// -----

// Allowed as ̀`babel-register` is not assigned.
require("babel-register");
var path = require("path");

// -----

// Allowed as `import` must be before `require`
import foo from "./foo";
var path = require("path");
```

# 지원하는 옵션들

- groups:[array] 그룹별 정렬
- pathGroups:[array of objects] : 그룹내의 패턴 매칭을 통해 우선순위 조정
- pathGroupsExcludedImportTypes : pathGroups 적용규칙의 예외처리.
- newlines-between: 그룹별로 개행문자를 넣을지, 금지할지 결정
- distinctGroup : 경로가 달라도 그룹화 수행
- alphabetize : 그룹별 알파벳 정렬 기준 설정

## groups:[array]

- import 하는 유형에 따라 그룹별로 정렬하는 옵션입니다.
- 기본값은 `["builtin", "external", "parent", "sibling", "index"]` 입니다.
- example
  ```jsx
  // 1. node "builtin" modules
  import fs from 'fs';
  import path from 'path';
  // 2. "external" modules
  import _ from 'lodash';
  import chalk from 'chalk';
  // 3. "internal" modules
  // (if you have configured your path or webpack to handle your internal paths differently)
  import foo from 'src/foo';
  // 4. modules from a "parent" directory
  import foo from '../foo';
  import qux from '../../foo/qux';
  // 5. "sibling" modules from the same or a sibling's directory
  import bar from './bar';
  import baz from './bar/baz';
  // 6. "index" of the current directory
  import main from './';
  // 7. "object"-imports (only available in TypeScript)
  import log = console.log;
  // 8. "type" imports (only available in Flow and TypeScript)
  import type { Foo } from 'foo';
  ```
  - builtin : node 기본 module
  - external : 외부 모듈 (like npm package)
  - internal : 내부 모듈 (like component)
  - parent : 디렉터리 경로가 상위
  - sibling: 디렉터리 경로가 형제
  - index: 디렉터리 경로가 현재 파일의 경로
  - object-imports : (typescript 한정) “object” import
  - type :(타입스크립트 한정) type import

## pathGroups:[array of objects]

- 경로명의 패턴을 통해 더 세부적인 순서를 정할 수 있습니다.

```jsx
{
  "import/order": ["error", {
    "pathGroups": [
      {
        "pattern": "~/**",
        "group": "external"
      }
    ]
  }]
}
```

- react 관련 import문을 다른 외부 라이브러리들 보다 더 위에 위치하고 싶은 경우에 사용합니다.

```jsx
{
  "import/order": ["error", {
    "pathGroups": [
		   {
				  "pattern": "react*",
				  "group": "external",
				  "position": "before"
			 }
    ]
  }]
}

```

| property       | required | type   | description                                                                   |
| -------------- | -------- | ------ | ----------------------------------------------------------------------------- |
| pattern        | x        | string | 해당하는 그룹 내에 매칭되는 문자열 패턴                                       |
| patternOptions |          | object | minimatch 의 옵션을 그대로 사용 (https://github.com/isaacs/minimatch#options) |
| group          | x        | string | 정의할 그룹의 이름                                                            |
| position       |          | string | pathGroup이 위치할 순서 정의                                                  |

## pathGroupsExcludedImportTypes

- pathGroups에 적용된 규칙에 예외를 적용하고싶은 import타입을 설정합니다.
- 기본값 : ["builtin", "external", "object"]
- 기본값때문에 경로 규칙이 제대로 동작하지 않는다면 빈 배열로 설정합니다.

```jsx
{
  "pathGroups": [
    {
      "pattern": "react*",
      "group": "external",
      "position": "before"
    }
  ],
  "pathGroupsExcludedImportTypes": ["react-hook-form"] //react-hook-from만 react* 규칙에서 제외한다.
}
```

- react-hook-from 라이브러리의 import문을 react로 시작하는 다른 라이브러리들(react, react-dom)과 분리하여 다른 외부라이브러리와 동일한 우선순위를 갖도록 하는 예제 코드입니다.

## distinctGroup

- pathGroup으로 나누어진 그룹을 합칠수 있게 해주는 옵션입니다
- 기본값 : `true`

```jsx
{
  "rules": {
    "import/order": [
      "error",
      {
        "groups": [
          {
            "pattern": "react",
            "group": "external"
          },
          {
            "pattern": "react-router-dom",
            "group": "external"
          },
          {
            "pattern": "./",
            "group": "internal"
          }
        ],
        "pathGroups": {
          "external": "before",
          "internal": "after"
        },
        "distinctGroup": false
      }
    ]
  }
}
```

예제

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from "react-router-dom";

const App = () => {
  return (
    <Router>
      <div>
        <h1>My App</h1>
        <p>Welcome to my React application.</p>
      </div>
    </Router>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

- React, ReactDOM, Router은 모두 path가 달라 분리되어야하나, distinctGroup : false로 인해 하나의 그룹으로 처리됩니다

## alphabetize

- 그룹내에서 알파뱃순으로 정렬하는 옵션
- 기본값은 `ignore`

| 옵션            | 기본값 | 설명                                        |
| --------------- | ------ | ------------------------------------------- |
| order           | ignore | asc(오름차순), desc(내림차순), ignore(무시) |
| caseInsensitive | false  | 대소문자 구분 옵션.                         |

# 참고했던 article & links

eslint-plugin-import에 관해 잘 정리되어있는 블로그 입니다.

[https://velog.io/@parksil0/eslint-plugin-import](https://velog.io/@parksil0/eslint-plugin-import)

옵션에 관한 상세한 정리

[https://tesseractjh.tistory.com/305](https://tesseractjh.tistory.com/305)

alias 이슈 해결 https://github.com/import-js/eslint-plugin-import/issues/2365

[https://github.com/import-js/eslint-plugin-import/issues/2365#issuecomment-1023773348](https://github.com/import-js/eslint-plugin-import/issues/2365#issuecomment-1023773348)

import/order

[https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md)

타입스크립트 resolver 에러

[https://github.com/import-js/eslint-import-resolver-typescript](https://github.com/import-js/eslint-import-resolver-typescript)
