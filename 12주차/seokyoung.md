## 6.3 패키지 번들의 선두주자, 롤업

### 롤업의 등장 배경과 소개

- 모듈이 많아질수록 웹팩의 번들 방식과 CommonJS의 단점이 부각되면서 이를 극복하고자 롤업이 등장
- 롤업은 ES2015의 ESModule을 기반으로 가능한 한 자바스크립트 번들을 평탄하게 만드는 것이 목표
- 모든 코드를 하나씩 모듈화하는 대신, 같은 장소에 배치해서 하나의 코드 덩어리로 만드는 방식

- 웹팩 → 코드 분할, HMR 등이 중요한 웹 애플리케이션에 적합
- 롤업 → 하나의 목적을 가진 여러 코드를 다른 개발자나 웹 서비스에 제공하기 위한 라이브러리(패키지)에 적합

### 롤업의 특징

- ESModule 기반 모듈 번들링
  - 롤업은 자바스크립트 모듈 번들러로서 ESModule을 효율적으로 번들링하는 데 초점
  - ESModule 기반 → 정적 모듈 분석 수행 → 이를 토대로 트리 셰이킹 가능
  - 모든 모듈을 평탄화해서 하나의 큰 코드로 만들어 기존 모듈 시스템이 가지고 있는 모듈 오버헤드 최소화
- 여러 형태의 번들 형태 지원
  - 롤업은 자바스크립트 라이브러리를 만드는 것을 지원하는 데 초점
  - 번들 결과물을 다양한 형태로 만들 수 있음
- 트리 셰이킹
  - 롤업은 처음부터 트리 셰이킹 기본적으로 지원
  - ESModule의 정적 기반 분석을 바탕으로 어떤 코드가 실제 사용되지 않는지를 빠르게 판단
  - 빌드하는 시점에 의존성 그래프 생성 → 사용되지 않는 모듈 탐지 → 트리 셰이킹 적용
  - 롤업을 거친 번들 파일은 실제 소스코드 크기보다 훨씬 더 작은 결과물 생성 가능

### 롤업의 주요 필드

> `!` rollup.config.js에 작성하는 필드

### input

- 번들의 시작점
- 단일 파일 지정 / 배열로 여러 개의 파일 지정 / 객체를 넣어 이름과 파일명을 맵 형태로 지정
- 인풋의 결과물은 아웃풋에 반영됨
  - 문자열이나 배열인 경우 해당 파일명을 기반으로 생성
  - 맵 형태로 넣은 경우 별도 이름 지정 가능

**문자열로 단일 파일 지정(진입점이 단 하나만 있는 경우)**

```jsx
export default {
  input: 'src/main.js',
}
```

**문자열 배열로 여러 개의 파일 지정(진입점 여러 개 있는 경우)**

```jsx
export default {
  input: ['src/main.js', 'src/a.js'],
}
```

**문자열로 단일 파일 지정(진입점이 단 하나만 있는 경우)**

```jsx
export default {
  input: {
    main: 'src/main.js',
    admin: 'src/admin.js',
    user_profile: 'src/user/profile.js',
  },
}
```

**파일 구조와 export를 유지하면서 번들링하기**

```jsx
import { globSync } from 'glob'
import path from 'node:path'
import { fileURLToPath } from 'node:url'

export default {
  input: Object.fromEntries(
    globSync('src/**/*.js').map(file => [
      path.relative('src', file.slice(0, file.length - path.extname(file).length)),
      fileURLToPath(new URL(file, import.meta.url)),
    ]),
  ),
}
```

이 방식을 사용하면 /src 하위에 있는 모든 파일을 읽어온 다음, /src/와 .js를 제거한다.

예를 들어, /src/main.js가 있다면 main이 되는 방식이다. 그리고 그 값으로는 `fileURLToPath(new URL(file, import.meta.url))` 형태로 읽어오는데, 이는 절대 경로를 반환한다. 이렇게 해두면 다음과 같은 형태로 결과물을 확인할 수 있다.

```jsx
{
	"main": "/Users/username/works/my-project/src/main.js",
	"utils/helper": "/Users/username/works/my-project/src/utils/helper.js",
	"components/Button": "/Users/username/works/my-project/src/component/Button.js",
	"components/Input": "/Users/username/works/my-project/src/component/Input.js",
}
```

이처럼 src 디렉터리의 내부 구조를 유지한 채로 input의 진입점에 내부의 모든 파일을 객체로 정의해서 빌드하면 개별 파일이 의도치 않게 트리 셰이킹 되는 것을 방지하고, 원본 프로젝트의 구조를 그대로 보존할 수 있으며, 각 파일을 독립적으로 번들링할 수 있다는 장점이 있다.

<aside>

**`?` fileURLToPath(import.meta.url)은 어떤 역할인가요?**

- fileToPath → 인수로 받은 URL의 파일 시스템 경로를 반환하는 함수
- url → 현재 모듈의 URL을 가져오는 데 사용
- 즉, 두 코드를 조합하면 현재 환경에서 현재 모듈의 파일 시스템 경로를 반환

이는 주로 현재 모듈의 디렉터리 경로를 얻거나 상대 경로를 활용할 때 기준점으로 사용할 수 있다.

</aside>

### external

- 번들링 시 외부 의존성을 명시하는 데 사용되는 필드
- 배열이나 함수를 값으로 받으며 **반환 받은 값은 번들링에 포함시키지 않음**

```jsx
import { shuffle } from 'loadsh-es'

const array = [1, 2, 3, 4, 5]
return shuffle(array)

export default {
  input: 'index.js',
  output: {
    file: 'dist/bundle.js',
    format: 'esm',
  },
  external: [''],
}
```

externale 필드에 아무것도 작성하지 않은 상태로 빌드를 시도하면 다음과 같은 경고 메시지가 출력된다.

```
rollup -c index.js → dist/index.js... (!) Unresolved dependencies https://rollupjs.org/trouble-shooting/#warning-treating-module-as-external-dependency lodash/shuffle (imported by "index.js") created dist/index.js in 66ms
```

이러한 경고가 나오는 이유는 롤업은 오로지 상대 경로로 선언된 모듈 ID만 해석하기 때문이다.

`'lodash-es'` 같은 Bare module specifier(맨 경로)는 Node.js의 module resolution 방식이 필요한데, 롤업은 Node의 module resolution을 기본 제공하지 않는다.

그래서 롤업은 **"저건 외부 의존성(external)이다"라고 판단하고 번들링을 포기**하게 된다.

상대 경로로 해석할 수 없는 node_modules 내부의 패키지 또는 Node.js 모듈은 다음과 같이 처리한다.

1. 해당 모듈을 어떤 방식으로 찾아가는지 알려주는 플러그인을 설치해서 번들리에 포함시키는 방법
2. 외부에서 주입되는 모듈로 선언하는 법

1번 방식으로 처리하게 된다면, 다음과 같은 문제가 발생할 수 있다.

- loadsh-es의 shuffle 구현에 필요한 모든 내용이 번들에 포함되어 번들 크기가 엄청나게 커짐
- 만약 패키지를 사용하는 쪽에서 이미 lodash-es를 사용하고 있다면?
  → 사용자의 node_modules에는 lodash-es의 shuffle에 필요한 코드가 두 벌이 포함되게 됨
- 만약 shuffle에 버그가 있거나 보안 취약점이 있어 버전업을 해야한다면?
  → 그때마다 번들을 새로 만들어서 배포해야 하는 불편함 발생

이럴 때 사용할 수 있는 필드가 바로 external이다.

- external 내부에 선언되어 있는 모듈 → 외부에서 주입한다고 간주해 **번들에 포함되지 않음**
- 대신 결과물 방식(CommonJS, ESModule, IIFE 등)에 맞게 외부에서 주입되는 형태의 코드로 작성

앞의 코드에 lodash-es를 external 필드에 추가하고 빌드해보자.

```jsx
export default {
  input: 'index.js',
  output: {
    file: 'dist/index.js',
    format: 'esm',
  },
  external: ['lodash-es'],
}
```

빌드 결과물은 다음과 같이 ESModeul에 맞는 방식인 import 구문으로 처리된 것을 볼 수 있다.

이렇게 하면 해당 패키지는 사용하는 측의 node_modules에 lodash-es가 있다는 가정 하에 처리된다.

```jsx
// dist/index.js
import { shuffle } from 'lodash-es'

const array = [1, 2, 3, 4, 5]
return shuffle(array)
```

**`?` 그렇다면 사용하는 측에 lodash-es를 어떻게든 설치하게 만들려면 어떻게 해야 할까?**

package.json의 dependencies로 작성해 해당 패키지를 설치할 때 무조건 함께 설치하게 할 수 있다.

**`?` 그렇다면 dependencies의 내용을 external 필드에 그대로 가져다 써도 되는 것일까?**

자바스크립트 패키지를 만든다면 대체로 그렇다고 볼 수 있다.

토스의 slash에서도 external 내부 필드로 패키지 관리자가 설치해줄 것으로 기대하는 package.json의 dependencies와 peerDependencies를 공통적으로 선언해 둔것을 볼 수 있다.

```jsx
const external = pkg => {
  const dependencies = Object.keys(packageJSON.dependencies || {})
  const peerDependencies = Object.keys(packageJSON.peerDependencies || {})
  const externals = [...dependencies, ...peerDependencies, ...builtins]

  return externals.some(externalPkg => {
    return pkg.startsWith(externalPkg)
  })
}
```

이처럼 패키지 관리자가 설치해서 node_modules에 있을 것으로 기대하는 모듈이나 jQuery와 같이 window 등 전역 변수에서 이미 선언되어 있는 필드를 바라볼 목적으로 번들에 포함시키지 않고 싶다면 external 필드를 사용하는 것이 좋다.

### output

웹팩과 마찬가지로 번들된 결과물을 어떻게 만들지 나타내는 필드

**output 필드 주요 옵션**

- file: input 필드로 받은 파일을 어떤 식으로 내보낼지 결정하는 필드
- dir: input 필드가 배열 등으로 다중 진입점으로 만들어졌다면 dir을 사용해 폴더 지정
- name: `umd, iife만 사용 가능` 브라우저, 워커, Node.js 같은 환경에서 번들된 코드의 결과 값을 넣기 위한 전역 변수를 지정하는 값
- globals: `umd, iife만 사용 가능` 번들의 특정 내용이 외부에 있다는 가정하에 번들링에 사용하는 값
  - external 필드와 비슷한 역할
  - 전역 네임스페이스 오염 및 수동으로 의존성을 관리해야 한다는 점에서 거의 사용되지 않는 방식・
- format: 롤업의 결과물을 어떤 모듈 시스템에 맞춰 만들지를 결정하는 필드
  - amd
  - cjs, commonjs
  - es, esm, module
  - iife
  - umd
  - system, systemjs
- plugins: 번들링 프로세스를 만드는 과정에서만 실행되는 플러그인을 지정하는 필드
  - 결과물에 대해 추가적인 처리・변형이 필요할 때 혹은 출력에 앞서 특정한 작업을 수행하기 위한 목적
  - output.plugins에서 사용되는 플러그인은 결과물을 만들어내는 프로세스 중간에 실행할 수 있음
  - 언제 필요? 특정한 output에만 실행하고 싶은 훅이 필요할 경우에 사용
  ```jsx
  // rollup.config.js
  import terser from '@rollup/plugin-terser'

  export default {
    input: 'main.js',
    output: [
      {
        file: 'bundle.js',
        format: 'es',
      },
      {
        file: 'bundle.min.js',
        format: 'es',
        plugins: [terser()], // plugin-terser를 추가해 압축되게끔 구성
      },
    ],
  }
  ```

<aside>

**롤업의 훅(hook)**

- 번들링 프로세스의 다양한 단계에서 사용자가 원하는 로직을 실행할 수 있도록 도와주는 기능
- 이를 토대로 롤업을 사용하는 개발자는 롤업 프로세스 중간에 원하는 기능을 수행할 수 있음
- output.plugins에서 사용하는 훅 - renderStart: 출력 생성이 시작될 때 호출 - banner: 번들 파일 시작 부분에 코드를 추가 - 주로 라이선스 정보나 빌드 정보 추가 시 사용 - footer: 번들 파일 끝 부분에 코드를 추가 - 추가적인 정보 혹은 마무리 코드 적용 시 사용 - intro: 번들 내용 직전에 코드를 추가 → banner와 실제 코드 사이에 위치 - 초기화 코드, 전역 변수 설정 시 사용 - outro: 번들 내용 직후에 코드를 추가 → 실제 코드와 footer 사이에 위치 - 정리용 코드나 최종 실행 코드를 넣는 용도로 사용 - renderChunk: 각 청크가 생성된 후 호출. 청크의 내용 수정 가능 - generateBundle: 모든 청크와 에셋이 생성된 후, 실제로 파일에 쓰기 전에 호출 - writeBundle: 모든 파일이 디스크에 쓰여진 후 호출 - closeBundle: 번들링 프로세스가 완전히 완료된 후 호출
</aside>

이처럼 **특정 결과물에 대해서만 추가 처리를 적용**하고 싶은 경우 output.plugins를 활용할 수 있다.

### plugins

- @rollup/plugin-terser: terser를 사용하여 압축된 번들 파일 생성 플러그인
- @rollup/plugin-node-resolve: node_modules에 있는 서드파티 모듈 해석 플러그인
- @rollup/plugin-commonjs: CommonJS → ES6 모듈 변환 플러그인
- @rollup/plugin-json: JSON 파일을 읽어들이고 모듈에 포함시킬 수 있게 도와주는 플러그인
- rollup-plugin-postcss: CSS 및 관련 전처리기 파일을 처리가히 위한 플러그인
- @rollup/plugin-replace: 번들링 시점에 코드의 특정 부분을 원하는 내용으로 대체하는 플러그인
  - 문자열 대체, 환경변수 주입, 조건부 코드의 실행 여부
- @rollup/plugin-babel: 바벨을 이용하여 번들링된 패키지를 트랜스파일하는 플러그인
- rollup-plugin-visualizer: 번들된 결과물을 분석을 위한 시각화 플러그인

## 6.4 번들 도구의 신흥 강자, 비트

### 비트의 등장 배경과 소개

**빌드 도구의 패러다임 전환**

> 상대적으로 느린 자바스크립트로 꼭 빌드할 필요가 있을까?
> 그냥 입력 값과 결과 값만 자바스크립트이면 되지 않을까?

→ SWC, esbuild, Biome, Parcel, turbopack 등 다른 언어 기반 도구 등장

**Vite가 빠를 수 있었던 이유**

- 개발 환경에서는 번들링하지 않고 ESModule 방식으로 코드 제공
- ts→js 변환 시 tsc 컴파일러가 아닌 esbuild로 트랜스파일

### Vite의 기본 개념과 특징

**롤업과 esbuild의 공존**

- 롤업+esbuild로 빠르고 효율적으로 빌드
- 이외에도 SWC를 적용한 @vitejs/plugin-react-swc, rollup@4 사용
- Vite의 장기적인 목표 → 더 빠르고 효율적인 빌드 도구
  - **러스트 기반**으로 전환 + 롤업을 대체하는 **롤다운** 도입

**의존성 사전 번들링**

Vite는 처음 개발 모드를 시작하면 프로젝트에 선언된 의존성을 미리 사전에 번들링한다.

이를 의존성 사전 번들링이라고 한다.

Vite가 의존성 사전 번들링 작업을 하는 데는 다음과 같은 이유가 있다.

- CommonJS와 UMD 간의 상호호환성
- 개발 모드에서 ESModule을 불러오는 것에 대한 성능 최적화

node_modules/.vite/deps 폴더에서 사전 번들링을 확인할 수 있으며,

해당 폴더의 메타데이터에서 needsInterop 필드를 통해 모듈 간 상호운용성 처리를 확인하고 작업한다.

**모던 웹 환경을 위한 도전**

> Vite provides opinionated features that push writing modern code. For example:
>
> - The source code can only be written in ESM, where non-ESM dependencies need to be [**pre-bundled as ESM**](https://vite.dev/guide/dep-pre-bundling) in order to work.

- Vite의 내부 의존성은 모두 ESModule로 작성된 패키지로 구성해야 함
- 부득이하게 CommonJs로 작성된 의존성을 사용할 경우 ESModule로 사용할 수 있도록 준비해야 함
- Vite@6 부터는 Vite API를 사용하는 외부 코드도 ESModule로 작성해야 함

따라서 앞으로 Vite를 기반으로 작성할 예정이라면 ESModule로 구성 파일을 작성해야 하며, 현재 운영 중인 Vite 기반 프로젝트 역시 향후 안정적인 지원을 위해서는 ESModule로 구성 파일의 마이그레이션을 검토해야 한다.

### 설정에 필요한 주요 필드

- root: 프로젝트의 최상위 디렉터리 위치 (일반적으로 index.html의 위치)
- base: public base path를 의미 → 배포하고자 하는 시스템이 최상위가 아닌 경우 유용
- mode: 개발 서버는 development / build로 실행될 경우 production으로 대체
  - stage, test 등 사용자가 원하는 다른 값과 연계해서 사용
  - Vite 내부적으로 사용하는 dotenv에도 영향
    - mode 값 없이 Vite에서 dev로 실행하는 경우: `.env.local`
    - mode의 값에 특정한 값을 넣어 실행되는 경우: `.env.[mode]` → `.env.stage`
    - build로 실행하는 경우: `.env.production`
- define: 전역 상수로 대체되는 값을 정의 → JSON.stringify로 직렬화 처리가 가능한 값만 선언
- plugins: 플러그인 목록. 롤업의 플러그인을 대부분 호환해서 사용 가능
  - 롤업 호환 플러그인: https://github.com/rollup/awesome
  - Vite 전용 플러그인: https://github.com/vitejs/awesome-vite#plugins
- publicDir/cacheDir
  - publicDir: 정적 에셋 디렉터리 (/public)
  - cacheDir: 캐시 파일 저장 디렉터리 (node_modules/vite)
- build: 빌드 관련 옵션
  - target: 최종 빌드 결과물을 어떤 브라우저에 맞춰 빌드할지 알려주는 옵션
  - outDir: 프로젝트 번들 결과물이 생성되는 폴더 (/dist) → .gitignore에 추가하는 걸 권장
  - minify: 코드를 압축시켜 경량화할지 여부를 나타내는 값
  - lib: 라이브러리 모드로 빌드할지 여부를 나타내는 값
    - entry, name, format, filename 설정 필요
  - rollupOptions: 롤업 옵션을 커스터마이징할 수 있는 필드
