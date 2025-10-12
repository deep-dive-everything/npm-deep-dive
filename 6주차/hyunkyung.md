## 3.3 pnpm: 디스크 공간 절약과 설치 속도의 혁신을 가져온 패키지 관리자

### 3.3.1 pnpm의 소개와 역사

- 다른 패키지 매니저와 달리 개인 개발자 졸탄 코찬이 개발한 프로젝트
- npm의 문제를 해결하기 위해 등장한 yarn classic도 여전히 문제를 가지고 있었기 때문에(현재는 해결됨) pnpm을 개발하게 됨
  - 여전히 평평한 의존성 트리를 사용해서 프로젝트가 직접 의존하지 않는 패키지에도 접근 가능
  - 평평한 의존성 트리를 만드는 과정이 복잡하고 느림
  - 일부 패키지는 고유의 node_modules 디렉터리를 필요로 하지만 평평한 트리 구조에서는 이를 관리하기 어려움

### 3.3.2 특징

#### 3.3.2.1 pnpm-lock.yml

- 다른 락 파일과 마찬가지로 설치되는 버전을 고정하기 위해 만들어짐
- yarn.lock처럼 YAML 파일 포맷 사용
- 패키지의 정합성을 보장하기 위해 SHA-512 기반의 해시 키를 포함하고 있으며, 해당 패키지의 의존성을 나타내는 dependencies 필드를 통해 각 패키지가 어떤 의존성을 가지고 있는지 확인할 수 있음

```yaml
packages:
  /react@18.3.1:
    resolution: { integrity: sha512-... }
    dependencies:
      loose-envify: 1.4.0
      object-assign: 4.1.1
```

- yarn.lock과 달리 dependencies 필드가 파일의 최상단에 위치해 있어, 현재 설치된 패키지와 버전을 한눈에 파악할 수 있음
- pnpm은 packages 섹션에서 설치된 패키지의 경로, 버전, 의존성 트리, 네임스페이스 구조를 함께 제공함
- 예를 들어 react의 의존성으로 loose-envify가 포함되어 있을 경우, yarn은 범위(^3.0.0, ^4.0.0 등)만 관리하지만 pnpm은 실제 설치된 버전(예: 4.0.0)을 명시함
  - 이러한 동작은 .npmrc 파일의 resolution-mode 설정을 통해 변경할 수 있으며, 같은 범위 내에서 중복 설치를 최소화하도록 설계되어 있음
- pnpm-lock.yaml은 yarn.lock이나 npm package-lock.json과 달리 가독성이 높고 diff 비교가 용이하도록 만들어졌으며, 패키지 관리자가 변경 사항을 빠르게 파악하고 작업 속도를 높일 수 있도록 돕는다는 장점이 있음

#### 3.3.2.2 글로벌 스토어의 하드 링크

- pnpm의 가장 큰 특징 중 하나는 글로벌 스토어를 활용한다는 점
- 각 패키지가 개별 프로젝트의 node_modules 폴더에 중복 설치되지 않고, 공용 저장소(~/.pnpm)에 한 번만 저장된 후 하드 링크로 연결됨

  - 하드 링크: 파일 시스템의 기능 중 하나로 동일한 데이터를 가리키는 여러 개의 파일 경로를 만드는 방식
  - 장점
    - 디스크 공간 절약: 여러 복사본을 만드는 대신 .pnpm에 있는 단일 패키지를 참조
    - 성능 향상: 파일을 복사하거나 상위 폴더를 계속 뒤져가며 찾는 것이 아니라 빠르게 원하는 패키지 찾을 수 있음
    - 일관성 유지와 안정성: 동일 파일을 여러 곳에서 참조하기 때문에 패키지 버전과 내용이 일관되게 유지

- 심볼릭 링크와 하드링크
  - 하드링크는 파일의 정보가 담겨있는 inode를 추가로 가리킴. 하지만 심볼릭 링크는 윈도우의 바로가기처럼 작동할뿐
  - 원본이 사라지면? 하드링크는 여전히 파일에 접근가능하지만 심볼릭 링크는 X
  - 비유
    - 하드링크: 등록된 집 주소(도로명주소와 지번주소)
    - 심볼릭링크: 주소를 적어놓은 쪽지

#### 3.3.2.3 평탄화되지 않은 node_modules

- pnpm은 평탄화된 node_modules 구조를 피하기 위해 다음과 같은 전략 채택

  - 프로젝트의 ./node_modules에는 선언된 dependencies와 같은 직접 의존성만 위치시킴
  - 이 패키지들의 실제 파일은 .pnpm에 하드 링크로 연결
  - 프로젝트의 직접 의존성에 필요한 패키지의 실제 원본은 ./node_modules 대신 ./node_modules/.pnpm에 평탄화해서 설치해둠

- package.json

```json
{
  "dependencies": {
    "react": "^18.3.1",
    "lodash": "^4.17.21"
  }
}
```

- node_modules 구조

```yaml
node_modules/
├─ .pnpm/
│  ├─ react@18.3.1/
│  │  ├─ node_modules/
│  │  │  ├─ react/                   ← react의 실제 코드
│  │  │  ├─ loose-envify/            ← react의 하위 의존성
│  │  │  ├─ object-assign/           ← react의 하위 의존성
│  │  │  └─ ...
│  │  └─ package.json
│  │
│  ├─ lodash@4.17.21/
│  │  ├─ node_modules/
│  │  │  └─ lodash/
│  │  └─ package.json
│  │
│  ├─ loose-envify@1.4.0/            ← 전역 저장소의 하드 링크 원본
│  │  └─ node_modules/loose-envify/
│  └─ object-assign@4.1.1/
│     └─ node_modules/object-assign/
│
├─ react → .pnpm/react@18.3.1/node_modules/react      ← 하드 링크
├─ lodash → .pnpm/lodash@4.17.21/node_modules/lodash  ← 하드 링크
└─ .bin/
```

#### 3.3.2.4 콘텐츠 어드레서블 스토리지

- 전통적인 npm 방식에서는 여러 프로젝트가 동일한 버전의 패키지를 사용하더라도, 각 프로젝트마다 중복 설치되어 디스크 공간 낭비와 설치 속도 저하가 발생함.
- pnpm의 방식

  - pnpm은 모든 의존성을 콘텐츠 어드레서블 스토리지에 저장
  - 파일의 내용을 기반으로 해시값(SHA-512 등)을 생성해 고유하게 식별하고, 동일한 파일은 한 번만 저장, 다른 프로젝트에서는 하드 링크 방식으로 재사용

- pnpm은 COW를 활용하여 위 기능을 가능하게 함
  - COW의 동작 원리
    - 데이터를 복사할 때 실제로 복사하지 않고 원본을 참조
    - 원본 데이터가 변경될 때만 새로운 복사본을 생성
- 왜 하드 링크 대신 COW인가?
  - 하드 링크는 하나의 파일 변경 시 모든 링크에 영향이 감
  - COW는 변경 시점에만 복사되어 스토어의 오염을 방지
  - 즉, node_modules 내용이 수정되어도 원본 스토어에는 영향 없음

#### 3.3.2.5 심볼릭 링크로 구성된 node_modules

- npm은 폴더명이 단순히 패키지명이지만, pnpm은 버전까지 명시되어 있음.
- npm의 경우 패키지 내용이 폴더 하위에 바로 위치하지만, pnpm에서는 .pnpm 폴더 하위에 위치함.
- pnpm은 패키지 간 의존 관계를 심볼릭 링크로 연결해 node_modules 구조를 구성함.
- 장점
  - 의존성 격리: 각 패키지가 자체 node_modules를 가져서 다른 의존성과 충돌하지 않음.
  - 버전 관리 용이: 버전별로 독립된 디렉터리를 사용해 여러 버전을 동시에 관리 가능.
  - 평탄화 문제 방지: 각 패키지가 실제로 참조하는 의존성에만 접근하므로 충돌 위험이 없음.
  - 디스크 공간 절약: 심볼릭 링크를 사용하므로 중복 파일 없이 여러 프로젝트에서 동일 패키지를 공유 가능.
- 예시
  - npm: node_modules/react/
  - pnpm: node_modules/.pnpm/react@18.3.1/node_modules/react
    → 실제 파일은 글로벌 스토어에 저장되고, node_modules에는 심볼릭 링크만 존재함.

#### 3.3.2.6 plug n play 지원

- pnpm도 pnp 모드를 지원하고 .npmrc 파일에 다음 두가지 설정을 추가하면 됨

```
node-linker=pnp
symlink=false
```

- node-linker: node_modules에 패키지를 설치할 때 어떤 링커 구조를 사용할지 정의함.
  - 설정 가능한 값:
    - isolated(기본값): pnpm의 기본 방식. pnpm이라는 가상 스토어 구조를 만들어 관리함
    - hoisted: npm이나 yarn classic처럼 평탄한 node_modules 구조를 사용
    - pnp: node_modules 폴더 없이, Yarn Berry의 PnP 방식으로 관리함
- pnpm의 pnp 모드는 yarn에서 불러온 코드를 기반으로 생성하므로 동작방식이 동일

## 3.4 npm, yarn, pnpm 비교

### 3.4.1 워크스페이스

#### 3.4.1.1 워크스페이스란 무엇인가?

- 일반적으로 하나의 프로젝트는 하나의 package.json을 가지고 단일 서비스나 라이브러리를 관리함
- 프로젝트 규모가 커질수록 여러 개의 자바스크립트 패키지로 분리해 의존성을 효율적으로 관리할 필요가 생김
- 대표 예시로 Babel이 있음
  – 수십 개의 플러그인과 핵심 라이브러리가 각각 독립된 패키지로 구성
- 워크스페이스는 이러한 여러 패키지를 하나의 프로젝트 내에서 통합 관리하고, 패키지 간 의존성을 원활히 연결하도록 돕는 구조
- Lerna라는 라이브러리를 사용해서 관리했던 떄가 있었으나 유지보수가 제대로 안되면서 어려움이 생기고 yarn과 pnpm이 워크스페이스 기능을 지원하게 되면서 많은 개발자들이 전환

#### 3.4.1.2 npm의 워크스페이스

- npm 워크스페이스는 7.x 버전부터 도입된 기능으로, 하나의 최상위 프로젝트(package.json)에서 여러 하위 패키지를 효율적으로 관리하기 위해 설계됨
- npm install을 실행하면 하위 패키지들이 링크로 연결되고, 각 패키지의 node_modules는 필요한 의존성을 심볼릭 링크로 참조함
- 이를 통해 하위 패키지 간 의존성이 자동으로 연결되어 패키지 관리 효율이 향상됨

- 실습
  - package-lock.json과 node_modules가 하나만 있음
  - symbolic link로 연결되어있어서 하위 패키지의 내용을 별도로 복사하거나 배포해놓지 않더라도 필요한 내용 참조 가능
  - 패키지 내에서 다른 패키지를 참조할 수 있기도 함

#### 3.4.1.3 yarn의 워크스페이스

- 실습
  - 레지스트리에서 설치된 일반 패키지는 ./yarn/berry라는 글로벌 스토어 경로 참조. 워크스페이스 내부에서 생성된 패키지들은 워크스페이스 최상위 위치를 기준으로 한 실제 로컬 경로를 참조

#### 3.4.2.1 의존성 관리

- package.json에 있는 의존성 설치
  - npm: npm install
  - yarn: yarn install
  - pnpm: pnpm install- 락 파일 기준으로 의존성 설치
  - npm: npm ci
  - yarn: yarn install --immutable --immutable-cache
  - pnpm: pnpm install --frozen-lockfile
- package.json의 버전 규칙에 맞춰 최신 버전으로 업데이트
  - npm: npm update
  - yarn: yarn up -R (※ yarn-plugin-semver-up 플러그인 필요)
  - pnpm: pnpm update
- 버전 규칙과 상관없이 모두 최신 버전으로 업데이트
  - npm: (없음)
  - yarn: yarn up
  - pnpm: pnpm update --latest
- 특정 패키지 최신 버전으로 업데이트
  - npm: npm update <패키지명>
  - yarn: yarn up <패키지명> (※ yarn-plugin-semver-up 필요)
  - pnpm: pnpm update <패키지명>
- 특정 패키지를 버전 규칙과 상관없이 최신 버전으로 업데이트
  - npm: npm update <패키지명>@latest
  - yarn: yarn up <패키지명> --latest
  - pnpm: pnpm update --latest <패키지명>
- 인터랙티브 의존성 업데이트
  - npm: (없음)
  - yarn: yarn upgrade-interactive
  - pnpm: pnpm update --interactive
- dependencies 추가
  - npm: npm install <패키지명>
  - yarn: yarn add <패키지명>
  - pnpm: pnpm add <패키지명>
- devDependencies 추가
  - npm: npm install <패키지명> --save-dev
  - yarn: yarn add <패키지명> --dev
  - pnpm: pnpm add <패키지명> --save-dev
- 의존성 제거
  - npm: npm uninstall <패키지명>
  - yarn: yarn remove <패키지명>
  - pnpm: pnpm remove <패키지명>
- 중복된 의존성 정리
  - npm: npm dedupe
  - yarn: yarn dedupe
  - pnpm: pnpm dedupe
- 특정 패키지 의존성 확인
  - npm: npm why <패키지명>
  - yarn: yarn why <패키지명>
  - pnpm: pnpm why <패키지명>

#### 3.4.2.2 글로벌 패키지 실행

- 전역 패키지 설치

  - npm: npm install -g <패키지명>
  - yarn: yarn berry에는 전역 패키지 설치라는 개념이 없음
  - pnpm: pnpm add --global <패키지명>

- 전역 패키지 업데이트
  - npm: npm update --global <패키지명>
  - yarn: Yarn Berry에는 전역 패키지 업데이트 개념이 없음
  - pnpm: pnpm update --global <패키지명>
- 전역 패키지 삭제
  - npm: npm remove --global <패키지명>
  - yarn: Yarn Berry에는 전역 패키지 삭제 개념이 없음
  - pnpm: pnpm remove --global <패키지명>
- 패키지에서 명령 실행
  - npm: npx <명령어>
  - yarn: yarn dlx <명령어>
  - pnpm: pnpm dlx <명령어>

#### 3.4.2.3 그 밖에 자주 쓰이는 명령어

- 프로젝트 초기 설정
  - npm: npm init
  - yarn: yarn init
  - pnpm: pnpm init
- 프로젝트를 레지스트리에 업로드
  - npm: npm publish
  - yarn: yarn npm publish
  - pnpm: pnpm publish
- 라이선스 정보 확인
  - npm: 없음
  - yarn: yarn-plugin-licenses 플러그인으로 확인 가능
  - pnpm: pnpm licenses
- 패키지 관리자 자체 버전업
  - npm: npm install -g npm, nvm, package.json에 packageManager 필드를 선언하는 corepack 사용
  - yarn: package.json에 packageManager 필드를 선언하는 corepack 사용
  - pnpm: npm install -g npm, package.json에 packageManager 필드를 선언하는 corepack 사용
- 보안 취약점 검사
  - npm: npm audit
  - yarn: yarn npm audit
  - pnpm: pnpm audit

### 3.4.3 벤치마크 테스트

#### 3.4.3.1 Next.js 설치 비교(2024.08~11)

##### 3.4.3.1.1 Full Cold

- 완전히 새로운 머신에서 프로젝트를 처음 클론하고 설치하는 과정
  - yarn classic < yarn node_modules, npm < yarn < pnpm (오른쪽으로 갈수록 빠름)
  - pnpm은 해당 프로젝트의 의존성이 파악되는 즉시 병렬로 다운로드하기 때문
    - 의존성 파악, 다운로드, 링크 과정을 병렬로 처리 ([각 단계에 대한 설명 - 토스 블로그](https://toss.tech/article/lightning-talks-package-manager))

##### 3.4.3.1.2 캐시만 존재하는 경우

- 캐시만 존재하는 경우(락파일 x)
  - npm < yarn node_modules, yarn classic < pnpm < yarn pnp
  - yarn pnp는 캐시만 있다면 pnp.cjs를 생성해서 경로만 지정해주면 되고 pnpm은 해당 글로벌 캐시를 COW 처리만 해주면 되기 때문

##### 3.4.3.1.3 캐시와 락 파일 모두가 존재하는 경우

- git 같은 버전 관리 시스템에서 저장소를 클론한 직후에 node_modules가 없는 경우
  - npm < yarn classic, yar node_modules < yarn pnp, pnpm

##### 3.4.3.1.4 pnpm 벤치마크

- [사이트](https://pnpm.io/benchmarks)

#### 3.4.3.2 벤치마크 테스트 결론

- npm은 특정 상황(npm용 node_modules가 미리 설정되어 있는 경우)을 제외하면 최악의 선택
- yarn classic 또한 매우 느리고 업데이트도 잘 안되고 있음
- yarn berry와 pnpm이 가장 빠른 성능 -> 가장 합리적인 선택
