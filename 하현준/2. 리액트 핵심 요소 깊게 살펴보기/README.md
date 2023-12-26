## 1. JSX란?

JSX는 자바스크립트 표준 코드가 아닌 페이스북이 임의로 만든 새로운 문법이다.
트랜스파일러를 거쳐야 자바스크립트 런타임이 이해할 수 있는 자바스크립트 코드로 변환이 된다.

**XML 스타일의 트리 구문을 작성하는데 많은 도움을 주는 새로운 문법**

### JSX의 정의

JSX는 `JSXElement`, `JSXAttributes`, `JSXChildren`, `JSXStrings` 4자리로 구성됌

**JSXElement**

HTML의 요소와 비슷한 역할을 하고, 아래 4가지 형태 중 하나여야 한다.

-   JSXOpeningElement: JSX를 시작하는 요소. ex) <JSXElement JSXAttributes(optional)>
-   JSXClosingElement: JSXOpeningElement가 종료됐음을 알리는 요소. ex) <JSXElement />
-   SXSelfClosingElement: 요소가 시작되고 종료를 스스로 하는 형태. ex) <JSXElement JSXAttributes(optional) />
-   JSXFragment: 아무런 요소가 없는 형태. </> 불가능, <></>는 가능. ex) <>JSXChildaren(optional)</>

**JSXAttributes**

JSXElement에 부여할 수 있는 속성을 의미한다.

-   JSXSpreadAttributes: 자바스크립트의 전개 연산자와 동일한 역할
-   JSXAttributeValue: 속성을 나타내는 키와 값으로 짝을 이뤄서 표현
    -   JSXAttributeName: 속성의 키 값
    -   JSXAttributeValue: 속성의 키에 할당할 수 있는 값
        -   큰따옴표로 구성된 문자열, 작은따옴표로 구성된 문자열, AssignementExpression, JSXElement
            ```tsx
            function valid1() {
                return (
                    <foo
                        attribute1=""
                        attribute2=""
                        attribute3={}
                        attribute4=<></>
                    ></foo>
                );
            }
            ```

**JSXChildren**

JSXElement의 자식 값을 나타냄

-   JSXChild: JSXChild가 0개 이상이 모여 JSXChildren을 구성한다.
    -   JSXText: {, <, >, }를 제외한 문자열
    -   JSXElement
    -   JSXChildExpression

**JSXStrings**

큰따옴표로 구성된 문자열, 작은 따옴표로 구성된 문자열, JSXText를 의미

\로 시작하는 이스케이프 문자 형태는 원래 이스케이프 처리를 해야 하지만 HTML에서는 그대로 사용이 가능

JSX에서도 마찬가지로 처리를 하지 않는다.

```tsx
<button>\</button>

let escape1 = '\' // error
```

## 가상 DOM과 리액트 파이버

### 가상 돔의 탄생 배경

브라우저가 웹페이지를 렌더링하는 과정은 매우 복잡하고 많은 비용이 듬(사용자의 인터랙션을 통해 다양한 정보를 노출하기 때문)

이러한 문제점을 해결하기 위해 실제 브라우저의 돔이 아닌 리액트가 관리하는 가상의 돔을 통해 돔계산을 브라우저가 아닌 메모리에서 계산하는 과정을 한번 거치게 되어 렌더링 과정을 최소화할 수 있고 브라우저와 개발자의 부담을 덜 수 있다.

그러나 무조건 빠른 것이 아니라 웬만한 애플리케이션을 만들 수 있을 정도로 충분히 빠르다는 것이다.

(https://twitter.com/dan_abramov/status/842329893044146176)

### 가상 DOM을 위한 아키텍처, 리액트 파이버

리액트 파이버란 가상 DOM과 렌더링 과정 최적화를 가능하게 해주는 것

파이버는 파이버 재조정자(fiber reconciler)가 관리하며, 재조정은 어떤 부분을 새롭게 렌더링해야 하는지 가상 돔과 실제 돔을 비교하는 알고리즘이다.

**리액트 파이버의 목표**

-   작업을 작은 단위로 분할하고 쪼갠 다음, 우선순위를 매긴다.
-   이러한 작업을 일시 중지하고 나중에 다시 시작할 수 있다.
-   이전에 했던 작업을 다시 재사용하거나 필요하지 않은 경우에는 폐기할 수 있다.

**파이버의 구현방식**

하나의 작업 단위를 처리하고 `finishedWork()`라는 작업으로 마무리한다. 이 작업을 커밋해 실제 브라우저 돔의 가시적인 변경 사항을 만들어 낸다.

-   **렌더 단계**에서는 리액트는 사용자에게 노출되지 않는 모든 비동기 작업을 수행한다. 파이버의 작업, 우선순위를 지정하거나 중지시키거나 버리는 등의 작업이 일어난다.
-   **커밋 단계**에서는 돔의 실제 변경사항을 반영하기 위한 작업, `commitWork()`가 실행된다. 커밋 단계는 동기식으로 동작되며 중단도 하지 못한다.

리액트와 다르게 **가급적이면 재사용**한다.

https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiber.js#L135

```tsx
function FiberNode(
    this: $FlowFixMe,
    tag: WorkTag,
    pendingProps: mixed,
    key: null | string,
    mode: TypeOfMode
) {
    // Instance
    this.tag = tag;
    this.key = key;
    this.elementType = null;
    this.type = null;
    this.stateNode = null;

    // Fiber
    this.return = null;
    this.child = null;
    this.sibling = null;
    this.index = 0;

    this.ref = null;
    this.refCleanup = null;

    this.pendingProps = pendingProps;
    this.memoizedProps = null;
    this.updateQueue = null;
    this.memoizedState = null;
    this.dependencies = null;

    this.mode = mode;

    // Effects
    this.flags = NoFlags;
    this.subtreeFlags = NoFlags;
    this.deletions = null;

    this.lanes = NoLanes;
    this.childLanes = NoLanes;

    this.alternate = null;
}
```

https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiber.js#L229

```jsx
function createFiber(
    tag: WorkTag,
    pendingProps: mixed,
    key: null | string,
    mode: TypeOfMode
): Fiber {
    // $FlowFixMe[invalid-constructor]: the shapes are exact here but Flow doesn't like constructors
    return new FiberNode(tag, pendingProps, key, mode);
}
```

https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiber.js#L648

```jsx
export function createFiberFromElement(
    element: ReactElement,
    mode: TypeOfMode,
    lanes: Lanes
): Fiber {
    let source = null;
    let owner = null;

    const type = element.type;
    const key = element.key;
    const pendingProps = element.props;
    const fiber = createFiberFromTypeAndProps(
        type,
        key,
        pendingProps,
        source,
        owner,
        mode,
        lanes
    );
    return fiber;
}

export function createFiberFromFragment(
    elements: ReactFragment,
    mode: TypeOfMode,
    lanes: Lanes,
    key: null | string
): Fiber {
    const fiber = createFiber(Fragment, elements, key, mode);
    fiber.lanes = lanes;
    return fiber;
}
```

파이버는 하나의 element에 하나가 생성되는 1:1 매핑 형태, 이때 매칭되는 정보를 가지고 있는 것이 `tag`

workTag를 받고 이는 https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactWorkTags.js 에서 확인이 가능하다.

파이버가 처리하는 작업들은 작은 단위로 나눠서 처리하거나, 애니메이션과 같이 우선순위가 높은 작업은 가능한한 빠르게 처리하거나, 낮은 작업을 연기시키는 등 좀 더 유연하게 처리가 된다.

리액트는 가상 DOM이 단순한 자바스크립트 객체로 관리되고 있고, UI를 문자열, 숫자, 배열과 같은 값으로 관리하는 **Value UI 라이브러리** 이다.

**리액트 파이버 트리**

파이버 트리는 더블 버퍼링 기술을 쓰는 트리이다.

> 더블 버퍼링이란?
> 현재 모습을 담은 파이버트리와 작업 중인 상태를 나타내는 workInProgress 트리를 쓰는데,
> workInProgress 트리의 작업을 현재 트리로 바꾸는 것을 말한다.

파이버의 작업 순서

1. beginInWork() 함수를 실행해 파이버 작업을 수행하는데, 더 이상 자식이 없는 파이버를 만날 때까지 트리형식으로 시작된다.
2. completeWork() 함수를 실행해 파이버 작업을 완료한다.
3. 형제가 있다면 형제로 넘어간다.
4. return으로 들어가 자신이 작업을 완료됐음을 알린다.

처음 반영시에는 초기 트리가 생성되지만 `setState` 를 통해 업데이트가 발생하면 workInProgress트리를 다시 빌드하고 새로 생성하지 않고 기존 파이버에서 업데이트된 props만을 통해 파이버 내부에서 처리한다.

→ ‘가급적 새로운 파이버를 생성하지 않는다’ 의 의미

## 클래스형 컴포넌트와 함수형 컴포넌트

### 클래스형 컴포넌트의 생명주기 메서드

생명주기 메서드가 실행되는 시점 3가지

-   마운트(mount)
-   업데이트(update)
-   언마운트(unmount)

**render**

컴포넌트가 UI를 렌더링하기 위해서 쓰인다. 마운트와 업데이트 시점에 일어남

**componentDidMount**

컴포넌트가 마운트되고 그다음으로 호출되는 메서드이다.

**componentDidUpdate**

컴포넌트가 업데이트가 일어난 이후 바로 실행된다.

**componentWillUnmount**

컴포넌트가 언마운트되거나 더 이상 사용되지 않기 직전에 호출된다.
메모리 누수나 불필요한 작동을 막기위한 클린업 함수를 호출히가 좋은 위치이다

**shouldComponentUpdate**

컴포넌트가 state나 props에 의해 다시 리랜더링이 되는 것을 막고 싶을 때 사용하면 되는 메서드이다.
state가 변화될때 리렌더링 되는 것은 자연스러운 일인데 이를 사용하는 것은 최적화 상황에서만 고려되어야 한다.

```jsx
shoudComponentUpdate(nextProps: Props, nextState: State) {
   return this.props.title !== nextProps,title || this.state.input !== nextState.input
}
```

**static** **getDerivedStateFromProps**

render를 호출하기 직전에 호출된다. 여기서 반환하는 객체는 해당 객체의 내용이 모두 state로 들어가 state에 영향을 미친다.

**getSnapShotBeforeUpdate**

DOM이 업데이트되기 직전에 호출된다. 돔이 렌더링되기 전에 **윈도우 크기를 조절**하거나 **스크롤 위치를 조정**하는 등의 작업을 처리하는데 유용하다.

```jsx
getSnapshotBeforeUpdate(prevProps: Props, prevState: State) {
	if (prevProps.list.length < this.props.list.length) {
		const list = this.listRef.current;
		return list.scrollHeight - list.scrollTop;
	}
	return null;
}

componentDidUpdate(prevProps: Props, prevState: State, snapshot: Snapshot) {
	// getSnapshotBeforeUpdate 로 넘겨받은 값을 처리할 수 있다.
	if (Snapshot !== null) {
		const list = this.listRef.current;
		list.scrollTop = list.scrollHeight - snapshot;
	}
}
```

**static** **getDerivedStateFromError**

에러상황에 발생하는 메서드이다. 훅으로 구현되지 않은 메서드이기에 이 메서드와 (**getSnapShotBeforeUpdate,componentDidCatch)**를 사용해야 한다면 클래스형 컴포넌트를 사용해야 한다.

하위 컴포넌트에서 에러가 발생했을 경우에 자식 리액트 컴포넌트를 렌더링할지 결정하는 용도로만 사용되어야 한다. 즉, 부수 효과(상태를 반환하는 것 외의 모든 작업)를 발생시켜서는 안된다.

**componentDidCatch**

마찬가지로 에러상황에 발생하는 메서드로 부수 효과를 처리할 수 있다. ErrorBoundary를 구현하기 위한 목적으로 자주 사용되고 ErrorBoundary를 여러개 선언해서 컴포넌트 별로 에러 처리를 다르게 적용할 수 있다.

```jsx
function App() {
	return (
		<ErrorBoundary name="parent">
			{* child에서 발생한 에러는 잡히지 않음 *}
			<ErrorBoundary name="child">
				<Child/>
			</ErrorBoundary>
		</ErrorBoundary>
	)
}
```

[React Lifecycle Methods diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

리액트 생명주기 다이어그램

클래스형 컴포넌트의 한계

-   데이터의 흐름을 추적하기 어렵다.
-   애플리케이션 내부 로직의 재사용이 어렵다.
-   기능이 많아질수록 컴포넌트의 크기가 커진다.
-   클래스는 함수에 비해 상대적으로 어렵다.
-   코드 크기를 최적화하기 어렵다. (사용하지 않은 메서드가 트리쉐이킹 되지 않고 번들에 포함됌)
-   핫 리로딩을 하는데 상대적으로 불리하다.

## 함수형 컴포넌트

### **함수형 컴포넌트 vs 클래스형 컴포넌트**

생명주기 메서드의 부재

`render`함수는 `React.Component`를 상속받아 구현하는 자바스크립트 클래스였기 때문에 함수형 컴포넌트에서는 사용이 불가능하다.

함수형 컴포넌트에서는 useEffect훅을 사용해서 **componentDidMount, componentDidUpdate, componentWillUnmount**를 비슷하게 구현할 수 있다.

즉, 비슷이지 똑같다는것이 아니기에 **생명주기를 위한 훅이 아니다.**

## 렌더링은 어떻게 일어나는가?

### 리액트의 렌더링이란?

리액트 애플리케이션 트리 안에 있는 모든 컴포넌트들이 현재 자신들이 가지고 있는 props와 state의 값을 기반으로 어떻게 UI를 구성하고 이를 바탕으로 어떤 DOM 결과를 브라우저에 제공할 것인지 계산하는 일련의 과정을 말한다.

### 리액트의 렌더링이 일어나는 이유

**리액트에서 렌더링이 일어나는 경우의 시나리오**

-   최초 렌더링: 처음 애플리케이션에 진입하면 렌더링해야할 결과물을 브라우저에 전달하기 위해 렌더링을 수행한다.
-   리렌더링: 최초 렌더링이 발생한 이후로 발생하는 모든 렌더링을 의미
    -   클래스형 컴포넌트
        -   setState가 실행되는 경우
        -   forceUpdate가 실행되는 경우
    -   함수형 컴포넌트
        -   useState()의 setter가 실행되는 경우
        -   useReducer()의 dispatch가 실행되는 경우
-   컴포넌트의 key props가 변경되는 경우:
    기본적으로 모든 컴포넌트에 사용할 수 있는 props이며 특히 배열에는 key를 붙여야 한다.

앞서 얘기했던 파이버 트리에서 current/worInProgress 트리사이의 변경을 구별하기위해 식별자로 key를 사용하기 때문이다.

-   props가 변경되는 경우
-   부모 컴포넌트가 렌더링 될 경우 → 부모 컴포넌트가 리렌더링 될경우 자식 컴포넌트도 **무조건** 리렌더링 됌

### 리엑트의 렌더링 프로세스

렌더링 프로세스가 시작되면 리액트는 컴포넌트의 루트에서부터 차근차근 아래쪽으로 내려가면서 업데이트가 필요하다고 지정돼 있는 모든 컴포넌트를 찾는다.

-   클래스형 컴포넌트 → render()함수를 실행
-   함수형 컴포넌트 → FunctionComponent() 자체를 실행

```jsx
function Hello() {
	return (
		<TestComponent a={35} b="yceffort">
			안녕하세요.
		</TestComponent>
	)
}

function Hello() {
	return React.createElement(
		TestComponent,
		{ a: 35, b: 'yceffort' },
		'안녕하세요',
	)
}

{type: TestComponent, props: { a: 35, b: 'yceffort', children: "안녕하세요" }}
```

렌더링프로세스는 다음과 같은 과정을 거쳐서 각 컴포넌트의 렌더링 결과물을 수집한 다음 리액트의 새로운 트리인 가상 DOM과 비교해 실제 DOM에 반영하기 위한 모든 변경사항을 수집힌다.

이러한 과정을 **재조정(Reconciliation)**이라고 한다. 재조정이 끝난 뒤에 하나의 동기 시퀀스로 DOM에 적용하여 결과물이 보이게 된다.

### 렌더와 커밋

**렌더 단계(Render Phase)**

컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 작업
`type`, `props`, `key` 중 하나라도 변경된 것이 있다면 변경이 필요한 컴포넌트로 체크한다.

**커밋 단계(Commit Phase)**

렌더 단계의 변경 사항을 실제 DOM에 적용해 사용자에게 보여주는 과정

위 두단계가 끝난 뒤에 모든 DOM 노드 및 인스턴스르 가르키도록 리액트 내부의 참조를 업데이트

그 다음 클래스형 컴포넌트는 `componentDidMount`, `componentDidUpdate`를 호출, 함수형 컴포넌트는 `useLayoutEffect`를 호출한다.

**리액트의 렌더링이 일어난다고 해서 무조건 DOM 업데이트가 일어나는 것은 아니다.
(리액트의 렌더링이 가시적인 변경이 일어나지 않아도 발생할 수 있다.)**

**동시성 렌더링**

동시성 렌더링이란 의도된 우선순위로 컴포넌트를 렌더링해 최적화할 수 있는 비동기 렌더링이다.

동시성 렌더링은 렌더링 중 렌더 단계가 비동기로 작동해 특정 렌더링의 우선순위를 낮추거나, 필요하다면 중단하거나 재시작하거나, 경우에 따라서는 포기할 수도 있다.

## 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션

최적화 기법에 사용되는 `useMemo`, `useCallback` 훅, 고차 컴포넌트 `memo`를 언제 사용하면 좋을지에 대한 얘기이다.

-   메모이제이션 최적화에 대한 의문점들
    언제 사용하는 것이 좋을까?
    렌더링이 자주 일어나는 컴포넌트?
    렌더링이 자주 일어나는 컴포넌트를 어떻게 알 수 있을까?
    코드를 꼼꼼히 읽어서 렌더링이 일어날 것 같은 영역에 모조리 추가할까?
    의존성 배열이 생략된 useEffect를 모든 컴포넌트에 추가해서 실제로 렌더링이 돌아가는지 확인해봐야 할까?
    무거운 연산의 기준은 무엇일까?
    실제로 코드를 작성할 때 함수의 실행 속도나 컴포넌트 렌더링 속도를 매번 계산하는 것이 좋을까?
    계산한다면 그 기준은 무엇일까?
    이렇게 최적화에 대해 매번 고민할 바엔 그냥 모든 컴포넌트를 메모이제이션 해버리는 게 낫지 않을까?
    함수 결과도 다 메모이제이션할까?
    렌더링 비용과 메모이제이션 비용중 어떤게 더 비싼걸까?

### 주장1. 섣부른 최적화는 독이다. 꼭 필요한 곳에만 메모이제이션을 추가하자

메모이제이션은 **값을 비교하고 렌더링 또는 재계산이 필요한지 확인하는 작업**, 이전에 **결과물을 저장해 두었다가 다시 꺼내와야 한다**는 두가지 비용이 든다. 이 비용이 리렌더링 비용보다 저렴한지는 상황에 따라 다르다.

그렇기에 **신중하게 접근해야 하며 섣부른 최적화는 항상 경계해야 한다. (’premature optimization’, ‘premature memoization’)**

[리액트 useMemo Docs](https://react.dev/reference/react/useMemo) 에서도 useMemo를 불필요하게 사용하지 않아도 될 방법에 대해 설명하고 있다.

### 주장2. 렌더링 과정의 비용은 비싸다. 모조리 메모이제이션해 버리자

두가지 주장에서 깔고 가는 전제

-   일부 컴포넌트에서는 메모이제이션을 하는 것이 성능에 도움이 된다.
-   섣부른 최적화인지 여부와는 관계없이, 해당 컴포넌트가 렌더링이 자주 일어나며 렌더링 사이에 비싼 연산이 포함돼 있고, 심지어 그 컴포넌트가 자식 컴포넌트 또한 많이 가지고 있다면 memo나 다른 메모이제이션 방법을 사용하는 것이 이점이 있을때가 있다.

**memo를 잘못 사용했을 때 내야하는 비용**

props에 대한 얕은 비교가 발생하면서 지불해야 하는 비용이다.
리액트는 기본적인 재조정 알고리즘으로 인해 이전 랜더링 결과를 저장해두고 있다.

따라서 props에 대한 얕은 비교만 비용으로 지불하면 된다.

**memo를 하지 않았을 때 발생할 수 있는 문제**

-   렌더링을 함으로써 발생하는 비용
-   컴포넌트 내부의 복잡한 로직의 재실행
-   모든 자식컴포넌트에서 반복해서 발생
-   리액트의 구트리와 신규 트리를 비교

매 실행마다 달라진 객체를 참조하는 값이 useEffect와 같은 의존성 배열에 쓰이는 예시이다.

```tsx
function useMath(number: number) {
    const [double, setDouble] = useState(0);
    const [triple, setTriple] = useState(0);

    useEffect(() => {
        setDouble(number * 2);
        setTriple(number * 3);
    }, [number]);

    return { double, triple };
}

function App() {
    const value = useMath(10);

    useEffect(() => {
        console.log(value.double, value.triple);
    }, [value]);

    // handleClick 함수 생략
    return (
        <>
            <h1>{counter}</h1>
            <button onClick={handleClick}></button>
        </>
    );
}
```

위 예시는 값이 value가 바뀌지 않았지만 계속해서 console.log가 출력된다.

handleClick으로 렌더링을 일으키면 number는 같은 인수를 넘겨주지만 console.log가 계속 출려되는데 그 이유는 **객채의 참조**가 달라지기 떄문이다.

useMemo로 반환값을 감싸어 참조의 투명성을 유지해준다. 이 주장은 결국 섣부른 최적화라 할지라도 누릴 수 있는 이점과 위험 비용이 큰 문제를 막아주기에 좋다고 주장한다.

```tsx
function useMath(number: number) {
    const [double, setDouble] = useState(0);
    const [triple, setTriple] = useState(0);

    useEffect(() => {
        setDouble(number * 2);
        setTriple(number * 3);
    }, [number]);

    return useMemo(() => {
        double, triple;
    }, [double, triple]);
}
```

**저자의 의견**

시간적 여유가 많다 → 주장 1의 의견으로 성능상 이점을 볼수 있는곳에만 적용

시간적 여유가 많지 않다.
→ 의심이 가는 곳에는 먼저 다 적용해보기
→ 일반적인 props의 경우 리렌더링 비용이 더 비싸기 때문에 다른 컴포넌트의 props로 들어가거나 활용될 여지가 있다면 `useMemo`, `useCallback`을 사용하는 것이 좋다
