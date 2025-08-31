# 1장

## 1.1 npm의 정의와 역사

### 1.1.1 npm의 역사와 배경

- 아이작이라는 개발자에 의해 만들어짐 (bash로 작성된 패키지 관리자)
- 초기에는 cdn에 웹 리소스를 업로드하고 `<script src="https://ajax.googleapis.com/ajax/jquery.min.js">` 과 같이 사용하는 것이 일반적 -> 리액트가 npm에 올라가면서 점점 npm에 자바스크립트 코드가 배포되기 시작

### 1.1.2 npm의 주요 기능

- 패키지 설치 및 관리
- 패키지 배포 및 공유
- 스크립트 실행: package.json의 scripts 필드를 사용해 복잡한 명령어를 간단한 CLI로 실행

### 1.1.3 npm과 관련된 유용한 사이트

- [번들포비아](https://bundlephobia.com/): npm에 업로드 되어있는 패키지의 크기와 관련된 정보를 얻을 수 있는 사이트
- [npm 트렌드](https://npmtrends.com/yup-vs-zod): 여러 패키지 중에서 무엇을 고를지 어려울 때 사용하는 사이트
- [unpkg](https://app.unpkg.com/react@19.1.1): 실제 npm에 업로드되어 있는 파일을 직접 확인할 수 있는 서비스
- [synk](https://security.snyk.io/vuln/npm): 오픈소스 취약점을 알려주는 페이지

## 1.2 유의적 버전이란?

### 1.2.1 유의적 버전의 등장 배경과 정의

- 소프트웨어의 버전을 어떻게 정의하고, 어떻게 올리고 관리하는지에 대한 규약
  - 계속해서 변화하는 의존성으로 인한 혼란을 제어하기 위해 만들어짐
- 유의적 버전의 주.부.수
  - 기존 버전과 호환되지 않는 API 변경 사항이 있다면 **주** 버전이 올라감
  - 기존 버전과 호환되면서 새로운 기능이 추가되면 **부** 버전이 올라감
  - 기존 버전과 호환되면서 버그를 수정한 것이라면 **수** 버전이 올라감

### 1.2.3 유의적 버전의 구체적인 명세

1. 공개된(사용자가 있는) 소프트웨어여야 함
2. 버전 번호는 각각이 자연수인 X.Y.Z의 형태로 해야 함
3. 특정 버전으로 패키지를 배포하고 나면 해당 버전의 내용은 절대 변경하지 말아야 함(변경분이 있다면 새 버전으로)
4. 주 버전 (0.y.z)는 초기 개발을 위해 쓴다.
5. 1.0.0 버전은 공개 API를 정의한다. 이후의 버전 번호는 이때 배포한 공개 API에서 어떻게 바뀌는지에 따라 올린다.
6. 수 버전 Z(x.y.Z | x>0)는 반드시 그 전 버전 API와 호환되는 버그 수정인 경우에만 올린다.
7. 공개 API에 기존과 호환되는 새로운 기능을 추가할 때는 반드시 부 버전 Y(x.y.z | x>0)을 올린다. 공개 API의 일부를 deprecated로 표시한 경우에도 반드시 올리도록 한다.
8. 기존과 호환되지 않는 변화가 있을 때는 반드시 주버전을 올린다.

```js
// 1.0.0
function sum(a: number, b: number) {
  return a + b;
}

// 1.1.0로 업그레이드 가능
function sum(a: number, b: number, c?: number) {
  return a + b + (c || 0);
}

// 2.0.0으로 업그레이드 해야함
// sum(1,2)의 호환성을 깨트리므로
function sum(a: number, b: number, c: number) {
  return a + b + c;
}
```

9. 수 버전 바로 뒤에 붙임표(-)를 붙이고 마침표(.)로 구분된 식별자를 더해서 pre-release 버전을 표기할 수 있음
10. 빌드 메타데이터는 수 버전이나 정식 배포 전 식별자 뒤에 더하기 기호를 붙인 뒤에 마침표로 구분된 식별자를 덧붙여서 표현할 수 있다. pre-release 버전도 많아지면 어떤 것에 대한건지 구분해줄 필요가 있음. 커밋 문자열을 많이 사용하는 추세 (예: 1.0.0-beta+exp.sha.51141f85)
11. 우선순위는 버전의 순서를 정렬할 때 서로를 어떻게 비교할지를 나타낸다. 주, 부, 수 버전 순서대로 우선순위를 가지며 같은 버전의 크기대로 먼저 비교한다. 빌드 메타데이터 정보는 우선순위 계산에 영향을 주지 않는다. pre-release의 경우 숫자는 숫자 크기, 알파벳이나 다른 문자가 있는 경우에는 아스키 문자열 정렬로 비교
    (예: 1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-rc < 1.0.0)

### 1.2.3 유의적 버전의 문법

- 정규 표현식을 사용해 문자열이 유의적 버전 문법에 맞는지 검증할 수 있음

```
^(0|[1-9][0-9]*)\.(0|[1-9][0-9]*)\.(0|[1-9][0-9]*)(-(0|[1-9A-Za-z-][0-9A-Za-z-]*)(\.[0-9A-Za-z-]+)*)?(\+[0-9A-Za-z-]+(\.[0-9A-Za-z-]+)*)?$
```

- 하지만 복잡한 정규표현식을 사용하는것보다 [semver](https://github.com/npm/node-semver)를 사용하는 것이 일반적

### Semver 라이브러리 사용 상황들

1. npm 패키지 개발할 때 - 의존성 호환성 확인

```js
const semver = require("semver");

// 사용자 Node.js 버전이 내 패키지와 호환되는지 확인
if (!semver.satisfies(process.version, "^14.0.0")) {
  console.error("Node.js 14 이상이 필요합니다");
  process.exit(1);
}
```

2. CLI 도구에서 업데이트 알림

```js
const currentVersion = "1.2.3";
const latestVersion = await getLatestVersion();

if (semver.gt(latestVersion, currentVersion)) {
  console.log(`새 버전 ${latestVersion}이 있습니다!`);
}
```

3. 자동 배포 스크립트에서 버전 증가

```js
const currentVersion = require("./package.json").version;
const nextVersion = semver.inc(currentVersion, "patch"); // 1.2.3 → 1.2.4
```

4. API 버전 호환성 체크

```js
// 클라이언트가 지원하는 API 버전 확인
const clientApiVersion = request.headers["api-version"];
if (!semver.satisfies(clientApiVersion, "^2.0.0")) {
  return res.status(400).json({ error: "Unsupported API version" });
}
```

## 1.3 유의적 버전과 npm 생태계의 명과 암

### 1.3.1 left-pad: 수천만 패키지에서 의존하는 유틸 패키지가 사라지면 어떻게 될까?

- left-pad: 문자열에 특정 숫자만큼 왼쪽에 공백이나 문자를 채워넣는 라이브러리(현재는 deprecated + js 네이티브 기능이 생겨서 자주 쓰이지는 않음)
- left-pad -> line-numbers -> babel 이런 의존관계가 존재했었음
- 개발자가 left-pad를 삭제하면서 line-numbers -> babel 차례로 사용을 못하게 되어버리고 babel을 사용하던 수많은 프로젝트에 에러 발생
- 이후 npm 팀은 이런 상황 방지를 위해 새로운 정책 추가
  - 다른 패키지가 의존하는 경우 패키지 삭제를 어렵게 만듦
  - 유명 패키지가 있을 경우 이를 부정사용하는 것을 방지하기 위해 'security-holder' 패키지 추가

### 1.3.2 everything: 모든 자바스크립트 패키지를 의존성으로 가진다면?

- 개발자들이 호기심으로 모든 자바스크립트 패키지를 의존성으로 갖는 `everything` 이라는 패키지를 개발
- npm의 모든 패키지들이 삭제가 안되는 사태 발생

### 1.3.3 is-promise: 잘못된 부 버전 업데이트가 만들어낸 사태

- is-promise가 잘못된 문법으로 작성된 버전을 부버전으로 업데이트 함
- 당시 is-promise -> inquirer -> create-react-app 이런 의존관계를 가지고 있었고 에러 발생
- is-promise가 버그를 수정해 출시하여 해결 되었음. 유의적 버전을 준수하려고 했으나 개발자의 실수로 문제가 발생한 케이스

### 1.3.4 color.js와 faker.js: 섣부른 부, 수 버전 업데이트는 독이 될 수도 있다.

- [faker.js와 colors.js 사태를 통해 살펴보는 오픈소스의 양면성](https://tech.pxd.co.kr/post/faker-js%EC%99%80-colors-js-%EC%82%AC%ED%83%9C%EB%A5%BC-%ED%86%B5%ED%95%B4-%EC%82%B4%ED%8E%B4%EB%B3%B4%EB%8A%94-%EC%98%A4%ED%94%88%EC%86%8C%EC%8A%A4%EC%9D%98-%EC%96%91%EB%A9%B4%EC%84%B1-226)

### 1.3.5: event-stream 사건: 오픈소스는 얼마나 안전한가?

- [event-stream 공격 사건의 전말](https://blog.ull.im/engineering/2018/11/30/event-stream-issue.html)

### 1.3.6 유의적 버전과 npm을 사용할 때 주의할 점

#### 1.3.6.1 유의적 버전은 규약일 뿐이다

- 고의건 실수건 메인테이너가 유의적 버전 체계를 위반하는 사건이 발생해왔음
- 새로운 버전의 패키지를 설치할 때는 많은 주의를 기울여야 함
  - CHANGELOG.md나 릴리스 노트를 살펴보면서 문제가 없는지 확인
  - ^나 ~를 사용하지 않고 고정된 버전을 사용하는 것도 방법

#### 1.3.6.2 무조건 설치하는 것이 능사는 아니다

- 설치에 앞서 아래 사항을 고려해볼 것

1. 정말로 대체 불가능한 패키지인가: 자체 구현하는 게 나을 수도 있다
2. 사용할 수 있는 만큼 충분히 검증됐는가(패키지 인지도 확인)
3. 타당한 dependencies를 가지고 있는가(package.json 확인)

#### 1.3.6.3 락 파일 변경에 주의를 기울이기

- lock 파일은 파일의 크기가 크고 diff를 github 인터페이스에서 확인하기 어려워 쉽게 머지됨
- npm install 대신 npm ci를 통해 lock 파일이 변경되지 않도록 하는 것이 좋음
- 좀 더 보수적으로 접근하고 싶다면 yarn과 Pnpm에서 지원하는 pnp 모드와 zero install을 사용해보는 것도 좋음

#### 1.6.3.4 보안 취약점에 귀 기울이기

- 오픈소스 보안 취약점을 알려주는 다양한 서비스 활용해보기(snyk, dependabot)
