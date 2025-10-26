# 4.4 Node.js는 어떻게 node_modules에서 패키지를 찾아갈까?

## 4.4.1 모듈 해석 알고리즘

- Node.js가 모듈을 로드하고 의존성을 해결하는 일련의 과정
- 불러오는 모듈 형식에 따라 모듈 경로를 결정하는 규칙과 단계를 정의
- CommonJS, ESModule 둘다 기본적인 원칙과 흐름은 유사하며 세부적인 차이가 있음

### 4.4.1.1 모듈 지정자

- require() 함수, import 문에서 특정 모듈을 식별하는 문자열
- 모듈 지정자 유형
    - 상대 경로 지정자
        - 현재 파일의 위치를 기준으로 하는 경로
        - `‘./’, ‘../’` 로 시작
    - 모듈 이름 지정자
        - 파일 시스템 경로가 아닌 모듈 이름만을 사용해서 참조하는 방식
        - 주로 node_modules 폴더 내 패키지나 내장 모듈 참조 시 사용
        - 패키지의 package.json 파일에 있는 `main` 이나 `exports` 필드로 정의된 주진입점 모듈을 가리킴
        - ESModule 해석 알고리즘에 따라 `exports` 필드가 없는 패키지는 파일 확장자를 포함해서 불러야함
    - 절대 경로 지정자
        - 파일 시스템의 최상위에서 시작하는 경로
        - `'/'` 로 시작하며 특정 위치의 파일이나 디렉터리를 가리킬 때 사용
- **모듈 이름 지정자는 Node.js의 모듈 해석 알고리즘에 의해 처리**
- 다른 지정자는 표준저인 URL 경로 해석 방식으로 처리

### 4.4.1.2 표준 상대 경로 URL 해석 문법

- 지정된 기본 URL을 기준으로 상대 경로를 절대 경로 URL로 변환하는 규칙, 방법을 정의
- WHATWG에서 정의한 URL 표준을 따르며, 이 표준은 URL의 형식, 구문 뿐 아니라 상대 URL을 절대 URL로 변환하는 방식도 설명하고 있음

### 4.4.1.3 하위 경로 불러오기와 하위 경로 내보내기

- Node.js의 하위 경로 불러오기와 하위 경로 내보내기는 package.json 파일의 `exports` 필드에 정의돼 있음 → ESModule 파일 구조와 접근 방식을 더 유연하고 관리하기 쉽게 만듦
- **하위 경로 불러오기**
    - 패키지 내부에서 가져올 모듈을 명확히 정의하는 데 사용
    - package.json 파일의 `imports` 필드를 통해 설정
    - 관습적으로 `#` 을 접두어로 사용해 해당 경로가 하위 경로로 불러온 모듈임을 나타냄
    - 예시 : #helper를 my-package의 import 필드에 정의
        
        ```jsx
        // package.json 
        {
        	"name": "my-package",
        	"imports": {
        		"#helper": "./lib/helper.js"
        	}
        }
        
        // 프로젝트 내부에서 import 문으로 가져와서 사용 가능 
        import helper from 'my-package/#helper'
        ```
        
    - 외부 패키지를 별칭처럼 나타낼 수도 있음
        
        ```jsx
        // package.json 
        {
        	"name": "my-package",
        	"imports": {
        		"#dep": "dep-node-native"
        	},
        	"dependencies": {
        		"dep-node-native": "^1.0.0"
        	}
        }
        
        // 아래 코드는 dep-node-native를 가져온 것과 동일
        import dep from 'my-package/#dep'
        ```
        
        - 하위 경로 불러오기에서 외부 패키지를 사용하는 기능의 존재 이유(코드상에서 외부 패키지를 바로 불러오면 되는데..?)
            - 하위 경로 불러오기는 **`imports` 와 `exports` 필드를 통해 개발 환경에 따라 적합한 파일을 조건부로 불러와야할 때** 유용하게 쓰임
            - e.g. dep-node-native 패키지가 Node.js 환경에서만 동작하고 브라우저에서 사용할 수 없는 경우 dep-node-native의 폴리필을 작성해 Node.js가 아닌 환경에서도 해당 파일을 불러와 사용할 수 있음
                
                ```jsx
                // node와 default 조건을 나눠서 Node.js 환경에서는 dep-node-native 패키지를, 그 외 환경에선 dep-polyfill.js을 불러오도록 처리
                
                // package.json 
                {
                	"imports": {
                		"#dep": {
                			"node": "dep-node-native",
                			"default": "./dep-polyfill.js"
                		}
                	},
                	"dependencies": {
                		"dep-node-native": "^1.0.0"
                	}
                }
                
                ```
                
- **하위 경로 내보내기**
    - 외부에 패키지를 내보낼 경로를 정의하는 방식
    - package.json 파일의 `exports` 필드를 통해 설정
    - 패키지 내부 구조가 변경되어도 외부 API를 유지하고 특정 파일만 외부에 노출 시켜 내부 구현을 숨길 수 있음
    - **참조할 파일 경로는 `"./"` 로 시작하는 상대 경로로 작성해야함**
    - 예시 : my-package에서 index.js와 style.css만 외부에서 사용하도록 적용
        
        ```jsx
        {
        	"name": "my-package",
        	"exports": {
        		".": "./src/index.js" // 별도의 하위 경로없이 가져올 때 사용
        		"./style.css": "./src.index.css" 
        		// 하위 경로를 붙여 정의 가능 -> 외부에서는 my-package/style.css처럼 참조 가능
        	}
        }
        ```
        
    - 하위 경로 내보내기도 환경 조건에 따라 파일 경로를 다르게 설정할 수 있고 이를 통해 CommonJS와 ESModule을 모두 지원하는 패키지를 만들 수 있음(자세한 내용은 4.5절에서 설명)
- 참고-하위 경로 내보내기에서 확장자를 명시해야 할까?
    - 정답은 없지만, 패키지 작성자는 확장자를 명시하는 방법/생략하는 방식 중 상황에 맞는 방식을 선택하여 일관성을 유지하는 게 좋음
    - .css 파일은 확장자를 명시하고, js 파일은 확장자를 생략하는 것도 하나의 전략

## 4.4.2 모듈 이름 지정자로 모듈을 로드하는 방법

### 4.4.2.1 CommonJS에서 모듈을 찾는 방법

- 그림 4.10 CommonJS 모듈 해석 알고리즘(Node.js에서 CommonJS가 외부 패키지 모듈을 해석해서 로드하는 과정)
    
### 4.4.2.1.1 전체 흐름

- 코어 모듈인 경우: X가 Node.js 내장 모듈인 경우 해당 모듈을 즉시 반환
- 절대 경로로 시작하는 경우: 절대 경로 지정자이므로 파일 시스템 최상위를 탐사 시작 지점으로 설정
- 상대 경로로 시작하는 경우: 파일을 찾는 LOAD_AS_FILE과 폴더를 찾는 LOAD_AS_DIRECTORY를 순서대로 수행. 경로를 찾지못할 경우 `not found` 에러를 던짐
- ‘#’으로 시작할 경우: package.json의 `imports` 필드에서 정의한 상대경로를 참조하므로, LOAD_PACKAGE_IMPORTS로 “imports” 필드를 분석해 모듈을 찾음
- LOAD_PACKAGE_SELF: 패키지 스코프 내에서 특정 모듈을 로드하는 과정. 주로 package.json의 exports 필드를 통해 모듈을 찾고 로드함
- LOAD_NODE_MODULES: node_modules 폴더에서 모듈을 찾는 과정. 현재 디렉터리에서부터 상위 디렉터리로 올라가며 node_modules 폴더를 탐색. npm 패키지 등 외부 패키지 로드 시 사용됨

### 4.4.2.1.2 기본 과정

- 이 과정은 독립적으로 실행되기도 하지만 LOAD_PACKAGE_SELF나 LOAD_NODE_MODULES 같은 과정에 포함되어 실행되기도 함
- LOAD_AS_FILE
    - 특정 모듈 지정자가 파일로서 존재하는지 확인 후 모듈을 로드하는 과정
    - 모듈이 실제 파일로 존재하는지, 파일이 올바른 확장자를 가지고 있는지 확인 후 파일을 로드
- LOAD_AS_DIRETORY
    - 특정 경로가 디렉터리일 경우 해당 디렉터리를 모듈로 로드하는 과정
    - 주로 폴더 경로를 나타내는 지정자를 해석할 때 사용됨
    - 해당 폴더가 존재하는지 확인 → 경로가 폴더인지 확인 → 내부에 package.json이 존재하는지 확인 → package.json이 있다면 main 필드 확인
        - 만약 main 필드가 없거나 main의 진입점에 해당하는 파일이 없으면 LOAD_INDEX를 수행
- LOAD_INDEX
    - Node.js는 기본 파일로 index.js, index.json, index.node를 순서대로 찾아서 로드함
    - index.js는 폴더 경로에 package.json이 있으면 type 필드를 고려하여 명시된 모듈 시스템으로 index.js를 로드함
- RESOLVE_ESM_MATCH
    - ESModule의 해석과 관련된 매칭 과정을 수행해 특정 조건에 맞는 모듈을 찾고 로드함

### 4.4.2.1.3 LOAD_PACKAGE_SELF

- 패키지 스코프 내에서 특정 모듈을 로드하는 과정
- 일반적으로 프로젝트 내부의 경로를 exports로 연결해서 사용하는 패턴은 자주 사용되지 않으므로 LOAD_PACKAGE_SELF 또한 자주 쓰이지는 않음

### 4.4.2.1.4 LOAD_NODE_MODULES

- node_modules에 설치된 외부 모듈을 가장 가까운 node_modules 폴더부터 시작해 부모 node_modules 로 거슬러 올라가며 탐색하는 과정
- npm 패키지와 같은 외부 모듈 로드 시 주로 사용되며, 개발자들이 모듈을 로드할 때 가장 많이 활용하는 과정
- LOAD_NODE_MODULES 과정
    
    ![모든 node_modules 경로 가져오기 (NODE_MODULES_PATHS) (1).png](attachment:248e54ac-605e-46b3-959a-06b8464fab52:모든_node_modules_경로_가져오기_(NODE_MODULES_PATHS)_(1).png)
    
- NODE_MODULES_PATH
    - 인자로 받은 START 경로를 기준으로 node_modules 폴더의 탐색 경로를 생성하는 과정

### 4.4.2.1.5 정리

- CommonJS의 모듈 해석 알고리즘
    - 내장 모듈 확인
        - require() 함수가 요청한 모듈이 Node.js의 내장 모듈인지 확인
    - 파일 해석
        - 명시된 파일 확장자가 있는 경우 해당 파일을 로드, 확장자가 없는 경우에는 .js, .json, .node 순으로 파일 탐색
    - 디렉터리 해석
        - 모듈 지정자가 디렉터리일 경우 해당 디렉터리의 package.json 파일의 main 필드 확인 or index.js 같은 기본 파일을 순차적으로 탐색 후 로드
    - node_modules 폴더 탐색
        - 현재 디렉터리 → 상위 디렉터리 순으로 이동하며 node_modules 폴더를 찾아 모듈을 탐색
- CommonJS의 모듈 로드 방식의 단점
    - 모듈 경로를 상대적으로 해석하므로 node_modules 폴더를 거슬러 올라가며 탐색할 때 오버헤드가 발생할 수 있음
    - 모듈을 동기적으로 로드하므로 모듈 로딩이 블로킹되어 성능 저하를 초래할 수 있음

### 4.4.2.2 ESModule에서 모듈을 찾는 방법

- ESModule의 모듈 해석 알고리즘
    

### 4.4.2.2.1 모듈 리졸버와 모듈 로더

- ESModule 모듈 리졸버의 특징
    - 파일 URL 기반 해석
        - 파일 시스템 경로를 직접 사용하지 않고 파일 URL을 기반으로 경로 해석
    - 기본 확장자가 없음
        - ESModule은 파일 확장자를 생략할 수 없음(URL 기반으로 동작하기 때문에 확장자를 포함한 경로를 명시해야함)
    - 폴더의 main이 없음
        - 폴더를 모듈로 지정하려면 반드시 폴더 내에서 파일을 명시해야함
        - CommonJS는 디렉터리 내에 index.js 같은 기본 파일을 자동으로 로드하지만 ESModule은 지원하지 않음
    - node_modules 폴더 기반 패키지 해석
        - ESModule도 node_modules 폴더를 통해 패키지를 해석할 수 있어 CommonJS와의 호환성을 유지함
- ESModule  기본 모듈 로더의 특징
    - 내장 모듈 로딩 지원
        - node:URL 스킴을 통해 Node.js의 코어모듈 로드 가능
    - 인라인 모듈 로딩 지원
        - data: URL 스킴을 통해 Base64 같은 인라인 데이터 모듈 로드 가능
    - 파일 모듈 로딩 지원
        - file: URL 스킴을 통해 파일 시스템에서 모듈 로드 가능
    - 다른 URL 프로토콜 실패
        - http: 나 https: 등 다른 URL 프로토콜을 통한 모듈 로딩을 지원하지 않음
    - 알 수 없는 확장자 로딩 실패
        - .cjs, .mjs, .js 외 확장자는 지원되지 않음
- CommonJS는 모듈을 동적으로 로드하므로 모듈 탐색과 로딩이 함께 이뤄지는 반면, **ESModule은 실행 전에 모듈을 정적으로 파싱 후 인스턴스화 하기 때문에 모듈 지정자의 절대 경로를 찾는 과정과 해당 경로로 모듈을 로드하는 과정이 분리**돼 있음

### 4.4.2.2.2 ESM_RESOLVE

- 특정 모듈을 로드하기 위해 모듈의 경로(지정자)를 찾는 과정
- 모듈의 경로를 해석할 뿐 실제로 모듈을 로드하진 않음

### 4.4.2.2.3 resolved 결정하기

- resolved에는 import문의 from 경로가 실제 모듈의 경로(파일 시스템부터 시작하는 절대 경로)로 변환되어 할당됨
- resolved는 모듈의 경로를 가리키는 변수로 아래 조건마다 다른 코드를 실행해 결정됨
    - 먼저 모듈 지정자가 이미 http://, file://, data:// 같은 절대 경로라면 해당 URL을 즉시 반환
    - 모듈 지정자가 상대 경로라면, 경로를 부모 URL과 결합해 절대 경로를 변환
    - 모듈 지정자가 ‘#’으로 시작하면 하위 경로 불러오기에 해당하므로 PACKAGE_IMPORTS_RESOLVE 과정 수행
    - 위 세 조건에 해당하지 않는 경우, 이 지정자는 상대나 절대 경로가 아닌 모듈 이름 지정자에 해당하므로 node_modules 폴더에서 해당 모듈을 찾기 위해 PACKAGE_RESOLVE 과정 수행

### 4.4.2.2.4 format 결정하기

- 모듈의 위치를 나타내는 resolved가 결정된 후 해당 모듈의 파일 형식을 결정해야 함
    - 단 아래 조건에 해당되지 않아야 resolved가 유효한 조건으로 간주될 수 있음
        - resolved에 ‘/’ 또는 ‘\\’ 문자가 포함된 경우
        - resolved가 가리키는 경로가 파일이 아닌 폴더일 경우
        - resolved 경로에 파일이 존재하지 않는 경우
- format은 ESM_FILE_FORMAT 과정을 통해 결정됨
- format을 결정하는 조건
    - .mjs 확장자로 끝나는 파일 : “module”
    - .cjs 확장자로 끝나는 파일 : “commonjs”
    - .json 확장자로 끝나는 파일 : “json”
    - package.json의 type 필드가 module로 설정된 경우 : “module”
    - 그 외의 경우 : undefined
- resolved와 format이 결정되면 다음 단계에서 resolved가 가리키는 경로에서 format에 맞는 형식으로 모듈을 로드하게 됨

### 4.4.2.2.5 PACKAGE_RESOLVE

- CommonJS의 LOAD_NODE_MODULES와 비슷한 단계로 부모 폴더부터 시작해 상위 폴더의 node_modules까지 검색하는 과정을 설명
- package.json의 “exports” 필드를 해석하기 위해 모듈 지정자를 packageName(패키지이름)과 packageSubpath(패키지의 하위 경로)로 분리하고 전역 node_modules가 위치한 경로까지 거슬러 올라가면서 node_modules에 패키지를 찾아 해당 패키지의 “exports” 필드에 명시된 진입점을 찾는 과정

### 4.4.2.2.6 정리

- ESModule의 모듈 해석 알고리즘 특징
    - 파일 확장자 확인
        - ESModule은 .js, .mjs 확장자가 있는 파일을 모듈로 인식함
    - 상대 및 절대 URL 해석
        - 상대 경로와 절대 경로 URL을 해석해 모듈의 실제 경로를 결정
    - node_modules 폴더 내 package.json 확인
        - node_modules 폴더에 있는 package.json 파일의 “exports” 필드 확인 후 모듈 경로를 결정
    - 기본 파일 인식 없음
        - 기본 파일인 index.js 같은 파일을 자동으로 로드하지 않음 → 폴더를 모듈로 지정하려면 폴더 내 특정 파일을 명시해야 함
    - 노드 모듈에서 모듈 로드
        - node_modules 디렉터리에서 모듈을 찾고 로드함
- ESM_RESOLVE 단계 요약
    - 지정자 해석
        - 모듈 경로를 부모 URL 기준으로 해석
    - 형식 결정
        - 파일 확장자나 package.json의 type 필드를 통해 모듈 형식을 결정함
    - 로드 단계 검증
        - 로드 단계에서 최종적으로 경로와 형식을 검증해 모듈을 로드함
- **CommonJS와 ESModule의 주요 차이점**
    - CommonJS는 모듈을 로드하면서 동시에 경로를 찾지만, ESModule은 경로를 먼저 찾고 나서 모듈을 로드함
        - ESModule의 동작방식은 전체 의존서에 대한 정적 분석과 트리셰이킹을 용이하게 해주므로 더 작은 번들 크기와 빠른 로드 시간을 제공
    - CommonJS는 파일시스템을 기반으로 하위 경로를 찾지만, ESModule는 우선 package.json의 “exports” 필드로 경로를 해석 후 이 정보를 통해 URL 기반으로 경로를 파악함
        - 그러므로 ESModule에서는 파일 확장자를 반드시 포함해야 함

# 4.5 CommonJS와 ESModule, 무엇이 정답일까?

## 4.5.1 오픈소스 패키지가 CommonJS와 ESModule을 동시에 지원하는 이유

- 자바스크립트 생태계의 다양한 환경과 요구사항을 충족하려면 두 모듈 시스템을 동시에 지원하는 것이 합리적일 수 있음
- 코드의 재사용성
    - 프로젝트가 기존에 CommonJS로 개발됐더라도 ESModule로 전환하거나 클라이언트와 서버 간에 코드를 재사용하려면 별도의 수정 없이 그대로 사용할 수 있어야함 → 두 모듈 시스템을 동시 지원하면 이런 변화에 더 쉽게 대응할 수 있음
- 다양한 개발 환경 지원
    - 일부 프로젝트는 Node.js 서버에서만 실행되고, 다른 프로젝트는 브라우저에서만 동작하거나, 웹팩 또는 Parcel과 같은 번들러를 사용해 빌드됨 → 다양한 환경을 지원함으로써 더 많은 사용자와 개발환경에서 해당 라이브러리를 활용할 수 있음

## 4.5.2 CommonJS와 ESModule을 동시에 지원하는 듀얼 패키지 개발하기

### 4.5.2.1 “main”과 “module” 필드

- Node.js에서 package.json의 “main” 필드로 선언한 파일은 기본적으로  CommonJS로 해석됨
    - “main” 필드만 존재하는 패키지는 require() 함수나 ESModule의 import로 불러올 수 있음(CommonJS 파일을 이름으로 내보낸 경우가 아니라면 직접 이름으로 import 할 순 없음)
- ESModule에서는 “main” 필드와는 별도로 “module” 필드를 추가해서 ESModule에서 로드할 수 있는 파일 경로를 명시할 수 있음
    - module 필드는 npm 공식문서에는 설명이 없지만 Node.js에서 CommonJS와 ESModule 간 상호 운용성을 위해 제안된 필드임
    - 현재는 Node.js 자체에서 ESModule 진입점을 지원하므로 “module” 필드를 Node.js에서도 사용할 수 있게 되었음(번들러와 Node.js 환경 모두에서 사용 가능)

### 4.5.2.2 조건부 내보내기

- Node.js 13.2.0 버전부터  “exports” 필드가 지원되어 “module” 필드 대신 사용해 듀얼 패키지를 개발하는 것이 효율적
- “exports” 필드는 “main”과 마찬가지로 패키지의 진입점을 지정할 수 있지만 더 상세하게 특정 조건에 따라 내보내는 경로를 직접 설정할 수 있음

## 4.5.3 ESModule 패키지 개발하기

- 듀얼 패키지 개발의 어려움과 이를 해결하기 위해 ESModule에만 집중하는 패키지 개발 방식이 주목받게 되었음

### 4.5.3.1 이중 패키지의 위험성

- 이중 패키지의 위험성
    - CommonJS와 ESModule을 모두 지원하는 패키지를 사용할 때 발생할 수 있는 문제 중 하나
    - CommonJS와 ESModule의 모듈 내보내기 방식 차이로 인해 CommonJS로 작성되어 ESModule 환경에서 import한 객체와 CommonJS로 가져온 객체는 서로 다른 객체로 인식됨

### 4.5.3.1.1 이중 패키지 위험성을 피하면서 이중 패키지를 제공하는 방법

- 패키지의 CommonJS 코드는 Node.js 환경에서만 사용하고, 별도의 ESModule 코드는 브라우저 등 다른 환경에서 사용되게 분리 → 이렇게하면 CommonJS 코드는 모든 Node.js 버전에서 사용 가능하며 import로 참조할 수 있음
    - 하지만 이 방식에서는 ESModule 구문이 제공하는 정적 분석의 장점을 활용할 수 없음
- 패키지를 CommonJS에서 완전히 ESModule로 전환하는 것
    - 하지만 이렇게 작성된 최신 패키지는 ESModule을 지원하는 Node.js 버전에서만 사용 가능
- 위 두 방식은 장단점이 있으므로 하나를 택하기 어려움
- Node.js는 이중 패키지 문제를 피하면서 두 모듈 시스템을 안전하게 제공할 수 있는 두 가지 접근 방식을 제안했음
    - ESModule 래퍼 활용
        - 패키지를 CommonJS로 작성하거나 ESModule 소스를 CommonJS로 변환 후 이름으로 내보내기를 정의하는 ESModule 래퍼파일을 추가
        - ESModule 래퍼 파일은 CommonJS 파일을 감싸서 CommonJS의 exports 객체를 이름으로 내보내는 역할을 함
    - 상태를 격리하기
        - CommonJS와 ESModule의 진입점을 분리해서 각각 개별적인 모듈로 정의
        - CommonJS와 ESModule의 두 가지 버전이 동시에 애플리케이션에서 사용될 수 있으므로 패키지의 상태가 격리되거나 패키지의 상태를 저장하지 않아야 함

### 4.5.3.2 거대한 퍀지ㅣ 사이즈

- 하나의 패키지에서 CommonJS와 ESModule 두 모듈 시스템을 동시에 지원하려면 동이랗ㄴ 코드를 두 모듈 시스템 버전으로 각각 배포해야하므로 번들 크기가 두배가 됨
- 이러한 이중 패키지로 인해 node_modules의 용량이 커지면 CI/CD 파이프라인과 서버리스 환경에서 상당한 부담으로 이어질 수 있음

### 4.5.3.3 ESModule로 전환하기까지의 험난한 여정

- 위에서 언급한 이중 패키지의 문제점으로 인해 최근에는 ESModule만을 지원하는 패키지를 배포하거나 기존 코드를 순수하게 ESModule로 전환하는 사례가 늘고 있음

### 4.5.3.3.1 순수한 ESModule 패키지를 개발하기 위한 준비

- 순수한 ESModule 패키지를 만들기 위해 기본적으로 설정해야 할 사항
    - 런타임이 코드를 ESModule로 해석하도록 설정해야 함
        - Node.js를 사용한다면 package.json의 type 필드를 “module”로 설정하고 exports 맵을 사용한다면 ESModule 빌드 결과물의 경로를 지정해야 함. main 필드를 사용하는 경우에도 ESModule의 경로로 수정해야함
    - 기존에 CommonJS로 작성된 코드가 있다면, 파일 내 CommonJS에서만 사용할 수 있는 코드를 모두 ESModule 코드로 대체해야 함
    - 사용 중인 번들러가 ESModule로 빌드되도록 설정을 수정하고, 프레임워크가 ESModule을 해석할 수 있도록 설정을 조정해야 함
- 위는 기본 설정이며 아래 추가 요소들로 인해 ESModule 관련 설정이 더 필요할 수 있음

### 4.5.3.3.2 타입스크립트 설정

- 타입스크립트는 기본적으로 CommonJS 모듈 형식을 사용하므로 ESModule 모듈 시스템을 사용하려면 tsconfig.json 파일엣 명시적으로 설정을 지정해야 함
- ESModule와 관련된 주요 필드 설정
    
    ```smalltalk
    {
    	"compilerOptions": {
    		"module": "es6",
    		"target": "es6",
    		"moduleResolution": "node16",
    		"esModuleInterop": true,
    		"allowSyntheticDefaultImports": true
    	}
    }
    ```
    
    - module과 target
        - target : 컴파일된 JS의 버전을 지정하며 타입스크립트 컴파일러가 어떤 ECMAScript 버전을 목표로 할지 결정함
        - module : 타입스크립트가 사용하는 모듈 형식을 지정
        - 위 두 설정은 서로 호환되어야함
    - moduleResolution
        - 모듈 해석 방식을 정의
        - node16은 최신 Node.js 모듈 해석 방식을 뜻함
    - esModuleInterop
        - CommonJS와의 상호운용성을 개선하기 위해 사용됨
        - import 구문으로 가져온 모듈은 require()로 가져온 것과 동일하게 동작하지 않기 때문에 이 옵션을 활성화해서 타입스크립트가 이러한 차이를 보완하는 헬퍼 함수를 통해 CommonJS 모듈을 ESModule과 유사하게 사용할 수 있음
    - allowSyntheticDefaultImports
        - CommonJS와의 상호운용성을 개선하기 위해 사용됨
        - 타입스크립트가 CommonJS 모듈에서 기본 내보내기를 허용하게 함
        - 이 설정을 활성화하면 타입스크립트가 일반적으로 기본 내보내기로 간주되는 구문을 자동으로 해석해줌

## 4.5.4 CommonJS와 ESModule, 무엇이 정답일까?

### 4.5.4.1 주장 1: CommonJS가 자바스크립트를 해치고 있다

- CommonJS는 서버 환경에서 자바스크립트 프로젝트의 필요에 맞춰 개발됐으나 여러 한계로 인해 ESModule이 등장했음
- Node.js는 초기부터 CommonJS로 작성돼 왔고, 기존의 CommonJS 인터페이스를 ESModule로 완전히 전환하기에는 어려움이 존재 → Node.js는 CommonJS 진입점을 유지하면서 ESModule의 진입점도 추가해 두 모듈 시스템을 동시에 지원하기로 결정
- ESModule과의 상호운용성 문제는 패키지 작성자의 몫으로 남게되었음
- 또한 클라우드 컴퓨팅이 에지 컴퓨팅, 서버리스 컴퓨팅 등의 방향으로 발전하며 서버 환경이 변화하고 있기 때문에 CommonJS가 더 이상 최선의 선택이 아닐 수 있음
- 결론 → 네트워크 및 자원 효율성을 높이면서 브라우저와 서버에서 모두 호환되는 코드를 작성할 수 있는 ESModule이 현재의 개발 환경과 요구에 더 부합

### 4.5.4.2 주장 2: CommonJs는 사라지지 않을 것이다

- CommonJS는 ESModule보다 모듈을 로드하고 시작하는 속도가 더 빠르다는 장점이 있음
    - 규모가 큰 애플리케이션에서 ESModule은 상대적으로 느리게 시작하지만 CommonJS는 빠름
- 점진적 로딩
    - CommonJS에서는 조건에 따라 파일을 로드하거나, 동적으로 생성된 경로에 지정자를 require()함수의 인자로 사용해 조건부 로드가 가능
    - ESModule로 동적 import() 함수를 제공하지만, 결국 이는 CommonJS의 동적 접근 방식이 여전히 유용하다는 것을 의미함
- 수많은 CommonJS 패키지와 프로젝트의 존재
    - 현재 수백만 개의 모듈이 npm에 게시돼 있으며, 대부분 CommonJS를 사용하고 있음

### 4.5.4.3 과연 무엇이 정답일까?

- 패키지가 ESModule만을 지원할지, ESModule과 CommonJS를 함께 지원할지는 전적으로 패키지 작성자의 선택에 달려있음
- 패키지 작성자는 패키지를 사용하려는 대상과 환경을 고려해 모듈 시스템을 결정할 수 있음
- Node.js 22버전에서도 여전히 CommonJS와 ESModule 간의 상호운용성을 강화하는 기능을 추가하고 있음 → 서버와 클라이언트 환경 모두에서 ESModule로 작성된 코드를 별도의 빌드 과정 없이 실행할 수 있는 환경을 마련하려는 노력으로 볼 수 있음.
- 이러한 Node.js의 노력은 ESModule로의 전환이 계속될 것임을 시사며, 기존 CommonJS 코드도 최소한의 수정으로 ESModule에서 실행 가능하도록 변화할 것으로 예상됨
