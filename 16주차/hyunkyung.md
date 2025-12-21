## 8.2 나만의 모노레포 프로젝트 만들기

### 8.2.1 디자인 시스템 소개

#### 8.2.1.1 디자인 시스템이란?

- 확장 가능한 디자인을 관리하기 위한 표준적인 표준 세트
- 디자인 시스템의 구성요소
  - 컴포넌트
    - 페이지를 구성하는 요소 중 재사용이 가능한 분리된 조각들
    - ex) 인풋, 버튼 ...
    - 작은 컴포넌트를 조합해서 더 큰 단위의 컴포넌트를 생성할 수도 있음
    - 작은 단위에서 더 큰 구체적인 단위로 결합하여 문제 해결하는 전략: 아토믹 디자인
    - 디자인 시스템의 컴포넌트는 아토믹 디자인의 원자, 분자, 유기체 단위까지 주로 해당
  - 패턴
    - 시스템을 사용할 제품에서 이러한 컴포넌트들을 논리적이고 일관되게 사용할 수 있게 하는 가이드라인
    - 컴포넌트에 대한 사용법
- 모범적으로 잘 만들어진 디자인 시스템 예시
  - [카본 디자인 시스템](https://carbondesignsystem.com/)
  - [머티리얼 디자인](https://m3.material.io/)
  - [오르빗 디자인 시스템](https://orbit.kiwi/)

#### 8.2.1.2 예제 프로젝트 소개

- [ndive-figma](https://www.figma.com/design/60fHi2F04BbdqDQWigiFjG/ndive-design-system)

##### 8.2.1.2.1. 패키지 목록

- 디자인 토큰을 추출하는 패키지 @ndive/design-tokens
  - 디자인 토큰: 다양한 컴포넌트에서 일관된 스타일을 유지하기 위해 각 속성에 이름을 부여한 값 [(example)](https://www.figma.com/design/60fHi2F04BbdqDQWigiFjG/ndive-design-system?node-id=6-54&t=HVtX4Amtps5ThIsM-1)
  - REST API를 통해 토큰을 자동으로 추출하는 기능
- 디자인 시스템 컴포넌트 패키지 @ndive/design-components
  - 버튼과 모달 같은 컴포넌트들을 포함하는 디자인 컴포넌트 패키지
  - 토큰 패키지를 통해 스타일을 활용하도록 작성
- 디자인 시스템 분석 툴 패키지 @ndive/design-tracker
  - react-scanner 처럼 리액트 컴포넌트 코드를 정적으로 분석해 컴포넌트와 프로퍼티의 사용 통계를 조회하는 패키지

##### 8.2.1.2.2 모노레포가 적합한 이유

- 복잡성: 각 컴포넌트가 독립적으로 관리되지만 전체 시스템의 일부분으로 동작해야 함(복잡한 구조를 한곳에서 관리)
- 버전 관리: 관심사를 분리하여 각 패키지가 필요 이상의 업데이트나 의존성을 받지 않게 설계할 수 있음
- 협업: 모든 코드를 한 곳에서 관리하므로 피드백 단계를 단축하고 디자인 시스템의 변경 사항을 즉시 확인하고 테스트하는 데 도움을 줄 수 있음
- 빌드 최적화 및 성능 개선: 터보레포와 같은 도구를 통해 캐싱이나 병렬 실행처럼 다양한 매커니즘을 통해 디자인 시스템 내 여러 패키지의 빌드를 최적화할 수 있음

##### 8.2.1.2.3 개발에 도움을 주는 내부 워크스페이스

- 비공개지만 개발을 도와주는 내부 워크스페이스를 둘 수 있음
- 공유 코드 재사용: 여러 프로젝트에서 반복적으로 사용하는 로직을 비공개 패키지로 만들어 모노레포에서 쉽게 공유할 수 있음 ex) tsconfig 설정
- 일관성 유지: 공통 코드의 변경이 발생할 경우 한 곳에서 관리
- 의존성 관리: 불필요한 외부 패키지 의존을 줄여 성능을 높이고 유지보수를 쉽게 만듦
- 문서화의 일관성: 테스트와 문서화를 담당하는 내부 워크스페이스를 포함하여 동일한 기준과 형식으로 문서화

### 8.2.2 pnpm 워크스페이스 및 터보레포 구성하기

- 기술 스택: 타입스크립트, 비트, 터보레포, pnpm + pnpm 워크스페이스

#### 8.2.2.2 pnpm 설정

- 최상단 package.json에 모노레포임을 명확히 하기 위해 "private" 필드 추가하고 패키지명 수정
  - 배포용이 아니라 관리용이라는 것 명시
- pnpm-workspace.yaml 생성
  - 공개 패키지는 /packages, 스토리북과 같은 애플리케이션은 /apps, 비공개 패키지는 /shard에 위치

#### 8.2.2.3 터보레포 초기 설정

- 최상단 package.json에 의존성을 추가하기 위해 `--workspace-root` 옵션을 지정하여 터보레포 환경 구성

```
pnpm add -D turbo --workspace-root
```

- turbo.json 추가
  - 코드를 빌드(build)하는 작업과 빌드하기 전에 이전 빌드 결과물을 제거(clean)하는 초기화 작업 필요
    - clean은 실행할때마다 반드시 실행해야 하므로 cache: false로 설정
    - build는 clean 작업 이후에 실행되어야 하므로 clean을 dependsOn에 추가
    - clean과 build 작업의 dependsOn에 ^를 설정했기 때문에 워크스페이스에서 동일하게 설정된 모든 작업을 올바른 의존성 순서대로 처리할 것
      - 내가 의존하는 패키지부터 실행 후 내꺼 실행

### 8.2.3 shared 공유 패키지 구현하기

#### 8.2.3.1 tsconfig.json의 공통 설정을 관리하는 패키지 @ndive/tsconfig

1. package.json 작성
2. 공통 구성을 담을 base.json 파일 작성
3. base.json을 확장한 react.json과 node.json 생성
4. package.json files 필드에 내보낼 파일들을 설정해 외부에서 구성 파일을 불러올 수 있게 함

#### 8.2.3.2 비트의 defineConfig() 래퍼 함수 패키지, @ndive/vite

1. package.json 작성
   - esmodule로 배포되므로 type 필드를 module로 설정
   - 비트 번들러와 함께 동작해야 하므로 vite 패키지를 peerDependencies로 추가
2. defineConfig 파일 작성
3. package.json에 exports와 main 경로를 추가해 패키지의 모듈 경로를 설정

#### 8.2.3.3

1. pnpm build 실행

### 8.2.4 @ndive/design-tokens 구현

#### 8.2.4.1 피그마 REST API

- 피그마에서 디자인 파일 및 데이터를 외부 애플리케이션에서 접근하고 활용할 수 있도록하는 공개 API
- 피그마 내 디자인 속성들은 코드로 전환 가능한 데이터 포맷
- 디자인 토큰을 추출하기 위해 주요하게 쓰일 두가지 API
  - https://api.figma.com/v1/files/:filekey
    - 피그마 파일의 키 값으로 해당 파일의 전체 구조를 트리 자료구조로 가져옴
    - 이때 데이터로부터 얻는 디자인 파일의 계층 구조를 구성하는 모든 요소를 노드라고 함 ex) 텍스트, 이미지, 그룹 등
  - https://api.figma.com/v1/files/:filekey/nodes?ids=1,2,3:https://api.figma.com/v1/files/:filekey
    - API로 얻은 노드 아이디로 해당 노드의 디자인 속성들을 조회
    - 각 노드는 내부 속성으로 색상, 위치, 크기와 같은 디자인 속성을 포함하고 있으며, 이 API를 통해 노드의 색상, 폰트, 크기 등 구체적인 정보를 가져올 수 있음

#### 8.2.4.2 @ndive/tsconfig를 참조해 타입스크립트 설정하기

1. package.json 에서 devDependencies로 워크스페이스 참조
2. tsconfig.json에서 extends

#### 8.2.4.3 디자인 토큰 추출하기

1. access token은 외부에서 저장후 fetch 해야함. design-components 폴더의 .env 파일에 저장 후 fetch

#### 8.2.5.2 @ndive/design-tokens 활용하기

- fetchColor.js를 이용해 color.json을 color.scss로 변환 (아이콘, 폰트도 추가)
- package.json에 해당 파일들을 호출하는 스크립트 작성
- pnpm fetch:color를 실행하면 script를 찾지 못한다는 오류 발생 -> turbo.json에 Tasks 정의
- @ndive/design-components에 국한된 작업이므로 해당 워크스페이스 내부에 turbo.json을 두고 작성 (이때 세가지 작업을 묶어 fetch:tokens로 정의)

#### 8.2.5.3 컴포넌트 작성

- 이때 아이콘을 src/index.ts 파일에 포함하지 않고 따로 내보냄 따로 (트리 셰이킹을 위해) -> add-icons-to-exports

#### 8.2.5.4 빌드하기

- vite.config.ts 파일에 지원 브라우저 환경 추가
- 디자인 시스템의 모든 스타일을 담아 style.css 파일 따로 내보내기 (nextjs 같은 정적 빌드 환경에서 효율적)
  - 컴포넌트별로 스타일이 중복 로딩되지 않음
  - 초기 렌더링 성능 최적화(head에 미리 담기)
  - style.css 캐싱
- vite-plugin-static-copy를 통해 정적 파일들을 원하는 경로로 복사

### 8.2.6 @ndive/design-tracker 구현

- 패키지를 사용하는 프로젝트에서 package.json에 pnpm run track 명령어 추가
- 결과물 data.json

```
{
    "ButtonPrimary": {
        "instance": 2,
        "props": {
            "text": {
                "type": "string",
                "count": 2
            },
            "size": {
                "type": "'small' | 'medium' | 'large'",
                "count": 2
            },
            "fillType": {
                "type": "'fill' | 'line'",
                "count": 2
            },
            "color": {
                "type": "'mainGreen' | 'mainYellow' | 'red' | 'teal'",
                "count": 2
            },
            "icon": {
                "type": "{ direction: 'front' | 'back'; component: React.ReactNode; } | undefined",
                "count": 1
            },
            "onClick": {
                "type": "React.MouseEventHandler<HTMLButtonElement> | undefined",
                "count": 1
            }
        }
    }
    ...
```

### 8.2.8 배포 살펴보기

- changesets 통해 배포 자동화
- [여러 워크스페이스들의 변화 감지](https://github.com/yujeongJeon/ndive-design-system/pull/52)
