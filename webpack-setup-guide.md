# Webpack 개발환경 설정 문서

## 큰 그림 이해하기

브라우저는 HTML, CSS, JS만 이해한다. 그러나 개발 시에는:

- JS를 **여러 파일로 쪼개서** 모듈로 관리하고
- CSS 대신 **SCSS** (변수, 중첩 등 편의기능이 있는 CSS 확장 문법)를 사용하고
- 코드 스타일을 **자동으로 검사/정리**할 필요가 있다

Webpack은 이런 파일들을 **브라우저가 이해할 수 있는 형태로 변환하고 묶어주는 도구**다.

```
                  ┌──────────┐
JS 여러 파일 ──→  │          │──→ bundle.js (합쳐진 JS 하나)
SCSS      ──→  │  Webpack  │──→ style (CSS로 변환)
이미지    ──→  │          │──→ 최적화된 이미지
                  └──────────┘
```

---

## 단계별 상세 설명

### 1단계 — `npm init` (프로젝트 초기화)

```
프로젝트 폴더
  └── package.json  ← 이 단계에서 생성되는 파일
```

`package.json`은 프로젝트의 **설명서** 역할을 하는 파일이다.

- 프로젝트 이름, 버전
- 사용 중인 패키지(라이브러리) 목록
- `npm run dev` 같은 명령어 단축키 (scripts)

이 파일이 없으면 npm이 패키지를 관리할 수 없다.

```bash
npm init -y
```

> `-y` 옵션은 모든 질문에 자동으로 yes를 입력한다. 이후 `package.json`을 직접 수정할 수 있다.

---

### 2단계 — Webpack 설치

```bash
npm install --save-dev webpack webpack-cli webpack-dev-server html-webpack-plugin
```

| 패키지                | 역할                                                       |
| --------------------- | ---------------------------------------------------------- |
| `webpack`             | 핵심 엔진                                                  |
| `webpack-cli`         | 터미널에서 webpack 명령어를 실행할 수 있게 해주는 CLI 도구 |
| `webpack-dev-server`  | 파일 저장 시 브라우저 자동 새로고침                        |
| `html-webpack-plugin` | 번들된 JS를 HTML에 자동으로 `<script>` 태그로 삽입         |

> **`--save-dev` 란?**
>
> npm 패키지는 두 종류로 구분된다.
>
> | 종류            | 옵션                     | 설명                             | 예시                   |
> | --------------- | ------------------------ | -------------------------------- | ---------------------- |
> | 서비스용 패키지 | `npm install`            | 실제 서비스에도 필요             | React, Vue, lodash     |
> | 개발용 패키지   | `npm install --save-dev` | 개발할 때만 필요, 배포 시 불필요 | Webpack, Babel, ESLint |
>
> `package.json`에는 다음과 같이 구분되어 기록된다.
>
> ```json
> {
>   "dependencies": {
>     "react": "^18.0.0"     ← 서비스에도 필요
>   },
>   "devDependencies": {
>     "webpack": "^5.0.0"    ← 개발할 때만 필요
>   }
> }
> ```
>
> **Webpack이 배포 후에 필요 없는 이유**
>
> Webpack은 파일을 "변환하고 묶어주는" 도구다. `bundle.js`를 생성하고 나면 그 역할은 완료된다.
> 브라우저는 완성된 `bundle.js`만 읽으면 되므로 Webpack은 더 이상 필요하지 않다.
>
> ```
> Word로 문서 작성 → PDF로 저장 → PDF를 전송
> → 받는 사람은 Word 없이도 PDF를 읽을 수 있다.
>   Word는 "만들 때"만 필요한 도구다. Webpack도 동일하다.
> ```
>
> 배포 시에는 `dist/` 폴더(완성된 결과물)만 서버에 올리면 된다. Webpack은 서버에 설치할 필요가 없다.

> **`node_modules` 란?**
>
> `npm install`을 실행하면 설치한 패키지들이 `node_modules/` 폴더에 저장된다.
> 패키지 하나가 또 다른 패키지에 의존하기 때문에, 직접 설치한 것보다 훨씬 많은 폴더가 생성된다.
>
> ```
> node_modules/
>   ├── webpack/
>   ├── webpack-cli/
>   ├── html-webpack-plugin/
>   └── ... (수백 개)    ← webpack이 의존하는 패키지들까지 전부 설치됨
> ```
>
> - `package.json`이 "어떤 패키지가 필요한지 목록"이라면, `node_modules`는 "실제 파일들"이다.
> - `package-lock.json`은 "정확히 어떤 버전을 설치했는지 기록"이다.
> - `node_modules`는 용량이 매우 크기 때문에 `.gitignore`에 추가해서 git에는 올리지 않는다.
>   대신 `package.json`만 공유하면, 다른 사람이 `npm install`로 동일한 환경을 재현할 수 있다.

---

### 3단계 — Babel 설치 및 설정 (JS 변환)

```
우리가 쓰는 최신 JS 문법  →  Babel  →  구형 브라우저도 이해하는 JS
  import / export                        var, function...
  화살표 함수 () => {}                    function() {}
```

최신 JS 문법(ES6+)을 **구형 브라우저도 읽을 수 있도록** 변환한다.

```bash
npm install --save-dev babel-loader @babel/core @babel/preset-env @babel/eslint-parser
```

| 패키지                 | 역할                                                    |
| ---------------------- | ------------------------------------------------------- |
| `@babel/core`          | Babel 핵심                                              |
| `@babel/preset-env`    | "얼마나 구형 브라우저까지 지원할지" 결정하는 설정 묶음  |
| `babel-loader`         | webpack이 JS 파일을 만날 때 Babel에게 넘겨주는 연결고리 |
| `@babel/eslint-parser` | Babel 문법(최신 JS)을 ESLint가 이해하도록 연결          |

> **`@babel/eslint-parser` 사용 시점**
>
> ESLint 기본 파서가 해석하지 못하는 문법이 있을 때만 필요하다.
>
> | 문법                           | 기본 파서 | `@babel/eslint-parser` |
> | ------------------------------ | --------- | ---------------------- |
> | `import`, `const`, 화살표 함수 | ✅        | 불필요                 |
> | JSX (`<h1>안녕</h1>`)          | ❌        | 필요                   |
> | 실험적 문법 (데코레이터 등)    | ❌        | 필요                   |
>
> JSX는 JS 표준 문법이 아니다. React에서만 쓰는 확장 문법으로, 기본 파서는 이를 문법 오류로 처리한다.
> `@babel/eslint-parser`를 사용하면 Babel이 먼저 해석한 결과를 ESLint에 전달해 오류를 방지한다.
>
> 현재 프로젝트는 표준 ES6+ 문법만 사용하므로 설치는 했지만 `eslint.config.js`에 등록하지 않았다.
> React 추가 시 아래와 같이 설정한다.
>
> ```javascript
> import babelParser from '@babel/eslint-parser';
>
> {
>   languageOptions: {
>     parser: babelParser,
>     parserOptions: {
>       requireConfigFile: false,
>       babelOptions: { presets: ['@babel/preset-react'] },
>     },
>   },
> }
> ```

설치 후 `.babelrc` 파일을 생성한다.

```json
{
  "presets": ["@babel/preset-env"]
}
```

> **`.babelrc`가 필요한 이유**
>
> Babel을 설치했다고 자동으로 변환이 시작되지 않는다. "어떻게 변환할지"를 별도로 지정해야 한다.
>
> ```
> Babel = 번역가
>
> 번역가를 고용했더라도
> "어떤 언어로 번역해줘?" 라고 지시하지 않으면 아무것도 할 수 없다.
>
> .babelrc = 번역 지시서
> "최신 JS → 구형 브라우저용 JS로 변환할 것" 이라고 명시하는 파일
> ```
>
> 이 파일이 없으면 Babel이 설치되어 있어도 아무 변환도 수행하지 않는다.
> 결과적으로 구형 브라우저에서 최신 JS 문법을 읽지 못해 오류가 발생한다.
>
> React를 사용하게 되면 한 줄만 추가하면 된다.
>
> ```json
> {
>   "presets": ["@babel/preset-env", "@babel/preset-react"]
> }
> ```
>
> 이와 같이 **필요한 변환 규칙을 추가해가는 설정 파일**이다.

> **`.babelrc` vs `babel.config.js` 차이**
>
> |              | `.babelrc`           | `babel.config.js`     |
> | ------------ | -------------------- | --------------------- |
> | 적용 범위    | 현재 패키지만        | 프로젝트 전체         |
> | 형식         | JSON                 | JavaScript            |
> | 주로 쓰는 곳 | 단일 패키지 프로젝트 | 모노레포, 복잡한 설정 |
>
> `babel.config.js`는 JavaScript로 작성하기 때문에 조건문 등 JS 문법을 활용할 수 있다.
>
> ```js
> // babel.config.js - 환경에 따라 다르게 설정 가능
> module.exports = {
>   presets: [
>     process.env.NODE_ENV === 'test'
>       ? '@babel/preset-env'
>       : ['@babel/preset-env', { modules: false }],
>   ],
> };
> ```
>
> 단순한 단일 프로젝트라면 `.babelrc`로 충분하다.
> `babel.config.js`는 프로젝트가 복잡해지거나 Jest(테스트 도구)를 사용할 때 주로 활용한다.

---

### 4단계 — SCSS 설치 및 설정

```
main.scss  →  sass-loader  →  css-loader  →  style-loader  →  브라우저에 적용
(SCSS 문법)    (CSS로 변환)    (JS가 읽을 수   (HTML에
                               있게 처리)      주입)
```

> Webpack은 loader를 **오른쪽에서 왼쪽 순서**로 실행한다. 각 loader가 파일을 가공해서 다음 loader에게 전달하는 방식이다.

```bash
npm install --save-dev sass sass-loader css-loader style-loader
```

| 패키지         | 역할                                    |
| -------------- | --------------------------------------- |
| `sass`         | SCSS → CSS 변환 엔진                    |
| `sass-loader`  | webpack ↔ sass 연결                     |
| `css-loader`   | CSS를 JS가 처리할 수 있게 변환          |
| `style-loader` | 변환된 CSS를 HTML `<style>` 태그에 삽입 |

---

### 5단계 — ESLint 설치 및 설정 (코드 검사기)

```javascript
// 이런 실수를 자동으로 감지
cosnt name = 'Kim'  // ← "cosnt가 뭐야?" 오류 표시
let x = 1          // ← "x를 사용하지 않음" 경고
```

코드의 **버그 가능성**, **나쁜 패턴**을 미리 감지해주는 도구다.

```bash
npm install --save-dev eslint@9 @eslint/js@9 eslint-plugin-prettier
```

> **⚠️ 버전을 반드시 `@9`로 고정해야 하는 이유**
>
> `@eslint/js`를 버전 없이 설치하면 최신 버전(10.x)이 설치되는데,
> `eslint@10`은 앞서 설치한 `@babel/eslint-parser`와 충돌이 발생한다.
>
> ```
> @babel/eslint-parser (이미 설치됨) → eslint 8 또는 9 버전 필요
> @eslint/js@* (버전 미지정 → 최신)  → eslint 10 버전 필요
>
> → 서로 다른 eslint 버전을 요구 → ERESOLVE 에러 발생
> ```
>
> `eslint@9`와 `@eslint/js@9`로 버전을 맞추면 `@babel/eslint-parser`와 호환되어 충돌이 해소된다.
>
> ```bash
> # ❌ 버전 미지정 시 충돌 발생
> npm install --save-dev eslint @eslint/js eslint-plugin-prettier
>
> # ✅ 버전 고정 필요
> npm install --save-dev eslint@9 @eslint/js@9 eslint-plugin-prettier
> ```

설치 후 `eslint.config.js` 파일을 생성한다.

```javascript
import js from '@eslint/js';
import globals from 'globals';

export default [
  js.configs.recommended,
  {
    languageOptions: {
      ecmaVersion: 'latest',
      sourceType: 'module',
      globals: {
        ...globals.browser, // document, window 등 브라우저 전역 변수 허용
      },
    },
    rules: {
      'no-unused-vars': 'warn',
      'no-console': 'off',
    },
  },
];
```

> **`document` is not defined ESLint 오류**
>
> `src/index.js`에서 `document.getElementById()`를 사용하면 ESLint가 오류를 표시한다.
>
> **원인**
> ESLint는 기본적으로 코드가 어떤 환경에서 실행되는지 알지 못한다.
> `document`, `window`, `console` 같은 변수가 브라우저에서는 기본으로 제공되지만,
> ESLint 입장에서는 "선언된 적 없는 변수"로 처리된다.
>
> **해결**
> `globals` 패키지의 `globals.browser`를 추가하면 브라우저 환경의 전역 변수들을 허용할 수 있다.
>
> ```javascript
> import globals from 'globals';
> // ...
> globals: {
>   ...globals.browser, // document, window, console, localStorage 등 전부 허용
> }
> ```
>
> `globals.browser`는 내부적으로 다음과 같은 변수 목록을 포함한다.
>
> ```javascript
> {
>   document: 'readonly',
>   window: 'readonly',
>   console: 'readonly',
>   localStorage: 'readonly',
>   // ... 수십 개
> }
> ```

---

### 6단계 — Prettier 설치 및 설정 (코드 포맷터)

```javascript
// 정리 전
const obj = { name: 'Kim', age: 20, city: 'Seoul' };

// Prettier 적용 후
const obj = {
  name: 'Kim',
  age: 20,
  city: 'Seoul',
};
```

- ESLint는 코드의 오류와 문제를 감지하고, Prettier는 코드 스타일을 자동으로 정리한다.
- 둘이 **충돌할 수 있어서** `eslint-config-prettier`로 충돌을 방지해야 한다.

```bash
npm install --save-dev prettier eslint-config-prettier
```

설치 후 `.prettierrc` 파일을 생성한다.

```json
{
  "singleQuote": true,
  "semi": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80
}
```

---

### 7단계 — webpack.config.js 작성

모든 설정을 하나의 파일로 통합하는 단계다.

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  // 시작점: 여기서부터 읽기 시작
  entry: './src/index.js',

  // 결과물 저장 위치
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    clean: true, // 빌드할 때마다 dist 폴더 초기화
  },

  module: {
    rules: [
      // .js 파일은 Babel로 처리
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },
      // .scss 파일은 sass → css → style 순서로 처리
      // (배열 오른쪽부터 실행 — 함수 합성 방식: style(css(sass(file))))
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader'],
      },
    ],
  },

  plugins: [
    new HtmlWebpackPlugin({
      template: './index.html', // 기준이 되는 HTML 파일
    }),
  ],

  devServer: {
    static: './dist',
    port: 3000,
    hot: true, // 저장 시 자동 새로고침
    open: true, // 서버 시작 시 브라우저 자동 열기
  },

  mode: 'development', // 'production'으로 변경하면 코드 최소화
};
```

> **`open: true` 실행 시 SyntaxError 오류 (Node.js 버전 문제)**
>
> **오류 메시지**
>
> ```
> SyntaxError: The requested module 'node:fs/promises' does not provide an export named 'constants'
>     at ... wsl-utils/index.js
> ```
>
> **원인**
> `open: true`는 서버 실행 시 브라우저를 자동으로 열어주는 옵션이다.
> 이 기능을 처리하는 내부 패키지(`wsl-utils`)가 Node.js 18.4 이상에서만 동작하는 API를 사용한다.
> Node.js 16 이하에서는 해당 API가 존재하지 않아 오류가 발생한다.
>
> **해결 방법 (둘 중 하나 선택)**
>
> 1. Node.js를 18.4 이상으로 업그레이드 (근본적 해결)
>    ```bash
>    nvm install 20   # Node.js 20 LTS 설치
>    nvm use 20
>    ```
> 2. `open: false`로 임시 우회 (브라우저 자동 열기 비활성화)
>    ```javascript
>    devServer: {
>      open: false, // 브라우저 자동 열기 끄기
>    }
>    ```
>    이 경우 서버는 정상 실행되며, 브라우저에서 `http://localhost:3000`을 직접 열면 된다.

`package.json`의 scripts에 단축 명령어를 추가한다.

```json
{
  "scripts": {
    "dev": "webpack serve --mode development",
    "build": "webpack --mode production",
    "lint": "eslint src/**/*.js",
    "format": "prettier --write src/**/*.js src/**/*.scss"
  }
}
```

| 명령어           | 설명                           |
| ---------------- | ------------------------------ |
| `npm run dev`    | 개발 서버 실행 (자동 새로고침) |
| `npm run build`  | 최종 결과물 생성 (dist 폴더)   |
| `npm run lint`   | ESLint 코드 검사 실행          |
| `npm run format` | Prettier 코드 정리 실행        |

> **`npm run dev`는 `dist/` 폴더를 생성하지 않는다**
>
> |             | `npm run dev`               | `npm run build`   |
> | ----------- | --------------------------- | ----------------- |
> | 결과물 위치 | 메모리 (디스크에 파일 없음) | `dist/` 폴더 생성 |
> | 목적        | 개발 중 빠른 확인           | 배포용 파일 생성  |
> | 속도        | 빠름                        | 상대적으로 느림   |
>
> `npm run dev`는 빌드 결과를 메모리에만 올려두고 서빙하기 때문에 `dist/` 폴더가 생성되지 않는다.
> 사이드바에서 `dist/` 폴더를 확인하려면 `npm run build`를 실행해야 한다.
>
> `npm run 이름`은 `package.json`의 `scripts`에 등록된 이름만 실행 가능하다.
> `dev`, `start`, `serve` 중 어떤 이름을 사용해도 동작은 동일하며, 이름만 다를 뿐이다.

> **webpack.config.js ESLint 빨간줄 문제**
>
> **원인**
> `eslint.config.js`의 `sourceType: "module"` 설정이 전체 파일에 적용되면서,
> CommonJS 방식(`require`, `module.exports`)으로 작성된 `webpack.config.js`에 ESLint 오류가 발생한다.
>
> **해결**
> `eslint.config.js`에 `webpack.config.js` 전용 예외 설정을 추가한다.
>
> ```javascript
> {
>   files: ["webpack.config.js"],
>   languageOptions: {
>     sourceType: "commonjs",
>     globals: {
>       require: "readonly",
>       module: "writable",
>       __dirname: "readonly",
>     },
>   },
> },
> ```
>
> **CommonJS vs ESM**
>
> |             | CommonJS                       | ESM                                  |
> | ----------- | ------------------------------ | ------------------------------------ |
> | 문법        | `require()` / `module.exports` | `import` / `export`                  |
> | 적합한 경우 | 순수 webpack 프로젝트          | Nuxt, Next.js 등 ESM 기반 프레임워크 |
> | `__dirname` | 기본 제공                      | 직접 구현 필요                       |
>
> ESM 방식 선택 시 `__dirname`을 아래와 같이 직접 구현해야 한다.
>
> ```javascript
> import { fileURLToPath } from 'url';
> import path from 'path';
> const __dirname = path.dirname(fileURLToPath(import.meta.url));
> ```

---

### 8단계 — 소스 파일 구조 생성

최종 폴더 구조:

```
webpack-setting/
  ├── src/
  │    ├── index.js          ← JS 시작점
  │    └── styles/
  │         └── main.scss    ← SCSS 시작점
  ├── dist/                  ← 빌드 결과물 (자동 생성됨, git에 올리지 않음)
  ├── index.html             ← 기본 HTML
  ├── webpack.config.js      ← webpack 설정
  ├── .babelrc               ← Babel 설정
  ├── eslint.config.js       ← ESLint 설정
  ├── .prettierrc            ← Prettier 설정
  └── package.json           ← 프로젝트 설명서
```

기본 파일 내용:

**index.html**

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Webpack App</title>
  </head>
  <body>
    <div id="app"></div>
    <!-- bundle.js는 HtmlWebpackPlugin이 자동으로 삽입함 -->
  </body>
</html>
```

> **`index.html`에 `<script>` 태그를 직접 넣지 않는 이유**
>
> 작성하는 `index.html`은 **템플릿** 역할을 한다. 브라우저가 직접 실행하는 파일이 아니다.
>
> webpack이 빌드하면 `dist/` 폴더에 **새로운 `index.html`** 을 생성하는데,
> 이때 `HtmlWebpackPlugin`이 번들 파일을 `<script>` 태그로 자동 삽입한다.
>
> ```
> 작성한 파일                  webpack 빌드 후 dist/index.html
> ─────────────────────────    ─────────────────────────────────
> <body>                       <body>
>   <div id="app"></div>   →     <div id="app"></div>
> </body>                         <script defer src="bundle.js"></script>
>                              </body>
> ```
>
> 직접 작성하지 않는 이유:
>
> - 번들 파일명이 빌드마다 바뀔 수 있다 (`bundle.abc123.js` 같은 해시 포함)
> - 어떤 파일을 넣을지 webpack이 정확히 알고 있으므로 자동화하는 것이 적합하다
> - 실수로 경로를 잘못 입력하는 상황을 방지할 수 있다
>
> 브라우저는 작성한 `index.html`이 아니라 `dist/index.html`을 실행한다.
>
> **Webpack vs Vite — `<script>` 태그 처리 방식 비교**
>
> |                            | Webpack                              | Vite                            |
> | -------------------------- | ------------------------------------ | ------------------------------- |
> | `index.html`에 script 태그 | 직접 작성하지 않음 (플러그인이 삽입) | 직접 작성 (빌드 시 경로만 교체) |
> | 진입점                     | JS 파일 (`entry: './src/main.js'`)   | HTML 파일 (`index.html`)        |
>
> **Webpack:**
>
> ```
> 개발자가 하는 것       HtmlWebpackPlugin이 하는 것
> ──────────────────    ──────────────────────────────────
> index.html            src/main.js → bundle.js 번들링
> (script 태그 없음)  → index.html에 <script src="bundle.js"> 삽입
>                       → dist/index.html로 출력
> ```
>
> **Vite:**
>
> ```
> 개발자가 하는 것                Vite가 하는 것
> ──────────────────────────    ──────────────────────────────────────
> index.html에 직접 작성:       개발 시: 그 경로 그대로 브라우저에 제공
> <script type="module"       빌드 시: main.ts → assets/main-abc123.js
>   src="/src/main.ts">               로 번들링 후 경로만 교체
> ```
>
> Webpack은 JS 파일이 진입점이므로 HTML은 템플릿 역할만 하고,
> Vite는 HTML 파일 자체가 진입점이므로 `<script>`를 직접 명시한다.

**src/index.js**

```javascript
import './styles/main.scss';

const app = document.getElementById('app');
app.innerHTML = '<h1>Webpack 개발환경 완성!</h1>';
```

**src/styles/main.scss**

```scss
$primary-color: #333;

body {
  margin: 0;
  font-family: sans-serif;
  color: $primary-color;

  h1 {
    color: tomato;
  }
}
```

---

## 전체 설치 명령어 순서 요약

```bash
# 1. 프로젝트 초기화
npm init -y

# 2. Webpack
npm install --save-dev webpack webpack-cli webpack-dev-server html-webpack-plugin

# 3. Babel
npm install --save-dev babel-loader @babel/core @babel/preset-env @babel/eslint-parser

# 4. SCSS
npm install --save-dev sass sass-loader css-loader style-loader

# 5. ESLint
npm install --save-dev eslint @eslint/js eslint-plugin-prettier

# 6. Prettier
npm install --save-dev prettier eslint-config-prettier
```

---

## 흐름 정리

```
개발자가 코드 작성
  │
  ├── ESLint   → 코드 오류/나쁜 패턴 경고
  ├── Prettier → 코드 스타일 자동 정리
  │
  └── npm run dev 실행
        │
        └── Webpack이 실행
              │
              ├── src/index.js
              │     └── babel-loader → 최신 JS 문법을 구형 브라우저용으로 변환
              │           └── 여러 JS 파일을 하나로 합침 → dist/bundle.js 생성
              │
              ├── src/styles/main.scss
              │     ├── sass-loader  → SCSS를 일반 CSS로 변환
              │     ├── css-loader   → CSS를 JS가 처리할 수 있는 형태로 변환
              │     └── style-loader → 런타임에 <style> 태그로 HTML에 주입
              │
              ├── index.html (템플릿)
              │     └── HtmlWebpackPlugin
              │           ├── 템플릿을 복사해서 dist/index.html 생성
              │           └── <script src="bundle.js"> 자동 삽입
              │
              └── 브라우저가 dist/index.html 실행
                    ├── <script src="bundle.js"> 로드 → JS 동작
                    └── bundle.js 안의 style-loader → <style> 태그 삽입 → CSS 적용
```

---

## 한마디 정리

> **Webpack이 하는 일:**
> 여러 파일(JS, SCSS, 이미지 등)을 브라우저가 이해할 수 있는 형태로 변환하고, 하나로 묶어주는 도구.

| 입력         | 변환                                          | 출력                   |
| ------------ | --------------------------------------------- | ---------------------- |
| `index.js`   | Babel로 최신 JS → 구형 브라우저 호환          | `bundle.js` 안에 포함  |
| `style.scss` | sass-loader → css-loader → style-loader       | `bundle.js` 안에 포함  |
| `index.html` | HtmlWebpackPlugin이 `<script>` 태그 자동 주입 | `dist/index.html` 생성 |

결국 `src/` 폴더 안의 여러 파일들이 → `dist/bundle.js` 하나로 합쳐진 것이다.
