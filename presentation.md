# Webpack 5 프론트엔드 개발환경 구성

---

## 왜 빌드 도구가 필요한가?

브라우저는 **HTML, CSS, 순수 JS**만 이해한다.

| 개발할 때 쓰고 싶은 것 | 브라우저가 이해하는 것 |
|---|---|
| JS 모듈 (import/export) | 하나로 합쳐진 bundle.js |
| 최신 JS 문법 (ES6+) | 구형 브라우저 호환 ES5 |
| SCSS | 일반 CSS |

> Webpack = 개발자 코드 → 브라우저가 읽을 수 있는 형태로 변환·번들링

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
`index.js`에서 `import`된 모든 파일을 따라가며 의존성 트리를 만든다.

```
src/index.js
  └── import './styles/main.scss'
  └── import './components/header.js'
        └── import ...
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
다른 파일(SCSS 등)은 **loader**를 통해 처리한다.

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

```js
plugins: [
  new HtmlWebpackPlugin({
    template: './index.html',
  }),
]
```

빌드할 때마다 달라지는 번들 파일명을 자동으로 HTML에 주입해주어, 수동으로 관리할 필요가 없습니다.

**쓰는 이유**

1. 번들 파일명이 빌드마다 바뀜 (해시 포함)
   → 플러그인이 자동으로 올바른 파일명을 script 태그에 삽입

2. `index.html`은 템플릿일 뿐, 브라우저가 실행하는 건 `dist/index.html`
   → 플러그인이 템플릿 기반으로 `dist/index.html`을 새로 생성

```
작성한 index.html          빌드 후 dist/index.html
────────────────────       ──────────────────────────────
<body>                     <body>
  <div id="app"></div>  →    <div id="app"></div>
</body>                        <script src="bundle.js"></script>
                           </body>
```

---

## 5. devServer — 개발 서버

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
| `hot: true` | HMR(Hot Module Replacement) 활성화 — 변경된 모듈만 교체, 페이지 상태 유지 |
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

## 정리

**얻은 것**

- CRA, Vite 내부에서 무슨 일이 일어나는지 파악
- 의존성 버전 충돌 직접 해결 경험
- 새 프로젝트 시작 시 바로 복붙해서 쓸 수 있는 보일러플레이트 완성

**GitHub 저장소**
[github.com/yeonseung12/webpack-starter](https://github.com/yeonseung12/webpack-starter)
