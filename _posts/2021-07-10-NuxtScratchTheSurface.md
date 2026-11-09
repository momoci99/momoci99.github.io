---
title: "Nuxt 찍먹하기"
tags:
  - Nuxt
---

Nuxt.js.. 살짝만 찍어먹어봅시다.

# 개요

React.js의 프레임 워크인 Next.js에서 영감을 받아 Vue.js에서 보다 쉽게 Vue.js를 사용할 수 있게 만든 프레임 워크입니다. SSR/SSG or SPA 방식으로 Vue 어플리케이션을 만들 수 있게 해줍니다.

# 어플리케이션 생성

Nuxt는 3가지 방법으로 설치 할 수 있습니다.

Yarn을 사용하는 방법

```bash
yarn create nuxt-app <project-name>
```

npx를 사용하는 방법

```bash
npx create-nuxt-app <project-name>
```

npm을 사용하는 방법

```bash
npm init nuxt-app <project-name>
```

## 설치과정

1. 프로젝트 명 설정

```powershell
? Project name:
```

2. 언어 설정

```powershell
? Programming language: (Use arrow keys)
❯ JavaScript
  TypeScript
```

3. package manager 설정

```powershell
? Package manager: (Use arrow keys)
❯ Yarn
  Npm
```

4. UI Framework 설정

```powershell
? UI framework:
❯ None
  Ant Design Vue
  BalmUI
  Bootstrap Vue
  Buefy
  Chakra UI
  Element
  Framevuerk
  Oruga
  Tachyons
  Tailwind CSS
  Windi CSS
  Vant
  View UI
  Vuetify.js
```

5. Nuxt.js 모듈 설정

```powershell
Nuxt.js modules: (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯◯ Axios - Promise based HTTP client
 ◯ Progressive Web App (PWA)
 ◯ Content - Git-based headless CMS
```

7. Linting Tools 설정

```powershell
? Linting tools: (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯◯ ESLint
 ◯ Prettier
 ◯ Lint staged files
 ◯ StyleLint
 ◯ Commitlint
```

8. 테스트 도구 설정

```powershell
? Testing framework: (Use arrow keys)
❯ None
  Jest
  AVA
  WebdriverIO
  Nightwatch
```

9. 렌더링 방식 설정

```powershell
? Rendering mode: (Use arrow keys)
❯ Universal (SSR / SSG)
  Single Page App
```

10. 배포 방식 설정

```powershell
? Deployment target: (Use arrow keys)
❯ Server (Node.js hosting)
  Static (Static/Jamstack hosting)
```

11. 개발 도구 설정

```powershell
❯◯ jsconfig.json (Recommended for VS Code if you're not using typescript)
 ◯ Semantic Pull Requests
 ◯ Dependabot (For auto-updating dependencies, GitHub only)
```

12. CI 설정

```powershell
❯ None
  GitHub Actions (GitHub only)
  Travis CI
  CircleCI
```

13. 버전 컨트롤 설정

```powershell
? Version control system: (Use arrow keys)
❯ Git
  None
```

13. 설정 완료

```powershell
🎉  Successfully created project test-nuxt

  To get started:

        cd test-nuxt
        yarn dev

  To build & start for production:

        cd test-nuxt
        yarn build
        yarn start

  To test:

        cd test-nuxt
        yarn test
```

위에 설치과정에서 저는 아래의 설정을 사용하였습니다.

- JavaScript
- Yarn
- Ant Design Vue
- Axios - Promise based HTTP client
- Progressive Web App (PWA)
- Content - Git-based headless CMS
- ESLint
- Jest
- Universal
- Server
- jsconfig.json
- GitHub Actions (GitHub only)
- Git

## 생성된 Nuxt 프로젝트 구조

```powershell
├── README.md
├── components
├── content
├── jest.config.js
├── node_modules
├── nuxt.config.js
├── package.json
├── pages
├── plugins
├── static
├── store
├── test
└── yarn.lock
```

## 프로젝트 구동

로컬 구동(개발)

```powershell
yarn run dev
```

프로덕션 배포용

```powershell
yarn run build
yarn run start
```

# SSR & SPA

Nuxt는 Server Side Rendering 혹은 Single Page Application 두가지 방식으로 앱을 생성할 수 있습니다.

위에서 보았던 프로젝트 생성 과정 중 Rendering mode 선택과정에서 방식을 설정 할 수 있습니다.

```powershell
? Rendering mode: (Use arrow keys)
❯ Universal (SSR / SSG)
  Single Page App
```

## SSR(Universal)방식으로 설정

SSR방식은 기본값이므로 큰 특이사항이 없습니다.

```jsx
export default {
  // Global page headers: https://go.nuxtjs.dev/config-head
  head: {
    title: 'test-nuxt',
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: 'description', name: 'description', content: '' },
      { name: 'format-detection', content: 'telephone=no' }
    ],
    link: [
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
    ]
  },

  // Global CSS: https://go.nuxtjs.dev/config-css
  css: [
    'ant-design-vue/dist/antd.css'
  ],

  // Plugins to run before rendering page: https://go.nuxtjs.dev/config-plugins
  plugins: [
    '@/plugins/antd-ui'
  ],
```

## SPA(Single Page Application)방식으로 설정

SPA로 설정시 SSR방식에 비해 다른점이 생깁니다.

1. nuxt.config.js에서 `ssr` 속성 `false`

   ```jsx
   export default {
     // Disable server-side rendering: https://go.nuxtjs.dev/ssr-mode
     ssr: false, //false로 명시적으로 설정

     // Global page headers: https://go.nuxtjs.dev/config-head
     head: {
       title: 'test-nuxt-spa',
       meta: [
         { charset: 'utf-8' },
         { name: 'viewport', content: 'width=device-width, initial-scale=1' },
         { hid: 'description', name: 'description', content: '' },
         { name: 'format-detection', content: 'telephone=no' }
       ],
       link: [
         { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
       ]
     },
   ```

2. build 실행시 /dist 디렉토리에 빌드 결과물이 생성됩니다.

   ```jsx
   .
   ├── README.md
   ├── components
   ├── content
   ├── dist
   ├── jest.config.js
   ├── jsconfig.json
   ├── node_modules
   ├── nuxt.config.js
   ├── package.json
   ├── pages
   ├── static
   ├── store
   ├── test
   └── yarn.lock
   ```

### Nuxt에 미들 웨어 추가하기

[https://blog.rhostem.com/posts/2018-10-28-nuxtjs-universal](https://blog.rhostem.com/posts/2018-10-28-nuxtjs-universal)

### Nuxt SPA앱 생성

[https://khwan.kr/blog/vue/2020-03-02-nuxt-spa-ssr/](https://khwan.kr/blog/vue/2020-03-02-nuxt-spa-ssr/)
