# 4장. CommonJS와 ESModule

- CommonJS
    - 주로 서버 환경에서 널리 사용됨
    - Node.js의 모듈 시스템으로 채택되어 초기 서버 사이드 자바스크립트 개발에 큰 영향을 미침
- ESModule
    - 자바스크립트의 표준 모듈 시스템
    - 브라우저 환경에서의 모듈화를 지원
    - 최신 자바스크립트 프로젝트에서 널리 채택되고 있음

# 4.1 자바스크립트 모듈화의 역사

- 모듈화
    - 코드를 작은 단위로 분할, 추상하하여 유지보수성을 높이고 재사용성을 향상 시킴
    - 모듈화는 프로젝트의 규모가 커질 수록 필연적으로 모듈 시스템으로 확장됨

## 4.1.1 자바스크립트 모듈화의 배경

### 4.1.1.1 1997년: ECMAScript 표준 제정

- 자바스크립트가 개발된 1995년 이후 마이크로소프트의 JScript 등 브라우저마다 다른 스크립트 언어가 등장하면서 호환성 문제가 대두되었음
- 이를 해결하기 위해 자바스크립트의 표준인 ECMAScript가 제정됨

### 4.1.1.2 1999년: Ajax의 등장

- 초창기 자바스크립트는 서버에서 전달된 HTML에 동적인 요소를 추가하는 보조적인 역할
- 1999년 Ajax가 등장하면서 브라우저와 서버가 비동기적으로 데이터를 교환할 수 있게 되어 웹 페이지의 동작 방식에 큰 변화가 생김 → 현대적인 웹 애플리케이션의 기반이 되었음

### 4.1.1.3 2008년: 구글의 V8 자바스크립트 엔진의 등작

- 초기 자바스크립트는 동적인 콘텐츠 업데이트, 사용자와의 상호작용에는 적합하지만, 복잡한 작업이나 대규모 애플리케이션 개발에는 적합하지 않다고 여겨짐
- 2005년 개발된 구글의 구글맵스는 자밧크릡트가 웹 애플리케이션 개발 언어로서 가진 가능성을 보여주는 계기가 됨
    
    → 이를 계기로 자바스크립트를 적극적으로 활용한 웹 애플리케이션의 구축이 본격화되었으며 클라이언트 측에서 수행해야하는 작업량이 증가하고 웹 애플리케이션의 복잡성이 높아지기 시작함
    
- 이런 배경으로인해 자바스크립트 엔진의 성능의 중요도가 높아졌으며, 2008년 구글은 V8 자바스크립트 엔진을 출시해 브라우저에서 실행되는 자바스크립트 코드의 속도를 크게 개선
- V8 엔진은 브라우저 외부에서도 사용할 수 있도록 개발되어, 라이언 달이 Node.js를 개발하게 됨

## 4.1.2 모듈화 이전의 자바스크립트

- 초기 자바스크립트에서는 하나의 파일에 모든 기능을 구현하거나 필요한 파일을 순서대로 불러오는 단순한 방식으로 코드를 작성했음
- 하지만 자바스크립트를 사용하는 코드의 양이 증가하고 구조가 복잡해지면서 아래와 같은 문제가 발생함
    - 전역 변수 문제
        - 모든 변수가 전역 스코프에 위치하면서 동일한 이름의 전역 변수가 다른 파일에 존재할 경우 기존 변수를 덮어쓰게됨
    - 복잡한 의존성 관리
        - 파일을 분리해도 모듈 간 의존 관계가 존재하면 올바른 순서대로 파일을 로드해야했음. 의존성이 많아질 수록 로드 순서 관리에 대한 부담이 증가함
    - 렌더링 지연 문제
        - 브라우저는 스크립트를 로드할 때마다 새 HTTP 연결을 생성하는데, 불러와야할 스크립트가 많을 경우 로드 시간이 길어지면서 렌더링이 블로킹되어 사용성이 저하됨
- 이런 문제들로 인해 자바스크립트에서도 모듈화가 필요하다는 인식을 갖게되었고 CommonJS, AMD, UMD, ESModule 등 다양한 모듈 시스템이 개발되었음

## 4.1.3 자바스크립트 모듈의 여러 시도들

### 4.1.3.1 즉시 호출 함수 표현식

- 모듈 시스템이 없던 초기에는 즉시 호출 함수 표현식(IIFE)을 사용해 모듈패턴을 구현했으며, ESModule이 도입되기 전에 주로 사용되었음
- 즉시 함수 호출 표현식(IIFE)
    - 함수를 정의하자마자 즉시 실행하는 방식
- 즉시 함수 호출 표현식(IIFE)을 활용한 모듈 패턴
    
    ```jsx
    ;(function(){
    	// 모듈 코드
    })()
    ```
    
    - 괄호로 둘러싸인 함수는 즉시 실행되어, 결과로 새로운 스코프가 형성됨
    - 새로 생긴 스코프 내에서 정의된 변수와 함수는 외부 스코프에서 접근할 수 없으므로 네임스페이스의 충돌을 방지하고 모듈 간의 의존성 관리가 가능함
- 예시
    
    ```jsx
    const CounterModule = (function () {
      // 외부에서 접근할 수 없는 private 변수
      let count = 0;
    
      // 외부에서 접근할 수 없는 private 함수
      function log(message) {
        console.log(`[CounterModule]: ${message}`);
      }
    
      // 외부에서 접근 가능한 public 메서드 (객체로 반환)
      return {
        increment() {
          count++;
          log(`count increased to ${count}`);
        },
        decrement() {
          count--;
          log(`count decreased to ${count}`);
        },
        getCount() {
          return count;
        }
      };
    })();
    
    CounterModule.increment(); // [CounterModule]: count increased to 1
    CounterModule.increment(); // [CounterModule]: count increased to 2
    console.log(CounterModule.getCount()); // 2
    
    // 외부에서는 private 변수에 접근 불가
    console.log(CounterModule.count); // undefined
    ```
    
- IIFE를 사용한 모듈 패턴의 문제점
    - 동일한 네임스페이스 충돌 문제
        - 프로젝트 규모가 커지거나 외부 라이브러리와 결합될 때 동일한 이름을 가진 모듈이 중복선언되어 충돌이 발생할 가능성이 커짐
        - IIFE 패턴은 전역에 모듈을 선언하므로 동일한 이름의 모듈이 존재할 경우 오류가 발생할 수 있음
    - 의존성 관리의 어려움
        - IIFE 방식에서는 모듈 간 의존 관계를 명시적으로 관리하기 어려움
        - 예시) ModuleB가 ModuleA에 의존할 경우 ModuleA → ModuleB로 로드 순서가 지켜져야하는데, IIFE 방식은 모듈 순서를 수동으로 관리해야 하므로 어려움이 존재
    - 예시
        
        ```jsx
        // module A
        const ModuleA = (function (){
        	const name = 'ModuleA'
        	function greet(){
        		console.log(`Hello from ${name}`)
        	}
        	return {
        		greet,
        	}
        })()
        
        // module B
        const ModuleB = (function (){
        	const name = 'ModuleB'
        	function greet(){
        		console.log(`Hello from ${name}`)
        	}
        	return {
        		greet,
        	}
        })()
        
        ModuleA.greet() // "Hello from ModuleA"
        ModuleB.greet() // "Hello from ModuleB"
        
        // 동일한 변수로 또 다른 모듈 선언 시
        const ModuleA = (function (){
        	const name = 'AnotherModuleA'
        	function greet(){
        		console.log(`Hello from ${name}`)
        	}
        	return {
        		greet,
        	}
        })()
        
        // 동일한 이름으로 모듈 정의 시 오류 발생
        // Uncaught SyntaxError: Identifier 'ModuleA' has already been declared
        ```
        

### 4.1.3.2 AMD(Asynchronous Module Definition)와 RequireJS

- AMD(Asynchronous Module Definition)
    - 모듈을 비동기적으로 로드하고 정의하는 자바스크립트의 모듈 정의 표준 중 하나
    - 주로 브라우저 환경에서 사용됨
    - RequireJS는 AMD를 구현한 대표적인 예시
- 특징
    - 모듈을 비동기적으로 로드하는 방식으로 브라우저에서 여러 모듈을 병렬로 로드할 수 있도록 도와 페이지 초기 로딩 속도를 향상시킴
    - 모듈 간 의존성을 명시적으로 정의하게 하여 각 모듈이 필요한 의존 모듈을 명시하고 해당 모듈이 로드될 때까지 기다릴 수 있게함
- 예시
    
    ```jsx
    // define() 함수를 이용한 AMD 모듈 정의
    define( // 의존 모듈들을 배열 형태로 넘기지만 로드 순서를 보장하지는 않음
    ['dependency1', ['dependency2'], function (dependency1, dependency2) {
      // 의존 모듈들은 순서대로 매개 변수에 담김
      // 의존 모듈들이 모두 로딩된 후 해당 함수를 실행함
      let count = 0;
    
      function log(message) {
        console.log(`[CounterModule]: ${message}`);
      }
    
      // 공개(public) API 반환
      return {
        increment() {
          count++;
          log(`count increased to ${count}`);
        },
        decrement() {
          count--;
          log(`count decreased to ${count}`);
        },
        getCount() {
          return count;
        }
      };
    });
    
    // main.js
    require(['CounterModule'], function (CounterModule) {
      CounterModule.increment(); // [CounterModule]: count increased to 1
      CounterModule.increment(); // [CounterModule]: count increased to 2
      console.log(CounterModule.getCount()); // 2
    
      // 외부에서는 count에 직접 접근 불가
      console.log(CounterModule.count); // undefined
    });
    ```
    
- RequireJS
    - AMD 명세에 몇 가지 기능을 추가해서 더 편리하게 사용할 수 있게 함
        - 모듈이 처음 실행 될 때 생성한 클로저를 통해 초기화 이후의 상태를 유지하거나 클래스 형태로 모듈을 구현할 수 있게 지원
        - <script> 태그 내부에서 RequireJS의 설정 옵션 제공 가능
- AMD와 RequireJS는 주로 브라우저 환경에서 사용되며, 대규모 애플리케이션에서 모듈화와 성능 최적화를 위해 의존성을 명시하고 비동기 로딩을 활용해 효율적인 자원 관리를 할 수 있게 해줌
- AMD 방식의 단점
    - 복잡한 문법
        - AMD는 define과 require 함수를 사용해 모듈과 의존성을 선언하는 방식. 함수 호출과 배열 형태의 의존성 선언이 필요하다보니 문법이 복잡하고 가독성이 떨어짐
        - AMD vs CommonJS vs ESModule
            
            ```jsx
            // AMD
            define(['moduleA', 'moduleB'], function (moduleA, moduleB){
            	moduleA.doSomething()
            	moduleB.doSomethingElse()
            })
            // CommonJS
            const moduleA = require('moduleA')
            const moduleB = require('moduleB')
            
            moduleA.doSomething()
            moduleB.doSomethingElse()
            
            // ESModule
            import {doSomething} from 'moduleA'
            import {doSomethingElse} from 'moduleB'
            
            doSomething()
            doSomethingElse()
            ```
            
    - 정적 분석의 어려움
        - AMD는 모듈과 의존성을 동적 함수 호출로 정의하므로 코드 작성 시점에서 의존성을 정적으로 분석하기 어려움 → 빌드 도구가 트리 셰이킹이나 코드 분할 같은 성능 최적화를 적용하기 어려워짐
    - 비동기 로드의 제한
        - AMD는 모듈을 비동기로 로드하므로 네트워크 상태에 따라 로드 지연이 발생할 수 있음. 동기 로드가 필요한 경우 불편함
    - 서버 환경에 대한 제약
        - AMD는 브라우저 환경을 중심으로 설계되었음 → Node.js 같은 서버 환경에서 사용하기엔 제약이 존재함. 반면 ESModule은 일관된 모듈화 방식을 제공함
- 이와 같은 단점으로 인해 AMD와 RequireJS의 사용 빈도가 감소했으며, 브라우저와 서버 환경에서 표준으로 지원되는 ESModule이 현대 자바스크립트의 기본 모듈 시스템으로 자리 잡았음

### 4.1.3.3 CommonJS

- 브라우저 이외의 환경에서 자바스크립트를 사용하기 위한 표준 모듈 시스템으로 등장
- 특징
    - 동기적으로 모듈을 로드
        - 이로 인해 서버사이드 환경에서 파일 시스템 접근이나 네트워크 요청과 같은 I/O 작업에 적합 → 서버 런타임에서 동작하는 Node.js는 이런 이유로 CommonJS를 표준으로 채택했음
    - exports와 require() 함수를 사용해 모듈을 가져오고 내보낼 수 있음

### 4.1.3.4 UMD(Univarsal Module Definition)

- AMD와 CommonJS는 각각 브라우저 환경과 서버 사이드 환경에서 모듈 시스템을 지원하기 위해 만들어진 두 가지 다른 표준
- UMD는 이 두 모듈 시스템을 통합해서 다양한 환경에서 호환성을 제공하기 위해 설계된 패턴
- 여러 환경에서 동일한 모듈을 사용할 수 있도록 지원하며 보편화된 템플릿 코드를 통해 AMD와 CommonJS 뿐만 아니라 브라우저의 전역 객체에도 대응함
- 구현 예시
    
    ```jsx
    ;(function (root, factory) {
      if (typeof define === 'function' && define.amd) {
        // AMD 환경
        define(['dependency'], factory);
      } else if (typeof module === 'object' && module.exports) {
        // CommonJS 환경
        module.exports = factory(require('dependency'));
      } else {
        // 전역 객체에 할당
        root.returnExports= factory(root.dependency);
      }
    })(typeof self !== 'undefined' ? self : this, function (dependency) {
    	// 모듈 코드
    	return {
    		// 모듈의 공개 API
    	}
    })
    ```
    
    - IIFE에 두 개의 인자를 전달해서 UMD 패턴 구현 가능
        - 첫 번째 인자는 브라우저 단에서 구현을 위한 root 값을 설정하는 데 사용. root가 undefined인 경우 this를 사용하는데, 이건 브라우저에서 window 전역 객체를 의미함
        - 두번째 인자로 모듈 코드를 정의하는 함수를 전달하여 다양한 환경에서 동일한 모듈 개념을 사용할 수 있도록 함
- 단점
    - 불필요한 코드 증가
        - 여러 환경에서의 호환성을 위해 다양한 조건문과 전역 객체에 대한 참조를 포함하므로 코드가 불필요하게 길어질 수 있음
    - 성능 저하
        - 모든 환경을 지원하기 위한 로직을 포함하므로 특정 환경에서만 사용할 때도 불필요한 로직이 실행될 수 있음
    - 최신 모듈 시스템과의 비교
        - ESModule이 표준화되고 브라우저와 Node.js 모두에서 지원됨에 따라 UMD가 필요했던 다양한 환경 간의 호환 문제가 많이 해소되었음 → 현대적인 코드 베이스에서는 UMD는 점차 구식 패턴으로 간주되고, ESModule이 선호되고 있음
    - 복잡한 유지보수
        - CommonJS, AMD, 전역 객체를 모두 지원하려다 보니 코드가 복잡해져 유지보수가 어려워질 수 있음
- 결론
    - UDM는 점차 사용 빈도가 줄어들고 있으며(기존 라이브러리와 호환성을 유지해야 하는 경우 제외), 현대 자바스크립트 환경에서는 ESModule이 이를 대체하고있음

### 4.1.3.5 SystemJS

- 동적으로 모듈을 로드하고 실행하는 자바스크립트 로더
- 주로 브라우저에서 모듈 간의 의존성을 해결하고 동적으로 모듈을 로드해야하는 복잡한 프로젝트에서 사용됨
- 특징
    - 필요한 시점에 모듈을 동적으로 로드할 수 있으므로 애플리케이션 실행 도중 필요한 모듈을 비동기적으로 로드하여 성능을 최적화할 수 있음
    - AMD, CommonJS, UMD, ESModule 등 다양한 모듈 형식을 지원하므로 기존 프로젝트의 모듈 형식을 변경하지 않고 쉽게 도입 가능
        
        → IE11과 같이 ESModule을 기본적으로 지원하지 않는 구형 브라우저에서 ESModule을 트랜스파일해서 사용할 때 유용하게 활용할 수 있음
        
- 단점
    - 표준화 되지 않은 방식
        - ESModule과 달리 표준 모듈 시스템이 아니므로 브라우저나 서버 환경에서 기본적으로 지원되지 않음 → 사용하기 위해 추가적인 로더 라이브러리가 필요
    - 성능 이슈
        - 다양한 모듈 형식을 지원하고 동적 로딩을 수행하기 때문에 모듈 로딩 시 성능에 미세한 영향을 줄 수 있음
    - 복잡성 증가
        - 여러 모듈 형식과 동적 로딩을 지원하기 때문에 코드 구조가 복잡해질 수 있음. 의존성 배열과 실행 메서드를 활용하는 방식은 ESModule에 비해 상대적으로 복잡함
    - 미래 사용성 감소
        - ESModule이 표준으로 채택되어 브라우저와 Node.js 같은 서버 환경에서 널리 사용되면서 SystemJS의 필요성이 점차 감소
        - ESModule은 기본적으로 브라우저와 서버에서 지원되므로 별도의 로더 없이 호환성을 보장하며 성능 최적화도 가능함
- 결론
    - SystemJS는 주로 구형 브라우저 지원이나 복잡한 동적 모듈 로딩이 필요한 특정 프로젝트에 한해 사용되고 있으며 그 외의 경우에는 사용 빈도가 줄어들고 있음

### 4.1.3.6 ESModule

- ES6에서 웹 브라우저와 Node.js에서 표준 모듈 시스템을 도입
- ESModule은 표준화된 형태로 자바스크립트 언어 자체에 통합되어 현재 대부분의 모던 브라우저와 Node.js 같은 서버 환경에서 표준으로 지원되며, 자바스크립트 생태계에서 모듈화의 표준으로 자리 잡았음

## 4.1.4 오늘날의 자바스크립트 모듈 시스템

- CommonJS, ESModule은 가장 널리 사용되는 두 가지 모듈시스템으로, 많은 자바스크립트 라이브러리가 이 두 모듈 시스템을 따르고 있음
- ESModule의 등장으로 브라우저와 서버 환경 모두에서 ESModule의 사용 빈도가 증가하고 있지만, 여전히 여러 환경에서  두 모듈 시스템을 혼용하는 경우가 많음
- 서버사이드에서는 CommonJS를, 클라이언트 사이드에서는ESModule를 사용해 코드를 구성하는 것이 일반적임

# 4.2 CommonJS란 무엇일까?

## 4.2.1 CommonJS의 탄생 배경

- 2000년대 후반, 자바스크립트가 서버 환경에서도 사용될 가능성이 열리면서 자바스크립트 생태계가 빠르게 확장되기 시작했음
- 2009년 1월 모질라 엔지니어 케빈 당구르는 자바스크립트를 서버에서 확장성 있게 활용하기 위해 필요한 표준을 고민하며 아래와 같은 몇 가지 요구사항을 제안함
    - 상호 호환되는 표준 라이브러리가 필요하다.
    - 서버와 웹 간에 상호작용할 수 있는 표준 인터페이스가 필요하다.
    - 다른 모듈을 로드할 수 있는 표준이 필요하다.
    - 코드를 패키지화하고 배포 및 설치하는 방법이 필요하다.
    - 패키지를 설치하고, 패키지 간 의존성을 해결하는 패키지 저장소가 필요하다.
- 이러한 요구는 서버사이드에서 자바스크립트 코드가 확장 가능하고 효율적으로 관리될 수 있도록 자바스크립트만의 모듈 시스템이 필요하다는 결론으로 이어짐 → 표준 명세 수립을 시작
- 2009년 9월 자바스크립트 콘퍼런트에서 CommonJS API 1.0 명세가 발표되었음 → 이후 CommonJS는 서버와 브라우저를 포함한 다양한 환경에서 사용할 수 있는 모듈시스템으로 자리잡음
- CommonJS 대표적인 모듈 예시
    - 모듈
    - 바이너리 문자열 및 버퍼
    - 문자 집합 인코딩
    - 바이너리, 버퍼 및 텍스트 I/O 스트림
    - 시스템 프로세스 인수, 환경 및 스트림
    - 파일 시스템 이넡페이스
    - 소켓 스트림
    - 단위테스트 어설션, 실행 및 보고
    - 웹 서버 게이트웨이 인터페이스, JSGI
    - 로컬 및 원격 패키지 및 패키지 관리

## 4.2.2 CommonJS의 명세

- CommonJS로 모듈을 구축하기 위해 따라야하는 세 가지 명세
    - 독립적인 실행 영역
        - 모든 모듈은 자신만의 독립적인 실행 영역을 가짐 → 각 모듈이 외부와 격리된 환경에서 실행되게 함으로써 동일한 변수 이름을 사용하더라도 다른 모듈의 변수에 영향을 주지 않음을 의미
    - exports 객체로 모듈 정의
        - 모듈 간 정보 교환이 필요할 때 외부에 공개할 기능만 exports 객체로 정의 가능 → 모듈이 필요한 기능 만을 외부에 노출할수 있음
    - require() 함수로 모듈을 사용
        - require() 함수를 통해 필요한 모듈을 불러올 수 있음. 함수의 첫 번째 인수로 불러올 모듈명을 전달하면 해당 모듈의 exports 객체가 반환됨 → 다른 파일에서 정의된 기능을 쉽게 가져오고 사용 할 수 있음
- CommonJS는 간단하고 명확한 문법으로 자바스크립트 모듈 시스템을 구현할 수 있는 방식으로 주목받았음
- Node.js 같은 프로젝트들이 CommonJS를 채택하면서 자바스크립트 모듈 시스템의 사실상 표준으로 자리잡았음
- ESModule 등장 전까지 많은 서드파티 벤더들이 CommonJS 명세를 기반으로 모듈 시스템을 구현했으며, 여전히 널리 사용되고 있음
- CommonJS는 서버 측 라이브러리에 국한되지 않고 데스크톱 애플리케이션, 브라우저 라이브러리에서도 CommonJS API를 기반으로 구현되어 다양한 환경에서의 활용 가능성을 보여줌

## 4.2.3 Node.js의 CommonJS

- Node.js 초기 단계에 CommonJS를 선택해 모듈 시스템으로 채택
- 서버사이드 자바스크립트 개발을 촉진하기 위해 만들어진 Node.js에서 CommonJS의 도입은 여러 이유와 이점을 제공함
    - 다양한 기능의 필요성
        - 서버사이드 개발에서는 파일 시스템 접근, 네트워크 통신 등 다양한 기능이 요구됐고 이를 효과적으로 관리하기 위해 모듈시스템이 필수적이었음. CommonJS는 이러한 기능을 구조화하여 제공할 수 있는 환경을 마련함
    - 논블로킹 I/O 모델과의 호환
        - Node.js는 비동기 처리를 위한 논블로킹 I/O 모델을 채택해 많은 동시 연결을 효과적으로 처리할 수 있도록 설계되었음. 이 모델에서는 모듈 단위로 빠르게 로딩, 실행하는 것이 중요했으며 CommonJS 동기적 로딩 방식이 이러한 블로킹 처리를 간편하게 처리할 수 있도록 도움
    - CommonJS의 보급도
        - Node.js 등장 당시 이미 많은 개발자들이 CommonJS를 사용하고 있었으며, 기존 CommonJS 모듈을 Node.js에서 그대로 활용할 수 있었음
- 이러한 이유로 Node.js는 CommonJS를 채택해 모듈 시스템 표준을 확립, 서버 사이드 자바스크립트 개발을 가능하도록 함

### 4.2.3.1 CommonJS 파일 규칙

- Node.js는 애플리케이션을 구성하는 모듈 파일 중 특정 파일을 CommonJS 모듈로 인식하고 해석함
- CommonJS 모듈로 판단하는 규칙
    - require()가 사용된 .js 파일들
    - 가장 가까운 package.json의 type 필드가 “commonjs”인 하위 .js파일들
        - type 필드를 명시적으로 설정하면 빌드 도구나 로더가 자바스크립트 파일을 해석할 때 유리하며 패키지 내에서 CommonJS 모듈 규칙을 적용하는데 도움을 줌
    - .cjs 확장자로 끝나는 파일들
        - 이러한 확장자를 가진 파일은 주로 CommonJS로 빌드된 결과물에서 볼 수 있음
- 일부 애플리케이션의 빌드 결과물에는 .cjs와 함께 .mjs 파일이 포함될 수 있는데, .mjs 확장자로 끝나는 파일은 ESModule로 해석되며 CommonJS 모듈은 .mjs 파일을 require()로 불러올 수 없음(require()함수가 ESModule의 로딩 방식과 호환되지 않기 떄문)

### 4.2.3.2 모듈 내보내기

- Node.js에서는 CommonJS 명세에 따라 exports 객체를 사용해 모듈을 외부로 공개할 수 있음
- 예시
    
    ```jsx
    // circle.js
    const {PI} = Math
    exports.area = (r) => PI * r ** 2
    
    // 가져와서 사용하는 예시
    // index.js
    const circle = require('./circle.js')
    
    console.log(circle.area(5)); // 원의 넓이
    ```
    
    - exports는 전역 변수처럼 사용할 수 있지만 실제로는 전역변수가 아님
    - Node.js에서는 모든 모듈이 독립된 함수 스코프 내에서 실행되므로 exports는 모듈 스코프에만 존재하는 객체
    - Node.js는 각 모듈을 함수로 감싸서 실행해 다른 모듈과 격리된 환경에서 동작하도록 하는데, 이를 모듈 래퍼(module wrapper)라고 함

### 4.2.3.2.1 module.exports

- Node.js는 export 객체 외 module.exports를 추가로 둬서 모듈이 외부에 노출하는 객체로 사용함
- module.exports에 Loan 클래스를 할당한 예제
    
    ```jsx
    // loan.js
    class Loan {
    	amount = 0
    	interestRate = 0.0
    	
    	constructor(amount, interestRate){
    		this.amount = amount * 10000
    		this.interestRate = interestRate
    	}
    }
    
    module.exports = {Loan}
    exports.LoanFromExports = Loan
    
    // index.js
    const loanModule = require('./loan')
    console.log(loanModule ) // { Loan: [class Loan] }
    
    const {Loan, LoanFromExports } = loanModule
    const loan = new Loan(1000, 1.1) 
    console.log(loan.amount, loan.interestRate) // 10000000 1.1
    
    const loanFromExports = new LoanFromExports(1000, 1.1) 
    // TypeError: LoanFromExport is not a constructor
    console.log(loanFromExports.amount, loanFromExports.interestRate )
    ```
    
    - require('./loan') 함수를 호출하면 Loan 객체만 존재. LoanFromExports를 사용할 때 타입에러가 발생하는 것을 확인할 수 있었으며 여기서 파악할 수 있는 사실은 아래와 같음
        - require() 함수는 module.exports를 반환한다.
        - exports를 단독으로 사용하면 require() 함수를 module.exports와 동일한 결과를 반환하지만 함께 사용하면 module.exports 값만 반환된다.
- 정리
    - exports와 module.exports는 동일한 객체를 참조함
    - exports에 { LoanFromExports } 와 같은 새로운 객체를 직접 할당하면 module.exports와의 참조가 끊어지면서 exports는 독립적인 객체가 됨
    - 하지만 이 변경은 module.exports에 영향을 주지 않으므로 외부에서 require() 함수로 모듈을 가져올 때 exports에 할당된 값은 사용할 수 없음
- Node.js가 exports 객체만으로도 모듈을 내보낼 수 있음에도 **module이라는 객체를 따로 제공**하는 이유
    - 모듈 자체를 하나의 인스턴스로 활용할 가능성을 열어두기 위함
    - 예시에 나온대로 Loan 클래스를 모듈로 만들어서 매번 새 인스턴스를 생성하고 싶은 경우, exports.Loan = Loan을 사용해도 되지만, Exports = Loan처럼 직접 할당하면  module.exports와의 참조가 끊어지게 됨
    - 클래슬를 모듈로 내보내고 싶은 경우 module.exports에 직접 할당하는 방식이 필요함. 또 이를 이용하여 모듈이 클래스 자체로 동작할 수 있는 구조를 만들 수 있음
    - exports = Loan 직접 할당 예시
        
        ```jsx
        // loan.js
        class Loan {
        	amount = 0
        	interestRate = 0.0
        	
        	constructor(amount, interestRate){
        		this.amount = amount * 10000
        		this.interestRate = interestRate
        	}
        }
        
        exports = Loan
        // index.js
        const loanModule = require('./loan')
        console.log(loanModule) // {}
        ```
        
    - module.expors로 구현한 예시
        
        ```jsx
        // loan.js
        class Loan {
        	amount = 0
        	interestRate = 0.0
        	
        	constructor(amount, interestRate){
        		this.amount = amount * 10000
        		this.interestRate = interestRate
        	}
        }
        
        module.expors = Loan
        // index.js
        const loanModule = require('./loan')
        console.log(loanModule) // [class Loan]
        const loan = new Loan(1000, 1.1) 
        console.log(loan.amount, loan.interestRate) // 10000000 1.1
        ```
        
        - 이 방법으로 구현하면 간편하게 클래스를 모듈로써 공개해서 인스턴스를 생성할 수 있음
- Node.js는 module.exports라는 인터페이스로 exports로 할 수 없었던 단순 객체, 그 외 함수, 클래스, 숫자, 문자열 등 어떤 값이라고 할당할 수 있는 유연성을 제공함

### 4.2.3.2.2 모듈 래퍼와 모듈 스코프

- Node.js는 모듈마다 독립적인 실행 영역을 제공하기 위해 모듈 래퍼 함수를 사용해 구현함
- 모듈래퍼(module wrapper)
    - 모듈의 코드가 실행되기 전에 모듈을 감싸는 함수
    - 모듈의 코드를 모듈 코드라고 하는데, 이는 모듈 래퍼의 스코프 내부에서만 유효하며 이 스코프를 모듈 스코프라고 함
- Node.js는 각 파일을 실행할 때 내부적으로 아래와 같이 함수로 감싸게 됨
    
    ```jsx
    (function (exports, require, module, __filename, __dirname) {
      // 모듈 스코프
    });
    ```
    
- 모듈래퍼로 모듈을 함수로 감싸 외부에 공개함으로써 얻을 수 있는 이점
    - exports로 할당되지 않은 모듈 내부의 로컬 변수나 함수들이 외부로부터 숨겨질 수 있음
        
        → 모듈 최상단에 위치한 변수나 함수가 전역객체에 포함되지 않으므로 전역 변수의 오염 가능성이 사라짐
        
    - 모듈 스코프에서 모듈에 고유한 정보를 담은 변수를 제공해 외부에서 모듈에 관한 정보를 쉽게 사용할 수 있음
        - 모듈 래퍼가 제공하는 모듈 스코프에서 생성되는 변수 다섯 개(CommonJS에서 제공하는 값)
            - module
                - Node.js에서 제공하는 module 객체. 현재 모듈에 대한 참조로 exports 값 외 filename, loaded, children 등의 정보를 포함
            - exports
                - 앞서 소개한 exports 객체를 가리키며, module.exports를 축약한 표현
            - require
                - 모듈을 가져오기 위한 함수
            - __filename
                - 현재 모듈 파일의 절대 경로
            - __dirname
                - 현재 모듈의 디렉터리 경로

### 4.2.3.3 모듈 가져오기

- require() 함수를 통해 모듈을 외부에서 가져올 수 있음
- require() 함수의 첫 번째 인수로 파일이 위치한 경로를 문자열로 할당하면 해당 위치에서 파일을 찾고, 파일이 내보낸 module.exports(exports) 객체를 반환함
    - 이 때 이 첫 번째 인수를 모듈 지정자(module specifier)라고 함
- require() 함수는 모듈 지정자르 기반으로 파일을 찾는 규칙을 따르는데, 이 규칙은 다음 5단계로 구성됨

### 4.2.3.3.1 파일 모듈(File modules)

- 파일 경로를 확장자까지 포함해 명확히 지정하면 require() 함수가 해당 파일을 불러옴
    - 절대 경로와 __dirname(현재 작업 디렉터리 기준)으로부터의 상대 경로 모두 사용 가능
- 파일 모듈 지정자에서 확장자를 생략할 경우 `.js, .json, .node` 순서대로 확장자를 붙여 찾게되며 이 외 확장자는 반드시 명시해야함
- 만약 지정된 경로에서 모듈을 찾지 못한다면, MODULE_NOT_FOUND 에러를 발생시킴

### 4.2.3.3.2 모듈로 정의된 폴더(Folders as modules)

- 폴더를 require()의 인수로 사용할 경우, 다음 세 가지 규칙에 따라 파일을 찾게됨
    - 폴더의 package.json 파일에 main 속성이 정의된 경우 해당 파일을 찾는다.
    - package.json 파일이 없거나 main 속성이 정의되지 않은 경우 해당 폴더의 index.js(또는 index.node)파일을 찾는다.
    - 그 외 경우에는 MODULE_NOT_FOUND 에러를 발생시킨다.

### 4.2.3.3.3 node_modules 폴더

- require() 대상이 모듈 코어 모듈이 아니고 파일 경로가 아닌 경우 현재 모듈의 디렉터리에서 `/node_modules` 폴더를 찾아 순차적으로 탐색함
- 해당 디렉터리에 없으면 부모 디렉터리의 `/node_modules` 폴더를 탐색하는 과정을 파일 시스템 최상위까지 반복하게됨 → 트리 역탐사
    
    ```jsx
    // /home/ry/projects/foo.js
    const lodash = require('bar.js')
    
    // 아래 순서대로 bar.js를 찾게됨
    // /home/ry/projects/node_modules/bar.js
    // /home/ry/node_modules/bar.js
    // /home/node_modules/bar.js
    // /node_modules/bar.js
    ```
    

### 4.2.3.3.4 전역 폴더(Global folders)

- 탐색 규칙에 따라 파일 모듈, 모듈로 정의된 폴더, node_modules를 순차적으로 탐색 후 모듈을 찾지 못한 경우 NODE_PATH 환경 변수가 설정돼 있다면 해당 경로에서 추가로 탐색하게됨
- NODE_PATH
    - 모듈 해석 알고리즘이 정의되기 전 다양한 경로에서 모듈의 로딩을 지원하기 위해 만들어짐
    - 하지만 NODE_PATH에 의존하면 시스템의 모듈 버전에 따라 다른 모듈이 로드될 수 있으므로 안정성과 일관성을 위해 사용 권장 X

### 4.2.3.3.5 코어 모듈(Core modules)

- Node.js는 lib 폴더에 여러 코어 모듈을 포함하며, node: 접두사가 붙은 모듈들은 module.builtinModules에 정의돼있음
- 코어 모듈은 Node.js에서 기본적으로 제공되는 모듈로 외부 의존성을 설치하지 않고도 사용 가능함

### 4.2.3.4 동기적으로 실행되는 require() 함수

- require() 함수는 동기적으로 동작하는 것이 특징인데, 이로 인해 Node.js에서 모듈을 가져올 때 다음과 같은 특징이 있음
    - 모듈 내에서 발생하는 여러 동작을 동기적으로 완료한 후에 내보낸 값을 사용할 수 있음
    - 빌드 시점에서는 module.exports 정보 중 무엇을 사용할지 알 수 없고, 코드가 평가되는 시점인 런타임 때 알 수 있음
    - 런타임 때 평가되므로 코드 어디서나 호출이 가능하며 조건부로 require()를 호출할 수도 있음
    - require() 함수는 단순하고 동기적으로 동작하므로 인수로 전달하는 모듈 경로를 동적으로 할당할 수 있음

### 4.2.3.5 require.cache

- require()로 불러온 모듈은 한 번 로딩된 후 캐시에 저장됨 → 캐싱 덕분에 모듈 로딩의 성능이 최적화
- 모듈이 캐싱된 정보는 require.cache 객체에서 확인 가능하며, 키-값 쌍의 객체이기 때문에 추가, 수정, 삭제가 가능함
- require.cache는 모듈 지정자로 받은 경로를 **절대 경로**로 변환 후 해당 경로를 키로 사용해 모듈을 캐싱함
    - 이로 인해 대소문자 구분이 없는 파일시스템을 사용하는 운영체제(e.g. 윈도우)에서도 대소문자가 다른 경우 모듈을 호출하면 각기 다른 모듈로 취급됨
        
        ```jsx
        // foo와 FOO는 다른 캐시에 저장됨
        require('foo.js')
        require('FOO.hs')
        ```
        
- Node.js의 코어모듈도 캐시에 저장된 객체를 다른 객체로 덮어쓸 수 있지만 이 경우 node: 접두어를 붙여 module.buildinModules의 모듈을 참조해야만 실제 내장 모듈의 구현체를 사용할 수 있음
- 또한 캐시 정보를 수정했다하더라도 모듈에 따라 의도했던 대로 동작하지 않을 수도 있으므로 코어 모듈의 캐시 정보를 덮어쓰는 것은 권장하지 않음

### 4.2.3.6 순환참조

- require() 함수를 사용해 모듈 간에 서로를 참조할 수 있는데, 다음과 같은 상황에서 순환 참조가 발생할 수 있음
    - A 모듈이 B 모듈을 참조하고, B 모듈이 다시 A 모듈을 참조하는 경우
    - A 모듈이 B 모듈을 참조하고, B 모듈이 C 모듈을 참조하며, C모듈이 다시 A모듈을 참조하는 경우
- 이러한 상황에서는 모듈 간에 무한루프가 발생할 수 있음 → 순환참조
- CommonJS는 require() 함수의 두 가지 특징을 통해 순환참조 문제를 해결 함
    - require() 함수는 모듈을 동기적으로 불러오므로 한 번에 하나씩 처리됨 → 각 모듈이 순차적으로 로딩되어 순환 참조가 발생해도 무한루프에 빠지지 않도록 제어 가능
    - requrie() 함수는 재참조를 대비해 모듈을 캐싱해서 보관함 → 모듈이 순환참조에 의해 재호출될 때 캐싱된 모듈이 반환되므로 무한로딩이 발생하지 않음

### 4.2.4 소스코드를 CommonJS로 빌드하기

- 애플리케이션 배포 시 소스코드느 실행 가능한 결과물로 변환되는 빌드 과정을 거치는데, CommonJS 파일로 빌드할 수도 있음
- 빌드 프로세스에서는 번들링 과정을 거치는데, 번들링을 수행하는 도구인 대부분의 번들러들은 트리셰이킹을 지원함
- 대표적인 자바스크립트 번들러에는 Webpack, Parcel, Rollup, Vite 등이 있으며, 이들 대부분 CommonJS 빌드를 지원함

### 4.2.4.1 모듈 래퍼는 클로저를 생성한다

- 모듈래퍼가 동작하는 방식은 외부에서 모듈을 사용할 때마다 해당 모듈이 클로저를 생성하는 것과 같음
- 클로저
    - 자신이 선언된 환경을 기억해서 내부 함수에서 외부 변수를 사용할 수 있게 하는 메커니즘
    - 클로저를 사용하면 개발자는 변수의 스코프를 의도적으로 조절할 수 있지만, 선언된 환경을 메모리에 유지해야하므로 메모리 사용비용이 발생하게 됨
    - 모듈래퍼는 클로저이므로 모듈 코드에서 사용하는 로컬 변수는 메모리에 저장됨
- 서버 환경에서 CommonJS는 모든 모듈이 로컬 디스크에 존재했기 때문에 크게 문제되지 않았지만, 브라우저 환경에서는 문제가 되었음 → 브라우저에서 로드해야할 모듈이 많을수록 렌더링을 지연시켜 성능 저하를 초래하기 때문
- 이와 같은 CommonJS의 성능 문제를 해결하기 위해 **Rollup, Webpack 같은 번들러는 모듈을 하나의 클로저로 통합하여 require() 호출로 인한 성능 문제를 줄였음**

### 4.2.4.2 require() 런타임에 사용할 모듈이 결정된다

- 다수의 클로저 생성을 방지하기 위해 모든 require() 호출을 단일 클로저로 연결하는 방식으로 해결할 수 있었지만, 이는 트리 셰이킹이 어려워진다는 단점이 있음
- 트리셰이킹이 어려워지는 이유
    - require() 함수의 동적 특정상 빌드 시점에는 어떤 모듈을 실제로 사용할 지 알 수 없고 런타임에서 사용할 모듈이 결정됨. 그래서 모든 모듈이 결과물에 포함되어 번들 크기가 크게 증가하게 됨

### 4.2.4.3 트리셰이킹에 대한 오해

- CommonJS가 무조건 트리셰이킹이 불가능한 건 아니지만 require()를 사용하는 경우 트리셰이킹이 어려움
- 트리셰이킹이 중요하다면 ESModule 형식으로 코드를 작성하거나 CommonJS 코드를 ESModule로 변환하는 도구를 사용하는 것이 효과적일 수 있음

# 4.3 ESModule이란 무엇일까?

## 4.3.1 ESModule의 탄생 배경과 도입

- ESModule은 자바스크립트의 공식적인 모듈 시스템으로 모듈화된 코드를 일관되고 효율적으로 관리할 수 있도록 설계되었음
- 브라우저 환경에서의 CommonJS는 다음과 같은 문제점을 가짐
    - 동기적 로딩방식
        - 모듈을 동기적으로 로드하므로 서버환경에서는 괜찮지만, 브라우저에서는 로딩 중 블로킹이 발생해 성능 저하를 초래할 수 있음
    - 프리로딩(preloading) 불가
        - 모듈을 미리 로드할 수 없으므로 브라우저에서의 로딩 최적화가 어려움
    - 트리셰이킹 및 최적화 어려움
        - 런타임에 모듈을 동적으로 로드하므로 의존성 분석이 어렵고, 트리셰이킹과 같은 최적화 작업이 어려움
    - 메모리 이슈
        - 모듈 래퍼는 클로저를 생성하여 모듈마다 독립적인 스코프를 제공하는데, 파일이 추가될 때마다 클로저가 생성되어 브라우저에서 메모리 사용량이 증가할 수 있음
    - 브라우저 호환성 문제
        - Node.js 환경을 위해 설계된 시스템이므로 브라우저에서 직접 사용하기 위해서 Browserify나 Webpack 같은 번들링 도구가 필요함
- 이러한 문제로 인해 CommonJS는 자바스크립트 표준 모듈 시스템으로 자리 잡지 못했고 ES6에서는 브라우저와 서버 환경 모두에서 일관되게 사용 가능한 표준모듈시스템은 ESModule이 도입되었음

## 4.3.2 ESModule의 특징

### 4.3.2.1 ESModule의 명세

- ES6부터 추가된 import와 export 키워드를 사용해 모듈을 가져오고 내보낼 수 있음

### 4.3.2.1.1 export

- 모듈에서 공개할 변수나 함수, 클래스를 명시하는 키워드
- 이름으로 내보내기(named export)와 기본 내보내기(default export)로 나뉘며, 모듈 하나에서 이름으로 내보내기는 여러개 존재할 수 있지만 기본 내보내기는 하나만 존재해야함
- 이름으로 내보내기(named export)
    - export { } 문법을 사용해 공개할 대상을 선언한 식별자를 객체의 속성으로 묶어 내보내는 방식
    - 한 파일에서 여러 모듈을 내보낼 때 유용함
    - import로 가져갈 때 내보낸 이름과 동일한 이름을 사용해야함
- 기본 내보내기(default export)
    - 하나의 대상만 내보낸다는 점에서 유용함
    - export default 문은 반드시 하나만 존재해야함
    - 모듈을 가져가는 곳에서 원하는 이름으로 사용할 수 있음

### 4.3.1.2 import

- 다른 모듈에서 내보낸 변수나 함수, 클래스를 가져오는 데 사용됨
- import와 함께 from 키워드를 사용해 가져올 모듈의 출처와 가져올 대상 모듈을 정의할 수 있음
- 이름으로 내보낸 모듈을 가져올 때는 내보내 모듈의 식별자를 사용해야함
- import * as 구문을 통해 내보낸 전체 식별자를 포함하는 객체를 가져올 수 있음
- 식별자 충돌을 피하기 위해 이름으로 내보낸 모듈을 가져올 때 as 키워드로 모듈에 별칭을 부여할 수 있음
- import를 사용해서 가져온 모듈은 모듈의 단일 인스턴스를 가져오므로 같은 모듈을 여러 번 가져와도 한 번만 로드됨 → require() 함수로 가져온 모듈을 캐싱해 한 번만 로드되는 CommonJS의 특징과 유사하지만 방법은 다름

### 4.3.2.1.3 import.meta

- ESModule은 CommonJS의 __dirname, __filename 대신 모듈의 정보를 제공하는 객체인 import.meta를 지원함
- import.meta 객체
    - ESModule 명세에 포함된 특별한 내장 객체로 현재 모듈에 대한 정보를 포함하고 있음
- import.meta는 아래와 같은 속성과 메서드를 제공함
    - import.meta.url
        - 현재 모듈의 URL을 나타내는 문자열
    - import.meta.resolve(moduleName)
        - 현재 모듈의 UR을 기반으로 모듈 지정자를 URL로 해석하는 메서드

### 4.3.2.1.4 .mjs 파일 확장자

- ECMAScript 표준에서 ESModule을 도입하면서 CommonJS와의 구분을 위해 .mjs라는 새로운 파일 확장자가 추가되었음

### 4.3.2.2 정적 모듈 로딩

- **ESModule은 모듈을 정적으로 로드하며 이는 빌드 시점에 모듈을 가져온다는 것을 의미**
    
    → 코드가 실행되기 전에 필요한 모듈이 이미 로드되어있고, 이는 번들러가 소스 코드를 번들링할 때 모든 의존성을 파악하고 포함시키는 방식으로 이뤄짐
    
- **정적 모듈 로딩의 장점(런타임 때 동적으로 모듈을 로드하는 방식과 비교)**
    - 불필요한 대기 시간 감소
        - 정적으로 모듈을 로드하여 코드 실행 전에 모듈을 가져오므로 런타임 중 모듈을 가져오는 대기 시간이 줄어듦
    - 코드 예측 가능
        - 코드 간의 의존성 관계를 명확하게 정의하고 예측할 수 있게 해줌
    - 모듈 캐싱과 최적화
        - 정적 모듈 로딩을 통해 브라우저나 실행환경에서 모듈을 캐싱하고 최적화 할 수 있음
    - 의존성 관리 용이
        - 모듈 간의 의존성을 명시적으로 관리할 수 있음
    - 번들 최적화
        - 번들링 도구가 코드를 분석해서 필요한 모듈만 번들에 포함시키므로 효율적으로 번들을 생성하는 데 도움을 줌
- ESModule의 모듈을 동적으로 로드하는 방법
    
    ```jsx
    import('./myModule.js')
    	.then((module) => {
    		// 모듈 사용
    		module.sum(1, 2)
    	})
    	.catch((error) => {
    		// 모듈 로드 실패 시 처리
    		console.error('모듈 로드 실패:', error)
    	})
    ```
    
    - 동적으로 가져온 모듈은 import() 함수가 비동기적으로 모듈을 로드하기 때문에 Promise를 반환함
    - 동적으로 모듈을 가져올 땐 모듈이 실제로 필요한 시점에만 로드되므로 비동기적으로 처리해야 함 → 다른 코드의 실행을 차단하지 않고 모듈을 가져올 수 있으며, 이로 인해 웹 애플리케이션의 성능을 향상시킬 수 있음

### 4.3.2.3 최상위 수준 await

- ESModule은 모듈 전체가 하나의 거대한 비동기 함수로 동작할 수 있는데, 이를 최상위 수준 await(top-level await)이라고 함
- 예시
    
    ```jsx
    // todoList.js
    let todoList
    const response = await fetch('https://test.com/todos')
    todoList = await response.json()
    export {todoList}
    
    // index.js
    import {todoList} from './toodList.js'
    console.log(todoList) // [{titile: '할 일', description: '...'}]
    ```
    
    - 최상위 수준 await을 사용한 todoList.js 모듈을 불러온 index.js는 toodList.js에서 비동기 처리가 완료되기 전까지는 실행을 멈춘 후 비동기 처리 완료 후 todoList 배열을 사용할 수 있음 → index.js에서 todoList를 바로 사용하고 있더라도 정적 분석을 통해 비동기 처리가 완료됐다는 것을 보장
    - 참고로 이 기능은 ES2022부터 사용이 가능

### 4.3.2.4 ESModule의 동작방식

- 모듈 간의 의존 관계는 개발자가 import 문을 작성할 때 생성되는데, ESModule은 파일 내부의 import 문을 따라 파일이 의존하는 다음 모듈을 찾아가면서 의존성 그래프를 구성함
- 이러한 과정은 ESModule 명세에서 모듈 파싱, 모듈 인스턴스화, 모듈 평가 세 단계로 나눠서 설명하고있음

### 참고 -  ESModule에서 파일을 로드하는 방법

- ESModule 명세는 모듈 파일을 불러오는 방법에 대한 내용은 기술하지 않는데, 이는 파일을 불러오는 로더가 ESModule을 사용 중인 환경에 따라 결정되기 때문
- 브라우저의 경우 HTML 명세를 따르며, Node.js는 ESModule 로더를 사용함
- **모듈이 정의된 파일을 불러오는 방법은 사용하는 환경의 로더가 결정하지만, 이 모듈 파일의 구문을 분석하고 인스턴스화해서 모듈을 평가하는 방법은 ESModule 명세에 기술돼있음**

### 4.3.2.4.1 모듈 파싱(module parsing)

- **브라우저나 자바스크립트 엔진이 로드한 모듈 파일을 해석해 모듈 레코드를 생성, 해당 모듈의 구문과 의존성을 분석하는 과정(모듈이 문법적으로 유효한지 확인, 모듈 구조를 이해하는 작업도 수행) → 모듈 파싱은 모듈 내용이 자바스크립트 코드로 해석될 수 있는지 확인하고 의존성을 분석하는 과정**
- 이 단계에서 모듈 레코드가 생성되는데, 이를 통해 모듈 로더가 요청 시 해당 모듈을 추적하고 관리할 수 있음
- **모듈 레코드(module record)**
    - 실제 메모리에 로드된 모듈의 값을 관리하는 구조체
    - 모듈의 구조적인 정보와 의존하는 다른 모듈들의 정보를 포함하며, 자바스크립트 엔진이 모듈의 상태를 관리하고 의존성을 추적하는데 사용됨
    - 해당 모듈의 상태, export된 항목, 의존하는 모듈들의 정보를 포함하고 있음
    - 모듈 레코드는 파싱 단계에서 생성되지만, 실제로 이 레코드로 함수와 변수들이 메모리에 할당되고 모듈 간의 참조관계가 설정되는 것은 그 다음단계인 모듈 인스턴스화 과정에서 업데이트 됨
- 파싱 과정
    - 문법 검사
        - ESModule은 자바스크립트 코드로 작성돼야 하므로 모듈 내용이 유효한 자바스크립트 문법을 따르는지 확인, 코드가 문법적으로 오류가 없는지 검사함
    - 토큰화
        - ESModule은 모듈의 문법이 유효하다면 모듈 내용 중 자바스크립트 코드의 키워드, 식별자, 연산자 등을 토큰으로 분리하는 토큰화를 수행함
    - 구문분석
        - 토큰화된 코드를 분석해서 문법적 구조를 이해하는 구문 분석 과정을 수행
    - 의존성 분석
        - 모듈에 다른 모듈을 가져오는 import 문이 포함된 경우, 해당 import 문을 분석해 다른 모듈에 대한 의존성을 파악

### 4.3.2.4.2 모듈 인스턴스화(**module instantiation)**

- **모듈의 export된 값들이 메모리에 할당되고 초기화되는 단계**
- 이 과정에서 모듈은 **사용 가능한 상태**가 되며, 해당 모듈에서 **import된 기능들도 메모리에 로드**됨
- 이 단계에서 모듈 레코드에 모듈 정보를 업데이트해서 export와 import문이 해석되고 필요한 값들이 메모리에 할당되어 모듈 간 참조가 연결되는데, 이러한 일련의 과정을 **의존성 해결(dependency resolution)**이라고 함
- 모듈 로드 시점에 export로 선언된 변수나 함수 등의 바인딩 정보가 먼저 설정되며 import는 해당 export의 **바인딩(메모리 참조)를 그대로** 가져옴 → 복사본이 아닌 같은 주소를 참조하므로 export 값이 변경되면 이를 import한 모든 모듈에서도 변경 내용이 반영됨
- 이 과정은 의존성 그래프를 생성하는 것과 유사함
    - export문 정보를 통해 깊이 우선 탐색을 진행하여 의존성을 끝까지 탐색하여 모든 export 연결을 완료 → 역탐색을 통해 import 항목들을 해당 export와 연결 진행
- **ESModule의 인스턴스화 과정 덕분에 CommonJS의 require() 함수와 동작 방식의 차이를 가짐**
    - require() 함수는 모듈 래퍼를 통해 exports 객체가 복사되어 전달되므로 모듈 내에서 값이 변경되어도 이를 가져온 다른 모듈에서는 변경사항이 반영되지 않음
    - ESModule에서는 export와 import가 동일한 메모리 주소를 참조하므로 변경사항이 반영됨(하지만 가져온 모듈의 값을 직접 변경할 순 없음. 원시 타입이 아닌 객체의 경우 객체 속성은 수정이 가능)

### 4.3.2.4.3 모듈 평가

- 모듈 인스턴스화 단계에서 모듈 의존성이 해결되고, export된 값들이 메모리에 할당되면 **모듈 평가단계에서 해당 모듈의 코드가 실제로 실행**됨
- **모듈 평가 과정에서 모듈 내의 모든 코드가 평가되어 실행되며 export된 값들이 최종적인 결괏값을 가짐**
- 참고
    - 이 책에서는 모듈 인스턴스화와 모듈 평가 단계를 명시적으로 분리해서 설명하는데, 이는 모듈 레코드와 실행 단계에서 어떻게 메모리가 동작하는지 설명하기 위함이며, 대부분의 경우 모듈 인스턴스화와 모듈 평가 단계를 동일시해서 설명함

### 4.3.2.5 ESModule의 순환 참조

- ESModule은 순환 참조 문제를 해결하기 위해 자바스크립트 비동기 메커니즘인 Promise를 활용함
    - 예시
        
        ```jsx
        // a.mjs
        import B from './b.mjs'
        console.log('A가 실행됨')
        export default 'A'
        
        // b.mjs
        import A from './a.mjs'
        console.log('B가 실행됨')
        export default 'B'
        
        // 결과
        // B가 실행됨
        // A가 실행됨
        ```
        
        - A모듈이 B모듈을 가져오는 과정에서 B모듈 평가가 완료되지 않았으므로 Promise가 반환됨
        - 이후 B모듈에서 A모듈을 가져올 때 A모듈은 이미 로드된 상태로 간주되므로 실제 참조를 반환
- 순환 참조 상황에서 모듈이 로드되기 전에 값에 접근하려고 하면 문제가 발생할 수 있음
    - 예시
        
        ```jsx
        // a.mjs
        import B from './b.mjs'
        export default 1
        
        // b.mjs
        import A from './a.mjs'
        console.log('B가 실행됨')
        export default A + 1; // Cannot access 'A' before initialization
        // B모듈에서 A의 값을 사용하려고 하지만, A는 아직 초기화 전이므로 오류가 발생
        ```
        
        - a.mjs 로드: a.mjs가 로드되며 B모듈을 가져오지만 B는 아직 로드 전이므로 Promise가 반환됨
        - b.mjs: b.mjs가 로드되며 A모듈을 가져오려고 하지만 A는 아직 초기화 전
        - b.mjs 평가: b.mjs가 A의 값을 사용하려고 시도하면서 A가 초기화되지 않아 오류가 발생함
- 결론 → ESModule의 순환 참조는 Promise로 해결할 수 있지만 모듈이 로드되기 전에 값을 사용하는 경우 런타임 오류가 발생할 수 있음

### 4.3.3 Node.js의 ESModule

- Node.js는 2009년 출시 이후 CommonJS 모듈 시스템을 기반으로 성장했지만, 2015년 자바스크립트 공식 표준 모듈 시스템으로 ESModule이 도입되면서 변화가 필요해졌으며, Node.js 12버전부터 공식저긍로 ESModule을 지원하기 시작했음
- Node.js가 ESModule을 지원하게 된 이유
    - 표준화 : ESModule은 브라우저 환경에서 널리 지원되고 있는 표준 모듈 시스템이므로 Node.js에서 지원 시 자바스크립트 생태계에서 서버와 클라이언트가 같은 모듈 시스템을 사용할 수 있게됨
    - 성능 최적화 :  ESModule은 트리셰이킹 등 최적화 기법을 적용하기 유리함

### 4.3.3.1 ESModule 로더

- Node.js에서 ESModule파일을 로드하는 로더의 특징(CommonJs와 비교)
    - 비동기적 로딩
        - CommonJS 로더는 동기적인 반면 ESModule 로더는 비동기로 동작하여 필요한 시점에 모듈을 로드해 다른 코드의 실행을 차단하지 않음
    - 몽키 패치 불가
        - 몽키 패치 : 프로그램의 메모리 내 소스 내용을 런타임 중 직접 변경하는 방식
        - require() 함수는 재정의가 가능한 함수이므로 같은 이름으로 재정의하여 동작을 수정할 수 있음. 하지만 import 키워드는 재정의가 불가능하므로 ESModule의 동작을 수정할 수 없음
    - 폴더 모듈 사용 불가
        - ESModule 로더는 폴더를 모듈로 사용할 수 없음
        - CommonJs에서는 폴더 경로만 지정해도 index.js 파일을 자동으로 찾지만 ESModule은 명시적으로 지정해야함
    - 확장자 명시 필요
        - ESModule 로더는 확장자 검색을 지원하지 않으므로 상대 경로나 절대 경로로 모듈 지정 시 반드시 .js, .mjs, .cjs 등 확장자를 명시해야함
    - CommonJS와의 상호운용성
        - require() 함수는 ESModule을 ㄷ로드할 수 없지만 ESModule 로더는 CommonJs 모듈을 로드할 수 있음

### 4.3.3.2 ESModule 파일 규칙

- Node.js는 다음과 같은 조건을 만족하는 파일을 ESModule로 처리함
    - .mjs 확장자로 끝나느 파일
    - 가장 가까운 상위 package.json의 “type” 필드가 “module”인 하위 .js 파일
    - —eval이나 STDIN으로실행 시 —input-type=module 플래그 사용
    - —experimental-detect-module 옵션을 사용사용한 경우
- 다음의 경우 ESModule이 아닌 CommonJS로 처리됨
    - .cjs 확장자로 끝나는 파일
    - 가장 가까운 package.json의 “type” 필드가 “commonjs”인 하위 .js 파일
    - —eval이나 STDIN으로실행 시 —input-type=commonjs 플래그 사용

### 4.3.3.3 import.meta

- Node.js는 ESModule에서 import.meta 객체를 지원함으로써 모듈의 메타데이터에 접근할 수 있는 기능을 제공함
- import.meta는 Node.js 환경에서 ESModule 내에서만 사용이 가능하며, CommonJs에서는 __dirname, __filename과 같은 변수만으로 파일 및 디렉토리 경로에 접근이 가능함

### 4.3.3.3.1 import.meta.url

- file:// 로 시작하는 절대 경로 형태의 URL을 반환해서 현재 모듈 파일의 위치를 가리키는데, 이를 통해 모듈이 로드된 정확한 경로를 확인할 수 있음
- 특히 파일 경로 기반의 작업에서 유용하며 모듈이 위치한 디렉터리, 파일의 위치를 동적으로 참조할 수 있도록 도움

### 4.3.3.3.2 import.meta.dirname과 import.meta.filename

- Node.js 21.2.0 이전 버전에서는 ESModule로 작성된 코드에서 __dirname, __filename에 해당하는 정보를 얻기위해 import.meta.url을 활용하여 복잡한 과정을 거쳐야했음
- Node.js 21.2.0 부터는 import.meta.dirname과 import.meta.filename을 지원하여 쉽게 이용할 수 있게됨

### 4.3.3.3.3 import.meta.resolve(specifier)

- import.meta.resolve()는 Node.js에서 특정 모듈 경로를 해석해 URL 형식의 절대 경로로 변환해주는 비동기 함수

### 4.3.3.2 CommonJS와의 상호운용성

- Node.js는 기존의 CommonJS 코드와 새로운 ESModule을 함께 사용할 수 있도록 호환성을 유지하는 방향으로 ESModule을 구현했음

### 4.3.3.4.1 import문과 require() 함수

- Node.js는 CommonJS와 ESModule 간 상호운용성을 지원함으로써 두 모듈 시스템이 서로의 모듈을 불러올 수 있게 함
    
    → 기존 CommonJS 코드와 ESModule 코드를 함께 사용할 수 있게 했음
    
- ESModule에서 import 문을 사용해 CommonJS 모듈을 로드할 수 있음
- CommonJS에서 ESModule을 로드하는 경우에도 import() 함수를 사용할 수 있지만 require() 함수로는 어려움(require 함수로 불러오는 건 실험적인 기능이며 계속 개발되고 있음)

### 4.3.3.4.2 CommonJS 네임스페이스

- ESModule과 CommonJS는 모듈 내보내기 방식에도 차이를 가짐
    - CommonJS는 module.exports 객체에 내보낼 대상 모듈을 포함시킴
    - ESModule은 export 문을 사용해 내보낼 대상을 결정
- CommonJS 모듈을 ESModule에서 사용하려면 기본 가져오기 사용을 권장하며, 사실 더 나은 방법은 ESModule로 작성된 의존성을 사용하는 것임

### 4.3.3.4.3 CommonJs와 ESModule의 차이점

- 개발자가 CommonJS에서 ESModule로 전환할 떄 반드시 고려하고 수정해야할 부분
    - require() 함수, exports 객체, module.exports 객체는 사용 할 수 없음
        - require() 함수가 필요하다면 module.createRequire()를 사용하여 require() 함수를 생성할 수 있음
    - __filename과 dirname 같은 CommonJS에서만 사용 가능한 변수는 사용할 수 없음
    - 애드온(addon)을 사용할 수 없음
        - 사용하려면 module.createRequire() 또는 process.dlopen을 사용해야함
    - 모듈의 절대 경로를 가져오는 require.resolve 함수를 사용할 수 없음
        - 대신 import.meta.resolve API 사용 혹은 module.createRequire()로 생성한 require을 활용해야 함
    - NODE_PATH 환경 변수를 사용할 수 없음
    - require.cache를 사용할 수 없음
        - ESModule 로더는 자체적으로 캐시를 관리하므로 require.cache에 의존하는 부분은 제거해야함
