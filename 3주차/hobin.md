## 지연로딩

애플리케이션 규모가 커지면서 자바스크립트의 코드양도 함께 증가한다. 이는 페이지 로딩 시간이 느려지는 결과를 낳는다.

이 문제는 두 가지 방법으로 완화할 수 있다.

1. 코드 분할
2. 지연 로딩

리액트에서는 lazy 함수를 이용해서 지연로딩을 구현할 수 있다. lazy 함수는 컴포넌트가 실제로 렌더링에 필요하게 되기 전까지는 컴포넌트를 읽어들이지도 않는다.

```tsx
import { lazy, Suspense, useState } from "react";
import FakeSidebarShell from "./FakeSidebarShell";

const Sidebar = lazy(() => import("./Sidebar"));

const MyComponent = ({ initialSidebarState }) => {
  const [showSidebar, setShowSidebar] = useState(initialSidebarState);
  
  return (
  	<div>
    	<button onClick={() => setShowSidebar(!showSidebar)}>
      	사이드바 토글
      </button>
      <Suspense fallback={<FakeSidebarShell />}>
      	{showSidebar && <Sidebar />}
      </Suspense>
    </div>
  )
}
```

### Suspense를 통한 더 나은 UI 제어

컴포넌트 트리 어디든 지연 로드되고 비동기로 읽어 들이는 요소를 두면, 해당 트리의 상위 계층 어디에서든 Suspense 컴포넌트를 사용해 이러한 요소를 처리한다. 즉, 로딩 상태를 어디서 처리할지 선택할 수 있다.

```tsx
import { lazy, Suspense, useState } from "react";
import FakeSidebarShell from "./FakeSidebarShell";

const Sidebar = lazy(() => import("./Sidebar"));

const MyComponent = ({ initialSidebarState }) => {
  const [showSidebar, setShowSidebar] = useState(initialSidebarState);
  
  return (
  	<Suspense fallback={<p>로딩중...</p>}>
    	<button onClick={() => setShowSidebar(!showSidebar)}>
      	사이드바 토글
      </button>
      {showSidebar && <Sidebar />}
      <main>...</main>
    </Suspense>
  )
}
```

책에서는 Suspense를 지연로딩이 필요한 컴포넌트를 감싸는 용도로만 사용해야 한다고 하는데, 나는 그렇게 생각하지 않는다. 필요에 따라 지연로딩이 필요없는 컴포넌트도 함께 묶어서 로딩 처리가 함께 되도록 처리하는게 유용할 때가 있다고 생각한다.

## useState와 useReducer

useState는 단일 상태관리를 하는데 적합하고, useReducer는 복잡한 상태관리를 하는데 적합하다.

더 간결한 useState 대신 useReducer를 사용할 때 얻는 이점 세 가지

1. 상태 업데이트 로직을 컴포넌트에서 분리한다. 이를 통해 reducer 함수는 단독으로 테스트 가능해지고 다른 컴포넌트에서 재사용 가능해진다.
2. 상태와 상태 변경방식이 reducer로 묶여 명시적으로 바뀐다. useState의 경우 상태들이 컴포넌트 트리 여러 군데로 흩어져 관리되기 쉽다.
3. 이벤트 소스 모델처럼 활용할 수 있다. 즉, 이벤트를 발생한 순서대로 저장해두면 그 이벤트들을 가지고 시간여행 디버깅, 실행 취소, 재실행, 낙관적 업데이트, 사용자 행동에 대한 분석추적 등에 사용할 수 있다. 
   **이벤트 소스 모델**: 상태(state)의 변화 자체를 저장하지 않고, 상태를 변화시킨 이벤트의 순서를 저장하는 아키텍처 패턴

### Immer와 편의성

리액트 상태는 불변성을 유지해야 해서, 중첩된 값을 변경하려면 해당 필드까지의 모든 상위 객체를 복사해야 한다. 이 과정이 매우 귀찮고 실수할 여지가 많다. immer는 귀찮은 객체 복사 과정을 알아서 해주며 개발자는 특정 필드만 업데이트하는 간결한 방식으로 상태 업데이트를 할 수 있게된다.

```tsx
import { useImmerReducer } from "use-immer";

const initialState = {
  user: {
    name: "John Doe",
    age: 28,
    address: {
      city: "New York",
      country: "USA",
    },
  },
}

const reducer = (draft, action) => {
  switch (action.type) {
    case "updateName":
      draft.user.name = action.payload;
      break;
    case "updateCity":
      draft.user.address.city = action.payload;
      break;
    default:
      break;
  }
}

const MyComponent = () => {
  const [state, dispatch] = useImmerReducer(reducer, initialState);
  ...
}
```

## 목요일까지 되는대로 말아보는 추가 자료

- 코드 분할의 여러가지 방식
- Suspense 를 바라보는 다양한 시각과 활용 예시
- 리듀서를 사용하면서 자연스럽게 생기는 이벤트 기반 사고 방식에 대한 예시

