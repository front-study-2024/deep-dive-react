- 함수 컴포넌트가 상태를 사용하거나 생명주기 메서드를 대체하는 등 다양한 작업을 하기 위해 훅이라는 개념이 도입되었다.

# 3.1 리액트의 모든 훅 파헤치기

## 3.1.1 useState

### useState의 구현

- useState는 함수가 호출이 종료된 이후에도 setState를 통해 state값을 변경할 수있다.
- 함수의 실행이 끝났음에도 state값을 기억할 수 있는 이유는 클로저를 때문이다.
- 클로저를 통해 함수(useState) 내부에 선언된 함수(setState)가 함수의 실행이 종료된 이후에도 state를 계속 참조할 수 있다.

### 게으른 초기화

- useState의 원소로 변수 대신 함수를 넘기는 것을 게으른 초기화라고 한다.
- 게으른 초기화 함수는 오로지 state가 처음 만들어질 때만 사용되고, 이후 리렌더링에서는 이 함수의 실행은 무시된다.
- 게으른 초기화는 브라우저 스토리지에 접근하거나, map, filter 등 배열에 대한 접근 등 무거운 연산이 요구될 때 사용하는 것이 좋다.

## 3.1.2 useEffect

- useEffect는 애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 메커니즘이다.

### useEffect란?

- 첫 번째 인수로 부수 효과가 포함된 함수를, 두 번째 인수로 의존성 배열을 전달한다.
- useEffect의 의존성 배열은 proxy나 옵저버 등 특별한 기능을 통해 관찰하는 것이 아니고, 렌더링할 때마다 의존성에 있는 값을 보면서 이전과 다른게 있다면 부수효과 함수를 실행하는 평범한 함수이다.
- 따라서 useEffect는 state와 props의 변화 속에 일어나는 렌더링 과정에서 실행되는 부수 효과 함수이다.

### 클린업 함수의 목적

- 클린업 함수는 리렌더링 이후에 실행되지만, 사용하는 값(state, props 등)은 함수가 정의됐을 당시에 선언된 값을 보고 실행한다.
- 클린업 함수는 단순 언마운트시 실행되는 함수로 보기 보다는, 리렌더링 됐을 때 의존성 변화가 있을 당시 이전의 값을 기준으로 실행되는, 말 그대로 이전 상태를 청소해 주는 개념으로 보는 것이 옿다.

### 의존성 배열

- 의존성 배열을 빈 배열로 두면 리액트는 최초 렌더링 직후 실행된 이후에는 더 이상 실행하지 않는다.
- 의존성 배열이 없다면, 렌더링이 발생할 때마다 실행된다.
- 그렇다면, 다음 두 코드는 무슨 차이가 있는걸까?

```jsx
// 1
function Component() {
  console.log('렌더링됨')
}

// 2
function Component() {
  useEffect(() => {
    console.log('렌더링됨')
  })
}
```

- 얼핏보면 비슷하게 동작할 것 같지만 다음과 같은 차이점이 있다.
    - 서버 사이드 렌더링 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해 준다. window 객체에 의존하는 코드를 사용해도 된다.
    - useEffect는 컴포넌트 렌더링이 완료된 이후에 실행된다. 1번 코드의 경우 렌더링되는 도중에 실행된다. 즉, 무거운 작업일 경우 렌더링을 방해하므로 성능에 악영향을 끼칠 수 있다.

### useEffect의 구현

- useEffect에서 의존성 배열 비교는 [Object.is](http://Object.is) 기반의 얕은 비교이다.
- 이전 의존성 배열과 현재 의존성 배열의 값에 하나라도 변경 사항이 있다면 콜백을 실행한다.

### useEffect를 사용할 때 주의할 점

- eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 자제해라
    - 콜백에서 state나 props를 사용하지만, 빈 배열을 의존성으로 하고 싶을 때, 즉, 컴포넌트를 마운트하는 시점에만 해당 작업을 하고 싶을 때 위 주석을 실행하곤 한다.
    - state, props와 같은 어떤 값의 변경과 useEffect의 부수 효과가 별개로 작동하게 된다. 즉, 콜백 함수에서 사용한 값과 실제 변경 사이에 연결 고리가 끊어지게 된다.
    - 정말로 의존성으로 빈 배열이 필요하다면, 마운트 시점에만 콜백 함수 실행이 필요한지 되물어보고, 만약 정말 ‘그렇다’라고 하면 useEffect 내 부수 효과가 실행될 위치가 잘못됐을 가능성이 크다.
    - 주석을 사용하는 대신, 메모이제이션을 적절히 활용해 해당 값의 변화를 막거나 적당한 실행 위치를 다시 한 번 고민해보자.
- useEffect의 첫 번째 인수에 함수명을 부여하라
    - 일반적으로 useEffect의 콜백 함수로 익명 함수를 넘겨준다.
    - 하지만 해당 코드가 복잡해질수록 무슨 일을 하는 useEffect 코드인지 파악하기 어려워진다.
    - 의도를 파악하기 쉽게하기 위해 변수에 이름을 붙이는 것처럼, useEffect의 함수도 적절한 기명 함수로 바꾸면 해당 목적을 파악하기 쉬워진다.
- 거대한 useEffect를 만들지 마라
    - useEffect는 부수 효과를 실행하는 함수이다. 이 부수 효과의 크기가 커질수록 앱 성능에 악영향을 미친다.
    - 최대한 useEffect는 가볍게 유지하고, 부득이하게 큰 useEffect를 만들어야 한다면 가능한 분리하여 관리하는 것이 좋다.
    - 또한, useCallback과 useMemo 등으로 사전에 가공한 내용들만 useEffect에 담아두는 것이 좋다.
- 불필요한 외부 함수를 만들지 마라
    - useEffect 내부에서 사용하는 로직의 경우, 외부 함수로 분리하기보단 가급적 useEffect 내부로 가져오자.
    - 불필요하게 외부 함수로 분리한다면 의존성 배열이 늘어나게되고, useCallback, useMemo 등 불필요한 코드가 많아지게 되므로 가독성이 떨어진다.
- useEffect의 콜백 함수로 비동기 함수가 안되는 이유
    - 비동기 useEffect는 state의 경쟁 상태를 야기할 수 있고, 클린업 함수의 실행 순서도 보장할 수 없다.

## 3.1.3 useMemo

- useMemo는 비용이 큰 연산에 대한 결과를 저장해 두고, 이 저장된 값을 반환하는 훅이다.
- 첫 번째 인자로 어떠한 값을 반환하는 함수를, 두 번째 인자로 의존성 배열을 전달한다.

## 3.1.4 useCallback

- 값을 기억하는 useMemo와 달리 인수로 넘겨받은 콜백 자체를 기억한다.
- 컴포넌트가 리렌더링 될 때마다 함수를 재생성한다.
- React.memo를 사용한 컴포넌트의 props로 함수를 넘겨줄 때, useCallback을 사용하지 않은 함수를 넘겨주면 컴포넌트의 메모이제이션이 작동하지 않는다. 이는 매번 함수를 재생성하기 때문에 이를 받은 컴포넌트는 props가 계속 변경된다고 인식하기 때문이다.
- useMemo와 useCallback은 메모이제이션을 하는 대상이 변수냐 함수냐 차이 뿐이다.
    - 참고로, 자바스크립트에서는 함수 또한 값으로 표현될 수 있으므로 useMemo를 사용해서 useCalback을 구현할 수 있다.
    - 하지만 이는 코드의 혼동을 일으키므로 메모이제이션 대상이 함수일 경우 useCallback을 사용하자

## 3.1.5 useRef

- useRef는 useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다는 공통점이 있다.
- 그러나 useState와 구별되는 큰 차이점 두 가지를 가지고 있다.
    - useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
    - useRef는 그 값이 변경되어도 렌더링을 발생시키지 않는다.
- 컴포넌트 함수 외부에 값을 선언하여 관리하는 방식과 다른점은 다음과 같다.
    - 컴포넌트가 렌더링될 때만 생성된다.
    - 컴포넌트 인스턴스가 여러 개라도 각각 별개의 값을 바라본다.

## 3.1.6 useContext

### Context란?

- 일반적으로 컴포넌트간 데이터를 넘길 때 props를 사용한다.
- 그러나 컴포넌트 사이의 거리가 멀어질수록 코드가 복잡해지고, props drilling이라는 문제가 발생한다.
- 이러한 props drilling을 극복하기 위해 Context라는 개념이 등장했다.
- Context를 사용하면 props 전달 없이 하위 컴포넌트에서 자유롭게 원하는 값을 사용할 수 있다.

### Context를 함수 컴포넌트에서 사용할 수 있게 해주는 useContext 훅

- useContext는 상위 컴포넌트 어딘가에서 선언된 Context.Provider에서 제공한 값을 사용할 수 있게된다.
- 여러 개의 Provider가 있다면 가장 가까운 Provider의 값을 가져온다.

### useContext를 사용할 때 주의할 점

- 컴포넌트 내부에 useContext를 선언돼 있다면 Provider에 의존성을 가지게 된다.
    - 의존성을 줄이려면 useContext를 사용하는 컴포넌트를 최대한 작게 하거나 혹은 재사용되지 않을 만한 컴포넌트에서 사용해야 한다.
- 모든 컨텍스트를 최상위 루트 컴포넌트에 넣는 것은 현명한 방법이 아니다.
    - 컨텍스트가 많아질수록 컴포넌트의 관심사가 많아지게 되는 것이고, 해당 데이터를 다수의 컴포넌트에서 사용할 수 있게 끔 해야 하므로 불필요한 리소스가 낭비된다.
    - 컨텍스트의 범위는 최대한 좁게 만들어야 한다.
- 컨텍스트와 useContext를 상태 관리 API로 오해하지 말자
    - 상태 관리 라이브러리가 되기 위해선 최소 다음 두 가지 조건을 만족해야 한다.
        1. 어떠한 상태를 기반으로 다른 상태를 만들어 낼 수 있어야 한다.
        2. 필요에 따라 이러한 상태 변화를 최적화할 수 있어야 한다.
    - 컨텍스트는 둘 중 어느 것도 만족하지 못한다.
    - props 값을 하위로 전달해 줄 뿐, useContext를 사용한다고 해서 렌더링이 최적화되지 않는다.
    - 렌더링을 최적화하기 위해선 React.memo 등 다른 최적화 기법과 함께 사용해야 한다.

## 3.1.7 useReducer

- useState와 비슷한 형태를 띠지만 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다.
- 반환값
    - state: useState의 반환 값과 같은 상태
    - dispatcher: state를 업데이트 하는 함수. setState와 달리 action을 넘겨준다.
- 인수
    - reducer: action을 정의하는 함수
    - initialState: 초깃값
    - init(옵션): useState에 함수를 넘겨줄 때처럼 게으른 초기화를 하고 싶을 때 사용하는 함수. initialState를 인수로 init 함수가 실행된다.
- useReducer는 state의 업데이트를 미리 정의한 dispatcher로만 제한하여 state 변경 시나리오를 제한하고 이에 대한 변경을 빠르게 확인할 수 있게끔 하는 것이 useReducer의 목적이다.

## 3.1.8 useInperativeHandle

- useImperativeHandle을 이해하기 위해서는 먼저 React.forwardRef에 대해 알아야 한다.

### forwardRef 살펴보기

- ref는 key와 마찬가지로 컴포넌트의 props로 사용할 수 있는 예약어이다.
- 상위 컴포넌트에서 하위 컴포넌트로 넘겨주고 싶은 ref가 있을 때, 하위 컴포넌트에서 forwardRef를 사용한다.

### useImperativeHandle이란?

- useImperativeHandle은 부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅이다.
- useImperativeHandle 훅을 사용하여 부모 컴포넌트로 부터 넘겨 받은 ref에 추가적인 동작을 정의할 수 있다.
- 이렇게 사용하면 부모 컴포넌트에서는 자식 컴포넌트에서 새롭게 설정한 ref.current 객체의 키와 값에 대해서도 접근할 수 있게 된다.

## 3.1.9 useLayoutEffect

- useLayoutEffect는 useEffect의 시그니쳐와 동일하지만, 모든 DOM 변경 후에 동기적으로 발생한다.
    - 여기서 말하는 DOM 변경은 브라우저에 반영되는 시점이 아닌, 렌더링 시점을 의미한다.
- 실행 순서는 다음과 같다
    1. 리액트가 DOM을 업데이트
    2. useLayoutEffect 실행
    3. 브라우저에 변경 사항을 반영
    4. useEffect 실행
- 동기적으로 발생하기 때문에, 리액트는 useLayoutEffect의 실행이 종료될 때까지 기다린 후에 화면을 그린다.
    - 이로 인해 앱 성능 문제가 발생할 수 있으므로 주의해서 사용해야 한다.
- useLayoutEffect는 DOM은 계산됐지만, 이것이 화면에 반영되기 전에 하고 싶은 작업(DOM 요소 기반 애니메이션, 스크롤 위치 제어 등)이 있을 때와 같이 꼭 필요한 때에만 사용하는 것이 좋다.

## 3.1.10 useDebugValue

- useDebugValue는 일반적으로 프로덕션 버전보단, 앱 개발 과정에서 사용된다.
- useDebugValue는 커스텀 훅 내부의 내용에 대한 정보를 남길 수 있는 훅이다.
- 두 번째 인수로 포매팅 함수를 전달하면 이에 대한 값이 변경됐을 때만 호출되어 포매팅된 값을 노출한다.
    - 첫 번째 인수의 값이 같으면 포매팅 함수는 호출되지 않는다.
- useDebugValue는 커스텀 훅에서만 사용이 가능하고 컴포넌트 레벨에서는 작동하지 않는다.

## 3.1.11 훅의 규칙

- 리액트 공식 문서에는 다음과 같은 훅의 규칙을 정리해 뒀다.
    1. 최상위에서만 훅을 호출해야 한다. 반복문이나 조건문, 중첩 함수 내에서 훅을 실행할 수 없다. 컴포넌트가 렌더링될 때마다 항상 동일한 순서로 훅이 호출되는 것을 보장해야 하기 때문이다.
    2. 훅을 호출할 수 있는 것은 리액트 함수 컴포넌트, 혹은 커스텀 훅 두가지 경우 뿐이다.
- 리액트 훅은 파이버 객체의 링크드 리스트의 호출 순서에 따라 저장된다. 그 이유는 각 훅이 파이버 객체 내에서 순서에 의존해 state나 effect의 결과에 대한 값을 저장하고 있기 때문이다. 고정된 순서에 의존해 훅 정보를 저장함으로써 이전 값에 대한 비교와 실행이 가능해진다.
- 그렇기 때문에 조건문, 반복문 등에 의해 훅의 순서가 깨지거나 보장되지 않을 경우 리액트 코드는 에러를 발생시킨다.

# 3.2 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

- 리액트에서 로직을 재사용할 수 있는 방법은 사용자 정의 훅(커스텀 훅)과 고차 컴포넌트(HOC)이다.

## 3.2.1 사용자 정의 훅(커스텀 훅)

- 커스텀 훅은 내부에 useState, useEffect 등 여러 훅을 가지고 자신만의 원하는 훅을 만드는 기법이다.
- 앞서 언급된 리액트 훅의 규칙을 따라야 하고, 함수의 이름은 use로 시작해야한다.

## 3.2.2 고차 컴포넌트

- 커스텀 훅은 리액트 훅을 기반으로 하므로 리액트에서만 사용할 수 있는 것에 반해, 고차 컴포넌트는 고차 함수의 일종으로, JS의 일급 객체, 함수의 특징을 이용하므로 리액트가 아니더라도 JS 환경에서 쓰일 수 있다.
- 리액트에서는 고차 컴포넌트 기법으로 다양한 최적화나 중복 로직 관리를 할 수 있다.
- 리액트에서 가장 유명한 고차 컴포넌트는 React.memo다.

### 고차 컴포넌트의 특징

- 고차 컴포넌트는 컴포넌트 전체를 감쌀 수 있다는 점에서 커스텀 훅보다 더욱 큰 영향력을 컴포넌트에 미칠 수 있다.
- 단순히 값을 반환하거나 부수 효과를 실행하는 커스텀 훅과 다르게, 고차 컴포넌트는 컴포넌트의 결과물에 영향을 미칠 수 있는 다른 공통 작업을 처리할 수 있다.
- 일반적으로 고차 컴포넌트는 with로 시작하는 네이밍을 사용해야 한다.

### 고차 컴포넌트를 사용할 때 주의할 점

- 부수 효과를 최소화해야 한다.
    - 고차 컴포넌트는 감싸지는 컴포넌트의 props를 임의로 수정, 추가, 삭제하는 일이 없어야 한다.
    - 컴포넌트에 무언가 추가적인 정보를 주고싶다면 기존 프롭스를 건드리지 말고, 별도의 프롭스를 내려주는 것이 좋다.
- 여러 개의 고차 컴포넌트를 감쌀 경우 복잡성이 커진다.
    - 고차 컴포넌트가 여러 개 겹쳐지면 wrapper hell에 빠지게 된다.
    - 고차 컴포넌트가 증가할수록 개발자는 이것이 어떤 결과를 만들어 낼지 예측하기 힘들다.
    - 그러므로 고차 컴포넌트는 최소한으로 사용하는 것이 좋다.

## 3.2.3 커스텀 훅과 고차 컴포넌트 중 무엇을 써야 할까?

### 커스텀 훅이 필요한 경우

- 단순히 리액트 제공 훅으로만 공통 로직을 뺄 수 있다면 커스텀 훅을 사용하는 것이 좋다.
- 커스텀 훅은 그 자체로는 렌더링에 영향을 미치지 못하기 때문에 반환 값으로 어떤 작업을 할지는 사용하는 개발자에게 달려 있다.
- 이로 인해 커스텀 훅은 부수 효과가 비교적 적다.
- 대부분의 고차 컴포넌트는 렌더링에 영향을 미치는 로직이 존재하므로 커스텀 훅에 비해 예측하기가 어렵다.

### 고차 컴포넌트를 사용해야 하는 경우

- 함수 컴포넌트의 반환값, 즉 렌더링의 결과물에도 영향을 미치는 공통 로직이라면 고차 컴포넌트를 사용하는 것이 좋다.
- 고차 컴포넌트는 공통화된 렌더링 로직을 처리하기에 훌륭한 방법이다.
