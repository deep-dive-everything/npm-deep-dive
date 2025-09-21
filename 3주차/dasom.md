# Npm Deep Dive
Dasom, 2025.09.20

## 2장 package.json과 npm 파헤치기

### 2.3 npm의 주요 명령어

> npm은 다양한 명령어를 제공하여 패키지 관리, 의존성 설치, 보안 검사 등의 작업을 효율적으로 수행할 수 있게 해준다.

#### 2.3.1 npm run

* package.json의 scripts에서 명령어를 찾아 실행하는 역할
* npm run 실행 시 node_modules/.bin이 PATH에 자동으로 추가됨
* 스크립트에 인수를 전달할 때는 `--` 사용: `npm run start -- --port=3000`

**실행 환경 차이 주의사항**
* npm run script와 bash에서 직접 실행하는 script의 위치가 다를 수 있음
* 이로 인해 실행 결과에 차이가 발생할 수 있으므로 주의 필요

#### 2.3.2 npm install과 npm ci

**npm install**
* package-lock.json 존재 유무와 관계없이 실행 가능
* package-lock.json이 없으면 package.json 기준으로 설치 후 락 파일 새로 생성
* 특정 패키지만 설치하는 것도 가능
* package-lock.json을 수정하는 경우가 있음

**npm ci**
* package-lock.json이 반드시 있어야 실행 가능
* package-lock.json 내용 그대로 의존성을 정확히 설치
* CI 환경에서 정확한 패키지 설치를 위한 목적으로 설계됨

#### 2.3.3 npm update

* package.json에 명시된 버전을 기준으로 의존성 업데이트 진행
* `^`나 `~` 같은 버전 표기가 있을 때 해당 표기를 만족하는 범위에서 업데이트
* 업데이트 후에는 CI를 통한 서비스 빌드 무결성 확인 등 최소한의 절차 필요

#### 2.3.4 npm dedupe

* 현재 패키지 트리를 기반으로 의존성을 단순화하는 명령어
* 패키지 트리 간 해결 가능한 의존성이 있다면 이를 단순화
* `--dry-run` 인수로 변경사항을 미리 확인 가능

#### 2.3.5 npm ls

* 현재 node_modules 구조 및 버전 확인 가능
* package.json 명시 의존성과 실제 설치된 의존성의 차이 파악 가능
* peerDependencies 구조가 올바르게 설치되었는지 확인할 때 매우 유용
* `npm ls <패키지명>`: 해당 패키지가 설치된 모든 의존성 표시

#### 2.3.6 npm explain

* `npm explain <패키지명>` 형태로 사용
* 대상 의존성이 왜 설치되었는지에 대한 정보까지 확인 가능
* 의존성을 만족하지 않는 경우 결과에 노출되지 않아 쉬운 파악 가능

#### 2.3.7 npm audit

* npm에서 제공하는 보안 취약점 검사 명령어
* 현재 의존 중인 패키지에서 취약점으로 알려진 패키지가 있는지 검사
* `npm audit fix`: npm이 해결 가능한 취약점을 적절한 방법으로 해결
* `npm audit fix --force`: 유의적 버전을 지키지 않고 강제 해결 (주의 필요)

#### 2.3.8 npm publish

* 현재 패키지를 레지스트리에 업로드하는 명령어

**업로드 규칙**
* package.json, README.md, LICENSE 등 필수 파일은 자동 업로드
* .gitignore와 .npmignore 파일에 명시된 파일/폴더는 업로드되지 않음
* .npmignore가 .gitignore보다 우선권을 가짐
* package.json의 files 필드가 선언된 경우 해당 파일만 업로드

**주의사항**
* 한번 npm에 업로드된 파일을 되돌리는 것은 매우 어려우므로 신중히 사용

#### 2.3.9 npm deprecate

* 업로드되어 있는 특정 패키지에 대해 사용자에게 경고 메시지를 보여주는 명령어
* 특정 버전 또는 패키지 전체를 지원 중단으로 처리 가능
* 적절한 메시지를 필수로 작성해야 함
* 지원 중단 처리를 되돌리려면 빈 문자열("")을 메시지로 지정

#### 2.3.10 npm outdated

* 현재 설치된 패키지 중 업데이트가 가능한 패키지를 확인하는 명령어

**확인 가능한 정보**
* Current: 현재 설치된 버전
* Wanted: package.json 의존성에서 유의적 버전을 만족하는 최대 버전
* Latest: 해당 패키지의 최신 버전
* Location: 해당 패키지가 설치된 위치

#### 2.3.11 npm view

* 특정 패키지의 정보를 확인하는 명령어
* `npm info`, `npm show`, `npm v`와 동일한 명령어
* 실행하는 위치에 상관없이 npm 명령어가 실행 가능한 CLI라면 사용 가능

### 2.4 npm install을 실행하면 벌어지는 일

> npm install을 실행하여 설치하는 과정은 생각보다 복잡한 절차를 거쳐 수행된다. package.json에 포함된 의존성이 매우 복잡하게 얽혀있기 때문에 실제로 어떤 패키지를 설치해야 하는지, 버전이 어떻게 결정되는지를 파악하고 결정된 버전을 node_modules에 어떻게 설치하는지 이해해야 한다.

#### 2.4.1 의존성 트리 분석의 핵심 @npmcli/arborist

* node_modules와 package.json의 트리를 관리하기 위한 CLI 도구
* Arborist 클래스의 인스턴스로 path와 registry 등의 인자를 받아 생성
* path는 프로젝트의 최상위 디렉터리, registry는 npm 패키지를 다운로드할 레지스트리

**2.4.1.1 loadActual**
* node_modules 내부의 실제 트리를 확인할 수 있는 메서드
* 파일 시스템을 직접 스캔해서 node_modules 디렉터리 내 모든 패키지 검색
* 패키지 간의 의존성 관계를 파악해 전체 의존성 트리 구성

**2.4.1.2 ArboristNode**
* 프로젝트 의존성 트리 내 각 노드를 나타내는 객체
* ArboristNode 클래스의 인스턴스로 노드와 관련된 중요한 정보를 담고 있음

**주요 속성**
* `name`, `parent`, `children`: 물리적 트리(폴더) 구조
* `package`: 해당 노드의 package.json 내용
* `path`, `realpath`, `location`: 경로 정보
* `edgesOut`, `edgesIn`: 의존성 그래프(이 노드가 요구/요구받는 관계)
* `dev`, `optional`, `peer`, `extraneous`: 의존성 유형/가지치기 판단 플래그

**2.4.1.3 loadVirtual**
* loadActual과 달리 가상의 트리를 만드는 메서드
* package-lock.json이나 npm-shrinkwrap.json을 기반으로 의존성 트리 생성
* npm cli에서 대표적으로 사용되며, 락 파일이 있는 경우에만 실행 가능

cf. npm-shrinkwrap.json은 package-lock.json과 거의 비슷하게 프로젝트의 정확한 의존성 트리를 고정하는 파일로, 배포 시에도 포함되어 의존성 버전을 강제로 고정하는 package-lock.json의 배포용 버전이다.

**2.4.1.4 buildIdealTree**
* @npmcli/arborist 라이브러리의 핵심 메서드
* package.json(요구사항)과 lockfile, 레지스트리 메타데이터 등을 바탕으로 이상적인 설치 상태(ideal tree)를 계산

**buildIdealTree가 충돌을 해결하며 최적의 트리를 구축하는 과정**
* initTree(): 초기 트리를 구축하기 위한 기본 설정 작업
* inflateAncientLockfile(): 오래된 락 파일이 있는지 확인
* applyUserRequests(options): 사용자의 요청을 적용
* buildDeps(): 이상적인 트리를 만들기 위한 본격적인 작업 수행
* fixDepFlags(): 각 패키지의 설치된 유형에 따른 플래그 설정
* pruneFailedOptional(): 의존성을 불러오는 데 실패한 작업이 있다면 에러 발생
* checkEngineAndPlatform(): 완성된 최적의 트리에서 Node.js 버전, npm 버전 등을 검사하여 현재 환경과의 호환성 확인

**2.4.1.5 reify**
* buildIdealTree에서 생성한 이상적인 트리를 실제로 구현하는 역할
* 이상적인 트리를 node_modules에 설치하고 package-lock.json에 반영하는 과정

**reify 실행 단계**
* reify 실행 위치 확인 후 node_modules 디렉터리 존재 여부 확인, 없으면 생성
* actual 트리와 ideal 트리를 각각 생성
* 생성된 두 트리를 비교
* 두 트리 간의 차이를 바탕으로 필요한 변경 사항 확인 후 패키지를 node_modules에 단계별로 설치하거나 제거
* 모든 변경 완료 후 이상적인 트리의 내용을 현재 트리에 복사하여 동기화
* 의존성에 대한 보안 취약점을 검사 후 발견된 취약점 정보 제공

**2.4.1.6 audit**
* npm 패키지의 취약점을 살펴보기 위해 사용하는 명령어
* @npmcli/arborist에서 audit 명령어를 수행하는 메서드 제공
* AuditReport 클래스에서 수행하는 작업: 취약점 분석 패키지 목록 가져오기 → 취약점 분석 정보 가져오기 → 취약점 분석 및 보고

#### 2.4.2 패키지 설치를 위한 패키지, pacote

> npm 패키지의 다운로드와 관리를 수행하는 라이브러리로, @npmcli/arborist가 의존성 트리를 분석한 후 이 라이브러리를 통해 실제 패키지를 설치한다.

**2.4.2.1 manifest**
* 해당 패키지의 manifest 정보를 가져오는 메서드
* 패키지의 이름과 버전을 인자로 받아 해당 패키지의 정보를 가져옴
* name, version, dependencies, dist, engines 등의 정보를 포함

**2.4.2.2 tarball**
* 패키지의 tarball 데이터를 메모리에 불러오는 작업을 수행

**2.4.2.3 extract**
* tarball을 불러오는 것을 넘어서 파일을 압축 해제 후 파일 시스템에 저장

#### 2.4.3 node_modules 살펴보기

**2.4.3.1 평탄화된 node_modules**
* npm은 의존성을 설치할 때 가능한 한 상위 node_modules 폴더에 패키지를 올려두는 방식을 사용
* 중복되는 하위 의존성을 각 패키지별로 중첩 설치하지 않고, 버전이 충돌하지 않는 한 루트에 한 번만 설치하여 트리를 평평하게 만듦

**평탄화의 장점**
* 중복 제거: 디스크 용량 절약
* 로드 속도 개선: require() 시 경로 탐색이 단순해짐
* 윈도우 경로 제한 문제 완화: 깊은 중첩 디렉터리 구조 방지

**평탄화의 단점**
* 버전 충돌: 서로 다른 버전이 필요한 경우 공통 버전은 루트에 두고 충돌 나는 버전만 해당 패키지의 node_modules 안에 별도로 설치
* 호이스팅의 의존성 문제: 상위에 끌어올려진 패키지를 의도치 않게 다른 모듈에서 require할 수 있어 잠재적 버그 발생 가능

**2.4.3.2 npm이 중복 설치를 피하는 방법**

**같은 버전 범위의 경우 (root에 한 번만 설치하여 중복 회피)**
* 여러 패키지가 동일한 버전 범위의 의존성을 요구하는 경우
* 루트 node_modules에 한 번만 설치하여 중복 제거

**버전 충돌의 경우 (공통은 root, 충돌 버전만 하위에 별도 설치)**
* 서로 다른 버전을 요구하는 경우
* 루트에는 최신/호환 가능한 버전 1개
* 충돌하는 버전만 필요한 패키지의 node_modules 내부에 중첩 설치
* 가능한 건 root에 공유, 충돌 나는 것만 필요한 패키지의 하위에 중첩 설치
