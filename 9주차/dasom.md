# Npm Deep Dive
Dasom, 2025.11.09

## 5장 트랜스파일과 폴리필

### 5.1 트랜스파일을 도와주는 도구, 바벨

> 바벨은 최신 자바스크립트 코드를 구형 브라우저와 호환되는 코드로 변환해주는 대표적인 트랜스파일러로, 플러그인 시스템을 통해 유연하고 확장 가능한 코드 변환을 제공한다.

#### 5.1.1 바벨의 필요성

**5.1.1.1 트랜스파일러의 필요성**

* 2015년 ES6 발표로 자바스크립트에 새로운 기능과 문법이 도입
* 클래스, 모듈, 화살표 함수, Promise, async/await 등 편리한 기능들이 추가
* 구형 브라우저(IE11, Safari 9 등)에서 지원하지 않는 것이 문제
* 최신 코드를 구형 브라우저에서 실행 가능한 코드로 변환하는 트랜스파일러의 필요성 대두

**트랜스파일러란?**
* 한 버전의 언어를 다른 버전으로 변환해주는 도구
* 자바스크립트에서는 최신 문법을 구형 브라우저 환경에서 동작하도록 코드를 변환하는 것

**트랜스파일 예시**
```javascript
// ES6
const sum = (a, b) => a + b

// ES5
var sum = function sum(a, b) {
  return a + b
}
```

cf. ES6의 모든 명세가 트랜스파일 가능한 것은 아니다. 어떤 ES6 기능은 ES5에서 제공하는 기능만으로 변환이 불가능할 수도 있다.

**5.1.1.2 바벨의 등장**

* 바벨 이전에는 구글의 Traceur나 es6-shim 등이 사용됨
* 2014년 세바스찬 맥켄지가 6to5라는 이름으로 최초 공개
* 초기에는 ES6 코드를 ES5로 트랜스파일하는 것이 전부
* 이후 바벨로 이름을 변경하면서 다양한 자바스크립트 버전과 실험적 기능을 지원하는 다목적 트랜스파일러로 발전

**5.1.1.3 바벨의 특징**

**플러그인 시스템**
* 필요한 기능만 선택적으로 트랜스파일할 수 있어 유연성과 확장성 제공
* 각 플러그인이 독립적인 모듈로 처리되어 개별 업데이트 및 유지보수 가능
* 개발자가 필요한 기능을 직접 개발해서 바벨에 추가 가능

**ES6을 넘어 광범위한 문법 지원**
* 최신 ECMAScript 기능 외에 ESNext, JSX, 타입스크립트 등 다양한 언어 확장 기능 지원
* React는 JSX와 최신 JS 문법을 사용하기 때문에 트랜스파일이 필요했으며, Create React App이나 초기 Next.js는 Babel + Webpack 조합을 사용

**프리셋 제공**
* 여러 플러그인을 묶어둔 프리셋 기능으로 쉽게 적용 가능
* 대표적인 프리셋: @babel/preset-env(최신 JS 변환), @babel/preset-react(JSX 변환), @babel/preset-typescript(타입 제거)

**활발한 커뮤니티 지원**
* 빠른 업데이트와 새로운 기능 추가

#### 5.1.2 바벨의 동작 방식

**5.1.2.1 추상 구문 트리(Abstract Syntax Tree)**

* 소스코드의 구조를 트리 형태로 표현한 자료 구조
* 컴파일러와 인터프리터가 소스코드를 분석하고 변환하는 데 사용하는 핵심 개념
* 코드의 구문적 요소들을 계층으로 나타내어 각 노드가 코드의 특정 구문 요소를 표현

**구성 요소**
* 노드: 변수나 함수 선언문 등 코드의 구문 요소를 의미하는 기본 단위
* 자식 노드: 특정 노드의 하위 요소로 해당 구문 요소의 세부 항목 포함
* 최상위 노드: 전체 프로그램이나 코드 블록을 나타내는 트리의 최상위 노드

**5.1.2.2 바벨이 코드를 변환하는 과정**

바벨은 파싱, 변환, 출력 세 단계로 코드를 변환:

1. **파싱(Parsing)**: 소스 코드를 AST로 변환 (`@babel/parser`)
2. **변환(Transforming)**: AST를 수정해 새로운 AST로 변환 (`@babel/traverse`, 플러그인)
3. **출력(Generating)**: 변환된 AST를 코드로 다시 변환 (`@babel/generator`)

**@babel/core**
* 위 세 단계를 모두 포함해서 API로 제공하는 바벨의 핵심 패키지
* @babel/parser, @babel/traverse, @babel/generator를 의존성으로 포함

**모듈화의 장점**
* 필요한 기능만 선택적으로 사용 가능
* 각 패키지를 독립적으로 업데이트하고 유지보수 가능

#### 5.1.3 바벨 사용해보기

**5.1.3.1 바벨 구성 파일**

* babel.config.* 또는 .babelrc.* 형식으로 생성
* .json, .js, .cjs, .mjs 등 다양한 확장자 사용 가능
* package.json의 babel 필드에도 설정 가능

**주요 옵션**
* presets: 바벨의 프리셋을 설정
* plugins: 개별 기능이나 실험적 기능을 추가로 트랜스파일
* env: 환경별 설정을 다르게 정의
* targets: 코드가 실행될 환경 지정

**targets 필드**
* 코드가 실행될 환경을 지정해 그 환경에 맞는 자바스크립트 구문으로 변환
* 필요한 변환과 폴리필을 최소화하여 번들 크기를 줄이고 성능 최적화

**targets 설정 방법**
* browsers: `last 2 versions`, `> 0.25%`, `ie 11`, `not dead` 등의 키워드 사용
* node: 특정 Node.js 버전 지정 (`current` 또는 `14` 등)
* esmodules: ESModule을 지원하는 환경을 목표로 지정
* .browserslistrc 파일로도 목표 브라우저 지정 가능

**5.1.3.2 트랜스파일의 한계**

**트랜스파일 가능한 ES6 기능**
* 클래스, 화살표 함수, 템플릿 리터럴, 구조 분해 할당, 스프레드 연산자 등
* 이들은 ES5 명세의 조합으로 대체 가능

**트랜스파일만으로는 변환이 어려운 기능**
* Promise 객체
* Map, Set, WeakMap, WeakSet 같은 ES6 컬렉션 객체
* Array.prototype.includes, String.prototype.startsWith, Object.assign 같은 새로 추가된 메서드
* async/await

cf. 트랜스파일만으로는 해결되지 않는 기능들을 지원하기 위해 폴리필(polyfill)이 필요하다.

**5.1.3.3 번들러와 함께 사용하기**

**바벨을 단독으로 사용할 때의 문제점**
* 모듈 시스템 변환 문제: 브라우저는 require()와 module.exports를 지원하지 않음
* 최적화 문제: 코드 분할 미지원, 트리 셰이킹 불가, 자바스크립트 외 파일 관리 불가

**해결 방법**
* 실제 프로젝트에서는 바벨을 웹팩이나 롤업과 같은 모듈 번들러와 함께 사용

### 5.2 폴리필을 도와주는 도구 core-js

> 폴리필은 트랜스파일만으로는 해결되지 않는 기능들을 지원하기 위해 낮은 ES 문법만 사용 가능한 구형 브라우저 환경에서도 작동하도록 전역에 생성되는 메서드나 객체를 의미한다.

**Promise를 구형 브라우저에서 사용하기 위한 조건**
* 생성자 함수, then/catch/finally 메서드, 체이닝
* Promise.all, Promise.allSettled, Promise.race, Promise.resolve, Promise.reject

→ Promise는 낮은 ES 버전의 문법만으로는 완벽하게 대체하기 어려우므로, Promise의 모든 인터페이스와 메서드 구현을 포함한 동일한 객체를 전역에 새롭게 정의해야만 사용 가능

#### 5.2.1 core-js란 무엇인가

* 자바스크립트 폴리필 라이브러리 중 가장 인기 있고 범용적인 폴리필
* 최신 자바스크립트 표준에서 정의된 기능들을 브라우저 호환성을 고려해서 사용할 수 있게 해주는 도구

**core-js의 특징**
* 폴리필 제공: 최신 ECMAScript 기능에 대한 폴리필 제공
* 모듈화: 필요한 기능만 선택적으로 로드 가능
* ECMAScript 제안 단계의 기능까지 지원: 아직 표준으로 채택되지 않은 제안 단계 기능의 폴리필도 일부 제공

**5.2.1.1 core-js 적용하기**

* 현재 기준 core-js의 추천 버전은 3
* core-js/stable: ECMAScript 표준의 최신 버전 기준으로 폴리필 제공 (어떤 기능이 필요한지 모호한 경우)
* 특정 기능만 필요한 경우: 필요한 폴리필을 직접 import (불필요한 코드 로드 방지로 성능 최적화)

#### 5.2.2 바벨과 core-js

* 일반적으로 core-js는 바벨의 트랜스파일 기능과 함께 사용
* 바벨은 core-js를 의존성으로 포함하여 프리셋과 플러그인에서 폴리필 기능 설정 가능
* core-js를 개발자가 직접 설정하지 않아도 되므로 단독 사용보다 권장되는 방법

**5.2.2.1 @babel/preset-env에 core-js 설정하기**

* corejs 필드를 추가하여 core-js 폴리필 사용 정의
* useBuiltIns 옵션과 함께 사용해야 의미 있음

**설정 예시**
```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        corejs: {version: '3.39.0', proposals: false},
        useBuiltIns: 'usage',
        targets: {
          browsers: ['ie >= 11'],
        },
      },
    ],
  ],
}
```

**주요 옵션**
* corejs: 사용할 core-js의 버전과 폴리필 최적화 설정
* targets.browsers: 브라우저 환경 지정 후 필요한 폴리필만 포함
* useBuiltIns: 바벨이 폴리필을 어떻게 처리할 것인지 동작 모드 설정
  * usage: 실제로 사용된 기능에 대해서만 필요한 폴리필 자동 추가 (일반적으로 권장)
  * entry: 엔트리 파일에서 한 번에 폴리필 추가 (불필요한 폴리필까지 포함될 수 있어 번들 크기 증가)

**5.2.2.2 런타임에 core-js 폴리필을 로드하기**

**core-js를 직접 임포트하는 방식의 문제점**
* 큰 번들 크기: 바벨의 헬퍼 함수가 필요한 모든 파일에 추가되어 중복 코드 발생
* 전역 네임스페이스 오염: 내장 기능들이 전역 스코프에 추가되어 다른 코드나 라이브러리와 충돌 위험

**해결 방법: @babel/plugin-transform-runtime 사용**
* 전역 오염이 없는 core-js를 사용하며 헬퍼 함수를 모듈로 사용
* 중복 삽입하는 헬퍼 함수나 폴리필을 공유 모듈(@babel/runtime) 형태로 변환

**설정 예시**
```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          browsers: ['ie >= 11'],
        },
      },
    ],
  ],
  plugins: [
    [
      "@babel/plugin-transform-runtime",
      {
        corejs: {version: 3, proposals: false},   
        helpers: true,    
        regenerator: true,  
      },
    ],
  ],
};
```

**@babel/preset-env vs @babel/plugin-transform-runtime 비교**

| 항목 | @babel/preset-env + corejs | @babel/plugin-transform-runtime |
|------|---------------------------|----------------------------------|
| 폴리필 주입 방식 | 전역에 patch | 모듈화(비전역, "pure") |
| 전역 오염 | 있음 | 없음 |
| 번들 크기 | usage면 필요분만 | 헬퍼 중복 제거로 작아질 가능성 큼 |
| 권장 사용처 | 애플리케이션(웹앱) | 라이브러리/SDK |

cf. 라이브러리 개발 시에는 전역 오염을 방지하기 위해 @babel/plugin-transform-runtime을 사용하는 것이 더 바람직하고, 일반 애플리케이션에서는 @babel/preset-env를 사용하는 것이 간편하다.
