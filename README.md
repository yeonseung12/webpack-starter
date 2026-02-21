# webpack-starter

Webpack 5 프론트엔드 개발환경 보일러플레이트

## 기술 스택

| 도구 | 역할 |
|------|------|
| Webpack 5 | 번들러 |
| Babel | 최신 JS → 구형 브라우저 호환 변환 |
| SCSS | CSS 전처리기 |
| ESLint | 코드 품질 검사 |
| Prettier | 코드 스타일 자동 정리 |

## 시작하기

```bash
# 의존성 설치
npm install

# 개발 서버 실행 (http://localhost:3000)
npm run dev

# 프로덕션 빌드
npm run build
```

## 명령어

```bash
npm run dev      # 개발 서버 실행 (파일 변경 시 자동 새로고침)
npm run build    # dist/ 폴더에 프로덕션 빌드 생성
npm run lint     # ESLint 코드 검사
npm run format   # Prettier 코드 정리
```

## 폴더 구조

```
webpack-starter/
├── src/
│   ├── index.js          # JS 진입점
│   └── styles/
│       └── main.scss     # SCSS 진입점
├── index.html            # HTML 템플릿
├── webpack.config.js     # Webpack 설정
├── .babelrc              # Babel 설정
├── eslint.config.js      # ESLint 설정
└── .prettierrc           # Prettier 설정
```
