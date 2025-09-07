## 2.1 package.json 톺아보기

### package.json

- Node.js 프로젝트의 정보를 기술하는 핵심 파일
- 프로젝트의 메타데이터 정의, 패키지 실행과 개발에 필요한 의존성 나열, 실행 가능한 스크립트를 설정하는 역할
- 자바스크립트 객체 형태가 아닌 순수한 JSON 형식으로 작성해야 함

### package.json의 주요 필드

#### 🔷 name

- Node.js 프로젝트의 이름을 선언하는 필드
- npm 레지스트리에 업로드하거나 내부적으로 해당 프로젝트를 다른 곳에서 참조할 목적이 없다면 없어도 무방
  - 웹 서비스 목적으로 만들어진 `package.json` → 빌드한 파일만 특정 서버에 업로드해서 사용하기 때문
- name 필드 규칙
  - 이름은 214자 이하이며, 글자 제한에는 스코프까지 포함
  - 스코프 뒤에 오는 이름은 `.`이나 `_`로 시작할 수 있음
  - 오직 소문자만 사용 가능
  - URL에서 표시할 수 없는 문자열(공백, #, %, ? 등 특수문자는 포함될 수 없음)은 사용 불가능
  - Node.js의 코어 모듈과 이름이 동일해서는 안 됨

#### scope

- 조직 또는 그룹을 의미하며, 여러 패키지 사이에 연관 있는 패키지를 묶고 싶을 때 주로 사용
- `@scope/packagename` 형태로 사용하고, 여기서 `@scope/` 같은 영역을 스코프라 함
- 스코프가 있는 패키지는 `node_modules/@scope/packagename`과 같이 스코프를 폴더로 추가해서 설치됨

#### 🔷 version

- npm 스트리에 업로드, 즉 패키지를 공개할 계획이라면 name 필드와 더불어 가장 신경써야 할 필드
  - 패키지를 공개할 계획이 없다면 name과 version 필드는 선택 사항
- name과 version의 조합은 항상 레지스트리 내부에서 고유해야 함

#### description

- 패키지에 대한 설명을 넣는 필드

#### keywords

- 패키지와 관련된 키워드를 입력하는 필드
- 보통 패키지를 설명할 수 있는 간단한 키워드 또는 패키지가 의존성을 가지고 있는 프레임워크나 라이브러리명을 선언해두는 것이 일반적

> ✨ `npm search`를 통해 description, keyword를 통해 패키지를 찾을 수 있음

#### homepage

- 패키지의 홈페이지 URL을 기재하는 곳

#### 🧐 npm info <packagename>

```bash
✓✓✓: npm info react

react@19.1.1 | MIT | deps: none | versions: 2483
React is a JavaScript library for building user interfaces.
https://react.dev/

keywords: react

dist
.tarball: https://registry.npmjs.org/react/-/react-19.1.1.tgz
.shasum: 06d9149ec5e083a67f9a1e39ce97b06a03b644af
mk4CMYhWIicdSflH91J9TyCyczcPFXJzrZ/ZXcgGRFeP6BU0BEJTw6tZdfQ==
.unpackedSize: 167.0 kB

maintainers:
- fb <opensource+npm@fb.com>
- react-bot <react-core@meta.com>

dist-tags:
beta: 19.0.0-beta-26f2496093-20240514               latest: 19.1.1
0-canary-de5a1b20-20250905
experimental: 0.0.0-experimental-de5a1b20-20250905  rc: 19.0.0-rc.1

published a month ago by react-bot <react-core@meta.com>
```

#### bugs

- 패키지에 버그가 있을 경우 제보할 수 있는 주소(url)나 이메일 주소(email)을 입력하는 필드

```json
{
  "bugs": {
    "url": "https://github.com/npm/example/issues",
    "email": "example@npmjs.com"
  }
}
```

- 만약 하나의 url로만 제공하고자 하는 경우, bugs 필드에 url을 문자열 형태로 입력도 가능

```json
{
  "bugs": "https://github.com/npm/example/issues"
}
```

### 🔷 license

- 패키지에 대한 라이선스를 지정하는 필드
- 패키지를 어떻게 사용할 수 있고, 어떤 제한이 있는지 알릴 수 있음
- 원하는 라이선스의 SPDX ID로 license 필드에 작성
  - SPDX ID(Software Package Data Exchange)
    - 오픈소스 소프트웨어 라이선스와 관련된 정보를 표준화된 방식으로 공유하고 전달하는데 사용
    - 각 라이선스를 식별할 수 있는 고유한 식별자를 통해 모든 소프트웨어에서 일관되게 사용하고 인지하는 데 목적
- 라이선스의 종류
  - MIT 라이선스
  - ISC 라이선스 (npm init으로 설치 시 기본값으로 사용되는 라이선스)
  - 아파치 라이선스 2.0
  - BSD 라이선스
- 라이선스 지정 방법
  1. 라이선스 ID를 작성하는 방법
  ```json
  {
    "license": "MIT"
  }
  ```
  2. 라이선스 파일을 명시하는 방법
  ```json
  {
    "license": "SEE LICENSE IN <LICENSE.md>"
  }
  ```
  3. 라이선스가 없는 경우 → 다른 사람이 사용하지 못하게 기재한 것이기 때문에 설치할 때 주의
  ```json
  {
    "license": "UNLICENSED"
  }
  ```

#### people: author와 contributors

- name, email, url 필드를 가진 `person` 객체를 사용할 수 있음
  - person 객체
  ```json
  {
    "name": "Barney Rubble",
    "email": "barney@npmjs.com",
    "url": "http://barnyrubble.npmjs.com/"
  }
  ```
  - 하나의 문자열로 작성하면 npm이 자동으로 파싱도 가능
  ```json
  {
    "author": "Barney Rubble <barney@npmjs.com> (http://barnyrubble.npmjs.com/)"
  }
  ```
- author는 한 명만 지정 가능하고, contributor는 여러 명 선언 가능

#### funding

- 패키지 개발에 직접적인 자금을 지원하는 방법에 대한 정보를 알려주는 필드

```json
{
  "funding": {
    "type": "individual",
    "url": "http://npmjs.com/donate"
  }
}

{
  "funding": {
    "type": "patreon",
    "url": "https://www.patreon.com/user"
  }
}

{
  "funding": "http://npmjs.com/donate"
}

{
  "funding": [
    {
      "type": "individual",
      "url": "http://npmjs.com/donate"
    },
    "http://npmjs.com/donate-also",
    {
      "type": "patreon",
      "url": "https://www.patreon.com/user"
    }
  ]
}
```

- npm fund 명령어로 프로젝트의 의존성에 있는 모든 funding 주소를 가져와서 보여줌
- 특정 패키지에 대한 펀딩 정보만 보는 것도 가능

#### 🔷 files

- npm 레지스트리에 업로드될 때 포함해야 할 파일 목록을 배열로 선언
- 패키지를 의존성으로 설치하는 경우 꼭 필요한 파일만 선택적으로 배포 가능
- 파일 패턴은 `.gitignore`와 유사한 문법으로 선언
- 일부 특수 파일 및 디렉토리는 파일 배열에 존재하는지 여부와 관계없이 포함되거나 제외
  - 항상 포함되는 파일
    - `package.json`
    - `README`
    - `LICENSE` / `LICENCE`
    - The file in the "main" field
    - The file(s) in the "bin" field
  - 항상 제외되는 파일
    - `.orig`
    - `.*.swp`
    - `.DS_Store`
    - `._*`
    - `.git`
    - `.hg`
    - `.lock-wscript`
    - `.npmrc`
    - `.svn`
    - `.wafpickle-N`
    - `CVS`
    - `config.gypi`
    - `node_modules`
    - `npm-debug.log`
    - `package-lock.json` (use [`npm-shrinkwrap.json`](https://docs.npmjs.com/cli/v11/configuring-npm/npm-shrinkwrap-json) if you wish it to be published)
    - `pnpm-lock.yaml`
    - `yarn.lock`
    - `bun.lockb`
- `.npmignore`를 사용하면 files 필드에 선언되었더라도 `.npmignore`에 의해 무시됨
  - `.npmignore`가 없다면 `.gitignore`가 대신
- ❗️ `.npmignore` 파일과 files 필드를 동시에 사용할 때는 주의 사항
  - 패키지 최상위에 있는 files 필드를 오버라이드하지 않지만 하위 디렉터리에서는 오버라이드
  - 패키지 최상위에 있는 `.npmignore`보다는 files에 있는 것이 우선시
  - 하위 패키지의 경우 반대로 `.npmignore`에 있는 것이 우선시되어 무시
- 일반적인 패키지의 일일이 파일을 지정하기보다는 glob 패턴을 사용해 여러 파일을 지정하는 것이 일반적
  - glob 패턴: 파일 경로 내의 집합을 지정하기 위해 사용되는 일종의 문자열 패턴
    - `-` : 0개 이상의 문자
    - `?`: 정확히 하나의 문자에 일치
    - `[]`: 이 안의 문자 중 하나와 일치
    - `**/`: 모든 디렉터리에 대한 재귀적인 탐색

#### main

- 패키지의 진입 파일
- main이 별도로 설정돼 있지 않다면 기본값으로 `index.js` 선언

#### browser

- 브라우저와 같은 클라이언트 측에서 사용하고자 한다면 main 필드 대신 browser 필드를 사용하여 패키지 진입 파일을 설정
- window 같은 객체에 의존 중인 모듈이 browser에 있다는 것을 알리는 데 유용

#### bin

- 실행 가능한 파일의 위치를 선언하는 곳
- `npm install -g <패키지명>`으로 전역 환경에 설치하면 해당 파일은 전역 bin 디렉터리 내부에 링크되거나 bin 필드에 지정된 파일을 실행하는 명령어가 생성되어 해당 이름으로 실행 가능
- 패키지가 다른 패키지의 의존성으로 설치된다면 `npm exec` 명령이나 다른 스크립트를 통해 해당 실행 파일을 호출 가능

```json
{
  "name": "create-react-app",
  "version": "5.0.1",
  "files": ["index.js", "createReactApp.js"],
  "bin": {
    "create-react-app": "./index.js"
  }
}
```

- `npm install -g create-react-app` 실행 → bin 필드에 있는 `create-react-app`을 npm이 전역 환경에 명령어로 등록 → `create-react-app <서비스명>` 실행 → `create-react-app`의 `index.js`가 실행
- 만약 bin에 객체 대신 파일명만 기재하면 `package.json`의 name으로 명령어가 등록됨
- Node.js로 실행될 파일을 bin 필드에 넣어뒀다면 이 파일 상단에 반드시 `#!/usr/bin/env node`를 선언해야 함 → 셔뱅(shebang)
- 셔뱅을 선언하면 굳이 bin 필드에 `node ./index.js` 로 지정하지 않아도 Node.js로 실행됨

#### 🔴 man

- 최근에는 패키지에 대한 매뉴얼이 거의 대부분 웹사이트 또는 깃허브에 존재하기 때문에 현대의 npm 패키지에서는 거의 사용되지 않는 필드
- man <패키지명>으로 실행하면 package.json에 있는 man 필드의 값이 유닉스 man 명령어로 실행됨

#### 🔴 directories

- CommonJS 1.0 사양에는 이 directories 필드를 선택 사항으로 선언해두었지만, 최근 들어 자주 사용되지 않는 필드
  - https://wiki.commonjs.org/wiki/Packages/1.0
- lib, src, docs, test, bin 등의 필드 선언 가능
- 프로젝트의 구조가 규격화되고, 굳이 패키지의 src나 test 등을 확인할 일도 별로 없기 때문에 사용되는 경우가 많이 줄어들었음

> _In the future, this information may be used in other creative ways._

#### repository

- 실제 패키지의 코드가 있는 곳을 기재하는 필드

```json
{
  "repository": {
    "type": "git",
    "url": "https://github.com/npm/cli.git"
  }
}
```

- GitHub, GitHub gist, Bitbucket, GitLab 등은 축약해서 사용 가능

```json
{
  "repository": "npm/example",

  "repository": "github:npm/example",

  "repository": "gist:11081aaa281",

  "repository": "bitbucket:user/repo",

  "repository": "gitlab:user/repo"
}
```

- 모노레포와 같이 package.json이 최상위 저장소 외에 하위 폴더에 있다면 해당 디렉터리 위치를 다음과 같이 명시 필요

```json
{
  "repository": {
    "type": "git",
    "url": "git+https://github.com/npm/cli.git",
    "directory": "workspaces/libnpmpublish"
  }
}
```

#### 🔷 scripts

- npm에서 기본적으로 제공하는 명령어뿐만 아니라 임의의 명령어를 선언해서 사용할 수 있는 필드
- `npm run-script <명령어>` 또는 `npm run <명령어>`를 통해 실행할 수 있음
- pre, post 접두사 지원
  - `npm run compress` 실행 시,
    - `npm run precompress` → `npm run compress` → `npm run postcompress` 순으로 실행

```json
{
  "scripts": {
    "precompress": "<compress 명령어가 실행되기 전에 실행할 내용>",
    "compress": "<compress 명령어에 따른 실행>",
    "postcompress": "<compress 명령어 이후에 실행할 내용>"
  }
}
```

- 특정 명령어 앞에 `키=값`을 지정하여 명령어가 실행되는 환경에서 `process.env.<키>`를 통해 값에 접근할 수 있음

```json
{
  "scripts": {
    "start": "REACT_PROFILE=dev next start"
  }
}
```

- 하지만 이러한 방식은 유닉스 계열의 운영체제에서만 사용가능하므로 윈도우에서는 사용 불가
  - 운영체제에 관계없이 환경변수를 일정하게 주입하기 위해 `dotenv`나 `cross-env` 사용을 권장
- scripts의 명령어로 다른 scripts를 실행할 수도 있음

```json
{
  "scripts": {
    "build": "next build",
    "test": "jest",
    "ci": "npm run build && npm run test"
  }
}
```

- 잘못 설정할 경우 무한루프에 빠지게 될 수도 있으므로, 명령어 간의 순환 참조를 피해야 함

```json
{
  "scripts": {
    "build": "npm run test",
    "test": "npm run build"
  }
}
```

#### 🔴 config

- 패키지에 scripts를 실행할 때 사용할 수 있는 다양한 설정 관련 값을 객체 형태로 모아둘 수 있음

```json
{
  "name": "foo",
  "config": {
    "port": "8080"
  }
}
```

- `npm_package_config_port` 형태로 패키지 내부의 자바스크립트에서 사용할 수 있음

```jsx
console.log(process.env.npm_package_config_port)
```

- 요즘에 들어서는 자주 사용하지 않음
  - json 형태로 작성해야 해서 관리가 어려움
  - 값에 접근하기 위해서는 긴 접두사(`npm_package_config`)가 필요
  - 따라서 실제 개발 환경에서는 `.env` 파일만으로 환경변수를 관리할 수 있는 `dotenv`를 더 많이 사용하는 편

#### 🔷 dependencies

- Node.js 프로젝트가 실행되는 데 필요한 외부 패키지 및 라이브러리를 정의하는 필드
- 프로젝트가 의존하는 패키지들과 해당 버전 범위가 명시
- dependencies 외에도 peerDependencies, devDependencies 등의 필드가 존재

#### overrides

- 패키지 자신이 참조하고 있는 의존성의 의존성 버전을 수정하고 싶을 때 유용
- 의존성 트리의 패키지 버전을 다른 버전 혹은 완전히 다른 패키지로 대체할 수 있음
- overrides로 패키지 의존성에 있는 모든 패키지의 버전 변경하기

```jsx
{
  "overrides": {
    "@npm/foo": "1.0.0"
  }
}
```

- overrides로 특정 패키지가 의존하고 있는 패키지 버전 변경하기

```jsx
{
  "overrides": {
    "@npm/foo": {
      "@npm/bar": "2.0.0"
    }
  }
}
```

- 주로 긴급한 보안 위협이 발생해서 빠르게 대응하고자 할 때 해당 패키지의 버전업을 기다릴 수 없는 경우에 주로 사용
  - https://blog.logto.io/ko/pnpm-upgrade-transitive-dependencies
- 사용하는 의존성 패키지의 일부의 버전 수정이 필요한데, 패키지가 새롭게 릴리즈되는 것을 기다리기 어려운 경우에도 사용
- 하지만 overrides는 **_패키지 개발자가 정해 놓은 의존성을 임의로 덮어쓰는 것_**으로 사용할 때 주의 필요

#### engines

- 패키지가 실행 가능한 Node.js 버전을 명시할 수 있음

```jsx
{
  "engines": {
    "node": "^18.19.1",
		"npm": "^10.4.0"
  }
}
```

- 사용자가 config 파일에서 engine-strict 설정과 활성화 하지 않았다면 단순히 경고만 노출

#### os

- 패키지가 실행 가능한 운영체제를 선언하고 싶을 때 사용하는 필드

```jsx
{
	"os": ["darwin", "linux"]
}
```

- !를 사용해 특정 os를 사용하지 못하게 할 수도 있음

```jsx
{
	"os": ["!win32"]
}
```

- Node.js는 process.platform 변수로 현재 운영체제를 판단하며, npm은 이를 package.json의 os와 비교해서 실행 가능 여부를 결정
- 아주 특별한 이유가 있는 것이 아니라면 거의 사용되지 않는 필드

#### cpu

- 특정 CPU 아키텍처를 선언하고 싶을 때 사용하는 필드
- Node.js에서 process.arch를 사용하면 실행 중인 환경의 CPU 아키텍처를 확인할 수 있음

```jsx
{
  "cpu": ["x64", "ia32"]
}
```

#### private

- 해당 필드를 활성화하면 npm이 해당 패키지를 절대로 npm 레지스트리에 업로드하지 않음
- 우발적으로 배포되는 것을 막는 최선의 보호 장치

#### 🔷 publishConfig

- 패키지를 배포할 때 필요한 설정 값을 선언할 때 사용
- 특정 패키지를 기본 npm 레지스트리가 아닌 다른 레지스트리에 배포하고 싶을 때 해당 레지스트리 주소 작성

```json
// GitHub 레지스트리에 업로드
{
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```

#### 🔷 workspaces

- npm@7부터 도입된 워크스페이스(workspace) 기능을 지원하기 위한 필드
- 워크스페이스란? 기본적으로 하나의 최상위 패키지 위에서 하위 여러 패키지를 관리하기 위한 방식
- 워크스페이스를 사용하면 최상위에 하나의 node_modules와 package-lock.json이 생기는 대신, 하위에 있는 패키지들은 최상위에 있는 node_modules를 보고 자신이 필요한 패키지들을 참조하거나 패키지 간에 서로 참조가 가능

```json
{
  "name": "workspace",
  "workspaces": ["./packages/*"]
}
```

```json
.
+-- package.json
`-- packages
	+-- a
	|  `-- package.json
	`-- b
		 `-- package.json
```

- npm workspaces 이외에도 yarn workspaces, lerna, Nx, Turborepo 등을 사용하여 모노레포를 관리

> ❗️ 여기서부터는 npm에서는 공식적으로 사용되지 않지만 Node.js에서 사용하는 필드

#### packageManager

- 실험적으로 운영 중인 필드
- 현재 프로젝트를 실행할 때 사용될 것으로 예상되는 패키지 관리자를 지정할 수 있음
  - npm, yarn, yarnpkg pnpm, pnpx 등에 `@version`을 추가해서 제공

```json
{
  "packageManager": "pnpm@8.14.3"
}
```

- Node.js에서 제공하는 corepack과 함께 유용하게 사용 가능
  - corepack은 Node.js에서 패키지 관리자를 사용하기 쉽게 도와주는 도구로, packageManager 필드와 함께 사용

```json
{
  "name": "test",
  "packageManager": "pnpm@8.14.3",
  "scripts": {
    "start": "node ./index.js"
  },
  "dependencies": {
    "react": "17.0.2"
  }
}
```

- `corepack enable`로 corepack 활성화 → yarn 실행하면

```json
$ yarn
Usage Error: This project is configured to use pnpm
```

- 반대로 packageManager를 yarn으로 변경하면 기존 로컬 시스템에 yarn이 없다고 하더라도 corepack이 package.json에 선언된 packageManager에 알맞은 yarn 버전을 설치해서 정상적으로 사용할 수 있게 해줌

> [!NOTE]
>
> #### 동작에는 아무런 문제가 없어 보이는데 왜 corepack은 여전히 실험 단계인가요?
>
> - Node.js에서 corepack을 기본으로 활성화할 것인가?
> - 만약 활성화한다면 Node.js에서 기본으로 동작하던 npm은 어떻게 변경해야 하는가?

#### type

- CommonJS와 ESModule 중 Node.js가 어떤 모듈 형식을 사용할지 알리는 필드
- commonjs(CommonJS) / module(ESModule)
- 선언하지 않으면 기본값은 commonjs

#### exports

- main의 대안으로, 패키지를 설치해서 사용하는 사용자에게 패키지의 진입점을 나타낼 수 있는 필드
- 하위 path를 상세하게 나타내거나 조건부 exports를 나타내는 데 사용
- 조건부 내보내기(conditional exports)를 사용하면 해당 패키지에서 CommonJS와 ESModule 두 가지를 동시에 지원하는 것이 가능
  - https://toss.tech/article/commonjs-esm-exports-field

#### imports

- 해당 패키지에서만 쓸 수 있는 구문으로, tsconfig의 경로 별칭을 지정할 수 있는 compilerOptions.paths와 동일하게 특정 불러오기에 대한 별칭을 지정할 수 있는 기능
- imports는 반드시 #으로 시작해야 함 → 다른 모듈 시스템이나 @scope와 같은 npm 시스템과의 충돌을 막기 위한 장치
- 적절히 활용 시 tsconfig.json의 path를 사용할 때와 마찬가지로 읽기 힘든 상대 경로를 깔끔하게 정리할 수 있음

#### 기타

- 외부 패키지 또는 라이브러리에서 사용하기 위한 설정 값 또한 package.json 내부에서 지정해서 활용 가능

```json
{
  "eslintConfig": {
    "extends": ["react-app", "react-app/jest"]
  },
  "lint-staged": {
    "**/*.{md}": "markdownlint",
    "**/*.{json,yaml}": "prettier --check",
    "**/*.{js,jsx,ts,tsx}": "eslint"
  },
  "msw": {
    "workerDirectory": "public"
  }
}
```

### package.json 생성하기

- `npm init` → cli를 통해 필수적인 내용 구성으로 package.json 설정 가능
- `npm init —yes`로 문답없이 초기 파일 생성도 가능
- `package.json`에 주석 다는 방법

  1. 주석 사용이 가능한 JSONC 파일 형식 사용
     - 주석이 포함된 JSON 파일을 처리해야 하므로 별도의 컴파일러 필요
  2. `//`키 사용

     - npm 팀에서 공식적으로 권장하는 방식

     ```json
     {
       "name": "test",
       "//": ["안녕하세요", "주석입니다."],
       "scripts": {},
       "dependencies": {}
     }
     ```

     - 단, scripts 안에 가 있다면 그 자체를 명령어로 판단하고, dependencies 안에 있다면 `//` 패키지를 시도하려고 하니 주의

     ```json
     {
       "name": "test",
       "scripts": {
         "//": "node index.js"
       },
       "dependencies": {
         "//": "hello, world"
       }
     }
     ```

  3. npm, Node.js 등에서 사용되지 않는 예약어 사용

     - 팀 단위로 확실하게 사용하기 위해서는 접두사 등의 규칙을 선언하는 것이 좋음

     ```json
     {
       "name": "test",
       "@scripts_comment": "아래 명령어를 수정하지 마세요",
       "scripts": {
         "start": "node index.js"
       }
     }
     ```

## npm config

> - npm에서 npm 레지스트리에 업로드하는 패키지 개발 시 주로 사용되는 설정 값

- `_auth`: npm 레지스트리에 인증할 때 사용되는 문자열 값 (기본값 null)
- 레지스트리: npm 패키지가 업로드되는 데이터베이스 주소
- engine-strict: package.json의 engines 필드를 엄격하게 적용할지 여부를 결정하는 값
- access: 패키지를 배포한 후, 다시 접근 레벨 제어하고 싶을 때 사용
- legacy-peer-deeps: peerDependencies가 맞지 않더라도 무시하고 과거 npm 버전과 동일하게 설치
- `.npmrc`: npm과 관련된 설정 값을 가지고 있는 파일로 config와 관련된 내용 기재 가능
  - `.npmrc` 파일 내 설정은 모두 키=값 형태로 지정
  - 프로젝트 최상위 경로에 위치한 .npmrc 파일이 가장 높은 우선순위
- `npm config list`로 현재 최상위 기준으로 npm 설정 확인 가능

> [!NOTE]
>
> #### 정리
>
> - package.json은 프로젝트의 메타데이터를 정의하고, 의존성 관리부터 스크립트 실행까지 다양한 역할을 수행하는 핵심 파일
> - 해당 프로젝트가 어떤 패키지를 필요로 하고 어떤 환경에서 동작하는지를 쉽게 파악할 수 있음

## 2.2 dependencies

### npm 버전과 버전에 사용되는 특수 기호

- 특정 버전을 선언하는 경우: 해당 버전과 정확히 일치하는 버전의 패키지만 설치
  ```json
  {
    "dependencies": {
      "react": "18.2.0"
    }
  }
  ```
- `^version` (캐럿): 호환되는 버전의 변경까지만 용인
  ```json
  {
    "dependencies": {
      "react": "^16.8.0"
    }
  }
  ```
- `~version` (틸드): 패치 버전의 변경까지만 용인
  ```json
  {
    "dependencies": {
      "react": "~16.8.0"
    }
  }
  ```
- `*` 애스터리스크: 아무 버전이나 상관 없으며, npm install이나 npm update 실행 시 실행 시점의 최신 버전 설치
- `x`: x.1.2, 1.x.2와 같이 사용할 수 있으며, x로 들어간 곳에는 어떠한 버전이 와도 상관없음을 의미
- `>` , `≥`, `≤`, `<`: 초과, 이상, 이하, 미만의 버전을 나타낼 때 사용
- `version || version`: 둘 중 하나를 만족하는 버전을 의미
- `version1 - version2`: 두 버전 사이의 범위를 만족하는 버전을 의미
- `http://`: .tar나 tar.gz 같이 tarball 파일이 업로드되어 있는 주소를 선언 가능
  - npm 레지스트리에 업로드하지 않고, 별도 CDN 등에 업로드한 패키지를 설치할 때 사용
- 깃허브 주소, git 주소, `file: {path}`를 통한 로컬의 특정 path에 있는 패키지 설치도 가능

### dependencies

- npm 패키지를 사용하거나 개발하기 위해 반드시 필요한 패키지를 의미
- npm install을 실행하면 이 항목에 정의된 패키지들이 자동으로 설치됨
- dependencies에 있는 패키지는 무조건 설치되기 때문에 npm으로 업로드될 패키지라면 dependencies를 신중하게 선언해야 함

```json
{
  "name": "a",
  "dependencies": {
    "b": "1.0.0"
  }
}
```

```json
{
  "name": "b",
  "dependencies": {
    "a": "1.0.0"
  }
}
```

```json
.
ㄴ node_modules/
		ㄴ a/
				ㄴ b
```

```json
.
ㄴ node_modules/
		ㄴ c/
				ㄴ a/
						ㄴ b
```

### devDependencies

- 해당 패키지를 사용하는 사용자가 아닌 해당 패키지를 개발하는 개발자가 필요로 하는 패키지를 의미
- 빌드, 테스트, 혹은 기타 패키지 이용에 직접적으로 필요하지 않은 개발 도구들이 devDependencies로 선언
- 사실상 npm으로 업로드되지 않는 패키지, 대다수의 애플리케이션 패키지의 경우 dependencies와 devDependencies의 차이가 없을 수 있음
  - 둘의 주요 차이점이 _‘해당 패키지가 다른 패키지에서 설치될 때 설치되는가?’_ 여부이기 때문에 이러한 차이점이 애플리케이션에서는 모호해짐

### peerDependencies

- 특정 호스트 패키지를 기반으로 작성된 패키지를 만들고 싶을 때 반드시 사용해야 하는 필드
  - 호스트 패키지: 해당 라이브러리나 패키지를 사용하는 프로젝트를 의미
- 호환성 선언 용도
  - 플러그인이 어떤 패키지와 호환되는지 명시적으로 선언하여 호환성을 보장
- 사용자에게 특정 패키지 설치에 주의를 주는 용도
  - 사용자 측에서 버전만 다른 동일한 무거운 패키지를 가지고 있을 때 중복 설치를 방지
  - peerDependencies로 선언해둔다면 패키지 관리자에 따라 설치가 정상적으로 진행되지 않거나 설치 후 오류 메시지 반환 (패키지 관리자마다 설정은 조금씩 다름)
  - 별다른 메시지 없이 무조건 설치하는 dependencies와는 달리 peerDependencies는 해당 구조가 만족하지 않는 경우 경고 문구를 내보내거나 설치를 막는 등의 추가적인 조치를 진행

### peerDependenciesMeta

- 선택적인 호스트 패키지를 선언할 때 유용
- `@yceffort/middleware`가 제공하는 형태가 Koa, Express, NestJS로 완벽히 격리되어 export 하고 있는 경우
  - 이 패키지를 사용하는 쪽에서는 세 개 중 하나의 경로만 취사선택해서 사용할 수 있어야 함

```jsx
import koaMiddelware from '@yceffort/middleware/koa'
import expressMiddleware from '@yceffort/middleware/express'
import nestjsMiddelware from '@yceffort/middleware/nestjs'
```

```jsx
// koa.js
import koa from 'koa'

export default function Middleware() {}

// express.js
import express from 'express'

export default function Middleware() {}

// nestjs.js
import { Middleware } from '@nestjs/common'

export default function Middleware() {}
```

- peerDependencies에 선언하는 경우 → 하나의 프레임워크만 사용하더라도 모두 선택해야 함

```jsx
{
	"name": "@yceffort/middleware",
	"peerDependencies": {
		"koa": "*",
		"express": "*",
		"nestjs": "*"
	}
}
```

- peerDependenciesMeta로 옵션 설정을 통해 세 개 중 하나의 프레임워크만 선택하더라도 특별히 에러를 반환하지 않게 설정할 수 있음
  - 패키지 목적에 부합
  - 사용하는 쪽에서 선택적으로 사용할 수 있게끔 도와주면서도 동시에 호환성 선언 가능
  - ❗️패키지가 없는 경우에도 에러를 발생하지는 않겠지만 실제로 코드를 실행하는 과정에서도 해당 패키지가 설치되어 있지 않다면 해당 패키지를 찾을 수 없다는 에러 발생

```jsx
{
	"name": "@yceffort/middleware",
	"peerDependencies": {
		"koa": "*",
		"express": "*",
		"nestjs": "*"
	},
	"peerDependenciesMeta": {
		"koa": {
			"optional": true
		},
		"express": {
			"optional": true
		},
		"nestjs": {
			"optional": true
		}
	}
}
```
