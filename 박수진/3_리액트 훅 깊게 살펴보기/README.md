# 3\_리액트 훅 요소 깊게 살펴보기

목차

---

## 3.1 리액트의 모든 훅 파헤치기

리액트 훅의 구현은 github 코드에서 구현체를 타고 올라가다보면, 외부에서 접근을 막아놓은 부분이 나오면서, 파악하기가 어렵다. 따라서 저자는 preact 구현을 기준으로 예제를 구성했다.

### 3.1.1 useState

함수형 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리할 수 있게 해주는 훅

#### useState 구현 살펴보기

- 일단 리액트에서 리랜더링이 일어나는 시나리오는 2장 정리의, 2.4에서 참고하자.
- 아래 코드는 버튼이 눌리면서 state가 hi 로 재할당되고, triggerRender가 호출됨으로 함수 컴포넌트를 다시 실행하는데, 이때 state 는 다시 hello 로 초기화된다. 따라서 결과물인 return의 값을 비교했을 때 같고 화면에는 변화가 없다.
- 직접 실행해보려했는데 타입스크립트를 사용하면 아래처럼 useState 의 setter에 값이 없는 것 자체를 오류로 표시해 실행을 할 수가 없다.

```tsx
function Code3_1_1() {
  const [, triggerRender] = useState();
  console.log("call fn component", num);

  let state = "hello";

  const handleClick = () => {
    state = "hi";
    triggerRender();
  };

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleClick}>hi</button>
    </>
  );
}
```

실제 useState 는 어떤 형태로 구현이 되어있을까?
작동방식을 대략적으로 흉내낸 코드를 보자.

```ts
const MyReact = function () {
  const global = {};
  let index = 0;

  function useState(initialState) {
    if (!global.states) {
      // 최초 접근시 빈 배열로 초기화
      global.states = [];
    }

    // state 초기값 설정
    const currentState = global.states[index] || initialState;
    global.states[index] = currentState;

    const setState = (function () {
      //클로저의 원리를 이용해서,  setState가 호출되는 시점의, index를 계속기억하고, 그곳에 새로운 값을 저장해준다.
      let currentIndex = index;
      return function (value) {
        global.states[currentIndex] = value;
      };
    })();
    // 다른 곳에서 useState를 쓸때,
    index = index + 1;
    return [currentState, setState];
  }
};
```

**참고자료**

- [preact useState 참고링크](https://github.com/preactjs/preact/blob/f7ccb9010077ecb46fc271224bbc5e015e00efe6/hooks/src/index.js#L173)

#### 게으른 초기화

게으른 초기화란 useState 인수로 원시값이 아닌,특정값을 넘기는 함수를 인수로 넣는 방식이다.

- state 가 처음 초기화 될때만 사용된다. 이후 리랜더링 발생시에는 이 함수의 실행이 무시된다.
- 초깃값이 복잡하거나 무거운 연산을 포함할 때, 사용하면 된다.
  - localStorage 나 sessionStorage에 대한 접근
  - map, filter,find 같은 배열에 대한 접근
  - 초기값 계산을 위해 함수 호출이 필요할 때 등

```tsx
// 초깃값이 있어 더이상 필요 없는 리랜더링 시에도 매번,해당값에 접근한다.
const [count, setCount] = useState(
  Number.parseInt(window.localStorage.getItem(cacheKey))
);

// 게으른 초기화를 쓰자. 함수를 사용해 값을 반환하게 하면된다.
const [count, setCount] = useState(() =>
  Number.parseInt(window.localStorage.getItem(cacheKey))
);
```

### 3.1.2 useEffect

어플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 매커니즘

#### useEffect란?

**형태**

```tsx
function Component() {
  useEffect(() => {
    window.addEventListener("click", clickEvent);

    return () => {
      window.removeListener("click", clickEvent);
    };
  }, [props, state]);
}
```

- 첫번째 인수로 실행할 부수효과가 포함된 함수, 두번째 인수로 의존성 배열 전달
- 의존성 배열이 변경되었을 때, 부수효과를 실행하는 원리
  - 자바스크립트의 proxy나 데이터 바인딩, 옵저버 같은 특별한 기능으로 관찰하는것이 아니다.
  - state 와 props의 변화속에 일어나는 리액트 랜더링 과정에서 호출되며, 의존성의 값이 달라진 것이 있으면 부수효과를 실행할 뿐이다.

**클린업 함수의 목적**

- useEffec는 콜백이 실행될 때마다 이전의 클린업 함수가 존재하면, 그 클린업 함수를 실행한 뒤에 콜백을 실행한다.
  - 이전에 등록했던 이벤트 핸들러를 삭제하고 이벤트를 추가함으로, 특정 이벤트의 핸들러가 무한 추가되는 것을 방지한다.
- 생명주기 메서드 언마운트 개념과는 조금 차이가 있다.
  - 언마운트 : ( 클래스 컴포넌트 용어 ) 특정 컴포넌트가 DOM 에서 사라지는 것
  - 클린업 함수 : 함수형 컴포넌트가 리랜더링 되었을 때, 의존성 변화가 있던 이전 값을 기준으로 실행되어 이전 상태를 청소하는 용도

**의존성 배열**

- 빈배열: 비교할 의존성 값이 없어서 최초 랜더링 직후에만 실행된다.
- 원하는 값을 넣어줌: 의존성 값이 변경되었을 때만 실행된다.
- 값을 넘기지 않음: 랜더링이 발생할 때 마다 실행된다.

**useEffect의 구현**

- 의존성 배열의 이전값과 현재 값의 얕은 비교를 Object.is를 기반으로하는 shallowEqual을 이용해서 진행하고, 변경이 있으면, callback 으로 선언한 부수효과 실행
  - 따라서 얉은 비교에서 변경사항이 포착이 안되는 객체의 중첩 객체의 값 변화등은 부수효과를 일으키지 않는 것이다.

**useEffect 주의사항**

1. eslint-disable-line, react-hooks/exhaustive-deps 주석은 최대한 자제하라
   - useEffect는 반드시 의존성 배열로 전달한 값의 변경에 의해 실해돼야하는 hook 이다.
   - 의존성 배열을 넘기지 않는다는 것은 state,props 의 변경과 별개로 부수효과가 발생하는 것이다.
   - useEffect에 빈배열을 넘기기전에 반드시 여기서 호출하는 것이 최선인지 검토하라.
2. useEffect의 첫번째 인수에 함수명을 부여하라

   - useEffect 의 첫번째 인수에 기명함수를 넣어 줌으로 useEffect의 목적을 파악하기 쉽게할 수 있다.

   ```tsx
   useEffect(
     function logActiveUser() {
       console.log(user.id);
     },
     [user.id]
   );
   ```

3. 거대한 useEffect를 만들지 마라
   - 부수효과가 커지면, 애플리케이션 성능에 좋징낳다.
   - 최대한 간결하고 가볍게 유지하라
   - 적은 의존성 배열을 사용하는 여러개의 useEffect 로 분리하는 것이 좋다.
   - 만약 여러 변수가 의존성 배열에 들어가야하면, useCallback 과 useMemo 등으로 정제한 내용을 useEffect 에 담아라
4. 불필요한 외부 함수를 만들지 마라
   - useEffect의 콜백함수로 외부의 함수를 가져다 쓰는 경우 useCallback으로 함수를 감싸고, 이 함수를 의존성 배열에 넣기도 하는데, useEffect 내부에 해당함수를 선언해서 의존성 배열도 제거하고, 더 간결하게 사용하면 좋다.

**왜 useEffect의 콜백인수로 비동기 함수를 바로 못넣을까?**

- useEffect의 경쟁상태: 만약 아래의 코드가 가능하다고 했을 때, 첫번째 호출된 비동기함수가 두번째 호출된 비동기함수 보다 늦게 응답을 받은 경우에 이전 state 기반으로 결과가 나온다.

```tsx
useEffect(async () => {
  // effect callbacks are synchronous to prevent race conditions
  const response = await fetch("http://sample.data.com");
  const result = await response.json();
  setData(result);
}, []);
```

useEffect 내에서 비동기를 구현할 때는 다음과 같이 해보자.

```tsx
useEffect(() => {
  let shouldIgnore = false;
  async function fetchData() {
    const response = await fetch("http://sample.data.com");
    const result = await response.json();
    if (!shouldIgnore) {
      setData(result);
    }
  }
  fetchData();
  return () => {
    // 또는 AbortController 를 등록해 이전 fetch 함수를 abort해주자.
    shouldIgnore = true;
  };
}, []);
```

### 3.1.3 useMemo

비용이 큰 연산에 대한 결과값을 저장(메모이제이션) 해두고, 이 저장된 값을 반환하는 훅

- 랜더링 발생시, 의존성 배열이 변경될 때만, 함수를 재 실행한다.
- 메모이제이션은 컴포넌트도 가능한데, 컴포넌트는 React.memo를 쓰면 된다.

```tsx
import { useMemo } from "react";

/*
인자1: 어떠한 값을 반환하는 생성함수 
인자2: 해당 함수가 의존하는 값의 배열을 전달
*/
const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b]);

const memoizedComponent = useMemo(
  () => <ExpensiveComponent value={value} />,
  [value]
);
```

### 3.1.4 useCallback

useMemo가 값을 기억한다면, useCallback은 인수로 넘겨받은 콜백자체를 기억한다. useCallback은 특정 함수를 재생성하지 않고 재사용한다.

왜 특정함수를 재생성하지 않아야할까?
**예시**
다음 컴포넌트에서 ChildComponent "1" 버튼을 눌렀을 때, "2" 버튼도 같이 리랜더링된다. status2는 안바꼈는데? 하지만 App컴포넌트가 리랜더 되면서 toggle2 함수가 바뀌어서 onChange 가 변경이 됨으로 리랜더링된다.

```tsx
function App() {
  const [status1, setStatus1] = useState(false);
  const [status2, setStatus2] = useState(false);
  const toggle = () => {
    setStatus1(!status1);
  };

  const toggle2 = () => {
    setStatus2(!status2);
  };

  return (
    <>
      <ChildComponent name="1" value={status1} onChange={toggle1} />
      <ChildComponent name="2" value={status2} onChange={toggle2} />
    </>
  );
}
```

toggle1, toggle2 를 useCallback 으로 감싸서 사용하면, 의존성 배열이 변경되지 않는 한 함수도 재생성되지 않고, 불필요한 리랜더링을 막는다.

```tsx
function App() {
  const [status1, setStatus1] = useState(false);
  const [status2, setStatus2] = useState(false);
  const toggle1 = useCallback(
    function toggle1() {
      setStatus1(!status);
    },
    [status1]
  );
  const toggle2 = useCallback(
    function toggle2() {
      setStatus2(!status);
    },
    [status2]
  );

  return (
    <>
      <ChildComponent name="1" value={status1} onChange={toggle1} />
      <ChildComponent name="2" value={status2} onChange={toggle2} />
    </>
  );
}
```

useCallback은 useMemo를 이용해서 구현이 되며, Preact 공식문서에도 확인이 가능하다.
반환문으로 함수 선언문을 반환해야하니, 좀더 직관적인 useCallback을 쓰자.

```tsx
const handleClick = useMemo(() => {
  return () => {};
});
```

### 3.1.5 useRef

useState와 유사하게, 랜더링이 일어나도 변경가능한 상태값을 저장한다.
**useState 와의 차이점**

- 반환값인 객체 내부에 있는 current로 값에 접근하고 변경할 수 있다.
- useRef는 값이 변하더라도 랜더링을 발생시키지 않는다.

**useRef가 필요한 이유**
다음과 같이 구현하면 안될까?

- 컴포넌트 실행전 value라는 값이 기본적으로 존재함으로 메모리에 불필요한 값을 가지게 한다.
- 컴포넌트가 여러개 생성될때, 컴포넌트에게 같은 value를 공유한다.
  > 이 문제들을 useRef가 모두 해결해 준다. 컴포넌트가 랜더링 될때 생성되며, 컴포넌트 인스턴스 하나당 하나의 값을 가진다.

```tsx
let value = 0;
function Component() {
  function handleClick() {
    value += 1;
  }
}
```

**useRef사용예시**

1. DOM 에 접근하는 용도

```tsx
import React, { useRef, useEffect } from "react";

const DOMAccessExample = () => {
  const myInputRef = useRef(null); // 랜더링 실행 전에는 초기값 인수를 값으로 가짐

  useEffect(() => {
    const inputElement = myInputRef.current; //이제 input DOM 에 접근 가능

    if (inputElement) {
      inputElement.focus();
    }
  }, []);

  return (
    <div>
      <label htmlFor="myInput">Type Something:</label>
      {/* input DOM 에 ref를 부착 */}
      <input id="myInput" type="text" ref={myInputRef} />
    </div>
  );
};

export default DOMAccessExample;
```

2. 원하는 시점의 값을 랜더링에 영향 미치지 않고 보관하고 싶을 때,

```tsx
const usePrevious = (value) => {
  const prevValueRef = useRef();

  useEffect(() => {
    prevValueRef.current = value;
  }, [value]);

  return prevValueRef.current;
};
```

**Preact의 useRef구현**

- 값이 변경되도 랜더링이 되면 안되기에, 빈배열을 의존성 배열로 줌으로 랜더링 마다 동일한 객체를 가리키는 결과를 낳는다.
- current 값을 바꿔도 객체의 참조값이 유지 된다.

```js
export function useRef(initialValue) {
  currentHook = 5;
  return useMemo(() => ({ current: initialValue }), []);
}
```

### 3.1.6 useContext

props 내려주기( drilling )를 극복하기 위해 등장한 개념으로 하위 컴포넌트 모두에서 자유롭게 사용이 가능하게 해준다.

- provider 에 의존성을 가진다. 가장 가까운 provider 의 값을 사용한다.
- useContext 가 랜더링 최적화를 해주지 않는다. 단지 context의 상태에 접근할 수 있게 하는 것 뿐이다.
  - 예시: useContext 를 사용하는 component 만 랜더링이 일어나야할 것 같은데, 그렇지 않은 컴포넌트도 부모에서 랜더링되면 같이 된다.

### 3.1.7 useReducer

useState 의 심화버전이라고 생각하면 된다.

```
const [state, dispatcher] = useReducer( reducer, initialState, init )
```

- 반환값은 useState 와 같이 길이가 2인 배열
- 인수는 2~3개의 인수가 필요하다

  - reducer
    앞서 선언한 state 와 action을 기반으로 state가 어떻게 변경될지 정의
  - initialState
    useReducer의 초깃값
  - init
    필수가 아님, useState에 함수를 넘겨주면 게으른 초기화가 일어나 듯이, 이곳에 인수를 념겨주는 함수를 넣으면 initialState 를 인수로 init 함수가 실행됨

- initialState로 초기화한 이후에, state 의 업데이트는 action에 따라 정의해둔 reducer를 호출해주는 dispatcher 를 이용하면 된다.

#### 어떨 때 사용?

number 나 boolean 같이 간단한 값을 관라하는 것은 useState로 충분하다. state 하나가 가져야할 값이 복잡하고, 이를 수정하는 경우의 수가 많아진다면 useReducer 로 관리하는 것이 효율적이다.

#### useState 와의 차이?

useReducer 이나 useState 는 둘다 세부 작동과 쓰임에만 차이가 있고, 클로저로 값을 가둬서 state를 관리한다.

#### 서로를 구현하기

- useReducer 로 useState 를 구현할 수 있고, useState 로 useReducer 를 구현할 수 있다.
- Preact의 useState 를 살펴보면, useReducer 로 구현되어 있다.
  useState 를 구현하기 위해서 useReducer의 dispatcher 를 값을 업데이트해서 반환하거나, 새로운 값을 반환하는 함수로 구현해주면 된다.
- useReducer을 useState로 구현?

  - 내부에서 useState 를 사용해서 값을 관리한다.
  - dispatch 함수를 다음과 같이 reducer를 실행해서 결과 값을 반환하게 구현한다. useCallback을 이용해서 메모이제이션 해준다.

  ```ts
  const dispatch = useCallback(
    (action) => setState((prev) => reducer(prev, action)),
    [reducer]
  );
  ```

### useImperativeHandle

#### forwardRef 살펴보기

useRef 로 생성된 ref는 주로 HTMLElement 에 접근하는 용도로 쓰인다. 하지만 리액트 컴포넌트에 직접 넣어서 사용할 수가 없다. ref는 예약어이기 때문이다. 이를 해결하기 위해서 props 의 이름을 다르게 줘서 사용할 수 있다.

```tsx
function ParentComponent() {
  const inputRef = useRef();
  return (
    <>
      <ChildComponent parentRef={inputRef} />
    </>
  );
}
```

forwardRef 는 위와 같이 동일한 작업을 하는 리액트 API

- 다른 HTMLElement 에 ref 를 전달하는 것과 같이 일관성을 제공
- 확실하게 ref 를 전달할 것임을 예측

```tsx
const TextInput = forwardRef((props, ref) => {
  return (
    <div onClick={() => inputRef.current.focus()}>
      <input ref={ref} type="text" />
    </div>
  );
});

function App() {
  const textInputRef = useRef(null);

  return (
    <div>
      <TextInput ref={textInputRef} />
    </div>
  );
}
```

#### useImperativeHandle 이란?

- 부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅
- 부모 컴포넌트에서 노출되는 값을 원하는 대로 바꿀 수 있다.
- ref 가 `{current: <HTMLElement> }` 같은 형태로 HTML만 주입할 수 있는 객체였다면, useImperativeHandle 훅을 사용해 원하는 값과 액션을 정의해서 접근을 할 수 있다.

```tsx
import { forwardRef, useRef, useImperativeHandle } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        focus() {
          inputRef.current.focus();
        },
        scrollIntoView() {
          inputRef.current.scrollIntoView();
        },
      };
    },
    []
  );

  return <input {...props} ref={inputRef} />;
});
```

`<input>` DOM node 를 모두 노출하지 않고 딱 메소드에 접근하고 싶을 수 있다. 그 때 다음과 같이 메소드를 지정을 해줄 수 있다.

### 3.1.9 useLayoutEffect

- 브라우저가 화면을 repaint 하기 전에 작동되는 useEffect
- 동기적으로 작동하기에 useLayoutEffect 가 실행이 되고나서 화면을 그리기에 성능에 문제될 수 있다.
- useEffect와 형태나 사용예제는 동일함

#### 실행순서

- 1. 리액트가 DOM을 업데이트
- 2. useLayoutEffect 실행
- 3. 브라우저에 변경사항을 반영
- 4. useEffect를 실행

#### 사용되는 케이스

DOM 은 계산 됐지만, 이게 화면에 반영되기 전에 하고 싶은 작업을 진행할 때 적용하면 useEffect 보다 자연스러운 사용자 경험을 제공할 수 있다.

- DOM 요소를 기반으로 한 애니메이션
- 스크롤 위치를 제어하는 등 화면에 반영되기 전 하고 싶은 작업

### 3.1.10 useDebugValue

- 리액트 개발 도구에 커스텀 훅에 대한 정보를 표시해주고 싶을 때 사용한다.
- 훅 내부에서만 실행할 수 있다.
- 두번째 인자로 포맷팅 함수를 전달하면, 첫번째 인자의 변화 될 때, 호출한다.

```ts
import { useDebugValue } from "react";

function useOnlineStatus() {
  useDebugValue(isOnline ? "Online" : "Offline");
}

/* 다음과 같이 값을 포맷팅해주는 함수를 두번째 인자에 넣어주면, 포맷팅 된 결과를 리액트 개발 도구에서 보여줌 */
useDebugValue(date, (date) => date.toDateString());
```

### 3.1.11 훅의 규칙

ESLint 규칙 react-hooks/rules-of-hooks 이 존재한다.

#### 규칙

1. 리액트 함수형 컴포넌트, 사용자 정의 훅 두가지 경우만 최상위에서 훅을 호출할 수 있다.
2. 반복문, 조건문, 중첩된 함수 내에서는 훅을 실행할 수없으며, 컴포넌트가 랜더링 될때 동일한 순서로 훅이 호출 되는 것이 보장된다.

## 3.2 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

중복코드를 줄여서 개발효율을 높이고, 유지보수를 쉽게 할 수 있다.

리액트에서 재사용할 수 있는 로직을 관리하는 방법

- 사용자 정의 훅 (custom hook)
- 고차 컴포넌트(higher order component)

### 3.2.1 사용자 정의 훅

서로 다른 컴포넌트 내부에서 같은 로직을 공유할 때 사용하며 이름은 use 로 시작해야한다.

#### 규칙

fetch를 기반으로 HTTP 요청을 하는 사용자 정의 훅을 한번 살펴보자.

```tsx
import "./App.css";
import { useEffect, useState } from "react";

function useFetch<T>(
  url: string,
  { method, body }: { method: string; body?: XMLHttpRequestBodyInit }
) {
  // 응답 결과
  const [result, setResult] = useState<T | undefined>();
  // 요청 중 여부
  const [isLoading, setIsLoading] = useState<boolean>(false);
  // 2xx, 3xx 로 정상 응답인지 여부
  const [ok, setOk] = useState<boolean | undefined>();
  // HTTP setStatus
  const [status, setStatus] = useState<number | undefined>();

  useEffect(() => {
    const controller = new AbortController();
    const signal = controller.signal;

    (async function fetchData() {
      setIsLoading(true);
      try {
        const response = await fetch(url, {
          method,
          signal,
          body,
        });
        const data = await response.json();
        setResult(data);
        setOk(response.ok);
        setStatus(response.status);
      } catch (error) {
        setResult(undefined);
        setOk(false);
        setStatus(undefined);
      } finally {
        setIsLoading(false);
      }
    })();

    return () => {
      controller.abort();
    };
  }, [url, method, body]);
  return { ok, result, isLoading, status };
}

interface Todo {
  userId: number;
  id: number;
  title: string;
  completed: boolean;
}

export default function App() {
  const { isLoading, result, status, ok } = useFetch<Array<Todo>>(
    "https://jsonplaceholder.typicode.com/todos",
    { method: "GET" }
  );

  useEffect(() => {
    if (!isLoading) {
      console.log("fetchResult >>", status);
    }
  }, [status, isLoading]);

  return (
    <div>
      {ok
        ? (result || []).map(({ userId, title }, index) => (
            <div key={index}>
              <p>{userId}</p>
              <p>{title}</p>
            </div>
          ))
        : null}
    </div>
  );
}
```

만약 훅으로 분리하지 않았다면 fetch API 호출을 해야하는 컴포넌트 내에서 최소 4개의 state 를 선언해서 구현해야하고, useReducer 를 썼다해도, useEffect 도 같이 써야한다.

#### 사용자 정의 훅 저장소 커뮤니티

- use-Hooks
- react-use
- ahooks

### 3.2.2 고차 컴포넌트

- HOC(Higher Order Component) 는 컴포넌트 자체의 로직을 재사용하기 위한 방법

#### React.memo란?

- 리액트에서 제공하는 API로 가장 유명한 고차 컴포넌트이다.
- props 를 비교해 이전과 props가 같다면 랜더링 자체를 생략하고 이전에 기억해 둔 (memoization) 컴포넌트를 반환한다.
- useMemo 를 사용하면 JSX 함수가 아닌, 값을 반환함으로 다음과같이 할당식을 사용해야한다.

```tsx
function Component() {
  const MemoComponent = memo(ChildComponent);
  const MemoizedChildComponent = useMemo(() => {
    return <ChildComponent />;
  }, []);
  return (
    <div>
      {MemoizedChildComponent}
      <MemoComponent />
    </div>
  );
}
```

#### 고차컴포넌트 만들기

고차 컴포넌트를 만들기 위해 고차함수 부터 한번 정리하고 넘어가자.

- 고차 함수란?
  - 함수를 인수로 받거나 결과로 반환하는 함수
  - 함수를 생성할때 선언한 환경 클로저에 변수들이 기억되어, 이 원리를 이용해서 다양한 작업을 해볼 수 있다.

고차 컴포넌트는 컴포넌트를 인수로 받아서 추가적인 기능을 수행하며, 기존 인수로 받는 컴포넌트의 props 는 건드려서는 안된다. 그외의 별도의 props는 상관이 없다.

```tsx
// Define the Higher-Order Component
function withLogging(WrappedComponent) {
  // Return a functional component
  return function WithLoggingComponent(props) {
    // Log props when component mounts
    useEffect(() => {
      console.log("Component props:", props);
    }, [props]);

    // Render the wrapped component with its props
    return <WrappedComponent {...props} />;
  };
}
```

### 3.2.3 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

랜더링에 영향을 미치는 로직도 재사용이 필요한 경우라면, 고차 컴포넌트로 구현을 하면 좋다.

```tsx
function HookeComponent() {
  const { loggedIn } = useLogin();
  if (loggedIn) {
    return <LoginComponent />;
  }

  return <>로그인 하시오</>;
}

//위의 상태에 따라 조건부 랜더링해주는 부분이 고차 컴포넌트안에서 처리를 해준다.
const HOCComponent = withLoginComponent(() => {
  <LoginComponent />;
});
```
