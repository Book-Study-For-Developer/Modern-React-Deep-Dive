# 10장 리액트 17과 18의 변경 사항 살펴보기

## 리액트 17버전 살펴보기

16버전과 다르게 새롭게 추가된 기능이 없고 호환성이 깨지는 변경사항을 최소화했다는 점이 가장 큰 특징이다.

‘한 애플리케이션 내에 여러 버전의 리액트가 존재하는 시나리오’를 통해 점진적인 업그레이드를 제공해주었다.

### 이벤트 위임 방식의 변경

```tsx
export default function Button() {
    const buttonRef = useRef<HTMLButtonElement | null>(null);

    useEffect(() => {
        if (buttonRef.current) {
            buttonRef.current.onClick = function click() {
                alert("안녕하세요!");
            };
        }
    }, []);

    function 안녕하세요() {
        alert("안녕하세요!");
    }

    return (
        <>
            <button onClick={안녕하세요}>리액트 버튼</button>
            <button ref={buttonRef}>그냥 버튼</button>
        </>
    );
}
```

직접 onClick 이벤트 핸들러 추가 방식과 react에 onClick 추가하는 방식 두가지가 있다.

실제 웹에서는 ‘그냥 버튼’의 경우 이벤트 리스너에 이벤트가 등록이 된다.

반면에 리액트에 부착한 이벤트의 경우 루트에 이벤트를 부착하는 **이벤트 위임방식**으로 구현되어있다.

이벤트 위임이란?

-   캡처: 이벤트 핸들러가 트리 최상단 요소에서 부터 시작해서 실제 이벤트가 발생한 타깃 요소까지 내려가는 것을 의미한다.
-   타깃: 이벤트 헨들러가 타깃 노드에 도달하는 단계다. 이 단계에서 이벤트가 호출된다.
-   버블링: 이벤트가 발생한 요소에서 부터 시작해 최상위 요소까지 다시 올라간다.

리액트 16버전에서는 모두 `document`에서 수행되었지만 react 17에서는 리액트 컴포넌트 최상단 트리, 즉 루트요소로 바뀌었다.

document에서 이벤트가 처리된다면 `e.stopPropagation`으로 **이벤트 전파를 막더라도 이미 document에 올라가 있으므로 이벤트 전파를 막을 수 없다.**

각 컴포넌트 트리 수준으로 격리시켜 이벤트 버를링으로 인한 혼선을 방지했다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/253d1ac1-0c8d-4179-8d90-21ade38e0aea/caeb91b5-466b-417c-88c8-b936dba49aa2/Untitled.png)

### import React from ‘react’가 더 이상 필요 없다: 새로운 JSX transform

리액트 17부터는 바벨과 협력해 import 구문이 없어도 JSX를 변환할 수 있게 됐다. 이는 번**들링 크기도 줄일** 수 있고 **컴포넌트 작성을 더욱 간결**하게 해준다.

```tsx
const Component = (
	<div>
		<span>hello world</span>
	</div>
)

// 리액트 16에서 변환되는 코드
var Component = React.createElement(
	'div',
	null,
	React.createElement('span',null, 'hello world'),
)

// 리액트 17에서 변환되는 코드
'use strict'
var _jsxRuntime = require('react/jsx-runtime')

var Component (0, _jsxRuntime.jsx)('div', {
	children: (0, _jsxRuntime.jsx)('span', {
		children: 'hellow world',
	})
})
```

즉 16에서는 React를 가져오는 코드가 필요하므로 import 구문이 필요하지만 17에서는 내부에서 `jsx-runtime`으로 코드를 변환해주므로 코드가 간결해졌다.

### 그 밖의 주요 변경 사항

이벤트 폴링 제거

리액트 16에서 이벤트 폴링이라 불리는 기능이 있었는데 이벤트를 처리하기 위한 `SyntheticEvent`가 있었다. 이 이벤트는 브라우저의 기본 이벤트를 한 번 더 감싼 이벤트 객체이다.

매 이벤트마다 한번 래핑한 이벤트를 사용해야 해서 매번 메모리 할당 작업이 일어날 수 밖에 없었고 메모리 누수를 방지하기 위해 주기적으로 해제해야 하는 번거로움이 있다.

이때 이벤트 풀링이란 `**SyntheticEvent` 풀을 만들어서 이벤트가 발생할 때마다 가져오는 것을 말한다.\*\*

-   이벤트 핸들러가 이벤트를 발생시킨다.
-   합성 이벤트 풀에서 합성 이벤트 객체에 대한 참조를 가져온다.
-   이 이벤트 정보를 합성 이벤트 객체에 넣어준다.
-   유저가 지정한 이벤트 리스너가 실행된다.
-   이벤트 객체가 초기화되고 다시 이벤트 풀로 돌아간다.

```tsx
function handleChange(e: ChangeEvent<HTMLInputElement>) {
    // e.persist()
    setValue(() => {
        return e.target.value;
    });
}
```

풀에서 이벤트를 받아오고, 이벤트가 종료되자마자 다시 초기화하는 방식때문에 사용자 쪽에서 직관적이지 않았다. 그래서 위 코드는 에러를 발생시킨다. 위의 코드를 정상적으로 사용하기 위해서는 `e.persist()`를 작성해줘야 한다.

이후에 모던 브라우저의 이벤트 처리 성능이 향상되고 별도 메모리 공간에 이벤트 객체를 할당해야하는 등 도움이 안되는 점에 사라졌다.

**useEffect 클린업 함수의 비동기 실행**

리액트 16까지는 동기적으로 처리됐지만 이 때문에 클린업 함수가 완료되기 전까지 다른 작업을 방해해 불필요한 성능저해가 문제가 되었다.

17부터는 비동기적으로 실행되어, 클린업 함수는 커밋 단계가 완료될 때까지 지연되게 된다.

**컴포넌트 undefined 반환에 대한 일관적인 처리**

16, 17버전에서는 undefined를 반환하면 오류를 발생시킨다. 그러나 **16에서 forwardRef나 memo에서 undefined를 만날 경우에는 에러를 발생시키지 않는 문제**가 있었다.

17에서 부터는 에러를 내뱉도록 되었다. (18에서는 undefined를 반환해도 에러를 발생시키지 않는다.)

## 리액트 18 버전 살펴보기

### 새로 추가된 훅 살펴보기

**useId**

컴포넌트 별로 유니크한 값을 생성하는 새로운 훅이다. useId를 통해 클라이언트와 서버에서 불일치를 피하면서 컴포넌트 내부에 고유한 값을 생성하게 도와주어 hydrate오류에서 벗어나게 해주었다.

**useTransition**

UI 변경을 가로막지 않고 상태를 업데이트할 수 있는 훅이다. 상태 업데이트를 긴급하지 않은 것으로 간주해 무거운 렌더링 작업을 조금 미룰 수 있어 좋은 사용자 경험을 제공할 수 있다.

https://react-ko.dev/reference/react/useTransition

**useDeferredValue**

리액트 컴포넌트 트리에서 리렌더링이 급하지 않은 부분을 지연할 수 있게 도와주는 훅이다. 디바운스와 비슷하지만 디바운스 대비 장점이 있다.

useTransition하고 사용법에 차이만 있을 뿐 동일한 역할을 하는 것이므로 상황에 맞는 방법을 선택하면 된다.

[useDeferredValue – React](https://react-ko.dev/reference/react/useDeferredValue)

**useSyncExternalStore**

테어링(tearing)현상을 방지하기 위해 나타난 훅이다. 렌더링을 일시 중지하거나 뒤로 미루는 등의 최적화가 가능해지면서 동시성 이슈가 발생할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/253d1ac1-0c8d-4179-8d90-21ade38e0aea/cb6cb554-9cf6-467e-a18a-4e44cb09af7f/Untitled.png)

1. 첫 번째 컴포넌트에서는 외부 데이터 스토어의 값이 파란색이었으므로 파란색으로 렌더링했다.
2. 그리고 나머지 컴포넌트들도 파란색으로 렌더링을 준비하고 있었다.
3. 그러다 갑자기 외부 데이터 스토어의 값이 빨간색으로 변경됐다.
4. 나머지 컴포넌트들은 렌더링 도중에 바뀐 색을 확인해 빨간색으로 렌더링했다.
5. 결과적으로 같은 데이터 소스를 바라보고 있음에도 컴포넌트의 색상이 달라지는 테어링 현상이 발생했다.

과거 리액트와 달리 랜더링이 중지됐다가 다시 실행하는 **‘양보’** 가 가능해 이러한 문제가 발생할 가능성이 존재하는 것이다.

```tsx
function subscribe(callback: (this: Window, ev: UIEvent) => void) {
    window.addEventListenr("resize", callback);
    return () => {
        window.removeEventListener("resize", callback);
    };
}

export default function App() {
    const windowSize = useSyncExternalStore(
        subscribe,
        () => window.innerWidth,
        () => 0
    );
    return <>{windowSize}</>;
}
```

윈도우의 innerWidth를 확인하는 코드인데 리액틔 외부에 있는 데이터 값이므로 이를 리렌더링으로 이어지게 하기 위해서는 `useSyncExternalStore` 가 필요하다.

```tsx
function useWindowWidth() {
    const [windowWidth, setWindowWidth] = useState(0);
    useEffect(() => {
        function handleResize() {
            setWindowWidth(window.innerWIdth);
        }

        window.addEventListenr("resize", callback);
        return () => {
            window.removeEventListener("resize", callback);
        };
    }, []);

    return windowWidth;
}
```

물론 이렇게 훅으로 만들 수 있지만 차이점이 존재한다.

```tsx
const PostsTab = memo(function PostsTab() {
	const width1 = useWindowWidthWithSyncExternalStore()
	const width2 = useWindowWidth()

	const items = Array.from({ length: 1500 }).map((_, i) => {
		<SlowPost key={i} index={i} />
	})

	return (
		<>
			<div>useSyncExternalStore {width1}px<div>
			<div>useEffect + useState (width2}px</div>
			<ul className="items">{items}</ul>
		</>
	)
})
```

width1은 정확한 width를 가져오지만, width2는 초기값인 0을 가져오는 차이점이 나타난다.

**useInsertionEffect**

useInsertionEffect는 CSS-in-js라이브러리를 위한 훅이다.

useEffect와 구조는 동일하되 실행되는 순서는 useInsertionEffect ⇒ DOM 변경작업 완료 ⇒ useLayoutEffect ⇒ useEffect 순서로 실행되게 된다.

**New Root API**

`CRA`를 통한 리액트 프로젝트를 생성한 기존 구조에서 `index.html`를 살펴보면, `<div id="root"></div>`라는 부분이 있었는데, 해당 엘리먼트가 바로 `Root`이다.

블로그 [강동희님의 게시물](https://tech.osci.kr/2022/05/03/react-18v/)을 참고하면 아래와 같이 설명하기도 하는데,

```jsx
// 18v 이전의 루트
ReactDOM.render(, document.getElementById(‘root’));
// 18v 이후의 루트 API
import * as ReactDOMClient from ‘react-dom/client’;
const container = document.getElementById(‘app’);
const root = ReactDOMClient.createRoot(container);
root.render();
```

구조를 보면 알겠지만 `ReactDOMClient`를 임포트 한 후 기존 `app`을 `Root`에 적용해 `render()`하는 방식이다. 즉, 기존 과정에서 `render`가 이루어지려면 `Root`를 거쳐간 후 진행이 되어야만 했는데, 이는 `Virtual DOM`을 사용하는 `React`특성상 반드시 필요한 작업이 아닐 뿐더러, 불필요한 리소스 낭비가 발생하기 때문이다.

그러므로, 새로 개발된 `ReactDOMClient`를 사용하여 `render()`가 호출되어야 할 시기를 정할 수 있다.

### 자동 배치(Automatic Batching)

리액트가 여러 상태 업데이트를 하나의 리렌더링으로 묶어서 성능을 향상시키는 방법을 의미한다.

```tsx
const [count, setCount] = useState(0);
function increase() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // count: 1
}

// prev 사용
function another() {
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    // count: 3
}

import { flushSync } from "react-dom";
// ReactDOM.flushSync() 사용
function flush() {
    flushSync(() => {
        setCount(count + 1);
    });
    flushSync(() => {
        setCount(count + 1);
    });
    flushSync(() => {
        setCount(count + 1);
    });
    // count: 3
}
```

### 더욱 엄격해진 엄격 모드

**리액트의 엄격 모드**

리액트 애플리케이션에서 발생할 수도 있는 잠재적인 버그를 찾는 데 도움이 되는 컴포넌트다.

**더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고**

리액트 클래스형 컴포넌트에서 사용되는 메서드 중 일부인 componentWillMount, componentWillReceiveProps, componentWillUpdate는 더이상 사용할 수 없고, 엄격 모드에서는 에러를 낸다

**문자열 ref 금지**

문자열 ref를 통해 DOM에 접근하게 되면 에러를 내뱉는다.

```tsx
class UnsafeClassComponent extends Component {
    componentDidMount() {
        console.log(this.refs.myInput);
    }

    render() {
        return (
            <div>
                <input type="text" ref="myInput" />
            </div>
        );
    }
}
```

-   문자열로 값을 주는 것은 여러 컴포넌트에 걸쳐 사용될 수 있으므로 충돌의 여지가 있다.
-   단순히 문자열로만 존재하기 때문에 실제로 어떤 ref에서 참조되고 있는지 파악하기 어렵다.
-   리액트가 계속해서 현재 렌더링되고 있는 컴포넌트의 ref의 값을 추적해야 하기 때문에 성능 이슈가 있다.

**findDOMNode를 사용하면 에러를 내뱉는다.**

https://react.dev/reference/react-dom/findDOMNode

공식 문서에서는 ref를 통해 하는 것을 권장한다.

예상치 못한 부작용 검사

리액트 엄격 모드 내부에서는 의도적으로 이중으로 호출한다.

-   클래스형 컴포넌트의 constructor, render, shouldComponentUpdate, getDerivedStateFromProps,
-   클래스형 컴포넌트의 setState의 첫 번째 인수
-   함수형 컴포넌트의 body
-   useState, useMemo, useReducer에 전달되는 함수

항상 순수한 결과물을 내고 있는지 개발자에게 인지시커 주기 위해서 두번 실행되는 것이다.

또, useEffect가 두번 작동하는데 이는 마운트 되고 언마운트되고 다시 마운트 시킨다. 이는 두번의 호출에도 자유로운 컴포넌트를 추구하는 컴포넌트를 만들어야 하기 때문이다.

### Suspense 기능 강화

리액트 16.6에 실험적으로 추가된 기능으로 컴포넌트를 동적으로 가져올 수 있게 도와주는 기능이다. 이때 문제점이 있었는데 Suspense로 감싼 컴포넌트의 useEffect가 실행되는 문제가 존재했다.

그래서 서버 사이드 렌더링 구조에서 Suspense를 활용하려면 useMount와 같은 훅을 통해 클라이언트에서만 작동하도록 처리해야 했다.

```tsx
function useMounted() {
    const [mounted, setMounted] = useState(false);

    useEffect(() => {
        setMounted(true);
    }, []);

    return mounted;
}

export default function CustomSuspense(props: ComponentProps<typeof Suspense>) {
    const isMounted = useMounted();

    if (isMounted) {
        return <Suspense {...props} />;
    }
    return <>{props.fallback}</>;
}
```

18 버전에서는 Suspense가 정식으로 지원되어 다음과 같은 내용들이 변경되었다.

-   effect 문제가 해결되었다.
-   Suspense에 의해 노출이 된다면 useLayoutEffect의 effect(componentDidMount)가, 가려진다면 useLayoutEffect의 cleanIp(componentWillUnmount)가 정상적으로 실행된다.
-   서버사이드 렌더링에서도 Suspense를 사용할 수 있게된다.
-   중첩된 Suspense의 fallback이 있다면 자동으로 스로틀되어 최대한 자연스럽게 보여주기 위해 노력한다.
