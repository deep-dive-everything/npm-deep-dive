## 프로젝트 환경 설정

- npm 레지스트리에서 패키지에 사용할 스코프(organization) 등록
  - npm 레지스트리: https://www.npmjs.com/
- Github에서 패키지 소스코드 관리용 저장소(repository) 생성
  - .gitignore → Node
  - 라이선스 → MIT License
- 로컬에 저장소 클론 후 패키지 작업 시작

### package.json 생성

- $ pnpm init으로 package.json 생성
- 기본 생성된 파일 기반으로 name, license, author 등의 주요 필드 수정
  - $schema: package.json 작성 시 자동 완성과 유효성 검사를 도와주는 역할
  - version: 1.0.0 → 0.1.0으로 시작하는 것을 권장
  - packageManager: `pnpm@10.3.0`으로 설정

### browserslist 작성

- 최상위 .browserslistrc 파일에 쿼리 정의: [> 1%, not dead](https://browsersl.ist/#q=%3E+1%25%2C+not+dead%0A)
- 추후에 번들도구와 연동하여 사용 확인

### 코드 스타일을 일관되게 유지하기 위한 각종 패키지 설정

- ESLint: $ pnpm add @naverpay/eslint-config -D
  - 미리 정의된 ESLint 룰 사용 (@naverpay/eslint-config)
  - .eslintrc에 extends로 추가
  - .eslintignore → 빌드 결과물(dist/), node_modules 폴더(node_modules/) 제외
  - package.json에 lint, lint:fix 스크립트 추가
- prettier: $ pnpm add @naverpay/prettier-config -D
  - 미리 정의된 prettier 룰 사용 (@naverpay/prettier-config)
  - .prettierrc에 추가
  - .prettierignore에 제외할 폴더 및 파일 추가
  - package.json에 prettier, prettier:fix 스크립트 추가
- markdownlint: $ pnpm add @naverpay/markdown-lint -D
  - 미리 정의된 markdownlint 룰 사용 (@naverpay/markdown-lint)
  - .markdownlint.jsonc에 extends로 추가
  ```json
  {
    "extends": "@naverpay/markdown-lint",
    "MD013": false,
    "MD024": { "siblings_only": true },
    "MD053": false
  }
  ```
  - package.json에 md, md:fix 스크립트 추가
- EditorConfig: $ pnpm add @naverpay/editorconfig -D
  - 코드 편집기의 설정을 통일하는 도구
  - 미리 정의된 EditorConfig 룰 사용 (@naverpay/editorconfig)
  - .editorconfig 파일을 생성하고 룰 추가
- Lefthook: 깃 훅 관리 도구 → go로 만들어짐
  - 깃 저장소에서 특정 이벤트가 발생할 때 자동으로 실행되는 스크립트
  - pre-commit, post-commit, pre-push 등

## 실제 코드 작성

- package.json에 필요한 의존성 설치하기
- ./src/types/index.ts에 필요한 타입 정의
- ./src/utils/index.ts에 HTMLElement에 필터를 입히는 getFilter 정의
- ./src/react.tsx로 React용 컴포넌트 선언
- ./src/next.tsx로 Next용 컴포넌트 선언
- ./src/index.ts로 앞에서 선언한 파일을 모두 모아서 다시 내보내기 → 배럴 파일

### 배럴 파일(barrel file)

**정의**

- 여러 모듈을 하나의 파일로 묶어서 다시 내보내는(export) 파일

```jsx
export * from './next'
export * from './react'
export * from './utils'
export * from './types'
```

**장점**

배럴 파일이 없다면 일일이 exports 필드에 각 모듈을 선언한 다음,
각 모듈을 다음과 같이 별개의 import 문으로 불러와야 했을 것이다.

```jsx
import { add } from 'date-fns/add'
import { addBusinessDays } from 'date-fns/addBusinessDays'
```

그러나 배럴 파일 덕분에 다음과 같이 일반적인 방식으로 불러올 수 있게 된다.

```jsx
import { add, addBusinessDays } from 'date-fns'
```

이처럼 배럴 파일은 여러 모듈의 진입점을 하나로 모을 수 있다는 특징 덕분에 여러 유틸의 진입점을 하나로 모으고 싶은 자바스크립트 패키지에서 많이 사용되는 패턴이다. 이렇게 함으로써 코드를 더욱 깔끔하고 관리하기 쉽게 만들 수 있다.

**단점**

- export 파일 간의 충돌 발생 가능성

```jsx
// ./test/a.js
export const data = 'a'

// ./test/b.js
export const data = 'b'

// ./test/index.js
export * from './a.js'
export * from './b.js'
```

- 모듈의 출처를 파악하기 어려움
- export default는 별도의 처리 필요

```jsx
// ./test/a.js
export const hello = 'hello'

export default function hi() {
  console.log('hi')
}

// barrel file
// ./test/index.js
export * from './a.js'
export { default as hi } from './a.js'

// ./index.js
import { hello, hi } from './test/index.js'

console.log(hello, hi()) // hello hi
```

- 다른 모듈의 부수 효과가 격리되지 않음
  - 배럴 파일에서 a와 b 파일을 모두 불러올 때 해당 모듈의 전체 코드를 실행
  - 배럴 파일은 불러온 파일을 **`모두 실행하는 방식으로 import** → **다시 export하는 방식**`으로 동작
- 트리 셰이킹의 불편함 (다만 오래된 문제라서 번들러마다 다양한 방법을 사용해 트리 셰이킹 시도)
  - 롤업 → 트리 셰이킹 `smallest` 설정
  - 웹팩 → package.json에 `sideEffects: false` 추가
  - Next.js → `optimizePackageImports`나 `modularizeImports` 사용

**배럴 파일의 모범사례**

- 모듈 내부의 부수 효과 최소화
  - 모듈 자체에서 부수 효과가 있을 경우 트리 셰이킹의 효과가 떨어지고 분석이 어려워짐
  - 각 모듈은 최대한 서로 부수 효과가 없는 상태로 격리되어 작성되는 것이 좋음
- package.json의 exports 필드 활용
  - 배럴 파일이 없더라도 `import a from 'library/a'` 방식으로 불러와서 사용 가능
  - 패키지를 사용하는 사용자들에게도 완벽하게 모듈이 격리되어 트리 셰이킹되어 쓸 수 있음
- 몇 가지 선택된 모듈만 내보내기
  - 각 파일에서 내보내는 파일을 철저하게 통제하거나,
  - 내보내기에 필요한 모듈만 선택적으로 내보내는 것이 좋음

## 번들 및 트랜스파일하기 위한 환경 구축

### build.lib를 설정해 라이브러리(패키지) 작성 시작하기

- vite.config.mts에 build.lib 필드 선언
- build.lib.entry 필드에 index.html 대신 사용할 시작점 선언
  - 객체를 선언해서 각 파일과 이 파일의 output으로 만들어 낼 파일명 지정
- ESModule과 CommonJS를 모두 지원하는 패키지를 만들기 위해 formats 필드 선언

### 타입스크립트의 경로 별칭 선언

- resolve.alias와 tsconfig.json 연동
  → tsconfig.json의 paths를 string으로 변환하는 우회적인 방법 필요
- `✔︎` vite-tsconfig-paths 플러그인 사용

### rollupOptions.external로 필요한 코드만 빌드하기

- build.rollupOptions.external 옵션을 사용해 빌드 제외 지정
  - react, next, react/jsx-runtime, next/image
  - 여기에 react/jsx-runtime, next/image 같은 서브 패스까지 번들에서 제외 필요
- package.json 필드를 읽어서 subpath까지 자동으로 처리할 수 있게 변경
  - package.json의 peerDependenciesd의 키(패키지명)를 전부 가져온 뒤,
  - 해당 의존성과 subpath 모두 external에 등록

```jsx
rollupOptions: {
	external: [...Object.keys(pkg.peerDependencies)].flatMap(dep => [
		dep,
		new RegExp(`^${dep}/.*`),
	]),
},
```

### 빌드 경고 해결하기 - 지시자 유지를 위한 플러그인 설치

```jsx
src/react.tsx (1:0): Module level directives cause errors when bundled, "use client" in "src/react.tsx" was ignored.
src/next.tsx (1:0): Module level directives cause errors when bundled, "use client" in "src/next.tsx" was ignored.
```

- rollup-preserve-directives → 지시자 유지를 위한 플러그인
  - 해당하는 파일이 어떤 인터프리터를 요구하는지 표현하는 셔뱅과 자바스크립트 지시자를
    빌드 결과물까지 유지해서 사용할 수 있음
  - 코드에서는 ‘use client’ 같은 지시자 유지 가능

### CommonJS와 ESModule 폴더별로 구별하기

- rollupOptions.output에 ‘es’, ‘cjs’ 별로 output dir 설정

```jsx
rollupOptions: {
	output: [
		{
			format: 'es',
			dir: 'dist/esm',
		},
		{
			format: 'cjs',
			dir: 'dist/cjs',
		},
	],
}
```

빌드 결과를 전후 비교하면 다음과 같다.

```jsx
./dist
├── index.js
├── index.js.map
├── index.mjs
├── index.mjs.map
├── next.js
├── next.js.map
├── next.mjs
├── next.mjs.map
├── react.js
├── react.js.map
├── react.mjs
├── react.mjs.map
├── utils.js
├── utils.js.map
├── utils.mjs
└── utils.mjs.map

0 directories, 16 files
```

```jsx
./dist
├── cjs
│   ├── index.js
│   ├── index.js.map
│   ├── next.js
│   ├── next.js.map
│   ├── react.js
│   ├── react.js.map
│   ├── utils.js
│   └── utils.js.map
└── esm
    ├── index.mjs
    ├── index.mjs.map
    ├── next.mjs
    ├── next.mjs.map
    ├── react.mjs
    ├── react.mjs.map
    ├── utils.mjs
    └── utils.mjs.map

2 directories, 16 files
```

### 리액트의 JSX를 자바스크립트 코드로 변환하기

- esbuild가 JSX 지원을 네이티브로 하기 때문에 기본적인 것들은 JSX → JS 코드로 변환 가능
- esbuild에서 지원하지 않는 고급 기능을 이용하고 싶은 경우 리액트 관련 플러그인 사용
  - @vite/plugin-react
  - @vite/plugin-react-swc
- 두 플러그인은 사용할 수 있는 옵션에만 약간의 차이가 있고, 사용하는 방법은 동일

**@vite/plugin-react**

- JSX 변환 시에 esbuild 위주로 사용해서 변환하는 플러그인
- 바벨 기반 HMR 지원 및 기본 확장자 jsx, tsx 외에 mdx와 같은 다른 확장자도 지원 가능
- 클래식 런타임으로도 변환 가능 (React.createElement 방식)
- 해당 플러그인이 esbuild가 아닌 바벨을 사용하는 경우
  - 바벨 관련 특별한 설정 값을 넘겨주는 경우
  - 리액트의 HMR 기능을 사용하는 경우
  - 개발 모드에서 클래스 런타임 방식으로 JSX를 변환하는 경우
- 바벨을 이용하기 때문에 @vite/plugin-react-swc보다 속도가 상대적으로 느림

**@vite/plugin-react-swc**

- SWC를 이용해 JSX를 변환하는 플러그인
- 빠른 속도가 장점
- 속도가 빠르지만 JSX를 변환할 때 17.x부터 제공되는 automatic 방식으로밖에 변환 불가
- 내부적으로 SWC를 사용하고 있어 바벨을 이용한 커스터마이징 불가

### browserslist를 활용한 트랜스파일 타깃 설정

**browserslist-to-esbuild**

- browserslist를 읽어서 esbuild.target이 이해할 수 있는 문법으로 고쳐주는 역할

```jsx
import browserslistToEsbuild from 'browserslist-to-esbuild'

export default defineConfig({
  build: {
    target: browserslistToEsbuild(), // browserslist와 target 매칭
  },
})
```

### 바벨과 core-js를 활용해 폴리필 삽입하기

- Vite는 바벨과 core-js와 바로 연동할 수 있는 방법이 마땅치 않아 롤업을 통해 폴리필 주입
- 이때 롤업에서 바벨을 사용할 수 있는 롤업 플러그인 @rollup/plugin-bable 사용
  → Vite는 대부분의 롤업 플러그인 사용 가능해서 Vite에서도 사용 가능

**@rollup/plugin-babel**

롤업에서 바벨을 사용할 수 있는 플러그인

```jsx
babel({
	babelHelpers: 'runtime',
	plugins: [
		['@babel/plugin-transform-runtime'],
		[
			'babel-plugin-polyfill-corejs3',
			{
				method: 'usage-pure',
				version: pkg.dependencies['core-js'],
				proposals: true,
			},
		],
	],
}),
```

- 폴리필 삽입을 위해 dependencies에 core-js-pure 추가
  - 이또한 롤업이 번들링하지 않게 rollupOptions.external에 추가

이렇게 하고 빌드를 하면 다음과 같은 빌드 결과물을 얻을 수 있다.

```jsx
// dist/esm/utils.mjs
import a from 'core-js-pure/features/instance/match-all.js'
import i from 'core-js-pure/features/instance/at.js'
import o from 'core-js-pure/features/instance/find-last.js'
```

### 폴리필이 원하는 대로 잘 들어가지 않아요

- [ ] 폴리필이 필요 없는 코드인데 자꾸 폴리필이 추가돼요
  → 정적 분석 시점에 오해할 수 있는 변수명은 사용하지 않는 것을 권장
- [ ] 브라우저 지원 범위 내에 있는 명세의 폴리필도 포함돼요
  → 과거 proposal 버전과 표준 적용 버전이 혼재되면서 발생한 문제라 종종 발생할 수 있는 문제
- [ ] babel-plugin-polyfill-corejs3이 폴리필을 삽입하게 하고 싶지 않다면
  → babel-plugin-polfyfill-corjs3의 shouldInjectPolyfill 옵션 사용
- [ ] 폴리필이 필요한데 폴리필이 추가되지 않아요
  → core-js 버전이 해당 폴리필을 지원하지 못하는 낮은 버전인지 확인하기
  → core-js 버전이 아직 해당 폴리필을 지원할 준비가 되어 있지 않은 경우인지 확인하기

### 타입스크립트 사용자를 위한 타입 제공 (`.d.ts`)

- 타입스크립트 또는 플러그인을 활용한 일반적인 타입 생성
  - tsc 사용
  - `✔︎` vite-plugin-dts 플러그인 사용
- `✔︎` tsup → 타입 파일 생성용으로 사용
  - esbuild를 기반으로 만들어진 패키지
  - 타입스크립트 기반으로 작성된 패키지를 손쉽게 번들링할 목적으로 만들어진 패키지

**devDependencies에 vite-plugin-dts 의존성 추가 및 plugins 필드에 추가**

```jsx
dts({ outDir: 'dist/types', rollupTypes: true }),
```

- outDir: 생성한 타입 파일을 어느 폴더에 저장할지
- rollupTypes: build.lib.entry당 하나의 타입 파일 생성

### package.json과 번들된 구조 맞추기

- (1) main과 module 필드 선언
  - main → cjs, module → esm으로 번들링된 파일을 가리킴
  - types 필드에는 타입 파일이 위치한 폴더 가리키게 설정
- (2) exports 필드 선언 (듀얼 패키지 지원을 위해 가장 중요) ([링크](https://github.com/yceffort/ndive-react-image/blob/main/package.json#L9C5-L50C7))
- (3) 최종 구성 파일과 번들링된 결과물 확인7

### 왜 리액트 버전은 18 이상으로 설정했을까?

- 리액트 17의 subpath 이슈로 인해 react/jsx-runtime을 찾을 수 없는 이슈
  - 리액트 18 버전에서는 exports가 추가되면서 해결됐으니 18버전으로 설정!
- 리액트 18을 사용하지 않으면서 위의 문제를 해결하는 방법
  - pnpm patch로 react 패키지를 수정해서 해결
  - 웹팩의 resolve.alias로 모듈에 별칭 지정해서 사용

### 간단한 테스트 코드 작성

패키지가 Vite 기반이므로 vitest로 테스트 코드 작성힌다.

- vite 설정과 플러그인을 그대로 이어받아 테스트를 수행할 수 있음
- Jest와 호환되는 API를 제공하기 때문에 사용하기 편리함
- 타입스크립트와 ESModule를 매끄럽게 지원
- vite와 통합되어 있어 테스트 환경 설정이 간단하고 빠르게 동작
- 구성 파일도 간단하게 작성 가능해 개발 편의성 상대적으로 높음

### 깃허브 액션을 활용한 CI 파이프라인 구축

- .github/workflows/ci.yaml 파일 작성

**workflow 작성 용어**

- name: 워크플로 이름 → 깃허브 액션에서 워크플로를 식별하는 역할
- on: 워크플로 실행 시점
- jobs: 워크플로에서 수행할 일 → setup, build, lint
- runs-on: 해당 job에 실행될 머신을 정의
- steps: 해당 job에서 순차적으로 실행할 작업 리스트

### changesets를 활용한 배포

코드를 안정적으로 배포하는 데 필요한 모든 기능을 재공하는 도구다.

**changeset-bot 설치하기**

- https://github.com/apps/changeset-bot

**npm 토큰 발급**

- 패키지 배포 시 사용하는 인증 토큰으로, 이 토큰을 통해 npm 레지스트리에 패키지 배포 가능

**npm 토큰을 저장소에 저장**

- Repository secrets 기능으로 github 저장소에 필요한 비밀 값을 저장

**저장소 코드에 changesets 설정**

- changesets/cli 설치
-
