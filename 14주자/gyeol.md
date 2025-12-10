# 7장 - 직접 자바스크립트 패키지 만들기

## 7.1 나만의 npm 패키지 만들기

## 7.1.5 간단한 테스트 코드 작성

- 단위 테스크 도구로는 자바스크립트 생태계에서 Jest를 사용하는 것이 일반적
- 예제에서는 비트에서 만든 Vitest 사용
- Vitest
    - 비트 설정과 플러그인을 그대로 이어받아 테스트 수행 가능
    - Jest와 호환되는 API를 제공하므로 사용하기 편리
    - 타입스크립트와 함께 사용하기 위해 별도의 설정이 필요한 Jest의 ts-jest와 달리 타입스크립트와 ESModule을 지원하는 것이 특징
    - Jest 보다 가볍고 빠르게 동작하며, 구성 파일도 간단하게 작성할 수 있어 편리함
- Vitest 설치
    
    ```jsx
    pnpm add vitest -D
    ```
    
- 패키지의 유틸을 테스트할 수 있는 테스트 코드 작성
    
    ```jsx
    // src/utils/index.test.ts
    import {describe, test, expect} from 'vitest'
    
    import {getFilter} from './index'
    
    describe('getFilter function', () => {
        test('should return default values when no parameters are provided', () => {
            const result = getFilter()
            expect(result).toEqual({
                filter: 'grayscale(0%) sepia(0%) brightness(100%) contrast(100%) blur(0px)',
            })
        })
    
        test('should apply grayscale correctly', () => {
            const result = getFilter({grayscale: 50})
            expect(result.filter).toContain('grayscale(50%)')
        })
    
        test('should apply sepia correctly', () => {
            const result = getFilter({sepia: 75})
            expect(result.filter).toContain('sepia(75%)')
        })
    
        test('should apply brightness correctly', () => {
            const result = getFilter({brightness: 150})
            expect(result.filter).toContain('brightness(150%)')
        })
    
        test('should apply contrast correctly', () => {
            const result = getFilter({contrast: 200})
            expect(result.filter).toContain('contrast(200%)')
        })
    
        test('should apply blur correctly', () => {
            const result = getFilter({blur: 5})
            expect(result.filter).toContain('blur(5px)')
        })
    
        test('should apply multiple filters correctly', () => {
            const result = getFilter({
                grayscale: 30,
                sepia: 20,
                brightness: 110,
                contrast: 90,
                blur: 2,
            })
            expect(result).toEqual({
                filter: 'grayscale(30%) sepia(20%) brightness(110%) contrast(90%) blur(2px)',
            })
        })
    
        test('should maintain order of filters', () => {
            const result = getFilter({
                blur: 3,
                grayscale: 40,
                contrast: 80,
                sepia: 10,
                brightness: 120,
            })
            expect(result.filter).toBe('grayscale(40%) sepia(10%) brightness(120%) contrast(80%) blur(3px)')
        })
    })
    ```
    
- package.json의 scripts에 추가
    
    ```jsx
    // package.json
    "scripts"; {
    	// ...
    	"test": "vitest run"
    }
    ```
    
- pnpm run test 실행 결과
  
   <img width="653" height="371" alt="image" src="https://github.com/user-attachments/assets/af6ea04c-0faf-4429-9b8a-17bb24eac9bb" />
    

## 7.1.6 깃허브 액션을 활용한 CI 파이프라인 구축

- 실습 프로젝트에서의 CI는 아래와 같이 두 가지를 수행할 예정
    - 코드의 문법적 오류를 검사하는 lint 작업
    - 테스트 코드를 빌드하고 실행하는 test 작업

### 위 작업을 하나의 파일에서 모두 수행할 수 있도록 깃허브 파이프라인 구성하기

- .github/workflows/ci.yml 파일 작성
    
    ```yaml
    # .github/workflows/ci.yml
    name: CI
    
    on:
        push:
            branches:
                - main
        pull_request:
    ```
    
    - name
        - 해당 워크플로의 이름
        - 깃허브 액션 페이지에서 워크플로를 식별하는 역할
    - on
        - 해당 워크플로가 언제 실행될지를 나타냄
        - 예제에서는 push.branchs를 통해 main 브랜치에 푸시가 발생할 때 실행되도록 설정했음
        - pull_request를 통해 PR이 발생했을 때도 실행되도록 설정함
- **워크플로에서 무엇을 할지 정의하기-환경 설정 및 의존성 설치를 위해 파일 수정**
    
    ```yaml
    # .github/workflows/ci.yml
    # ...
    jobs:
        setup:
            runs-on: ubuntu-latest
            steps:
                - uses: actions/checkout@v4
                - uses: pnpm/action-setup@v4
                - name: Use Node.js
                  uses: actions/setup-node@v4
                  with:
                      node-version: '20'
                      cache: 'pnpm'
                - name: Install dependencies
                  run: pnpm install --frozen-lockfile
    ```
    
    - setup
        - 임의로 정한 이름
        - **CI 환경에서 프로젝트를 체크아웃하고, pnpm과 Node.js를 세팅한 뒤, 의존성을 설치하는 단계**로 구성됨
    - runs-on
        - 해당 job이 실행될 머신을 정의
        - 예제에서 사용한 ubuntu-latest를 일반적으로 많이 사용
    - steps
        - 이 job이 순차적으로 실행할 작업 목록
    - actions/checkout@v4
        - GitHub Actions 실행 환경에 **현재 리포지토리 코드를 가져오는 단계**
        - CI가 코드에 접근할 수 있도록 워크플로의 기본 시작점
    - pnpm/action-setup@v4
        - CI 환경에 **pnpm을 설치하고, 사용할 수 있는 환경**을 설정함
    - actions/setup-node@v4
        - 노드를 설치하고 사용할 수 있도록 환경을 구성함(예시에서는 20버전을 설치하라고 명시)
        - `cache: 'pnpm'` 옵션을 사용해 의존성 설치 시 캐시를 사용하도록 함
    - Install dependencies
        - lockfile(`pnpm-lock.yaml`)을 기준으로 **재현 가능한 설치** 수행
- **워크플로에서 무엇을 할지 정의하기-빌드 작업을 수행해 코드의 정합성을 확인하기**
    
    ```yaml
    # .github/workflows/ci.yml
    # ...
    jobs:
    	  setup:
    	    # ...
    	  build:
    	      needs: setup
    	      runs-on: ubuntu-latest
    	      steps:
    	          - uses: actions/checkout@v4
    	          - uses: pnpm/action-setup@v4
    	          - uses: actions/setup-node@v4
    	            with:
    	                node-version: '20'
    	                cache: 'pnpm'
    	
    	          - name: Install dependencies
    	            run: pnpm install --frozen-lockfile
    	          - name: Build
    	            run: pnpm run build
    	          - name: Test
    	            run: pnpm run test
        lint:
            needs: setup
            runs-on: ubuntu-latest
            steps:
                - uses: actions/checkout@v4
                - uses: pnpm/action-setup@v4
                - uses: actions/setup-node@v4
                  with:
                      node-version: '20'
                      cache: 'pnpm'
                - name: Install dependencies
                  run: pnpm install --frozen-lockfile
                - name: Lint
                  run: pnpm run lint
                - name: Prettier
                  run: pnpm run prettier
    ```
    
    - build와 lint 둘다 setup이 완료된 후 실행되도록 설정
    - build 작업에서는 pnpm run build를 실행해 빌드를 수행하고 pnpm run test를 실행해 테스트를 수행함
    - lint 작업에서는 lint와 prettier를 실행해 코드이 문법적 오류를 검사하고 코드의 일관성을 유지하도록 함
- ci.yaml 파일이 main에 머지되면 actions 에서 workflow가 실행되는 것을 확인할 수 있음


    <img width="712" height="439" alt="image" src="https://github.com/user-attachments/assets/75a7b61c-fae9-4740-810e-c7d9b67911a3" />

    https://github.com/hotdog1004/react-image/actions/runs/19957853669
    
    - 자꾸 에러나서 왜 그런가했더니..eslint, prettier 설치를 안 해서 그런 것…예시 package.json보면 따로 설치 안 했던데….naverpay eslint 패키지 버전 차이인가?(책에서는 1, 나는 2 사용)
    - eslint, prettier 최신 버전으로 설치하고 eslintrc, ignore 파일 삭제, eslint.config.js 추가해서 처리했음

## 7.1.7 changesets를 활용한 배포

- changesets
    - 코드가 변경됐을 때 안정적으로 배포하는데 필요한 모든 기능을 제공하는 도구
    - 코드 기여자가 변경된 코드를 푸시하면 해당 변경사항을 어떻게 릴리스해야 하는지 선언 후 제공된 정보를 기반으로 배포까지 자동화함

### 7.1.7.1 changeset-bot 설치

- https://github.com/apps/changeset-bot 에서 install → 해당 패키지 저장소에만 설치

### 7.1.7.2 npm 토큰 발급

- npm 토큰
    - 패키지 배포시 사용하는 인증토큰
    - 이 토큰을 통해  npm 레지스트리에 패키지 배포 가능함
- 토큰 생성

    <img width="611" height="746" alt="image" src="https://github.com/user-attachments/assets/79db8928-86e7-4274-871d-c05848d8b10a" />
    
    - 생성한 토큰은 잘 보관해야함

### 7.1.7.3 npm 토큰을 저장소에 저장

- 토큰은 봇이 배포를 위해 지속적으로 깃허브에서 사용해야 하므로 저장소 어딘가에 저장해둬야 함
- 이를 위해 깃허브의 Repository secrets 기능을 사용
- 깃허브 저장소 페이지 → Settings → Secrets and variables → Actions 에서 New repository secret을 클릭해 시크릿 추가
    

    <img width="810" height="521" alt="image" src="https://github.com/user-attachments/assets/f77205c8-ac5e-4a40-a88a-ef3d492c304d" />

- 토큰 생성 후 모습
    
    <img width="890" height="625" alt="image" src="https://github.com/user-attachments/assets/9f7aa39b-6074-4dcc-9834-c850bfb8ddd5" />


### 7.1.7.4 저장소 코드에 changesets 설정

- @changesets/cli 패키지 설치하기
    
    ```yaml
    pnpm add -D @changesets/cli
    ```
    
- changesets 사용하기 위해 초기화 명령어 실행
    
    ```yaml
    pnpm changeset init
    ```
    
- 초기화가 실행되면 .chnageset 폴더에 README.md와 config.json 파일이 생성됨
    - README.md
        - chnagesets와 관련된 안내 문서
    - config.json
        - chnagesets의 구성파일
- config.json
    
    ```yaml
    {
      "$schema": "https://unpkg.com/@changesets/config@3.1.2/schema.json",
      "changelog": "@changesets/cli/changelog",
      "commit": false,
      "fixed": [],
      "linked": [],
      "access": "restricted",
      "baseBranch": "main",
      "updateInternalDependencies": "patch",
      "ignore": []
    }
    ```
    
    - $schema
        - Changesets 설정의 스키마 파일
        - IDE에서 자동 완성, 타입 검사를 받기 위해 사용됨
    - changelog
        - 각 release 시 changelog를 생성할 때 사용할 함수 또는 패키지
        - `"@changesets/cli/changelog"` → 기본 changelog 생성기 사용
    - commit
        - Changeset 적용 시 자동으로 git commit 할지 여부
        - `false` → 자동 커밋 안 함 (직접 커밋해야 함)
    - fixed
        - “버전이 항상 함께 움직여야 하는 패키지들”을 묶는 배열
        - **모노레포에서 특정 패키지들을 lock-step 버전으로 묶을 때 사용**
    - linked
        - fixed와 비슷한 패턴이지만 좀 더 유연한 연동 버전 관리
        - 그룹 내 변경된 패키지들에 대해서만 그룹 내 가장 최신버전으로 업데이트함
        - 보통 잘 사용하지 않음
    - access
        - Publish 시 npm 접근 레벨 설정
        - `"restricted"` → private 패키지처럼 접근 제한됨 (기본값)
        - `"public"` → npm 공개 배포용
    - baseBranch
        - Changesets가 변경 비교 기준으로 삼는 기본 브랜치
        - `"main"` → main 기준으로 release PR 생성함
    - updateInternalDependencies
        - 모노레포 내부 패키지가 서로 의존할 때 버전 증가 방식을 지정
        - `"patch"` → 내부 의존성 버전은 최소한 패치 정도로 올리기
        - 예: 패키지 A 변경 시 A를 의존하는 B도 최소 patch version 증가
    - ignore
        - Changeset 생성에서 제외할 패키지 목록
- 깃허브에서 자동으로 배포하기 위한 워크 플로 구축하기
    - 기본브랜치에서 새로운 코드가 머지되면 changesets가 배포가 필요한 지 판단 후 배포를 수행하게 됨
    - ./github/workflows/publish.yml 파일 추가
        
        ```yaml
        name: Changesets
        on:
            push:
                branches:
                    - main
        
        concurrency: ${{ github.workflow }}-${{ github.ref }}
        
        jobs:
            release:
                runs-on: ubuntu-latest
                steps:
                    - uses: actions/checkout@v4
                    - uses: pnpm/action-setup@v4
                    - name: Use Node.js
                      uses: actions/setup-node@v4
                      with:
                          node-version: '20'
                          cache: 'pnpm'
        
                    - name: Install dependencies
                      run: pnpm install --frozen-lockfile
        
                    - name: build
                      run: pnpm run build
        
                    - name: Create and publish versions
                      uses: changesets/action@v1
                      with:
                          publish: pnpm release
                          commit: '🚀 update versions'
                          title: '📦 update versions'
                          version: pnpm changeset-version
                      env:
                          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
                          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        ```
        
        - 메인 브랜치에 푸시가 발생하면 위 publish job이 실행됨
        - `with`
            - 액션을 실행할 때 함께 사용할 변수를 의미
            - publish: pnpm release
                - Changesets가 “publish 단계”에 실행할 명령어
                - 이 명령어는 package.json의 scripts에 다음과 같이 추가되어있어야함
                    
                    ```yaml
                    "scripts": {
                      "release": "changeset publish"
                    }
                    ```
                    
        - **`commit: '🚀 update versions'`**
            - Changeset이 자동 버전 생성을 할 때 → 변경된 package.json, CHANGELOG.md 등을 자동 커밋함
        - **`title: '📦 update versions'`**
            - Changesets가 버전 올라갈 때 GitHub에 릴리즈 PR을 생성함.
        - **`version: pnpm changeset-version`**
            - “version 단계”에서 실행할 명령
            - Changesets가 새 버전을 계산해 package.json과 changelog를 업데이트할 때 `pnpm changeset-version` 명령을 실행하는데, 이 작업은 코드 스타일을 맞춰주지 않으므로 포매팅도 함께 수행하는 스크립트를 추가해야함
            - 이 명령어는 package.json의 scripts에 다음과 같이 추가되어있어야함
                
                ```yaml
                "scripts": {
                  "changeset-version": "changeset version && pnpm run md:fix"
                }
                ```
                
        - env
            - **`GITHUB_TOKEN`**
                - 릴리즈 PR 생성 / 자동 커밋 등 GitHub API 호출에 필요
                - GitHub Action이 제공하는 기본 토큰 사용
            - **`NPM_TOKEN`**
                
                : npm publish 를 수행하려면 필요
                
                - npm 계정 → 토큰 생성 → GitHub repository secrets에 저장 → 여기서 사용
                - 없으면 publish 실패함

### 7.1.7.5 changesets를 사용해 배포하기

- 먼저 깃허브 액션이 코드를 수정하고 PR을 열 수 있는 권한을 부여해야 함
- 깃허브 저장소 페이지 → Settings → Actions → General → **Actions permissions → Workflow permissions 를 아래와 같이 수정**
    
    <img width="910" height="360" alt="image" src="https://github.com/user-attachments/assets/41b328f2-27f9-4328-8358-fa87c4577606" />

    - 읽기와 쓰기 권한을 모두 부여하고 액션이 PR을 열 수 있도록 체크
- chnageset-bot 테스트를 위해 다른 브랜치 생성 후 변경사항으로 PR 생성하면 아래와 같이 댓글이 추가됨
    
    <img width="948" height="618" alt="image" src="https://github.com/user-attachments/assets/99801c45-05e0-4d5b-8ab4-2b2d7dd21385" />

    
    - changeset-bot이 생성한 코멘트로, 해당 PR에 패키지 변경사항이 감지됐는데 이 변경사항을 새로운 버전에 포함시킬지 묻는 내용임
        - 변경사항이 새로운 버전에 포함돼야할 경우 두번째 줄을 클릭
        - 그게 아니라면 첫번째 줄을 클릭하면 됨
- 두번째 줄 클릭 → 마크다운 파일 작성 후 커밋, 푸시를 진행하면 changeset-bot이 작성한 댓글 본문이 Changeset detected로 변경됨
    
    <img width="993" height="796" alt="image" src="https://github.com/user-attachments/assets/e655ec4e-a2d0-40df-9da3-48a679fe8b3a" />

- PR을 머지하면 changesets가 배포를 위한 PR을 다시 생성한 것을 확인할 수 있음
    - 이 PR에는 기존 변경사항에 대한 내용이 본문에 추가되며, 이 PR이 병합될 경우 변경되는 패키지의 버전을 미리 보여줌
- chnagesets가 생성한 PR을 머지하면 깃허브 액션에서 npm 배포를 위한 작업을 수행함
- 배포가 정상적으로 완료되면 CHANGELOG, 깃허브 릴리스, npm 레지스트리에 배포된 것을 확인할 수 있음

## 7.1.8 정리

- 패키지 과정에서 기억해야 할 핵심 사항
    - 지원 범위 설정
        - 패키지의 지원 환경을 명확히 정의할 것
        - 환경에 맞게 트랜스파일하거나 폴리필을 추가할 수 있는 도구를 선택해야 함
        - 비트나 롤업 같은 번들러가 이러한 과정을 원활이 지원하는지도 확인할 것
    - 빌드 결과물 점검
        - 패키지 배포 전 빌드된 결과물을 반드시 검토, 테스트할 것
        - 패키지 코드는 다양한 런타임, 모듈시스템, 의존성 환경에서 동작해야 하므로 모든 변수를 고려해 확인해야 함
    - 의존성 관리
        - depnendencies와 peerDependencies 설정에는 각별히 주의할 것
        - 의존성을 추가하는 것은 신중하게 결정하고 필요한 최소한으로 유지하는 것이 좋음
    - 유의적 버전 관리
        - changesets과 같은 도구는 유의적 버전 관리를 돕지만 책임은 온전히 개발자의 몫임
        - 패키지 배포 전 항상 버전 번호가 변경된 이유와 방식이 유의적 버전 규칙에 부합하는지 검토해야 함
    - 안정적인 배포 환경
        - 배포 작업은 깃허브 액션과 같은 통제된 환경에서 수행해야 함
        - 이를 통해 동일한 환경에서 일관되게 패키지를 배포할 수 있고, 배포 토큰과 같은 민감 정보를 안전하게 관리할 수 있음

# 7.2 나만의 CLI 패키지 만들기

## 7.2.1 제작할 CLI 패키지 구상

- 실습에서는 CLI를 통해 비밀번호를 생성하는 패키지를 만들 예정
- 패키지 체크리스트별 상세 내용
    - 아이디어 검증 및 기술적 타당성 검토
        - npx 명령어를 통해 사용자가 원하는 길이의 비밀번호를 생성하는 패키지를 만든다.
        - Node.js 기반으로 만들 것이며, 사용자가 원하는 길이의 비밀번호를 생성하는 기능을 제공할 예정
    - 라이센스 선택
        - MIT 라이센스 사용
    - 적당한 이름 고르기
        - @ndive/password-generator
    - 지원 환경
        - Node.js 18 버전 이상을 지원
    - 개발 환경 및 프로젝트 구조
        - 번들러 - 비트, 롤업
        - 트랜스파일 도구 - 바벨
        - 타입 체킹 - 타입스크립트
        - 단위 테스트 - Vitest
        - 모듈 시스템 - ESModule만 지원
    - 의존성 관리 계획
        - Node.js 런타임에서만 사용되는 패키지이므로 비교적 의존성 부담이 덜한 편
        - 아래 세 패키지가 의존성으로 포함될 예정
            - chalk - 터미널ㅇ서 분필처럼 글자에 색상을 입혀 출력해주는 패키지
            - core-js
                - Node.js 18 버전까지 안정적으로 지원하기 위해 폴리필을 삽입할 것이므로 폴리필 지원 목적으로 core-js 사용예정
            - meow
                - 사용자 친화적인 CLI 도구를 만들 수 있게 도와주는 패키지
    - CI 및 CD 설정
        - 이전과 동일하게 설정

## 7.2.2 프로젝트 환경 설정

### 7.2.2.1 .browserslistrc

- CLI 패키지는 브라우저에서 실행되는 패키지와 달리 Node.js 환경만 고려하면 되므로 .browserslistrc도 Node.js 버전만 지정하면 됨
- Node.js 버전은 18 버전 이상을 다루기 위해 아래와 같이 작성
    
    ```yaml
    # .browserslistrc
    node >= 18
    ```
    

### 7.2.2.2 package.json

- package.json 파일은 다음과 같은 점을 고려하여 작성할 것
    - ESModule 모듈 시스템만 지원하기 위해 type 필드는 module로 설정
    - .browserslistrc로 Node.js 18 버전 이상만 지원하기로 했으므로, engines 필드를 통해 Node.js 18 버전 이상을 지정할 것 → Node.js 18 버전 이상이 아닌 환경에서 패키지를 설치하면 경고 메세지가 나타나게 됨
- 참고
    
    https://github.com/yceffort/ndive-password-generator/blob/main/package.json
    

### 7.2.2.3 tsconfig.json

- 이전 절과 유사하지만, 환경이 Node.js 이므로 다음과 같은 차이가 있
    
    ```yaml
    {
        "$schema": "http://json.schemastore.org/tsconfig",
        "compilerOptions": {
            "types": ["node"],
            "target": "ES2022",
            "module": "ESNext",
            "moduleResolution": "Bundler",
            "outDir": "./dist",
            "rootDir": "./src",
            "strict": true
        },
        "include": ["src/**/*"]
    }
    ```
    
    - target
        - 특별히 최신 JS 기능을 사용하지 않을 것이므로 .browserslistrc에 지정한 Node.js 버전과 맞춰 target을 작성하면 불필요하게 트랜스파일되는 과정을 줄일 수 있음
        - Node.js 버전이 ECMAScript의 어느 버전까지 지원하는지 알고싶다면 아래 페이지에서 확인할 수 있음
            
            [Node.js ES2015/ES6, ES2016 and ES2017 support](https://node.green/#ES2022)
            
        - 위 자료에 따르면 Node.js 18 버전 이상에서는 ES2022까지 무난하게 지원하므로 target도 2022로 설정하면 됨
    - types
        - Node.js 환경에서만 사용될 패키지이므로 node 타입만 추가하면 됨

### 7.2.2.4 vite.config.ts

- 빌드하는 데 사용할 vite.config.ts 파일 작성하기
- 앞서 package.json에서 type 필드로 ESModule 프로젝트로 설정해뒀으므로, vite.config.mts가 아닌 vite.config.ts로 작성함
- 작성 예시
    
    ```tsx
    import {babel} from '@rollup/plugin-babel'
    import browserslistToEsbuild from 'browserslist-to-esbuild'
    import {defineConfig} from 'vite'
    
    import pkg from './package.json'
    
    const SUPPORT_TARGETS = browserslistToEsbuild()
    
    export default defineConfig({
        plugins: [
            babel({
                babelHelpers: 'bundled',
                presets: [
                    [
                        '@babel/preset-env',
                        {
                            useBuiltIns: 'usage',
                            corejs: {version: '3.39.0', proposals: true},
                            debug: true,
                        },
                    ],
                ],
                extensions: ['.js', '.jsx', '.ts', '.tsx'],
                exclude: 'node_modules/**',
            }),
        ],
        build: {
            outDir: 'dist',
            lib: {
                entry: {
                    index: './src/index.ts',
                },
                formats: ['es'],
            },
            rollupOptions: {
                external: [...Object.keys(pkg.dependencies)].flatMap((dep) => [dep, new RegExp(`^${dep}/.*`)]),
            },
            target: SUPPORT_TARGETS,
        },
    })
    ```
    
- 이전 실습에서 설정했던 vite.config.mts와의 차이
    - 리액트 플러그인 제거
        - 리액트 기반 프로젝트가 아니므로 관련 플러그인을 모두 제거함
    - 폴리필 방식 변경
        - 이전 절에서는 브라우저 환경에서 사용되는 @ndive/react-image 패키지 특성으로 인해 런타임에 폴리필을 @babel/runtime-corejs3으로부터 가져오고, 해당 패키지를 사용하는 환경의 전역 오염을 방지하기 위해 @babel/plugin-transform-runtime을 사용했음
        - 하지만 이번 예제에서는 @babel/preset-env 플러그인을 사용하는 방식으로 변경했으며, 이유는 아래와 같음
            - CLI 환경의 특성
                - CLI를 실행하는 것 자체가 독립적인 프로세스로 실행되므로 전역 오염이 발생할 가능성이 없음
            - 실행 환경의 제한
                - 이 CLI는 Node.js라는 특정 런타임 환경에서만 실행되므로 다양한 불특정 기기를 사용하는 브라우저 환경과 달리 일반 사용자의 환경이나 성능을 고려할 필요가 상대적으로 적음
        - 추가로, @rollup/plugin-babel의 bundled 옵션을 통해 웹 애플리케이션 환경과 유사하게 필요한 바벨 헬퍼를 모두 번들링해서 사용함. → 이 접근 방식은 CLI 도구의 특성에 더 적합+불필요한 복잡성을 줄이면서도 필요한 기능을 효율적으로 제공함
    - build.lib.entry
        - package.json의 exports를 사용해 여러 subpath를 지원했던 이전 예제와 달리, 이번 예제는 CLI로 사용되는 패키지이므로 고려해야할 진입점은 index 뿐임 → build.lib.entry를 통해 index 진입점 및 해당 파일 경로를 지정
    - build.lib.formats
        - 이번 예제에서는 ESModule만 지원하므로 듀얼패키지를 위한 cjs, es 포맷이나 별도 디렉터리 설정을 할 필요가 없음 → build.lib.formats을 통해 es 포맷만 지정

## 7.2.3 실제 코드 작성

### 7.2.3.1 src/generator.ts

- 비밀번호를 생성하는 로직을 작성
- 이 파일은 사용자가 입력한 길이 만큼의 랜덤 문자열을 생성하는 역할
    
    ```tsx
    export const charset = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()_+-=[]{}|;:,.<>?'
    
    export const MIN_PASSWORD_LENGTH = 8
    
    export function generatePassword(length: number): string {
        if (length < MIN_PASSWORD_LENGTH) {
            throw new Error(`Password length must be at least ${MIN_PASSWORD_LENGTH}`)
        }
    
        let password = ''
        for (let i = 0; i < length; i++) {
            password += charset[Math.floor(Math.random() * charset.length)]
        }
        return password
    }
    ```
    
- generatePassword
    - 사용자가 입력한 길이만큼 랜덤 문자열을 생성하는 함수
    - 사용자가 입력한 길이가 최소 비밀번호 길이보다 작을 경우 에러를 발생

### 7.2.3.2 src/generator.test.ts

- 비밀번호 생성 함수를 테스트하기 위한 로직 작성
- Vitest를 사용해 비밀번호 생성 함수가 정상 동작하는 지 테스트
    
    ```tsx
    import {describe, test, expect} from 'vitest'
    
    import {generatePassword, charset, MIN_PASSWORD_LENGTH} from './generator.js'
    
    describe('generatePassword', () => {
        test('should generate a password of the specified length', () => {
            const length = 12
            const password = generatePassword(length)
            expect(password.length).toBe(length)
        })
    
        test('should generate passwords using only allowed characters', () => {
            const password = generatePassword(1000)
            const isOnlyAllowedChars = password.split('').every((char) => charset.includes(char))
            expect(isOnlyAllowedChars).toBe(true)
        })
    
        test('should generate different passwords on subsequent calls', () => {
            const password1 = generatePassword(1000)
            const password2 = generatePassword(1000)
            expect(password1).not.toBe(password2)
        })
    
        test(`should throw an error if length is less than ${MIN_PASSWORD_LENGTH}`, () => {
            expect(() => generatePassword(0)).toThrow()
        })
    })
    ```
    

### 7.2.3.3 src/cli.ts

- CLI 주요 로직을 담당
- 사용자의 입력을 처리하고 옵션을 파싱하며, 생성된 비밀번호를 출력
- meow 라이브러리를 통해 CLI 인터페이스 구현하고 chark를 사용해 콘솔 출력에 색상을 입힘
    
    ```tsx
    import chalk from 'chalk'
    import meow from 'meow'
    
    import {generatePassword, MIN_PASSWORD_LENGTH} from './generator'
    
    const DEFAULT_LENGTH = 12
    
    export function run(): void {
        const cli = meow(
            `
        Usage
          $ password-generator [length]
    
        Options
          --length, -l  Length of the password (default: 12)
    
        Examples
          $ password-generator
          $ password-generator 16
          $ password-generator -l 20
      `,
            {
                importMeta: import.meta,
                flags: {
                    length: {
                        type: 'number',
                        shortFlag: 'l',
                        default: 12,
                    },
                },
            },
        )
    
        /**
         * @description password-generator 10 과 password-generator -l 10 은 같은 결과를 반환한다. 더 편리한 사용을 위해 전자를 지원하며, 두개 모두가 들어올 경우 전자가 우선순위를 갖는다.
         */
        const length = cli.input[0] ? parseInt(cli.input[0], 10) : cli.flags.length || DEFAULT_LENGTH
    
        // const length = cli.input[0] ? parseInt(cli.input[0], 10) : DEFAULT_LENGTH
    
        if (isNaN(length) || length < MIN_PASSWORD_LENGTH) {
            // eslint-disable-next-line no-console
            console.error(chalk.red(`Error: Password length must be at least ${MIN_PASSWORD_LENGTH} characters.`))
            return process.exit(1)
        }
    
        const password = generatePassword(length)
    
        // eslint-disable-next-line no-console
        console.log(chalk.green(`Generated password (${length} characters)`))
        // eslint-disable-next-line no-console
        console.log(chalk.blue(password))
    }
    ```
    
    - 이 파일은 run이라는 함수를 내보내고 해당 함수가 실제 CLI가 실행되는 로직을 담당하도록 만드는 것이 목적

### 7.2.3.4 src/index.ts

```tsx
#!/usr/bin/env node
import {run} from './cli'

run()
```

- index.ts는 CLI의 진입점을 나타내며 run 함수를 호출해 CLI를 실행함
- `#!/usr/bin/env node`
    - Node.js 환경에서 실행되는 스크립트임을 나타내는 셔뱅
    - 이 셔뱅이 있어야 해당 프로그램을 node로 실행해야한다는 것을 알릴 수 있고, 이를 통해 npx로 해당 패키지를 실행할 수 있게됨
    - 셔뱅 : 스크립트 파일의 첫 줄에서 어떤 인터프리터로 실행할지를 지정하는 특별한 선언
- index.ts 파일을 별도로 분리한 이유
    - 파일 관심사 분리
        - 실제 CLI 실행과 관련된 로직은 cli.ts에서, 셔뱅과 같은 실행환경 설정은 index.ts에 분리함으로써 각 파일이 담당하는 역할을 명확히 할 수 있음
        - 패키지를 처음 보는 사람이라도 index.ts를 통해 Node.js 기반 CLI 패키지라는 사실을 쉽게 파악할 수 있음
    - 모듈성 증대
        - cli.ts파일은 셔뱅이 없는 순수한 TS 모듈로 유지할 수 있으므로 다른 컨텍스트에서도 쉽게 import하고 사용 가능
    - 유연성 증가
        - 필요에 따라 다른 실행환경이나 설정을 쉽게 적용 할 수 있음
        - CLI가 실행되는 Node.js 버전을 강제하고싶다면, index.ts에 관련 로직을 추가하면됨
- 이와 같은 이유로, 일반적인 Node.js 기반 CLI 패키지는 index.ts에 셔뱅을 선언하고 CLI 실행로직을 분리하는 방식을 채택하고 있음

## 7.2.4 결과물 확인

### 7.2.4.1 ./dist/index.js 살펴보기

- package.json의 script에 추가된 build 명령어를 사용하면 vite를 사용해 코드를 번들링 할 수 있음
    
    ```tsx
    pnpm run build
    ```
    
- build 후 파일을 확인하면, cli.ts, index.ts, generator.ts로 작성한 코드가 하나의 파일로 번들링되어 dist/index.js 파일로 생성된 것을 확인할 수 있음

### 7.2.4.2 폴리필이 삽입된 이유 분석하기

- 번들링된 코드를 보면, 맨 첫줄에 폴리필이 삽입된 것을 확인할 수 있음
    
    ```tsx
    // dist/index.js
    import 'core-js/modules/es.regexp.flags.js'
    ```
    
- es.regexp.flags는 정규표현식에 활성화된 플래그를 문자열로 반환하는 역할을 하는데, 이 속성은 ES2015에 추가되어 node ≥ 18을 지원함 → 예시로 만든 패키지 역시 ES2022를 target으로 지정했기 때문에 불필요한 폴리필임
- 그럼에도 불구하고 core-js가 해당 폴리필을 삽입한 이유는 아래와 같음
    - 자바스크립트는 정적 타입 언어가 아닌 동적 타입 언어이므로 바벨이 폴리필을 삽입할 때 시도하는 정적 분석만으로는 .flags 속성을 사용하는 변수가 RegExp인지 무엇인지 알 수 없음. 그러므로 바벨은 .flags의 존재를 보면 해당 코드가 실제로  RegExp.prototype.flags인지 여부와 상관없이 보수적으로 폴리필을 삽입함
    - RegExp.prototype.flags 자체는 폴리필이 필요없지만, ECMA2024부터 추가된 정규표현식 연산자 Regexp V Flag의 폴리필을 삽입하기 위해서는 transform-unicode-sets-regex 플러그인이 필요함 → 해당 플러그인의 안정적인 지원을 위해 es.regexp.flags를 폴리필로 삽입하는 것을 짐작할 수 있음
- 바벨과 core-js를 사용한다고 해서 완벽하게 폴리필을 제공하는 것이 아니므로, 폴리필이 삽입되는 이유를 이해하고 가능하다면 지원환경을 좁혀 core-js와 바벨의 필요성 자체를 없애는 편이 더 좋을 수 있음

### 7.2.4.3 직접 사용해보기

- 빌드된 CLI를 직접 사용하기 위해 아래 명령어를 ./dist/index.js를 터미널에서 실행
    - 셔뱅이 있으므로 node ./dist/index.js가 아닌 ./dist/index.js로 실행할 수 있음
- 실행 결과
    
    ```tsx
    ./dist/index.js
    zsh: permission denied: ./dist/index.js
    ```
    
- 해당 파일에 대한 실행 권한이 없으므로 빌드 이후 자동으로 권한을 부여할 수 있도록 아래와 같이 적용
    
    ```tsx
    // package.json
    {
    	"scripts": {
    		"build": "vite build --config vite.config.ts",
    		"postbuild": "chmod +x dist/index.js",
    	}
    }
    ```
    

### 로컬에서 배포하지 않은 CLI 패키지 테스트하기

- npm link를 통해 패키지를 전역에 설치하면 어디서든 사용할 수 있음 → 이를 통해 로컬에서도 배포전 CLI를 테스트할 수 있음
- npm link는 패키지를 전역 node_modules에 등록하기 때문에 @ndive와 같은 스코프가 생략되므로, **스코프가 있는 패키지를 등록하는경우 스코프를 생략하고 실행**해야함

## 7.2.6 CLI를 만드는데 유용한 패키지

### 7.2.6.1 meow

- Node.js 환경에서 간단하고 경량화된 CLI 애플리케이션을 빠르게 구축하기 위한 패키지
- 특징
    - 간단한 설정
        - meow는 최소한의 코드로 CLI 애플리케이션 설정이 가능
    - 자동 도움말 생성
        - 사용자가 제공한 설명을 바탕으로 기본적인 도움말 메시지를 자동 생성하며 -help 플래그로 접근 가능
    - 유연한 인자 파싱
        - 명령줄 인자와 플래그를 쉽게 파싱 가능

### 7.2.6.2 Commnder.js

- Node.js 환경에서 복잡한 CLI를 구축하기 위한 강력한 라이브러리
- 특징
    - 명령어 및 하위 명령어 지원
        - 하나의 CLI 안에서 `init`, `build`, `deploy` 같은 명령어와 그보다 더 깊은 하위 명령어 구조를 쉽게 만들 수 있
    - 자동 도움말 생성
        - 옵션, 설명, 사용 예시 등을 기반으로 `--help` 호출 시 자동으로 보기 좋은 도움말 문서를 생성
    - 옵션과 인자의 유연한 파싱
        - `--flag`, `-f`, `--port 3000`, `--port=3000` 등 다양한 형태의 옵션 표현을 안정적으로 파싱
    - 타입 검증 및 강제 변환
        - 옵션 값을 숫자, boolean 등으로 자동 변환하거나 커스텀 파서로 원하는 타입으로 강제 변환 가능

### 7.2.6.3 Inquirer.js

- 사용자의 다양한 입력을 처리하기 위해 사용할 수 있는 라이브러리
- 특징
    - 다양한 프롬프트 타입 지원
        - 입력, 확인, 목록 선택, 체크박스 등 다양한 유형의 사용자 입력 처리 가능
    - 입력 검증 및 변환
        - 사용자 입력에 대한 유효성 검사와 데이터 변환 기능 제공
    - 비동기 처리
        - Promise 기반의 API를 통해 비동기 작업 쉽게 처리 가능
    - @inquirer/core 기능을 통한 세밀한 제어
        - 핵심 기능을 별도 라이브러리 형태로 제공하여 이를 통해 세밀한 제어가 가능

### 7.2.6.4 chalk

- Node.js 환경에서 터미널 출력에 스타일과 색상을 적용하기 위해 만들어진 패키지
- 특징
    - 다양한 색상과 스타일
        - 텍스트 색상, 배경색, 굵게, 기울임꼴, 밑줄 등 다양한 스타일링 옵션 제공
    - 중첩 및 조합 가능한 스타일
        - 여러 스타일을 쉽게 조합하고 중첩 가능
    - 자동 색상 지원 감지
        - 터미널의 색상 지원 여부를 자동으로 감지해 적절히 대응
    - 256색상 및 트루 컬러 지원
        - 지원되는 터미널에서 더 풍부한 색상 표현 가능

### 7.2.6.5 ora

- Node.js 환경에서 우아한 터미널 스피너를 제공하는 패키지
- 장시간 실행되는 작업의 진행상황을 시각적으로 표시해 사용자에게 더 나은 경험을 제공

## 7.2.7 정리

- CLI 패키지 만들 때 고려해야할 사항
    - Node.js 셔뱅(#!/usr/bin/env node)을 실행 파일의 첫 줄에 포함할 것
    - 실행파일에 적절한 실행권한을 부여할 것
    - 사용자 친화적인 CLI 인터페이스를 구현할 것
- CLI 패키지 예시
    - create-react-app 같은 프로젝트 보일러 플레이트 생성기
    - ESLint나 Prettier 같은 코드 스타일 및 품질 관리 도구
    - 개발 프로세스 자동화 도구
    - 팀 생산성 향상을 위한 커스텀 유틸리티 도구
