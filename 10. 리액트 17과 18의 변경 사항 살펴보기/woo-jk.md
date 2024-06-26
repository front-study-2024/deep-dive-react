# 10.1 리액트 17 버전 살펴보기

## 10.1.1 리액트의 점진적인 업그레이드

- 하나의 애플리케이션에서 버전이 다른 리액트를 사용할 수 있다
    - 예를 들어 새로운 리액트 버전에서 기존에 사용하던 api의 지원이 중단되었을 경우,
    - 전체 앱은 최신 리액트 버전으로 업그레이드하고 일부분에서만 기존의 리액트 버전을 사용하는 방식이 가능하다
- 그러나 이는 관리해야할 지점이 많아져 복잡도가 크게 올라가니, 리액트 팀에서도 한꺼번에 업그레이드가 불가능한 어쩔수 없는 상황에서 차선책으로만 사용하는 것을 추천하고 있다

## 10.1.2 이벤트 위임 방식의 변경

- 리액트는 이벤트 핸들러를 추가하면 이벤트 위임 방식으로 처리한다
- 기존 16 버전까지는 `document`에 이벤트 핸들러를 달아서 수행한다
- 17 버전부터는 `document`가 아닌, 리액트 컴포넌트 최상단 트리, 즉 루트 요소로 바뀌었다.
- 그 이유는 앞서 말한 점진적인 업그레이드 지원, 그리고 다른 바닐라 JS 혹은 JQuery등이 혼재 되어있는 경우 혼란을 방지하기 위해서이다.

## 10.1.3 import React from ‘react’가 더 이상 필요 없다: 새로운 JSX transform

- 17 버전부터 바벨과 협력하여 import React 구문 없이 JSX를 변환할 수 있게 됐다.
    - 기존 버전에서는 JSX 변환시 `React.createElement` 를 사용하기 때문에 해당 import 문이 꼭 필요했다.
    
    ```jsx
    // JSX
    const Component = (
      <div>
        <span>hello world</span>
      </div>
    )
    
    // 변환 후
    var Component = React.createElement(
      'div',
      null,
      React.createElement('span', null, 'hello world'),
    )
    ```
    
    - 17 버전부터는 다음과 같이 변환된다.
    
    ```jsx
    var _jsxRuntime = require('react/jsx-runtime')
    
    var Component = (0, _jsxRuntime.jsx)('div', {
      children: (0, _jsxRuntime.jsx)('span', {
        children: 'hello world',
      }),
    })
    ```
    
    - JSX 변환시 `react/jsx-runtime` 을 require 하는 구문이 자동으로 추가된다.
- `import React` 문이 제거된다는건 단순히 코드가 간결해지는 효과 뿐만 아니라, 번들 사이즈도 조금이나마 줄어든다는 효과도 있다.

## 10.1.4 그 밖의 주요 변경 사항

### 이벤트 풀링 제거

- 과거 리액트 16의 이벤트는 `SyntheticEvent` 라는 이벤트 객체로 브라우저의 기본 이벤트를 한 번 감싸져있었다. 이를 이벤트 풀링이라고 한다.
- 이 방식은 이벤트가 발생할때마다 래핑 이벤트를 새롭게 만들고 지워주는 방식을 반복했다.
- 그러나 해당 방식은 다음과 같이 이벤트 핸들러 내에서 비동기 코드를 사용한다면 에러가 났다.

```jsx
const [value, setValue] = useState('');

function handleChange(e) {
  setValue(() => {
    return e.target.value
  })
}
```

- 이벤트 핸들러 호출이 끝나면 이벤트가 null로 초기화 되기 때문에 발생하는 문제이고, 이를 해결하려면 `e.persist()` 와 같은 처리가 필요했다.
- 위와 같은 처리가 개발자 입장에서 매우 번거롭기도 했고, 또한 모던 브라우저에서는 이와 같은 방식이 성능 향상에 큰 도움이 안되었기 때문에 이러한 이벤트 풀링 개념을 삭제했다.

### useEffect 클린업 함수의 비동기 실행

- 기존 버전에서는 useEffect의 클린업 함수가 동기적으로 처리되었다.
- 그러나 이는 클린업 함수가 완료되기 전까지 다른 작업을 방해하므로 성능 저하로 이어졌다.
- 17 버전부터는 화면이 완전히 업데이트된 이후에 클린업 함수가 비동기적으로 실행된다.

# 10.2 리액트 18 버전 살펴보기

## 10.2.1 새롭게 추가된 훅 살펴보기

### useId

- `useId`는 컴포넌트별로 유니크한 값을 생성하는 훅이다.
- 컴포넌트 내부에서 유니크한 값을 생성하는 것은 쉽지않다.
    - 특히, 서버 사이드 렌더링의 경우 렌더링 했을 때의 값과 하이드레이션 했을 때의 값이 일치 하지 않아 문제가 발생하는 경우가 많다.
    - 이러한 문제를 해결하기 위해 `useId` 가 생겨났다.

### useTransition

- `useTransition` 훅은 UI 변경을 가로막지 않고 상태를 업데이트할 수 있는 훅이다.
- `useTransition` 훅의 `startTranstion` 함수를 사용하여 렌더링을 실행하면 비동기적으로 렌더링하여 렌더링에 의한 블로킹을 막을 수 있다.
- `useTranstion` 은 리액트 18의 변경 사항의 핵심 중 하나인 `동시성`을 다룰수 있게 해주는 훅이다.
    - 과거 리액트의 모든 렌더링은 동기적으로 작동해 렌더링이 느린 작업이 있을 경우 전체적인 성능 문제가 발생한다.
    - `useTransition` 과 같은 동시성을 지원하는 기능을 사용하면 렌더링 과정에서 로딩 화면을 보여주거나, 현재 진행 중인 렌더링을 버리는 등의 작업을 할 수 있게 된다.
- `useTransition` 의 주의 사항은 다음과 같다
    - `startTranstion` 는 반드시 setState와 같은 상태 업데이트 함수만 넘길 수 있다. 다른 값을 사용하고 싶다면 `useDeferredValue` 를 사용해야 한다.
    - `startTranstion` 으로 넘겨주는 상태 업데이트는 다른 모든 동기 상태 업데이트로 인해 실행이 지연될 수 있다.
    - `startTransition` 으로 넘겨주는 함수는 반드시 동기함수여야 한다.

### useDeferredValue

- `useDeferredValue` 는 리액트 컴포넌트 트리에서 리렌더링이 급하지 않은 부분을 지연할 수 있게 도와주는 훅이다.
- 디바운스와 비슷하지만, 디바운스와 다른점은 고정된 지연 시간 없이 첫 번째 렌더링이 완료된 이후에 지연된 렌더링을 수행한다.
    - 그러므로 이 지연된 렌더링은 중단할 수도 있으며, 사용자의 인터랙션을 차단하지도 않는다.
- `useTansition` 과의 차이점은 다음과 같다.
    - `useTransition` 은 state 값을 업데이트하는 함수를 감싸서 사용하는 반면, `useDeferredValue` 는 state 값 자체만을 감싸서 사용한다.
    - 직접적으로 상태를 업데이트할 수 있는 코드에 접근할 수 있다면 `useTransition` 을 사용하는 것이 좋고, props와 같이 상태 업데이트에 관여할 수 없고 값만 받아야 하는 상황이라면 `useDeferredValue` 를 사용하면 좋다.

### useSyncExternalStore

- `useSyncExternalStore` 는 리액트 외부의 저장소에서 사용되는 데이터를 추적하고, 해당 데이터가 변경되면 리액트 컴포넌트의 리렌더링을 발생시키는 훅이다.
- `useSyncExternalStore` 을 사용하면 테어링 현상을 방지할 수 있다.
    - 테어링 현상이란, 여러 컴포넌트에서 하나의 상태를 바라보고 있지만, 렌더링 과정에서 데이터 변경이 일어나는 등의 이유로 다른 값을 렌더링 하는 현상을 의미한다.
- `useSyncExternalStore` 에서 받는 각 인자는 다음과 같은 역할을 한다.
    - 훅의 첫 번째 인자는 `subscribe` 로, 콜백 함수를 받아 스토어에 등록하는 용도로 사용된다.
    - 두 번째 인자 `getSnapshot` 으로, 컴포넌트에 필요한 현재 스토어의 데이터를 반환하는 함수다.
    - 마지막 인수는 옵셔널 값으로, 서버 사이드 렌더링 시 내부 리액트를 하이드레이션 하는 도중에만 사용된다.

### useInsertionEffect

- `useSyncExternalStore` 가 상태 관리 라이브러리를 위한 훅이라면, `useInsertionEffect` 는 CSS-in-JS 라이브러리를 위한 훅이다.
    - CSS의 추가 및 수정은 렌더링 하는 작업 대부분을 다시 계산해 작업해야 하는데, 이는 매우 무거운 작업이다.
- 기본적인 훅 구조는 `useEffect`와 동일하지만 실행 시점이 다르다.
    - DOM이 실제로 변경되기 전에 동기적으로 실행된다.
    - `useInsertionEffect` - `useLayoutEffect` - `useEffect` 순서대로 실행이 된다.
    - `useLayoutEffect` 는 모든 DOM의 변경 작업이 끝난 후 실행되는 반면, `useInsertionEffect` 는 DOM 변경 작업 이전에 실행된다.
    - 이러한 차이는 브라우저가 스타일을 입혀서 DOM을 재계산하지 않아도 된다는 점에서 큰 차이가 있다.

## 10.2.2 react-dom/client

- 클라이언트에서 리액트 트리를 만들 때 사용되는 API들이 변경되었다.

### createRoot

- 기존 react-dom에 있던 `render` 메서드를 대체할 새로운 메서드이다.
- 리액트 18의 기능을 사용하고 싶다면 `createRoot`와 `render`를 함께 사용해야 한다.

### hydrateRoot

- 서버 사이드 렌더링 애플리케이션에서 하이드레이션 하기 위한 새로운 메서드이다.

## 10.2.3 react-dom/server

### renderToPipeableStream

- 리액트 컴포넌트를 HTML로 렌더링하는 메서드
- 스트림을 지원하는 메서드로, HTML을 점진적으로 렌더링할 수 있다.
- Suspense를 활용해 빠르게 렌더링이 필요한 부분을 먼저 렌더링하고 오래걸리는 부분은 이후에 렌더링 되게끔 할 수 있다.

### renderToReadableStream

- `renderToPipeableStream` 이 Node.js 환경에서 렌더링을 위해 사용된다면, `renderToReadableStream` 은 웹 스트림을 기반으로 작동한다는 차이가 있다.

## 10.2.4 자동배치(Automatic Batching)

- 자동 배치는 리액트가 여러 상태 업데이트를 하나의 리렌더링으로 묶어서 성능을 향상시키는 방법을 의미한다.
- 예를 들어 버튼 클릭 한 번에 setState를 두 번 실행할 경우, 리액트 17 이전 버전에서는 두 번 렌더링 되지만, 리액트 18에서는 한 번만 렌더링 된다.
- 만약 이러한 자동 배치를 사용하고 싶지 않다면 `flushSync` 라는 함수를 사용하면 된다.

## 10.2.5 더욱 엄격해진 엄격 모드

### 리액트의 엄격 모드

- 리액트의 엄격 모드는 리액트에서 제공하는 컴포넌트 중 하나로, 리액트 앱에서 발생할 수 있는 잠재적인 버그를 찾는 데 도움이 되는 컴포넌트이다.
- 엄격 모드는 개발자 모드에서만 작동한다.

### 엄격 모드에서 하는 작업

- 더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고
- 문자열 ref 사용 금지
    - createRef, useRef로 생성한 객체가 아닌, 문자열로 ref를 주입하는 것을 금지한다.
- findDOMNode에 대한 경고 출력
    - findDOMNode는 실제 DOM 요소에 대한 참조를 가져올 수 있는 메서드이지만, 사용이 지양된다.
- 구 Context API 사용 시 발생하는 경고
- 예상치 못한 부작용(side-effects) 검사
    - 엄격 모드에서는 다음 내용을 의도적으로 이중으로 호출한다.
        - 클래스 컴포넌트의 constructor, render, shouldComponentUpdate, getDerivedStateFromProps
        - 클래스 컴포넌트의 setState의 첫 번째 인수
        - 함수 컴포넌트의 body
        - useState, useMemo, useReducer에 전달되는 함수
    - 두 번씩 실행하는 이유는 다음과 같다.
        - 함수형 프로그래밍 원칙에 따라 모든 컴포넌트는 순수하다고 가정한다.
        - 즉, 항상 순수한 결과물을 내고 있는지 개발자에게 확인시켜 주기 위해 두 번 실행하게 되는 것이다.

### 리액트 18에서 추가된 엄격 모드

- 향후 리액트에서는 컴포넌트가 마운트 해제된 상태에도 컴포넌트 내부의 상태값을 유지할 수 있는 기능을 제공할 예정이라고 리액트 팀에서 밝혔다.
- 컴포넌트가 최초 마운트될 때 자동으로 모든 컴포넌트를 마운트 해제하고 두 번째 마운트에서 이전 상태를 복원하는 방식으로 구현된다고 한다.
- 이를 위해 엄격 모드에서 useEffect(마운트)를 두 번 실행시키는 내용을 추가하여, 여러번 마운트되어도 문제 없는지 개발자가 확인할 수 있다.

## 10.2.6 Suspense 기능 강화

- 기존 18 이전의 Suspense는 몇 가지 문제가 있었다.
    - 컴포넌트가 아직 보이기도 전에 useEffect가 실행되는 문제
    - Suspense를 서버에서 사용할 수 없는 문제
- 18 버전의 Suspense는 실험 단계를 벗어나 정식으로 지원됐고, 변경된 내용은 다음과 같다.
    - 컴포넌트가 실제로 화면에 노출될 때 effect가 실행된다.
    - 컴포넌트가 보이거나 사라질 때 effect가 정상적으로 실행된다. Suspense가 끝나면 effect가, Suspense가 다시 시작되면 cleanUp이 실행된다.
    - 서버에서 실행할 수 있게되었다.
    - Suspense내에 스로틀링이 추가되었다. 화면이 너무 자주 업데이트되는 것을 방지하기 위해 다음 렌더링을 보여주기 전에 잠시 대기한다.

## 10.2.7 인터넷 익스플로러 지원 중단에 따른 추가 폴리필 필요

- 리액트 18부터는 다음과 같은 자바스크립트 기능을 사용할 수 있다는 가정하에 배포된다.
    - Promise
    - Symbol
    - Object.assign
- IE 환경에서는 이 세 가지 기능을 지원하지 않으므로, IE를 지원하고 싶다면 이를 위한 폴리필을 반드시 추가해야한다.

## 10.2.8 그 밖에 알아두면 좋은 변경사항

- 컴포넌트에서 undefined를 반환해도 에러가 발생하지 않고, null을 반환한 것과 동일하게 처리된다.
- renderToNodeStream이 지원 중단됐고, 그 대신 renderToPipeableStream을 사용하는 것이 권장된다.

## 10.2.9 정리

- 리액트 18 버전 업의 핵심은 동시성 렌더링이다.
- 과거 렌더링 과정은 한번 시작하면 멈출 수 없는 동기 작업이었고, 무조건 결과를 봐야 끝나는 작업이었다.
- 그러나 리액트 18에서는 렌더링을 일시 중단하거나, 중간에 포기하고 다시 시작할 수도 있게되었다.
- 렌더링 과정이 한층 복잡해졌지만, 리액트는 `useSyncExternalStore` 과 같은 훅을 제공해 UI 일관성을 유지할 수 있도록했다.
- 이로인해 사용자는 웹 앱에서 렌더링으로 인해 UI가 방해받는 현상을 겪지 않을 수 있고, 사용자의 반응성 또한 확보할 수 있게 되었다.
