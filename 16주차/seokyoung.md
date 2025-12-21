## 디자인 시스템이란?

> _확장 가능한 디자인을 관리하기 위한 포괄적인 표준 세트_

- 컴포넌트(component) + 패턴(pattern)
  - 컴포넌트: 기술과 기능에 초점
  - 패턴: 컴포넌트에 대한 사용법을 기술

결론적으로 디자인 시스템은 컴포넌트와 패턴으로 하나의 시스템을 형성해 디자이너, 개발자 등 이해관계자들의 협업을 촉진하고, 시각적인 일관성을 보장하며, 중복된 작업을 줄이는 데 중점을 둔다.

이를 통해 제품 디자이의 일관성을 유지하고, 디자인 및 개발 작업을 효율화하는 데 목적이 있다.

[→ Everything you need to know about Design Systems](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969?gi=6143b15bcb12)

## 디자인 시스템 만들기

> 피그마 → [ndive-design-system](https://www.figma.com/design/60fHi2F04BbdqDQWigiFjG/ndive-design-system?node-id=6-54&p=f)

### 패키지 목록

- @ndive/design-tokens: 디자인 토큰 추출 패키지
- @ndive/deisgn-components: 디자인 시스템 컴포넌트 패키지
- @ndive/design-tracker: 디자인 시스템 분석 툴 패키지

### 프로젝트 구성 비교 (멀티레포 vs. 모노레포)

- 복잡성: 각 컴포넌트가 독립적으로 관리되지만 결국 전체 시스템의 일부분으로 동작해야 함
  - `✔︎` 모노레포 → 한 곳에서 복잡한 구조 관리 + 컴포넌트 간의 의존성과 업데이트 체계적으로 처리
- 버전 관리: 관심사 분리를 통해 필요 이상의 업데이트나 의존성을 받지 않도록 설계해야 함
  - `✔︎` 모노레포 → 워크스페이스 분리를 통한 코드 안정성 높이고 필요 시 일관성있게 패키지 버전 관리
- 협업: 협업하는 팀들이 한 공간에서 디자인에 중요한 사항과 개발 상황을 일관성 있게 다룰 수 있음
  - `✔︎` 모노레포 → 피드백 단계 단축 및 디자인 시스템의 변경 사항 빠르게 확인 가능
- 빌드 최적화 및 성능 개선: 캐싱이나 병렬 실행 등 다양한 매커니즘을 통해 빌드 최적화
  - `✔︎` 모노레포 → 컴포넌트 간 의존성 최적화 및 변경된 부분만 빌드하도록 설정

결론적으로 모노레포를 도입하면 일관성과 효율성을 유지할 수 있는 동시에 유지보수와 협업이 가능한 디자인 시스템을 만들 수 있다.

### 내부 워크스페이스

**내부 워크스페이스 장점**

- 공유 코드 재사용
- 일관성 유지
- 의존성 관리
- 문서화의 일관성

**내부 워크스페이스 구성**

- @ndive/design-stroybook: @ndive/design-components의 스토리북
- @ndive/tsconfig: tsconfig.json 공통 설정 관리
- @ndive/vite: vite.config.js 공통 설정 관리 → 번들러 설정을 함수로 감싸 내보내기
- shopping-web: 디자인 시스템 요소를 활용한 예시 웹 애플리케이션

## pnpm 워크스페이스 및 터보레포 구성하기

### 예제 프로젝트 환경

**기술 스택**

- typescript, vite, turborepo, pnpm + pnpm workspace

**지원 범위**

- ESModule만 지원
- 모던 브라우저 환경 지원 → `browserslist: [> 1%, not dead]`
- 폴리필이 필요하지 않은 코드로 작성

### pnpm 설정

```bash
$ mkdir ndive-design-system && cd ndive-design-system
$ pnpm init
```

**package.json 설정**

- 패키지명 스코프를 포함한 형태로 수정 및 private 활성화

```json
{
  "name": "@samseburn/design-system",
  "private": true
}
```

**pnpm-workspace.yaml 생성**

```yaml
packages:
  - './packages/*'
  - './apps/*'
  - './shared/*'
  - './examples/*'
```

### 터보레포 초기 설정

**터보레포 설치**

```bash
$ pnpm add -D turbo --workspace-root
```

**turbo.json 설정**

```json
{
  "$schema": "https://turbo.build/schema.json",
  "ui": "tui",
  "tasks": {}
}
```

**모든 패키지에서 사용될 공통 작업(task) 생성**

- build: 코드 빌드 → clean 작업 이후에 실행되어야 하므로 clean을 dependsOn에 추가
- clean: 빌드 초기화 → 실행할 때마나 반드시 실행해야 하므로 캐싱 비활성화
- dependsOn에 ^ 추가 → 워크스페이스에서 동일하게 설정된 모든 작업을 올바른 의존성 순서대로 처리

```json
"tasks": {
	"clean": { "dependsOn": ["^clean"], "cache": false },
	"build": { "dependsOn": ["clean", "^build"], "output": ["dist/**"] }
}
```

**최상단 package.json의 scripts에 build 스크립트 추가**

```json
"scripts": {
	"build": "turbo build"
},
```

## shared 공유 패키지 구현하기

> 내부 워크스페이스를 구성해 공개 패키지를 구현할 준비를 한다.

### tsconfig 패키지

> tsconfig.json 공통 설정 관리 패키지

**base.json 설정**

- react.json 및 node.json에서 확장되는 기본 구성 파일

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "composite": false,
    "declaration": true,
    "declarationMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "inlineSources": false,
    "isolatedModules": true,
    "moduleResolution": "Bundler",
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "preserveWatchOutput": true,
    "skipLibCheck": true,
    "strict": true,
    "allowJs": true,
    "noEmit": true
  }
}
```

**react.json 설정**

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "target": "ES6",
    "lib": ["dom", "dom.iterable", "esnext"],
    "module": "esnext",
    "jsx": "react-jsx"
  }
}
```

**node.json 설정**

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext"
  }
}
```

**files로 내보낼 파일들을 설정해 외부에서 구성 파일 불러올 수 있게 설정**

```json
"files": [ "base.json", "react.json", "node.json" ]
```

### vite 패키지

> Vite의 defineConfig() 래퍼 함수 패키지

**package.json 설정**

```json
{
  "name": "@samseburn/vite",
  "private": true,
  "type": "module",
  "peerDependencies": {
    "vite": "^5 || ^6"
  }
}
```

**defineConfig() 래퍼 함수**

- pkg: package.json의 정보 (exports, main 필드 필요)
- entry: 빌드할 진입점 경로
- target: Node.js 및 리액트 환경에 맞춰서 지원
- resolve: 타입스크립트 경로 별칭 설정 지원
- rollupOptions(optional): 패키지 외부에서 필요한 설정 추가 지원
- plugins(optional): 플러그인 추가 지원

모든 구현을 마쳤다면 최종적으로 vite 패키지의 package.json의 extports와 main 경로를 추가해 패키지의 모듈 경로를 설정한다.

```json
{
  "main": "dist/defineConfig.js",
  "types": "./dist/defineConfig.d.ts",
  "exports": {
    ".": {
      "import": "./dist/defineConfig.js",
      "types": "./dist/defineConfig.d.ts"
    }
  }
}
```

### 빌드

```bash
$ pnpm build

ESM ⚡️ Build success
DTS ⚡️ Build success
dist/defineConfig.js
dist/defineConfig.d.ts
Tasks: 2 successful, 2 total
```

- 이때, `shared/vite/tsconfig.json`을 명시적으로 추가 필요

<aside>

### 왜 tsconfig 하나로 모든 게 해결됐나?

- `tsup --dts`는 **TypeScript compiler를 그대로 사용**
- ESM 패키지(`type: module`) + subpath import (`vite`, `type-fest`)
- → **`moduleResolution: NodeNext` 없으면 무조건 깨짐**

즉,

> JS 번들러(esbuild)는 통과
>
> 타입 체커(TypeScript)는 실패
>
> → tsconfig 문제의 전형적인 패턴

이번에 정확히 그 케이스였어요.

</aside>

- 코드를 수정하지 않고 다시 build를 실행하면 캐싱된 해시를 작업에 재사용 → 새 build 작업 수행 x

```bash
┌─ @samseburn/vite#clean > cache bypass, force executing c118ed464752ec0f

> @samseburn/vite@ clean /Users/seokyoung/Documents/workspace/npm-deep-dive-example/samseburn-desig
n-system/shared/vite
> rm -rf dist
└─ @samseburn/vite#clean ──
┌─ @samseburn/vite#build > cache hit, replaying logs fffeb7e9fd8beeb5

> @samseburn/vite@ build /Users/seokyoung/Documents/workspace/npm-deep-dive-example/samseburn-desig
n-system/shared/vite
> tsup defineConfig.ts --dts --format esm --out-dir dist

CLI Building entry: defineConfig.ts
CLI Using tsconfig: tsconfig.json
CLI tsup v8.5.1
CLI Target: esnext
ESM Build start
ESM dist/defineConfig.js 2.22 KB
ESM ⚡️ Build success in 63ms
DTS Build start
DTS ⚡️ Build success in 1964ms
DTS dist/defineConfig.d.ts 1.90 KB
└─ @samseburn/vite#build ──

 Tasks:    2 successful, 2 total
Cached:    1 cached, 2 total
  Time:    1.44s
```

이처럼 turbo.json의 inputd이나 코드 수정 및 연관된 의존성이 수정되지 않은 이상 터보레포는 새로운 작업을 수행하지 않고 이전에 캐싱된 작업을 빠르게 재실행한다.

## design-tokens 패키지 구현

> 디자인 토큰을 생성하는 공개 패키지

- figma REST API: Bearer Token 인증 방식으로 접근 권한 설정 필요
  - `https://api.figma.com/v1/files/:filekey`
  - `https://api.figma.com/v1/files/:filekey/nodes?ids=1,2,3`
- tsconfig 공유 패키지를 참조해 타입스크립트 설정 → `"@samseburn/tsconfig": "workspace:*"`
- 디자인 토큰 추출하기 → 색상, 글꼴, 아이콘
  - setColor()
  - setType()
  - setIcon()

### 빌드하기

- vite 공유 패키지를 참조해 빌드 설정 → `"@samseburn/vite": "workspace:^"`

```bash
$ pnpm build

> @samseburn/design-system@1.0.0 build /Users/seokyoung/Documents/workspace/npm-deep-dive-example/samseburn-design-system
> turbo build

turbo 2.7.0

• Packages in scope: @samseburn/design-tokens, @samseburn/tsconfig, @samseburn/vite
• Running build in 3 packages
• Remote caching disabled
┌─ @samseburn/vite#clean > cache bypass, force executing ca766719481e69d3

> @samseburn/vite@ clean /Users/seokyoung/Documents/workspace/npm-deep-dive-example/samsebu
rn-design-system/shared/vite
> rm -rf dist
└─ @samseburn/vite#clean ──
┌─ @samseburn/design-tokens#clean > cache bypass, force executing 46f0857fb838befa

> @samseburn/design-tokens@0.0.1 clean /Users/seokyoung/Documents/workspace/npm-deep-dive-e
xample/samseburn-design-system/packages/design-tokens
> rm -rf dist
└─ @samseburn/design-tokens#clean ──
┌─ @samseburn/vite#build > cache miss, executing 9f0b522454febef4

> @samseburn/vite@ build /Users/seokyoung/Documents/workspace/npm-deep-dive-example/samsebu
rn-design-system/shared/vite
> tsup defineConfig.ts --dts --format esm --out-dir dist

CLI Building entry: defineConfig.ts
CLI Using tsconfig: tsconfig.json
CLI tsup v8.5.1
CLI Target: esnext
ESM Build start
ESM dist/defineConfig.js 2.22 KB
ESM ⚡️ Build success in 65ms
DTS Build start
DTS ⚡️ Build success in 2401ms
DTS dist/defineConfig.d.ts 1.90 KB
└─ @samseburn/vite#build ──
┌─ @samseburn/design-tokens#build > cache miss, executing 535abd19676751f7

> @samseburn/design-tokens@0.0.1 build /Users/seokyoung/Documents/workspace/npm-deep-dive-e
xample/samseburn-design-system/packages/design-tokens
> vite build --config vite.config.ts

You are using Node.js 21.7.1. Vite requires Node.js version 20.19+ or 22.12+. Please upgrad
e your Node.js version.
vite v7.3.0 building client environment for production...
✓ 6 modules transformed.

[vite:dts] Start generate declaration files...
dist/config/index.mjs   0.06 kB │ gzip: 0.08 kB
dist/index.mjs          0.17 kB │ gzip: 0.13 kB
dist/utils/color.mjs    0.25 kB │ gzip: 0.19 kB
dist/utils/utils.mjs    0.47 kB │ gzip: 0.30 kB
dist/utils/api.mjs      1.42 kB │ gzip: 0.53 kB
dist/getFoundation.mjs  2.34 kB │ gzip: 0.99 kB
[vite:dts] Start rollup declaration files...
Analysis will use the bundled TypeScript version 5.8.2
[vite:dts] Declaration files built in 5409ms.

✓ built in 5.55s
└─ @samseburn/design-tokens#build ──

 Tasks:    4 successful, 4 total
Cached:    0 cached, 4 total
  Time:    16.254s
```

## design-components 패키지 구현

- tsconfig 공유 패키지를 참조해 타입스크립트 설정 → `"@samseburn/tsconfig": "workspace:*"`
  - 이때 `@samseburn/tsconfig/react.json`을 확장
- design-token 패키지 활용
- 색상 데이터를 SCSS로 변환하는 fetchColor.js 작성
