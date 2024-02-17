# 5장 리액트와 상태 관리 라이브러리

## 5.1 상태 관리는 왜 필요한가?

### 웹 어플리케이션에서 상태란?

- 의미를 지니며, 웹 어플리케이션의 시나리오에 따라 지속적으로 변경되는 값
- 상태로 분류되는 것
  - UI
    - 예) 다크/라이트 모드, 알림창의 노출 여부
  - URL
    - 예) 주소속 query param
  - 폼
    - 예) 로딩여부, 제출여부, 유효성 등
  - 서버에서 가져온 값
    - 예) API 요청

### 상태관리의 필요성

- 상태들이 점차 증가했고, 이를 효과적으로 관리하는 방법을 고민하게 됨
- tearing 현상을 어떻게 방지할 것인가?
  - tearing 현상 : 상태 변화가 일어남에 따라 모든 요소들이 변경되며 애플리케이션이 찢어지는? 현상
  - 참고자료
    - [github discussion](https://github.com/reactwg/react-18/discussions/69)
    - [관련 영상](https://www.youtube.com/watch?v=V1Ly-8Z1wQA&t=1079s)

### 리액트 상태 관리의 역사

- Flux 패턴의 등장
  - 시기
    - 2014 리액트 등장과 비슷한 시기에 FLUX 패턴과 이를 사용한 FLUX 라이브러리 등장
  - 배경
    - 웹어플리케이션 규모가 커지고 상태도 많아짐에 따라 양방향 데이터 바인딩으로 데이터가 변경되는 것이 관리가 어려움
      - 데이터 관리를 단방향으로 하자. view 에서 바로 상태 변경을 못하게 하자. -> 데이터의 흐름을 추적하기 쉽고, 코드 이해가 용이 해진다.
  - 구조
    - https://facebookarchive.github.io/flux/docs/in-depth-overview
      - 액션(Action): 어떤 작업을 처리할 액션과 액션을 발생할때 필요한 데이터 이를 디스패처(Dispatcher)로 보낸다 .
      - 디스패처(Dispatcher): 액션을 스토어에 보내는 역할
      - 스토어(Store): 실제 상태에 따른 값과 상태를 변경할 수 있는 메서드
      - 뷰(View): 리액트 컴포넌트에 해당하는 부분, 스토어에서 만들어진 데이터를 화면에 랜더링한다. 사용자의 인터렉션에 따라 뷰에서 액션을 호출 할 수 있다.
  - 단점
    - 사용자의 입력에 따라 데이터를 갱신하고, 어떻게 업데이트 할지 코드를 작성해야하니 코드 양과 수고로움은 증가한다.
- 시장 지배자 리덕스의 등장
  - 배경
    - 리액트와 단방향 데이터 흐름이 두각을 나타날때 등장했다.
  - 설명
    - Flux 구조에 Elm 아키텍처를 도입했다.
      - Elm?
        - 웹페이지를 선언적으로 작성하기 위한 언어
        - 예시
          ```elm
          module Main exposing (..)
          import Browser
          import Html exposing (Html, div, h1, p, button, text)
          -- Model 정의
          type Model =
              { message : String
              }
          -- 초기 상태
          init : Model
          init =
              { message = "Elm로 HTML 생성하기"
              }
          -- Msg 타입 정의
          type Msg
              = ButtonClicked
          -- Update 함수
          update : Msg -> Model -> Model
          update msg model =
              case msg of
                  ButtonClicked ->
                      { model | message = "버튼이 클릭되었습니다!" }
          -- View 함수
          view : Model -> Html Msg
          view model =
              div []
                  [ h1 [] [ text model.message ]
                  , p [] [ text "이것은 Elm을 사용하여 생성된 간단한 HTML입니다." ]
                  , button [ onClick ButtonClicked ] [ text "클릭하세요!" ]
                  ]
          -- Elm 프로그램 실행
          main =
              Browser.sandbox { init = init, update = update, view = view }
          ```
          - Elm 아키텍처
            - 모델(model) : 애플리케이션 상태
            - 뷰(View) : 모델을 표현하는 HTML
            - 업데이트(Update) : 모델을 수정하는 방식
    - 순서
      - 1.  하나의 상태객체를 스토어에 저장한다.
      - 2.  이 객체를 업데이트 하는 작업을 디스패치해 reducer 함수로 상태를 업데이트 한다.
      - 3.  새로운 상태가 생성되면 어플리케이션에 전파한다.
  - 단점
    - 하고자 하는 일에 비해 보일러플레이트가 많다.
      - 액션 타입, 액션을 수행할 creator 함수, dispatcher, selector ... 등
    - 보완: 현재는 이러한 작업을 간소화 해줄 수 있게 발전됨
- Context API 와 useContext
  - 3.1 리액트의 모든 훅 파헤치기에서 이야기 했듯이 Context API 는 상태관리가 아닌 주입을 도와 주는 기능이다.
    - Q. 하지만 상태를 변경할 수 있는 함수를 context 상태 객체에 포함을 시키면 상태변경도 되는데, 그건 ... 관리라고 하기엔 내장기능이 아니어서 ??
  - 리액트 16.3 버전 이전 context
    - getChildContext() 메서드로 자식에게 context 를 넘겨주는 방식이었다.
    - 문제점
      - 상위 컴포넌트가 getChildContext를 호출하면서 shouldComponentUpdate 가 실행되면서 불필요한 렌더링이 일어난다.
      - context 를 인수로 받아야 사용할 수 있었다.
  - 리액트 16.3 새로운 context
    - 이전 context 의 문제점을 해결한 방식
      ```tsx
      import React, { createContext, useContext, useState } from "react";
      // Context 생성
      const MyContext = createContext();
      // Context의 Provider 컴포넌트
      const MyProvider = ({ children }) => {
        const [count, setCount] = useState(0);
        const increment = () => {
          setCount(count + 1);
        };
        // Provider는 값을 제공하며, 이 값을 소비하는 컴포넌트들은 MyContext.Provider로 감싸진 상위 계층에서 값을 가져올 수 있습니다.
        return (
          <MyContext.Provider value={{ count, increment }}>
            {children}
          </MyContext.Provider>
        );
      };
      // 값을 사용할 컴포넌트
      const MyComponent = () => {
        // useContext를 통해 MyContext에서 값을 가져옵니다.
        const { count, increment } = useContext(MyContext);
        return (
          <div>
            <p>Count: {count}</p>
            <button onClick={increment}>Increment</button>
          </div>
        );
      };
      // App 컴포넌트에서 MyProvider로 감싸져 있는 경우 MyComponent에서 MyContext에 접근할 수 있습니다.
      const App = () => {
        return (
          <MyProvider>
            <MyComponent />
          </MyProvider>
        );
      };
      export default App;
      ```
      - createContext 로 context 를 생성후 provider 로 상태를 주입한다.
      - Consumer 나 useContext 이용해서 상태에 접근한다.
- 훅의 탄생, 그리고 React Query와 SWR
  - 훅의 탄생
    - 리액트 16.8 에서 함수형 컴포넌트에 사용할 수 있는 다양한 훅이 추가 되며, state 도 손쉽게 재사용이 가능하게 변경이 되었다.
    - 간결하고 직관적인 훅의 방식으로, 다양한 자신만의 훅들을 만들기 시작한다.
      ```tsx
      const useToggle = (initialValue) => {
        const [value, setValue] = useState(initialValue);
        const toggle = () => {
          setValue(!value);
        };
        return [value, toggle];
      };
      ```
  - HTTP 요청 상태관리 라이브러리
    - React Query
      ```tsx
      import { useQuery } from "react-query";
      const fetchData = async () => {
        const response = await fetch("https://api.example.com/data");
        return response.json();
      };
      const MyComponent = () => {
        const { data, isLoading, error } = useQuery("myData", fetchData);
        if (isLoading) return <p>Loading...</p>;
        if (error) return <p>Error: {error.message}</p>;
        return <p>Data: {data}</p>;
      };
      ```
    - SWR
      ```tsx
      import useSWR from "swr";
      const fetcher = (url) => fetch(url).then((res) => res.json());
      const MyComponent = () => {
        const { data, error } = useSWR("https://api.example.com/data", fetcher);
        if (!data) return <p>Loading...</p>;
        if (error) return <p>Error: {error.message}</p>;
        return <p>Data: {data}</p>;
      };
      ```
      - 첫번째 인수인 API 주소가 키로도 사용된다.
      - 동일한 API 키로 호출하면, 재조회 하지않고, useSWR 의 캐시 값을 활용한다.
- Recoil, Zustand, Jotai, Valtio
  ```tsx
  // Recoil (React State Management Library)
  import { atom, useRecoilState } from "recoil";
  const recoilState = atom({
    key: "recoilState",
    default: 0,
  });
  function RecoilComponent() {
    const [state, setState] = useRecoilState(recoilState);
    return (
      <div>
        <p>Recoil State: {state}</p>
        <button onClick={() => setState(state + 1)}>Increment</button>
      </div>
    );
  }
  // Jotai (React State Management Library)
  import { atom, useAtom } from "jotai";
  const jotaiAtom = atom(0);
  function JotaiComponent() {
    const [state, setState] = useAtom(jotaiAtom);
    return (
      <div>
        <p>Jotai State: {state}</p>
        <button onClick={() => setState((prev) => prev + 1)}>Increment</button>
      </div>
    );
  }
  // Zustand (React State Management Library)
  import create from "zustand";
  const useZustand = create((set) => ({
    zustandState: 0,
    increment: () => set((state) => ({ zustandState: state.zustandState + 1 })),
  }));
  function ZustandComponent() {
    const { zustandState, increment } = useZustand();
    return (
      <div>
        <p>Zustand State: {zustandState}</p>
        <button onClick={increment}>Increment</button>
      </div>
    );
  }
  // Valtio (React State Management Library)
  import { proxy, useProxy } from "valtio";
  const valtioState = proxy({ valtioState: 0 });
  function ValtioComponent() {
    const state = useProxy(valtioState);
    return (
      <div>
        <p>Valtio State: {state.valtioState}</p>
        <button onClick={() => (valtioState.valtioState += 1)}>
          Increment
        </button>
      </div>
    );
  }
  ```
  - 특징
    - 훅을 활용해 원하는 만큼의 상태를 지역적으로 관리하는 것이 요즘( 2024 ) 떠오르는 상태 관리 라이브러리들의 특징이다.
    - 리덕스, MobX 도 react-redux 나 mobx-react-lite 를 설치하면 훅으로 상태를 가져올 수 있지만, 위 라이브러리들은 라이브러리 설치없이 그것이 가능하다.

### 정리

- 다양한 상태관리 라이브러리가 나오고 있는데, 다양한 옵션을 살펴보고 어떻게 구현하는지도 살펴보고 선택을 해보자.

## 5.2 리액트 훅으로 시작하는 상태 관리

리액트와 리덕스를 마치 하나의 프레임워크, 업계 표준으로 여기는 것에서, 리액트 16.8에서 등장한 훅과 함수형 컴포넌트 패러다임으로 Context API, useState, useReducer 를 기반으로 컴포넌트 내부에서 지역적으로 상태를 관리하는 방식이 떠오르고 있다 ( 2024 기준 )

### 1. 가장 기본적인 방법: useState와 useReducer

```tsx
function useCounter(initCount: number = 0, amount: number = 1) {
  const [counter, setCounter] = useState(initCount);
  function inc() {
    setCounter((prev) => prev + 1);
  }
}
```

- 훅을 사용할 때마다 컴포넌트별로 초기화 되므로 컴포넌트별로 다른 상태를 가진다. 이를 지역상태 (localState) 라 한다.
- 여러 컴포넌트에 걸쳐 공유하려면, 공유하려는 컴포넌트의 상위 컴포넌트에서 사용해서 또 props로 값을 내려주는 식으로 컴포넌트 트리를 설계해야한다.

### 2. 지역 상태의 한계를 벗어나보자: useState의 상태를 바깥으로 분리하기

- 조건
  - 컴포넌트 외부에 상태를 두고 여러 컴포넌트가 같이 쓸 수 있어야한다.
  - 외부의 상태를 참조하는 모든 컴포넌트는 상태가 변화될 때마다 리랜더링되어 최신 상태를 기준으로 랜더링 되어야한다.
  - 상태가 원시값이 아닌 객체일때 감지하지않는 값이 변할때는 리랜더링이 발생해서는 안된다.
- 직접구현하기
  ```tsx
  type Initializer<T> = T extends any ? T | ((prev:T)=>T : never
  type Store<State> = {
  get: () => State
  set: ( action :Initializer<State>) =>State
  subscribe: ( callback : ()=> void ) => () => void
  }
  export const createStore = <State extends unknown>(initialState: Initializer<State>,):Store<State> =>{
  // state 또는 게으른 초기화 함수로 store 기본값을 초기화
  let state = typeof initialState !== 'function'? initialState : initialState()
  // Set 을 이용해서 중복없이 callback 함수를 등록
  const callbacks = new Set<()=>void>()
  // 매번 최신값을 가져오게 합니다
  const get = () => state
  // 내부 변수를 최신화하고, 등록된 callback을 모조리 실행합니다.
  const set = (nextState:State | ( prev:State)=> State ) => {
  	state = typeof nextState = "function" ?(nextState as 	(prev: State) => State)(state) : nextState
  	callbacks.forEach((callback)=> callback())
  	return state
  }
  // subscribe 는 callback set 에 callback 을 등록합니다.
  const subscribe = ( callback : () => void ) => {
  	callbacks.add(callback)
  	return ( )=> {
  	callbacks.delete(callback)
  }
  }
  }
  // 이제 컴포넌트 랜더링을 유도할 사용자 정의 훅이 필요하다
  // 단 객체의 다른 속성 영향을 안 받게 하기 위해서 다음과 같이 selector 로 state 를 관리한다.
  export const useStoreSelector = <State extends unknown, Value extends unknown>(store:Store<State>,selector:(state:State) => Value ) => {
  	const [state, setState] =useState(()=>selector(store.get()))
  useEffect(()=>{
  	const unsubscribe = store.subscribe(()=>{
  		const value = selector( store.get() )
  		setState(value)
  	})
  	return unsubscribe
  	},[store, selector ])
  	return state
  	}
  }
  ```
  - useStoreSelector 대신 React에서 지원하는 useSubscription 훅으로 동일한 기능을 구현할 수 있습니다.
    ```tsx
    import React, { useMemo } from "react";
    import { useSubscription } from "use-subscription";
    // In this example, "input" is an event dispatcher (e.g. an HTMLInputElement)
    // but it could be anything that emits an event and has a readable current value.
    function Example({ input }) {
      // Memoize to avoid removing and re-adding subscriptions each time this hook is called.
      const subscription = useMemo(
        () => ({
          getCurrentValue: () => input.value,
          subscribe: (callback) => {
            input.addEventListener("change", callback);
            return () => input.removeEventListener("change", callback);
          },
        }),
        // Re-subscribe any time our input changes
        // (e.g. we get a new HTMLInputElement prop to subscribe to)
        [input]
      );
      // The value returned by this hook reflects the input's current value.
      // Our component will automatically be re-rendered when that value changes.
      const value = useSubscription(subscription);
      // Your rendered output goes here ...
    }
    ```
    - [use-subscription npm link](https://www.npmjs.com/package/use-subscription)
    - 변경이 진행 되는 동안에,

### useState 와 Context를 동시에 사용해보기

```tsx
// 전체가 공유할 Store Context 를 만들어 줍니다.
export const CounterStoreContext = createContext<Store<CounterStore>>(createStore<Counter Store>({count:0, text:'hello' }))
// 지역에서만 공유할 Store 를 초기화해주는 Provider
export const CounterStoreProvider = ({initialState, children}):PropsWithChildren<{initialState:CounterStore}> )=> {
const storeRef = useRef<Store<CounterStore>>()
if(!storeRef.current){
	storeRef.current = createStore(initialState)
}
return (
	<CounterStoreContext.Provider value={storeRef.current}>
	{children}
	</CounterStoreContext.Provider>
)
}
```

- 다음과 같이 context 로 store 를 제공하면, 자식 컴포넌트에 따라 보여주고 싶은 데이터를 Context 로 잘 격리하면 된다.

### 상태관리 라이브러리 살펴보기

- Recoil
  - 특징
    - 하위 메타에서 주도적 개발로, 새로운 리액트 기능에대해 잘 지원할 것으로 기대
      - 리액트 18에서 제공되는 동시성 랜더링, 서버 컴포넌트, Streaming SSR 등 지원 예상
    - 불확실한 정식버전 오픈 일정
      - 2020 처음 만들어졌는데 여전히 실험 운영 중
      - 안정성, 성능을 보장할 수 없어서 서비스에 채택하기 어려운 부분
  - 핵심
    ```tsx
    // App.js
    import React from "react";
    import {
      RecoilRoot,
      atom,
      selector,
      useRecoilState,
      useRecoilValue,
      useSetRecoilState,
    } from "recoil";
    // Atom for the counter state
    const counterState = atom({
      key: "counterState",
      default: 0,
    });
    // Selector to double the counter value
    const doubledCounter = selector({
      key: "doubledCounter",
      get: ({ get }) => get(counterState) * 2,
    });
    function CounterComponent() {
      // Using useRecoilState to read and update the counter state
      const [count, setCount] = useRecoilState(counterState);
      // Using useRecoilValue to read the doubled counter value without subscribing to updates
      const doubledValue = useRecoilValue(doubledCounter);
      // Using useSetRecoilState to get the setter function for the counter state
      const setCountDirectly = useSetRecoilState(counterState);
      return (
        <div>
          <p>Counter: {count}</p>
          <p>Doubled Counter: {doubledValue}</p>
          <button onClick={() => setCount(count + 1)}>Increment</button>
          <button
            onClick={() => setCountDirectly((prevCount) => prevCount - 1)}
          >
            Decrement Directly
          </button>
        </div>
      );
    }
    function App() {
      return (
        <RecoilRoot>
          <CounterComponent />
        </RecoilRoot>
      );
    }
    export default App;
    ```
    - 상태 관리
      - RecoilRoot
        - 최상위 컴포넌트로 선언해 스토어 생성
          - 스토어에는 상태값에 접근, 변경할 수 있는 함수 존재
          - 값 변경이 일어나며, 하위 컴포넌트에 알림
      - atom
        - Recoil 의 최소 상태단위
        - 고유 key 로 구별
    - 어떻게 랜더링을 일으키는지
      - 훅
        - 종류
          - useRecoilValue
            - atom 값을 읽어옴
          - useRecoilState
            - useState 처럼 값을 가져오고 변경하는 훅
        - atom 의 상태 변화를 구독하고, forceUpdate 통해 리랜더링 실행
- Jotai
  - 특징 (v 1.8.3 기준)
    - atom
      - recoil 처럼 최소 상태 단위
      - atom 하나로 파생된 상태도 가능
      - atom 의 값은 store 에 존재, atom 객체 그 자체를 키로 활용해서 저장
      - atom 값이 변경되면 renderIfChanged 가 불리며 최신값을 랜더링
    - useAtomValue 로 getter만 가지고 올 수도 있다.
    - localStorage 연동등 다양한 작업을 Jotai 에서 지원한다.
   - 장점
     - 타입을 잘 지원
     - 리액트 18 변경 API 원활한 지원
     - recoil에 많은 영감을 받아 유사한 부분이 있고, 보완하려는 노력이 보임
       - atom key 관리가 필요없게 추상화됨
       - atom 에서 파생된 값을 만들기 위해 selector 를 사용할 필요가 없음
       - 메모이제이션이나 최적화 없이도 불필요한 리랜더링을 줄인 설계
     - recoil과 다르게 정식버전도 출시되고
  - 코드
    ```tsx
    // App.js
    import React from "react";
    import { Provider, atom, useAtom } from "jotai";
    // Atom for the counter state
    const counterAtom = atom(0);
    function CounterComponent() {
      // Using useAtom to read and update the counter state
      const [count, setCount] = useAtom(counterAtom);
      return (
        <div>
          <p>Counter: {count}</p>
          <button onClick={() => setCount(count + 1)}>Increment</button>
          <button onClick={() => setCount(count - 1)}>Decrement</button>
        </div>
      );
    }
    function App() {
      return (
        <Provider>
          <CounterComponent />
        </Provider>
      );
    }
    export default App;
    ```
- zustand(v4.1.1)
  - 특징
    - 리덕스에 영감을 받아 하나의 스토어를 중앙집중형으로 관리하는 방식
      - 스토어가 어떻게 만들어 지나? 
  - 코드
    ```tsx
    // App.tsx
    import React from "react";
    import create from "zustand";
    // Define the store
    interface CounterState {
      count: number;
      increment: () => void;
    }
    const useCounterStore = create<CounterState>((set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 })),
    }));
    // CounterComponent
    const CounterComponent: React.FC = () => {
      const { count, increment } = useCounterStore();
      return (
        <div>
          <p>Count: {count}</p>
          <button onClick={increment}>Increment</button>
        </div>
      );
    };
    // App Component
    const App: React.FC = () => {
      return (
        <div>
          <CounterComponent />
        </div>
      );
    };
    export default App;
    ```

### 정리

- 상태를 관리하는 라이브러리들은 상태관리에 있어서 조금 차이가 있고, 리랜더링을 일으키는 부분은 거의 동일하다 ( 리액트의 리랜더링 방법이 제한적 ) 따라서 다운로드와 이슈 관리가 잘되는 라이브러리를 선택하면 좋지 않을까
