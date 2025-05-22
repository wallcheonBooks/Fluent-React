# 느낀점

## 렌더프롭 패턴은 죽지 않았다

책에서는 렌더프롭 패턴이 더 이상 널리 사용되지 않으며 사실상 리액트 훅으로 대체되었다고 한다. 하지만 컴포넌트의 내부 상태를 끌어올리지 않고 컴포넌트 내부의 api을 컴포넌트 외부에서 활용하게 하는 방법은 렌더프롭 패턴이 유일하다.

```tsx
import { Dialog } from '@ark-ui/react/dialog'
import { Portal } from '@ark-ui/react/portal'

export const RenderFn = () => (
  <Dialog.Root>
    <Dialog.Trigger>Open Dialog</Dialog.Trigger>
    <Portal>
      <Dialog.Backdrop />
      <Dialog.Positioner>
        <Dialog.Content>
          <Dialog.Title>Dialog Title</Dialog.Title>
          <Dialog.Description>Dialog Description</Dialog.Description>
          <Dialog.CloseTrigger>Close</Dialog.CloseTrigger>
        </Dialog.Content>
      </Dialog.Positioner>
    </Portal>
    <Dialog.Context>{(dialog) => <p>Dialog is {dialog.open ? 'open' : 'closed'}</p>}</Dialog.Context>
  </Dialog.Root>
)
```

위의 예시를 보면 Dialog.Context 로 합성 컴포넌트를 제공하여 다이얼로그 컴포넌트 내부의 상태를 사용할 수 있게 하여 커스텀 가능성을 끌어올렸다.

## 책에서 말하는 제어프롭 패턴은 구현이 잘못되었다

```tsx
function Toggle({ on, onToggle }) {
  const [isOn, setIsOn] = React.useState(false);
  
  const handleToggle = () => {
    const nextState = on === undefined ? !isOn : on;
    if (on === undefined) {
      setIsOn(nextState);
    }
    if (onToggle) {
      onToggle(nextState)
    }
  }
  
  return (
  	<button onClick={handleToggle}>
    	{on !== undefined ? on : isOn ? "On" : "Off"}
    </button>
  )
}
```

위의 책에 예시는 useControlledState 와 같은 훅을 간단히 구현한 것인거 같은데, 컴포넌트 자체가 Toggle 이라 on이 boolean 값일 것이라 잘 티가 나지 않지만, 일반적인 경우 외부에서 prop을 주입받고 있는지를 prop === undefined 로 체크하면 안된다. 외부에서 넘겨준 prop 이 undefined 일 경우가 있기 때문이다.

제대로 된 방법은 in 연산자를 활용하는 것이다. ex) if ("on" in props) { ... } else { ... }

## 프롭 게터

```tsx
export const getDroppableProps = () => {
  return {
    onDragOver: (event) => {
      event.preventDefault();
    },
    onDrop: (event) => {},
  }
}
```

이 패턴은 zag.js 에서 아주 적극적으로 활용된다. ([zag.js accordion 예시](https://zagjs.com/components/react/accordion))

```tsx
import * as accordion from "@zag-js/accordion"
import { useMachine, normalizeProps } from "@zag-js/react"
import { useId } from "react"

const data = [
  { title: "Watercraft", content: "Sample accordion content" },
  { title: "Automobiles", content: "Sample accordion content" },
  { title: "Aircraft", content: "Sample accordion content" },
]

function Accordion() {
  const service = useMachine(accordion.machine, { id: useId() })

  const api = accordion.connect(service, normalizeProps)

  return (
    <div {...api.getRootProps()}>
      {data.map((item) => (
        <div key={item.title} {...api.getItemProps({ value: item.title })}>
          <h3>
            <button {...api.getItemTriggerProps({ value: item.title })}>
              {item.title}
            </button>
          </h3>
          <div {...api.getItemContentProps({ value: item.title })}>
            {item.content}
          </div>
        </div>
      ))}
    </div>
  )
}
```

## 재개가능성

발표자들에게 맡긴다.

## 어쩌면 모든 것은 서버 사이드 렌더링을 위한것이었나?

리액트 파이버 아키텍처, 코드분할과 지연로딩, 서스펜스, 스트리밍 이 모든 리액트에서의 기술 발전이 서버 사이드 렌더링을 위한 사전 작업이었다고 느껴졌다.

아직 안정된 기능은 아니지만 [Next.js Partial Prerendering](https://nextjs.org/learn/dashboard-app/partial-prerendering#how-does-partial-prerendering-work) 에서 보면,

서스펜스를 정적 코드와 동적 코드 사이의 경계로 사용하여 한 페이지 안에서도 코드를 분할하고 필요한대로 각각 스트리밍 방식으로 지연로딩하여 리액트 파이버 아키텍처로 렌더링 우선순위 관리하면서 렌더링하는데 미쳤다는 생각밖에 안들었다.

근데 이제 또 서버 컴포넌트라는 개념까지 발전한다고? 리액트가 오래되긴 했나보다.

## 너무 개요만 설명하는거 아닌가?

책의 저자가 이것보다 더 들어가면 너무 딥해지니까 여기까지만 알고 넘어가라 라고 하는거 같은데, 살짝 진짜 겉만 핥은 느낌이 있다. 진짜 전문가들의 전문가는 어떻게 되는걸까?
