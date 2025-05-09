# Chapter3 가상 DOM

가상 DOM(Virtual DOM)이라고 하는 주제는 리액트를 처음 공부하게 되면 누구나 접하게 되는 주제다.
리액트에서는 렌더링이라는 개념이 굉장히 중요한데, 렌더링을 할 때 가상 DOM이라는 개념을 이용하기 때문에 잘 알아보도록 하자.

## 3.3 가상 DOM 작동 방식
(편의상 가상 DOM은 VDOM, 실제 DOM은 DOM이라고 표현하겠습니다)

VDOM은 DOM의 문제점을 해결하기 위해서 만든 자바스크립트 객체로 DOM을 모델링 한 것을 의미하는데,
엄밀히는 DOM의 문제점이라기 보다는 DOM을 다루면서 생길 수 있는 까다로운 부분들을 제어하기 위해서 만들어진 개념이라고 생각한다.

보통은 브라우저 렌더링 과정, 즉 Critical Rendering Path(CRP)에서 reflow, repaint 과정을 최적화하기 위해서 VDOM이라는 개념이 탄생했다고 생각하는데,
이 외에도 브라우저마다 다양한 DOM 구현의 차이를 추상화해서 일관된 API를 제공하는 역할도 하고 있다. VDOM이라고 하는 layer가 있기 때문에 실제 브라우저가 다른 방식으로 동작해도 문제가 생기지 않는다.

### 3.3.1 리액트 엘리먼트

DOM에 추가할 element를 만들 때는 document.createElement와 같은 API를 사용하듯이, 리액트에서 사용할 React Element를 만들 때는 React.createElement 함수를 사용해서 생성한다.
DOM -> document.createElement, VDOM -> React.createElement와 같은 개념이라고 생각할 수 있다.

리액트 엘리먼트를 콘솔에 찍어보면 아래와 같은 필드들을 볼 수 있습니다.

```javascript
const element = React.createElement(
  "div",
  { className: "my-class" },
  "Hellp, world!"
);

console.log(element);
{
  $$typeof: Symbol(react.element),
  type: 'div',
  key: null,
  ref: null,
  props: {
    className: "my-class",
    children: "Hello, world!"
  },
  _owner: null,
  _stroe: {}
}
```

엘리먼트는 관련 prop이나 속성과 같이 리액트에서 ui를 표현하기 위해서 필요한 정보들을 담고 있는 자료 구조라고 볼 수 있습니다.

각 속성에 대해서 간단히 살펴보면,

- $$typeof: 객체가 유효한 리액트 엘리먼트인지 확인할 때 사용하는 특수한 심벌, XSS(Cross Site Scripting) 공격을 막기 위해서 사용하는 필드이기도 하며, 어떤 엘리먼트인지를 표현한다. (element fragment, portal, profiler, provider)
  
- type: 컴포넌트의 종류를 나타내는데 dom element일 수도 있고, 함수(함수 컴포넌트)일 수도 있다.
개발자가 작성한 리액트 컴포넌트가 렌더링되는 방식은 엘리먼트의 아래로 계속 탐색하며, 스칼라 값(숫자, 문자열, 불리언과 같이 더이상 분해되지 않는 값)을 만나면 텍스트 노드로 렌더링한다.

- ref: 기본 DOM 노드에 대한 참조, DOM을 직접 조작하는 경우에 필요하다.

### 3.3.2 가상 DOM과 실제 DOM 비교

### 3.3.3 효율적인 업데이트

## 3.4 돌아보기

## 3.5 복습하기

## 3.6 미리보기
