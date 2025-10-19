# Npm Deep Dive
Dasom, 2025.10.12

## 3장 npm의 대항마 yarn과 pnpm 

### 3.3 pnpm: 디스크 공간 절약과 설치 속도의 혁신을 가져온 패키지 관리자

> pnpm은 2017년에 등장한 패키지 관리자로, 글로벌 스토어와 하드 링크를 활용하여 디스크 공간을 획기적으로 절약하고 설치 속도를 개선한 것이 가장 큰 특징이다.

#### 3.3.1 pnpm의 소개와 역사

* yarn의 한계에서 영감을 받아 2017년에 등장
* 다른 패키지 매니저와 달리 개인 개발자 졸탄 코찬이 개발한 프로젝트

**pnpm의 메인 개발자가 지적한 npm과 yarn classic의 문제점**
* 여전히 평평한 의존성 트리를 사용 (현재 PnP 모드를 기본으로 제공하는 yarn berry는 이 문제를 대부분 해결)
* 프로젝트가 직접 의존하지 않는 패키지에도 접근이 가능함
* 평평한 의존성 트리를 만드는 과정이 복잡하고 느림
* 일부 패키지는 고유의 node_modules 디렉터리를 필요로 하지만, 평평한 트리 구조에서는 이를 관리하기 어려움

#### 3.3.2 pnpm의 특징

**3.3.2.1 pnpm-lock.yml**
* yarn.lock과 마찬가지로 YAML 파일 포맷을 사용
* 패키지의 정합성을 보장하기 위해 SHA-512 기반의 해시 키를 포함

**주요 특징**
* dependencies 필드를 통해 각 패키지가 어떤 의존성을 가지고 있는지 확인 가능
* dependencies 필드가 락 파일 최상단에 위치하여 현재 설치된 패키지와 버전을 한눈에 파악 가능
* packages 섹션에서 설치된 패키지의 경로, 버전, 의존성 트리, 네임스페이스 구조를 함께 제공

**버전 처리 방식의 차이**
* react의 의존성인 loose-envify가 `"js-tokens": "^3.0.0 || ^4.0.0"`을 가질 때
* yarn은 이 정보를 버전 범위로 관리
* pnpm은 이를 최상위 허용 버전인 4.0.0으로 설정해서 관리
* 이러한 동작은 .npmrc 파일의 resolution-mode 설정을 통해 변경 가능

cf. pnpm-lock.yaml은 yarn.lock이나 package-lock.json과 달리 가독성이 높고 diff 비교가 용이하도록 만들어졌으며, 패키지 관리자가 변경 사항을 빠르게 파악하고 작업 속도를 높일 수 있다는 장점이 있다.

**3.3.2.2 글로벌 스토어의 하드 링크**

> pnpm의 가장 큰 특징 중 하나는 글로벌 스토어를 활용한다는 점으로, 각 패키지가 개별 프로젝트의 node_modules 폴더에 중복 설치되지 않고 공용 저장소(~/.pnpm)에 한 번만 저장된 후 하드 링크로 연결된다.

* pnpm은 각 패키지가 설치하는 하위 의존성을 node_modules가 아닌 기본적으로 ~/.pnpm이라고 하는 글로벌 스토어에 저장
* 필요한 의존성들을 하드 링크라는 방식으로 연결해서 사용

**하드 링크란?**
* 파일 시스템의 기능 중 하나로, 동일한 데이터를 가리키는 여러 개의 파일 경로를 만드는 방식
* 각 경로는 같은 파일을 참조하기 때문에 어느 경로로 접근하든 동일한 파일에 도달할 수 있음

**하드 링크 방식의 장점**
* 디스크 공간의 절약: 여러 패키지에서 동일한 패키지를 참조할 때 여러 복사본을 만드는 대신 .pnpm에 있는 단일 패키지를 참조
* 성능 향상: 파일을 복사하거나 상위 폴더를 계속 뒤져가면서 node_modules를 찾는 대신 하드 링크를 사용해 빠르게 원하는 패키지를 찾을 수 있음
* 일관성 유지와 안정성: 하드 링크로 동일한 파일을 여러 곳에서 참조하므로 패키지 버전과 내용이 일관되게 유지

**하드 링크와 심볼릭 링크(소프트 링크) 비교**

**공통점**
* 원본 파일이 수정되면 이를 참조하는 모든 링크에서도 변경 사항을 볼 수 있음

**차이점**
* 하드 링크는 inode를 추가로 가리키는 것
  * 원본이 삭제되더라도 같은 inode를 가리키는 다른 하드 링크가 있으면 파일이 살아있음
  * 같은 파일 시스템 안에서만 사용 가능
  * 원본과 속성도 똑같이 공유
* 심볼릭 링크는 윈도우의 바로가기처럼 작동
  * 원본이 삭제되면 심볼릭 링크는 사용할 수 없음
  * 다른 파일 시스템에서도 만들 수 있음
  * 원본 파일과는 다른 독립적인 속성을 가질 수 있음

cf. inode는 파일이 생성될 때마다 부여되는 고유번호로, 해당 파일의 정보가 담겨있다.

**3.3.2.3 평탄화되지 않은 node_modules**

pnpm은 평탄화된 node_modules의 구조를 피하기 위해 아래와 같은 전략을 채택함:

1. 프로젝트의 ./node_modules에는 선언된 dependencies와 같은 직접 의존성만 위치시킨다
2. 이 패키지들의 실제 파일을 .pnpm에 하드 링크로 연결한다
3. 프로젝트의 직접 의존성에 필요한 패키지의 실제 원본은 ./node_modules 대신 ./node_modules/.pnpm에 평탄화해서 설치해둔다
4. 각 패키지의 의존성은 앞서 평탄화해둔 .pnpm에 하드 링크로 참조하게 한다

**pnpm의 node_modules 구조**
```
node_modules/
├─ .pnpm/
│  ├─ react@18.3.1/
│  │  ├─ node_modules/
│  │  │  ├─ react/                   ← react의 실제 코드
│  │  │  ├─ loose-envify/            ← react의 하위 의존성
│  │  │  └─ object-assign/           ← react의 하위 의존성
│  │
│  ├─ lodash@4.17.21/
│  │  └─ node_modules/
│  │     └─ lodash/
│  │
│  ├─ loose-envify@1.4.0/            ← 글로벌 스토어의 하드 링크 원본
│  └─ object-assign@4.1.1/
│
├─ react → .pnpm/react@18.3.1/node_modules/react      ← 하드 링크
├─ lodash → .pnpm/lodash@4.17.21/node_modules/lodash  ← 하드 링크
└─ .bin/
```

**3.3.2.4 콘텐츠 어드레서블 스토리지**

* 파일이나 데이터 블록을 고유한 해시값으로 식별해서 저장하는 시스템
* 해시값은 파일 내용을 기반으로 생성되므로 동일한 파일은 동일한 해시값을 갖게 됨

**전통적인 npm 방식의 문제**
* 프로젝트 수만큼 동일한 리액트 패키지가 중복 설치됨
* 디스크 공간 낭비와 설치 속도 저하

**pnpm의 해결 방법**
* 의존성을 콘텐츠 어드레서블 스토리지에 저장
* node_modules가 콘텐츠 어드레서블 스토리지에 있는 파일을 참조하기 위해 Copy-on-Write(COW)라는 기술을 사용
* COW를 지원하지 않는 환경의 경우 하드링크를 사용

**Copy-on-Write(COW)**
* 데이터 복사를 최적화하는 기술
* 데이터의 복사본을 생성할 때 원본 데이터를 그대로 참조하고 실제 복사를 수행하지 않음
* 원본 데이터가 변경될 때만 새로운 복사본을 생성하는 방식
* 하드 링크는 하나의 파일 변경 시 모든 링크에 영향이 가지만, COW는 변경 시점에만 복사되어 스토어의 오염을 방지

**3.3.2.5 심볼릭 링크로 구성된 node_modules**

react@18.3.1만 있는 프로젝트를 npm과 pnpm으로 각각 설치 후 비교했을 때의 차이점:

**구조적 차이**
* npm은 폴더명이 단순히 패키지명이지만, pnpm은 버전까지 명시되어 있음
* npm은 패키지 내용이 폴더 하위에 바로 위치하지만, pnpm은 node_modules 폴더 하위에 위치함

**pnpm 구조의 장점**
* 의존성 격리: 각 패키지는 자체 node_modules를 가지고 있어 다른 의존성과 충돌하지 않음
* 버전 관리의 용이성: 각 패키지 버전이 독립된 공간을 차지하므로 여러 버전을 관리하기가 용이함
* 평탄화된 node_modules의 위험성 회피: 각 의존성은 실제로 자신이 참조하는 의존성에만 접근할 수 있으므로 평탄화된 node_modules로 인해 발생할 수 있는 문제를 회피

**3.3.2.6 Plug'n'Play 지원**

pnpm에서 PnP 모드를 활성화하기 위해 .npmrc 파일에 아래 설정을 추가:

```
node-linker=pnp
symlink=false
```

**node-linker**
* node_modules에 패키지를 설치할 때 사용할 링커를 정의
* 옵션: isolated(기본값) / hoisted / pnp
  * isolated: pnpm의 기본 방식으로 .pnpm 가상 스토어 구조를 만들어 관리
  * hoisted: npm이나 yarn classic처럼 평탄한 node_modules 구조를 사용
  * pnp: node_modules 폴더 없이 Yarn Berry의 PnP 방식으로 관리

**symlink**
* boolean 값
* node-linker를 pnp로 설정할 경우 symlink: false를 지정해야 함

cf. pnpm이 생성한 PnP 모드의 결과물은 yarn의 PnP와 거의 유사하다. 실제로 pnpm의 PnP 모드는 yarn에서 불러온 코드를 기반으로 생성하기 때문에 방식이 거의 동일하다.

### 3.4 npm, yarn, pnpm 비교

#### 3.4.1 워크스페이스

**3.4.1.1 워크스페이스란 무엇인가?**

> 프로젝트의 규모가 커지고 복잡해지면서 하나의 프로젝트를 여러 개의 자바스크립트 패키지로 분리하고, 패키지 간의 의존성을 효율적으로 관리할 필요성이 증가했다. 워크스페이스 구조는 독립적인 패키지들이 하나의 프로젝트 내에서 원활하게 작동하고 관리될 수 있도록 지원한다.

**대표적인 예: 바벨**
* 최신 자바스크립트 기능을 다양한 브라우저에서 사용할 수 있도록 코드 변환을 지원하는 트랜스파일러 라이브러리
* 수십 가지의 플러그인을 필요로 하며, 각 플러그인과 핵심 라이브러리가 독립된 패키지로 존재

**모노레포 vs 워크스페이스**
* 둘 다 여러 프로젝트(패키지)를 하나의 저장소에서 관리하기 위한 개념
* 모노레포: 단일 저장소에서 여러 패키지를 함께 관리하는 방식
* 워크스페이스: 패키지 매니저가 제공하는 기능으로 여러 패키지를 효율적으로 설치하고 관리하는 도구
* 워크스페이스 기능은 npm, yarn, pnpm 모두 기본적으로 지원

**3.4.1.2 npm의 워크스페이스**

* 비교적 최근인 7.x 버전에 도입
* 하나의 최상위 패키지에서 여러 하위 패키지를 효율적으로 관리하기 위해 설계됨

**동작 방식**
* npm install 실행 시 하위 패키지들이 링크로 연결
* 각 패키지의 node_modules 폴더에 필요한 패키지들이 심볼릭 링크로 참조
* 하위 패키지 간의 의존성도 자동으로 연결되어 패키지 관리를 더욱 효율적으로 할 수 있음

**npm 워크스페이스의 특징**
* 최상위 경로에 단 하나의 package-lock.json만 생성
* node_modules도 최상위 경로에 단 하나만 존재 (Node.js의 모듈 참조 방식으로 인해 가능)
* package.json의 workspaces 필드에 관리하고자 하는 폴더를 추가해서 설정
* 하위 패키지의 명령어 실행 시 `--workspaces` 옵션 추가
* npm에 아직 릴리스되지 않은 하위 패키지들은 node_modules에 심볼릭 링크 형태로 연결

**3.4.1.3 yarn의 워크스페이스**

* 패키지 관리자 생태계에서 워크스페이스 개념을 처음 도입 (yarn classic 시절부터 지원)
* npm과 마찬가지로 최상위 package.json 파일에 workspaces 필드를 추가해서 설정 가능

**yarn workspaces foreach 명령어 옵션**
* `--all`: 프로젝트 내 모든 워크스페이스에서 제공된 명령어를 실행
* `--parallel`: 워크스페이스 작업을 병렬로 실행
* `--since`: 현재 브랜치를 main 브랜치와 비교해서 수정된 패키지를 대상으로만 명령어를 실행
* `-pt`: --parallel과 --topological을 합친 것으로 명령어를 병렬로 실행하되 패키지 의존성 순서에 맞춰서 진행

**3.4.1.4 pnpm의 워크스페이스**

* 워크스페이스를 선언하기 위해 pnpm-workspace.yaml 파일을 프로젝트 최상위 디렉터리에 생성해야 함
* yarn과 마찬가지로 패키지 버전을 선언할 때 `workspace:` 접두사를 사용하는 워크스페이스 전용 프로토콜을 지원

**pnpm 워크스페이스 명령어**
* `pnpm run -r`: 워크스페이스 내 모든 프로젝트를 순회하며 명령어를 실행 (r은 recursive의 약자)
* 별도의 명령어 없이도 패키지 의존성 관계에 따라 명령어를 실행 (의존성이 없으면 알파벳 순서로 실행)
* `--filter` 옵션을 통해 특정 워크스페이스에서만 명령어가 실행되도록 제한 가능

cf. pnpm은 워크스페이스에 필요한 기본 기능을 간결하게 제공하지만, 버저닝과 같은 복잡한 기능은 changesets나 Rush 같은 서드파티 라이브러리를 사용하는 것을 권장한다. npm 레지스트리에 패키지를 업로드하기 위해서는 버저닝이 매우 중요하므로 pnpm+서드파티 라이브러리 조합은 필수적이다.

#### 3.4.2 명령어 비교

**3.4.2.1 의존성 관리**

**package.json에 있는 의존성 설치**
* npm: `npm install`
* yarn: `yarn install` 또는 `yarn`
* pnpm: `pnpm install`

**lock 파일 기준으로 엄격하게 설치**
* npm: `npm ci`
* yarn: `yarn install --immutable --immutable-cache`
* pnpm: `pnpm install --frozen-lockfile`

**package.json의 버전 규칙에 맞춰 최신 버전으로 업데이트**
* npm: `npm update`
* yarn: `yarn up -R '**'` (yarn-plugin-semver-up 플러그인 필요)
* pnpm: `pnpm update`

**버전 규칙에 상관없이 모두 최신 버전으로 업데이트**
* npm: 없음
* yarn: `yarn up`
* pnpm: `pnpm update --latest`

**특정 패키지만 최신 버전으로 업데이트**
* npm: `npm update <패키지명>@latest`
* yarn: `yarn up <패키지명>`
* pnpm: `pnpm update --latest <패키지명>`

**인터랙티브 의존성 업데이트**
* npm: 없음
* yarn: `yarn upgrade-interactive`
* pnpm: `pnpm update --interactive`

**의존성 제거**
* npm: `npm uninstall <패키지명>`
* yarn: `yarn remove <패키지명>`
* pnpm: `pnpm remove <패키지명>`

**중복되는 범위의 의존성 제거**
* npm: `npm dedupe`
* yarn: `yarn dedupe`
* pnpm: `pnpm dedupe`

**해당 패키지가 왜 필요한지 확인**
* npm: `npm why <패키지명>`
* yarn: `yarn why <패키지명>`
* pnpm: `pnpm why <패키지명>`

**3.4.2.2 글로벌 패키지 실행**

**전역 패키지 설치**
* npm: `npm install --global <패키지명>`
* yarn: yarn berry에는 전역 패키지 설치하는 개념이 없음
* pnpm: `pnpm add --global <패키지명>`

**패키지에서 명령 실행**
* npm: `npm exec <패키지명>` 또는 `npx <패키지명>`
* yarn: `yarn dlx <패키지명>`
* pnpm: `pnpm dlx <패키지명>`

**3.4.2.3 그 밖에 자주 쓰이는 명령어**

**프로젝트 초기 설정**
* npm: `npm init`
* yarn: `yarn init`
* pnpm: `pnpm init`

**프로젝트 레지스트리에 업로드**
* npm: `npm publish`
* yarn: `yarn npm publish`
* pnpm: `pnpm publish`

**보안 검사**
* npm: `npm audit`
* yarn: `yarn npm audit`
* pnpm: `pnpm audit`

#### 3.4.3 벤치마크 테스트

**3.4.3.1 Next.js 설치 비교**

**3.4.3.1.1 Full Cold (완전히 캐시가 없는 상태)**
* 완전히 새로운 머신에서 프로젝트를 처음 클론하고 설치하는 과정
* 완전히 캐시가 없는 상태에서 설치하는 경우는 드물기 때문에 참고만 할 것
* 성능: pnpm > yarn > yarn node_modules = npm > yarn classic

**3.4.3.1.2 캐시만 존재하는 경우**
* yarn pnp가 pnpm보다 아주 약간 빠름
* 성능: yarn pnp > pnpm > yarn node_modules = yarn classic > npm
* yarn pnp은 캐시만 있다면 pnp.cjs를 생성해서 경로만 지정해주면 되고, pnpm은 해당 글로벌 캐시를 COW 처리만 해주면 되기 때문에 가장 빠름

**3.4.3.1.3 캐시와 락 파일 모두가 존재하는 경우**
* Git 같은 버전 관리 시스템에서 저장소를 클론한 직후 node_modules 폴더가 없는 것과 동일
* 일반적인 CI 환경에 해당됨
* 성능: pnpm > yarn pnp > yarn classic > yarn node_modules > npm

**3.4.3.2 벤치마크 테스트 결론 (2024.11 기준)**

* npm은 특정 상황을 제외하면 다른 패키지 관리자에 비해 현저히 느림
* yarn classic도 대부분의 경우 pnpm, yarn berry에 비해 느림
  * yarn classic은 보안 패치 외 특별한 기능 개선이 이뤄지지 않고 있으므로 선택할 이유가 거의 없음
* 대부분의 경우 yarn berry와 pnpm이 가장 빠른 성능을 보였음
  * yarn berry가 가장 뛰어난 성능을 나타냄
  * pnpm이 하드링크나 COW 작업이 필요한 반면, yarn berry는 필요한 의존성을 다운로드하면서 동시에 미리 준비된 pnp.cjs 파일에 현재 의존성 그래프를 JSON 형식으로 생성하기만 하면 되기 때문

cf. 모던 자바스크립트 프로젝트를 만드는 경우 yarn berry나 pnpm을 선택하는 것이 가장 합리적이다.
