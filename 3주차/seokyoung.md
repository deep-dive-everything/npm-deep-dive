# 2장 package.json과 npm 파헤치기

## 2.3 npm의 주요 명령어

> **공식문서** > https://docs.npmjs.com/cli/v10/commands/npm

- npm run: 뒤에 오는 명령어를 package.json에서 찾아서 실행하는 역할
  - npm run을 직접 실행하는 과정에서는 보이지 않지만 node_modules가 PATH에 추가된다.

```tsx
{
	"scripts": {
		"lint": "node_modules/.bin/eslint ."
	}
}
```

- `npm install`과 `npm ci`: package.json에 선언된 의존성 설치
  - `npm install`: package-lock.json의 존재 유무와 관계없이 모두 실행할 수 있음
  - `npm ci`: package-lock.json이 있는 경우에만 실행할 수 있으며, package-lock.json의 내용을 수정하지 않고, 그대로 의존성 설치
- `npm update`: package.json에 명시된 버전을 기준으로 의존성 업데이트
  - `npm update —save`: package.json에 변경 사항 반영
- `npm dedupe`: 현재 패키지 트리를 기반으로 의존성 단순화 및 중복 제거
  - `npm dedupe —dry-run`: 의존성 트리가 어떻게 변경되는지 미리보기
- `npm ls`
  - `npm ls`: 현재 트리 구조 확인
  - `npm ls <package>`: package.json 내에서 해당 패키지가 설치되어 있는 모든 의존성 리스트업
- `npm explain <package>`:해당 패키지가 왜 설치됐는지에 대한 정보 확인(패키지명을 반드시 지정하여 실행)
  - 의존성을 만족하지 않는 경우에는 출력 결과에 포함되지 않음
- `npm audit`: 현재 의존하고 있는 패키지에서 취약점으로 알려진 패키지가 있는지 검사
  - `npm audit fix`: 유의적 버전을 해치지 않는 선에서 취약점 해결
  - `npm audio fix —force`: 유의적 버전을 지키지 않고 강제로 취약점 해결

> 보안 취약점을 손쉽게 해결할 수 있는 방법
> npm audit을 비롯한 여러 보안 취약점 해결을 도와주는 도구가 있지만 어디까지나 보조적인 도구

- 패키지에 보안 취약점이 있지만 해당 기능을 사용하지 않을 경우
- 보안 취약점이 있는 패키지에서 대응이 늦거나 포기한 경우
- 보안 취약점이 있는 패키지가 업데이트됐지만 이에 의존하는 패키지에서 대응하지 않는 경우
  >
- `npm publish`: 현재 패키지를 레지스트리에 업로드
  - 업로드된 패키지는 레지스트리에 등록되어 다른 사용자가 `npm install`로 설치해서 사용할 수 있음
  - 파일 업로드 규칙
    - 패키지 설치 및 배포에 필수적으로 필요한 파일들을 자동으로 업로드 - package.json, README.md, LICENSE 등
    - .gitignore와 .npmignore가 있다면 해당 파일 내에 명시된 파일이나 폴더는 업로드 무시
    - 만약 .gitignore와 .npmignore 두 파일 모두 존재한다면 .npmignore가 우선권을 가짐
    - node_modules를 비롯해 .DS_Store, .svn 파일도 무시
    - 만약 package.json에 files가 선언된 경우 해당 파일만 업로드
- `npm deprecate`: 업로드되어 있는 특정 패키지에 대해 사용자에게 경고 메시지 출력
- `npm outdated`: 현재 설치된 패키지 중 현재 시간 기준 최신 버전이 아닌, 업데이트가 가능한 패키지를 비교 및 확인 가능
  - Current: 현재 설치된 버전
  - Watned: 현재 package.json에 있는 의존성에서 유의적 버전을 만족하는 최대 버전
  - Latest: 해당 패키지의 최신 버전
  - Location: 해당 패키지가 설치된 위치
- `npm view`: 특정 패키지의 정보 확인
  - `npm info`, `npm show`, `npm v`, 등으로
  - 실행하는 위치에 상관없이 언제 어디서든 npm 명령어가 실행 가능한 CLI라면 사용 가능

## 2.4 npm install을 실행하면 벌어지는 일

### 의존성 트리 분석의 핵심 `@npmcli/arborist`

- node_modules와 package.json의 트리르 관리하기 위한 CLI 도구
- Arborist 클래스 인스턴스: path, registry등 의 인자를 받아 생성
  - path: 프로젝트의 최상위 디렉터리
  - registry: npm 패키지를 다운로드할 레지스트리

```tsx
const Arborist = require('@npmcli/arborist')

const arborist = new Arborist({
  path: '/path/to/project',
  registry: 'https://registry.npmjs.org',
})
```

- loadActual: node_modules 내부의 실제 트리를 확인할 수 있는 메서드
  - 파일 시스템을 직접 스캔해서 node_modules 디렉터리 내 모든 패키지 검색
    - 패키지 간 의존성 관계를 파악해 전체 의존성 트리 구성
  - npm install 과정에서 파일 시스템을 기반으로 한 트리 분석이 중요한 이유
    - package.json에 선언된 의존성과 실제 node_modules에 설치된 패키지가 일치하는지 확인하기 위해
    - 만일 두 트리 간에 차이가 발견되면 상황에 따라 node_modules를 다시 구축해야 할 수도 있음

```tsx
const Arborist = require('@npmcli/arborist')

const arborist = new Arborist({
  path: '/path/to/project',
  registry: 'https://registry.npmjs.org',
})

const actualTree = await arborist.loadActual()
```

- ArboristNode: 프로젝트의 의존성 트리 내 각 노드를 나타내는 객체
  - name: 패키지 이름
  - version: 패키지 버전
  - location: 패키지가 설치된 경로
  - path: 패키지가 설치된 절대 경로
  - resolved: 패키지의 tarball 경로
  - Edge: 의존성 관계를 나타낼 때 사용하는 객체
    - edgesIn(Set): 다른 노드들이 현재 노드를 의존하는 관계. 특정 노드가 다른 노드에 의해 여러 번 의존 관계를 맺을 수 있기 때문에 Set 자료구조 사용
    - edgesOut(Map): 현재 노드가 다른 패키지들에 대해 가지고 있는 의존성. 패키지의 의존성에 중복이 존재하지 않고, 키를 통해 빠르게 의존성을 찾기 위해 Map 자료구조 사용
  - type: 각 의존성이 선언된 형태
    - prod: dependencies
    - dev: devDependencies
    - peer: peerDependencies
    - optional: optionalDependencies
- loadVirtual: 가상의 트리를 만드는 메서드
  - loadActual이 node_modules를 직접 스캔해서 트리를 구축하는 것과 달리, loadVirtual은 package-lock.json이나 npm-shrinkwrap.json을 기반으로 의존성 트리 생성
  - 파일 시스템을 읽는 대신, 두 파일의 정보를 바탕으로 이상적이 트리를 메모리에 가상으로 생성하는 과정
- buildIdealTree: package.json과 package-lcok.json을 바탕으로 가장 이상적인 트리 생성하는 메서드
  - 트리 초기화, 락 파일 업데이트, 의존성 구축 및 버전 충돌 해결, 최종적으로 엔진이나 플랫폼 버전 등을 해석해 최적의 트리 생성
  - 이상적인 트리? package.json에 선언된 의존성 버전을 충족 하면서 중복 설치를 최소화하고 버전 충돌을 최소화한 구조
  - 충돌? 같은 이름의 패키지가 동일한 위치에 이미 존재하거나 부모 노드가 해당 패키지의 peerDependencies를 요구하는데 그 조건을 만족하지 못하는 상황을 포함
- reify: buildIdealTree에서 생성한 이상적인 트리를 실제로 구현하는 역할
  - 구현한다? 이상적인 트리를 node_modules에 설치하고 package-lock.json에 반영하는 과정
  - @npmcli/arborist에서 중요한 역할을 담당하며, loadActual, loadVirtual, buildIdealTree 등을 통해 생성된 트리를 실제 파일 시스템에 반영
- audit: npm 패키지의 취약점을 살펴보기 위해 사용하는 명령어
  - audit 명령어를 수행하는 AuditReport 클래스
    1. 취약점을 분석해야 하는 패키지 목록을 가져옴
    2. 패키지 목록을 가져온 다음, 취약점 분석을 위한 정보를 가져옴
    3. 2번에서 불러온 정보를 바탕으로 취약점을 분석하고 보고
  - reify 마지막 과정에 포함되어 있어, reify를 실행하면 audit이 자동으로 실행

### 패키지 설치를 위한 패키지, `pacote`

> @npmcli/arborist가 의존성 트리를 분석한 후, pacote를 통해 실제 패키지를 npm에서 가져와 설치하게 된다.

- manifest: 해당 패키지의 manifest 정보를 가져오는 메서드
  - @npmcli/arborist의 buildIdealTree에서 패키지의 manifest 정보를 가져온 후, 이 정보를 바탕으로 의존성 트리 구축할 때 사용됨
- tarball: 패키지의 tarball 데이터를 메모리에 불러오는 작업을 수행하는 메서드
- extract: tarball을 불러오는 것을 넘어서 파일을 압축 해제해서 파일 시스템에 저장하는 메서드

### node_modules 살펴보기

> npm install 이후 생성되는 node_modules 내에서는 무슨 일이 벌어질까?

```tsx
{
	"dependencies": {
		"react": "^18.2.0",
		"react-dom": "^18.2.0"
	}
}
```

```
.
├── js-tokens
├── loose-envify
├── react
├── react-dom
└── scheduler
```

- 평탄화된 node_modules: 모든 node_moudles 내부의 폴더를 평탄화해서 같은 레벨의 폴더에 일괄적으로 설치
  - 실제로 선언된 의존성은 두 개일 뿐이지만 해당 의존성이 의존하는 의존성을 포함해서 모든 패키지가 같은 폴더 내에 동일한 수준으로 설치됨
  - 모든 패키지가 동일한 수준으로 설치되는 이유
    - 동일한 패키지의 중복 문제
    - 깊으면 깊어질수록 덩달아 길어지는 경로
  - 평탄화 설치: 위의 문제를 막기 위해 npm@3부터 node_modules를 평탄화해서 될 수 있는 한 node_modules의 최상위에 설치하는 방식으로 문제 해결
  - 유령 의존성(phantom dependency): 평탄화 작업으로 인해 실제 의존성으로 선언하지 않은 패키지가 실행 가능해지는 문제
- npm이 중복 설치를 피하는 방법 → 유의적 버전 규칙 안에서 해결 가능할 때 package.json 의존성 해결이 이뤄짐

> 정리
>
> - npm install의 진행 과정
>   - npm 핵심 라이브러리인 @npmcli/arborist를 통해 수행
>   - arborist는 pacote를 이용해 패키지의 manifest 정보 가져오기
>   - tarball 데이터를 메모리에 불러오기
>   - extract로 압축을 해제해 파일 시스템에 저장하기
> - node_modules의 평탄화
>   - 설치 과정에서 중복을 제거하고, 가능한 한 효율적인 디렉터리 구조 유치

## 2.5 node_modules는 무엇일까?

### node_modules의 역할

- 의존성 관리: package.json에 명시된 의존성 목록을 기준으로 필요한 패키지를 설치해서 관리
  → 프로젝트 의존성을 일관되게 유지 가능
- 경로 해결: require() 함수나 import 문으로 모듈을 가져올 때 해당 모듈을 검색하는 주요 경로
  → 외부 모듈을 간편하게 로드 가능
- 네임스페이스 관리: 중복 모듈이 서로 영향을 미치지 않도록 각 패키지가 독립적으로 사용할 수 있는 네임스페이스 관리

### node_modules의 구조

```
// react, react-dom을 설치했을 때의 node_modules 구조
node_modules/
├── .bin
├── js-tokens
├── loose-envify
├── react
├── react-dom
└── schedular
```

- .bin: 각 패키지의 package.json에 정의된 bin 필드의 각 명령어를 참조해서 실행 가능한 명령어 제공
- 서브 패키지 node_modules 폴더: 의존하는 버전이 서로 다를 경우 개별 패키지의 node_modules 서브 폴더에 필요한 버전을 추가로 설치해서 관리
- .cache: 여러 빌드 도구와 패키지 관리 도구들이 성능 향상을 위해 캐시 데이터를 저장하는 장소
  - 웹팩: 빌드 성능을 향상시키기 위해 컴파일된 모듈과 기타 빌드 아티팩트를 .cache 폴더에 저장
  - 바벨: 동일한 파일을 다시 컴파일할 때 속도를 높이기 위해 .cache 폴더에 컴파일 결과 캐시
  - 빌드 도구와 컴파일러의 속도를 크게 향상
  - 반복적인 작업의 성능 최적화

### 심볼릭 링크(Symbolic Link)

- 파일 시스템 내에서 특정 파일이나 디렉터리에 대한 참조를 다른 위치에 생성하는 기능
- 실제 파일의 복사본이 아닌, 원본 경로를 참조해서 원본 파일에 접근하는 일종의 ‘링크’ 역할
- 심볼릭 링크를 통해 원본 파일의 변경 사항을 링크가 참조하는 모든 위치에서 즉시 반영 가능
- 심볼릭 링크 직접 활용하기
  - `npm link`: 패키지의 심볼릭 링크를 전역으로 생성
  - `npm link <package>`: 다른 패키지에서 대상 패키지의 심볼릭 링크 설치
  - 패키지를 npm 레지스트리에 배포하지 않고 로컬에서 직접 디버깅할 때 유용
- 심볼릭 링크의 장점
  - 개발 효율성: 개발 중인 패키지를 쉽게 테스트하고, 원본 파일을 수정할 때마다 즉시 반영할 수 있음
  - 저장 공간 절약: 단일 원본 파일에 대한 참조로 여러 위치에서 접근 가능
  - 빠른 업데이트: 심볼릭 링크로 연결된 파일은 원본 파일이 변경될 때 자동으로 최신 상태로 유지
- 로컬에서 심볼릭 링크를 제거하는 방법: 전역과 로컬 프로젝트에서 사용된 링크를 모두 제거해야 함!
  1. `npm unlink <package>`로 로컬 프로젝트에서 심볼릭 링크 제거
  2. `npm rm -g <package>로` 전역 심볼릭 링크 제거
- node_modules에서 심볼릭 링크 활용
  - node_modules/.bin: 프로젝트 내에 설치된 CLI 도구들을 효율적으로 관리하고, 쉽게 실행할 수 있도록 돕는 역할
  - 워크스페이스: 워크스페이스 사용 시, 각 패키지가 node_modules 폴더 내에서 심볼릭 링크로 연결되어 의존성을 설치할 때 별도의 패키지 관리 없이도 각 패키지 간의 참조가 자동으로 처리
    - 다수의 관련 패키지를 단일 저장소에서 효율적으로 개발하고 관리 가능
    - 의존성 충돌을 방지하고 일관된 버전 유지 가능지하고 일관ㄷ
  - 패키지 관리자: pnpm의 경우 디스크 공간 절약과 패키지 설치 속도 향상을 위해 심볼릭 링크를 적극적으로 활용

## 2.6 bin 필드와 npx

### CLI 패키지

- 명령줄 인터페이스(Command Line Interface)를 통해 사용되는 소프트웨어 패키지
- 주로 터미널 같은 명령 프롬프트에서 패키지 명령어를 입력해서 실행할 수 있음
- CLI 패키지 개발의 이점
  - 반복 작업의 자동화: 빌드, 배포, 테스트 등 반복적인 작업을 자동화
  - 프로젝트 구조화: 초기 설정과 구조화를 표준화해서 일관성 있는 개발 환경 구축 가능
  - 사용자 친화적인 인터페이스 제공: 복잡한 작업을 쉽게 수행할 수 있도록 보조

### bin 필드

- bin 필드 설정하기
- 패키지 설치와 실행
  - npx 혹은 npm exec를 통해 직접 접근
  - npm run-script로 호출

### npx

- npx의 주요 기능
  - 로컬 패키지 실행
  - 전역 설치 없이 실행
  - 특정 버전의 패키지 실행
  - 스크립트 실행
- npx에서 사용할 수 있는 옵션
  - `—package`(`-p`): 특정 패키지를 지정해서 명령어를 실행할 수 있는 옵션
  - `-c`: `npm run-script`와 비슷하게 셸 환경에서 문자열을 실행할 수 있으며, 여러 명령을 순차적으로 실행하거나 복잡한 구문의 명령을 실행하는 데 유용
  - `—yes`/`—no`: 패키지가 설치되어 있지 않으면 설치 여부를 물을 때 설정하는 옵션
    - `—yes`: 자동으로 패키지 설치 및 실행
    - `—no`: 패키지가 없으면 명령어를 실행하지 않음
- npx는 어떻게 패키지를 찾아서 실행할까?
  1. 입력 명령어 파싱
  2. 로컬 패키지 탐색
  3. 캐시 확인
  4. 레지스트리에서 패키지 검색
  5. 패키지 설치
  6. 패키지 실행
  7. 패키지 정리
- npx와 npm exec의 차이점
  - npx와 달리 npm exec은 —를 사용해 npm 명령어의 일관된 동작과 옵션 처리르 보장
  - 명령어와 그 옵션들을 더 직관적으로 구분 가능
