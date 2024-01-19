## 1. 상태 관리는 왜 필요한가?

### 리액트 상태 관리의 역사

Flux 패턴의 등장

![MVC 패턴은 모델과 뷰가 많아질수록 복잡도가 증가](https://velog.velcdn.com/images/andy0011/post/4a1f159f-6972-4028-8a18-9a383cf5e44d/image.png)

MVC 패턴은 모델과 뷰가 많아질수록 복잡도가 증가

!https://miro.medium.com/v2/resize:fit:1400/0*ZTe7dIoLlUFYFbML.png

- 액션(Action): 어더한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터를 의미
- 디스패처(dispatcher): 액션을 스토어에 보내는 역할을 함, 액션이 정의한 **타입과 데이터를 모두 스토어**에 보냄
- 스토어(store): 실제 상태에 따른 값과 상태를 변경할 수 있는 메서드를 가지고 있다.
- 뷰(view): 리액트의 컴포넌트이고, 스토어에서 만들어진 데이터를 가져와 화면을 렌더링하는 역할을 한다.

단방향 데이터의 흐름 방식의 단점은 사용자의 입력에 따라 데이터를 갱신하고 화면을 어떻게 업데이트해야 하는지도 코드로 작성해야 하므로 코드의 양이 많아지고 개발자도 수고로워진다.

장점은 데이터의 흐름은 모두 액션에 의해 한 방향으로 고정되기에 **데이터의 흐름을 추적하기 쉽고 코드를 이해하기 수월**해진다.

**시장 지배자 리덕스의 등장**

리덕스는 Flux 패턴을 따르고 Elm 아키텍처를 도입했다.

<aside>
💡 [Elm 아키텍처란?](https://kyunooh.gitbooks.io/elm/content/architecture/)

- Model — 어플리케이션의 상태 the state of your application
- Update — 상태를 수정하는 부분 a way to update your state
- View — HTML로 상태를 표현하는 부분 a way to view your state as HTML

```elm
import Html exposing (..)

-- MODEL

type alias Model = { ... }

-- UPDATE

type Msg = Reset | ...

update : Msg -> Model -> Model
update msg model =
  case msg of
    Reset -> ...
    ...

-- VIEW

view : Model -> Html Msg
view model =
  ...

```

</aside>

리덕스를 쓰기 위해서는 해야할 사전 작업이 매우 많기 때문에 불편했다.
상태를 변경하기 위해 매우 많은 작업이 필요

- 예시 코드
    
    ```jsx
    // Actions
    const INCREMENT = "INCREMENT";
    const DECREMENT = "DECREMENT";
    
    // Action Creator
    export const increment = () => {
        return {
            type: INCREMENT,
        };
    }
    export const decrement = () => {
        return {
            type: DECREMENT,
        };
    }
    
    // 초기값 설정
    const initialState = {
        number: 0,
    };
    
    // counterReducer
    export default function store(state = initialState, action) {
        switch (action.type) {
            case INCREMENT:
                return {
                    number: state.number + 1,
                };
            case DECREMENT:
                return {
                    number: state.number - 1,
                };
            default:
                return state;
        }
    }
    출처: https://warmdeveloper.tistory.com/56 [하새의 블로그:티스토리]
    ```
    
    ```
    import { combineReducers } from "redux";
    import store from "./store";
    
    export default combineReducers({
        counter: store,
    });
    
    ```
    
    ```jsx
    import React from 'react';
    import ReactDOM from 'react-dom/client';
    import './index.css';
    import App from './App';
    
    import { createStore } from "redux";
    import { Provider } from "react-redux";
    import rootReducer from "./store/reducer";
    
    const root = ReactDOM.createRoot(document.getElementById('root'));
    const store = createStore(rootReducer);
    
    root.render(
      <React.StrictMode>
          <Provider store={store}>
    		    <App />
          </Provider>
      </React.StrictMode>
    );
    
    ```
    
    ```jsx
    import { useDispatch, useSelector } from "react-redux";
    import { decrement, increment } from "../store/modules/counter";
    
    const Main = () => {
    	// dispatch를 통해 상태 업데이트
    	const dispatch = useDispatch();
      const count = useSelector((state) => state.counter.number);
       
    	return (
    		<div className={"test"}>
    	    <button onClick={() => dispatch(increment())}> + </button>
    	    <button onClick={() => dispatch(decrement())}> - </button>
        </div>
      )
    }
    export default Main;
    ```
    
    단순히 숫자를 카운트하는 코드이지만 코드 양이 매우 방대해진 것을 볼 수 있다.
    

## 2. 리액트 훅으로 시작하는 상태 관리

### 가장 기본적인 방법: `useState`와 `useReducer`

useState와 useReducer를 사용해서 상태 관리를 만들어 나가는 예제이다. 

먼저 간단한 훅의 코드와 사용법이다.

```tsx
function useCounter(initCounter = 0) {
	const [counter,setCounter] = useState(initCount);
	
	function inc() {
		setCounter((prev) => prev + 1);
	}
	
	return { counter, inc }
}

function Counter1() {
	const { counter, inc } = useCounter();
	
	return (
		<>
			<h3>Counter1: {counter}</h3>
			<button onClick={inc}+</button>
		</>
	)
}
function Counter2() {
	const { counter, inc } = useCounter();
	
	return (
		<>
			<h3>Counter1: {counter}</h3>
			<button onClick={inc}+</button>
		</>
	)
}
```

사용자 정의 훅을 통해 counter상태를 격리하여 손쉽게 관리하였다.

그러나 여기서 컴포넌트별로 훅을 사용하므로 **컴포넌트에 따라 서로 다른 상태**를 가질 수 밖에 없다. (이는 클로저이기 때문에 당연한..?)

두 컴포넌트가 동일한 상태를 바라보게 하기 위해서는 컴포넌트 밖으로 한단계 끌어 올리고 props로 넘기는 방법이 있다.

```tsx

function Counter1() {
	const { counter, inc } = useCounter();
	
	return (
		<>
			<h3>Counter1: {counter}</h3>
			<button onClick={inc}+</button>
		</>
	)
}
function Counter2() {
	const { counter, inc } = useCounter();
	
	return (
		<>
			<h3>Counter1: {counter}</h3>
			<button onClick={inc}+</button>
		</>
	)
}

function Parent() {
	const { counter, inc } = useCounter();
	
	return (
		<>
			<Counter1 counter={counter} inc={inc} />
			<Counter2 counter={counter} inc={inc} />
		</>
	)
}
```

개선을 진행하였지만 props형태로 필요한 컴포넌트에 제공해야 한다는 점이 불편해보인다.

### 지역 상태의 한계를 벗어나보자: useState의 상태를 바깥으로 분리하기

```tsx
export type State = { counter: number };

let state: State = {
	counter: 0,
}

export function get(): State {
	return state;
}

type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

export function set<T>(nextSTate: Initializer<T>) {
	state = typeof nextState === 'function' ? nextState(state) : nextState
}

function Counter() {
	const state = get();
	
	function handelClick() {
		set((prev: State) => ({ counter: prev.counter + 1 }));
	}

	return (
		<>
			<h3>{state.counter}</h3>
			<button onClick={handleClick}>>+</button>
		</>
	)
	
}
```

위 코드는 상태를 잘 업데이트 하고 잘 받아오지만 **리렌더링을 일으키는 장치가 어디에도 존재하지 않기** 때문에 state의 업데이트가 화면에 노출되지 않는다.

리렌더링을 하기 위한 작업

- useState, useReducer의 반환값 중 두 번째 인수(setState, dispatcher)가 어떻게든 호출된다.
- 부모 컴포넌트가 리렌더링 되거나 해당 함수가 다시 실행돼야 한다.

첫번째 방법으로 밖에 할 수 없으니 고쳐보자

```tsx
function Counter1() {
	const [count, setCount] = useState(state);

	function handleClick() {
			set((prev: State) => { 
				const newState = { counter: prev.counter + 1 }

				setCount(newState);
				return newState;
			});
	}

	return (
		<>
			<h3>{state.counter}</h3>
			<button onClick={handleClick}>>+</button>
		</>
	)
}

function Counter2() {
	const [count, setCount] = useState(state);

	function handleClick() {
			set((prev: State) => { 
				const newState = { counter: prev.counter + 1 }

				setCount(newState);
				return newState;
			});
	}

	return (
		<>
			<h3>{state.counter}</h3>
			<button onClick={handleClick}>>+</button>
		</>
	)
}
```

이 코드의 이상한 점이 한 두가지가 아니다. 일단 한쪽의 업데이트로 인해 다른쪽 컴포넌트는 렌더링 되지 않는다. 당연히 다른 컴포넌트는 리렌더링 되지 않기 때문이다. 또, 같은 상태를 바라보지만 컴포넌트 내부에 useState라는 별도의 상태로 다시 관리해야 하는 비효율적인 방식을 써야 한다.

이를 통해 외부의 상태를 참조하면서 리렌더링까지 자연스럽게 일어나려면 다음과 같은 조건을 만족해야 한다.

- window나 gloabal이 아니더라도 **컴포넌트 외부 어딘가에 상태를 두고** 여러 컴포넌트가 같이 쓸 수 있어야 한다.
- 외부의 상태를 사용하는 컴포넌트는 **상태의 변화를 알아채어 변화가 일어날 때 리렌더링이 일어나서 최신 상태값 기준으로 렌더링**해야 한다. (상태를 참조하는 모든 컴포넌트에서도 마찬가지)
- 상태가 객체인 경우에는 **객체 내부의 property중 변하지 않는 값을 참조중인 컴포넌트에서는 리렌더링이 일어나서는 안된다.**

이를 토대로 새로운 상태 관리 코드를 만들어 보자

```tsx
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

type Store<State> = {
	get: () => State
	set: (action: Initializer<State>) => State
	subscribe: (callback: () => void) => () => void
}
```

- get: 변수 대신 함수로 만들어서 항상 새로운 값을 가져오게 함
- set: 기존의 useState와 동일하게 값 또는 함수를 받을 수 있게 함
- subscribe: store의 변경을 감지하고 싶은 컴포넌트들이 자신의 callback 함수를 등록해 두는 곳

다음은 이제 실제 store이다.

```tsx
export const createStore = <State extends unknown>( initialState: Initializer<State>): Store<State> => {
	
	// 스토어 내부에 상태를 정의해서 관리
	let state = typeof initialState !== 'function' ? initialState : initialState();

	// 자료형에 관계없이 유니크한 값을 저장할 수 있는 Set을 사용
	const callbacks = new Set<() => void>()

	const get = () => state;

	const set = (nextState: State | ((prev: State) => State)) => {
		state = typeof nextState === 'function' ? (nextState as (prev: State) => State)(state) : nextState;
		
		// 값의 업데이트가 일어나면 콜백 목록의 전체를 실행한다.
		callbacks.forEach((callback) => callback());

		return state;
	}

	// 받은 콜백을 추가한다.
	const subscribe = (callback: () => void) => {
		callbacks.add(callback);

		/// 클린업 실행시 기존의 콜백은 제거한다.
		return () => {
			callbacks.delete(callback)
		}
	}

	return { get, set, subscribe };
}
```

마지막으로 컴포넌트 렌더링을 유도할 커스텀 훅이다.

```tsx
export const useStore = <State extends unknown>(store: Store<State>) => {
	const [state, setState] = useState<State>(() => store.get());

	useEffect(() => {
		const unsubscribe = store.subscribe(() => {
			setState(store.get());
		});

		return unsubscribe;
	},[store])

	return [state, store.set] as const;
}
```

이훅은 useEffect를 통해서 store의 subscribe를 등록해놓은 뒤에 **store값이 변경될 때 마다 state값이 변경됨을 보장받을 수 있고** 클린업 함수를 통해 callback함수가 중복되지 않도록 방지했다.

사용 하는 방법은 단순하게 useStore를 써서 사용하면 된다.

```tsx
const store = createStore({ count: 0 });

function Counter1() {
	const [state, setState] = useStore(store);

	function handleClick() {
		setState((prev) => { count: prev.count + 1 });;
	}

	return (
		<>
			<h3>{state.counter}</h3>
			<button onClick={handleClick}>>+</button>
		</>
	)
}

```

이 훅은 원하는 방식대로 동작하지만 store값이 [객체인 경우에 문제](https://www.notion.so/6-5a5bcc0b168d4582aaf33734bb49ad71?pvs=21)를 아직 해결하지 못했다.

```tsx
export const useStoreSelector = <State extends unknown, Value extends unknown>(
	store: Store<State>,
	selector: (state: State) => Value
) => {
	const [state, setState] = useState(() => selector(store.get()));

	useEffect(() => {
		const unsubscribe = store.subscribe(() => {
			const value = selector(store.get())
			setState(value);
		})
	
		return unsubscribe;
	},[store, selector])

	return state;
}
```

selector를 인자로 받아서 이제 객체의 값중 일부가 변경되었을 때의 문제를 해결했다.

이러한 내용을 통해 구현된 훅(라이브러리)은 `[useSubscription](https://www.npmjs.com/package/use-subscription)`이 있다. react에서는 `useSyncExternalStore`로 재작성돼 있다.

현재는 교재의 코드가 아닌 react의 `useSyncExternalStore` 로 재작성 되어 있다.

[Point useSubscription to useSyncExternalStore shim (#24289) · facebook/react@4997515](https://github.com/facebook/react/commit/4997515b96eede5ab1ca622e0439a0707f8d4afd#diff-177c7dfb427f9112334848f428127e6267ed3c25284f1ed6bb4b01d683be7ed6)

### useState와 Context를 동시에 사용해 보기

위에서 만들었던 훅은 단점이 있는데, 훅과 스토어를 사용하는 구조는 반드시 하나의 스토어만 가지게 된다는 것이다. 만약 훅을 사용하는 서로 다른 스코프에서 스토어의 구조는 동일하되, 여러 개의 서로 다른 데이터를 공유해 사용하고 싶다면 여러개의 스토어를 만들어야 한다.

- Q. 여기서 나는 의문이 들었음 → “서로 다른 스코프에서 스토어의 구조는 동일하되, 여러 개의 서로 다른 데이터를 공유” 하는 이 작업이 굳이 스토어를 여러개 만들어야 할까 생각이 듦.
    
    하나의 스토어에 여러개의 객체 property로 관리해도 되는것이 아닌가? 하고 생각이 들었다.
    
    ```tsx
    const store1 = createStore({ count: 0 });
    const store2 = createStore({ count: 0 });
    const store3 = createStore({ count: 0 });
    
    function Counter1() {
      const counter = useStoreSelector(store1, useCallback((state) => state.count, []);
    
    	// 생략...
    }
    ```
    
    하지만 이렇게 쓴다면?
    
    ```tsx
    const store = createStore({ 
    	countForSome1: { count: 0 },  
    	countForSome2: { count: 0 }, 
    	countForSome3: { count: 0 }, 
    });
    
    function Counter1() {
      const counter = useStoreSelector(store, useCallback((state) => state.countForSome1.count, []);
    
    	// 생략...
    }
    ```
    
    이렇게 쓴다면 불편해지는게 있을까?? 라고 생각했지만 직접 구현해보면서 불편한점이 보임
    
    [](https://codesandbox.io/p/devbox/c7fz47?layout=%7B%22sidebarPanel%22%3A%22EXPLORER%22%2C%22rootPanelGroup%22%3A%7B%22direction%22%3A%22horizontal%22%2C%22contentType%22%3A%22UNKNOWN%22%2C%22type%22%3A%22PANEL_GROUP%22%2C%22id%22%3A%22ROOT_LAYOUT%22%2C%22panels%22%3A%5B%7B%22type%22%3A%22PANEL_GROUP%22%2C%22contentType%22%3A%22UNKNOWN%22%2C%22direction%22%3A%22vertical%22%2C%22id%22%3A%22clqyyy0ap00063b6lc7q9z9ey%22%2C%22sizes%22%3A%5B85.30715005035246%2C14.692849949647538%5D%2C%22panels%22%3A%5B%7B%22type%22%3A%22PANEL_GROUP%22%2C%22contentType%22%3A%22EDITOR%22%2C%22direction%22%3A%22horizontal%22%2C%22id%22%3A%22EDITOR%22%2C%22panels%22%3A%5B%7B%22type%22%3A%22PANEL%22%2C%22contentType%22%3A%22EDITOR%22%2C%22id%22%3A%22clqyyy0ap00023b6lve742old%22%7D%5D%7D%2C%7B%22type%22%3A%22PANEL_GROUP%22%2C%22contentType%22%3A%22SHELLS%22%2C%22direction%22%3A%22horizontal%22%2C%22id%22%3A%22SHELLS%22%2C%22panels%22%3A%5B%7B%22type%22%3A%22PANEL%22%2C%22contentType%22%3A%22SHELLS%22%2C%22id%22%3A%22clqyyy0ap00033b6lh3ax3u1j%22%7D%5D%2C%22sizes%22%3A%5B100%5D%7D%5D%7D%2C%7B%22type%22%3A%22PANEL_GROUP%22%2C%22contentType%22%3A%22DEVTOOLS%22%2C%22direction%22%3A%22vertical%22%2C%22id%22%3A%22DEVTOOLS%22%2C%22panels%22%3A%5B%7B%22type%22%3A%22PANEL%22%2C%22contentType%22%3A%22DEVTOOLS%22%2C%22id%22%3A%22clqyyy0ap00053b6lz8o4qkdb%22%7D%5D%2C%22sizes%22%3A%5B100%5D%7D%5D%2C%22sizes%22%3A%5B50%2C50%5D%7D%2C%22tabbedPanels%22%3A%7B%22clqyyy0ap00023b6lve742old%22%3A%7B%22tabs%22%3A%5B%7B%22id%22%3A%22clqyyy0ap00013b6lgypvhjg3%22%2C%22mode%22%3A%22permanent%22%2C%22type%22%3A%22FILE%22%2C%22filepath%22%3A%22%2Fpublic%2Findex.html%22%2C%22state%22%3A%22IDLE%22%7D%5D%2C%22id%22%3A%22clqyyy0ap00023b6lve742old%22%2C%22activeTabId%22%3A%22clqyyy0ap00013b6lgypvhjg3%22%7D%2C%22clqyyy0ap00053b6lz8o4qkdb%22%3A%7B%22id%22%3A%22clqyyy0ap00053b6lz8o4qkdb%22%2C%22activeTabId%22%3A%22clqyz4ssr003r3b6lpw6tqdul%22%2C%22tabs%22%3A%5B%7B%22type%22%3A%22TASK_PORT%22%2C%22taskId%22%3A%22start%22%2C%22port%22%3A3000%2C%22id%22%3A%22clqyz4ssr003r3b6lpw6tqdul%22%2C%22mode%22%3A%22permanent%22%2C%22path%22%3A%22%2F%22%7D%5D%7D%2C%22clqyyy0ap00033b6lh3ax3u1j%22%3A%7B%22id%22%3A%22clqyyy0ap00033b6lh3ax3u1j%22%2C%22activeTabId%22%3A%22clqyz4obz00253b6l5gn82spt%22%2C%22tabs%22%3A%5B%7B%22type%22%3A%22TASK_LOG%22%2C%22taskId%22%3A%22start%22%2C%22id%22%3A%22clqyz4obz00253b6l5gn82spt%22%2C%22mode%22%3A%22permanent%22%7D%5D%7D%7D%2C%22showDevtools%22%3Atrue%2C%22showShells%22%3Atrue%2C%22showSidebar%22%3Atrue%2C%22sidebarPanelSize%22%3A15%7D)
    
    실행 해볼 수 있는 환경인데, 일단 먼저 counter하나하나 불변성을 유지하면서 업데이트하는게 매우 불편하고, 함수를 하나로 합칠때도 매우 힘들어짐
    

이 문제를 해결하는 좋은 방법은 react의 context이다.

```tsx
export const useCounterContextSelector = <State extends unknown>(
	selector: (state: CounterStore) => State,
) => {
	const store = useContext(CounterStoreContext);

	const subscription = useSubscription(
		useMemo(
			() => ({
				getCurruntValue: () => selector(store.get()),
				subscribe: store.subscribe,
			}),
			[store, selector],
		)
	)
	
	return [subscription, store.set] as const;
}
```

이를 사용하는 예제이다.

```tsx
const ContextCounter = () => {
	const id = useId();

	const [counter, setStore] = useCounterContextSelector(
		useCallback((state: CounterStore) => state.count, []),
	)
	function handleClick() {
		setStore((prev) => ({ ...prev, count: prev.count + 1}));
	}
	
	return (
		<div>
		{counter}
		<button onClick={handleClick}>>+</button>
		</div>
	)
}

const ContextInput = () => {

	const id = useId();
	const [text, setStore] = useCounterContextSelector(
		useCallback((state: CounterState) => state.text, []),
	)
	
	function handleChange(e: ChangeEvent<HTMLInputElement>) {
    store.set((prev) => ({ ...prev, text: e.target.value }));
  }

  return (
    <>
      <h3>{text}</h3>
      <input value={text} onChange={handleChange} />
    </>
  );
}

export default function App() {
	return (
		<>
			<ContextCounter />
			<ContextInput />
			<CounterStoreProvider initialState={{ count: 10, text: 'hello' }}>
				<ContextCounter />
				<ContextInput />
					<CounterStoreProvider initialState={{ count: 20, text: 'welcome' }}>
						<ContextCounter />
						<ContextInput />
					</CounterStoreProvider>
			</CounterStoreProvider>
		</>
	)

}
```

Context를 사용하는 경우 가까운 Provider를 가져오므로 각각 state를 provider에 따라 격리 시킬 수 있다.

스토어를 기반으로 어떤 값을 보여줄지만 고민하면 되므로 좀 더 편리한 코드를 작성할 수 있다.

### 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기

**페이스북이 만든 상태 관리 라이브러리 Recoil**

Recoil은 페이스북에서 만든 상태관리 라이브러리이다. (현재는 [리코일에 대한 이슈](https://medium.com/@clockclcok/recoil-%EC%9D%B4%EC%A0%9C%EB%8A%94-%EB%96%A0%EB%82%98-%EB%B3%B4%EB%82%BC-%EC%8B%9C%EA%B0%84%EC%9D%B4%EB%8B%A4-ff2c8674cdd5)가 분분..)

RecoilRoot

[](https://github.com/facebookexperimental/Recoil/blob/main/packages/recoil/core/Recoil_RecoilRoot.js#L552)

[replaceState의 구현](https://github.com/facebookexperimental/Recoil/blob/main/packages/recoil/core/Recoil_RecoilRoot.js#L436)

```tsx
const replaceState = (replacer: TreeState => TreeState) => {
    startNextTreeIfNeeded(storeRef.current);
    // Use replacer to get the next state:
    const nextTree = nullthrows(storeStateRef.current.nextTree);
    let replaced;
    try {
      stateReplacerIsBeingExecuted = true;
      replaced = replacer(nextTree);
    } finally {
      stateReplacerIsBeingExecuted = false;
    }
    if (replaced === nextTree) {
      return;
    }

    if (__DEV__) {
      if (typeof window !== 'undefined') {
        window.$recoilDebugStates.push(replaced); // TODO this shouldn't happen here because it's not batched
      }
    }

    // Save changes to nextTree and schedule a React update:
    storeStateRef.current.nextTree = replaced;
    if (reactMode().early) {
      notifyComponents(storeRef.current, storeStateRef.current, replaced);
    }
    nullthrows(notifyBatcherOfChange.current)();
  };
```

상태가 변화하면 이 변경된 상태를 `notifyComponents` 를 통해 하위 컴포넌트로 전파해 리렌더링 한다.

[notifyComponents](https://github.com/facebookexperimental/Recoil/blob/main/packages/recoil/core/Recoil_RecoilRoot.js#L436)

```tsx
function notifyComponents(
  store: Store,
  storeState: StoreState,
  treeState: TreeState,
): void {
  const dependentNodes = getDownstreamNodes(
    store,
    treeState,
    treeState.dirtyAtoms,
  );
  for (const key of dependentNodes) {
    const comps = storeState.nodeToComponentSubscriptions.get(key);
    if (comps) {
      for (const [_subID, [_debugName, callback]] of comps) {
        callback(treeState);
      }
    }
  }
}
```

notifyComponent는 스토어를 사용하고 있는 모든 하위 의존성을 검색한다음 해당 컴포넌트들의 콜백을 모두 실행한다.

- atom: 상태를 나타내는 Recoil의 최소 상태 단위
- useRecoilValue: atom의 값을 읽어오는 훅
- useRecoilState: 단순히 atom의 값을 가져오기도 하고 값을 변경도 할 수 있는 훅

특징

- 리덕스와 달리 redux-saga, redux-thunk 등 추가적인 미들웨어를 사용하지 않고 비동기 작업을 처리할 수 있다.
- 정식 버전인 1.0.0의 출시 시점이 불확실해 주의가 필요

**Recoil에서 영감을 받은, 그러나 조금 더 유연한 Jotai**

[atom](https://github.com/pmndrs/jotai/blob/main/src/vanilla/atom.ts#L88)

```tsx
export function atom<Value, Args extends unknown[], Result>(
  read: Value | Read<Value, SetAtom<Args, Result>>,
  write?: Write<Args, Result>,
) {
  const key = `atom${++keyCount}`

  const config = {
    toString: () => key,
  } as WritableAtom<Value, Args, Result> & { init?: Value }

  if (typeof read === 'function') {
    config.read = read as Read<Value, SetAtom<Args, Result>>
  } else {
    config.init = read
    config.read = function (get) {
      return get(this)
    }
    config.write = function (
      this: PrimitiveAtom<Value>,
      get: Getter,
      set: Setter,
      arg: SetStateAction<Value>,
    ) {
      return set(
        this,
        typeof arg === 'function'
          ? (arg as (prev: Value) => Value)(get(this))
          : arg,
      )
    } as unknown as Write<Args, Result>
  }
  if (write) {
    config.write = write
  }
  return config
}
```

Recoil과 다르게 key를 내부적으로 설정하고 init, read, write를 가진 config를 return한다.

atom에 대한 상태를 저장하는 것은 useAtomValue에서 한다.

[`useAtomValue`](https://github.com/pmndrs/jotai/blob/main/src/react/useAtomValue.ts#L59)

[책에 있는 코드](https://github.com/pmndrs/jotai/blob/476a3f24cb417257d47c319c155b76198a4f9593/src/core/useAtomValue.ts#L8-L86)와 현재는 코드가 달라졌다.

```tsx
export function useAtomValue<Value>(atom: Atom<Value>, options?: Options) {
  const store = useStore(options)

  const [[valueFromReducer, storeFromReducer, atomFromReducer], rerender] =
    useReducer<
      ReducerWithoutAction<readonly [Value, Store, typeof atom]>,
      undefined
    >(
      (prev) => {
        const nextValue = store.get(atom)
				// 최신값을 가져와서 값이 같으면 이전값을 리턴
        if (
          Object.is(prev[0], nextValue) &&
          prev[1] === store &&
          prev[2] === atom
        ) {
          return prev
        }
				// 아니라면 최산상태를 리턴
        return [nextValue, store, atom]
      },
			// 초기값은 undefined
      undefined,
			// 초기화 함수는 초기에 전달하는 값
      () => [store.get(atom), store, atom],
    )

  let value = valueFromReducer

// 이전 상태와 다르면 rerender진행
  if (storeFromReducer !== store || atomFromReducer !== atom) {
    rerender()
    value = store.get(atom)
  }

  const delay = options?.delay
  useEffect(() => {
		// atom을 subscrition하고 클린업에 unsubsciption하는 부분
    const unsub = store.sub(atom, () => {
      if (typeof delay === 'number') {
        // delay rerendering to wait a promise possibly to resolve
        setTimeout(rerender, delay)
        return
      }
      rerender()
    })
    rerender()
    return unsub
  }, [store, atom, delay])

  useDebugValue(value)
  // TS doesn't allow using `use` always.
  // The use of isPromiseLike is to be consistent with `use` type.
  // `instanceof Promise` actually works fine in this case.
  return isPromiseLike(value) ? use(value) : (value as Awaited<Value>)
}
```

useReducer부분을 보면 [valueFromReducer, storeFromReducer, atomFromReducer] 이 state이고 

rerender가 dispatch이다.

[`useAtom`](https://github.com/pmndrs/jotai/blob/main/src/react/useAtom.ts#L38C1-L47C2)

```tsx
export function useAtom<Value, Args extends any[], Result>(
  atom: Atom<Value> | WritableAtom<Value, Args, Result>,
  options?: Options,
) {
  return [
    useAtomValue(atom, options),
    // We do wrong type assertion here, which results in throwing an error.
    useSetAtom(atom as WritableAtom<Value, Args, Result>, options),
  ]
}

// useSetAtom.ts
export function useSetAtom<Value, Args extends any[], Result>(
  atom: WritableAtom<Value, Args, Result>,
  options?: Options,
) {
  const store = useStore(options)

  const setAtom = useCallback(
		// store에 atom에 해당하는 값을 set하는 함수를 제공
    (...args: Args) => {
      // ...
      return store.set(atom, ...args)
    },
    [store, atom],
  )

  return setAtom
}

// Provider.ts
export const useStore = (options?: Options): Store => {
  const store = useContext(StoreContext)
  return options?.store || store || getDefaultStore()
}
```

useAtom은 useState와 동일한 배열을 제공하고 현재의 결과를 반환하는 `useAtomValue`와 atom을 수정할 수 있는 `useSetAtom`을 반환한다.

특징 

- Recoil과 다르게 atom에서 키를 별도로 관리하지 않아도 된다.
- Recoil에서는 atom에서 파생된 값을 만들기 위해서는 selector가 필요했지만, Jotai 에서는 atom만으로 가능

**작고 빠르며 확장에도 유연한 Zustand**

[vanilla.ts](https://github.com/pmndrs/zustand/blob/main/src/vanilla.ts#L60)

```tsx
const createStoreImpl: CreateStoreImpl = (createState) => {
  type TState = ReturnType<typeof createState>
  type Listener = (state: TState, prevState: TState) => void
  let state: TState
  const listeners: Set<Listener> = new Set()

  const setState: StoreApi<TState>['setState'] = (partial, replace) => {
    // TODO: Remove type assertion once https://github.com/microsoft/TypeScript/issues/37663 is resolved
    // https://github.com/microsoft/TypeScript/issues/37663#issuecomment-759728342
  
  const nextState =
      typeof partial === 'function'
        ? (partial as (state: TState) => TState)(state)
        : partial

    if (!Object.is(nextState, state)) {
      const previousState = state
      state =
        replace ?? (typeof nextState !== 'object' || nextState === null)
          ? (nextState as TState)
          : Object.assign({}, state, nextState)
      listeners.forEach((listener) => listener(state, previousState))
    }
  }

  const getState: StoreApi<TState>['getState'] = () => state

  const subscribe: StoreApi<TState>['subscribe'] = (listener) => {
    listeners.add(listener)
    // Unsubscribe
    return () => listeners.delete(listener)
  }

  const destroy: StoreApi<TState>['destroy'] = () => {
    if (import.meta.env?.MODE !== 'production') {
      console.warn(
        '[DEPRECATED] The `destroy` method will be unsupported in a future version. Instead use unsubscribe function returned by subscribe. Everything will be garbage-collected if store is garbage-collected.',
      )
    }
    listeners.clear()
  }

  const api = { setState, getState, subscribe, destroy }
  state = createState(setState, getState, api)
  return api as any
}

export const createStore = ((createState) =>
  createState ? createStoreImpl(createState) : createStoreImpl) as CreateStore
```

partial과 replace를 통해 객체일 때 필요에 따라서 나눠서 사용할 수 있게 해놨다.

해당 코드 내에서는 import문이 하나도 없고 오로지 createStore만 export를 해서 사용한다는 것이다.

**Zustand의 리액트 코드**

[useStore](https://github.com/pmndrs/zustand/blob/main/src/react.ts#L19)

```tsx
import useSyncExternalStoreExports from 'use-sync-external-store/shim/with-selector'
const { useSyncExternalStoreWithSelector } = useSyncExternalStoreExports

export function useStore<TState, StateSlice>(
  api: WithReact<StoreApi<TState>>,
  selector: (state: TState) => StateSlice = api.getState as any,
  equalityFn?: (a: StateSlice, b: StateSlice) => boolean,
) {
  // ...
  const slice = useSyncExternalStoreWithSelector(
    api.subscribe,
    api.getState,
    api.getServerState || api.getState,
    selector,
    equalityFn,
  )
  useDebugValue(slice)
  return slice
}
```

useSyncExternalStoreWithSelector 는 이름만 다를뿐 `useSyncExternalStore` 훅과 동일하지만 **원하는 값을 가져올 수 있는 selector**와 **동등 비교를 할 수 있는 equialityFn 함수**를 받는 다는 차이가 있 다.

이제 react에서 사용할 수 있도록 스토어를 만들어주는 함수(`[createImpl](https://github.com/pmndrs/zustand/blob/main/src/react.ts#L101)`)이다.

```tsx
const createImpl = <T>(createState: StateCreator<T, [], []>) => {
  // ...
  const api =
    typeof createState === 'function' ? createStore(createState) : createState

  const useBoundStore: any = (selector?: any, equalityFn?: any) =>
    useStore(api, selector, equalityFn)

  Object.assign(useBoundStore, api)

  return useBoundStore
}

export const create = (<T>(createState: StateCreator<T, [], []> | undefined) =>
  createState ? createImpl(createState) : createImpl) as Create

export default ((createState: any) => {
  return create(createState)
}) as Create
```

useBoundStore에 [api](https://www.notion.so/6-5a5bcc0b168d4582aaf33734bb49ad71?pvs=21)(createStore)를 그대로 복사해서 넣어서 기존에 Vanilla로 만든 기능을 그대로 가져왔다고 볼 수 있다.

특징

- 코드의 양이 적고 빠르게 스토어를 만들고 사용할 수 있다.
- 라이브러리의 번들링 용량이 다른 라이브러리에 비해 작다

### npm trends

https://npmtrends.com/jotai-vs-recoil-vs-zustand