## 리액트의 모든 훅 파헤치기

### **useState**

함수형 컴포넌트 내부에 상태를 정의하고, 상태를 관리해줄 수 있게 해주는 훅이다.

```tsx
function Component() {
  const [, triggerRender] = useState();

  let state = "hello";

  function handleButtonClick() {
    state = "hi";
    triggerRender();
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleButtonClick}>hi</button>
    </>
  );
}
```

버튼 클릭시 강제로 리랜더링을 해도 state는 hello로 변경되지 않는다. 리렌더링 될때 마다 함수는 매번 다시 실행되고, state는 매번 다시 hello로 초기화되기 때문이다.

그렇다면 useState는 호출된 이후에도 내부에 선언된 지역변수 state를 계속 참조해서(클로저) 상태를 관리할 수 있는 것이다.

**게으른 초기화**

`useState`에 원시값을 넘기는 것이 아닌 함수를 넘기는 것을 게으른 초기화(lazy initialization) 이라고 한다. [초깃값이 복잡하거나 무거운 연산을 포함하고 있을 때 사용하는 것](https://react.dev/reference/react/useState#avoiding-recreating-the-initial-state)이다.

```tsx
import { useState } from "react";

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: "Item " + (i + 1),
    });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  const [text, setText] = useState("");

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button
        onClick={() => {
          setText("");
          setTodos([
            {
              id: todos.length,
              text: text,
            },
            ...todos,
          ]);
        }}
      >
        Add
      </button>
      <ul>
        {todos.map((item) => (
          <li key={item.id}>{item.text}</li>
        ))}
      </ul>
    </>
  );
}
```

함수 자체를 넘겨도 초기에만 동작하고 이후 렌더링에는 동작하지 않는다.

### useEffect

useEffect의 흔한 답변

- useEffect는 두 개의 인수를 받는데, 첫 번째는 콜백, 두 번째는 의존성 배열이다. 이 두번째 의존성 배열의 값이 변경되면 첫 번째 인수인 콜백을 실행
- 클래스형 컴포넌트의 생명주기 메서드와 비슷한 작동을 구현할 수 있다. 두 번째 의존성 배열에 빈 배열을 넣으면 컴포넌트가 마운트될 때만 실행된다.
- useEffect는 클린업 함수를 반환할 수 있는데, 이 클린업 함수는 컴포넌트가 언마운트될 때 실행된다.

위의 정의는 어느정도 맞지만 완전히 정확하진 않다. 또한, **생명주기 메서드를 대체하기 위해 만들어진 훅이 아니다.**

책에서 말하는 정의란 **“애플리케이션 내 컴포넌트의 여러 값들을 활용해 부수 효과를 만드는 매커니즘 이다.”**

useEffect는 의존성 배열이 변경될 때마다 콜백을 실행하지만 이 의존성 배열이 변경된 것을 어떻게 아는가?

```tsx
function Component() {
  const counter = 1;

  useEffect(() => {
    console.log(counter); // 1, 2, 3, 4
  });

  return (
    <>
      <h1>{counter}</h1>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

렌더링할 때마다 의존성에 있는 값을 보면서 이 의존성의 값이 이전과 다른게 하나라도 있으면 부수 효과를 실행하는 평범한 함수이다.

**클린업 함수의 목적**

```tsx
// 최초 실행
useEffect(() => {
  function addMouseEvent() {
    console.log(1);
  }

  window.addEventListner("click", addMouseEvent);

  return () => {
    console.log("클린업 함수 실행!", 1); // 1이 들어감
    window.removeEventListenr("click", addMouseEvent);
  };
}, [counter]);

// 다음 렌더링
useEffect(() => {
  function addMouseEvent() {
    console.log(2);
  }

  window.addEventListner("click", addMouseEvent);

  return () => {
    console.log("클린업 함수 실행!", 2); // 2가 들어감
    window.removeEventListenr("click", addMouseEvent);
  };
}, [counter]);
```

클린업 함수는 새로운 값과 함께 렌더링된 뒤에 실행되지만 새로운 값을 읽는 것이 아닌, 함수가 정의됐을 당시에 선언됐던 이전 값을 보고 실행된다.

만약 여기서 `removeEventListener`를 실행하지 않았으면 이벤트가 무한히 등록될 것이기에 클린업 함수를 넣어줘야 하는 것이다.

클린업 함수는 언마운트의 개념과는 다르고 **말 그대로 이전 상태를 청소해 주는 개념**으로 보는 것이 맞다.

**의존성 배열**

컴포넌트가 랜더링되는지 확인하고 싶을 때 useEffect의 의존성 배열이 없는 형태로 작성하는데 그렇다면 그냥 console을 찍어도 되지 않을까?

```tsx
function Component() {
	console.log('렌더링됨');
}

function Component() {
	useEffect(() => {
		console.log('렌더링됨');
	})
```

두 코드의 차이점

- 서버 사이드 렌더링 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해준다.
  window 객체의 접근에 의존하는 코드를 사용해도 된다.
- useEffect는 컴포넌트 렌더링의 부수 효과, 즉 컴포넌트의 렌더링이 완료된 이후에 실행된다. 그러나 직접 실행은 렌더링되는 도중에 실행되고, 서버 사이드 렌더링의 경우에 서버에서도 실행된다. 또한 만약 무거운 작업을 할 경우 렌더링을 방해해 성능에 악영향을 미칠 수 있다.

**useEffect를 사용할 때 주의할 점**

**eslint-disable-line, [react-hooks/exhaustive-deps](https://github.com/facebook/react/blob/main/packages/eslint-plugin-react-hooks/README.md) 주석은 최대한 자제하라**

의존성 배열에 포함돼 있지 않은 값이 있을 때 발생시키는 ESLint 이다.

반드시 의존성 배열로 전달한 값의 변경에 의해 실행돼야 하는 훅이다. 의존성 배열을 모두 채우지 않는다면 useEffect에서 사용한 콜백 함수의 실행과 내부에서 사용한 값의 실제 변경 사이에 연결고리가 끊어지게 된다.

**useEffect의 첫 번째 인수에 함수명을 부여하라**

useEffect의 코드가 복잡해지고 무슨 일을 파악하기 어려워지면 익명 함수가 아닌 기명 함수로 바꾸는 것이 좋다.

```tsx
useEffct(
  function logActiveUser() {
    logging(user.id);
  },
  [user.id]
);
```

목적을 명확히 하고 책임을 최소한으로 좁힐 수 있게 된다.

**거대한 useEffect를 만들지 마라**

렌더링 시 의존성이 변경될 때마다 부수 효과를 실행하기에 크기가 커질수록 애플리케이션 성능에 악영향을 미친다. 만약 거대한 useEffect를 만들어야 한다면 작은 의존성 배열을 여러개 사용하게 쓰는게 좋다.

**불필요한 외부 함수를 만들지 마라**

```tsx
// Bad
function Component({ id }: { id: string }) {
  const [info, setInfo] = useState<number | null>(null);
  const controllerRef = useRef<AbortController | null>(null);
  const fetchInformation = useCallback(async (fetchId: string) => {
    controllerRef.current?.abort();
    controllerRef.current = new AbortController();

    const result = await fetchInfo(fetchId, {
      sginal: controllerRef.signal,
    });
    setInfo(await result.json());
  }, []);

  useEffect(() => {
    fetchInformation(id);
    return () => controllerRef.current?.abort();
  }, [id, fetchInformation]);
}

// Good
function Component({ id }: { id: string }) {
  const [info, setInfo] = useState<number | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    (async () => {
      const result = await fetchInfo(id, { sginal: controller.signal });
      setInfo(await result.json());
    })();

    return () => controller.abort();
  }, [id]);
}
```

useEffect 외부에 불필요한 함수를 선언해 가독성이 많이 떨어졌다. useEffect 내부로 옮겨서 useCallback도 지우고 가독성도 높였다.

[useEffect에서 발생하는 경쟁 상태(race condition)문제](https://react.dev/learn/you-might-not-need-an-effect#fetching-data)

### useMemo

비용이 큰 연산에 대한 결과를 메모이제이션 해두고 저장된 값을 반환하는 훅이다. 값 뿐만 아니라 컴포넌트 또한 useMemo로 저장이 가능하다.

```tsx
const MemoizedComponent = useMemo(() => <ExpensiveComponent value={value />, [value])
// 물론 React.memo를 쓰는 것이 현명하다.
```

### useCallback

인수로 넘겨받은 콜박 자체를 기억하는 훅이다. 특정 함수를 새로 만들지 않고 다시 재사용하는 것이다.

```tsx
function App() {
  const [status1, setStatus1] = useState(false);

  // 여기에도 기명 함수를 사용
  // 개발자 도구에서 디버깅을 용이하게 하기 위함이다.
  const toggle1 = useCallback(
    function toggle1() {
      setStatus1(!status1);
    },
    [status1]
  );

  return <ChildComponent onChange={toggle1} />;
}
```

자식 컴포넌트에는 이제 status1 이 변하지 않는 한 재생성하지 않는다.

이때 useMemo로도 useCallback을 구현할 수 있다.

```tsx
const toggle1 = useMemo(() => {
  return () => setStatus1(!status1);
}, [status1]);
```

값을 반환하는 대신 함수 선언문을 반환하면 되는데 이는 코드 작성에 혼돈을 야기할 수 있으므로 따로 제공해주는 훅이 있는 것으로 보인다. (역할은 똑같음)

### useRef

useRef는 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다. 이때 useState와 두가지 차이점이 있다.

- useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
- useRef 값이 변하더라도 렌더링을 발생시키지 않는다.

개발자가 원하는 시점의 값을 렌더링에 영향을 미치지 않고 보관해 두고 싶을 때 사용하는 것이 좋다.

```tsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, []);

  return ref.current;
}

function SomeComponent() {
  const [counter, setCounter] = useState(0);
  const previousCounter = usePrevious(counter);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  // 0 undefined
  // 1, 0
  // 2, 1
  return (
    <button onClick={handleClick}>
      {counter} {previousCounter}
    </button>
  );
}
```

이전의 값을 기억하고 있을 수 있다.

### useContect

Context 란?

리액트에서는 부모가 가지고 있는 데이터를 자식에게도 넘겨주려면 props가 가장 일반적이다. 만약 자식의 깊이가 매우 깊다면 props로 필요한 위치까지 계속 내려줘야 하는 props drilling이 발생한다.

단순히 값을 전달하기 위해 props를 받고 사용하는게 매우 번거로워 진다.

**Context를 함수형 컴포넌트에서 사용할 수 있게 해주는 useContext**

```tsx
function ContextProvider({ children, text }: PropsWithChildren<{ text: string }>) {
	return <MyContet.Provider value={{ hello: text}}>{children}</MyContet.Provider>
}

function useMyContext() {
	const context = useContext(MyContext)
	if (context === undefined) {
		throw new Error('useMyContet는 ContextProvider 내부에서만 사용할 수 있습니다');
	}
	return context'
}

function ChildComponent() {
	const { hello } = useMyContext()

	return <>{hello}</>
}

function ParentComponent() {
	return <>
		<ContextProvider text="react">
			<ChildComponent />
		</ContectProvider>
	</>
}
```

콘텍스트 내부에서 해당 콘텍스트가 존재하는 환경인지 체크해주어 타입스크립트의 타입추론까지 명확하게 가져갈 수 있다.

**useContext를 사용할 때 주의할 점**

useContext가 선언돼 있으면 Provider에 의존성을 가지고 있게 되어 재활용이 어려운 컴포넌트가 된다. 컨텍스트가 미치는 범위는 필요한 환경에서 최대한 좁게 만들어야 한다.

그리고 상태관리를 위한 리액트의 API로 오해하고 있는데 **단순히 상태를 주입해주는 API** 로 봐야 한다.

상태 관리 라이브러리가 되기 위한 조건

- 어떠한 상태를 기반으로 다른 상태를 만들어 낼 수 있어야 한다.
- 필요에 따라 이러한 상태 변화를 최적화할 수 있어야 한다.

Context는 둘 중 하나도 하지 못하기에 상태 관리 라이브러리가 아니다.

Q. 캡슐화 관점에서는 오히려 하나의 컨텍스트로 묶으면 재사용성을 높일 수 있을 것 같은데.. 저는 책에서와 좀 다르다고 생각합니다.

### useReducer

useState의 심화버전으로 보면 된다.

`useReducer`의 반환 값

- **state**: 현재 useReducer 가 가지고 있는 값
- **dispatcher**: state를 업데이트하는 함수. state를 변경할 수 있는 액션을 넘기게 된다.

`useReducer`의 인수

- **reducer**: 기본 action을 정의하는 함수
- **initialState**: useReducer의 초깃값을 의미
- **init**: useState의 게으른 초기화 처럼 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수이다.

useReducer로 useState 구현하기

```tsx
function reducer(prevState, newState) {
  return typeof newState === "function" ? newState(prevState) : newState;
}

// 초기값 처리
function init(initialArg: Initializer) {
  return typeof initialArg === "function" ? initialArg() : initialArg;
}

function useState(initialArg) {
  return useReducer(reducer, initialArg, init);
}
```

useState로 useReducer 구현하기

```tsx
const useReducer = (reducer, initialArg, init) => {
  const [state, setState] = useState(init ? () => init(initialArg) : initialArg);

  // dispatch 함수 선언
  const dispatch = useCallback((action) => setState((prev) => reducer(prev, action)), [reducer]);

  return useMemo(() => [state, dispatch], [state, dispatch]);
};
```

둘다 클로저를 활용하여 상태를 관리한다.

### useImperativeHandle

useImperativeHandle을 이해하기 위해 forwardRef를 먼저 알아야 한다.

**forwardRef 살펴보기**

상위 컴포넌트에서 하위 컴포넌트로 ref를 전달하고 싶을때 사용한다. 물론 props를 통해서도 ref를 전달할 수 있지만 리액트에서는 forwardRef를 만들었다.

**네이밍의 자유가 주어진 props보다는 `forwardRef`를 사용하면 좀 더 확실하게 ref를 전달할 것임을 예측할 수 있고 사용하는 쪽에서도 안정적으로 받아서 사용할 수 있게 나타나게 되었다.**

```tsx
const ChildComponent = forwardRef(
  (props, ref) => {
    return <div>안녕!</div>;
  },
  [ref]
);

function ParentComponent() {
  const inputRef = useRef();

  return (
    <>
      <input ref={inputRef} />
      <ChildComponent ref={inputRef} />
    </>
  );
}
```

ref를 사용해서 props를 받고 쓸 수 있게 되어 조금 더 일관적인 방법으로 ref를 사용할 수 있게 된다.

**useImperativeHandle 란?**

부모에게서 넘겨받은 ref를 원하는대로 수정할 수 있는 훅이다.

```tsx
const Input = forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({ alert: () => alert(props.value) }), [props.value])

	return <input ref={ref} {...props} />
})

function App() {
	const inputRef = useRef();

	const handleClick() {
		inputRef.current.alert()
	}

	return <>
		<input ref={inputRef} />
		<button onClick={handleClick} />
	</>
}
```

`forwardRef`에서 useImperativeHandle안에 선언한 `alert`함수를 통해 alert을 호출할 수 있게 된다.

### useLayoutEffect

useEffect와 동일하나, 모든 돔이 변경 후에 **동기적**으로 발생한다.

실행 순서

1. 리액트가 DOM을 업데이트
2. `useLayoutEffect`를 실행
3. 브라우저에 변경 사항을 반영
4. `useEffect`를 실행

동기적으로 발생하기 때문에 컴포넌트가 잠시 동안 일시 중지되는 것과 같은 일이 발생하게 된다.

따라서 **DOM은 계산됐지만 이것이 화면에 반영되기 전에 하고 싶은 작업**이 있을 때 사용하는 것이 다

### useDebugValue

리액트 애플리케이션을 개발하는 과정에서 사용되는데, 디버깅하고 싶은 정보를 이 훅에다 사용하면 리액트 개발자 도구에서 볼 수 있다.

```tsx
function useDate() {
  const date = new Date();
  useDebugValue(date, (date) => `현재 시간: ${date.toISOString()}`);

  return date;
}

function App() {
  const date = useDate();

  // 생략
}
```

개발자 도구에서 date가 변경되었을 때만 호출되어 포메팅된 값이 찍히는 걸 확인할 수 있게 된다.

### 훅의 규칙

- 최상위에서만 훅을 호출해야 한다. 반복문이나 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다.
- 훅을 호출할 수 있는 것을 리액트 함수형 컴포넌트, 사용자 정의 훅의 두가지 경우만 있다.

조건문, 루프내에서 사용할 수 있는 훅: https://react.dev/reference/react/use

리액트 훅은 파이버 객체의 링크드 리스트의 호출 순서에 따라 저장되기 때문에 순서를 보장받도록 코드가 짜여져야 한다. (조건문 내에 있으면 안됌)

## 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

### 사용자 정의 훅

서로 다른 컴포넌트 내부에서 같은 로직을 공유하고자 할 때 사용하는 것

```tsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);
  return isOnline;
}
```

복잡하고 반복되는 로직은 사용자 정의 훅으로 간단하게 만들어서 손쉽게 중복되는 로직을 관리할 수 있다.

### 고차 컴포넌트

고차 컴포넌트(HOC, Higher Order Component) 는 컴포넌트 자체의 로직을 재사용하기 위한 방법이다. 리액트에서 가장 유명한 고차 컴포넌트는 `React.memo` 이다.

**고차 함수 만들어보기**

리액트의 함수형 컴포넌트도 **결국 함수이기 떄문에 함수를 기반으로 고차 함수를 만드는 것을 먼저 이해**해야 한다.

```tsx
const setState = (function () {
  let currentIndex = index;

  return function (value) {
    global.states[currentInex] = value;
  };
})();
```

useState에서 반환된 두 번째 배열의 값으로 실행할 수 있는 함수를 반환하는데 이것도 고차 함수로 볼 수 있다.

```tsx
function add(a) {
  return function (b) {
    return a + b;
  };
}

const result = add(1); // result는 반환한 함수가 된다.
const result2 = result(2); // 반환한 함수에 1(a), 2(b)가 합쳐져 3이 반환된다.
```

a = 1이라는 정보가 담긴 클로저가 result에 포함됐고 result(2)가 호출하면서 클로저를 활용해 결과가 3이 반환된다.

**고차 함수를 활용한 리액트 고차 컴포넌트 만들어보기**

고차 함수의 특징에 따라 개발자가 만든 또 다른 함수를 반환할 수 있다는 점에서 고차 컴포넌트를 사용해 인증 정보에 따라 별도의 컴포넌트를 그려줄 수 있다.

```tsx
function withLoginComponent<T>(Component: ComponentType<T>) {
  return function (props: T & LoginProps) {
    const { loginRequired, ...restProps } = props;

    if (loginRequired) {
      return "로그인이 필요합니다.";
    }
  };

  return <Component {...(restProps as T)} />;
}

const Component = withLoginComponent((props: { value: string }) => {
  return <h3>{props.value}</h3>;
});

function App() {
  const isLogin = true;

  return <Component value="text" loginRequired={isLogin} />;
}
```

`withLoginComponent` 는 함수(함수형 컴포넌트)를 인수로 받아 컴포넌트를 반환하는 고차 컴포넌트이다. 고차 컴포넌트는 컴포넌트의 결과물에 영향을 미칠 수 있는 다른 공통된 작업을 처리할 수 있다.

사용자 정의 훅과 다르게 고차 컴포넌트는 `with`로 시작하는 이름을 사용해야 한다.

**주의할 점**

- 부수효과를 최소화해야 한다.
  반드시 컴포넌트의 props를 임의로 수정, 추가, 삭제하는 일은 없어야 한다.
- 여러 개의 고차 컴포넌트로 컴포넌트를 감쌀 경우 복잡성이 커진다.

### 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

**사용자 정의 훅이 필요한 경우**

useEffect, useState와 같이 리액트에서 제공되는 훅으로만 공통 로직을 격리할 수 있다면 사용자 정의 훅을 사용하는 것이 좋다. **사용자 정의 훅만으로는 렌더링에 영향을 미치지 못하기**에 반환되는 값으로 무엇을 할지는 개발자에게 달려있다.

따라서 컴포넌트 내부에 미치는 영향을 최소화해 개발자가 훅을 원하는 방향으로만 사용할 수 있다는 장점이 있다.

```tsx
// 사용자 정의 훅
function HookComponent() {
  const { loggedIn } = useLogin();

  useEffect(() => {
    if (!loggedIn) {
    }
  }, [loggedIn]);
}

// 고차 컴포넌트
const HOCComponent = withLoginComponent(() => {});
```

고차 컴포넌트의 경우 사용자 정의 훅에 비해 예측하기가 어렵다.

**고차 컴포넌트를 사용해야 하는 경우**

만약 공통 컴포넌트를 노출하는 것이 좋거나 에러 바운더리와 비슷하게 어떤 특정에러가 발생했을 때 에러 컴포넌트를 노출 시키고 싶을 경우엔 고차 컴포넌트를 사용하는 것이 좋다.

```tsx
// 사용자 정의 훅
function HookComponent() {
  const { loggedIn } = useLogin();

  if (!loggedIn) {
    return <LoginComponent />; // 공통 컴포넌트 노출
  }
  return "안녕하세요";
}

// 고차 컴포넌트
const HOCComponent = withLoginComponent(() => {
  return "안녕하세요";
});
```

고차 컴포넌트를 사용할 경우 loggedIn의 상태를 알 필요 없이 공통화된 렌더링 로직을 처리된 HOC를 사용하면 된다.
