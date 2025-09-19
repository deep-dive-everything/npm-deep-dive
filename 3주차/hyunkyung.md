## 2.3 npm의 주요 명령어

### 2.3.1 npm run

- npm run 뒤에 오는 명령어를 package.json에서 찾아 실행하는 역할을 함
- npm run의 특별한 처리

```json
{
  "scripts": {
    "lint": "eslint ."
  }
}
```

- `npm run lint`는 되지만 `npm run eslint`는 실패하는 이유는 npm run을 실행하면 node_modules가 PATH에 추가되기 때문. 따라서 아래처럼 실행되는 것과 같은 것

```json
{
  "scripts": {
    "lint": "node_modules/.bin/eslint ."
  }
}
```

### 2.3.2 npm install과 npm ci

- 의존성을 설치하는 명령어
- npm install은 package-lock.json이 없거나 내용이 맞지 않아도 패키지를 설치할 수 있지만 npm ci는 에러를 발생시킴(ci 환경에서 정확하게 패키지를 설치하기 위한 목적)

### 2.3.3 npm update

- 의존성을 업데이트하는 명령어
- ^나 ~ 같은 버전 표기가 있을 때 해당 표기를 만족하는 범위에서 업데이트 진행

### 2.3.4 npm dedupe

- 현재 패키지 트리를 기반으로 의존성을 단순화하는 명령어
- 동일한 패키지의 여러 버전이 포함되거나 서로 의존하는 패키지 간에 중복된 의존성이 추가될 때 불필요한 중복 제거해줌

### 2.3.5 npm ls

- npm ls <패키지명>은 package.json 내에서 <패키지>가 설치되어 있는 모든 의존성 보여줌

### 2.3.6 npm explain

- npm ls와 비슷해보이지만 다름
- npm explain은 뒤에 반드시 패키지명을 지정해야함
- 정확히 이 의존성이 왜 설치되어있는지도 알려줌

### 2.3.7 npm audit

- npm에서 제공하는 보안 취약점을 검사하는 명령어

### 2.3.8 npm publish

- 현재 패키지를 레지스트리에 업로드하는 멸영어
- files가 선언된 경우 해당 파일만 업로드됨

### 2.3.9 npm deprecate

- 업로드 되어있는 특정 패키지에 대해 사용자에게 경고 메세지를 보여주는 명령어

### 2.3.10 npm outdated

- 현재 설치된 패키지 중 현재 시간 기준 최신 버전이 아닌, 업데이트가 가능한 패키지를 볼 수 있는 명령어

### 2.3.11 npm view

- 특정 패키지의 정보를 확인하는 명령어 (npm info, npm show, npm v와 같음)

## 2.4 npm install을 실행하면 벌어지는 일

### 2.4.1 의존성 트리 분석의 핵심 @npmcli/arborist

#### 2.4.1.1 loadActual

- node_modules 내부의 실제 트리를 확인할 수 있는 메서드
- package.json과 node_modules에 설치된 패키지가 일치하는지 확인해야할때 사용할 수 있음

#### 2.4.1.2 ArbistNode

- 의존성 트리의 각 노드를 구성하는 객체
- ArbistNode가 담고있는 정보
  - name: 패키지의 이름
  - version: 버전
  - loacation: 패키지가 설치된 상대 경로
  - path: 패키지가 설치된 절대 경로
  - resolved: 패키지의 tarball 경로
  - Edge: 의존성 관계를 나타낼 때 사용하는 객체
  - type: type은 각 의존성이 어떤 종류로 선언되어 있는지를 나타냄
    - Prod: dependencies
    - dev: devDependencies
    - peer: peerDependencies
    - optional: optionalDependencies

#### 2.4.1.3 loadVirtual

- loadActual과 달리 가상의 트리를 만드는 메서드
- node_modules를 직접 스캔해서 트리르 구축하지 않고 package-lock.json이나 npm-shrinkwrap.json을 기반으로 의존성 트리 생성
- 두 파일의 정보를 바탕으로 이상적인 트리를 메모리에 가상으로 생성

#### 2.4.1.4 buildIdealTree

- package.json과 Package-lock.json을 바탕으로 가장 이상적인 트리가 만들어짐
- 중복 설치와 버전 충돌이 최소화된 버전
- 아래와 같은 과정을 거침
  - initTree(): 초기 트리를 구축하기 위한 기본 설정 작업 수행. 최적의 트리를 만들기 위한 초기 변수 생성
  - inflateAncientLockfile(): 오래된 락 파일이 있는지 확인하는 작업을 거침
  - applyUserRequests: 사용자의 요청을 적용하는 작업(최상위 노드를 기준으로 패키지를 수정하거나 제거하는 작업)
  - buildDeps(): 이상적인 트리를 만들기 위한 본격적인 작업. Package.json을 분석해서 각 의존성을 해석하고 트리 구축
  - fixDepFlags(): 각 패키지는 설치된 유형에 따라서 별도의 플래그가 생성되는데 이 플래그를 설정하는 메서드
    - Extraneous: package.json 파일에 명시되어 있지 않지만 Node_modules에는 존재하는 패키지
    - peer: peerDependencies에 명시된 패키지
    - dev: devDependencies에 명시된 패키지
    - optional: optionalDependencies에 명시된 패키지
  - pruneFailedOptional(): 의존성을 불러오는 데 실패한 작업이 존재한다면 이 과정에서 에러 발생
  - checkEngineAndPlatform(): 완성된 트리에서 Node.js 버전, npm 버전 등을 검사하여 에러를 발생시키거나 경고메세지 출력

#### 2.4.1.5 reify

- buildIdealTree에서 생성한 이상적인 트리를 실제로 구현하는 역할
- 아래와 같은 과정 거침
  - 먼저 Reify가 실행되는 위치를 확인하고, node_modules 디렉터리가 있는지 확인한 후 없으면 생성
  - actual 트리와 ideal 트리 각각 생성
  - 두 트리를 비교하여 필요한 변경 사항을 확인하고 node_modules에 단계별로 설치하거나 제거
  - 모든 변경이 완료되면 이상적인 트리의 내용을 현재 트리에 복사해서 동기화
  - 의존성에 대한 보안 취약점을 검사하고 발견된 취약점에 대한 정보 제공
- npm install과 npm ci를 실행할 때 reify 메서드 사용됨

#### 2.4.1.6 audit

- 취약점을 분석하고 보고하는 역할. npm install의 마지막에 실행됨

### 2.4.2 패키지 설치를 위한 패키지, Pacote

- arborist는 의존성 트리를 분석하는 역할을 맡는다면, 실제 패키지를 npm에서 가져오는 역할은 별도의 패키지인 pacote가 담당

#### 2.4.2.1 manifest

- 해당 패키지의 manifest 정보를 가져오는 메서드로, 패키지의 이름과 버전을 인자로 받아 해당 패키지의 정보를 가져옴
- 이 정보를 바탕으로 의존성 트리를 구축

#### 2.4.2.2 tarball

- 패키지의 tarball 데이터를 메모리에 불러오는 작업 수행
- 이 데이터는 .tgz 확장자의 Buffer 형태로 반환되며, 이 데이터를 바탕으로 패키지를 설치하거나 관리하는 데 사용됨

#### 2.4.2.3 Extract

- tarball을 불러오는 것을 넘어서 파일을 압축 해제해서 파일 시스템에 저장

### 2.4.3 node_modules 살펴보기

#### 2.4.3.1 평탄화된 node_modules

- 어떤 패키지가 어떤 패키지에 의존하는 구조라도 트리 형식으로 설치되는 것이 아닌 동일한 레벨에 평탄화되어 설치(호이스팅)
  - 평탄화 이전의 방식은 동일한 패키지가 중복 설치되거나 구조가 깊을 수록 경로가 깊어져 비효율적이었음
- 하지만 실제로 의존성으로 선언하지 않은 패키지가 실행 가능해진다는 문제점(유령의존성)이 생김

#### 2.4.3.2 npm이 중복 설치를 피하는 방법

- 여러 패키지가 버전만 다른 B라는 패키지에 의존할 때(유의적 버전 충돌이 일어날 때) 하위폴더에 중복 설치를 통해 각 패키지가 필요로 하는 특정 버전을 만족시킴
