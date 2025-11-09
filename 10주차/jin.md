복습

- 폴리필 : 최신 웹 기술을 지원하지 않는 브라우저에서 최신 기능을 사용할 수 있도록 해주는 코드
- 트랜스파일 : 최신 JS 코드를 구 브라우저에서도 동작하도록 변환

# 5.3 최선의 폴리필과 트랜스 파일은 무엇일까?

뭘 지원하는지 명확히 알아야 컴팩트하게 구현해서 번들 크기 최소화해야 한다.

- 폴리필과 트랜스파일에 영향을 주는 요소
    - 브라우저 지원해야하는 범위, Node.js 버전 범위

## browserlist

- 어떤 버전으로 얼마나 지원할 지 정의하는 트랜스파일, 폴리필 작업의 사전 작업 도구
- 일관된 호환성을 위해 브라우저 및 Node.js 버전 설정을 공유할 수 있게 하는 도구
    - caniuse-lite의 정적인 정보 사용
        - caniuse : 브라우저가 표준을 언제부터 구현했는지 보여주는 오픈 소스 사이트
        - caniuse-db : caniuse의 원본 JSON 데이터
        - caniuse-lite : caniuse-db의 축약 버전 NPM 패키지로 업데이트가 잦음
- 사용처 : babel, eslint, auto prefixer, stylelint, webpack, paruel, rollup 등등
- 적용 방법
    - pakage.json 에 browserlist 필드에 명시
    - .browserlistrc 별도 파일 명시
- 작성 방법
    - [쿼리 문법](https://github.com/browserslist/caniuse-lite)
- 디버깅
    - [공식 플레이그라운드](https://browsersl.ist/)
    - .browserlistrc 파일 위치에서 CLI 명령어로 확인 가능

## core-js-compat

- browserlist로 설정한 지원 범위에 알맞는 폴리필만을 추가하는 패키지
- 바벨 플러그인으로 파생됐는데 browerlist로 특정 버전 명시하기 때문에 지원하지 않으면 아예 폴리필 코드를 만들지를 않음 → 번들 크기 최적화 가능
    - babel-plugin-polyfill-corejs3 사용을 공식적으로 추천 (다른 것은 호환성 문제 보유)

## 지원 범위 선정 기준

프로젝트 기획, 성격, 개발환경에 따라 다름

1. 사용자 기반
2. 프로젝트 고려
3. 최신 기능 활용 여부
4. 현재 기술 스택 고려

# 5.4 바벨과 core.js의 대안

왜? 성능 최적화가 중요한 경우 다른 선택지를 고려해야할 필요성이 최신 브라우저 개발 상황에서는 있음

## 트랜스파일러(babel) 대체

1. 타입스크립트 컴파일러 : 정적타입검사 + 트랜스파일
    1. tsconfig.json 에 target 필드에 버전 명시 (default es5 ⇒ 번들 크기 커지기 때문에 구형 브라우저 지원 목적이 아니라면 지정하는 것을 권장)
        1. 트랜스파일에 영향을 줄 수 있는 다른 필드 : module, lib, sourceMap, jsx 등
2. SWC : 빌드 속도가 중요한 경우 유용한 Rust 기반 고성능 트랜스파일러 
    1. .swcrc 파일 작성 
        1. jsc, module, env 등

| 구분 | **Babel** | **TypeScript Compiler (tsc)** | **SWC** |
| --- | --- | --- | --- |
| **핵심 역할** | 최신 JS → 하위 JS로 변환 (문법 중심) | TypeScript → JS로 컴파일 (+타입체크) | 최신 JS/TS → JS로 초고속 변환 (Rust 기반) |
| **언어 지원** | JavaScript (TS는 플러그인 필요) | TypeScript + 일부 JS | JavaScript + TypeScript |
| **작성 언어** | JavaScript (Node.js 기반) | TypeScript (자체) | Rust (Native 속도) |
| **속도** | 느림 (JS로 동작, AST 변환 많음) | 중간 (타입 검사로 느려질 수 있음) | 매우 빠름 (Rust + 병렬 처리) |
| **타입 검사 기능** | ❌ 없음 (단순 변환기) | ✅ 있음 (정적 타입 분석) | ⚠️ 없음 (변환만 가능, 타입검사는 tsc와 함께 써야 함) |
| **출력 제어 세밀도** | ✅ 매우 세밀 (plugin 기반, 커스텀 AST 가능) | ⚪ 단순 (옵션 중심) | ⚪ 비교적 제한적 (Babel보단 단순) |
| **플러그인/에코시스템** | ✅ 매우 풍부 (예: preset-env, plugin-proposal-* 등) | ⚪ 제한적 (타입 관련 중심) | ⚠️ 아직 적음 (Babel 호환 API는 있음) |
| **빌드 도구 연동성** | ✅ Webpack, Rollup, Vite, Metro 등 기본 지원 | ⚪ 주로 TS 프로젝트 전용 | ✅ Vite, Next.js, Turbopack, Rspack 등 최신 빌드 툴과 잘 맞음 |
| **폴리필 지원** | ✅ core-js, preset-env, browserslist | ❌ 없음 (문법만 변환) | ⚠️ 실험적(별도 플러그인 필요) |
| **커스텀 확장성** | ✅ 뛰어남 (AST 단계에서 변환 가능) | ⚪ 제한적 (transform 옵션 일부) | ⚪ 빠르지만 세밀한 커스터마이징은 어려움 |
| **주요 사용 사례** | Babel + preset-env로 레거시 브라우저 대응 빌드 | 순수 TS 프로젝트 빌드 | Next.js, SWC CLI, Rspack, Vite(SWC 옵션) |
| **장점 요약** | 플러그인 다양, 세밀 제어 가능 | 타입 검증 포함, 안정적 | 초고속, 현대적 빌드 환경에 최적 |
| **단점 요약** | 느리고 설정 복잡, JS 실행 의존 | 느리고 JS 변환 최적화 부족 | 플러그인 생태계 부족, 세부 변환 한계 |
| **대표 속도 비교 (대략)** | 1x (기준) | 0.8x | ⚡ 10~20x 빠름 (Rust 병렬 처리) |

## 폴리필(core-js 대체)

1. es-shims
2. polyfill.js

| 구분 | **core-js** | **es-shims** | **polyfill.js (polyfill-library/Polyfill.io)** |
| --- | --- | --- | --- |
| 지향점 | **원스톱, 광범위 커버리지** | **스펙 정밀·미세 단위 모듈** | **런타임 UA 감지로 자동 서빙** |
| 커버 범위 | ES5~ES202x 전역·정적·프로토타입 + Collections/Iterators/Symbol 등 (일부 Stage 제안 포함 가능) | 주로 확정 표준의 **개별 메서드 단위**(예: `array-includes`, `object.assign`) + `es-abstract` 기반 엄격 구현 | 요청한 기능 리스트를 **브라우저(UA)** 에 맞춰 동적으로 묶어 전달 |
| 주입 방식 | ① **엔트리(전역 패치)**: `core-js/stable` ② **자동 선별**: `@babel/preset-env(useBuiltIns)` 또는 `babel-plugin-polyfill-corejs3` ③ **포니필**: `core-js-pure/*` | **필요 모듈만 수동 import**(전역 패치/샴 or 포니필 패키지 선택) | `<script src="...polyfill.js?features=...">`처럼 **CDN/자체호스팅 스크립트**로 런타임 주입 |
| Browserslist 연동 | **강함** (preset-env / polyfill-corejs3 결합 시 타깃 브라우저 기반 **필요한 것만**) | **약함** (기본은 수동 판단·선택) | **미사용**. 대신 **UA 기반** 런타임 결정 |
| 전역 오염 | 기본 **전역 패치**, 필요 시 **pure(포니필)**로 회피 가능 | 패키지별로 **전역/포니필** 선택 | 기본 **전역 패치** 성격(글로벌에 붙임) |
| 번들 크기/성능 | 엔트리 전체 주입 시 큼 → **usage 모드**나 **pure 모듈**로 최소화 가능 | **정밀 선택**으로 작게 유지 쉬움(대신 관리 비용↑) | 네트워크 추가·TTFB 영향, 캐시 전략 중요. 빌드 없이 빠르게 적용 가능 |
| 자동화/툴 연동 | **아주 성숙**(Babel/Vite/Webpack 등) | 자동화 약함(개발자가 판단·선택) | 빌드 체인 **불필요**. SSR/레거시 빠른 임시 대응에 유용 |
| 스펙 정합성(코너케이스) | 높음(테스트 풍부). 제안 포함 시 변화 가능성 ↑ | **최고 수준**(ECMAScript 내부 연산까지 충실) | 서비스의 구현 수준에 의존. 주로 실용 호환성 지향 |
| 유지보수 난이도 | **낮음~중간**(원스톱 + 자동 선별) | **중간~높음**(무엇이 필요한지 직접 판단) | **낮음**(URL로 제어) ~ **중간**(호스팅/캐시/보안 고려) |
| 적합한 상황 | **일반 웹앱**: 폭넓은 호환 + **자동 최소 폴리필** | **라이브러리/SDK**: 전역 오염 최소 + **정밀 스펙 일치** | **빌드 없이 빠른 대응**, 레거시 브라우저 지원을 **런타임**에 해결하고 싶을 때 |
