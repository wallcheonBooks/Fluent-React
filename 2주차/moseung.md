# 2장 JSX
JSX 는 ``Javascript + eXtension``의 줄임말. 즉 Javascript의 Extension임 마치 XML(eXtensible Markup Language)처럼.
## 2.2 JSX의 장점
JSX는 Javascript를 사람이 더 이해하기 쉬운 코드로 만들 수 있게 해주는 도구.
### 모승 생각
이를 관심사의 혼합이라고 생각합니다. 순수 HTML, JS, CSS를 각각의 파일에서 수평적으로 관리하는것이 아니라 수직적으로 관리하기 위한것.
<br>
![image](https://github.com/user-attachments/assets/7eff2e27-c789-4e5a-8e81-17386a46331b)
<br>

[수평 관심사와 수직 관심사](https://velog.io/@teo/separation-of-concerns-of-frontend#%EC%9B%B9-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%EA%B0%80-%EA%B0%80%EC%A0%B8%EC%98%A8-%EC%83%88%EB%A1%9C%EC%9A%B4-%ED%8C%A8%EB%9F%AC%EB%8B%A4%EC%9E%84%EC%9D%98-%EB%B3%80%ED%99%94-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%ED%9D%AC%EB%A7%9D%ED%8E%B8)

## 2.4 내부 동작
### 2.4.1 자바스크립트의 컴파일
자바스크립트는 3간계 과정을 거칩니다. ``토큰화``, ``구문 분석``, ``코드 생성``
1. 토큰화 : 문자열을 의미있는 토큰으로 분해하는것. 토크나이저가 상태를 갖고 있고, 각 토큰의 자식의 부모나 자식에 대한 상태를 갖고 있는것을 렉서라고 부름.
2. 구문 분석 : 토큰을 가져와 구문 트리로 변환하는 과정.<img width="667" alt="스크린샷 2025-05-06 오후 9 01 20" src="https://github.com/user-attachments/assets/ce8071cb-cf96-4ae3-ab29-21d012aeb9e1" />
```tsx
// 이렇게 작성하면 syntax error가 나는건 이때임
const a 1;
```
3. 코드 생성 : 컴파ㅣㄹ러가 추상 구문 트리에서 기계어를 생성하는 과정. (최신 브라우저 엔진에서 성능 최적화를 하는건 이때임, ex: Inline Caching, Dead Code Elimination, Constant Folding)

### 알쓸신잡
🧩 1. 인터프리터 (Interpreter)
정의:
한 줄씩 코드를 읽고 바로 실행하는 방식. 처음부터 전체 코드를 기계어로 번역하지 않고, 실행 도중 해석함.

특징:

빠른 시작 시간

느린 실행 속도 (반복문 등에서 매번 해석)

JS 예시:

초기 자바스크립트 엔진 (예: SpiderMonkey 초기 버전)
자바스크립트는 원래 인터프리터 언어로 설계됨. HTML 안에서 실행되기 위해 빠른 파싱이 중요했기 때문.

🧩 2. JIT 컴파일러 (Just-In-Time)
정의:
코드를 인터프리팅 하면서, 자주 실행되는 코드를 런타임 도중 기계어로 변환해 캐싱해서 빠르게 실행하는 방식. 인터프리터와 컴파일러의 중간 단계.

특징:

런타임 분석 + 최적화

초기엔 느리지만 반복될수록 빨라짐

JS 예시:

V8 (Chrome), SpiderMonkey (Firefox), JavaScriptCore (Safari)
이들은 전부 JIT 기반 엔진을 사용해 인터프리팅과 함께 반복되는 코드를 컴파일함.
특히 V8은 TurboFan + Ignition 구조를 통해 코드 실행 중 최적 경로를 찾고 native 코드를 생성함.

🧩 3. 네이티브 컴파일러 (Native Compiler)
정의:
소스코드를 목표 플랫폼(OS+CPU)의 기계어로 바로 컴파일하는 방식. 빌드하면 실행파일(EXE 등)이 나오는 구조.

특징:

빌드 타임에 플랫폼에 맞게 최적화

실행 속도 빠름

이식성 낮음

JS 관련 예시:

Bun
JavaScript로 작성된 코드를 실행 가능한 네이티브 바이너리로 빌드할 수 있음. Zig로 작성된 런타임이 네이티브 성능을 목표로 함.

esbuild / swc
TS/JS를 매우 빠르게 native binary로 처리하는 빌드 도구 (Go, Rust로 작성됨)

🧩 4. 크로스 컴파일러 (Cross Compiler)
정의:
하나의 플랫폼에서 다른 플랫폼용 실행 파일을 만드는 컴파일러. 예: Windows에서 Raspberry Pi용 Linux 바이너리를 만드는 것.

특징:

타겟 플랫폼에 실제 환경 없어도 빌드 가능

IoT, 임베디드, 모바일에서 많이 사용

JS 관련 예시:

React Native + Hermes
React Native 앱을 안드로이드나 iOS 네이티브 앱으로 빌드할 때, JavaScript 코드를 크로스 컴파일해 Hermes bytecode로 만든 후 해당 플랫폼의 네이티브 코드와 연동함.

AssemblyScript + WebAssembly
TypeScript 스타일의 코드 → WASM으로 컴파일 (브라우저나 모바일에서 실행)

### 2.4.2 JSX로 자바스크립트 구문 확장하기
JSX는 브라우저가 이해할 수 있는 언어가 아니니 이를 컴파일해줘야한다.
<br>

### 모승생각
이전에 멘토링할때 썻던 컨텐츠가 있어서 그림 첨부. tsx -> jsx -> js<br>
App.tsx → [Babel/SWC] → App.js → [Webpack] → bundle.js
<br>

<img width="293" alt="image" src="https://github.com/user-attachments/assets/f7d8c1f7-f3a7-46e2-904f-7ab462414b6d" />
[최신 모듈 번들러(feat. NextJS, React)](https://velog.io/@endmoseung/modulebundlerevolution#3-%EC%B5%9C%EC%8B%A0-%EB%AA%A8%EB%93%88-%EB%B2%88%EB%93%A4%EB%9F%ACfeat-nextjs-react)

## 2.8 복습하기
1. JSX는 뭐고 장단점은 무엇인지
2. JSX와 HTML의 차이점은 뭔지
3. 텍스트 문자열은 어떻게 기꼐어가 되는지
4. JSX 표현식이란 뭐고 어떤 좋은점이 있는지
