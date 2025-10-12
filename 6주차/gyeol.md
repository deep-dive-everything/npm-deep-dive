# 3.3 pnpm: 디스크 공간 절약과 설치 속도의 혁신을 가져온 패키지 관리자

## 3.3.1 pnpm의 소개와 역사

- yarn의 한계에서 영감을 받아 2017년에 등장함
- pnpm의 메인 개발자인 졸탄 코찬이 지적한 npm과 yarn classic의 문제점
    - 여전히 평평한 의존성 트리를 사용(현재 PnP 모드를 기본으로 제공하는 yarn berry는 이 문제를 대부분 해결했음)
        - 프로젝트가 직접 의존하지 않는 패키지에도 접근이 가능함
        - 평평한 의존성 트리를 만드는 과정이 복잡하고 느림
        - 일부 패키지는 고유의 node_modules 디렉터리를 필요로 하지만, 평평한 트리 구조에서는 이를 관리하기 어려움

## 3.3.3 특징

### 3.3.2.1 pnpm-lock.yml

- yarn.lock과 마찬가지로 YAML 파일 포맷을 사용함
- 패키지의 정합성을 보장하기 위해 SHA-512 기반의 해시 키를 포함
- 해당 패키지의 의존성을 나타내는 dependencies 필드를 통해 각 패키지가 어떤 의존성을 가지고 있는지 확인 가능
- dependencies 필드가 락 파일 최상단에 위치 → 현재 설치된 패키지와 버전을 한눈에 파악 가능
- js-tokens 버전 처리 방식의 차이
    - react의 의존성인 loose-envify는 dependencies로 “js-tokens”: “^3.0.0 || ^4.0.0”을 가지는데,
        - yarn은 이 정보를 버전 범위로 관리
        - **pnpm은 이를 최상위 허용 버전인 4.0.0으로 설정해서 관리**

### 3.2.2.2 글로벌 스토어의 하드 링크

- pnpm은 각 패키지가 설치하는 하위 의존성을 node_modules가 아닌 기본적으로 **~/ .pnpm이라고 하는 글로벌 스토어에 저장**
- 필요한 의존성들을 하드 링크라는 방식으로 연결해서 사용함
- 하드 링크
    - 파일 시스템의 기능 중 하나로, 동일한 데이터를 가리키는 여러 개의 파일 경로를 만드는 방식
    - 각 경로는 같은 파일을 참조하기 때문에 어느 경로로 접근하든 동일한 파일에 도달할 수 있음
- 하드 링크 방식의 장점
    - 디스크 공간의 절약
        - 여러 패키지에서 동일한 패키지를 참조할 때 여러 복사본을 만드는 대신 .pnpm에 있는 단일 패키지를 참조하게 되어 디스크 공간 절약 가능
    - 성능 향상
        - 파일을 복사하거나 상위 폴더를 계속 뒤져가면서 node_modules를 찾는 대신 하드 링크를 사용해 빠르게 원하는 패키지를 찾을 수 있음
    - 일관성 유지와 안정성
        - 하드 링크로 동일한 파일을 여러 곳에서 참조하므로 패키지 버전과 내용이 일관되게 유지되어 여러 곳에서 참조하더라도 동일성을 보장할 수 있음
- **하드 링크와 심볼릭 링크(소프트 링크)**
    - 공통점
        - 원본 파일이 수정되면 이를 참조하는 모든 링크에서도 변경 사항을 볼 수 있음
    - 차이점
        - 하드 링크는 inode를 추가로 가리키는 것 → 원본이 삭제되더라도 같은 inode를 가리키는 다른 하드 링크가 있으면 파일이 살아있음(단, 같은 파일 시스템 안에서만 쓸 수 있고 원본과 속성도 똑같이 공유)
        - 심볼릭 링크는 윈도우의 바로가기처럼 작동함 → 원본이 삭제되면 심볼릭 링크는 사용할 수 없음(다른 파일 시스템에서도 만들 수 있고, 원본 파일과는 다른 독립적인 속성을 가질 수 있음)
    - inode
        - 파일이 생성될 때마다 inode라는 고유번호가 부여되는데, inode에는 해당 파일의 정보가 담겨있음

### 3.2.2.3 평탄화되지 않은 node_modules

- pnpm은 평탄화된 node_modules의 구조를 피하기 위해 아래와 같은 전략을 채택함
    - 프로젝트의 ./node_modules에는 선언된 dependencies와 같은 직접 의존성만 위치시킨다.
    - 이 패키지들의 실제 파일을 .pnpm에 하드 링크로 연결한다.
    - 프로젝트의 직접 의존성에 필요한 패키지의 실제 원본은 ./node_modules 대신 ./node_modules/ .pnpm에 평탄화해서 설치해둔다.
    - 각 패키지의 의존성은 앞서 평탄화해둔 .pnpm에 하드 링크로 참조하게 한다.
- 프로젝트 최상위에 react@18.3.1을 설치했을 때 생성되는 node_modules의 구조를 도식화한 그림
    
    <img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/709a26f7-f1e4-46b0-9692-25a60376f5ae" />

    

### 3.3.2.4 콘텐츠 어드레서블 스토리지

- 파일이나 데이터 블록을 고유한 해시값으로 식별해서 저장하는 시스템
- 해시값은 파일 내용을 기반으로 생성되므로 동일한 파일은 동일한 해시값을 갖게됨
- 전통적은 npm 방식에서는 프로젝트 수만큼 동일한 리액트 패키지가 중복설치되는데, pnpm은 이를 해결하기 위해 의존성을 콘텐츠 어드레서블 스토리지에 저장함
- node_modules가 콘텐츠 어드레서블 스토리지에 있는 파일을 참조하기 위해 Copy-on-Write(COW)라는 기술을 사용함(기본적으로는 COW를 사용하지만 COW를 지원하지 않는 환경의 경우 하드링크를 사용함)
    - COW : 데이터 복사를 최적화하는 기술. 데이터의 복사본을 생성할 때 원본 데이터를 그대로 참조하고 실제 복사를 수행하지 않다가 원본 데이터가 변경될 때 새로운 복사본을 생성하는 방식

### 3.3.2.5 심볼릭 링크로 구성된 node_modulesmdo

- react@18.3.1만 있는 프로젝트를 npm과 pnpm으로 각각 설치 후 비교(node_modules/ .pnpm/node_modules는 제거)
    - npm
        
        <img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b47d5c02-b7d6-4dc6-98c3-47857da6a1c1" />

        
    - pnpm
        
        <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/c69ba6c7-fd7e-4c79-9250-df87db075f5d" />

        
    - 차이점
        - npm은 폴더명이 단순히 패키지명이지만, pnpm은 버전까지 명시돼 있음
        - npm은 패키지 내용이 폴더 하위에 바로 위치하지만, pnpm은 node_modules 폴더 하위에 위치함
    - pnpm의 이러한 구조는 다음과 같은 장점이 있음
        - 의존성 격리
            - 각 패키지는 자체 node_modules를 가지고 있어 다른 의존성과 충돌하지 않음
        - 버전 관리의 용이성
            - 각 패키지 버전이 독립된 공간을 차지하므로 여러 버전을 관리하기가 용이함
        - 평탄화된 node_modules의 위험성 회피
            - 각 의존성은 실제로 자신이 참조하는 의존성에만 접근할 수 있으므로 평탄화된 node_modules로 인해 발생할 수 있는 문제를 회피할 수 있음

### 3.3.2.6 Plug n Play 지원

- pnpm에서 PnP 모드를 활성화하기 위해 .npmrc 파일에 아래 설정을 추가하면 됨
    
    ```jsx
    node-linker=pnp
    symlink=false
    ```
    
    - node-linker
        - node_modules에 패키지를 설치할 때 사용할 링커를 정의(옵션: isolated/hoisted/pnp)
    - symlink(boolean)
        - node-linker를 pnp로 설정할 경우 symlink: false를 지정해야 함
- pnpm이 생성한 PnP 모드의 결과물은 yarn의 PnP와 거의 유사함
    - 실제로 pnpm의 PnP 모드는 yarn에서 불러온 코드를 기반으로 생성하기 때문에 방식이 거의 동일함

# 3.4 npm, yarn, pnpm 비교

## 3.4.1 워크스페이스

### 3.4.1.1 워크스페이스란 무엇인가?

- 프로젝트의 규모가 커지고 복잡해지면서 하나의 프로젝트를 여러 개의 자바스크립트의 패키지로 분리하고, 패키지 간의 의존성을 효율적으로 관리할 필요성이 증가했음
- 워크스페이스 구조는 독립적인 패키지들이 하나의 프로젝트 내에서 원활하게 작동하고 관리될 수 있도록 지원함
- 대표적인 예로 바벨이 있음
    - 바벨은 최신 자바스크립트 기능을 다양한 브라우저에서 사용할 수 있도록 코드 변환을 지원하는 트랜스파일러 라이브러리
    - 이런 작업을 수행하기 위해 수십 가지의 플러그인을 필요로 하며, 각 플러그인과 핵심 라이브러리과 독립된 패키지로 존재함
- **모노레포 VS 워크스페이스**
    - 둘다 여러 프로젝트(패키지)를 하나의 저장소에서 관리하기 위한 개념
    - 모노레포는 단일 저장소에서 여러 패키지를 함께 관리하는 방식
    - 워크스페이스는 패키지 매니저가 제공하는 기능으로 여러 패키지를 효율적으로 설치하고 관리하는 도구
- 워크스페이스 기능은 npm, yarn, pnpm 모두 기본적으로 지원하고 있음

### 3.4.1.2 npm의 워크스페이스

- 비교적 최근인 7.x 버전에 도입
- 하나의 최상위 패키지에서 여러 하위 패키지를 효울적으로 관리하기 위해 설계됨
- npm 워크스페이스를 설정한 후 npm install을 실행하면 하위 패키지들이 링크로 연결되며 각 패키지의 node_modules 폴더에 필요한 패키지들이 심볼릭 링크로 참조됨 → 이를 통해 하위 패키지 간의 의존성도 자동으로 연결되어 패키지 관리를 더욱 효율적으로 할 수 있음
- npm 워크스페이스에서는 최상위 경로에 단 하나의 package-lock.json만 생성되며, node_modules도 최상위 경로에 단 하나만 존재(Node.js의 모듈 참조 방식으로 인해 이런 구조가 가능)
- npm에서 워크스페이스는 package.json의 workspaces 필드에 관리하고자 하는 폴더를 추가해서 설정할 수 있음
- 하위 패키지의 명령어를 실행할 때는 —workspaces 옵션을 추가하면 됨
- npm에 아직 릴리스되지 않은 하위 패키지들은 node_modules에 심볼릭 링크 형태로 연결되며, 이를 통해 npm 레지스트리에 올리지 않아도 사용이 가능함

### 3.4.1.3 yarn의 워크스페이스

- 패키지 관리자 생태계에서 워크스페이스 개념을 처음 도입했음(yarn classic 시절부터 워크스페이스 기능을 지원)
- npm과 마찬가지로 최상위 package.json파일에 workspaces 필드를 추가해서 설정 가능
- 각 워크스페이스에 있는 스크립트를 실행하기 위해 yarn workspaces foreach라는 명령어를 제공함
- **yarn workspaces foreach의 옵션**
    - —all
        - 프로젝트 내 모든 워크스페이스에서 제공된 명령어를 실행함
    - —parallel
        - 워크스페이스 작업을 병렬로 실행함
    - —since
        - 현재 브랜치를 main 브랜치와 비교해서 수정된 패키지를 대상으로만 명령어를 실행할 수 있음
    - -pt
        - —parallel과 —topological을 합친 것으로 명령어를 병렬로 실행하되 패키지 의존성 순서에 맞춰서 진행한다는 것을 의미함

### 3.4.1.4 pnpm의 워크스페이스

- 워크스페이스를 선언하기 위해 pnpm-workspace.yaml 파일을 프로젝트 최상위 디렉터리에 생성해야함
- yarn과 마찬가지로 패키지 버전을 선언할 때 workspace: 접두사를 사용하는 워크스페이스 전용 프로토콜을 지원함
- pnpm에서 워크스페이스 하위 명령어를 실행하려면 pnpm run -r을 사용하면 됨(r: recursive의 약자로, 워크스페이스 내 모든 프로젝트를 순회하며 명령어를 실행하도록 도움)
- pnpm은 별도의 명령어 없이도 패키지 의존성 관계에 따라 명령어를 실행함(의존성이 없으면 알파벳 순서로 실행)
- —filter 옵션을 통해 특정 워크스페이스에서만 명령어가 실행되도록 제한할 수 있음
- pnpm은 워크스페이스에 필요한 기본 기능을 간결하게 제공하지만, 버저닝과 같은 복잡한 기능은 changesets나 Rush 같은 서드파티 라이브러리를 사용하는 것을 권장 → npm 레지스트리에 패키지를 업로드하기 위해서는 버저닝이 매우 중요하므로 pnpm+서드파티 라이브러리 조합은 필수적임

## 3.4.2 명령어 비교

### 3.4.2.1 의존성 관리

- package.json에 있는 의존성 설치
    
    ```jsx
    // npm
    npm install
    // yarn
    yarn install
    yarn
    // pnpm
    pnpm install
    ```
    
- lock 파일 기준으로 package.json에 있는 의존성을 엄격하게 설치
    
    ```jsx
    // npm
    npm ci
    // yarn
    yarn install --immutable --immutable-cache
    // pnpm
    pnpm install --frozen-lockfile
    ```
    
- package.json의 버전 규칙에 맞춰 최신 버전으로 업데이트
    
    ```jsx
    // npm
    npm update
    // yarn
    yarn up -R '**'
    yarn-plugin-semver-up 플러그인 사용
    // pnpm
    pnpm update
    ```
    
- package.json의 버전 규칙에 상관없이 모두 최신 버전으로 업데이트
    
    ```jsx
    // npm
    N/A
    // yarn
    yarn up
    // pnpm
    pnpm update --latest
    ```
    
- package.json의 버전 규칙에 맞춰 특정 패키지만 업데이트
    
    ```jsx
    // npm
    npm update <패키지명>
    // yarn
    yarn-plugin-semver-up 플러그인 사용
    // pnpm
    pnpm update <패키지명>
    ```
    
- package.json의 버전 규칙에 상관없이 특정 패키지를 최신 버전으로 업데이트
    
    ```jsx
    // npm
    npm update <패키지명>@latest
    // yarn
    yarn up <패키지명>
    // pnpm
    pnpm update --latest <패키지명>
    ```
    
- 인터랙티브 의존성 업데이트(인터랙티브 의존성 업데이트는 CLI에서 직접 선택해서 업데이트하는 방식을 의미)
    
    ```jsx
    // npm
    N/A
    // yarn
    yarn upgrade-interactive
    // pnpm
    pnpm update --interactive
    ```
    
- package.json에 dependencies 의존성을 직접 추가
    
    ```jsx
    // npm
    npm install <패키지명>
    // yarn
    yarn add <패키지명>
    // pnpm
    pnpm add <패키지명>
    ```
    
- package.json에 devDependencies 의존성을 직접 추가
    
    ```jsx
    // npm
    npm install <패키지명> --save-dev
    // yarn
    yarn add <패키지명> --dev
    // pnpm
    pnpm add <패키지명> --save-dev
    ```
    
- package.json에서 의존성을 제거
    
    ```jsx
    // npm
    npm uninstall <패키지명>
    // yarn
    yarn remove <패키지명>
    // pnpm
    pnpm remove <패키지명>
    ```
    
- 중복되는 범위의 의존성을 제거
    
    ```jsx
    // npm
    npm dedupe
    // yarn
    yarn dedupe
    // pnpm
    pnpm dedupe
    ```
    
- 해당 패키지가 왜 필요한지, 어느 패키지에 의존되어 설치하는지 확인
    
    ```jsx
    // npm
    npm why <패키지명>
    // yarn
    yarn why <패키지명>
    // pnpm
    pnpm why <패키지명>
    ```
    

### 3.4.2.2 글로벌 패키지 실행

- 전역 패키지 설치
    
    ```jsx
    // npm
    npm install --global <패키지명>
    // yarn
    // yarn berry에는 전역 패키지 설치하는 개념이 없음
    // pnpm
    pnpm add --global <패키지명>
    ```
    
- 전역 패키지 업데이트
    
    ```jsx
    // npm
    npm update --global <패키지명>
    // yarn
    // 없음
    // pnpm
    pnpm update --global <패키지명>
    ```
    
- 전역 패키지 업데이트 삭제
    
    ```jsx
    // npm
    npm remove --global <패키지명>
    // yarn
    // 없음
    // pnpm
    pnpm remove --global <패키지명>
    ```
    
- 패키지에서 명령 실행
    
    ```jsx
    // npm
    npm exec <패키지명>
    npx <패키지명>
    // yarn
    yarn dlx <패키지명>
    // pnpm
    pnpm dlx <패키지명>
    ```
    

### 3.4.2.3 그 밖에 자주 쓰이는 명령어

- 프로젝트 초기 설정
    
    ```jsx
    // npm
    npm init
    // yarn
    yarn init
    // pnpm
    pnpm init
    ```
    
- 프로젝트 레지스트리에 업로드
    
    ```jsx
    // npm
    npm publish
    // yarn
    yarn npm publish
    // pnpm
    pnpm publish
    ```
    
- 라이선스 정보
    
    ```jsx
    // npm
    // 없음
    // yarn
    yarn-plugin-licenses 플러그인으로 확인 가능
    // pnpm
    pnpm licenses
    ```
    
- 패키지 관리자 자체 버전업
    
    ```jsx
    // npm
    npm install -g npm, nvm, pacakge.json에 packageManager 필드를 선언하는 corepack 사용
    // yarn
    pacakge.json에 packageManager 필드를 선언하는 corepack 사용
    // pnpm
    pnpm install -g npm, pacakge.json에 packageManager 필드를 선언하는 corepack 사용
    ```
    
- 보안 검사
    
    ```jsx
    // npm
    npm audit
    // yarn
    yarn npm audit
    // pnpm
    pnpm audit
    ```
    

## 3.4.3 벤치마크 테스트

### 3.4.3.1 Next.js 설치 비교

### 3.4.3.1.1 Full Cold

- 완전히 새로운 머신에서 프로젝트를 처음 클론하고 설치하는 과정
- 완전히 캐시가 없는 상태에서 설치하는 경우는 드물기 때문에 참고만 할 것
- 성능
    - pnpm > yarn > yarn node_modules = npm > yarn classic

### 3.4.3.1.2 캐시만 존재하는 경우

- 성능
    - yarn pnp가 pnpm보다 아주 약간 빠름
    - yarn pnp > pnpm > yarn node_modules = yarn classic > npm
- yarn pnp은 캐시만 있다면 pnp.cjs를 생성해서 경로만 지정해주면 되고, pnpm은 해당 글로벌 캐시를 COW 처리만 해주면 되기 때문에 가장 빠름

### 3.4.3.1.3 캐시와 락 파일 모두가 존재하는 경우

- Git 같은 버전 관리 시스템에서 저장소를 클론한 직후 node_modules 폴더가 없는 것과 동일. 일반적인 CI 환경에 해당됨
- 성능
    - pnpm > yarn pnp > yarn classic > yarn node_modules > npm

### 3.4.3.1.5 pnpm 벤치마크 테스트 결론(2024.11 기준)

- npm은 특정 상황을 제외하면 다른 패키지 관리자에 비해 현저히 느림
- yarn classic 도 대부분의 경우 pnpm, yarn berry에 비해 느림(yarn classic은 보안 패치 외 특별한 기능 개선이 이뤄지지 않고 있으므로 선택할 이유가 거의 없음)
- 대부분의 경우 yarn berry와 pnpm이 가장 빠른 성능을 보였음(yarn berry가 가장 뛰어난 성능을 나타냈음)
    - pnpm이 하드링크나 COW 작업이 필요한 반면, yarn berry는 필요한 의존성을 다운로드하면서 동시에 미리 준비된 pnp.cjs 파일에 현재 의존성 그래프를 JSON 형식으로 생성하기만 하면 되기 때문
- **모던 자바스크립트 프로젝트를 만드는 경우 yarn berry나 pnpm을 선택하는 것이 가장 합리적임!**
