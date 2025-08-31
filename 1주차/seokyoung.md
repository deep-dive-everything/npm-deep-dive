## 1.1 npm의 정의와 역사

- npm은 현재 전 세계 자바스크립트 개발자가 공통으로 사용하는 패키지 관리 시스템이자 CLI 도구
- npm을 통해 전 세계에서 업로드된 자바스크립트 패키지를 손쉽게 설치하고 업데이트하며, 프로젝트 간의 의존성을 효율적으로 관리할 수 있음

### 1.1.1 npm의 역사와 배경

#### 🤔 npm은 Node Package Manager이다?

- 아니다! [npm 공식문서](https://github.com/npm/cli?tab=readme-ov-file#faq-on-branding)에서도 이를 바로잡고 있다.
- npm은 npm으로 써주시길…🙏 (우리애 Npm, NPM 아닙니다)
- npm은 사실 bash utility인 pm을 이어받아 진화시킨 프로젝트
  - pm은 pkgmakeinst의 약자
  - 그러므로 npm은 Node pm, new pm 정도로 소개하는 게 맞다.

#### 🤔 npm 이전에 자바스크립트 패키지를 제공하려면 어땠을까?

- 초기에는 대부분 Node.js 생태계에서 동작하는 코드만 npm에 업로드

```tsx
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
```

- 프런트엔드나 웹에서 사용하는 리소스는 script 형태로 사용할 수 있도록 CDN에 업로드하여 제공
- 페이스북(현 메타)이 리액트가 npm에 올라가는 것을 필두로 프런트엔드에서 동작하는 자바스크립트 코드 역시 npm에 배포하는 것을 선호하기 시작
- 왜?
  - CDN에 업로드하거나 다운로드할 때 발생하는 비용, 보안 문제, 버전 관리 어려움 등의 이슈
  - 에서 npm은 자유롭고 패키지의 버전을 관리할 수 있다는 점에서 유지 보수에서 이점

### 1.1.2 npm의 주요 기능

- 패키지 설치 및 관리
  - npm이 가진 가장 기본적인 기능
  - package.json의 dependencies와 devDependencies에 선언된 패키지를 node_modules에 설치해서 Node.js 기반 프로젝트에서 사용할 수 있게 도와줌
  - 설치 이외에 오래된 패키지 업데이트, 보안에 취약한 패키지 확인, 의존성 임의 수정도 가능
- 패키지 배포 및 공유
  - npm 레지스트리 → 오픈소스 생태계
  - 유료로는 private 패키지도 업로드 가능
- 스크립트 실행
  - package.json의 scripts 필드를 통해 원하는 스크립트를 실행 가능
  - 프로젝트에서 자주 사용되는 복잡한 명령어를 간단한 CLI로 실행 가능
  - PATH도 등록 가능하여 eslint나 node_modules에 있는 .bin 스크립트도 실행 가능
- npmjs.com
  - npm에 업로드 된 패키지 정보 확인 가능
  - 자바스크립트 패키지의 개요, 실제 npm에 업로드 된 코드, 의존성, 패키지에 의존하는 다른 패키지, 현재 업로드 된 패키지 버전 리스트, 주간 다운로드 수 등

### 1.1.3 유용한 사이트

- 번들포비아: https://bundlephobia.com/
  - npm에 업로드 되어 있는 패키지의 크기와 관련된 정보를 얻을 수 있는 사이트
  - 배포 전에 패키지 크키 체크 목적으로 유용

![image.png](attachment:952189f9-47e3-4e02-a764-4f9f7faa9f5b:image.png)

![image.png](attachment:b833b660-847a-40df-971e-17ba92b69d88:image.png)

- npm 트렌드: https://npmtrends.com/
  - 비슷한 목적의 패키지의 현재 상태, 다운로드 횟수 등의 비교할 수 있는 사이트
  - 기술 스택 선정 시 하나의 지표로 삼는 목적으로 유용
- unpkg: https://unpkg.com/
  - 실제 npm에 업로드 되어 있는 파일을 직접 확인할 수 있는 사이트
  - node_modules를 직접 보는 것처럼 한눈에 확인 가능
  - node_modules에 설치하기 전에 실제 패키지가 어떻게 구성되어 있는지 확인할 때 유용
  ```
  https://unpkg.com/:package@:version/:file
  ```
  - [unpkg.com/react/](https://unpkg.com/react/)
    ![image.png](attachment:f3da16ef-1c05-4436-8f25-0a7b7b306baf:image.png)
  - https://unpkg.com/react@18.3.1/umd/react.production.min.js
    ![image.png](attachment:479df298-16bf-44a2-967c-a9df45a4e692:image.png)
    > UMD 빌드가 React 19에서 사라졌다.
    > 사유 → HTML script 태그 내에서 모듈을 불러올 수 있기 때문에 CDN을 통해 모듈을 불러오는 형태로
    > 사용할 수 있게 되었다.
    > https://ko.react.dev/blog/2024/04/25/react-19-upgrade-guide#umd-builds-removed
  - 모듈 시스템 관련 글
    - https://klmhyeonwooo.tistory.com/107
    - https://blog.rhostem.com/posts/2019-06-23-universal-module-definition-pattern
- snyk: https://security.snyk.io/vuln/npm
  - 전 세계의 다양한 오픈소스의 취약 소개 (Medium, High, Critical 순)
  - 패키지 설치 전 해당 패키지에 보안 취약점이 있는지 확인할 때 유용

## 1.2 유의적 버전이란?

> 소프트웨어의 버전을 어떻게 정의하고, 어떻게 올리고 관리하는지에 대한 규약

### 유의적 버전의 의의

- 개발자→사용자: 해당 소프트웨어의 예상되는 변경점을 사용자에게 손쉽게 알려줄 수 있음
- 사용자→개발자: 개발자가 유의적 버전 체계를 지켰다는 가정하에 해당 소프트웨어에 유연하게 의존할 수 있는 시스템을 만들 수 있음

### 1.2.2. 유의적 버전 명세

- [유의적 버전 2.0.0-ko2의 유의적 버전 명세](https://semver.org/lang/ko/#%EC%9C%A0%EC%9D%98%EC%A0%81-%EB%B2%84%EC%A0%84-%EB%AA%85%EC%84%B8-semver)

> [!NOTE]
>
> #### 요약
>
> 버전을 **주.부.수(Major.Minor.Patch)** 숫자로 하고:
>
> 1. 기존 버전과 호환되지 않게 API가 바뀌면 **“주(主) 버전”**을 올리고,
> 2. 기존 버전과 호환되면서 새로운 기능을 추가할 때는 **“부(部) 버전”**을 올리고,
> 3. 기존 버전과 호환되면서 버그를 수정한 것이라면 **“수(修) 버전”**을 올린다.
>
> 주.부.수 형식에 정식배포 전 버전이나 빌드 메타데이터를 위한 라벨을 덧붙이는 방법도 있다.

### 1.2.3 유의적 버전의 문법

> npm에서 제공하는 semver 패키지를 이용하여 유의적 버전의 유효성 확인, 버전 간의 비교, 범위 비교 가능

```jsx
const semver = require('semver')

semver.valid('1.2.3') // '1.2.3'
semver.valid('a.b.c') // null
semver.clean('   =v.1.2.3   ') // '1.2.3'
semver.satisfies('1.2.3', '1.x || >=2.5.0 || 5.0.0 - 7.2.3') // true
semver.gt('1.2.3', '9.8.7') // false
semver.lt('1.2.3', '9.8.7') // true
semver.minVersion('>=1.0.0') // '1.0.0'
semver.valid(semver.coerce('v2')) // '2.0.0'
semver.valid(semver.coerce('42.6.7.9.3-alpha')) // '42.6.7'
```
