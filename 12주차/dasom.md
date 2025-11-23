# Npm Deep Dive
Dasom, 2025.11.23

## 6장 자바스크립트 번들 도구 살펴보기

> 복습
> 번들러란? 웹 어플리케이션을 구성하는 다양한 리소스들을 모아서 하나 또는 여러개로 합치는 도구
> 필요했던 이유는? 코드 스플리팅, 트리 셰이킹 등을 통한 성능 상의 이점, 호환성 문제 해결 등

### 6.3 패키지 번들의 선두주자, 롤업

#### 6.3.1 롤업의 등장 배경과 소개

	•	웹팩이 등장한 2012년에는 SPA 환경에서 복잡한 웹 애플리케이션을 효과적으로 묶는 것이 주요 요구
	•	2015년에는 ES6와 함께 ESModule이 처음 제안되었고, CommonJS 기반의 모듈 방식이 가진 제한이 논의되기 시작
	•	웹팩도 초기에는 CommonJS 기반이라 동기적 모듈 로딩, 정적 분석 난이도, 트리 셰이킹 제약 등이 존재
	•	롤업은 ESModule을 기반으로 더 정적이고 평평한(flat) 번들을 만드는 데 집중
→ 모듈 오버헤드를 줄이고, 사용되지 않는 코드를 적극적으로 제거하는 방향

웹팩 vs 롤업 예시 비교

웹팩 번들 결과:
```js
// src/moduleA.js
export const greet = () => {
 console.log("hello from module A")
}

// src/moduleB.js
import {greet} from "./moduleA"

greet()
```

롤업 번들 결과:
```js
const greet = () => {
 console.log("hello from module A")
}

greet()
```
중간 모듈 래퍼 없이 하나의 파일로 통합되는 형태.

cf. 롤업과의 비교
	•	초기 롤업은 코드 분할이나 HMR 지원이 부족해서, 웹 애플리케이션 번들러로 사용하기에는 제약이 많았음
	•	반면 패키지·라이브러리처럼 “여러 모듈을 하나의 결과물로 제공하는 구조”에 강점
	•	현재는 경계가 많이 흐려진 상태
	•	웹팩은 ESModule을 안정적으로 지원
	•	롤업은 코드 분할과 HMR 기능을 강화하면서 웹 앱 번들링까지 확장


6.3.2 롤업의 기본 개념과 특징

6.3.2.1 ESModule 기반 번들링
	•	ESModule을 기반으로 정적 분석이 가능해짐
	•	모듈을 평탄화하면서 불필요한 래퍼 코드 제거
	•	트리 셰이킹 효과 극대화

6.3.2.2 다양한 출력 포맷 지원
	•	CommonJS, ESM, AMD, UMD, IIFE, SystemJS 등 다양한 포맷 생성 가능
	•	라이브러리 번들링에 유리
	•	웹팩보다 출력 포맷 선택 폭이 넓은 편


```js
// ESModule
const greet = () => {
 console.log("hello from module A")
}

greet()
```

```js
// AMD
define(function() {

  'use strict'

  const greet = () => {
 console.log("hello from module A")
}

greet()

})
```

```js
// CommonJS
'use strict'

const greet = () => {
 console.log("hello from module A")
}

greet()
```

```js
// IIFE
;(function() {
'use strict'
const greet = () => {
 console.log("hello from module A")
}

greet()
})()
```

```js
// UMD
;(function(factory){
  typeof define === 'function' && define.amd ? define(factory) : factory()
})(function() {
'use strict'
const greet = () => {
 console.log("hello from module A")
}

greet()
})
```

```js
// SystemJS
System.register([], function() {
'use strict'
return {
excute: function() {
const greet = () => {
 console.log("hello from module A")
}

greet()},
}
})
```

6.3.2.3 트리 셰이킹
	•	ESModule의 정적 구조를 활용한 코드 제거
	•	초기부터 트리 셰이킹을 핵심 기능으로 내세웠던 번들러

------

6.3.2.4 주요 필드

* input
 	•	단일 파일, 배열, 객체 형태 모두 지원 (결과물은 이후 output에 반영됨)
  ```js
  // rollup.config.js
  export default {
    input: "src/main.js", // 여기에 여러 방식으로 넣을 수 있는 것
  output: {...}
  }
  ```
  •	fileURLToPath를 사용하면 src 디렉터리 구조를 유지한 채 input을 자동 생성함
  
* external
	•	번들에 포함하지 않을 외부 의존성을 명시
	•	라이브러리 번들 크기 증가 방지
	•	패키지 사용자가 별도로 설치하도록 유도하는 용도

* output
	•	번들 결과물의 포맷, 파일명, 이름 지정
	•	name / globals 필드는 UMD·IIFE에서만 사용되는 특징적인 옵션

* plugins
	•	롤업 기능을 확장하는 구성 요소
	•	코드 변환, 압축, 파일 처리 등 다양한 작업 수행
	•	자주 사용하는 플러그인:
  	•	@rollup/plugin-terser: 코드 압축
  	•	@rollup/plugin-node-resolve: node_modules 해석
  	•	@rollup/plugin-commonjs: CJS → ESM 변환
  	•	@rollup/plugin-json: JSON 파일 처리
  	•	@rollup/plugin-postcss: CSS 및 전처리기 처리
  	•	@rollup/plugin-replace: 문자열/환경변수 대체
  	•	@rollup/plugin-babel: Babel 연동
  	•	@rollup/plugin-visualizer: 번들 분석


### 6.4 번들 도구의 신흥 강자, 비트

#### 6.4.1 비트의 등장 배경과 소개

	•	기존 번들러의 공통점: 대부분 JS로 작성
	•	JS는 런타임 타입 체크, JIT 기반 컴파일, 단일 스레드 이벤트 루프 등의 구조적 이유로 고성능 빌드 도구에 한계
	•	이러한 이유로 JS가 아닌 언어로 구현된 도구들이 등장
  	•	SWC(러스트 기반 JS/TS 컴파일러)
  	•	esbuild(Go 기반 번들러)
  	•	Parcel·Turbopack(러스트 기반)
	•	Vite 1.0.0에서는 다음 전략 사용
  	•	개발 모드: 브라우저 ESModule 기반 로딩
  	•	TS → JS 변환 및 의존성 사전 번들링: Go 기반 esbuild 사용
	•	cf. Vite는 자체는 JS/TS로 작성되어 있지만, 핵심 속도 요소는 esbuild와 Rollup을 조합해 얻는 구조

#### 6.4.2 비트의 기본 개념과 특징

6.4.2.1 롤업과 esbuild의 공존
	•	Vite는 “개발 단계”에서는 esbuild 중심
	•	“빌드 단계”에서는 Rollup을 사용
	•	모든 빌드를 esbuild로 처리하기에는 아직 기능 격차가 있어 두 도구를 조합
	•	최근에는 Rollup 팀의 Rust 기반 번들러인 Rolldown도 실험적으로 활용

6.4.2.2 의존성 사전 번들링
	•	개발 서버 첫 실행 시 node_modules 의존성을 esbuild로 빠르게 번들링
	•	필요한 이유:
	•	브라우저는 CommonJS를 이해하지 못함
	•	패키지마다 모듈 방식이 달라서 ESM 포맷으로 통일 필요
	•	ESM 모듈을 브라우저가 각각 요청하면 네트워크 요청 수가 과도하게 증가
	•	esbuild 속도가 매우 빠르기 때문에 사전 번들 비용이 거의 부담되지 않음
→ 개발 환경에서 ESM 기반의 빠른 모듈 로딩을 가능하게 하는 핵심 과정

6.4.2.3 모던 웹환경을 위한 도전
	•	Vite 6 버전부터는 내부 의존성뿐 아니라 외부 코드도 ESM을 권장
	•	CommonJS와 ESM을 동시에 유지하는 복잡도를 줄이고 ESM 중심 생태계로 이동
	•	CRA(create-react-app)를 대체하는 흐름이 이미 자리 잡은 상태
  

#### 6.4.3 설정에 필요한 주요 필드

root
	•	프로젝트의 최상위 디렉터리
	•	index.html과 정적 파일 제공 기준이 되는 경로

base
	•	배포 시 기본 경로 설정
	•	Next.js의 basePath와 유사
	•	환경 변수는 process.env가 아니라 import.meta.env로 접근

mode
	•	명령어에 따라 development / production 자동 설정
	•	사용자가 별도 모드를 지정해 환경 분리 가능
	•	NODE_ENV 대비 유연한 구조

define
	•	빌드 시점에 코드 내부에서 사용할 전역 상수 정의
	•	웹팩의 DefinePlugin과 동일한 용도

plugins
	•	롤업 플러그인과 높은 호환성
	•	대부분의 rollup 플러그인을 그대로 사용 가능

build
	•	build.target: 빌드 타깃 브라우저 설정
	•	build.outDir: 출력 디렉터리
	•	build.minify: 압축 여부
	•	build.lib: 라이브러리 모드 빌드
	•	build.rollupOptions: Rollup 설정을 그대로 전달해 커스터마이징


