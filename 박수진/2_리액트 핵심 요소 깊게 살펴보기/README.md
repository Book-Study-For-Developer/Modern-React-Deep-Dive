# 2\_리액트 핵심 요소 깊게 살펴보기

목차

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [2.1 JSX 란?](#21-jsx-란)
  - [JSX의 정의](#jsx의-정의)
    - [JSXElement](#jsxelement)
  - [JSXAttributes](#jsxattributes)
  - [JSXChildren](#jsxchildren)
    - [JSXStrings](#jsxstrings)
  - [JSX 예시](#jsx-예시)
  - [JSX는 어떻게 자바스크립트에서 변환될까?](#jsx는-어떻게-자바스크립트에서-변환될까)
  - [정리](#정리)
- [2.2 가상 DOM과 리액트 파이버](#22-가상-dom과-리액트-파이버)
  - [DOM과 브라우저 렌더링 과정](#dom과-브라우저-렌더링-과정)
  - [가상 DOM의 탄생 배경](#가상-dom의-탄생-배경)
  - [가상 DOM 을 위한 아키텍처, 리액트 파이버](#가상-dom-을-위한-아키텍처-리액트-파이버)
  - [파이버와 가상 DOM](#파이버와-가상-dom)
  - [정리](#정리-1)
- [2.3 클래스형 컴포넌트와 함수형 컴포넌트](#23-클래스형-컴포넌트와-함수형-컴포넌트)
  - [클래스형 컴포넌트의 생명주기###](#클래스형-컴포넌트의-생명주기)
  - [클래스형 컴포넌트의 한계](#클래스형-컴포넌트의-한계)
  - [함수형 컴포넌트](#함수형-컴포넌트)
  - [정리](#정리-2)
- [2.4 렌더링은 어떻게 일어나는가?](#24-렌더링은-어떻게-일어나는가)
  - [리액트 랜더링이 일어나는 이유](#리액트-랜더링이-일어나는-이유)
  - [리액트 랜더링 프로세스](#리액트-랜더링-프로세스)
    - [랜더](#랜더)
    - [커밋](#커밋)
  - [정리](#정리-3)
- [2.5 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션](#25-컴포넌트와-함수의-무거운-연산을-기억해-두는-메모이제이션)

<!-- /code_chunk_output -->

## 2.1 JSX 란?

- JSX는 자바스크립트 표준 코드가 아닌 페이스북(메타)이 임의로 만든 새로운 문법
  - 하지만 리액트에 종속적이지않은 독자적 문법이다.
- 자바스크립트에서 표현하기 힘들었던, XML 스타일의 트리구문을 작성을 간편하게 한 문법
  - HTML, XML 외에도 다른 구문으로 확장될 수 있게 고려됨
- 자바스크립트 런타임이 이해할 수 있는 자바스크립트 코드로 변환되는 과정이 꼭 필요

### JSX의 정의

- JSX 는 4가지 컴포넌트 JSXElement, JSXAttributes, JSXChildren, JSXString 로 구성된다

#### JSXElement

JSXElement 의 종류

1. JSXOpeningElement : `<JSXElement JSXAttributes(optional)>`
   - 아래의 JSXClosingElement 와 반드시 쌍을 이뤄서 구성되어야 한다.
2. JSXClosingElement : `<JSXElement />`
3. JSXSelfClosingElement: `<JSXElement JSXAttributes(optional) />`
4. JSXFragment: `<>JSXChildren(optional)</>`
   - JSXSelfClosingElement 형태로는 사용이 불가능 하다.

JSXElementName

1. JSXIdentifier : 자바스크립트 식별자와 같은 규칙, 숫자, $,\_ 가 아닌 특수문자로 시작할 수 없는 단어
2. JSXNamespacedName
   - `JSXIdentifier : JSXIdentifier `
   - 두개 이상의 : 로 잇는 것은 안됨
3. JSXMemberExpression
   - `JSXIdentifier.JSXIdentifier
   - 위의 JSXNamespacedName 과 다르게 여러개의 . 으로 연결할 수 있다.

### JSXAttributes

JSXElement에 부여할 수 있는 속성

1. JSXSpreadAttributes
   - `{...AssignmentExpression}`
     - 자바스크립트의 전개 연산자와 동일한 역할
     - 객체 뿐 아니라 자바스크립트에서 AssignmentExpression 로 객체를 나타낼 수 있는 표현식
     - example : `<YourComponent {...(props.theme ? { focus: true } : { focus: false })} />`
2. JSXAttribute
   - 속성의 키와 값으로 짝을 이룸
   - 가능한 값
     - 큰따옴표, 작은따옴표로 구성된 문자열 : "string"
     - { AssignmentExpression }: 자바스크립트의 값 할당 표현식 ( 변수에 값을 넣을 수 있는 표현식)

### JSXChildren

JSXElement의 자식 값을 나타냄. JSX로 부모와 자식 관계를 나타낼 수 있다. JSXChild가 0개 이상이어야 함

- JSXChild
  - JSXText : {, <, >, } 를 제외한 문자열
  - JSXElement
  - JSXFragment
  - { JSXChildExpression(optional) } : 자바스크립트의 값 할당 표현식

#### JSXStrings

- HTML 에서 사용가능한 문자열은 모두 JSXStrings 에서도 가능하다.
- 자바스크립트와의 차이점은 \ 로 시작하는 이스케이프 문자 형태소를 HTML 에서는 이스케이프 없이 사용할 수있다.

### JSX 예시

```
// 전개 연산자로 옵션 넣기
const ComponentA = <A {...{required: true}}>

// 속성만 넣어도 가능
const ComponentB = <A required/>

// 리액트에서는 유효하지 않거나 잘 사용되지 않아도 JSX에서는 유효한 문법
function ComponentC( ){
	return <C.A></C.B>
}

function ComponentD( ){
	return <A:B.C></A:B.C>
}

function ComponentE( ){
	return <$></$>
}
```

### JSX는 어떻게 자바스크립트에서 변환될까?

babel 의 @babel/plugin-transform-react-jsx 플러그인 은 JSX 구문을 자바스크립트가 이해할 수 있는 형태로 변환해 준다.

```
const ComponentC = (
  <div>
    <span>time</span>
    <span>date</span>
  </div>
  )

// 기본 트랜스파일

const ComponentC = /*#__PURE__*/React.createElement("div", null, /*#__PURE__*/React.createElement("span", null, "time"), /*#__PURE__*/React.createElement("span", null, "date"));


// 자동 런타임 트랜스파일

import { jsx as _jsx } from "react/jsx-runtime";
import { jsxs as _jsxs } from "react/jsx-runtime";
const ComponentC = /*#__PURE__*/_jsxs("div", {
  children: [/*#__PURE__*/_jsx("span", {
    children: "time"
  }), /*#__PURE__*/_jsx("span", {
    children: "date"
  })]
});
```

[babel 트랜스파일 테스트](https://babeljs.io/repl)

- 위의 결과를 활용해서 리팩토링에 활용이 가능하다

```

function TextOrHeading ({isHeading}){
return createElement(isHeading? 'h1': 'span'),
{className: 'text'},
children
}

// 👀 위의 케이스의 경우 jsx에서 dynamic 하게 태그를 바꿔주는 방식이 더 직관적이지 않을까?

function TextOrHeading ({tag,children}:{tag:'h1'|'span'}){
return (<tag className='text'>{children}</tag>)
```

JSX 를 js 로 반환한 값을 보면, 결국 React.createElement로 반환되고 첫번째 인수를 JSXElement 로 받기때문에, JSXElement 만 다르고, JSXAttributes, JSXChildren 이 동일한 경우 바로 createElement 를 활용해서 jsx 를 생성할 수 있다.

### 정리

- 리액트에서만 JSX 가 쓰이는 것이 아니다 . Preact, SolidJS, NanoJSX 등 다양한 라이브러리도 JSX 를 채택하고 있다. 따라서 JSX 의 JSXNamespacedName , JSXMemberExpression과 같이 리액트에서 안쓰이는 JSX 의 속성도 인식해 두자.
- JSX가 HTML 과 자바스크립트의 문법이 뒤섞여 가독성이 안좋아진다는 의견도 있다.
- 리액트 내부에서 JSX 가 결국 자바스크립트 코드 createElement 로 번역된다는 것을 알면 더 효율적인 코드를 짤 수있을 거다!

<참고자료>
https://facebook.github.io/jsx/
https://react.dev/learn/writing-markup-with-jsx
https://react.dev/learn/javascript-in-jsx-with-curly-braces

## 2.2 가상 DOM과 리액트 파이버

### DOM과 브라우저 렌더링 과정

![rendering-process](image.png)

DOM 이란 ?

- DOM( Document Object Model ) 이란, 웹페이지의 콘텐츠와 구조를 어떻게 보여줄지에 대한 정보를 담은 웹페이지에 대한 인터페이스이다. 트리구조로 되어있다.
- 브라우저 랜더링의 과정은 다음과 같다.
  - 1. 브라우저에 사용자가 요청한 URL 을 방문해서 HTML을 다운로드 한다.
  - 2. HTML 을 파싱해서 DOM 노드로 구성된 트리(DOM)을 만드는데, 파싱 시에 외부 리소스(stylesheet, script, image)링크를 만나면 바로 네트워크 요청으로 가져오게 된다.
    - 특정 리소스는 다운로드될때 까지 html 파싱을 멈추게 된다.
    - html의 끝까지 파싱이 끝나는 시점에, CSS노드로 구성된 트리(CSSOM)도 생성된다.
  - 3. 생성된 DOM 을 순회하면서,페이지를 랜더링하는 데 필요한 눈에 노드만 (display: none 같은 요소 제외) 스타일정보를 노드에 적용하며 랜더 트리를 형성한다.
  - 4. 완성된 렌더트리는 레이아웃 과정을 거치면서, 각 노드가 브라우저 화면의 어느 위치와 크기로 나타나야할지 계산 된다.
  - 5. 페인팅을 통해 최종 렌더링 트리를 가져와 화면에 픽셀을 렌더링합니다.

<참고>
[critical rendering path-mdn](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path)
[critical rendering path-web.dev](https://web.dev/articles/critical-rendering-path/render-tree-construction?hl=ko)
[랜더러 프로세스 살펴보기](https://developer.chrome.com/blog/inside-browser-part3?hl=ko)

### 가상 DOM의 탄생 배경

- 싱글 페이지 어플리케이션(SPA)에서는 하나의 페이지에서 모든 작업이 일어남으로 추가 랜더링 작업이 필요한 경우가 많다. 처음 랜더링이 되고 나서, 다양한 사용자의 인터렉션에 따라 DOM 의 변경으로 레이아웃과 리페인팅이 발생하면서 많은 비용이 든다.
- SPA의 라우팅이 변경되는 경우, 사이드바나 헤더 같은 요소를 제외하고 대부분의 요소를 삭제하고, 다시 삽입, 계산해야하는 작업이 들어간다.

-> 여러번 발생할 렌더링 과정을 최소화 하고, 브라우저와 개발자의 부담을 덜어주기 위해 리액트에는 가상DOM 방식이 사용된다.

가상DOM

- 리액트에서 메모리에서 가상DOM을 이용해 계산을 한 후, DOM 변경에 대한 준비가 완료되었을때, 브라우저의 실제 DOM 에 반영됨
- 랜더링 방식에 있어서 여러번 랜더링하거나, DOM을 일일이 직접 조작해야하는 수고 줄이면서도, 어플리케이션을 개발하는 데 문제가 없이 빠르다는 것이지, 브라우저의 DOM 조작보다 가상DOM이 빠르다는 것은 오해이다.

<참고 자료>
[리액트 가상DOM](https://youtu.be/gc-kXt0tjTM)

**virtual dom에 관한 오해**
[virtual dom has tradeoff](https://medium.com/@dan_abramov/youre-missing-the-point-of-react-a20e34a51e1a)
[virtual-dom-is-pure-overhead](https://svelte.dev/blog/virtual-dom-is-pure-overhead)

<읽어보기>
https://vercel.com/blog/how-react-18-improves-application-performance

### 가상 DOM 을 위한 아키텍처, 리액트 파이버

**리액트 파이버의 목표**
리액트 웹 어플리케이션에서 발생하는 애니메이션, 레이아웃, 사용자 인터렉션의 반응성 문제를 해결하는 것

- 리액트 16 이전에는 스택 조정자를 사용했는데, 랜더링에 필요한 작업들이 동기적으로 실행됨에 따라 인터랙션에 반응성문제가 발생했다. ( 책의 예시: input 창에 입력했을 때, 다른 랜더링 작업들로 인해 늦게 입력이 됨)

**리액트 파이버의 작업**
파이버는 하나의 작업 단위로 구성되어있으며 스택조정자와 다르게 비동기적으로 수행한다. 다음과 같은 작업을 한다.

- 작업을 작은 단위로 분할하고 우선순위를 매길 수 있다.
- 작업을 멈추고, 재시작할 수 있다.
- 작업을 재사용하거나 버릴 수 있다.

**파이버의 구현**
[react-FiberNode](https://github.com/facebook/react/blob/c5b9375767e2c4102d7e5559d383523736f1c902/packages/react-reconciler/src/ReactFiber.js#L135)
위의 코드에서 FiberNode 를 살펴보자

```js
function FiberNode(
  this: $FlowFixMe,
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  /*하나의 element 당 fiber 하나가 생성되는 1:1 관계
  종류: 리액트의 컴포넌트, html 의 DOM, ContextProvider 등의 리액트 앱을 구성하는 요소들의 값들을 가진다. */
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null; // 파이버 자체에 대한 참조 정보를 가지며,이 참조를 바탕으로 리액트는 파이버 관련 상태에 접근한다.

  // Fiber
  /* 파이버가 트리형식을 구성하기 위해 필요한 정보들이 정의됨 리액트 컴포넌트 트리와 다른 것은 children이 없고 하나의 child만 존재한다
  */
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0; // 형제들 사이에서 자신의 위치가 몇 번째인지 숫자로 표현

  this.ref = null;
  this.refCleanup = null;

  this.pendingProps = pendingProps;// 아직 작업처리되지 않은 props
  this.memoizedProps = null;// 랜더리이 완료된 이루 pendingProps 를 memoizedProps로 저장
  this.updateQueue = null;// 상태 업데이트, 콜백함수, DOM 업데이트 등 필요한 작업을 담아두는 큐
  this.memoizedState = null;// 함수형 컴포넌트의 훅 목록이 저장됨
  this.dependencies = null;

  this.mode = mode;

  // Effects
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;

  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  this.alternate = null; // 리액트의 파이버 트리가 두개인데, 반대편 파이버 트리를 가리킴
  //... 생략

```

- 파이버는 단순한 자바스크립트 객체이다.
  - 리액트 핵심 원칙 : UI 를 문자열, 숫자, 배열과 같은 값으로 관리한다.
- 파이버의 생성
  - 파이버를 생성하는 다양한 함수가 있다.
  - 컴포넌트가 최초 마운트 되는 시점에 생성되어 이후에 가급적 재사용된다.
- 파이버의 실행
  - 실행 시점 : state 가 변경 ,생명주기 메서드 실행, DOM의 변경이 필요한 시점
  - 바로 처리하기도 하며, 스케줄링 하기도한다. ( 애니메이션같은 우선순위가 높은 작업은 빨리 처리하는 식)

**리액트 파이버 트리**

- 리액트 내부에 두개의 파이버 트리가 존재한다. 현재모습을 담은 current트리와 작업중인 상태를 나타내는 workInProgress 트리 두개다.
- 리액트에서 업데이트 해야 할 사항이 발생되면, 새로운 workInProgress 트리를 생성한다. current 트리를 기준으로 workInProgress 트리가 빌드된다.
  - setState와 같은 업데이트 작업은 시도때도 없이 일어나는데, 이때 기존에 있는 객체를 재활용해서 내부 속성값만 초기화하는 식으로 트리를 업데이트한다.
  - 이 기존 트리를 순회해서 새로운 트리를 만드는 작업이 예전에 동기식으로 처리했다는 작업인데, 이제는 파이버단위로 비동기적으로 또 우선순위를 정해서 처리가 된다.
- 리액트 파이버의 작업이 끝나면, 리액트는 current 트리의 포인터를 workInProgress 트리로 가르키는 더블 버퍼링 기술을 사용해서 변경한다.
  - 더블 버퍼링: 컴퓨터 그래픽 분야 용어로 다음에 그려야할 그림을 미리 그린 후, 현재상태를 새로운 그림으로 바꾸는 기법이다.

**파이버 작업순서**

1. beginWork() 함수를 실행해서 파이버 작업 수행 더 이상 자식이 없는 파이버를 만날때까지, 트리구조를 따라 beginWork()실행
   - 상태flag가 기록됨.
2. 자식이 없다면 completeWork() 실행
   - 트리가 빌드됨
3. 형제가 있다면 형제로 넘어가서 반복
4. 작업이 끝나면 return을 통해 작업완료를 알리고, 루트노드가 완성되면 최종적으로 commitWork() 가 수행되며, 변경사항을 비교해 업데이트가 필요한 사항이 DOM에 반영됨

### 파이버와 가상 DOM

- [그림으로 살펴보는 fiber 동작](https://www.youtube.com/watch?v=ZCuYPiUIONs)
- https://github.com/acdlite/react-fiber-architecture
  [React Fiber Reconciliation: How it Works (Part 1)-Tejas Kumar](https://youtu.be/rKk4XJYzSQA?si=Am37TNla7Dgu5fZ8)
- [React Fiber Reconciliation: How it Works (Part 2)-Tejas Kumar](https://youtu.be/Zan16X8VvGM?si=n_XYqHjdrm-JM6UF) : 실제로 코드상에서 fiber 메소드 실행순서를 보여주는 영상
- [React reconciliation: how it works and why should we care
  ](https://youtu.be/724nBX6jGRQ?si=VtRKWJd7Zmt1GaAf)
- [React 파이버 아키텍처 분석](https://d2.naver.com/helloworld/2690975)

### 정리

- 가상DOM 과 리액트의 핵심은 화면에 표시되는 UI 를 자바스크립트 값(파이버)으로 관리하고, DOM의 변경사항을 리액트의 내부의 파이버 재조정자가 알고리즘을 통해 관리함으로, 개발자가 쉽게 웹어플리케이션을 유지보수하고 관리하게 돕는다.
  <참고자료>

[어떻게 브라우저가 동작하는가](https://developer.mozilla.org/en-US/docs/Web/Performance/How_browsers_work)

## 2.3 클래스형 컴포넌트와 함수형 컴포넌트

- 함수형 컴포넌트는 리액트 0.14 버전부터 도입. 이때는 생명주기나 상태가 필요없을때만 사용되었다.
- 16.8 이후 훅이 소개되어 상태와 생명주기 작업도 함수형 컴포넌트에서도 사용할 수 있게 되니 보일러플레이트가 간단한 함수형 컴포넌트가 더 많이 쓰이게 되었다.

**클래스형 컴포넌트 살펴보기**

[예제코드2.10]

```tsx
import React from "react";

// props 타입을 선언한다.
interface SampleProps {
  required?: boolean;
  text: string;
}

// state 타입을 선언한다.
interface SampleState {
  count: number;
  isLimited?: boolean;
}

/* extend 로 만들고 싶은 컴포넌트를 extend 해야한다. React.Component, React.PureComponent 클래스를 사용가능하다.
 */

// Component에 제네릭으로 props, state를 순서대로 넣어준다.
class SampleComponent extends React.Component<SampleProps, SampleState> {
  /* 
  constructor(): 컴포넌트가 초기화 되는 시점에 호출됨
  - super(): 상위 컴포넌트의 생성자 함수를 먼저 호출해서 상위 컴포넌트에 접근할 수 있게 한다.
  - props: 컴포넌트에 특정속성을 전달하는 용도로 쓰인다.  
  - state: 클래스형 컴포넌트 내부에서 관리하는 값, 객체여야하고 이값에 변화가 있으면 리랜더링이 발생된다.
    - ES2022에 추가된 클래스필드로, contructor 에서 초기화 하지 않고, 바로 클래스 내부 필드에 선언할 수 있게 되었다.
  - 메서드: 랜더링 함수 내부에서 사용되는 함수로, DOM 에서 발생되는 이벤트와 함께 사용됨
  */

  // constructor에서 props를 넘겨주고, state의 기본값을 설정한다.
  private constructor(props: SampleProps) {
    super(props);
    this.state = {
      count: 0,
      isLimited: false,
    };
    this.handleClickDecrease = this.handleClickDecrease.bing(this);
  }

  /*메서드 방식1️⃣ : 화살표 함수를 사용하면 실행시점이 아닌 작성시점의 this가 상위스코프가 되어 state 에 바로 접근 가능 */
  private handleClickIncrease = () => {
    const newValue = this.state.count + 1;
    this.setState({ count: newValue, isLimited: newValue >= 10 });
  };
  /* 메서드 방식2️⃣: 일반함수 호출시, this가 전역객체 (strict mode: undefined)가 되므로 contructor 에서 바인딩해준다. */
  private handleClickDecrease() {
    const newValue = this.state.count - 1;
    this.setState({ count: newValue, isLimited: newValue <= -10 });
  }

  // render에서 이 컴포넌트가 렌더링할 내용을 정의한다.
  public render() {
    // props와 state 값을 this, 즉 해당 클래스에서 꺼낸다.
    const {
      props: { required, text },
      state: { count, isLimited },
    } = this;

    return (
      <h2>
        Sample Component
        <div>{required ? "필수" : "필수아님"}</div>
        <div>문자: {text}</div>
        <div>count: {count}</div>
        <button onClick={this.handleClickIncrease} disabled={isLimited}>
          증가
        </button>
        <button onClick={this.handleClickDecrease} disabled={isLimited}>
          감소
        </button>
        {/* 메서드 방식3️⃣: 랜더링 함수 내부에서 함수를 새롭게 만드는 방식 
        - 랜더링이 일어날때 마다 새로운 함수를 생성,할당함으로 최적화 수행이 어렵다.
        */}
        <button
          onClick={() => {
            this.handleClickDecrease();
          }}
          disabled={isLimited}
        >
          알람
        </button>
      </h2>
    );
  }
}

export default SampleComponent;
```

###클래스형 컴포넌트의 생명주기###
![Alt text](image-1.png)https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

**생명주기가 실행되는 시점**

- 마운트 (mount) : 컴포넌트가 마운팅(생성)되는 시점
- 업데이트 (update) : 컴포넌트 내용이 업데이트 되는 시점
- 언마운트 (unmount) : 컴포넌트가 더이상 존재 않는 시점

**render()**

- 실행시점: 마운트, 업데이트
- 컴포넌트가 UI를 랜더링하기 위해 쓰임
- 클래스형 컴포넌트의 필수 값
- 부수효과가 없는 순수함수여야한다
  - 같은 props 와 state 에는 같은 결과물을 반환
  - this.setState 호출은 안된다.

**componentDidMount()**

- 실행시점 : 마운트
- this.setState 가 허용됨 => 이후 즉시 랜더링을 시도한다.
  - API 호출 후의 업데이트
  - DOM 에 의존적인 작업
  - 만능이 아니다. 생성자 함수에서 실행 할 수 없는작업인지 잘 검토하고 실행하자.

**componentDidUpdate()**

- 실행시점 : 컴포넌트 업데이트 후
- this.setState 가 사용 가능하나, 적절한 조건문과 함께 사용하지 않으면 계속 이 메서드가 불리며 무한루프에 갇힐 수 있다.

```tsx
componentDidUpdate(prevProps: MyComponentProps, prevState: MyComponentState) {
  if (this.props.someValue !== prevProps.someValue) {
    this.setState({ /* update state */ });
  }
}
```

**componentWillUnmount()**

- 실행시점 : 컴포넌트 언마운트 직전에 호출
- 메모리 누수, 클린업 함수를 호출하기 위한 최적의 위치
  - 이벤트 리스너의 제거
  - setInterval, setTimeout 으로 생성된 타이머 제거

**shouldComponentUpdate()**

- 실행시점: state나 props의 변경으로 컴포넌트 리랜더링 되기 전에 호출
- 불필요한 리랜더링을 막아 준다.
  => 예 : setState 가 불렸지만, state 값이 변화가 없는 경우
- 성능 최적화의 상황에서 고려
- PureComponent를 확장해서 사용하는 경우 props와 state 에 대해 얇은 비교를 수행하도록 shouldComponentUpdate 함수를 작성하는 것과 같다.

```tsx
  shouldComponentUpdate(nextProps: CounterProps, nextState: CounterState) {
    return nextProps.value !== this.props.value;
  }

```

**static getDerivedStateFromProps()**

- 실행시점: render를 호출 직전
- static 으로 선언되어있어 this 에 접근할 수 없다.
- 반환하는 객체가 모두 state 로 들어가며, null 반환시 아무일도 일어나지 않는다.
- 사용을 권장하지 않습니다.

[You Probably Don't Need Derived State](https://ko.legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)

**getSnapShotBeforeUpdate()**

- DOM 이 업데이트되기 직전에 호출
- DOM에 랜더링 되기 전에 윈도우 크기나, 스크롤 위치를 반환해서 componentDidUpdate 에서 받아서 다시 조정할 때 쓰인다.

  ```js
    getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the scroll position so we can adjust scroll later.
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    // (snapshot here is the value returned from getSnapshotBeforeUpdate)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }
  ```

**getDerivedStateFromError()**

- 정상적인 생명주기가 아닌 자식컴포넌트의 에러상황에서 실행되는 메서드
- 23.12 기준 아직 기본훅으로 지원되지 않고 있다.
- 에러가 발생했을 때 어떻게 리액트 컴포넌트를 랜더링 용도로 정의해둔 state 를 반환한다
- 다른 부수효과를 발생시키지 않을 것을 권장한다.

```js
  static getDerivedStateFromError(error) {
    return {
      hasError: true,
      error: error,
    };
  }
```

**componentDidCatch()**

- 위의 getDerivedStateFromError가 실행되고 나서 실행이 된다.
- error 와, error 발생 정보인 errorInfo 를 반환한다.
- getDerivedStateFromError에서 실행하지 못했던 부수효과를 실행할 수 있다.
  - 에러정보를 로깅

getDerivedStateFromError ,componentDidCatch 두 메서드는 ErrorBoundary를 만들기 위한 목적으로 많이 사용된다.

- ErrorBoundary 는 어플리케이션 전역에서 처리되지 않은 에러를 처리하기 위한 용도로 사용된다.
- ErrorBoundary 외부의 에러는 잡을 수 없다.
- 개발모드에서는 componentDidCatch에서 잡힌 오류도 window 까지 전파되어 window error 이벤트로 추적이 가능하다.
- componentDidCatch의 errorInfo 의 componentStack 인수는 어느 컴포넌트에서 에러가 발생했는지 알수 있는 단서를 제공한다. 따라서 기명함수또는 displayName을 쓰는 것이 좋다.

###클래스형 컴포넌트의 한계

- 데이터의 흐름을 추적하기 어려움 : 여러 메서드에서 state 의 업데이트가 일어날 수 있으며, 어떤 흐름을 통해 state 가 변경되고 랜더링이 일어나는 지 판단이 어렵다.
- 애플리케이션 내부 로직의 재사용이 어려움 : 컴포넌트 간 중복되는 로직이 있을 때, 고차 컴포넌트 또는 컴포넌트 상속으로 중복 코드를 관리하는 것은 복잡도가 증가하고 코드 흐름 파악이 어렵게 한다.
- 함수 컴포넌트에 비해 상대적으로 어려움 : this 를 비롯해서, 다양한 생명주기 매서드 등 고려할 것이 많음
- 최종 결과물 번들 크기를 줄이는데 불리함. 불필요한 메서드도 트리쉐이킹 되지 않는것을 볼 수 있다.
- 핫 리로딩 시에 instance를 새로 생성하기에, state를 잃어 버린다.
  - 핫리로딩 : 코드 변경이 있을때 앱을 다시 시작 않고 변경사항을 적용하는 방법

###함수형 컴포넌트

- 생명주기 메서드의 부재 : React.Component 를 상속받아 구현할 때 사용할 수 있는 메서드이기에 함수형에서는 사용할 수 없다. 단, useEffect 를 이용해서 유사하게 구현할 수 있다.
- 랜더링이 일어날때 그 순간의 값인 props 와 state 를 기준으로 랜더링 된다.
  - 클래스형 컴포넌트는 시간의 흐름에 따라 변화하는 this 를 기준으로 랜더링이 일어난다.
  - 예 : 3초뒤 props 값을 출력하는 타이머 컴포넌트가 있을 때, prop값이 3초전에 변경되어도, 함수형 컴포넌트는 변경전의 값을 출력한다.

### 정리

- 지금 까지 많은 코드들이 클래스형 컴포넌트로 작성이 되었다. 이 흐름을 알기위해 클래스형 컴포넌트 지식도 필요할 것이다.
- 또한 자식 컴포넌트에 발생한 에러에 대한 처리는 클래스형 컴포넌트로만 가능해서 에러 처리를 위해서라도 클래스형 컴포넌트에 대한 지식이 필요하다.

## 2.4 렌더링은 어떻게 일어나는가?

**리액트의 렌더링이란**
리액트에서의 랜더링은 모든 컴포넌트들이 자신의 props 와 state 를 기반으로 브라우저에 제공하기 위한 DOM 결과물을 계산하는 과정

###리액트 랜더링이 일어나는 이유
**리액트에서 랜더링이 발생하는 시나리오**

1. 최초 랜더링 : 첫 어플리케이션 진입시 랜더링 할 결과물을 만들어 내기위해 최초 랜더링을 진행한다.
2. 리랜더링
   - 클래스형 컴포넌트 리랜더링
     - setState 가 실행되는 경우
     - forceUpdate 가 실행되는 경우
   - 함수형 컴포넌트
     - useState 두번째 배열요소 setter 가 실행되는 경우
     - useReducer 두번째 벼열 요소인 dispatch 가 실행이 되는 경우
   - 컴포넌트 공통
     - key props 가 변경되는 경우
       > key 가 왜 필요한가 ?
       >
       > - 형제 요소들 사이에서 동일한 요소를 식별하게 하는 값이다.
       > - 리액트 파이버가 두 트리사이에서 리랜더링이 필요한 컴포넌트를 최소화하기 위해 key로 구별한다.
       > - key 값이 바뀌면, 강제로 리랜더링을 일으키는 것이 가능하다.
     - props 가 변경되는 경우
     - 부모 컴포넌트가 랜더링 되는 경우

이 시나리오가 아니면 리랜더링을 발생시키지 않는다. 따라서 mobx-react, react-redux 같은 경우 위에서 언급한 방법중 하나를 사용해서 리랜더링을 발생시키게 된다.

###리액트 랜더링 프로세스
리액트의 랜더링은 랜더 단계와 커밋 단계 두단계로 분리되어 실행된다.

#### 랜더

- 컴포넌트를 실행해서 (render() 또는 return) 반환된 랜더링 결과물과 current fiber tree 를 비교해 변경이 필요한 컴포넌트를 체크해서 working fiber tree를 만듭니다.
  - type, props, key 가 다르면 변경이 필요한 컴포넌트로 체크된다.

#### 커밋

- 랜더 단계의 변경사항을 실제 DOM 에 적용해 사용자에게 보여준다.
- 리액트 내부의 참조를 업데이트해서 working fiber tree 를 current tree로 변경 한다.

=> 이 두가지 과정으로 이우어진 리액트의 랜더링은 항상 동기식이었는데, 리액트 18부터 의도된 우선 순위로 컴포넌트를 렌더링해 최적화할 수 있는 동시성 랜더링이 도입되었다.

☆ 랜더링이 일어난다고 무조건 DOM 업데이트가 일어나지 않는다. 랜더링을 수행해도 커밋 이 필요 없으면, 커밋 단계는 생략이 가능하다.

### 정리

- 부모가 변경되면 자식 컴포넌트는 리랜더링 됨으로, memo 를 이용해서 자식 컴포넌트가 동일 할때는 랜더링을 생략할 수 있다.
- 리액트의 랜더링 시나리오를 정확히 이해하면 컴포넌트의 투리 구조를 개선하거나 불필요한 랜더링 횟수를 줄여서 성능 좋은 리액트 웹 어플리케이션을 만드는데 도움이 될 것이다.

## 2.5 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션

리액트에서는 useMemo, useCallback 훅과 고차 컴포넌트인 memo 는 랜더링 최적화를 위해 리액트에서 제공하는 API 이다.

**memo를 썼을 때 지불해야하는 비용**

- 랜더링 결과물을 저장하기위한 cpu사용과 메모리
- props 에 대한 얕은 비교 : props 구조가 크고 복잡해지면 비용이 커질 수 있다.

**memo를 하지 않았을 때 발생하는 문제**

- 랜더링을 함으로 발생하는 비용
- 컴포넌트 내부 복잡한 로직의 재실행
- 자식 컴포넌트에서 반복해서 일어남

**useCallback, useMemo**

- 객체 내부의 값 또는 함수가 일정한데, 참조가 변하면 이를 사용하는 컴포넌트의 재랜더링을 일으킴으로 useCallback, useMemo 를 이용해서 같은 값에 대한 같은 결과물을 가지게 해야한다.

=> 즉 메모이제이션을 하지 않았을 때 치러야 할 잠재적인 위험 비용이 더 클 가능성이 높으므로, 최적화에 대한 확신이 없다면 가능한 모든곳에 메모이제이션을 활용해 최적화를 하자.
=> 랜더링 최적화에 시간을 쓸 여유가 있다면 개발자 도구나 useEffect 를 활용해 어떻게 랜더링이 일어나는 지 파악 후 필요한 곳만 최적화 하자

Q. svelte 는 가상DOM을 안쓰는 것인지? 다른 가상DOM을 안쓰는 프레임워크들은 어떤 방식으로 랜더링 최적화를 할까? 
Q. 다들 리액트 메모이제이션 주장1,2에 대해 공감이 가시는지, 어떻게 적용하고 계신가요? 의견이 있으신가요?