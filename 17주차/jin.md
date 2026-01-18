# 9.2 패키지 개발에 도움이 되는 팁

## 9.2.1 선택이 아닌 필수, ESModule

9.2.1.1 브라우저 환경에 최적화된 ESModule

- CommonJS는 서버 런타임 환경 사용 목적으로 등장하여 브라우저 환경 동작에서는 부족한 점이 있음
    - 모듈을 동기적으로 불러오는 방식(import)은 브라우저 환경에서 네트워크 지연을 유발하고 병렬 다운로드를 불가능하게 해서 비효율적
    - 모듈을 동적으로 로드(다운로드)할 수 없어서 브라우저 성능 최적화 부족
    - 트리 셰이킹, 최적화 부족
    - 모듈래퍼가 클로저를 만듦으로 인해 브라우저에서 메모리 소비가 불필요하게 증가
    - 사용자 환경에서 require() 함수가 재정의되면 의도치 않은 동작 유발 가능
- ESModule은 CommonJS 문제를 해결
    - 모듈을 빌드 시점에 로드함으로 트리 셰이킹, 캐싱 등을 통해 최적화 가능
    - 비동기로 모듈을 블러오므로 네트워크 지연 감소, 로딩 속도 개선
    - import, export 키워드를 사용해 모듈을 정의할 수 있으며 재정의는 불가능

⇒ ESModule을 사용하면 코드 번들링 중 오버헤드를 줄일수도 있고, 서버/브라우저 두 환경에서 일관된 코드 문법 사용할 수 있으므로 앞으로 더 주목받을 것임.

9.2.1.2 이중 패키지의 어려움

- 정말 필요한 경우가 아니라면 단일 모듈 시스템이 좋음

9.2.1.3 ESModule 패키지에서의 올바른 트리 셰이킹 방법

- pakage.json의 exports 필드를 활용하여 사용자가 전체를 로드하지 않고 특정 모듈 경로만 명시하여 필요한 모듈만 가져오는 명확한 설계가 가능
  
  ```jsx
  {
    "name": "my-package",
    "version": "1.0.0",
    "main": "index.mjs",
    "exports": {
          "./utils/object":"./dist/utils/objects.js",
          "./utils/string":"./dist/utils/string.js"
    }
  }
  ```

배럴 파일(여러 모듈 내보내기를 단일 import 문을 사용하여 단일 모듈로 통합하는 방법)을 반드시 사용해야 하는 경우 최적화할 수 있는 방법

- export * from 대신 특정 모듈만 명시적으로 작성
- package.json에서 sideEffects 필드를 false로 설정하여 사용되지 않는 코드가 번들에 포함되지 않도록 함
- 나눌 수 있는 파일 경로는 최대한 나누어서 명시할 것
- 최적화 확인 방법
    - vite bundle visualizer
        
        ```jsx
        # In your vite project's root
        $ npx vite-bundle-visualizer
        # Then open stats.html in browser
        
        # Use specified vite config file
        $ npx vite-bundle-visualizer -c your.config.js
        ```
        
    
    <img width="2888" height="1648" alt="image" src="https://github.com/user-attachments/assets/9bce267d-d709-4087-a320-27107d6def73" />

    

## 9.2.2 package.json 올바르게 작성하기

9.2.2.1 main과 exports

- 모듈을 외부로 노출하지 않는 경우를 제외하고는 main 필드와 exports 필드 함께 설정 권장
- main : 패키지의 진입점
- exports : 모듈이 외부로 노출되는 파일 및 경로 정의 ( 패키지의 하위 경로 지정, 브라우저/서버 환경에 따른 진입 경로 다르게 지정 등 제어 가능)
- main 필드를 통해 고전 환경에서 사용 가능하도록 하고 exports의 세밀한 제어를 통해 최신 환경에 적합한 설정 제공

```jsx
{
  "name": "my-package",
  "version": "1.0.0",
  "main": "index.mjs", // CJS 기본 진입점
  "exports": {
        "import": "./index.mjs", // ESM 진입점
        "require": "./index.cjs", // CJS 진입점
        "./submodule":"./dist/submodule.js"
  }
}
```

- 순수 ESM 패키지의 경우 main 필드를 정의하지 않아도 되는 조건
    - type을 module 로 ESM 패키지임을 명시
    - Node.js 12 버전 이상, webpack@5, rollup@1.20.0 이상

9.2.2.2 packageManager와 engines

프로젝트에서 사용해야하는 패키지 관리 도구와 Node.js 환경을 명시적으로 설정하는데 사용되어 일관성을 보장하여 협업에 도움을 줌

packageManager

- 사용 중인 패키지 관리 도구와 버전을 명시하여 프로젝트 복제, 설치 시 동일한 버전을 설치하도록 알림
    - 코어팩과 같이 사용하면 자동 설치, 실행 가능

engines 

- Node.js 및 기타 도구 버전을 명시
    - 패키지 매니저가 engines 조건을 엄격히 검사

9.2.2.3 추가적으로 고려해야 할 필드

package.json

- type : 모듈 시스템
- files : 배포되는 파일 제한 (포함할 파일만 명시)
- keywords : 검색어
- repository : 패키지 저장소로 이동
- $schema : 오타 방지

### 9.2.3 올바른 트랜스파일과 폴리필 적용하기

- 폴리필 : 최신 웹 기술을 지원하지 않는 브라우저에서 최신 기능을 사용할 수 있도록 해주는 코드
- 트랜스파일 : 최신 JS 코드를 구 브라우저에서도 동작하도록 변환

9.2.3.1 browserlist 설정

- 전역 네임스페이스 오염 방지를 위해 필요한 폴리필만 불러오기

9.2.3.2 모든 패키지가 트랜스파일과 폴리필이 필요한 것은 아니다.

9.2.3.2.1 모던 브라우저의 네이티브 지원

- 현재 브라우저와 최신 Node.js는 이미 ECMAScript의 기능을 기본적으로 지원하기 때문에 트랜스파일, 폴리필이 오히려 코드를 복잡하게 만들어서 성능 저하를 초래할 수 있음.

9.2.3.2.2 사용자 애플리케이션과의 충돌

- 폴리필은 사용자가 직접 설정해야할 영역이다. 폴리필을 추가해놓으면 사용자 설정에 따라 중복되거나 충돌이 발생하며 오히려 최적화에 악영향을 미칠 수 있음

9.2.3.2.2.1 소수 브라우저를 위해 번들에 다수의 오버헤드를 야기함

- 목표 지원 환경 확인
- 패키지 크기와 성능 최적화
- 기능 테스트

⇒ 패키지 목적을 정확하게 파악해서 필요한 것만 지원

## 9.2.4 dependencies 는 신중하게 추가하라

9.2.4.1 정말 필요한 패키지만 포함하라

크기에 영향을 주기 때문에 실행 환경에서 필요한 패키지만을 포함하고 아래 기준을 고려하여 최적화

- 핵심 기능에 필요한 의존성만 포함
- 간단한 기능은 직접 구현
- 적합한 범주의 패키지 구분 (devdepndencies와 구분하기)

9.2.4.2 추가될 패키지의 모듈 시스템을 검토하라

- ESM, CJS을 둘 다 지원하면 exports 필드에서 require와 inport 진입점 검토
- 타입스크립트의 경우 type 지원하는지 확인
- 호환성이 중요한 경우 exports에서 default 필드로 기존 환경과의 호환성 보장하는지 확인

9.2.4.3 안전한 패키지인지 확인

- 패키지 출처 확인
- 보안 취약점 점검
- 하위 의존성 보안 점검
    - npm ls, pnpm list
- 권한 최소화
- 최신 버전 패키지 사용

9.2.4.4 최적화된 패키지를 선택하라

- 의존성 체인 확인 (npm ls, pnpm list)
- 모듈 최적화 확인
    - ESM
    - 트리 셰이킹 확인 (sideEffects, exports 경로)
- 경량화된 패키지인지 확인
    - https://bundlephobia.com/
    
    <img width="1110" height="1646" alt="image" src="https://github.com/user-attachments/assets/b16be8d8-1b18-4a6a-87c8-6420ed22deb4" />

    
    <img width="1058" height="1348" alt="image" src="https://github.com/user-attachments/assets/de795037-9426-475b-9347-0565a6a796b2" />

    

## 9.2.5 코드에 신뢰를 주는 테스트 코드와 벤치마크 테스트

패키지 신뢰성과 성능을 증명하고 개선할 방향을 제시할 수 있는 과정

잘 작성된 테스트는 사용자에게 신뢰를 주고 잠재적 기여자들에게 코드 베이스 이해에 도움을 줌

다양한 상황, 엣지 케이스를 다룰 수 있도록 설계하는 것이 중요 (사용자, 기여자, 장기 운영에 큰 이점)

- 단위 테스트
- 통합 테스트
- 성능 테스트 및 벤치 마크
    - Tinybench, Vitest
- 종단 간 테스트
    - Cypress, Playwright, Puppeteer

9.2.6 올바른 문서 작성법

- README.md : 목적, 사용법(이름, 용도, 예시 등), 기여/문제 보고, 라이선스, 배지 (패키지 정보, 현재 상태,기여 독려, 커뮤니티)
- CONTRIBUTING.md : 기여 방법, 행동 강령, 기준, 책임
- CHANGELOG.md : 변경 내역
    - changesets , Release Please 도구 활용 : 자동 업데이트 및 배포
