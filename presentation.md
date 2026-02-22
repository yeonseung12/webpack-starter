# Webpack 5 프론트엔드 개발환경 구성

## 왜 빌드 도구가 필요한가?

브라우저는 **HTML, CSS, 순수 JS**만 이해한다.

| 개발할 때 쓰고 싶은 것 | 브라우저가 이해하는 것 |
|---|---|
| JS 모듈 (import/export) | 하나로 합쳐진 bundle.js |
| 최신 JS 문법 (ES6+) | 구형 브라우저 호환 ES5 |
| SCSS | 일반 CSS |

> Webpack = 여러 JS 파일과 리소스(CSS, 이미지 등)를 분석해서 브라우저가 읽을 수 있는 하나의 번들 파일로 묶어주는 **모듈 번들러**
> 
> 개발자 코드 → 브라우저가 읽을 수 있는 형태로 변환·번들링

---

## webpack.config.js 란?

파일 번들 시 webpack이 할 일을 설정해주는 파일입니다.

## webpack.config.js 구조

```js
module.exports = {
  entry: './src/index.js',   // 1. 시작점
  output: { ... },           // 2. 결과물 저장 위치
  module: { rules: [...] },  // 3. 파일별 처리 규칙 (loader)
  plugins: [...],            // 4. 추가 기능
  devServer: { ... },        // 5. 개발 서버 설정
  mode: 'development',       // 6. 빌드 모드
};
```

---

## 1. entry — 시작점

```js
entry: './src/index.js'
```

Webpack이 파일을 읽기 시작하는 진입점.
`index.js`에서 `import`된 모든 파일을 따라가며 의존성 트리를 만들고, 연결된 파일들을 전부 하나의 `bundle.js`로 합친다.

```
src/index.js
  └── import './styles/main.scss'    ←┐
  └── import './components/header.js' ←┤  전부 찾아서 하나로 묶음
        └── import './utils.js'      ←┘
                  ↓
              bundle.js
```

---

## 2. output — 결과물 저장

```js
output: {
  path: path.resolve(__dirname, 'dist'),
  filename: 'bundle.js',
  clean: true,
}
```

| 옵션 | 설명 |
|------|------|
| `path` | 결과물을 저장할 폴더 (`dist/`) — 절대 경로로 지정해야 함 |
| `filename` | 번들 파일명 |
| `clean: true` | 빌드 전 `dist/` 폴더 자동 초기화 |

---

## 3. module.rules — 파일별 처리 규칙

Webpack은 기본적으로 JS만 이해한다.
다른 파일(SCSS 등)은 **loader**를 통해 JS로 변환한 뒤 번들에 포함시킨다.

`rules` 배열에 규칙을 정의하면, webpack이 파일을 만날 때마다 `test`에 매칭되는 규칙을 찾아 해당 loader를 실행한다.

```
파일 발견 → test 패턴과 대조 → 매칭된 loader 실행 → 변환된 결과를 번들에 포함
```

### JS — Babel loader

```js
{
  test: /\.js$/,
  exclude: /node_modules/,
  use: 'babel-loader',
}
```

| 옵션 | 설명 |
|------|------|
| `test` | 어떤 파일에 적용할지 (정규식) |
| `exclude` | 제외할 대상 |
| `use` | 사용할 loader |

`babel-loader`는 webpack과 Babel을 연결해주는 다리 역할이다.
실제 변환은 `@babel/core` + `@babel/preset-env`가 수행하고, `.babelrc`에서 변환 규칙을 설정한다.

```
webpack이 .js 파일 발견
  → babel-loader 호출
    → @babel/core + @babel/preset-env가 실제 변환 수행
```

최신 JS 문법(ES6+) → 구형 브라우저 호환 ES5로 변환

```
arrow function () => {}  →  function() {}
const, let               →  var
import / export          →  require / module.exports
```

### SCSS — 3단계 loader

```js
{
  test: /\.scss$/,
  use: ['style-loader', 'css-loader', 'sass-loader'],
}
```

> loader는 배열 **오른쪽 → 왼쪽** 순서로 실행된다. (함수 합성 방식: `style(css(sass(file)))`)

```
sass-loader       →    css-loader                    →    style-loader
SCSS → CSS 변환        JS에서 CSS를 import할 수 있게 해줌    <style> 태그를 생성해 CSS 내용을 페이지에 반영
```

---

## 4. plugins — HtmlWebpackPlugin

`plugins`는 Webpack 빌드 과정에 **추가 기능을 주입**하는 설정입니다.
로더(loader)가 파일 변환을 담당한다면, 플러그인은 번들링 전반에 걸친 작업을 처리합니다.

```js
plugins: [
  new HtmlWebpackPlugin({
    template: './index.html',
  }),
]
```

루트의 `index.html`을 템플릿으로 삼아 빌드 시 `dist/index.html`을 자동 생성하고, 번들 파일 경로를 `<script>` 태그에 자동 삽입한다.

```
작성한 index.html          빌드 후 dist/index.html
────────────────────       ──────────────────────────────
<body>                     <body>
  <div id="app"></div>  →    <div id="app"></div>
</body>                        <script src="bundle.js"></script>
                           </body>
```

**참고 — 배포 환경에서 해시 파일명과 함께 쓸 때**

배포 시에는 파일명에 해시를 붙이는 것이 일반적이다.

```js
filename: 'bundle.[contenthash].js'
```

캐시 문제가 생기는 건 **프로덕션**에서다.

```
사용자가 브라우저로 접속 → bundle.js 캐싱
→ 개발자가 새 버전 배포 (파일명 동일)
→ 브라우저는 캐시된 구버전 사용
```

파일명에 해시를 붙이면 내용이 바뀔 때 해시도 바뀌어, 브라우저 입장에서 완전히 새로운 파일이 된다. 빌드마다 파일명이 달라지는 상황에서 HtmlWebpackPlugin이 그 파일명을 읽어 `<script>`에 자동으로 주입해준다.

개발 환경에서는 `webpack-dev-server`가 파일을 디스크에 저장하지 않고 메모리에서 직접 서빙하고, `hot: true`(HMR)로 변경사항을 WebSocket으로 브라우저에 바로 푸시한다. 브라우저가 캐시를 볼 틈이 없어 해시가 불필요하다.

> 이 프로젝트는 `filename: 'bundle.js'`로 고정 — 개발용 보일러플레이트라 별도 캐시 관리가 불필요하다.

---

## 5. devtool — 소스맵

번들링하면 여러 파일이 하나로 합쳐지고 압축되어, 에러가 나도 브라우저 콘솔이 이렇게 표시한다.

```
bundle.js:1  Uncaught TypeError: Cannot read property of undefined
```

소스맵은 번들된 코드와 원본 코드를 연결해, 원본 파일 기준으로 디버깅을 도와주는 매핑 파일이다. 소스맵이 있으면 브라우저가 에러 위치를 원본 기준으로 보여준다.

```
bundle.js:1  →  src/components/header.js:23
```

```js
devtool: 'eval-source-map'  // 개발 환경 추천 — 빠르고 정확
```

`mode: 'development'`만 설정해도 Webpack이 `devtool`을 `'eval'`로 자동 설정하지만, 정확한 줄 번호까지 보려면 명시적으로 지정하는 게 낫다.

---

## 6. devServer — 개발 서버

`webpack-dev-server`를 설치하면 `webpack.config.js`의 `devServer` 옵션이 활성화되어 로컬 개발 서버를 구성할 수 있다.

```js
devServer: {
  port: 3000,
  hot: true,
  open: true,
}
```

| 옵션 | 설명 |
|------|------|
| `port` | 개발 서버 포트 (`http://localhost:3000`) |
| `hot: true` | HMR(Hot Module Replacement) 활성화 — 새로고침하지 않고, 변경된 모듈만 교체, 페이지 상태 유지 |
| `open: true` | 서버 시작 시 브라우저 자동 열기 |

> `npm run dev`는 결과물을 **메모리에만** 올림 → `dist/` 폴더 생성 안 됨
> 실제 파일 확인은 `npm run build` 실행 필요

---

## 6. mode — 빌드 모드

`mode`를 통해 파일을 어떻게 번들링할 것인지 설정할 수 있습니다.

```js
mode: 'development'  // 또는 'production'
```

| 모드 | 설명 |
|------|------|
| `development` | 빠른 빌드, 디버깅용 소스맵 포함, 코드 압축 안 함 |
| `production` | 코드 압축·난독화, 불필요한 코드 제거, 파일 용량 최소화 |

`webpack.config.js`에 `development`로 고정되어 있어도, `npm run build` 실행 시 `--mode production`이 우선 적용됩니다.
→ 개발 시엔 `development`, 배포 시엔 `production`으로 자동 전환되는 구조입니다.

### 빌드 속도 차이

| | development | production |
|---|---|---|
| 빌드 속도 | 빠름 | 느림 |
| 이유 | 압축·최적화 없이 그대로 번들링 | Tree Shaking, 코드 압축, 모듈 통합 등 무거운 최적화 수행 |

> `npm run dev`가 코드 변경 시 빠르게 재빌드해서 HMR로 반영할 수 있는 것도 이 덕분이다.

---
