## 1. 서버 사이드 렌더링이란?

### **싱글 페이지 애플리케이션이란?**

싱글 페이지 애플리케이션(Single Page Application: SPA) 는 렌더링과 라우팅에 필요한 대부분의 기능을 서버가 아닌 브라우저의 자바스크립트에 의존하는 방식을 의미한다.
최초의 페이지에서 데이터를 불러온 이후 페이지 전환은 자바스크립트와 브라우저의 `history.pushState` 와 `history.replaceState`로 이뤄진다.

사이트의 소스 보기로 HTML코드를 보면 <body/> 내부에 아무런 코드가 없고, 최초에 데이터를 불러온 이후에 자바스크립트 리소스와 브라우저 API를 기반으로 모든 작동이 이뤄진다.

### **싱글 페이지 렌더링 방식의 유행과 JAM 스택의 등장**

- 과거 PHP나 JSP를 기반으로 만들어졌을 때 서버 사이드에서 렌더링이 이뤄짐
- 자바스크립트가 서서히 다양한 작업을 수행하게 되면서 자바스크립트를 모듈화하는 방안으로 **CommonJS**와 **AMD(Asynchronous Module Definition)**이 나왔다.
- 이후 웹페이지의 모든 영역페이지에서의 렌더링에서부터 인터랙션까지 모두 아우를 수 있는 싱글 페이지 렌더링이 인기를 얻게 됐다.
- 싱글 페이지 애플리케이션의 유행으로 인해 **JAM 스택**이라는 새로운 용어가 나타났다.
- LAMP 스택 - Linux(운영체제), Apache(서버), MySQL(데이터베이스), PHP/Python (웹프레임워크) 로 구성되었다.
  - 과거엔 자바스크립트에서 할 수 있는 일이 제한적이여서 서버 의존적인 문제로 인해 확장성에도 걸림돌로 작용
  - 사용자가 늘어나면 서버를 확장해야 했지만 클라우드 개념 부족으로 인해 서버 확장이 매우 번거로움
- JAM 스택 - Javascript, API, Markup 스택이 등장했고, 사용자의 클라이언트에서 실행되어 서버 확장성 문제에서 자유로워졌다.
  - JAM 스택의 인기 + Node.js의 고도화 → **MEAN(MongoDB, Express.js, AngularJS, Node.js)** 나 **MERN(MongoDB, Express.js, React, Node.js)** 스택처럼 API서버도 자바스크립트로 구현하는 구조가 인기를 끌었다.

### 새로운 패러다임의 웹서비스를 향한 요구

![[평균 자바스크립트 코드의 크기](https://httparchive.org/reports/state-of-javascript?start=earliest&end=latest&view=list)](https://prod-files-secure.s3.us-west-2.amazonaws.com/253d1ac1-0c8d-4179-8d90-21ade38e0aea/b1be6348-fb23-4a8b-8474-292f4a7b0ee5/Untitled.png)

[평균 자바스크립트 코드의 크기](https://httparchive.org/reports/state-of-javascript?start=earliest&end=latest&view=list)

![[스크립트가 페이지당 소비하는 CPU 시간](https://httparchive.org/reports/loading-speed?start=earliest&end=latest&view=list)](https://prod-files-secure.s3.us-west-2.amazonaws.com/253d1ac1-0c8d-4179-8d90-21ade38e0aea/c2a7d0dd-358a-4dcf-94fc-c74591667d48/Untitled.png)

[스크립트가 페이지당 소비하는 CPU 시간](https://httparchive.org/reports/loading-speed?start=earliest&end=latest&view=list)

현재의 웹 애플리케이션은 다양한 작업을 처리하고 있고, 앱 내부에서도 웹처럼 구동되는 경우도 많다. **사용자의 기기와 인터넷 속도 등 웹 전반을 이루는 환경이 크게 개선됐음에도 실제 사용자들이 느끼는 웹 애플리케이션의 로딩 속도는 예전이나 지금이나 큰 차이가 없거나 오히려 더 느리다는 것이다.**

## 서버 사이드 렌더링이란?

서버 사이드 렌더링은 최초에 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자에게 화면을 제공하는 방식이다.

싱글 페이지 애플리케이션과 서버 사이드 렌더링의 차이는 **웹페이지 렌더링의 책임을 어디에** 두느냐다.

- 싱글 페이지 애플리케이션: 사용자에게 제공되는 자바스크립트 번들에서 렌더링 담당
- 서버 사이드 방식: 모두 서버에서 렌더링에 필요한 작업 수행

**장점**

- 최초 페이지 진입이 비교적 빠르다.
  SPA라면 자바스크립트 리소스를 다운로드하고 HTTP 요청을 수행한 이후에 응답의 결과를 가지고 화면의 렌더링을 해야 하지만 서버에서 이뤄지기에 빠르게 렌더링 된다.
  모든 경우에 서버 사이드 렌더링이 이점을 가진다고 볼 수는 없지만 화면 렌더링이 HTTP 요청에 의존적이거나 렌더링해야 할 HTML의 크기가 커지면 상대적으로 빠를 수 있다.
- 검색 엔진과 SNS 공유 등 메타데이터 제공이 쉽다.
  검색 엔진이 사이트에서 필요한 정보를 가져가는 과정
  1. 검색 엔진 로봇이 페이지에 진입
  2. 페이지가 HTML 정보를 제공해 로봇이 HTML을 다운로드**(자바스크립트 코드는 실행하지 않음)**
  3. 다운로드한 HTML 내부의 오픈 그래프(Open Graph)나 메타(meta) 태그 정보를 기반으로 페이지의 검색 정보를 가져와 검색 엔진에 저장
  자바스크립트 코드를 실행하지 않기에 정적인 정보만 가져온다. SPA는 자바스크립트에 의존하기에 페이지에 최초로 진입했을때 검색엔진에 걸리지 않는다.
  반면 서버 사이드 렌더링은 서버에서 최초 렌더링을 하고 검색 엔진에 제공할 정보도 가공하므로 최적화에 대응하기 매우 용이하다.
- 누적 레이아웃 이동이 적다.
  **누적 레이아웃 이동(Cumlulative Layout shift)란** 페이지를 보여준 이후에 뒤늦게 어떤 HTML 정보가 추가되거나 삭제되어 화면이 덜컥거리는 사용자 경험을 말한다.
  SPA의 경우엔 페이지 콘텐츠가 API 요청에 의존하고, API 요청의 응답 속도가 제각각이기에 CLS가 발생할 수 있다. 반면 SSR의 경우 요청이 모두 완료된 이후에 완성된 페이지를 제공하므로 이러한 문제에서 자유롭다.
  그렇다고 해서 완전히 자유롭진 않다.
  ex) `useEffect`는 컴포넌트가 마운트된 이후에 실행되므로 문제의 소지가 존재
- 사용자의 디바이스 성능에 비교적 자유롭다.
  서버에서 렌더링을 수행하면 리소스의 부담을 사용자 디바이스가 아닌 서버에 나눌 수 있으므로 조금 더 자유로워질 수 있다.
- 보안이 좀 더 안전하다.
  JAM 스택의 공통적인 문제점은 애플리케이션의 모든 활동이 브라우저에 노출된다는 것이다.
  API 호출과 인증 같이 사용자에게 노출되면 안되는 민감한 정보도 포함되기에 항상 방지할 준비가 돼 있어야 한다. 반면 SSR의 경우 **민감한 작업을 서버에서 수행해 브라우저에는 결과만 제공하므로 보안 위협을 피할 수 있다.**

**단점**

- 소스코드를 작성할 때 항상 서버를 고려해야 한다.
  브라우저 전역 객체인 window 또는 sessionStorage와 같이 브라우저에만 있는 전역 객체를 고려해야 한다.
  외부에 의존하고 있는 라이브러리에도 서버에 대한 고려가 돼 있지 않다면 다른 대안이나 클라이언트에서만 실행될 수 있도록 처리해야 한다. 잠재적인 위험을 가진 코드를 **모두 클라이언트에서 실행하는 것이 해결책은 아니다. 결국 클라이언트에서 실행되는게 많아지면 SSR의 장점을 잃는 것이다.**
- 적절한 서버가 구축돼 있어야 한다.
  SSR의 경우엔 사용자의 요청을 받아 렌더링을 수행할 서버가 필요하고, 서버를 구축하게 되면 **물리적인 가용량**, 장애 상황에 대한 **복구 전략**, 프로세스 다운을 대비한 **PM2 같은 프로세스 매니저** 등 여러가지 방안을 고려해야 한다.
- 서비스 지연에 따른 문제
  SPA의 경우엔 최초에 어떤 화면이라도 보여준 상태에서 느린 작업이 수행되기에 “로딩 중”과 같이 적절히 안내하여 사용자가 기다릴 여지를 준다.
  그러나 SSR은 **렌더링 작업이 끝나기까지 사용자에게 어떠한 정보도 제공할 수 없다.** 이로 인해 더 안 좋은 사용자 경험을 제공할 수 있다.

## SPA와 SSR을 모두 알아야 하는 이유

두 방법 중 어느 것이 옳다고 단언할 수 없지만 SPA와 SSR의 방식인 MPA에 대해서 다음과 같이 이야기 할 수 있다.

1. **가장 뛰어난 SPA는 가장 뛰어난 MPA보다 낫다.**
   최초 진입 시에는 필요한 정보만 최적화해서 요청해 렌더링하고, 코드 분할 또한 칼같이 이뤄져서 불필요한 자바스크립트 리소스 다운로드도 방지하고, 라우터 이동에도 변경에 필요한 부분만 교체해 UX도 챙겼다. 그렇다면 SPA의 매꾸러운 라우팅보다 MPA의 성능이 따라오지는 못한다.
2. 펑균적인 SPA는 평균적인 MPA보다 느리다.
   MPA는 매번 서버에 렌더링을 요청하고 서버는 매번 비슷한 성능의 렌더링을 수행할 것이다. 그러나 SPA는 렌더링/라우팅 최적화가 돼 있지 않다면 사용자의 기기에 따라 성능이 다르고, 모든 상황을 최적화하는 것은 매우 어렵다. 또한 MPA에서 발생하는 라우팅 문제를 해결하기 위한 다양한 API가 브라우저에 추가되고 있다. - [페인트 홀딩](https://developer.chrome.com/blog/paint-holding?hl=ko): 같은 origin에서 라우팅이 일어날 경우 화면을 잠깐 하얗게 띄우는 대신 이전 페이지의 모습을 잠깐 보여주는 기 - [back forward cache(bfcache)](https://web.dev/articles/bfcache?hl=ko): 브라우저 앞으로가기, 뒤로가기 실행 시 캐시된 페이지를 보여주는 기법 - [Shared Element Transitions](https://github.com/WICG/view-transitions): 라우팅이 일어났을 때 두 페이지에 동일 요소가 있다면 해당 콘텍스트를 유지해 부드럽게 전환되게 하는 기법

**현대의 SSR은 최초의 진입시에는 완성된 HTML을 제공받고, 이후 라우팅에서는 서버에서 내려받은 자바스크립트를 바탕으로 SPA처럼 작동된다.** 따라서 서버에서의 렌더링, 클리이언트에서의 렌더링을 모두 이해해야 두 가지 장점을 완벽하게 취하는 제대로 된 웹서비스를 구축할 수 있다.

## 2. 서버 사이드 렌더링을 위한 리액트 API 살펴보기

React는 서버에서 렌더링 할 수 있는 API 도 제공하고 이는 Node.js와 같은 서버 환경에서만 실행할 수 있다.

### renderToString

인수로 넘겨받은 리액트 컴포넌트를 렌더링해 HTML 문자열로 반환하는 함수다.

```tsx
function ChildrenComponent({ fruits }: { fruits: Array<string> }) {
	function handleClick() {
		console.log('hello');
	}

	return (
		<ul>
			{fruits.map((fruit) => (
				<li key={fruit} onClick={handleClick}>
				{fruit}
				</li>
			)}
		</ul>
	)
}

function SampleComponent() {
	return (
		<>
			<div>hello</div>
			<ChildrenComponent fruits={['apple', 'banana', 'peach']} />
		</>
	)
}

const result = ReactDOMServer.renderToString(
	React.createElement('div', { id: 'root' }, <SampleComponent />),
)

// result

<div id="root" **data-reactroot="">**
	<div>hello</div>
	<ul>
		<li>apple</li>
		<li>banana</li>
		<li>peach</li>
	</ul>
</div>
```

실제 브라우저가 그려야 할 HTML 결과로 만들어 낸다.

이때 `useEffect`같은 훅과 `handleClick`과 같은 이벤트 핸들러는 포함되지 않는다. 검색 엔진이나 메타 정보를 위해 미리 준비한 채로 제공할 수 있다.

그리고 `data-reactroot` 속성이 존재하는데 이 속성은 **hydrate함수에서 루트를 식별하는 기준점이 된다.**

### renderToStaticMarkup

`renderToString`과 매우 유사하지만 다른점은 `data-reactroot` 속성을 만들지 않는다.

```tsx
const result = ReactDOMServer.renderToStaticMarkup(
	React.createElement('div', { id: 'root' }, <SampleComponent />),
)

// result

<div id="root"**>**
	<div>hello</div>
	<ul>
		<li>apple</li>
		<li>banana</li>
		<li>peach</li>
	</ul>
</div>
```

`renderToStaticMarkup` 는 완전히 순수한 HTML만을 반환하기에 hydrate를 수행하면 서버와 클라이언트의 내용이 맞지 않는다는 에러가 발생한다. 즉, 아무런 브라우저 액션이 없는 정적인 내용만 필요한 경우에 유용하다.

### renderToNodeStream

`renderToString` 과 결과물이 완전히 동일하지만 브라우제어사는 사용하는 것이 불가능하다. `renderToNodeStream`의 결과물은 Node.js의 ReadableStream(utf-8 인코딩된 바이트 스트림)이여서 Node.js에서만 사용할 수 있다.

`renderToString`으로 생성해야하는 HTML크기가 매우 크다면 서버에 부담이 가므로 스트림을 활용해서 크기가 큰 데이터를 청크 단위로 분리해 순차적으로 처리할 수 있다는 장점이 있다.

```tsx
export default function App({ todos }: { todos: Array<TodoResponse> }) {
	return (
		<>
			<ul>
				{todos.map((todo, index) => (
					<Todo key={index} todo={todo} />
				)}
			</ul>
		</>
	)
}

// Node.js 코드
;(async () => {
	const response = await fetch('http://localhost:3000');

	try {
		for await (const chunk of reponse.body) {
			console.log('----chunk----')
			console.log(Buffer.from(chunk.toString());
		}
	} catch (err) {
		console.error(err.stack);
	}
})()
```

todo의 크기가 매우 크다면 `renderToNodeStream` 을 통해 청크로 분리돼 내려오게 해 서버의 부담을 줄일 수 있다. (리액트 서버 사이드 렌더링 프레임워크는 모두 `renderToNodeStream`을 채택)

### renderToStaticNodeStream

`renderToNodeStream` 과 제공하는 결과물은 동일하나, 리액트 자바스크립트에 필요한 리액트 속성이 제공되지 않는다. 순수 HTML 결과물이 필요할 때 사용하는 메서드이다.

### hydrate

hydrate는 `renderToStream`, `renderToNodeStream`으로 생성된 HTML 컨텐츠에 자바스크립트 핸들러나 이벤트를 붙이는 역할을 한다.

```tsx
const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);

// containerId 를 가리키는 element는 서버에서 렌더링된 HTML의 특정 위치를 의미한다.
const element = document.getElementById(containerId);
ReactDOM.hydrate(<App />, element);
```

render와 다르게 hydrate는 이미 렌더링된 HTML이 있다는 가정하에 작업이 수행되고 이벤트를 붙이는 작업만 실행한다.

### 서버 사이드 렌더링 예제 프로젝트

특정 /api 에서 할 일 목록을 가져오고, 각 할 일을 클릭해 useState로 완료 여부를 변경할 수 있는 간단한 구조로 설계한 것이다.

```tsx
import React from "react";
import { hydrate } from "react-dom";

import App from "./components/App";
import { fetchTodo } from "./fetch";

async function main() {
  const result = await fetchTodo();

  const app = <App todos={result} />;
  const el = document.getElementById("root");

  hydrate(app, el);
}

main();
```

**hydrate는 서버에서 완성한 HTML과 하이드레이션 대상이 되는 HTML의 결과물이 동일한지 비교하는 작업을 거치므로, 이 비교 작업을 무사히 수행하기 위해 한 번 더 데이터를 조회한다. (이 부분이 이해가 안갑니다… 하하**

```tsx
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>SSR Example</title>
  </head>
  <body>
    __placeholder__
    <script src="https://unpkg.com/react@17.0.2/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17.0.2/umd/react-dom.development.js"></script>
    <script src="/browser.js"></script>
  </body>
</html>
```

- **placeholder** : 서버에서 리액트 컴포넌트를 기반으로 만든 HTML 코드를 삽입하는 자리
- unpkg: npm 라이브러리를 CDN으로 제공하는 웹서비스이다. (번들링을 제외하고 unpkg에서 제공한 react,react-dom을 사용해 처리)
- browser.js: 클라이언트 리액트 애플리케이션 코드를 번들링했을 때 제공되는 리액트 자바스크립트 코드 (**placeholder** 에 HTML이 삽입된 후에 자바스크립트 이벤트 핸들러가 붙는다.)

**Server.ts**

**createServer**

```tsx
function main() {
  createServer(serverHandler).listen(PORT, () => {
    console.log(`Server has been started ${PORT}...`); // eslint-disable-line no-console
  });
}
```

3000번 포트를 이용하는 HTTP 서버를 만드는 코드

**serverHandler**

```tsx
async function serverHandler(req: IncomingMessage, res: ServerResponse) {
  const { url } = req;

  switch (url) {
    // ...

    default: {
      res.statusCode = 404;
      res.end("404 Not Found");
    }
  }
}
```

서버가 라우트(주소)별로 어떻게 동작할지 정의하는 함수이다.

```tsx
 switch (url) {
		case '/': {
      const result = await fetchTodo()

      const rootElement = createElement(
        'div',
        { id: 'root' },
        createElement(App, { todos: result }),
      )
      const renderResult = renderToString(rootElement)

      const htmlResult = html.replace('__placeholder__', renderResult)

      res.setHeader('Content-Type', 'text/html')
      res.write(htmlResult)
      res.end()
      return
    }
```

/ 에 접근했을 때 `renderToString` 를 사용해 리액트 컴포넌트 HTML을 만들고 ‘**placeholder**’ 자리에 대체하여 응답을 제공한다.

```tsx
case '/stream': {
      res.setHeader('Content-Type', 'text/html')
      res.write(indexFront)

      ****const result = await fetchTodo()
      const rootElement = createElement(
        'div',
        { id: 'root' },
        createElement(App, { todos: result }),
      )

      const stream = renderToNodeStream(rootElement)
      stream.pipe(res, { end: false })
      stream.on('end', () => {
        res.write(indexEnd)
        res.end()
      })
      return
}
```

여기서는 **indexFront**, **indexEnd** 로 index.html이 반반 나뉘어진 코드에서 각각 응답을 기록하고 청크단위로 생성하여 나머지 반을 붙여서 최종 결과물을 브라우저에 리턴한다. **결과물은 동일하고 서버에서의 차이점밖에 없다.**

최종 코드는 [여기서](https://github.com/wikibook/react-deep-dive-example/tree/main/chapter4/ssr-example) 확인이 가능하다.

## 3. Next.js 톺아보기

### Next.js란?

Vercel이라는 미국 스타트업에서 만든 리액트 기반 서버 사이드 렌더링 프레임워크다. Next.js 전에 페이스북 팀에서 리액트 기반 서버 사이드 렌더링을 위해 고려했던 프로젝트인 `[react-page](https://github.com/facebookarchive/react-page)`도 있었다.

### Next.js 시작하기

**pages/\_app.tsx**

애플리케이션의 전체 페이지의 시작점이고, 공통으로 설정해야 하는 것들을 여기서 실행할 수 있다.

- 에러 바운더리를 사용해 애플리케이션 전역에서 발생하는 에러 처리
- reset.css 같은 전역 CSS 선언
- 모든 페이지에 공통으로 사용 또는 제공해야 하는 데이터 제공

**[pages/\_document.tsx](https://github.com/wikibook/react-deep-dive-example/blob/main/chapter4/next-example/src/pages/_document.tsx)**

```tsx
import { Html, Head, Main, NextScript } from "next/document";

export default function Document() {
  return (
    <Html lang="en">
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

\_document.tsx는 애플리케이션의 HTML을 초기화하는 곳이다.

- <html>, <body>에 DOM 속성을 추가할 때 사용한다.
- 무조건 서버에서만 실행되기에 onClick과 같은 이벤트 핸들러를 추가하는 것은 불가능
- `getServerSideProps`, `getStaticProps`등 서버에서 사용 가능한 데이터 불러오기 함수는 사용할 수 없다.
- CSS-in-JS의 스타일을 서버에서 모아 HTML로 제공하는 작업도 가능하다.

**[pages/\_error.tsx](https://github.com/wikibook/react-deep-dive-example/blob/main/chapter4/next-example/src/pages/_error.tsx)**

```tsx
import { NextPageContext } from "next";

function Error({ statusCode }: { statusCode: number }) {
  return <>{statusCode ? `서버에서 ${statusCode}` : "클라이언트에서"} 에러가 발생했습니다.</>;
}

Error.getInitialProps = ({ res, err }: NextPageContext) => {
  const statusCode = res ? res.statusCode : err ? err.statusCode : "";
  return { statusCode };
};

export default Error;
```

클라이언트에서 발생하는 에러 또는 서버에서 발생하는 500 에러를 처리할 목적으로 쓰인다.
개발 모드에서는 접근이 불가능하고 프로덕션 빌드에서 확인이 가능하다.

**[pages/404.tsx](https://github.com/wikibook/react-deep-dive-example/blob/main/chapter4/next-example/src/pages/404.tsx)**

```tsx
import { useCallback } from "react";

export default function My404Page() {
  const handleClick = useCallback(() => {
    console.log("hi"); // eslint-disable-line no-console
  }, []);
  return (
    <h1>
      페이지를 찾을 수 없습니다. <button onClick={handleClick}>클릭</button>
    </h1>
  );
}
```

페이지 이름 그대로 404 페이지를 정의할 수 있는 파일이다. 원하는 스타일의 404 페이지를 이곳에서 만들 수 있다.

**[pages/500.tsx](https://github.com/wikibook/react-deep-dive-example/blob/main/chapter4/next-example/src/pages/500.tsx)**

서버에서 발생하는 에러를 핸들링하는 페이지이다. \_error.tsx와 500.tsx중 500.tsx의 우선순위가 높고 둘다 없다면 Next.js에서 기본으로 제공하는 페이지가 노출된다.

### **서버 라우팅과 클라이언트 라우팅 차이**

Next.js는 서버 사이드 렌더링을 수행하지만 동시에 SPA처럼 클라이언트 라우팅 또한 수행한다.

```tsx
export default function Hello() {
  console.log(typeof window === "undefined" ? "서버" : "클라이언트");

  return <>hello</>;
}

export const getServerSideProps = () => {
  return {
    props: {},
  };
};

// Home.tsx
import type { NextPage } from "next";
import Link from "next/link";

const Home: NextPage = () => {
  return (
    <ul>
      <li>
        {/* next의 eslint 룰을 잠시 끄기 위해 추가했다. */}
        {/* eslint-disable-next-line */}
        <a href="/hello">A 태그로 이동</a>
      </li>
      <li>
        {/* 차이를 극적으로 보여주기 위해 해당 페이지의 리소스를 미리 가져오는 prefetch를 잠시 꺼두었다. */}
        <Link prefetch={false} href="/hello">
          next/link로 이동
        </Link>
      </li>
    </ul>
  );
};

export default Home;
```

`a`태그로 이동하는 것과 `Link`태그로 이동하는 것은 차이점이 있다.
`a` 태그로 이동하는 경우에는 페이지를 만드는데 필요한 **모든 리소스를 처음부터 다 가져오고**, 서버와 클라이언트가 동시에 찍힌다.

반면 `Link` 태그로 이동하는 경우에는 **클라이언트에서만 필요한 자바스크립트만 불러온 뒤** 클라이언트 라우팅/렌더링 방식으로 동작한다.

**페이지에서 getServerSideProps를 제거하면 어떻게 될까?**

Hello컴포넌트 에 `getServerSideProps`를 제거한 뒤 빌드 후 실행해보면 a태그, Link태그에 상관없이 서버에 로그가 남지 않는다.

**getServerSideProps가 있는 빌드와 getServerSideProps가 없는 빌드**

`getServerSideProps` 가 있으면 서버 사이드 런타임 체크가 이뤄지고, 없으면 서버 사이드 렌더링이 필요 없는 페이지로 간주해서 빌드 시점에 미리 페이지를 만들어 버린다. 따라서 typeof window를 모두 object로 바꿔 빌드 시점에 미리 트리쉐이킹을 해버려 ‘클라이언트’로 출력이 되버린다.

**/pages/api/hello.ts**

api라고 작성된 디렉토리 안에서는 서버의 API를 정의하는 폴더이다.

오로지 서버에서만 실행되고 window, document 등 브라우저에서 접근할 수 있는 코드를 작성하면 문제가 발생한다.

**서버에서 내려주는 데이터를 조합해 BFF(backend-for-fronted) 형태로 활용하거나, 풀스택 애플리케이션을 구축하고 싶을 때, 혹은 CORS 에러를 우회하기 위해 사용될 수 있다.**

## Data Fetching

### getStaticPaths와 getStaticProps

```tsx
export const getStaticPaths: GetStaticPaths = (async = () => {
  return {
    paths: [{ params: { id: "1" } }, { params: { id: "2" } }],
    fallback: false,
  };
});

export const getStaticProps: GetStaticProps = (async = ({ params }) => {
  const { id } = params;

  const post = await fetchPost(id);

  return {
    props: { posts },
  };
});

export default function Post({ post }: { post: Post }) {
  // post 페이지 렌더링
}
```

- `getStaticPaths`는 /pages/post/[id] 가 접근 가능한 주소를 정의하는 함수다. 위에서는 /post/1, /post/2만 접근 가능하게 하고 나머지의 경우 404를 반환한다.
- `getStaticProps`는 해당 페이지로 요청이 왔을 때 제공할 props를 반환하는 함수다. 각각 함수의 응답 결과를 변수로 가져와 { post }로 반환한다.

**두 함수를 사용하면 빌드 시점에 미리 데이터를 불러온 다음에 정적인 HTML 페이지를 만들 수 있다.**

즉, 사용자가 접근할 수 있는 페이지를 모조리 빌드해 두고 배포하면 사용자는 굳이 페이지가 렌더링되는 것을 기다릴 필요 없이 페이지를 받기만 하면되서 빠르게 페이지를 확인할 수 있다.

### getServerSideProps

getServerSideProps는 서버에서 실행되는 함수로 해당 함수가 있으면 무조건 페이지 진입 전에 이 함수를 실행한다. 응답 값에 따라 페이지의 루트 컴포넌트에 props를 반환하거나, 다른 페이지로 리다이렉트 시킬 수 있다.

```tsx
export const getSercerSideProps: GetServerSideProps = async (context) => {
	const {
		query: { id = '' },
	} = context;
	const post = await fetchPost(id.toString())
	return {
		props: { post },
	}
}

export defautl function Post({ post }: { post: Post }) {
}
```

[context.query.id](http://context.query.id) 를 통해 /post/[id] 같은 경로의 id 값에 접근이 가능하고 이 값을 통해 렌더링을 수행할 수 있다.

```html
<body>
  <script id="__NEXT_DATA__" type="application/json">
    {
      "props": {
        "pageProps": {
          "post": { "title": "안녕하세요", "contents": "반갑습니다." }
        },
        "__N_SSP": true
      },
      "page": "/post/[id]",
      "query": { "id": "1" },
      "buildId": "development",
      "isFallback": false,
      "gssp": true,
      "scriptLoader": []
    }
  </script>
</body>
```

HTML이 getSererSideProps 기반으로 렌더링되어 있고 여기서 눈여겨봐야 할 것은 `id="__NEXT_DATA__"` 의 script 이다.

SSR의 동작 방식

1. 서버에서 fetch등으로 렌더링에 필요한 정보를 가져온다.
2. 가져온 정보를 기반으로 HTML을 완성
3. 2번의 정보를 클라이언트에 제공
4. 3번의 정보를 바탕으로 클라이언트에서 hydrate작업을 한다. (DOM에 리액트 라이프사이클과 이벤트 핸들러를 추가)
5. hydrate로 만든 리액트 컴포넌트 트리와 서버에서 만든 HTML이 다르면 불일치 에러를 뱉는다.
6. fetch등을 이용해 정보를 가져온다.

1번에서 가져온 정보를 HTML에 script형태로 내려주어 1 ~ 6 번 작업을 반복하지 않아도 되어 불필요한 요청을 막고 시점 차이로 인한 결과물의 차이도 막을 수 있다**. 즉, 6번에서 재용청하는 대신에 script 를 그대로 읽어서 동일한 데이터를 가져오는 것이다. 또한 window 객체에도 저장해 둔다.**

그리고 JSON으로만 props를 제공해주기에 JSON으로 직렬화할 수 없는 값(class, Date, 함수 등)은 제공이 불가능하다.

getServerSideProps는 매 페이지를 호출할때마다 실행되고, 이 실행이 끝나기 전까지는 사용자에게 어떠한 HTML도 보여줄 수 없기 때문에 **최대한 간결하게 작성**해야 한다.

```tsx
export const getSercerSideProps: GetServerSideProps = async (context) => {
	const {
		query: { id = '' },
	} = context;
	const post = await fetchPost(id.toString())

	**if (!post) {
		redirect: {
			destination: '/404'
		}
	}**

	return {
		props: { post },
	}
}
```

리다이렉트 처리를 하는것은 클리이언트에서 하는 것보다 서버에서 하는것이 더 자연스럽다.
클라이언트에서 처리할 경우 자바스크립트가 어느 정도 로딩 된 이후에 실행할 수밖에 없지만 `getServerSideProps`를 사용하면 해당 페이지를 바로 리다이렉트 처리를 해 자연스럽게 처리가 가능하다.

### getInitialProps

```tsx
import Link from "next/link";
import { NextPageContext } from "next";

export default function Todo({ todo }: { todo: { userId: number; id: number; title: string; completed: boolean } }) {
  return (
    <>
      <h1>{todo.title}</h1>
      <ul>
        <li>
          <Link href="/todo/1">1번</Link>
        </li>

        <li>
          <Link href="/todo/2">2번</Link>
        </li>

        <li>
          <Link href="/todo/3">3번</Link>
        </li>
      </ul>
    </>
  );
}

Todo.getInitialProps = async (ctx: NextPageContext) => {
  const {
    query: { id = "" },
    // asPath,
    // query,
    // res,
  } = ctx;
  const response = await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`);
  const result = await response.json();
  return { todo: result };
};
```

루트 함수에 정적 메서드를 추가하여 props 객체가 아닌 단순 객체를 반환한다는 점이 다르다.

`getInitialProps`는 라우팅에 따라서 **서버와 클라이언트 모두에서 실행 가능한 메서드**이다.

가급적이면 `getStaticProps`나 `getSercerSideProps`를 사용하는 것이 좋고, `getInitialProps` 는 \_app.tsx, \_error.tsx와 같이 제한돼 있는 페이지에서만 사용하는 것이 좋다.

## 스타일 적용하기

**전역 스타일**

CSS Reset, 브라우저에 기본으로 제공되는 스타일을 초기화하는 스타일의 경우 \_app.tsx를 활용하면 된다.

**컴포넌트 레벨 CSS**

**[name].module.css**와 같이 사용하면 컴포넌트 레벨 CSS를 사용할 수 있고, 다른 컴포넌트의 클래스명과 겹쳐서 스타일에 충돌이 일어나지 않도록 고유한 클래스명을 제공해준다.

```tsx
import styles from "./Button.module.css";

export function Button() {
  return (
    <button type="button" className={styles.alert}>
      경고!
    </button>
  );
}
```

alert 클래스를 입히더라도 스타일 충돌을 방지하기 위해 다른 클래스로 변경되어 스타일이 적용된다.

**SCSS와 SASS**

sass 패키지를 설치하여 별도의 설정 없이 사용이 가능하며 `variable`을 컴포넌트에서 사용하고 싶다면 `export`문법을 통해 사용이 가능하다.

```tsx
$primary: blue;

:export {
	primary: $primary;
}

import styles from "./Button.module.scss";

export Button() {
	return (
		<span style={{color: styles.primary }}>
			안녕하세요.
		</span>
	)
}
```

나머지는 컴포넌트 레벨 CSS와 동일하게 동작한다.

**CSS-in-JS**

[Next.js 공식문서](https://nextjs.org/docs/pages/building-your-application/routing/custom-document#customizing-renderpage)에도 나와 있듯이 랜더링을 커스텀할 수 있고, Styled-components를 사용하기 위해서는 별도의 설정이 필요하다.

```tsx
import Document, {
  Html,
  Head,
  Main,
  NextScript,
  DocumentContext,
  DocumentInitialProps,
} from 'next/document'

export default function MyDocument() {
  render() {
    return (
      <Html lang="en">
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}

MyDocument.getInitialProps = async (
    ctx: DocumentContext
  ): Promise<DocumentInitialProps> => {
		const sheet = new ServerStyleSheet()
    const originalRenderPage = ctx.renderPage

		try {

	    // Run the React rendering logic synchronously
	    ctx.renderPage = () =>
	      originalRenderPage({
	        // Useful for wrapping the whole react tree
	        enhanceApp: (App) => App,
	        // Useful for wrapping in a per-page basis
	        enhanceComponent: (Component) => Component,
	      })

	    // Run the parent `getInitialProps`, it now includes the custom `renderPage`
	    const initialProps = await Document.getInitialProps(ctx)

	    return {
        ...initialProps,
        styles: [
          <>
            {initialProps.styles}
            {sheet.getStyleElement()}
          </>,
        ],
      };
		} finally {
			sheet.seal();
		}
  }
```

- ServerStyleSheet은 styled-components의 스타일을 서버에 초기화해서 사용되는 클래스이다.
- ctx.renderPage에 기존에 해야하는 작업(originalRenderPage)와 추가적인 작업을 정의
- `const initialProps = await Document.getInitialProps(ctx)` 는 기존의 \_document.tsx의 수행에 필요한 `getInitialProps`를 생성하는 작업을 한다.
- 마지막으로 기존 props와 styled-component가 모아둔 자바스크립트 파일 내 스타일을 반환한다. 즉, \_document 렌더링될 때 styled-components의 스타일도 함께 내려줄 수 있게 된다.

만약 위의 작업을 하지 않는다면, 스타일이 브라우저에서 뒤늦게 추가되어 FOUC(flash of unstyled content) 가 발생해 생 HTML이 노출되게 된다. **CSS-in-JS를 사용한다면 모두 서버에서 스타일을 모아 서버 사이드 렌더링 시에 일괄 적용하는 과정을 거쳐야 한다.**

### \_app.tsx 응용하기

```tsx
import type { AppProps } from "next/app";

export default function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />;
}

App.getInitialProps = async (context: AppContext) => {
  const appProps = await App.getInitialProps(context);
  const isServer = Boolean(context.ctx.req);
  console.log(`[${isServer ? "서버" : "클라이언트"}] ${context.router.pathname}에서 ${context.ctx?.req?.url}를 요청함`);
  return appProps;
};

export default MyApp;
```

getInitialProps의 흥미로운 작동 방식이 숨겨져 있는데 라우팅을 반복하면 다음과 같다.

1. 가장 먼저 자체 페이지에 getInitialProps가 있는 곳을 방문
   - **[서버]** /test/GIP에서 /test/GIP를 요청
2. getServerSideProps가 있는 페이지를 <Link>를 이용해서 방문
   - **[서버]** /test/GSSP에서 /\_next/data/{hash}/test/GSSP.json을 요청
3. 다시 1번의 페이지를 <Link>를 이용해서 방문
   - **[클라이언트]** /test/GSSP에서 undefined를 요청
4. 다시 2번의 페이지를 <Link>를 이용해서 방문
   - **[서버]** /test/GSSP에서 /\_next/data/{hash}/test/GSSP.json을 요청

페이지 방문 최초 시점인 1번은 SSR이 전체적으로 작동해야 해서 페이지 전체를 요청하고 이후에는 클라이언트 라우팅을 수행하기 위해 `getServerSideProps`결과를 json파일만을 요청해서 가져온다.

**이러한 특성을 잘 활용하면 최초에 접근했을 때만 실행하고 싶은 내용을 담을 수 있다.**

```tsx
App.getInitialProps = async (context: AppContext) => {
	const {
		ctx: { req },
		router: { pathname },
	} = context

	if (req && !req.url?.startsWith('/_next') && !['/500', '/400', '/_error'].includes(pathname) {
		doSomethingOnlyOnce()
	}

	return appProps;
}
```

- req가 있다면 서버로 오는 요청이다
- req.url이 /\_next로 시작하지 않으면 클라이언트 렌더링으로 인해 발생한 getServerSideProps 요청이다.
- pathname이 에러페이지가 아니라면 정상적인 페이지 접근
