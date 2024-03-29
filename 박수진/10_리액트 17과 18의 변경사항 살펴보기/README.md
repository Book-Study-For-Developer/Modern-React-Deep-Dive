# 10장 리액트 17과 18의 변경 사항 살펴보기

2013 년 페이스북이 출시한 리액트는 11년이라는 시간이 흘렀다. 2024.2월기준 10억 다운로드를 돌파했다… 2022.8 월 18.2 버전이 릴리스 되었지만, 현재 사용되는 버전을 분석한 결과 버전 16의 비율이 높았다.

- 리액트가 더 가볍고 빠른 최신기능을 제공해 가고 있으니 팔로우 업 하면 좋다.
- 리액트 의존적인 라이브러리 ( peerDependencies 가 리액트에 의존) 가 react 최신 버전만 의존하게 되면, 해당 라이브러리를 사용하지 못할 수 있으니, 업데이트 해가는 것이 좋을 수 있다.

## 10.1 리액트 17버전 살펴보기

- 버전 16 과 다르게 새롭게 추가된 기능 없음
- 기존 코드의 수정에 필요로하는 변경사항을 최소화

### 10.1.1 리액트의 점진적인 업그레이드

- 17버전 전에는 주버전이 릴리즈 되면 이전 버전의 API 제공을 중단하며 전체 어플리케이션을 새로 업그레이드 하도록 했고, 개발자는 과거버전 머무르기 또는 현재버전 업데이트 둘중 하나를 선택해야했다.
- 17버전에는 점진적 업그레이드를 지원한다.
  - 일부 트리와 컴포넌트에 대해 다른 리액트 버전을 을 선택하는 점진적 버전업이 가능해진다.
    - 이는 버전 17 내부에서 16버전을 버전에 맞게 랜더링해주도록 되어있기 때문이다.
      → 리액트 팀은 이를 차선책으로 리액트 버전을 통일하는것이 좋다고 말한다.

### 10.1.2 이벤트 위임 방식의 변경

- 이벤트의 단계

  1. 캡처 : 이벤트 핸들러가 트리 최상단 요소 부터 시작해 이벤트 발생 타깃 요소까지 내려가는 것
  2. 타깃 : 이벤트 핸들러가 타깃 노드에 도달
  3. 버블링: 이벤트가 발생한 요소에서 다시 최상위 요소로 올라가는 것

- 이벤트 위임: 이벤트 단계의 원리를 활용해서 이벤트를 상위컴포넌트에 붙이는 것
  - li 요소들을 가지고 있는 ul 이 있고 li 에 이벤트가 필요하면, 상위 컴포넌트인 ul 에 붙이는 것
  - 리액트는 최초 release 부터 이벤트 위임을 적극적으로 사용했다.
    - 리액트 16에서는 이벤트가 각 요소가 아닌 document 에 위임된다.
      - ⁉️(이 부분을 아무리 읽어봐도 이해가 안간다. )따라서 하위 요소에서 이벤트 전파를 막는 e.stopPropagation 을 사용해도 이미 모든 이벤트가 document 에 있어서 document의 이벤트 전파는 막을 수 없다
      - 🔗[event-propagation-event-bubbling-event-catching-beginners-guide](https://www.freecodecamp.org/news/event-propagation-event-bubbling-event-catching-beginners-guide/)
    - 리액트 17 부터는 이벤트가 최상단 요소에 위임된다.

### 10.1.3 import React from ‘react’ 가 더이상 필요없다 : 새로운 JSX transfrom

- JSX 는 바벨이나 타입스크립트를 활용해서 일반 JS 로 변환하는 과정이 필요하다.
  - 16버전 : import React from “react” 가 있어야 JSX 변환이 사용가능
  - 17버전 : JSX 변환을 해보면 react/jsx-runtime 을 불러오는 require 구문이 추가되어 있음을 알 수 있다.

### 10.1.4 그 밖의 주요 변경 사항

- 이벤트 풀링 제거
  - 리액트 16에서 이벤트 풀링이라는 기능은 SyntheticEvent 풀을 만들어서 이벤트를 재사용하는데, 이벤트가 종료되면 바로 syntetic event 가 null 로 초기화 되어, 비동기 코드에서는 event 에 접근하면 null 이 뜨는 현상이 있어 e.persist() 를 사용해야했다.
  - 모던 브라우저에서 이벤트 처리에 대한 성능이 많이 좋아졌고, 비동기 코드에서 접근하기 위해 별도 메모리 공간에 합성이벤트 객체를 할당해야하는 점등이 성능향상에 크게 도움이 안되어 삭제 됨
- useEffect 클린업 함수의 비동기 실행
  - 리액트 17버전 부터 클린업 함수가 비동기적으로 실행된다. 컴포넌트의 커밋단계가 완료될 때까지 지연된다. 즉 리랜더링이 일어난 뒤에 실행되어 화면 업데이트가 반영되는 시간인 commitTime 이 빨라진 것을 볼 수 있다.
- 컴포넌트의 undefined 반환에 대한 일관적인 처리
  - 컴포넌트 내부에서 undefined 를 반환하면 에러가 발생했었는데, 리액트 18 부터는 에러가 발생하지 않는다고 한다.

### 10.1.5 정리

리액트16→17 은 큰 변화가 없다 . 다음 버전 업을 위한 버전업 정도 이므로 16을 사용중이라면 17버전으로 업그레이드 할 것을 추천한다.

## 10.2 리액트 18버전 살펴보기

리액트 18 에서는 리액트 16.8 에서 처음 선보였던 새로운 훅을 대거 추가했고, 동시성을 지원하며 다양한 개발자들의 의견을 들으며 심혈을 쏟았다.

### 10.2.1 새로 추가된 훅 살펴보기

- useId : 컴포넌트별로 유니크한 값을 생성하는 새로운 훅
  - 서버 사이드 랜더링환경에서 하이드레이션이 일어날때도 서버와 클라이언트에서 동일한 값을 가져야 에러가 발생하지 않는데, 리액트 17까지는 까다로운 작업이었다.
  - 생성 알고리즘에 대해 간단히 소개하면, id는 현재 트리에서 자신의 위치를 나타내는 32글자의 이진 문자열로 이루어져 있다.
- useTransition
  - 랜더링이 오래 소요되는 컴포넌트 A가 있다고 하자. 다른 탭을 눌러서 컴포넌트 B 를 보여야한다고 할 때, A 의 랜더링이 끝나야 해서 지연이 발생했다
  - useTransition을 쓰면 리액트 18의 동시성을 사용해 상태가 변경되면 기존 랜더링이 취소 될 수도, 다른 랜더링을 가로막지 않을 수도 있어서 사용자에게 좀 더 자연스러운 서비스를 경험하게 한다.
  - 컴포넌트에서만 사용 가능한 훅이다. 동기함수를 넘겨줘야한다.
- useDeferredValue
  - 리랜더링이 급하지 않은 부분을 지연할 수 있게 도와주는 훅
  - 디바운스와 비슷하지만 고정된 지연시간 없이 첫번째 랜더링 이후 useDeferredValue 로 지연된 랜더링을 수행한다.
  - useTransition 과 지연된 랜더링을 한다는 점에서 같지만, 상태업데이트에 관여 할 수 없고 props 처럼 오로지 값만 받아야하는 상황이면 useDeferredValue 를 사용해야한다.
- useSyncExternalStore
  - useSubscription의 구현이 리액트 18에서 useExternalStore 로 대체되었다.
  - 리액트 18에서 랜더링이 동기적이 아닌, 중지했다가 실행하는 등의 비동기가 가능해지면서 tearing 현상( 같은 상태값을 보고 있는데, 다르게 랜더링 되는 현상)이 일어날 수 있는데 이를 해결하기 위한 훅이다.
- useInsertionEffect
  - CSS-in-js 라이브러리를 위한 훅
  - Next.js 에 styled-components 를 적용하려면, styled-component 가 사용하는 스타일을 모아서 `<style>`태그에 삽입하는 작업이 필요한데, 이 작업을 훅으로 만든 것이다.
  - DOM 이 실제로 변경되기 전에 동기적으로 실행됨으로, 다시 스타일계산을 하지 않아도 된다.

### 10.2.2 react-dom/client

- 클라이언트에서 리액트 트리를 만들 때 사용되는 API 가 변경됨
- createRoot
  - render -> createRoote + render
  ```jsx
  const root = ReactDOM.createRoot(container);
  ```
- hydrateRoot
  - 서버사이드 랜더링 어플리케이션에서 하이드레이션을 하기 위한 새로운 매소드
  ```jsx
  const root = ReactDOM.hydrateRoot(container, <App />);
  ```
- 위의 두메서드는 onRecoverableError 를 인수로 받아서 에러발생시 실행할 콜백을 등록할 수 있다.

### 10.2.3 react-dom/server

클라이언트의 변화와 마찬가지로 서버에서도 컴포넌트 생성 API에 변경이 있다.

- renderToPipeableStream

  - 리액트 컴포넌트를 HTML 로 랜더링 하는 메서드
  - 스트림을 지원해서, 점진적으로 랜더링하면서, 중간에 script 를 삽입하는 등 작업이 가능하다.
  - renderToPipeableStream 을 쓰면 아직 불러오지 못한 데이터 부분을 Suspens의 fallback 으로 받는다.

  ```jsx
  const App = () => {
    return (
      <div>
        <h1>My App</h1>
        <Suspense fallback={<div>Loading...</div>}>
          <MyComponent />
        </Suspense>
      </div>
    );
  };
  ```

- renderToReadableStream
  - renderToPipeableStrea이 Node.js 의 랜더링을 위한 것이라면, renderToReadableStream 은 웹 스트림 기반으로 작동한다. 이는 웹 스트림을 사용하는 Cloudflare, Deno 같은 모던 엣지 런타임 환경에서 사용된다.

### 10.2.4 자동 배치 (Automatic Batching)

- 리액트가 여러 상태 업데이트를 하나의 리랜더링으로 묶어서 성능을 향상 시키는 방법
- 버튼 클릭 한번에 두개 이상의 state 를 동시에 업데이트 할때, 자동 배치에서는 이를 하나의 리랜더링으로 묶어서 수행할 수 있다.
- 리액트 17의 자동 배치 : 이벤트 핸들러 내부에서는 자동 배치 작업이 이루어졌고, Promise와 setTimeout 같은 비동기 이벤트에서는 자동배치가 이뤄지고 있지 않았다. -> 리액트 18에서 부터는 오든 업데이트가 배치 작업으로 최적화 된다.
- 리액트 18에서도 자동배치를 하고 싶지 않은 경우에는 reactDom 의 flushSync를 사용하자

### 10.2.5 엄격해진 엄격모드

리액트에서 제공하는 컴포넌트로 잠재적인 버그를 찾는데 도움이 된다. 개발자 모드에서만 작동한다.

1. 더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고

- UNSAFE\_ 가 붙은 옛 생명주기들 ... componentWillMount ... 등을 사용하면 경고를 한다.

2. 문자열 ref 사용금지

   ```
       class WrongRefExample extends Component(){
         componentDidMount(){
           console.log(this.refs.myInput)
         }

         return (
           <div>
             {/* 잘못된 방법: ref에 문자열을 할당 */}
             <input ref="myInput" type="text" />
           </div>
         );
     };

   ```

3. findDOMNode에 대한 경고 출력

- 클래스형 컴포넌트 인스턴스에서 사용되었던 실제 DOM 요소의 참조를 자져오는 메서드로 더이상 권장하지 않음
- 문자열 ref와 마찬가지로 findDOMNode 는 createRef, useRef를 사용하는 방향으로 전환되어 지원 중단됐다.

2. 구 Contexgt API 사용시 경고 발생
3. 예상치 못한 부작용 검사

- 리액트 엄격모드에서는 다음을 두번씩 호출해서 컴포넌트가 순수한 결과물을 내는지 확인한다.
  - 클래스형 컴포넌트의 constructor, render, shouldComponentUpdate, getDerivedStateFromProps
  - 클래스형 컴포넌트의 setStat의 첫번째 인수
  - 함수형 컴포넌트의 body
  - useState, useMemo, useReducer 에 전달되는 함수
- 리액트 17에서는 엄격모드 console.log 를 기록하지 않았다면, 리액트 18에서는 기록을하되 회색으로 표시한다.
- 향후 리액트에서는 컴포넌트가 마운트 해제된 상태에서도 컴포넌트 내부 상태값을 유지할 수 있는 기능을 제공할 예정이라고 한다.
  - 이후에 있을 변경을 위해 StrictMode 에 고의로 useEffect를 두번 작동시키는 내용을 추가했다.
  - 두번의 useEffect 호출에도 자유로운 컴포넌트를 작성해야한다.

### 10.2.6 Suspense 기능 강화
Suspense 는 리액트 16.6 버전에서 실험버전으로 도입되어, 컴포넌트를 동적으로 가져올 수 있게 한다.
- 18 이전의 Suspense 에는 몇가지 문제가 있었고 18에서 해결이 되었다.
  - 컴포넌트가 아직 보이기 전에 useEffect 가 실행되는 문제 
    -> 실제로 컴포넌트가 노출될때 effect 실행
  - 서버에서 Suspense 를 사용할 수 없었던 문제
    -> 서버에서는 fallback의 트리를 먼저 클라이언트에 제공하고, 불러올 준비가 되면 랜더링 됨
### 10.2.7 인터넷 익스플로러 지원중단에 따른 추가 폴리필 필요
- 이제 리액트는 Promise, Symbol, Object.assign을 사용할 수 있다는 가정하에 배포됨
- 요즘 출시되는 대부분의 라이브러리가 ES5 지원을 중단하는 추세로, 웹서비스가 인터넷 익스플로러 11을 지원해야하면, 폴리필 설치 및 트랜스 파일에 신경을 써야함

### 10.2.8 그 밖의 변경사항
- 이제 컴포넌트에서 undefined 를 반환해도 에러 없이 null 반환과 같이 처리됨
  - `<Suspense fallback={undefined}>` 도 null 과 동일하게 처리됨
- renderToNodeStream 이 지원 중단되고, renderToPipableStream 을 사용하는 것이 권장된다.

### 10.2.9 정리
- 리액트 18 버전업의 핵심은 동시성 랜더링이다. 이제 랜더링은 한번 시작하면 멈출 수 있고, 나중에 다시 시작 할 수도 있다. 하지만 이런 과정에서도 UI 가 일관 되게 표시하게 보장한다.
- 리액트 외부 상태로 일관성 유지가 안될때 (티어링) useSyncExternalStore 훅을 사용해서 일관성을 유지하게 한다.
- 앞으로 개발할 어플리케이션에 동시성 모드를 염두에 두고 있다면 사용하려는 라이브러리가 이를 완전히 지원하는 지 검토가 필요하다.
