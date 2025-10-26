# Npm Deep Dive
Dasom, 2025.10.21

## 4장 CommonJS와 ESModule

### 4.4 Node.js는 어떻게 node_modules에서 패키지를 찾아갈까?
4.2, 4.3절을 통해 각 모듈 시스템이 모듈을 어떻게 내보내고 가져오는 지 알아보았다.
4.4절을 통해서는 Node.js가 내부 모듈부터 외부 모듈까지 어떻게 로드하는지 구체적인 방식을 살펴본다.

#### 4.4.1 모듈 해석 알고리즘
**모듈 해석 알고리즘:** Node.js가 모듈을 로드하고 의존성을 해결하는 일련의 과정
즉, Node.js가 require()나 import를 만났을 때, “이 모듈이 실제로 어떤 파일을 가리키는지” 찾는 과정
각 모듈 시스템별 해석 알고리즘을 알아보기 전, 모듈 식별을 위한 모듈 지정자부터 학습

**4.4.1.1. 모듈 지정자**

Node.js의 모듈 지정자는 require()함수, import문에서 특정 모듈을 식별하는 문자열
종류
* 상대 경로 지정자: `import Entry from "./components/Entry"` (현재 파일 위치를 기준으로 하는 경로)
* 모듈 이름 지정자: `import * as $ from "jquery"` (주로 node_modules 폴더 내의 패키지나 내장 모듈을 함초할 때 사용)
* 절대 경로 지정자: `const config = require("file:///opt/nodejs/config.js")` (파일 시스템의 최상위에서 시작하는 경로)

cf. URL지정자: `import {Octokit} from "https://cdn.skypack.dev/octokit"` (웹에서 호스팅 되는 라이브러리 접근 때 사용, 이 지정자는 Node.js의 모듈 해석 알고리즘이 아닌 **표준적인 URL 경로 해석 방식**으로 해결)

**4.4.1.2. 표준 상대 경로 URL 해석 문법**

지정된 기본 URL을 기준으로, 상대 경로를 절대 경로로 변환하는 규칙과 방법. (WHTATWG -Web Hypertext Application Technology Working Group-에서 정의한 표준을 따름)

> e.g. 기본 URL: https://example.com/docs/example일 때
> * 상대URL ./api -> https://example.com/docs/example/api
> * 상대URL ../api -> https://example.com/docs/api
> * 상대URL /api -> https://example.com/api

**4.4.1.3. 하위 경로 불러오기와 하위 경로 내보내기**

(보다 쉬운 설명을 위해 책 내용이 아닌 방식으로 설명)

**하위 경로 불러오기**는 패키지 내부 경로를 직접 가져오는 행위. (e.g. `import { format } from 'date-fns/format';`)

그런데, 위 상황에서 패키지 내부 구조가 바뀌면 위 코드는 깨지게 됨. 즉, 사용자 입장에서 내부 경로에 의존하면 위험하기 때문에 Node.js는 하위 경로 내보내기 규칙을 도입함.

**하위 경로 내보내기**는 package.json의 "exports"필드를 이용해 패키지 외부에서 접근가능한 경로를 명시적으로 정의함.
```json
// package.json
{
  "name": "my-lib",
  "type": "module",
  "exports": {
    ".": "./src/index.js",
    "./utils": "./src/utils/index.js"
  }
}
```
위와 같이 정의하면

```js
// 가능
import myLib from 'my-lib';           // -> ./src/index.js
import utils from 'my-lib/utils';     // -> ./src/utils/index.js

// 불가능
import something from 'my-lib/src/utils/index.js'; // 직접 접근 불가
```
즉, exports에 정의된 경로만 접근 가능하게 막아서 패키지 내부 구조를 바꿔도 외부 사용자는 영향을 받지 않음.


#### 4.4.2 모듈 이름 지정자로 모듈을 로드하는 방법

Node.js 문저를 통해서 CommonJS와 ESModule의 모듈 해석 알고리즘을 상세히 알 수 있음.

링크1: https://nodejs.org/api/modules.html#all-together

링크2: https://nodejs.org/docs/latest-v18.x/api/esm.html#resolution-and-loading-algorithm

(이 부분 역시 더 쉬운 설명을 위해 일부 요약 및 생략, 각색함)

**4.4.2.1. CommonJS에서 모듈을 찾는 방법**

CommonJS는 동기적으로 모듈을 탐색하고, 파일 시스템 기반으로 순차적으로 탐색

탐색 순서 요약

1.	내장(core) 모듈 확인
•	예: fs, path, http 등 Node.js가 기본 제공하는 모듈이면 바로 반환.

2.	파일 경로 확인
•	'./', '../', '/'로 시작하면 로컬 파일 경로로 해석.
•	.js, .json, .node 순서로 확장자 시도.

3.	디렉터리 탐색
•	폴더를 지정한 경우, 해당 폴더의 package.json에서 "main" 필드 확인.
•	없으면 index.js 로드.

4.	node_modules 탐색
•	현재 디렉터리 → 상위 디렉터리 순으로 역탐색하며 node_modules 폴더 검색.

5.	캐싱 처리 (require.cache)
•	한 번 불러온 모듈은 캐시에 저장되어 재사용.

> 예시.
> const express = require('express');
> → Node.js는 다음 순서로 탐색:
>	1.	core 모듈인지 확인
>	2.	./express.js 있는지 확인
>	3.	./node_modules/express/ 폴더 찾기
>	4.	express/package.json의 "main" 확인
>	5.	없으면 express/index.js 로드

**4.4.2.2. ESModule에서 모듈을 찾는 방법**

ESModule은 비동기적으로 동작하며, URL(또는 절대 경로) 기반 해석을 사용. 

탐색 순서 요약

1.	절대 / 상대 경로 확인
•	'./', '../', '/'가 없으면 “패키지 이름(import specifier)”으로 인식.
•	ESM은 반드시 확장자(.js/.mjs/.json) 명시해야 함. 
(자동 확장자 보완 X)

3.	패키지 해석
•	package.json의 "exports" 또는 "main" 필드 사용.
•	"exports"가 있으면 그 규칙을 우선함.

5.	폴더 지정 시
•	CommonJS처럼 index.js 자동 로드하지 않음.
•	"exports"에서 직접 지정되어야 함.

7.	node_modules 탐색
•	CommonJS와 유사하지만, "exports" 필드를 반드시 따름.

9.	URL 기반 경로 해석
•	내부적으로 import.meta.url을 기준으로 상대 경로를 계산.

> 예시.
> const express = require('express');
> → Node.js는 다음 순서로 탐색:
>	1.	node_modules/express/package.json 찾기
>	2.	"exports" 필드가 있다면 해당 경로 사용
>	3.	없다면 "main" 필드 확인
>	4.	명시된 파일 로드 (확장자 필수)

#### 4.4.3 정리

CommonJS와 ESModule은 기본적인 모듈 해석 방식에서 중요한 차이가 있기 때문에 이 둘을 혼용할 경우 예상치 못한 문제를 일으킬 수 있음.
특히, CommonJS는 동적 로딩을 허용하나 ESModule은 그렇지 않음. 
따라서 개발 시 어떤 모듈 시스템을 선택할지 신중한 고려가 필요함.


### 4.5 CommonJS와 ESModule, 무엇이 정답일까?

서버사이드에서는 주로 CommonJS가 사용되고 클라이언트 사이드에서는 ESModule을 사용하는 경우가 많음.
하지만 웹과 서버에서 모두 동작해야하거나 다양한 환경을 고려하는 경우 어떤 모듈을 선택할지 신중하게 고려애햐 함.

#### 4.5.1 오픈소스 패키지가 CommonJS와 ESModule을 동시에 지원하는 이유

> 이유: 다양한 환경(서버·브라우저·번들러)을 모두 호환하기 위해서

여러 환경에서 사용하는 상황에서는 서버 측 코드는 CommonJS로, 클라이언트 측 코드는 ESModule로 빌드해 두 환경 분리 가능. 

하지만 코드가 서버와 클라이언트 양쪽에서 실행되는 경우가 있어 각각 분리하는 경우 런타임 에러가 발생할 수 있어, 두 가지 방식 모두 지원하는 것이 중요함.

이 경우 외에도 코드의 재사용성이나 다양한 개발환경 지원을 위해 두가지 방식 모두 지원하는 것이 중요함.

#### 4.5.2 CommonJS와 ESModule을 동시에 지원하는 듀얼 패키지 개발하기 

라이브러리가 두 가지 모듈 시스템을 동시에 지원할 수 있도록 패키지를 개발하는 방법

package.json설정 예시
```json
{
  "name": "my-lib",
// main, module 필드를 각각 추가해 각 파일 경로 명시
  "main": "./dist/index.cjs",     // CommonJS 진입점
  "module": "./dist/index.mjs",   // ESModule 진입점
// 조건부 내보내기
  "exports": { 
    "import": "./dist/index.mjs", // ESM용
    "require": "./dist/index.cjs" // CJS용
  }
}
```

-> Node.js는 require()면 "require" 경로를, 브라우저나 번들러가 import면 "import" 경로를 자동 선택

#### 4.5.3 순수한 ESModule 패키지 개발하기

하지만 패키지 개발자 입장에서 실제로 이중 패키지를 지원하는 것은 간단하지 않음. 

복잡성으로 인해 오로지 ESModule로만 패키지를 지원하려는 흐름도 늘고 있음.

**이중 패키지 위험성**
	•	코드 중복 (파일 2벌 유지)
	•	import/require 간 상태 공유 문제
	•	서로 다른 런타임에서 의도치 않은 버그 발생
	•	빌드나 번들러 설정이 복잡해짐

**위험을 줄이는 두가지 방법**
1.  ESModule 래퍼(Wrapper) 사용

CommonJS 파일 위에 ESModule 형태의 래퍼 파일을 얹는 방식.

```json
"exports": {
  "import": "./esm-wrapper.mjs",
  "require": "./index.cjs"
}
```
→ 실제 코드는 CommonJS지만, ESModule 환경에서는 .mjs 래퍼를 통해 접근.

2. 진입점 분리

CommonJS와 ESModule을 아예 별개의 진입점으로 구분

```json
"exports": {
  ".": {
    "import": "./esm/index.js",
    "require": "./cjs/index.js"
  }
}
```
→ 각 모듈이 완전히 독립되어 충돌을 피함.

하지만, 이런 식으로 이중 패키지 위험을 해결하더라도
패키지 사이즈가 커진다는 문제가 생기고, ESModule로 전환을 하는 움직임이 늘었다고 하더라도 이 과정이 험난하다는 문제는 여전히 존재함.

**순수한 ESModule로 개발/전환하기**

```json
{
  "type": "module"
}
```
-> Node.js가 .js 파일을 ESModule로 해석하게 만듦

이 외 기존 CommonJS(require, module.exports) 코드는 모두 import/export로 바꿔야 하며, TypeScript도 compilerOptions.module을 esnext로 변경 필요.

단, 구형 Node.js 버전(12 미만)은 지원 불가하며 CommonJS환경과 호환성 이슈 발생 가능


#### 4.5.4 CommonJS와 ESModule, 무엇이 정답일까?

**주장 1. CommonJS가 자바스크립트를 해치고 있다**
* CommonJS의 동기적 모듈 로딩 방식은 브라우저 환경에서 비효율적이다.
* 모듈을 최적화하기 어려워 사용하지 않는 모듈을 제거하며 번들크기를 최소화하기 힘들다.
* 브라우저에서 기본적으로 지원되지 않아 웹팩, Rollup, Parcel 같은 번들러와 트랜스파일러가 필요하다.

**주장 2. CommonJS는 사라지지 않을 것이다**
* CommonJS는 ESModule모다 모듈을 로드하고 시작하는 속도가 더 빠르다.
* 점진적 로딩이 가능하다.
* 수많은 CommonJS 패키지와 프로젝트가 여전히 존재한다.


#### 4.5.5 정리

선택은 패키지 작성자의 몫이다.
필자는 ESModule이 모듈 시스템의 통일을 이끌어갈 것이라 예상한다.
