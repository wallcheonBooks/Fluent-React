# 5.5. 강력한 패턴

소프트웨어 디자인 패턴이 중요한 이유
1. 재사용성 : 흔히 겪는 문제에 대한 재사용 가능한 해결책을 제공해 소프트웨어 개발 시간과 노력을 절약할 수 있음
2. 표준화 : 문제를 해결하는 표준 방법을 제공하므로 개발자끼리 서로 잘 이해하고 소통할 수 있음
3. 유지 보수성 : 유지 보수 및 수정이 쉬운 코드로 구조화하는 방법을 지원해 소프트웨어 시스템의 수명을 향상시킬 수 있음
4. 효율성 : 일반적인 문제에 효율적인 해결책을 제공해 소프트웨어 시스템의 성능을 개선할 수 있음

"좋은 와인처럼 오래도록 잘 유지되게 코드를 작성하는 법을 찾는 데 도움이 됨"
제어를 고려함. 즉, 사용자에게 얼마나 많은 권한을 부여할지, 프로그램이 얼마나 많이 처리할지 고려함

## 5.5.1 프레젠테이션/컨테이너 컴포넌트

- 프레젠테이션 컴포넌트 : UI를 렌더링
- 컨테이너 컴포넌트 : UI 상태를 처리

- 단일 책임 원칙 때문에 매우 유용
- 관심사를 분리해 모듈화, 재사용, 테스트를 가능하게 함

```tsx
// PresentationalCounter.js
import React from 'react';

// 프레젠테이션 컴포넌트: UI 표시만 담당
const PresentationalCounter = ({ count, onIncrement, onDecrement }) => {
  return (
    <div className="counter">
      <h2>카운터</h2>
      <div className="counter-display">{count}</div>
      <div className="counter-controls">
        <button onClick={onDecrement} className="counter-button">-</button>
        <button onClick={onIncrement} className="counter-button">+</button>
      </div>
    </div>
  );
};

export default PresentationalCounter;

// ContainerCounter.js
import React, { useState } from 'react';
import PresentationalCounter from './PresentationalCounter';

// 컨테이너 컴포넌트: 상태 관리 및 로직 담당
const ContainerCounter = () => {
  // 상태 관리
  const [count, setCount] = useState(0);

  // 이벤트 핸들러
  const handleIncrement = () => {
    setCount(count + 1);
  };

  const handleDecrement = () => {
    setCount(count - 1);
  };

  // 프레젠테이션 컴포넌트에 props 전달
  return (
    <PresentationalCounter 
      count={count} 
      onIncrement={handleIncrement}
      onDecrement={handleDecrement}
    />
  );
};

export default ContainerCounter;
```

### 프레젠테이션 컴포넌트 (Presentational Components)

- UI 렌더링에 집중
- 데이터를 props로 받아 표시
- 상태를 거의 갖지 않음 (UI 상태 제외)
- 주로 함수형 컴포넌트로 구현
- 데이터가 어디서 오는지, 어떻게 변하는지 모름

### 컨테이너 컴포넌트 (Container Components)

- 데이터 처리와 상태 관리에 집중
- 데이터 소스와 통신
- 상태를 관리하고 변경
- 프레젠테이션 컴포넌트에 데이터와 콜백 제공
- 일반적으로 스타일링 코드 포함하지 않음

1. 두 컴포넌트 유형의 명확한 역할 구분
    - 프레젠테이션 컴포넌트: UI에만 집중
    - 컨테이너 컴포넌트: 상태 관리와 비즈니스 로직에 집중
2. 이 패턴이 가져오는 구체적인 장점들
    - 관심사 분리로 코드 유지보수성 향상
    - UI 컴포넌트 재사용성 증가
    - 개발 역할 분담 용이성
3. 단점들과 현대적 대안
    - 보일러플레이트 코드와 파일 수 증가 문제
    - Props 드릴링 이슈
    - React Hooks를 통한 대안 접근법

이 패턴은 Dan Abramov에 의해 널리 알려졌으며, 특히 Redux와 함께 사용될 때 효과적이었습니다. 그러나 React Hooks와 Context API의 등장으로 로직 분리와 상태 관리의 새로운 패러다임이 도입되어, 이 전통적인 패턴의 사용은 감소하는 추세입니다.

최신 React 개발에서는 Custom Hooks를 통해 로직을 분리하고, 필요한 경우 Context API로 상태를 공유하는 방식이 더 간결하고 유연하게 평가되고 있습니다.

```tsx
// Counter.js - 로직과 UI가 통합된 단일 컴포넌트
import React from 'react';
import { useCounter } from './useCounter';

const Counter = ({ initialCount = 0 }) => {
  // 커스텀 훅에서 상태와 메서드 가져오기
  const { count, increment, decrement, reset } = useCounter(initialCount);
  
  return (
    <div className="counter">
      <h2>카운터</h2>
      <div className="counter-display">{count}</div>
      <div className="counter-controls">
        <button onClick={decrement} className="counter-button">-</button>
        <button onClick={increment} className="counter-button">+</button>
        <button onClick={reset} className="counter-button">리셋</button>
      </div>
    </div>
  );
};

export default Counter;
```

## 5.5.2 고차 컴포넌트

위키피디아 정의
> 수학 및 컴퓨터 과학에서 고차 함수는 다음 중 하나 이상을 수행하는 함수이다. 하나 이상의 함수를 인수로 취한다. 함수를 결과로 반환한다.

다른 컴포넌트를 인수로 받아 두 컴포넌트의 합성 결과인 새로운 컴포넌트를 반환하는 컴포넌트.
여러 컴포넌트에서 공유하는 동작을 반복 작성하고 싶지 않을 때 사용.

기본 구조
```tsx
// 기본 HOC 패턴
const withSomething = (WrappedComponent) => {
  return function EnhancedComponent(props) {
    // 추가 기능이나 데이터 로직
    const extraData = { value: 'something' };
    
    // 원본 컴포넌트에 추가 props 전달
    return <WrappedComponent {...props} extraData={extraData} />;
  };
};

// 사용 방법
const EnhancedComponent = withSomething(MyComponent);
```

실제 예시
```tsx
const withAuth = (WrappedComponent) => {
  return function WithAuth(props) {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    const [user, setUser] = useState(null);
    
    useEffect(() => {
      // 사용자 인증 상태 확인 로직
      const checkAuth = async () => {
        try {
          const userData = await authService.getCurrentUser();
          setUser(userData);
          setIsAuthenticated(true);
        } catch (error) {
          setIsAuthenticated(false);
          setUser(null);
        }
      };
      
      checkAuth();
    }, []);
    
    // 인증되지 않은 경우 처리
    if (!isAuthenticated) {
      return <Navigate to="/login" />;
    }
    
    // 인증된 경우 원본 컴포넌트 렌더링
    return <WrappedComponent {...props} user={user} />;
  };
};

// 사용 방법
const ProtectedDashboard = withAuth(Dashboard);
```

## 고차 컴포넌트 합성

여러 HOC를 조합하여 컴포넌트에 여러 기능을 추가할 수 있습니다.

중첩 방식 (기본)
```tsx
// 여러 HOC를 중첩하여 적용
const EnhancedComponent = withAuth(withData(withTheme(BaseComponent)));
```
이 방식은 코드가 안쪽에서 바깥쪽으로 읽혀 직관적이지 않을 수 있습니다.

명시적 변수 할당 방식
```tsx
// 단계별로 명시적 변수 할당
const ComponentWithTheme = withTheme(BaseComponent);
const ComponentWithData = withData(ComponentWithTheme);
const ComponentWithAuth = withAuth(ComponentWithData);
```
이 방식은 가독성이 좋지만 코드가 장황해질 수 있습니다.

## Compose 유틸

Compose 함수는 여러 함수를 인자로 받아 단일 함수로 합성하는 유틸리티입니다. HOC 합성에 특히 유용합니다.

Compose로 HOC 합성하기

```tsx
// compose를 사용한 HOC 합성
const enhance = compose(
  withAuth,      // 세 번째로 적용
  withData,      // 두 번째로 적용
  withTheme      // 첫 번째로 적용
);

// enhance는 이제 세 HOC를 합성한 하나의 함수
const EnhancedComponent = enhance(BaseComponent);
```


# 고차 컴포넌트(HOC)와 훅(Hooks) 비교

|측면|고차 컴포넌트 (HOC)|훅 (Hooks)|
|---|---|---|
|**코드 재사용**|컴포넌트 로직을 래핑하여 재사용<br>여러 컴포넌트에 동일한 기능 추가 가능<br>컴포넌트 계층 구조 변경됨|로직만 추출하여 재사용<br>컴포넌트 내에서 직접 호출<br>컴포넌트 계층 구조 변경 없음|
|**렌더링 조직**|컴포넌트 트리에 추가 계층 생성<br>"래퍼 지옥(wrapper hell)" 발생 가능<br>렌더링 출력 직접 조작 가능|추가 컴포넌트 계층 없음<br>더 얕은 컴포넌트 트리 유지<br>렌더링 결과 간접 조작 (상태/효과 통해)|
|**프롭 조작**|props 추가/수정/필터링 가능<br>render props 패턴 지원<br>prop 이름 충돌 위험 존재|props 직접 조작 불가<br>컴포넌트 내에서 props 사용만 가능<br>props 이름 충돌 없음|
|**상태 관리**|래핑된 컴포넌트와 별도 상태 관리<br>상태 공유 시 명시적 props 필요<br>상태 로직 캡슐화|컴포넌트 내에서 직접 상태 관리<br>커스텀 훅으로 상태 로직 추출<br>더 직관적인 상태 접근|
|**수명 주기 관리**|전체 수명 주기 메서드 사용 가능<br>명확한 마운트/언마운트 처리<br>깊은 컴포넌트 제어|useEffect로 수명 주기 시뮬레이션<br>의존성 배열로 효과 제어<br>정리(cleanup) 함수로 언마운트 처리|
|**합성 용이성**|compose 함수 필요<br>중첩된 HOC 순서가 중요<br>복잡한 합성 구조|여러 훅 연속 호출로 간단히 합성<br>호출 순서 중요하나 직관적<br>간결한 합성 구문|
|**테스트 용이성**|컴포넌트와 HOC 분리 테스트 가능<br>래핑된 컴포넌트 테스트 복잡<br>모킹 필요 경우 많음|커스텀 훅 독립적 테스트 용이<br>renderHook 유틸리티 사용 가능<br>컴포넌트와 훅 분리 테스트|
|**타입 안정성**|제네릭 타입으로 복잡한 구현 필요<br>Props 타입 추론 어려움<br>타입 정의 오버헤드 발생|간단하고 직관적인 타입 정의<br>타입 추론 잘 작동<br>더 적은 타입 코드 요구|



## 5.5.3 렌더 프롭

Render Props는 React에서 컴포넌트 간에 코드를 공유하기 위한 테크닉으로, 함수를 컴포넌트의 props로 전달하여 컴포넌트가 무엇을 렌더링할지 동적으로 결정하게 하는 패턴입니다.

## 기본 개념

Render Props 패턴의 핵심은 컴포넌트에 함수를 props로 전달하고, 이 함수가 해당 컴포넌트 내에서 호출되어 무엇을 렌더링할지 결정하는 것입니다. 이 함수는 일반적으로 컴포넌트의 내부 상태나 로직에 접근할 수 있습니다.

```tsx
// Render Props 패턴의 기본 구조
<DataProvider render={data => (
  <div>데이터: {data.name}</div>
)}/>
```


## Render Props의 장점

1. **코드 재사용성**: 컴포넌트 간에 상태나 로직을 쉽게 공유할 수 있습니다.
2. **관심사 분리**: 데이터 로직과 UI를 명확히 분리할 수 있습니다.
3. **유연성**: 컴포넌트의 렌더링 방식을 외부에서 결정할 수 있어 유연합니다.
4. **동적 렌더링**: 런타임에 렌더링 방식을 바꿀 수 있습니다.
5. **데이터 추상화**: 데이터를 어떻게 가져오는지 숨기고 필요한 데이터만 노출합니다.

## Render Props의 단점

1. **콜백 지옥**: 여러 Render Props 컴포넌트를 중첩하면 코드가 복잡해질 수 있습니다.
2. **렌더링 효율성**: 매 렌더링마다 새로운 함수가 생성될 수 있어 최적화가 필요할 수 있습니다.
3. **코드 가독성**: 복잡한 구조에서는 데이터 흐름 추적이 어려울 수 있습니다.
4. **함수형 자식에 대한 TypeScript 지원**: 타입 정의가 복잡할 수 있습니다.

## HOC vs Render Props vs Hooks

세 패턴 모두 코드 재사용을 위한 패턴이지만 접근 방식이 다릅니다:

|측면|Render Props|HOC (고차 컴포넌트)|Hooks|
|---|---|---|---|
|**구현 방식**|함수를 props로 전달하여 렌더링 결정|컴포넌트를 래핑하여 새 컴포넌트 반환|함수 호출로 상태와 로직 사용|
|**코드 가독성**|중첩 시 복잡해질 수 있음|래퍼 지옥 가능성|간결하고 읽기 쉬움|
|**컴포넌트 계층**|계층 추가 없음|추가 컴포넌트 계층 생성|계층 추가 없음|
|**합성**|함수 합성으로 조합 가능|compose 함수로 조합|훅 호출 순서로 조합|
|**디버깅**|컴포넌트 구조 명확|래핑 구조로 추적 어려움|React DevTools에서 훅 검사|
|**사용 시기**|동적 렌더링이 필요할 때|컴포넌트 변환이 필요할 때|함수형 컴포넌트에서 상태/로직 재사용 시|

## 실제 사용 예: Render Props로 구현한 토글 기능

jsx

```jsx
class Toggle extends React.Component {
  state = { on: false };
  
  toggle = () => {
    this.setState(prevState => ({ on: !prevState.on }));
  };
  
  render() {
    return this.props.children({
      on: this.state.on,
      toggle: this.toggle
    });
  }
}

// 사용 예시
function App() {
  return (
    <Toggle>
      {({ on, toggle }) => (
        <div>
          <button onClick={toggle}>
            {on ? '켜짐' : '꺼짐'}
          </button>
          <div>{on ? '내용이 보입니다' : '내용이 숨겨집니다'}</div>
        </div>
      )}
    </Toggle>
  );
}
```

## 같은 기능의 Hook 구현 비교

jsx

```jsx
// 커스텀 훅으로 구현
function useToggle(initialState = false) {
  const [on, setOn] = useState(initialState);
  
  const toggle = useCallback(() => {
    setOn(prevState => !prevState);
  }, []);
  
  return [on, toggle];
}

// 사용 예시
function App() {
  const [on, toggle] = useToggle();
  
  return (
    <div>
      <button onClick={toggle}>
        {on ? '켜짐' : '꺼짐'}
      </button>
      <div>{on ? '내용이 보입니다' : '내용이 숨겨집니다'}</div>
    </div>
  );
}
```

Render Props 패턴은 React Hooks가 도입되기 전에 널리 사용되었으며, 특히 클래스 컴포넌트에서 코드 재사용을 위한 주요 패턴 중 하나였습니다. 현재는 Hooks가 더 간결한 대안을 제공하지만, 동적 렌더링이 필요한 특수한 경우에는 여전히 Render Props 패턴이 유용하게 사용됩니다.



## 5.5.4 제어 프롭

제어 컴포넌트와 제어 프롭 패턴은 React에서 컴포넌트의 상태와 동작을 제어하는 중요한 패턴입니다.

## 제어 컴포넌트(Controlled Components)

제어 컴포넌트는 React에서 폼 요소(input, textarea, select 등)의 상태를 React 컴포넌트의 state로 관리하는 패턴입니다.

### 기본 개념

1. **상태 관리**: 폼 요소의 값이 React 컴포넌트의 state에 의해 제어됨
2. **단일 진실 공급원(Single Source of Truth)**: 컴포넌트 상태가 UI 상태의 유일한 출처
3. **이벤트 핸들링**: 사용자 입력에 따라 state를 업데이트하는 핸들러 함수 사용

### 제어 컴포넌트 예시

jsx

```jsx
import React, { useState } from 'react';

function ControlledForm() {
  const [name, setName] = useState('');
  
  const handleChange = (event) => {
    setName(event.target.value);
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();
    alert(`제출된 이름: ${name}`);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <label>
        이름:
        <input 
          type="text" 
          value={name} 
          onChange={handleChange} 
        />
      </label>
      <button type="submit">제출</button>
    </form>
  );
}
```

이 예시에서 `input` 요소의 `value`는 React의 `name` 상태에 의해 제어되며, 사용자가 입력할 때마다 `handleChange` 함수가 호출되어 상태를 업데이트합니다.

## 제어 프롭 패턴(Controlled Props Pattern)

제어 프롭 패턴은 제어 컴포넌트 개념을 확장하여, 재사용 가능한 컴포넌트의 내부 상태와 동작을 외부(부모 컴포넌트)에서 제어할 수 있게 하는 패턴입니다.

### 기본 개념

1. **상태 위임**: 컴포넌트의 내부 상태를 부모 컴포넌트로 위임
2. **동작 제어**: 컴포넌트의 동작을 외부에서 제어 가능하게 함
3. **양방향 바인딩**: 값(value)과 변경 핸들러(onChange) props 쌍을 통한 상태 관리

### 제어 프롭 패턴 예시: 커스텀 토글 컴포넌트

jsx

```jsx
// Toggle.js - 제어 프롭 패턴을 사용한 재사용 가능한 컴포넌트
function Toggle({ on, onChange }) {
  const handleClick = () => {
    // 변경 핸들러를 통해 부모 컴포넌트에 알림
    onChange(!on);
  };
  
  return (
    <button 
      className={`toggle ${on ? 'toggle--on' : 'toggle--off'}`}
      onClick={handleClick}
    >
      {on ? 'ON' : 'OFF'}
    </button>
  );
}

// App.js - 부모 컴포넌트에서 Toggle 컴포넌트 사용
function App() {
  const [isOn, setIsOn] = useState(false);
  
  return (
    <div>
      <Toggle on={isOn} onChange={setIsOn} />
      <p>현재 상태: {isOn ? '켜짐' : '꺼짐'}</p>
    </div>
  );
}
```

이 예시에서 `Toggle` 컴포넌트는 자체 상태를 관리하지 않고, `on`과 `onChange` props를 통해 상태와 동작이 부모 컴포넌트에 의해 제어됩니다.

## 제어 프롭 패턴의 장점

1. **상태 관리 일원화**: 애플리케이션 상태를 상위 컴포넌트에서 관리할 수 있음
2. **예측 가능성**: 컴포넌트의 동작과 상태가 명확하게 제어됨
3. **재사용성**: 다양한 상황에서 동일한 컴포넌트를 다른 동작으로 재사용 가능
4. **테스트 용이성**: 컴포넌트의 동작을 외부에서 제어할 수 있어 테스트가 쉬움
5. **상태 지속성**: 컴포넌트가 언마운트되어도 상태가 상위 컴포넌트에 유지됨

## 제어 프롭 패턴의 단점

1. **보일러플레이트 코드**: 상태와 핸들러를 항상 전달해야 함
2. **복잡성 증가**: 여러 컴포넌트 간 상태 동기화가 필요할 때 복잡해질 수 있음
3. **성능 이슈**: 상태 변경 시 불필요한 리렌더링이 발생할 수 있음
4. **동시 제어 충돌**: 내부/외부 상태 관리가 충돌할 가능성

## 제어 프롭 패턴과 다른 패턴 비교

### 제어 프롭 vs 비제어 컴포넌트

|측면|제어 프롭|비제어 컴포넌트|
|---|---|---|
|**상태 관리**|부모 컴포넌트에서 관리|컴포넌트 내부에서 관리|
|**데이터 흐름**|양방향 (props + 콜백)|단방향 (내부 → 외부)|
|**유연성**|높음 (외부에서 상태 접근/조작)|제한적 (외부에서 접근 어려움)|
|**사용 복잡성**|상대적으로 복잡|단순|
|**적합한 상황**|폼 유효성 검사, 상태 지속 필요 시|단순한 UI 요소, 독립적 동작|

### 제어 프롭 vs Hooks

|측면|제어 프롭|Hooks|
|---|---|---|
|**구현 방식**|props로 상태와 핸들러 전달|함수 호출로 상태와 로직 사용|
|**재사용성**|컴포넌트 레벨|로직 레벨|
|**캡슐화**|컴포넌트 인터페이스 노출|내부 로직 추상화|
|**조합**|컴포넌트 조합|훅 조합|
|**적합한 상황**|UI + 동작 패키징|순수 로직 재사용|


제어 프롭 패턴은 컴포넌트 인터페이스를 명확히 정의하고 재사용성을 높이는 강력한 패턴으로, 특히 복잡한 폼 요소나 상호작용이 많은 UI 컴포넌트를 개발할 때 유용합니다.


## 5.5.5 프롭 컬렉션

프롭 컬렉션은 React 컴포넌트 디자인 패턴으로, 자주 함께 사용되는 props 그룹을 미리 정의된 객체로 제공하여 개발자 경험을 향상시키고 코드의 재사용성을 높이는 기법입니다.

## 기본 개념

프롭 컬렉션 패턴의 핵심 개념은 다음과 같습니다:

1. **관련 props 그룹화**: 함께 사용되는 props를 객체로 묶어 제공
2. **편의성 향상**: 여러 props를 일일이 전달하는 대신 스프레드 연산자(...)로 간편하게 적용
3. **일관성 유지**: 관련 컴포넌트에 일관된 props 세트 적용
4. **접근성 향상**: 접근성 관련 속성을 기본 제공

## 기본 구현 예시

jsx

```jsx
function useToggle() {
  const [on, setOn] = useState(false);
  
  const toggle = () => setOn(!on);
  
  // 프롭 컬렉션: 토글 버튼에 필요한 props 모음
  const togglerProps = {
    'aria-pressed': on,
    onClick: toggle
  };
  
  // 프롭 컬렉션: 토글 상태를 표시하는 요소에 필요한 props 모음
  const displayProps = {
    'aria-live': 'polite',
    'aria-atomic': true
  };
  
  return {
    on,
    toggle,
    togglerProps,
    displayProps
  };
}

// 사용 예시
function ToggleComponent() {
  const { on, togglerProps, displayProps } = useToggle();
  
  return (
    <div>
      <button {...togglerProps}>
        {on ? '켜짐' : '꺼짐'}
      </button>
      
      <div {...displayProps}>
        현재 상태: {on ? '활성화' : '비활성화'}
      </div>
    </div>
  );
}
```

이 예시에서 `togglerProps`와 `displayProps`는 프롭 컬렉션으로, 각각 토글 버튼과 상태 표시 요소에 필요한 속성들을 묶어 제공합니다.

## 실제 사용 예시: 접근성 있는 Dialog 컴포넌트

jsx

```jsx
function useDialog() {
  const [isOpen, setIsOpen] = useState(false);
  const dialogRef = useRef(null);
  const openButtonRef = useRef(null);
  
  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);
  
  // Dialog가 열릴 때 포커스 관리
  useEffect(() => {
    if (isOpen && dialogRef.current) {
      dialogRef.current.focus();
    }
  }, [isOpen]);
  
  // ESC 키로 Dialog 닫기
  useEffect(() => {
    const handleKeyDown = (event) => {
      if (isOpen && event.key === 'Escape') {
        close();
        openButtonRef.current?.focus();
      }
    };
    
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [isOpen]);
  
  // 프롭 컬렉션: 트리거 버튼에 대한 props
  const triggerProps = {
    ref: openButtonRef,
    onClick: open,
    'aria-haspopup': 'dialog',
    'aria-expanded': isOpen
  };
  
  // 프롭 컬렉션: 대화상자에 대한 props
  const dialogProps = {
    ref: dialogRef,
    role: 'dialog',
    'aria-modal': true,
    tabIndex: -1,
    hidden: !isOpen
  };
  
  // 프롭 컬렉션: 닫기 버튼에 대한 props
  const closeButtonProps = {
    onClick: close,
    'aria-label': '닫기'
  };
  
  return {
    isOpen,
    open,
    close,
    triggerProps,
    dialogProps,
    closeButtonProps
  };
}

// 사용 예시
function AccessibleDialog() {
  const { 
    isOpen, 
    triggerProps, 
    dialogProps, 
    closeButtonProps 
  } = useDialog();
  
  return (
    <>
      <button {...triggerProps}>
        대화상자 열기
      </button>
      
      <div className="dialog-container" {...dialogProps}>
        <div className="dialog-content">
          <h2>접근성 있는 대화상자</h2>
          <p>이 대화상자는 키보드 접근성과 스크린 리더 지원이 포함되어 있습니다.</p>
          <button {...closeButtonProps}>×</button>
        </div>
      </div>
      
      {isOpen && <div className="backdrop" />}
    </>
  );
}
```

이 예시에서는 대화상자(Dialog) 구현에 필요한 여러 접근성 관련 속성들을 프롭 컬렉션으로 그룹화하여 제공합니다.

## 프롭 게터(Prop Getters) 패턴

프롭 컬렉션의 확장 버전으로, 프롭 게터 패턴은 함수를 통해 동적으로 props를 생성하고 사용자 정의 props와 병합할 수 있는 유연성을 제공합니다.

jsx

```jsx
function useToggle() {
  const [on, setOn] = useState(false);
  
  const toggle = () => setOn(!on);
  
  // 프롭 게터: 사용자 정의 props와 병합 가능
  const getTogglerProps = (userProps = {}) => {
    return {
      'aria-pressed': on,
      onClick: callAll(toggle, userProps.onClick),
      ...userProps
    };
  };
  
  return {
    on,
    toggle,
    getTogglerProps
  };
}

// 여러 함수를 하나로 결합하는 유틸리티
function callAll(...fns) {
  return (...args) => {
    fns.forEach(fn => fn && fn(...args));
  };
}

// 사용 예시
function ToggleComponent() {
  const { on, getTogglerProps } = useToggle();
  
  return (
    <button
      {...getTogglerProps({
        className: 'custom-button',
        onClick: () => console.log('추가 동작!'),
        id: 'my-toggle'
      })}
    >
      {on ? '켜짐' : '꺼짐'}
    </button>
  );
}
```

프롭 게터 패턴은 사용자가 자신의 props를 추가하면서도 기본 기능을 유지할 수 있는 유연성을 제공합니다.

## 프롭 컬렉션 vs 프롭 게터

| 측면             | 프롭 컬렉션              | 프롭 게터                       |
| -------------- | ------------------- | --------------------------- |
| **유연성**        | 제한적 (정적 props 집합)   | 높음 (동적 props 생성, 사용자 정의 병합) |
| **사용 편의성**     | 매우 간단 (스프레드 구문만 사용) | 약간 복잡 (함수 호출 필요)            |
| **이벤트 핸들러 충돌** | 충돌 가능성 있음           | 충돌 방지 메커니즘 제공               |
| **사용 사례**      | 단순한 컴포넌트, 정적 요구사항   | 복잡한 상호작용, 사용자 정의 옵션 필요 시    |


## 프롭 컬렉션 패턴의 장점

1. **개발자 경험 향상**: 일반적으로 함께 사용되는 props를 쉽게 적용
2. **접근성 향상**: 접근성 관련 속성을 쉽게 포함
3. **일관성 유지**: 관련 컴포넌트 간에 일관된 동작 보장
4. **코드 간소화**: 반복적인 props 설정 감소
5. **의도 명확화**: props가 그룹화되어 목적이 명확해짐

## 프롭 컬렉션 패턴의 단점

1. **유연성 제한**: 기본 프롭 컬렉션은 사용자 정의를 위한 유연성이 제한적
2. **충돌 가능성**: 이벤트 핸들러가 덮어쓰여질 수 있음 (프롭 게터로 해결 가능)
3. **투명성 부족**: 어떤 props가 실제로 적용되는지 명확하지 않을 수 있음
4. **과도한 props**: 필요하지 않은 props까지 포함될 수 있음

## 실제 라이브러리에서의 사용 예: Downshift

[Downshift](https://github.com/downshift-js/downshift)는 프롭 게터 패턴을 효과적으로 활용한 대표적인 라이브러리입니다:

jsx

```jsx
import { useSelect } from 'downshift';

function Dropdown() {
  const {
    isOpen,
    selectedItem,
    getToggleButtonProps,
    getMenuProps,
    getItemProps
  } = useSelect({
    items: ['사과', '바나나', '오렌지', '포도'],
    onSelectedItemChange: ({ selectedItem }) => 
      console.log(`선택된 항목: ${selectedItem}`)
  });

  return (
    <div>
      <button {...getToggleButtonProps()}>
        {selectedItem || '과일 선택'}
      </button>
      
      <ul {...getMenuProps()} style={{ display: isOpen ? 'block' : 'none' }}>
        {isOpen &&
          ['사과', '바나나', '오렌지', '포도'].map((item, index) => (
            <li key={`${item}${index}`} {...getItemProps({ item, index })}>
              {item}
            </li>
          ))}
      </ul>
    </div>
  );
}
```

이 예시에서 Downshift는 `getToggleButtonProps`, `getMenuProps`, `getItemProps`와 같은 프롭 게터를 제공하여 접근성이 높은 드롭다운 구현을 간소화합니다.

## 결론

프롭 컬렉션은 React 컴포넌트 API 디자인에서 강력한 패턴으로, 특히 복잡한 UI 컴포넌트를 구현할 때 개발자 경험을 향상시킵니다. 더 고급 유연성이 필요한 경우 프롭 게터 패턴으로 확장할 수 있으며, 두 패턴 모두 현대적인 React 라이브러리와 컴포넌트 디자인에서 널리 사용됩니다.

## 5.5.6 복합 컴포넌트

# 복합 컴포넌트(Compound Components) 패턴

복합 컴포넌트 패턴은 React에서 관련된 컴포넌트들이 함께 작동하여 복잡한 UI 구성요소를 만드는 디자인 패턴입니다. 이 패턴은 HTML의 `<select>`와 `<option>` 요소처럼 컴포넌트들이 서로 의미적으로 연결되어 함께 동작하는 방식을 모방합니다.

## 기본 개념

복합 컴포넌트 패턴의 핵심 개념은 다음과 같습니다:

1. **부모-자식 관계**: 주요 부모 컴포넌트와 여러 하위 컴포넌트로 구성
2. **암시적 상태 공유**: 부모 컴포넌트가 내부 상태를 관리하고 자식 컴포넌트와 암시적으로 공유
3. **선언적 구문**: 컴포넌트의 계층 구조를 직관적으로 표현
4. **관심사 분리**: 각 컴포넌트가 UI의 특정 부분에 집중

## 기본 구현 예시: 탭 컴포넌트

jsx

```jsx
// 복합 컴포넌트 패턴을 사용한 탭 컴포넌트
import React, { createContext, useState, useContext } from 'react';

// 탭 컨텍스트 생성
const TabContext = createContext();

// 부모 컴포넌트
function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);
  
  // 컨텍스트 값 정의
  const contextValue = {
    activeIndex,
    setActiveIndex
  };
  
  return (
    <TabContext.Provider value={contextValue}>
      <div className="tabs">{children}</div>
    </TabContext.Provider>
  );
}

// 탭 헤더 컴포넌트
function TabList({ children }) {
  return (
    <div className="tab-list" role="tablist">
      {children}
    </div>
  );
}

// 각 탭 버튼 컴포넌트
function Tab({ children, index }) {
  const { activeIndex, setActiveIndex } = useContext(TabContext);
  const isActive = activeIndex === index;
  
  return (
    <button
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveIndex(index)}
      role="tab"
      aria-selected={isActive}
      id={`tab-${index}`}
      aria-controls={`panel-${index}`}
    >
      {children}
    </button>
  );
}

// 탭 패널 컨테이너
function TabPanels({ children }) {
  return <div className="tab-panels">{children}</div>;
}

// 각 탭 내용 컴포넌트
function TabPanel({ children, index }) {
  const { activeIndex } = useContext(TabContext);
  const isActive = activeIndex === index;
  
  return (
    <div
      className={`tab-panel ${isActive ? 'active' : ''}`}
      role="tabpanel"
      id={`panel-${index}`}
      aria-labelledby={`tab-${index}`}
      hidden={!isActive}
    >
      {children}
    </div>
  );
}

// 컴포넌트 구성을 위한 정적 속성 할당
Tabs.TabList = TabList;
Tabs.Tab = Tab;
Tabs.TabPanels = TabPanels;
Tabs.TabPanel = TabPanel;

export { Tabs };
```

이 패턴을 사용하면 다음과 같이 선언적이고 직관적인 방식으로 탭 UI를 구성할 수 있습니다:

jsx

```jsx
function App() {
  return (
    <Tabs>
      <Tabs.TabList>
        <Tabs.Tab index={0}>첫 번째 탭</Tabs.Tab>
        <Tabs.Tab index={1}>두 번째 탭</Tabs.Tab>
        <Tabs.Tab index={2}>세 번째 탭</Tabs.Tab>
      </Tabs.TabList>
      
      <Tabs.TabPanels>
        <Tabs.TabPanel index={0}>첫 번째 탭 내용입니다.</Tabs.TabPanel>
        <Tabs.TabPanel index={1}>두 번째 탭 내용입니다.</Tabs.TabPanel>
        <Tabs.TabPanel index={2}>세 번째 탭 내용입니다.</Tabs.TabPanel>
      </Tabs.TabPanels>
    </Tabs>
  );
}
```

## React.Children과 React.cloneElement를 사용한 구현

초기의 복합 컴포넌트 패턴은 주로 `React.Children.map`과 `React.cloneElement`를 사용하여 구현되었습니다:

jsx

```jsx
function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);
  
  return (
    <div className="tabs">
      {React.Children.map(children, child => {
        return React.cloneElement(child, {
          activeIndex,
          setActiveIndex
        });
      })}
    </div>
  );
}

function TabList({ children, activeIndex, setActiveIndex }) {
  return (
    <div className="tab-list" role="tablist">
      {React.Children.map(children, (child, index) => {
        return React.cloneElement(child, {
          index,
          isActive: index === activeIndex,
          setActiveIndex
        });
      })}
    </div>
  );
}
```

하지만 현대적인 React에서는 Context API를 사용하는 방식이 더 유연하고 명확합니다.

## 복합 컴포넌트 패턴의 장점

1. **직관적인 API**: JSX의 계층 구조가 UI 구조를 직접적으로 반영
2. **유연한 구성**: 컴포넌트의 순서와 조합을 자유롭게 구성 가능
3. **상태 관리 캡슐화**: 내부 상태 로직이 부모 컴포넌트에 캡슐화됨
4. **접근성 향상**: 관련 ARIA 속성이 적절히 연결됨
5. **관심사 분리**: 각 서브컴포넌트가 고유한 책임을 가짐

## 복합 컴포넌트 패턴의 단점

1. **컴포넌트 의존성**: 자식 컴포넌트가 특정 부모 컴포넌트에 의존
2. **사용 제약**: 정해진 계층 구조를 따라야 함
3. **구현 복잡성**: Context API나 클론 엘리먼트를 사용한 구현이 복잡할 수 있음
4. **타입 정의 어려움**: TypeScript에서 정적 속성과 컴포넌트 타입 정의가 복잡

## 실제 사용 예시: 접근성 있는 아코디언 컴포넌트

jsx

```jsx
import React, { createContext, useContext, useState } from 'react';
import { useId } from 'react-id-generator';

// 아코디언 컨텍스트
const AccordionContext = createContext();
const ItemContext = createContext();

function Accordion({ children, multiple = false }) {
  const [expandedItems, setExpandedItems] = useState([]);
  
  const toggleItem = (itemId) => {
    setExpandedItems(prev => {
      if (multiple) {
        // 다중 선택 모드
        return prev.includes(itemId) 
          ? prev.filter(id => id !== itemId) 
          : [...prev, itemId];
      } else {
        // 단일 선택 모드
        return prev.includes(itemId) ? [] : [itemId];
      }
    });
  };
  
  return (
    <AccordionContext.Provider value={{ expandedItems, toggleItem }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ children }) {
  const itemId = useId();
  const headerId = `accordion-header-${itemId}`;
  const panelId = `accordion-panel-${itemId}`;
  
  return (
    <ItemContext.Provider value={{ itemId, headerId, panelId }}>
      <div className="accordion-item">{children}</div>
    </ItemContext.Provider>
  );
}

function AccordionHeader({ children }) {
  const { itemId, headerId, panelId } = useContext(ItemContext);
  const { expandedItems, toggleItem } = useContext(AccordionContext);
  const isExpanded = expandedItems.includes(itemId);
  
  return (
    <button
      className={`accordion-header ${isExpanded ? 'expanded' : ''}`}
      onClick={() => toggleItem(itemId)}
      id={headerId}
      aria-expanded={isExpanded}
      aria-controls={panelId}
    >
      {children}
      <span className="accordion-icon">{isExpanded ? '−' : '+'}</span>
    </button>
  );
}

function AccordionPanel({ children }) {
  const { itemId, headerId, panelId } = useContext(ItemContext);
  const { expandedItems } = useContext(AccordionContext);
  const isExpanded = expandedItems.includes(itemId);
  
  return (
    <div
      className={`accordion-panel ${isExpanded ? 'expanded' : ''}`}
      id={panelId}
      role="region"
      aria-labelledby={headerId}
      hidden={!isExpanded}
    >
      {children}
    </div>
  );
}

// 정적 속성 할당
Accordion.Item = AccordionItem;
Accordion.Header = AccordionHeader;
Accordion.Panel = AccordionPanel;

export { Accordion };
```

사용 예시:

jsx

```jsx
function App() {
  return (
    <Accordion multiple>
      <Accordion.Item>
        <Accordion.Header>첫 번째 섹션</Accordion.Header>
        <Accordion.Panel>
          <p>첫 번째 섹션의 내용입니다.</p>
        </Accordion.Panel>
      </Accordion.Item>
      
      <Accordion.Item>
        <Accordion.Header>두 번째 섹션</Accordion.Header>
        <Accordion.Panel>
          <p>두 번째 섹션의 내용입니다.</p>
        </Accordion.Panel>
      </Accordion.Item>
      
      <Accordion.Item>
        <Accordion.Header>세 번째 섹션</Accordion.Header>
        <Accordion.Panel>
          <p>세 번째 섹션의 내용입니다.</p>
        </Accordion.Panel>
      </Accordion.Item>
    </Accordion>
  );
}
```

## 복합 컴포넌트와 다른 패턴 비교

### 복합 컴포넌트 vs Props 기반 구성

jsx

```jsx
// 복합 컴포넌트 방식
<Tabs>
  <Tabs.TabList>
    <Tabs.Tab index={0}>첫 번째</Tabs.Tab>
    <Tabs.Tab index={1}>두 번째</Tabs.Tab>
  </Tabs.TabList>
  <Tabs.TabPanels>
    <Tabs.TabPanel index={0}>내용 1</Tabs.TabPanel>
    <Tabs.TabPanel index={1}>내용 2</Tabs.TabPanel>
  </Tabs.TabPanels>
</Tabs>

// Props 기반 방식
<Tabs
  items={[
    { label: '첫 번째', content: '내용 1' },
    { label: '두 번째', content: '내용 2' }
  ]}
/>
```

|측면|복합 컴포넌트|Props 기반 구성|
|---|---|---|
|**유연성**|높음 (자유로운 구성)|제한적 (고정된 구조)|
|**가독성**|구조가 명확하게 보임|복잡한 데이터 구조에서 가독성 저하|
|**사용 편의성**|약간 장황할 수 있음|간결한 API|
|**커스터마이징**|쉬움 (각 부분 직접 제어)|제한적 (props로 전달 가능한 옵션만)|
|**적합한 상황**|복잡한 UI, 다양한 변형 필요|단순한 UI, 일관된 구조|

### 복합 컴포넌트 vs 고차 컴포넌트

|측면|복합 컴포넌트|고차 컴포넌트|
|---|---|---|
|**구현 방식**|컴포넌트 계층 구조|컴포넌트 래핑 함수|
|**상태 공유**|Context API로 암시적 공유|Props로 명시적 전달|
|**구성 스타일**|선언적 JSX 계층 구조|함수 호출과 합성|
|**적합한 상황**|복잡한 UI 구성 요소|횡단 관심사 (인증, 데이터 로딩 등)|

## 실제 라이브러리에서의 사용 예: React Bootstrap

[React Bootstrap](https://react-bootstrap.github.io/)은 복합 컴포넌트 패턴을 효과적으로 활용한 라이브러리입니다:

jsx

```jsx
import { Dropdown } from 'react-bootstrap';

function MyDropdown() {
  return (
    <Dropdown>
      <Dropdown.Toggle variant="success" id="dropdown-basic">
        메뉴 선택
      </Dropdown.Toggle>

      <Dropdown.Menu>
        <Dropdown.Item href="#/action-1">작업 1</Dropdown.Item>
        <Dropdown.Item href="#/action-2">작업 2</Dropdown.Item>
        <Dropdown.Item href="#/action-3">작업 3</Dropdown.Item>
      </Dropdown.Menu>
    </Dropdown>
  );
}
```

## 효과적인 복합 컴포넌트 설계를 위한 팁

1. **명확한 계층 구조**: 컴포넌트 계층 구조가 UI 구조를 직관적으로 반영하도록 설계
2. **Context 활용**: React Context API를 사용하여 상태와 이벤트 핸들러 공유
3. **정적 속성 최소화**: 지나치게 많은 정적 속성은 API를 복잡하게 만들 수 있음
4. **접근성 고려**: ARIA 속성을 적절히 연결하여 접근성 보장
5. **폴백 처리**: 올바른 계층 구조가 아닐 경우 적절한 오류 메시지나 폴백 UI 제공
6. **명확한 문서화**: 컴포넌트 구조와 사용 방법에 대한 명확한 문서 제공

## 결론

복합 컴포넌트 패턴은 복잡한 UI 구성요소를 직관적이고 선언적인 방식으로 구현할 수 있게 해주는 강력한 패턴입니다. 이 패턴은 특히 탭, 아코디언, 드롭다운, 메뉴 등 관련 컴포넌트들이 함께 작동해야 하는 UI에 적합합니다. 컴포넌트 라이브러리를 설계할 때 이 패턴을 고려하면 사용자에게 더 유연하고 직관적인 API를 제공할 수 있습니다.



## 5.5.7 상태 리듀서


상태 리듀서 패턴은 React에서 컴포넌트 상태 관리를 위한 고급 디자인 패턴으로, 컴포넌트의 상태 변경 로직을 사용자가 재정의하거나 확장할 수 있게 해주는 기법입니다. 이 패턴은 주로 복잡한 상태 로직을 가진 재사용 가능한 컴포넌트나 라이브러리를 만들 때 유용합니다.

## 기본 개념

상태 리듀서 패턴의 핵심 개념은 다음과 같습니다:

1. **리듀서 함수 사용**: 상태 변경을 관리하기 위해 리듀서 함수(`(state, action) => newState`) 사용
2. **액션 타입 정의**: 컴포넌트의 상태 변경 이벤트를 액션 타입으로 정의
3. **제어 역전(IoC)**: 상태 변경 로직을 컴포넌트 사용자에게 위임
4. **기본 동작 제공**: 기본 리듀서 구현을 제공하지만 사용자가 재정의 가능

## useReducer와 상태 리듀서 패턴

React의 `useReducer` 훅은 상태 리듀서 패턴의 기본 구현을 제공합니다:

jsx

```jsx
// useReducer 기본 사용 예
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  return (
    <div>
      카운트: {state.count}
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
    </div>
  );
}
```

## 상태 리듀서 패턴 구현 예시

복잡한 토글 컴포넌트에 상태 리듀서 패턴을 적용한 예시입니다:

jsx

```jsx
import React, { useReducer } from 'react';

// 액션 타입 정의
const TOGGLE_ACTIONS = {
  TOGGLE: 'TOGGLE',
  RESET: 'RESET',
  ON: 'ON',
  OFF: 'OFF'
};

// 기본 리듀서 구현
function toggleReducer(state, action) {
  switch (action.type) {
    case TOGGLE_ACTIONS.TOGGLE:
      return { on: !state.on };
    case TOGGLE_ACTIONS.RESET:
      return { on: false };
    case TOGGLE_ACTIONS.ON:
      return { on: true };
    case TOGGLE_ACTIONS.OFF:
      return { on: false };
    default:
      return state;
  }
}

function useToggle({ reducer = toggleReducer, initialOn = false } = {}) {
  const [{ on }, dispatch] = useReducer(reducer, { on: initialOn });
  
  const toggle = () => dispatch({ type: TOGGLE_ACTIONS.TOGGLE });
  const reset = () => dispatch({ type: TOGGLE_ACTIONS.RESET });
  const setOn = () => dispatch({ type: TOGGLE_ACTIONS.ON });
  const setOff = () => dispatch({ type: TOGGLE_ACTIONS.OFF });
  
  return {
    on,
    toggle,
    reset,
    setOn,
    setOff,
    TOGGLE_ACTIONS
  };
}

// 사용 예시
function ToggleComponent() {
  const { on, toggle } = useToggle();
  
  return (
    <div>
      <button onClick={toggle}>
        {on ? '켜짐' : '꺼짐'}
      </button>
    </div>
  );
}
```

## 사용자 정의 리듀서로 동작 확장하기

상태 리듀서 패턴의 핵심 장점은 기본 동작을 확장하거나 재정의할 수 있다는 것입니다:

jsx

```jsx
function App() {
  // 사용자 정의 리듀서 구현
  function customReducer(state, action) {
    // 토글 상태가 이미 켜져 있을 때 TOGGLE 액션 무시
    if (action.type === TOGGLE_ACTIONS.TOGGLE && state.on) {
      return state; // 현재 상태 유지
    }
    // 그 외의 경우 기본 리듀서 동작 사용
    return toggleReducer(state, action);
  }
  
  const { on, toggle } = useToggle({ reducer: customReducer });
  
  return (
    <div>
      <p>한 번 켜지면 끌 수 없는 토글: {on ? '켜짐' : '꺼짐'}</p>
      <button onClick={toggle}>토글</button>
    </div>
  );
}
```

## 더 복잡한 상태 리듀서 패턴: actionTypes 공유

컴포넌트 라이브러리를 만들 때는 액션 타입을 공유하여 사용자가 모든 상태 변경을 재정의할 수 있게 하는 것이 중요합니다:

jsx

```jsx
function useAccordion({ reducer, initialState = { expandedItems: [] } } = {}) {
  // 액션 타입 정의
  const actionTypes = {
    TOGGLE_ITEM: 'TOGGLE_ITEM',
    EXPAND_ALL: 'EXPAND_ALL',
    COLLAPSE_ALL: 'COLLAPSE_ALL',
    EXPAND_ITEM: 'EXPAND_ITEM',
    COLLAPSE_ITEM: 'COLLAPSE_ITEM'
  };
  
  // 기본 리듀서 구현
  const defaultReducer = (state, action) => {
    switch (action.type) {
      case actionTypes.TOGGLE_ITEM: {
        const { itemId } = action.payload;
        return {
          ...state,
          expandedItems: state.expandedItems.includes(itemId)
            ? state.expandedItems.filter(id => id !== itemId)
            : [...state.expandedItems, itemId]
        };
      }
      case actionTypes.EXPAND_ALL: {
        const { itemIds } = action.payload;
        return { ...state, expandedItems: itemIds };
      }
      case actionTypes.COLLAPSE_ALL:
        return { ...state, expandedItems: [] };
      case actionTypes.EXPAND_ITEM: {
        const { itemId } = action.payload;
        if (state.expandedItems.includes(itemId)) return state;
        return { ...state, expandedItems: [...state.expandedItems, itemId] };
      }
      case actionTypes.COLLAPSE_ITEM: {
        const { itemId } = action.payload;
        return {
          ...state,
          expandedItems: state.expandedItems.filter(id => id !== itemId)
        };
      }
      default:
        return state;
    }
  };
  
  // 사용자 정의 리듀서 또는 기본 리듀서 사용
  const [state, dispatch] = useReducer(
    reducer || defaultReducer,
    initialState
  );
  
  // 액션 생성 함수들
  const toggleItem = itemId => 
    dispatch({ type: actionTypes.TOGGLE_ITEM, payload: { itemId } });
  
  const expandAll = itemIds => 
    dispatch({ type: actionTypes.EXPAND_ALL, payload: { itemIds } });
  
  const collapseAll = () => 
    dispatch({ type: actionTypes.COLLAPSE_ALL });
  
  const expandItem = itemId => 
    dispatch({ type: actionTypes.EXPAND_ITEM, payload: { itemId } });
  
  const collapseItem = itemId => 
    dispatch({ type: actionTypes.COLLAPSE_ITEM, payload: { itemId } });
  
  return {
    expandedItems: state.expandedItems,
    toggleItem,
    expandAll,
    collapseAll,
    expandItem,
    collapseItem,
    actionTypes, // 액션 타입 노출
  };
}
```

## 사용자 정의 리듀서 예시

jsx

```jsx
function CustomAccordion({ items }) {
  // 아코디언의 기본 동작을 수정하는 사용자 정의 리듀서
  function customReducer(state, action) {
    // 기본 아코디언 리듀서 가져오기
    const { actionTypes } = useAccordion();
    
    if (action.type === actionTypes.TOGGLE_ITEM) {
      const { itemId } = action.payload;
      const isExpanding = !state.expandedItems.includes(itemId);
      
      // 확장 시에만 로깅
      if (isExpanding) {
        console.log(`아이템 ${itemId} 확장됨`);
      }
      
      // 동시에 하나만 열리도록 제한
      if (isExpanding) {
        // 다른 모든 항목은 닫고 선택한 항목만 열기
        return { ...state, expandedItems: [itemId] };
      } else {
        // 선택한 항목 닫기
        return { ...state, expandedItems: [] };
      }
    }
    
    // 기본 동작 사용
    const defaultReducer = useAccordion().defaultReducer;
    return defaultReducer(state, action);
  }
  
  const { expandedItems, toggleItem } = useAccordion({ reducer: customReducer });
  
  return (
    <div className="accordion">
      {items.map(item => (
        <div key={item.id} className="accordion-item">
          <button
            onClick={() => toggleItem(item.id)}
            className="accordion-header"
            aria-expanded={expandedItems.includes(item.id)}
          >
            {item.title}
          </button>
          {expandedItems.includes(item.id) && (
            <div className="accordion-panel">{item.content}</div>
          )}
        </div>
      ))}
    </div>
  );
}
```

## 상태 리듀서 패턴의 장점

1. **유연성**: 컴포넌트 사용자가 상태 변경 로직을 완전히 제어 가능
2. **확장성**: 기본 동작을 유지하면서 특정 액션에 대한 처리만 변경 가능
3. **예측 가능성**: 액션과 리듀서 기반 상태 관리로 상태 변경이 예측 가능
4. **테스트 용이성**: 리듀서는 순수 함수로 테스트하기 쉬움
5. **관심사 분리**: 상태 로직과 UI 렌더링 로직의 명확한 분리

## 상태 리듀서 패턴의 단점

1. **복잡성**: 간단한 컴포넌트에는 과도한 패턴일 수 있음
2. **러닝 커브**: 리듀서 개념과 액션 타입 이해가 필요
3. **보일러플레이트**: 액션 타입 정의와 리듀서 구현에 코드가 많이 필요
4. **디버깅 어려움**: 사용자 정의 리듀서가 있을 때 문제 추적이 복잡할 수 있음

## 상태 리듀서 vs 다른 패턴 비교

### 상태 리듀서 vs 제어 컴포넌트

|측면|상태 리듀서|제어 컴포넌트|
|---|---|---|
|**상태 제어**|상태 변경 로직 제어|상태 값 직접 제어|
|**유연성**|기존 로직 확장 가능|전체 상태 관리 대체|
|**복잡성**|더 복잡한 구현|비교적 단순한 구현|
|**적합한 상황**|복잡한 상태 전이 로직이 있는 컴포넌트|단순한 입력 필드나 폼 요소|

### 상태 리듀서 vs 렌더 프롭

|측면|상태 리듀서|렌더 프롭|
|---|---|---|
|**제어 목적**|상태 변경 로직 제어|렌더링 출력 제어|
|**패턴 유형**|상태 관리 패턴|UI 구성 패턴|
|**확장성**|액션 핸들링 확장|UI 렌더링 확장|
|**적합한 상황**|상태 전이 로직 제어가 필요한 경우|UI 변형이 많은 경우|

## 실제 라이브러리에서의 사용 예: Downshift

[Downshift](https://github.com/downshift-js/downshift)는 상태 리듀서 패턴을 효과적으로 활용한 라이브러리입니다:

jsx

```jsx
import { useSelect } from 'downshift';

function CustomSelect() {
  // 사용자 정의 상태 리듀서 구현
  function stateReducer(state, actionAndChanges) {
    const { type, changes } = actionAndChanges;
    
    // 특별한 경우 처리: 선택 후 메뉴가 닫히지 않도록 설정
    switch (type) {
      case useSelect.stateChangeTypes.ItemClick:
        // 선택 시 isOpen 상태를 true로 유지
        return { ...changes, isOpen: true };
      default:
        // 기본 동작 유지
        return changes;
    }
  }
  
  const {
    isOpen,
    selectedItem,
    getToggleButtonProps,
    getMenuProps,
    getItemProps
  } = useSelect({
    items: ['사과', '바나나', '오렌지', '포도'],
    stateReducer
  });

  return (
    <div>
      <button {...getToggleButtonProps()}>
        {selectedItem || '과일 선택'}
      </button>
      
      <ul {...getMenuProps()} style={{ display: isOpen ? 'block' : 'none' }}>
        {isOpen &&
          ['사과', '바나나', '오렌지', '포도'].map((item, index) => (
            <li key={`${item}${index}`} {...getItemProps({ item, index })}>
              {item}
            </li>
          ))}
      </ul>
    </div>
  );
}
```

## 효과적인 상태 리듀서 패턴 구현을 위한 팁

1. **명확한 액션 타입**: 모든 상태 변경 이벤트에 대한 명확한 액션 타입 정의
2. **액션 타입 공개**: 컴포넌트 사용자에게 모든 액션 타입을 공개
3. **기본 동작 제공**: 합리적인 기본 동작을 제공하는 기본 리듀서 구현
4. **변경 사항 유형화**: 어떤 상태 속성이 각 액션으로 변경될 수 있는지 문서화
5. **사용자 정의 예시 제공**: 사용자 정의 리듀서 구현 예시 제공
6. **불변성 유지**: 리듀서에서 상태 불변성 원칙 준수

## 결론

상태 리듀서 패턴은 복잡한 상태 관리 로직을 가진 재사용 가능한 컴포넌트를 만들 때 강력한 도구입니다. 이 패턴은 컴포넌트의 기본 동작을 제공하면서도 사용자에게 상태 변경에 대한 완전한 제어권을 제공합니다. 특히 UI 라이브러리나 복잡한 폼 컴포넌트와 같이 다양한 상황에서 유연하게 동작해야 하는 컴포넌트에 적합합니다. 이 패턴의 복잡성에도 불구하고, 제공하는 유연성과 제어성은 복잡한 컴포넌트 API 설계에서 큰 가치를 제공합니다.



---
# 복습하기

## 1. 리액트에서 메모화란 무엇이며, 컴포넌트 렌더링을 최적화하는 데 어떻게 사용할 수 있을까?

메모화(Memoization)는 이전에 계산한 결과를 저장하고 동일한 입력이 주어질 때 다시 계산하지 않고 저장된 결과를 반환하는 최적화 기법입니다.

### 리액트에서의 메모화 도구:

1. **React.memo**: 함수형 컴포넌트를 메모화하는 고차 컴포넌트입니다.
    
    ```jsx
    const MemoizedComponent = React.memo(MyComponent);
    ```
    
    - 컴포넌트의 props가 변경되지 않으면 리렌더링을 방지합니다.
    - 얕은 비교를 기본으로 사용하며, 필요시 커스텀 비교 함수를 제공할 수 있습니다.
2. **useMemo**: 계산 비용이 높은 값을 메모화합니다.
    
    ```jsx
    const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
    ```
    
    - 의존성 배열의 값이 변경될 때만 값을 재계산합니다.
3. **useCallback**: 함수를 메모화합니다.
    
    ```jsx
    const memoizedCallback = useCallback(() => {
      doSomething(a, b);
    }, [a, b]);
    ```
    
    - 의존성 배열의 값이 변경될 때만 함수를 재생성합니다.

### 렌더링 최적화 전략:

1. **렌더링이 비용이 큰 컴포넌트에 React.memo 적용**:
    
    ```jsx
    const ExpensiveComponent = React.memo(({ data }) => {
      // 데이터 처리 로직...
      return <div>{/* 복잡한 UI */}</div>;
    });
    ```
    
2. **계산 비용이 높은 연산을 useMemo로 최적화**:
    
    ```jsx
    function DataGrid({ items, filter }) {
      const filteredItems = useMemo(() => {
        return items.filter(item => {
          // 복잡한 필터링 로직
          return complexFilterFunction(item, filter);
        });
      }, [items, filter]);
      
      return <div>{filteredItems.map(item => <Item key={item.id} {...item} />)}</div>;
    }
    ```
    
3. **이벤트 핸들러를 useCallback으로 메모화**:
    
    ```jsx
    function SearchComponent({ onSearch }) {
      const [query, setQuery] = useState('');
      
      const handleSearch = useCallback(() => {
        onSearch(query);
      }, [query, onSearch]);
      
      return (
        <div>
          <input value={query} onChange={e => setQuery(e.target.value)} />
          <SearchButton onClick={handleSearch} />
        </div>
      );
    }
    ```
    

## 2. 리액트 상태 관리를 위해 useReducer를 사용하면 어떤 우세한 점이 있으며, useReducer는 useState와 어떻게 다른가?

useReducer는 복잡한 상태 로직을 관리하기 위한 useState의 대안입니다.

### useReducer와 useState의 차이점:

|측면|useState|useReducer|
|---|---|---|
|**복잡성**|단순한 상태에 적합|복잡한 상태 로직에 적합|
|**상태 업데이트**|직접 상태 설정 함수 호출|액션 객체를 디스패치|
|**상태 구조**|일반적으로 독립적인 상태 변수|관련 상태를 객체로 통합|
|**예측 가능성**|상태 업데이트가 분산됨|중앙화된 상태 업데이트 로직|
|**로직 위치**|컴포넌트 내부에 분산됨|리듀서 함수에 집중됨|
|**테스트 용이성**|컴포넌트와 함께 테스트해야 함|순수 함수로 독립적 테스트 가능|

### useReducer의 우세한 점:

1. **복잡한 상태 로직 관리**:
    
    ```jsx
    const initialState = { count: 0, loading: false, error: null };
    
    function reducer(state, action) {
      switch (action.type) {
        case 'INCREMENT':
          return { ...state, count: state.count + 1 };
        case 'DECREMENT':
          return { ...state, count: state.count - 1 };
        case 'FETCH_START':
          return { ...state, loading: true, error: null };
        case 'FETCH_SUCCESS':
          return { ...state, loading: false, count: action.payload };
        case 'FETCH_ERROR':
          return { ...state, loading: false, error: action.payload };
        default:
          return state;
      }
    }
    
    function Counter() {
      const [state, dispatch] = useReducer(reducer, initialState);
      
      // 상태 업데이트를 디스패치로 처리
      const increment = () => dispatch({ type: 'INCREMENT' });
      const decrement = () => dispatch({ type: 'DECREMENT' });
      const fetchCount = async () => {
        dispatch({ type: 'FETCH_START' });
        try {
          const response = await fetchCountFromAPI();
          dispatch({ type: 'FETCH_SUCCESS', payload: response.count });
        } catch (error) {
          dispatch({ type: 'FETCH_ERROR', payload: error.message });
        }
      };
      
      return (
        <div>
          <p>Count: {state.count}</p>
          {state.loading && <p>Loading...</p>}
          {state.error && <p>Error: {state.error}</p>}
          <button onClick={increment}>+</button>
          <button onClick={decrement}>-</button>
          <button onClick={fetchCount}>Fetch Count</button>
        </div>
      );
    }
    ```
    
2. **관련 상태 그룹화**: 논리적으로 관련된 상태 변수들을 하나의 객체로 관리합니다.
    
3. **상태 업데이트 예측 가능성**: 모든 상태 업데이트가 리듀서 함수를 통해 이루어지므로 상태 변경을 추적하기 쉽습니다.
    
4. **상태 전이 명확화**: 액션 타입을 통해 상태 변경의 의도가 명확해집니다.
    
5. **컴포넌트 외부에서 상태 로직 정의**: 리듀서 함수를 컴포넌트 외부에 정의하여 관심사 분리와 테스트가 용이합니다.
    

## 3. React.lazy와 Suspense 컴포넌트를 사용하는 리액트 앱에서 지연 로딩을 어떻게 구현할 수 있을까?

지연 로딩(Lazy Loading)은 애플리케이션의 초기 로드 시간을 줄이기 위해 필요한 시점에 코드를 로드하는 기법입니다.

### React.lazy와 Suspense를 사용한 코드 분할:

1. **기본 컴포넌트 지연 로딩**:
    
    ```jsx
    import React, { Suspense } from 'react';
    
    // 동적 임포트를 사용한 컴포넌트 지연 로딩
    const LazyComponent = React.lazy(() => import('./LazyComponent'));
    
    function App() {
      return (
        <div>
          <h1>애플리케이션</h1>
          <Suspense fallback={<div>로딩 중...</div>}>
            <LazyComponent />
          </Suspense>
        </div>
      );
    }
    ```
    
2. **라우트 기반 코드 분할**:
    
    ```jsx
    import React, { Suspense } from 'react';
    import { BrowserRouter, Routes, Route } from 'react-router-dom';
    
    // 라우트별 컴포넌트 지연 로딩
    const Home = React.lazy(() => import('./pages/Home'));
    const About = React.lazy(() => import('./pages/About'));
    const Dashboard = React.lazy(() => import('./pages/Dashboard'));
    
    function App() {
      return (
        <BrowserRouter>
          <Suspense fallback={<div>페이지 로딩 중...</div>}>
            <Routes>
              <Route path="/" element={<Home />} />
              <Route path="/about" element={<About />} />
              <Route path="/dashboard" element={<Dashboard />} />
            </Routes>
          </Suspense>
        </BrowserRouter>
      );
    }
    ```
    
3. **조건부 로딩**:
    
    ```jsx
    import React, { Suspense, useState } from 'react';
    
    const HeavyFeature = React.lazy(() => import('./HeavyFeature'));
    
    function App() {
      const [showFeature, setShowFeature] = useState(false);
      
      return (
        <div>
          <button onClick={() => setShowFeature(true)}>
            고급 기능 표시
          </button>
          
          {showFeature && (
            <Suspense fallback={<div>기능 로딩 중...</div>}>
              <HeavyFeature />
            </Suspense>
          )}
        </div>
      );
    }
    ```
    
4. **중첩된 Suspense 경계**:
    
    ```jsx
    import React, { Suspense } from 'react';
    
    const Header = React.lazy(() => import('./Header'));
    const Content = React.lazy(() => import('./Content'));
    const Sidebar = React.lazy(() => import('./Sidebar'));
    
    function App() {
      return (
        <div className="app">
          <Suspense fallback={<div>헤더 로딩 중...</div>}>
            <Header />
          </Suspense>
          
          <div className="main">
            <Suspense fallback={<div>사이드바 로딩 중...</div>}>
              <Sidebar />
            </Suspense>
            
            <Suspense fallback={<div>콘텐츠 로딩 중...</div>}>
              <Content />
            </Suspense>
          </div>
        </div>
      );
    }
    ```
    
5. **프리페칭(Prefetching)을 통한 사용자 경험 개선**:
    
    ```jsx
    import React, { Suspense, useState, useEffect } from 'react';
    
    // 동적 임포트 정의
    const HeavyFeature = React.lazy(() => import('./HeavyFeature'));
    
    function App() {
      const [showFeature, setShowFeature] = useState(false);
      
      // 마우스가 버튼 위에 올라갔을 때 미리 로드
      const prefetchFeature = () => {
        const prefetch = import('./HeavyFeature');
      };
      
      return (
        <div>
          <button 
            onClick={() => setShowFeature(true)}
            onMouseOver={prefetchFeature}
          >
            고급 기능 표시
          </button>
          
          {showFeature && (
            <Suspense fallback={<div>기능 로딩 중...</div>}>
              <HeavyFeature />
            </Suspense>
          )}
        </div>
      );
    }
    ```
    

## 4. 리액트에서 메모화를 사용할 때 발생할 수 있는 잠재적 문제는 무엇이며 어떻게 완화할 수 있을까?

메모화는 강력한 최적화 도구이지만, 잘못 사용하면 오히려 성능 문제를 야기할 수 있습니다.

### 잠재적 문제 및 완화 방법:

1. **과도한 메모화**:
    
    - **문제**: 모든 컴포넌트를 메모화하면 메모리 사용량이 증가하고 비교 연산으로 인한 오버헤드가 발생할 수 있습니다.
    - **완화 방법**: 렌더링 비용이 높거나 자주 리렌더링되는 컴포넌트에만 선택적으로 메모화를 적용합니다.
    
    ```jsx
    // 복잡한 계산이나 큰 렌더링 트리가 있는 경우에만 메모화
    const ExpensiveComponent = React.memo(({ data }) => {
      // 복잡한 렌더링 로직
    });
    
    // 가벼운 컴포넌트는 메모화하지 않음
    const SimpleComponent = ({ text }) => <span>{text}</span>;
    ```
    
2. **잘못된 의존성 배열**:
    
    - **문제**: useMemo나 useCallback에서 의존성 배열이 잘못되면 예상치 못한 버그가 발생할 수 있습니다.
    - **완화 방법**: 의존성 배열을 정확히 지정하고, eslint-plugin-react-hooks를 사용하여 검증합니다.
    
    ```jsx
    // eslint-disable-next-line react-hooks/exhaustive-deps
    const memoizedValue = useMemo(() => compute(a, b), []); // 위험! a와 b가 변경되어도 재계산되지 않음
    
    // 올바른 방법
    const memoizedValue = useMemo(() => compute(a, b), [a, b]);
    ```
    
3. **참조 동일성 문제**:
    
    - **문제**: 객체나 배열은 매 렌더링마다 새로운 참조가 생성되어 메모화가 효과적이지 않을 수 있습니다.
    - **완화 방법**: 불변 데이터 구조를 사용하고, 객체와 배열을 useMemo로 메모화합니다.
    
    ```jsx
    // 잘못된 사용 - 매번 새 객체 생성
    function Component({ item }) {
      // 매 렌더링마다 새 객체 생성
      const itemWithExtra = { ...item, extra: 'data' };
      return <ChildComponent item={itemWithExtra} />;
    }
    
    // 개선된 방법
    function Component({ item }) {
      const itemWithExtra = useMemo(() => {
        return { ...item, extra: 'data' };
      }, [item]);
      return <ChildComponent item={itemWithExtra} />;
    }
    ```
    
4. **깊은 비교의 비용**:
    
    - **문제**: 커스텀 비교 함수에서 깊은 비교를 사용하면 성능 저하가 발생할 수 있습니다.
    - **완화 방법**: 가능한 얕은 비교를 사용하고, 필요한 경우 선택적으로 깊은 비교를 적용합니다.
    
    ```jsx
    // 비용이 많이 드는 방법
    const MemoizedComponent = React.memo(MyComponent, (prevProps, nextProps) => {
      return deepEqual(prevProps, nextProps); // 모든 속성에 대한 깊은 비교
    });
    
    // 개선된 방법
    const MemoizedComponent = React.memo(MyComponent, (prevProps, nextProps) => {
      // 필요한 속성만 선택적으로 비교
      return prevProps.id === nextProps.id && 
             prevProps.name === nextProps.name;
    });
    ```
    
5. **메모리 누수**:
    
    - **문제**: 메모화된 값이 더 이상 필요하지 않아도 메모리에 계속 유지될 수 있습니다.
    - **완화 방법**: 메모화가 필요한 범위를 제한하고, 컴포넌트의 생명주기를 고려합니다.
    
    ```jsx
    function ParentComponent() {
      // 자식 컴포넌트가 언마운트되면 이 메모화된 값도 가비지 컬렉션됨
      return shouldShowChild ? (
        <ChildComponent />
      ) : null;
    }
    
    function ChildComponent() {
      // 이 컴포넌트가 마운트된 동안만 메모리에 유지됨
      const memoizedValue = useMemo(() => expensiveComputation(), []);
      return <div>{memoizedValue}</div>;
    }
    ```
    
6. **잘못된 최적화 타이밍**:
    
    - **문제**: 성능 측정 없이 무작정 최적화하면 불필요한 복잡성만 증가할 수 있습니다.
    - **완화 방법**: React Profiler나 Performance 탭을 사용하여 실제 성능 병목을 식별한 후 최적화합니다.
    
    ```jsx
    // React Profiler를 사용한 성능 측정
    import { Profiler } from 'react';
    
    function onRenderCallback(
      id, // 방금 커밋된 Profiler의 "id" prop
      phase, // "mount" 또는 "update"
      actualDuration, // 렌더링에 걸린 시간
      baseDuration, // 최적화 없이 렌더링하는 데 걸릴 예상 시간
      startTime, // 언제 React가 이 업데이트를 렌더링하기 시작했는지
      commitTime // 언제 React가 이 업데이트를 커밋했는지
    ) {
      console.log(`${id} 렌더링 시간: ${actualDuration}ms`);
    }
    
    function App() {
      return (
        <Profiler id="App" onRender={onRenderCallback}>
          <MyComponent />
        </Profiler>
      );
    }
    ```
    

## 5. 리액트에서 컴포넌트에 프롭으로 전달된 함수를 메모화하기 위해 useCallback 훅을 어떻게 사용할 수 있나?

useCallback은 함수를 메모화하여, 의존성이 변경되지 않는 한 동일한 함수 참조를 유지합니다. 이는 특히 함수를 props로 하위 컴포넌트에 전달할 때 불필요한 리렌더링을 방지하는 데 유용합니다.

### useCallback 기본 사용법:

```jsx
import React, { useState, useCallback } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  // count가 변경될 때만 함수 재생성
  const handleCountChange = useCallback(() => {
    setCount(prevCount => prevCount + 1);
  }, []);
  
  // text가 변경될 때만 함수 재생성
  const handleTextChange = useCallback((e) => {
    setText(e.target.value);
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Text: {text}</p>
      
      <ChildComponent 
        onCountChange={handleCountChange}
        onTextChange={handleTextChange}
      />
    </div>
  );
}

// React.memo로 최적화된 자식 컴포넌트
const ChildComponent = React.memo(({ onCountChange, onTextChange }) => {
  console.log('ChildComponent rendered');
  return (
    <div>
      <button onClick={onCountChange}>증가</button>
      <input onChange={onTextChange} />
    </div>
  );
});
```

### 외부 의존성이 있는 콜백 메모화:

```jsx
function SearchComponent({ onSearch, searchTerm }) {
  // searchTerm이 변경될 때만 함수 재생성
  const handleSearch = useCallback(() => {
    onSearch(searchTerm);
  }, [onSearch, searchTerm]);
  
  return (
    <div>
      <input value={searchTerm} onChange={...} />
      <SearchButton onClick={handleSearch} />
    </div>
  );
}
```

### 함수 내에서 최신 상태 참조하기:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  // 의존성 없이 항상 최신 count 값에 접근
  const handleClick = useCallback(() => {
    setCount(prevCount => prevCount + 1);
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <ExpensiveButton onClick={handleClick} />
    </div>
  );
}
```

### 데이터 변환 함수 메모화:

```jsx
function DataProcessor({ data, onProcessed }) {
  // 데이터 처리 함수 메모화
  const processData = useCallback((item) => {
    const result = complexProcessing(item);
    onProcessed(result);
  }, [onProcessed]);
  
  return (
    <div>
      {data.map(item => (
        <DataItem 
          key={item.id}
          item={item}
          onProcess={() => processData(item)}
        />
      ))}
    </div>
  );
}
```

### 이벤트 핸들러에서 여러 상태 업데이트:

```jsx
function FormComponent() {
  const [values, setValues] = useState({ name: '', email: '' });
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleSubmit = useCallback((e) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    // API 호출
    submitForm(values)
      .then(response => {
        setIsSubmitting(false);
        // 성공 처리
      })
      .catch(error => {
        setIsSubmitting(false);
        // 오류 처리
      });
  }, [values]);
  
  return (
    <form onSubmit={handleSubmit}>
      {/* 폼 필드들 */}
      <SubmitButton disabled={isSubmitting} />
    </form>
  );
}
```

### 비동기 콜백 메모화:

```jsx
function AsyncComponent({ id, onDataLoaded }) {
  const fetchData = useCallback(async () => {
    try {
      const response = await fetch(`/api/data/${id}`);
      const data = await response.json();
      onDataLoaded(data);
    } catch (error) {
      console.error('데이터 로드 실패:', error);
    }
  }, [id, onDataLoaded]);
  
  return <DataLoader onLoad={fetchData} />;
}
```

### 부모 컴포넌트에서 콜백 체인:

```jsx
function ParentComponent() {
  const [items, setItems] = useState([]);
  
  // 아이템 추가 함수
  const addItem = useCallback((item) => {
    setItems(prevItems => [...prevItems, item]);
  }, []);
  
  // 아이템 제거 함수
  const removeItem = useCallback((id) => {
    setItems(prevItems => prevItems.filter(item => item.id !== id));
  }, []);
  
  return (
    <div>
      <ItemList items={items} onRemove={removeItem} />
      <AddItemForm onAdd={addItem} />
    </div>
  );
}
```

메모화는 리액트 애플리케이션 최적화를 위한 강력한 도구이지만, 필요한 경우에만 선택적으로 사용하고 항상 성능 측정을 통해 실제 개선 효과를 확인하는 것이 중요합니다.