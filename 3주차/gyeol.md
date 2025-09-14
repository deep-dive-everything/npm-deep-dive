## 2.3 npm의 주요 명령어

### 2.3.1 npm run

- npm run 뒤에 오는 명령어를 package.json에서 찾아서 실행하는 역할
- script에 인수를 전달하고 싶으면 —를 사용하면 됨
    
    ```jsx
    {
    	"scripts": {
    		"start": "node index.js"
    	}
    }
    ```
    
    - `npm run start -- --port=3000`
- npm run script에서 script가 실행되는 위치와 실제 bash에서 실행하는 script의 위치는 다를 수 있으므로 실행 결과에 차이가 발생할 수 있음
    - e.g. eslint

### 2.3.2 npm install과 npm ci

- 공통점
    - 둘다 의존성을 설치하는 명령어
- npm install
    - package-lock.json의 존재 유무와 관계없이 모두 실행 가능
    - package-lock.json이 없는 경우
        - package.json에 선언된 버전 공식에 맞춰 설치 후 이 내용을 담은 package-lock.json을 새로 생성
    - package-lock.json이 있는 경우
        - 해당 내용에 따라 설치
    - package-lock.json을 수정하는 경우
        - package-lock.json에 없는 패키지를 설치할 때
        - package-lock.json의 package.json 내용과 맞지 않는 경우
    - 뒤에 패키지명을 선언하여 특정 패키지만 설치하는 것도 가능
- npm ci
    - package-lock.json이 있는 경우에만 실행 가능하며 package-lock.json의 내용 그대로 의존성을 설치함
    - npm ci의 목적은 package-lock.json에 명시된 의존성을 정확하게 설치하는 것이므로 package-lock.json이 없음면 설치가 불가능

### 2.3.3 npm update

- 의존성을 업데이트하는 명령어로 package.json에 명시된 버전을 기준으로 업데이트 진행
- package.json에 `^` 나 `~` 처럼 버전 표기가 있는 경우 해당 표기를 만족하는 범위에서 업데이트 진행
- npm update 이후에는 CI를 통해 서비스 빌드의 무결성을 확인하는 등 최소한의 절차를 갖추는 게 좋음

### 2.3.4 npm dedupe

- 현재 패키지 트리를 기반으로 의존성을 단순화하는 명령어
- 패키지 트리 간 해결할 수 있는 의존성이 있다면, 이를 단순화함
- `—dry-run` 인수를 추가하면 node_modules나 package-lock.json 업데이트 전 변경사항을 미리 확인할 수 있음

### 2.3.5 npm ls

- 현재 node_modules 구조 및 버전 확인 가능
- package.json에 명시된 의존성과 실제로 설치된 의존성의 차이 확인이 가능하며 의존성 관계가 잘못 형성된 경우 원인 파악도 가능함
- peerDependencies를 만족하는 구조로 올바르게 설치돼있는지 확인할 때 매우 유용
- `npm ls <패키지명>` 은 package.json 내에서 <패키지>가 설치돼 있는 모든 의존성을 보여줌
- `npm ls` 는 현재 트리 구조를 간략하게 확인할 수 있음

### 2.3.6 npm explain

- npm explain <패키지명> 의 형태로 사용해야하며, 대상 의존성이 왜 설치되었는지에 대한 정보까지 확인 가능 → 의존성을 만족하지 않는 경우 이 결과에 노출되지 않으므로 쉬운 파악이 가능함

### 2.3.7 npm audit

- npm에서 제공하는 보안 취약점을 검사하는 명령어
- 현재 의존하고 있는 패키지에서 취약점으로 알려진 패키지가 있는지 검사를 진행
- npm audit fix 명령어를 통해 npm이 해결가능한 취약점을 적절한 방법으로 해결할 수 있음
    - npm audit fix —force를 사용해 유의적 버전을 지키지 않고 강제 해결하는 방법도 있음. 하지만 이는 빌드 실패나 패키지에 이상이 생기는 등 문제가 생길 수 있으므로 주의 필요

### 2.3.8 npm publish

- 현재 패키지를 레지스트리에 업로드하는 명령어
- 명령어를 실행한다고 해서 무조건 업로드 되는 건 아님. 아래의 규칙을 지켜야 업로드가 가능
    - package.json, README.md, LICENSE와 같이 패키지 설치 및 배포에 필수적으로 필요한 파일들은 자동으로 업로드됨
    - .gitignore와 .npmignore가 있다면 해당 파일 내에 명시된 파일이나 폴더는 업로드 되지 않음(두 파일이 모두 존재할 경우 .npmignore가 우선권을 가짐)
    - node_modules를 비롯해 .DS_Store, .svn 같은 파일도 무시됨
    - 만약 package.json에 files가 선언된 경우 해당 파일만 업로드됨
- 한번 npm에 업로드된 파일을 돌리는 것은 매우 어려우므로 npm publish는 신중히 사용해야함

### 2.3.9 npm deprecate

- 업로드돼 있는 특정 패키지에 대해 사용자에게 경고 메세지를 보여주는 명령어
- 특정 버전 또는 패키지 전체를 지원중단으로 처리할 수 있음
- npm deprecate 명령어를 실행하기 위해서는 적절한 메세지를 필수로 작성해야함
- 만약 지원 중단 처리를 되돌리고 싶다면 메세지에 “”와 같이 빈 문자열을 지정하면 됨

### 2.3.10 npm outdated

- 현재 설치된 패키지 중 현재 시간 기준 최신 버전이 아닌 업데이트가 가능한 패키지를 볼 수 있는 명령어
- 이 명령어를 실행하면 현재 설치된 패키지 중 가장 최신 버전과 비교해서 어떤 패키지가 업데이트 가능한지 확인이 가능함
- npm outdated를 통해 확인할 수 있는 내용들
    - Current
        - 현재 설치된 버전
    - Wanted
        - 현재 package.json에 있는 의존성에서 유의적 버전을 만족하는 최대 버전
    - Latest
        - 해당 패키지의 최신 버전
    - Location
        - 해당 패키지가 설치된 위치

### 2.3.11 npm view

- npm info, npm show, npm v와 같은 명령어로 특정 패키지의 정보를 확인하는 명령어
- 실행하는 위치에 상관없이 npm 명령어가 실행 가능한 CLI라면 사용 가능

## 2.4 npm install을 실행하면 벌어지는 일

- npm install을 실행하여 설치하는 과정은 생각보다 복잡한 절차를 거쳐 수행됨
- package.json에 포함된 의존성은 매우 복잡하게 얽혀있음 → 실제로 어떤 패키지를 설치해야 하는지, 버전이 어떻게 결정되는지를 파악해야하며 결정된 버전을 node_modules에 어떻게 설치하는지 이해해야함

### 2.4.1 의존성 트리 분석의 핵심 @npmcli/arborist

- @npmcli/arborist는 node_modules와 package.json의 트리를 관리하기 위한 CLI 도구
    
    ```jsx
    const Arborist = require('@npmcli/arborist')
    
    const arborist = new Arborist({
    	path: 'path/to/project',
    	registry: 'https://registry.npmjs.org',
    	// 이외에 레지스트리와 관련된 다양한 인증 정보를 전달할 수 있음
    })
    ```
    
    - arborist는 Arborist 클래스의 인스턴스로 path와 registry 등의 인자를 받아 생성됨
    - path는 프로젝트의 최상위 디렉토리를 가리키며, registry는 npm 패키지를 다운로드할 레지스트리를 가리킴

### 2.4.1.1 loadActual

- arborist 클래스 인스턴스에서 제공하는 대표적인 메서드로, node_modules 내부의 실제 트리를 확인할 수 있음
- 파일 시스템을 직접 스캔해서 node_modules 디렉터리 내 모든 패키지를 검색함. 이 과정에서 패키지 간의 의존성 관계를 파악해 전체 의존성 트리를 구성하게 됨
    
    ```jsx
    const Arborist = require('@npmcli/arborist')
    
    const arborist = new Arborist({
    	path: 'path/to/project',
    	registry: 'https://registry.npmjs.org',
    })
    
    const actualTree = await arborist.loadActual()
    ```
    

### 2.4.1.2 ArboristNode

- 프로젝트 의존성 트리 내 각 노드를 나타내는 객체
- 이 객체는 ArboristNode 클래스의 인스턴스로 노드와 관련된 중요한 정보를 담고 있음
- 주요 속성
    - `name`, `parent`, `children` : 물리적 트리(폴더) 구조
    - `package` : 해당 노드의 `package.json` 내용
    - `path`, `realpath`, `location` : 경로 정보
    - `edgesOut`, `edgesIn` : 의존성 그래프(이 노드가 요구/요구받는 관계)
    - `dev`, `optional`, `peer`, `extraneous` : 의존성 유형/가지치기 판단 플래그
    - `resolve(name)` : `require(name)` 시 실제로 해석될 노드 찾기

### 2.4.1.3 loadVirual

- loadActual와 달리 가상의 트리를 만드는 메서드
- **loadActual은 node_modules를 직접 스캔해 트리를 구축**하지만 **loadVirual은 package-lock.json이나 npm-shrinkwrap.json을 기반으로 의존성 트리를 생성**함(lockfile을 기반으로)
- **npm-shrinkwrap.json**
    - package-lock.json과 거의 비슷하게, 프로젝트의 정확한 의존성 트리(패키지 버전과 서브 의존성까지 모두)를 고정(lock)하는 파일
    - 배포 시에도 포함되어 의존성 버전을 강제로 고정하는, package-lock.json의 배포용 버전
- loadVirual이 대표적으로 사용되는 곳은 npm cli → npm cli 역시 락 파일이 있는 경우에만 실행이 가능함.

### 2.4.1.4 buildIdealTree

- @npmcli/arborist 라이브러리의 핵심 메서드
- 이 메서드를 호출하면 package.json(요구사항)과 lockfile, 레지스트리 메타데이터 등을 바탕으로 **이상적인 설치 상태(ideal tree)**를 계산함
- buildIdealTree가 충돌을 해결하며 최적의 트리를 구축하는 과정
    - initTree()
        - 초기 트리를 구축하기 위한 기본 설정 작업을 수행
    - inflateAncientLockfile()
        - 오래된 락 파일이 있는지 확인하는 작업 수행
    - applyUserReuqests(options)
        - 사용자의 요청을 적용하는 작업을 수행
    - buildDeps()
        - 이상적인 트리를 만들기 위한 본격적인 작업을 수행. package.json을 분석 후 각 의존성을 해석하여 이를 기반으로 트리를 구축함
    - fixDepFlags()
        - 각 패키지는 설치된 유형에 따라 별도의 플래그가 생성되는데, 이러한 플래그를 설정하는 메서드
    - pruneFailedOptional()
        - 의존성을 불러오는 데 실패한 작업이 존재한다면 이 과정에서 에러를 발생시킴
    - checkEngineAndPlatform()
        - 완성된 최적의 트리에서 Node.js 버전, npm 버전 등을 검사 후 현재 환경과 비교하여 호환되는지 확인

### 2.4.1.5 reify

- buildIdealTree에서 생성한 이상적인 트리를 실제로 구현하는 역할 → 이상적인 트리를 node_modules에 설치하고 package-lock.json에 반영하는 과정을 의미함
- reify이 실행되는 단계
    - reify가 실행되는 위치를 확인 후 node_modules 디렉토리가 있는지 확인한 후 없으면 생성함
    - acual 트리와 ideal 트리를 각각 생성
    - 생성된 두 트리를 비교
    - 두 트리 간의 차이를 바탕으로 이상적인 트리와 현재 트리 사이에 필요한 변경 사항을 확인 후 필요 시 패키지를 node_modules에 단계별로 설치하거나 제거함
    - 모든 변경이 완료되면 이상적인 트리의 내용을 현재 트리에 복사하여 동기화함
    - 의존성에 대한 보안 취약점을 검사 후 발견된 취약점에 대한 정보를 제공함

### 2.4.1.6 audit

- npm 패키지의 취약점을 살펴보기 위해 사용하는 명령어로 @npmcli/arborist에서 audit 명령어를 수행하는 메서드를 제공함
- 이 메서드는 AuditReport라는 클래스에서 수행하는데, 클래스가 하는 작업은 아래와 같음
    - 취약점을 분석해야 하는 패키지를 목록을 가져옴
    - 패키지 목록을 가져온 후 취약점 분석을 위한 정보를 가져옴
    - 2번에서 불러온 정보를 바탕으로 취약점을 분석하고 보고함

### 2.4.2 패키지 설치를 위한 패키지, pacote

- npm 패키지의 다운로드와 관리를 수행하는 라이브러리
- @npmcli/arborist가 의존성 트리를 분석한 후 이 라이브러리를 통해 실제 패키지를 설치함

### 2.4.2.1 manifest

- 해당 패키지의 manifest 정보를 가져오는 메서드로, 패키지의 이름과 버전을 인자로 받아 해당 패키지의 정보를 가져옴
- 정보에는 name, version, dependencies, dist, engines 등의 정보를 포함하고 있음

### 2.4.2.2 tarball

- 패키지의 tarball 데이터를 메모리에 불러오는 작업을 수행

### 2.4.2.3 extract

- tarball을 불러오는 것을 넘어서 파일을 압축 해제 후 파일 시스템에 저장함

### 2.4.3 node_modules 살펴보기

### 2.4.3.1 평탄화된 node_modules

- npm은 의존성을 설치할 때 가능한 한 상위 node_modules 폴더에 패키지를 올려두는 방식을 사용 → 중복되는 하위 의존성을 각 패키지별로 중첩 설치하지 않고, 버전이 충돌하지 않는 한 루트에 한 번만 설치하여 트리를 평평하게 만듦
- 장점
    - 중복 제거
        - 디스크 용량 절약
    - 로드 속도 개선
        - require()시 경로 탐색이 단순해짐
    - 윈도우 경로 제한 문제 완화
        - 깊은 중첩 디렉토리 구조 방지
- 단점
    - 버전 충돌
        - 서로 다른 버전이 필요한 경우 공통 버전은 루트에 두고 충돌 나는 버전만 해당 패키지의 node_modules 안에 별도로 설치함9부분저긍로 다시 중첩 구조가 생길 수 있음)
    - 호이스팅의 의존성 문제
        - 상위에 끌어올려진 패키지를 의도치 않게 다른 모듈에서 require할 수 있어 잠재적 버그를 만들기도 함

### 2.4.3.2 npm이 중복 설치를 피하는 방법

- 예시 - 같은 버전 범위(root에 한 번만 설치하여 중복회피)
    - `app`이 `foo@1`, `bar@2`를 쓰고, 둘 다 `lodash ^4.17.0`을 요구
        
        ```json
        // app/package.json
        {
          "dependencies": {
            "foo": "1.0.0",
            "bar": "2.0.0"
          }
        }
        // foo/package.json -> "lodash": "^4.17.0"
        // bar/package.json -> "lodash": "^4.17.0"
        ```
        
    - 설치 결과(개념도)
        
        ```
        node_modules/
          foo/
          bar/
          lodash/   <- 한 번만 설치됨
        ```
        
- 예시 - 버전 충돌(공통은 root, 충돌 버전만 하위에 별도 설치)
    - `foo`는 `lodash ^3.10.0`, `bar`는 `lodash ^4.17.0`을 요구
        
        ```jsx
        node_modules/
          bar/
          foo/
            node_modules/
              lodash@3.x  <- 충돌 버전만 foo 내부에 별도 설치
          lodash@4.x      <- 루트에는 최신/호환 가능한 버전 1개
        ```
        
    - 가능한 건 root에 공유, 충돌 나는 것만 필요한 패키지의 하위에 중첩 설치
