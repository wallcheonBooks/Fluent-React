
# React Fiber 아키텍처 정리


**React 16~현재(React 18, 19)**까지의 모든 버전은 Fiber를 기반으로 동작

Fiber는 단순한 렌더링 알고리즘이 아니라, React의 실행 모델 그 자체다. 작업을 분할하고, 우선순위 기반으로 스케줄링하고, 상태를 관리하는 등 React의 “두뇌” 역할을 한다.

React 18에서 도입된 Concurrent Mode 역시 Fiber 구조의 스케줄러 확장 위에서 구현되었다.


이전 React의 **동기적 렌더링 방식**은 UI가 복잡해질수록 병목 현상이 발생하고, 렌더링 중 오류가 발생하면 전체 애플리케이션이 중단되는 등 여러 한계를 가짐

이를 극복하기 위해 React 팀은 **Fiber 아키텍처**를 설계하였고 아래와 같은 목표를 가짐

- 렌더링을 작업 단위로 나누어 처리
- 각 작업에 우선순위를 부여하여 스케줄링 가능하게 함
- 렌더링 도중에도 작업 중단 및 재개 가능
- 에러에 강한 구조 제공 (Error Boundary)
- 애니메이션, 레이아웃, 제스처 같은 실시간성 UI를 위한 기반 마련

---

## 기존 React의 문제점 vs Fiber의 개선

### 🔴 기존 React의 한계

- 모든 렌더링 작업을 한 번에 처리함 (동기 처리)
- 렌더링 중 오류가 발생하면 전체 앱이 중단됨
- 에러 발생 위치 파악이 어려워 디버깅이 까다로움

### 🟢 Fiber의 도입으로 인한 변화

- 렌더링 작업을 세분화하여 처리 (incremental rendering)
- 작업 단위에 우선순위를 부여해 중요도 높은 UI 먼저 렌더링 가능
- 오류 경계를 통해 개별 컴포넌트 단위의 예외 처리 가능
- SSR, Portals, Fragments 등 다양한 기능 도입

---

## 오류 경계 (Error Boundaries)

오류 경계(Error Boundary)는 컴포넌트 단위에서 발생한 오류를 포착하고 fallback UI를 렌더링할 수 있는 구조

```jsx
class ErrorBoundary extends React.Component {
  componentDidCatch(error, info) {
    // 오류 로깅 처리
  }

  render() {
    if (this.state.hasError) {
      return <FallbackComponent />;
    }
    return this.props.children;
  }
}
```

- Fiber는 컴포넌트 트리를 순회하며 오류가 발생하면 가장 가까운 Error Boundary를 찾아 fallback UI를 렌더링
- try-catch 문법 대신 **선언적 에러 처리** 방식을 따름

---


### Incremental Rendering

- 렌더링 작업을 프레임 단위로 나눠 실행
- 긴 렌더링 작업도 사용자 입력 등을 방해하지 않도록 처리
- 실제로는 `requestIdleCallback`과 같은 브라우저 API를 활용해 빈 시간에 렌더링을 수행함

### Portals

```jsx
ReactDOM.createPortal(child, domNode);
```

- 모달, 팝오버처럼 DOM 계층 밖에 있어야 하는 UI를 렌더링할 때 유용
- React 트리상 위치는 유지하면서, 실제 DOM 위치는 분리 가능

### Fragments

```jsx
<>
  <Item />
  <Item />
</>
```

React 컴포넌트는 하나의 루트 요소만 반환할 수 있었다. 그렇기에 불필요한 div 요소로 컴포넌트를 감싸서 DOM에 불필요한 노드를 추가하게 됨

fiber 도입하고 fragment가 도입됨으로써 불필요한 div 요소를 사용하지 않음

fragment는 fiber 노드로 표현되지만 실제 dom 노드로는 렌더링 되지 않음.

---

### Fiber 구조 상세


Fiber는 컴포넌트에 대한 정보를 담고 있는 **가상의 스택 프레임**

일반적으로 JS의 콜 스택은 중단이나 우선순위 변경이 불가능하지만, Fiber는 렌더링 작업을 **명시적으로 관리**할 수 있게 만들어 줌

### Fiber 노드의 속성들

| 속성 | 설명 |
|------|------|
| `type` | 컴포넌트 또는 DOM 노드의 타입 |
| `key` | 리스트 항목 식별용, diff 알고리즘에서 핵심 |
| `child`, `sibling`, `return` | 자식, 형제, 부모 노드를 참조 |
| `pendingProps` | 새 렌더링에 사용할 데이터 |
| `memoizedProps` | 이전 렌더링에서 사용한 데이터 |
| `alternate` | current ↔ workInProgress 를 연결 |
| `effectTag` | commit 단계에서 실행할 작업 표시 |

### alternate 구조

```js
currentFiber.alternate === workInProgressFiber;
```

React는 항상 두 개의 Fiber 트리를 유지

- `current`: 현재 렌더링 상태
- `workInProgress`: 다음 렌더링을 위한 준비 상태

모든 변경 사항은 `workInProgress`에서 수행되며, 커밋 시에만 `current`에 반영

---

### Fiber 작동 원리

#### Render Phase (비동기 가능)

```js
function workLoop() {
  while (nextUnitOfWork !== null) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
  }
}
```

- `beginWork(fiber)` → 현재 작업에 필요한 props 계산, 자식 fiber 생성
- `completeUnitOfWork(fiber)` → 자식이 없는 경우 위로 올라가서 작업 마무리

#### Commit Phase (동기)

- DOM 변경, ref 설정/해제, 라이프사이클 호출 등이 발생
- 효과(effect)만 모아서 별도로 선형 리스트로 관리 → 효율적 적용

---

### Reconciliation 알고리즘 (재조정)

React는 diffing 최적화를 위해 다음과 같은 원칙을 따름

- 엘리먼트 타입이 다르면 → 통째로 교체
- 같은 타입이면 → props만 비교 후 재사용
- key 속성이 안정적이지 않으면 불필요한 unmount 발생

이로 인해 `key` 속성은 리스트 렌더링에서 반드시 유일하게 지정되어야 한다.

---

## 그래서 React 19 에서는?

- React Compiler (React Forget): useMemo, useCallback 없이도 성능 최적화를 컴파일 타임에 자동화하는 기능.

- Asset Loading, Actions, Form State Sync 등 서버와 클라이언트를 연결하는 "React Server Components"와의 통합 강화.

- Transition, Suspense, useDeferredValue, useTransition 등의 체계적 활용 → 모두 Fiber의 우선순위 기반 스케줄링 모델 없이는 불가능

즉, Fiber는 계속 사용되며, 새로운 기능들이 Fiber 위에서 ‘실행 방식’을 발전시키는 방향으로 설계되고 됨.

단지 Virtual DOM 을 지양하고있따.

이는 Virtual DOM 의 단순한 개념으로 많은 사람들이 오해를 하고 있다. 어떤 오해? Virtual DOM 은 렌더 결과 객체를 의미하고, fiber 은 렌더링 처리 단위(스케줄링 + 상태 관리+ 효과추적)라고.

그래서 react 팀은  **"React Element → Fiber Node → Commit"** 을 강조하기 위해서 Virtual DOM 이라는 개념을 제거하고자했따.


## 📚 참고 자료

- [React v16.0 공식 블로그](https://legacy.reactjs.org/blog/2017/09/26/react-v16.0.html#better-error-handling)
- [Facebook Engineering: React 16 내부 구현](https://engineering.fb.com/2017/09/26/web/react-16-a-look-inside-an-api-compatible-rewrite-of-our-frontend-ui-library/)
- [GitHub: React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture?tab=readme-ov-file)
- [React 공식문서: Reconciliation](https://legacy.reactjs.org/docs/reconciliation.html)
- [Angular.Love: Inside Fiber 분석글](https://angular.love/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react)
