# 8장 좋은 리액트 코드 작성을 위한 환경 구축하기

버그를 방지하기 위해 여러 방법이 있는데. **정적 코드 분석**은 코드의 실행과 별개로 코드 스멜을 찾고 사전에 코드를 수정할 수 있게 한다. 자바스크립트 생태계에서 가장 많이 쓰이는 정적 코드 분석 도구인 **ESLint** 를 알아보자

## 8.1 ESLint 를 활용한 정적코드 분석

### 8.1.1 ESLint 살펴보기

ESLint는 코드를 이렇게 분석한다.

1. 자바스크립트 코드를 문자열로 읽는다.
2. 자바스크립트 코드를 분석할 수 있는 파서(parser) 로 코드를 구조화한다.
3. 2번에서 구조화한 트리를 AST(Abstract Syntax Tree)라 하며 이 구조화 된 트리를 기준으로 각종 규칙과 대조한다.
4. 규칙과 대조했을 때 위반한 코드를 알리거나 수정한다.

그렇다면 어떻게 자바스크립트를 파싱하는 것일까? [AST explorer](https://astexplorer.net/) 에서 파서를 설정하고 결과를 확인할 수 있다.

- ESLint 는 기본적으로 [espree](https://github.com/eslint/espree) 파서를 사용한다.
- espree 파서는 코드의 정확한 위치와 세세한 정보도 분석해 알려주기 때문에, ESLint 나 Prettier 같은 도구가 코드의 줄바꿈 들여쓰기 등을 파악할 수있게 한다.
- 타입스크립트의 경우에도 @typescript-eslint/typescript-estree espree 기반 파서가 있고 타입스크립트 코드를 분석해 구조화한다.

espree 로 간단한 자바스크립트 코드를 변환해보자

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/136b4da4-78ed-4060-a159-c317e2852dd1/5137f681-a6f8-4c92-ab29-4d0ba0800c63/Untitled.png)

eslint 의 debugger 사용을 제한하는 규칙 no-debugger 코드를 살펴보면

```jsx
module.exports = {
  meta: {
    type: "problem",

    docs: {
      description: "Disallow the use of `debugger`",
      recommended: true,
      url: "https://eslint.org/docs/latest/rules/no-debugger",
    },

    fixable: null,
    schema: [],

    messages: {
      // messages : 규칙을 어겼을 떄 반환하는 경고 문구
      unexpected: "Unexpected 'debugger' statement.",
    },
  },

  // create 함수 : 실제 코드에서 문제점을 확인하는 부분
  create(context) {
    return {
      DebuggerStatement(node) {
        context.report({
          node,
          messageId: "unexpected",
        });
      },
    };
  },
};
```

- espree로 만들어진 트리를 순회해서 특정 조건(create)을 만족하는 코드를 찾고, 이것을 코드전체에 반복한다.
- eslint 에서 기본적으로 제공하는 자바스크립트 관련 규칙이 있으며 타입스크립트, 리액트 등 생태계를 둘러싼 다양한 규칙이 있다.

### 8.1.2 eslint-plugin 과 eslint-config

eslint-plugin 과 eslint-config 는 모두 ESLint 와 관련된 패키지 인데 각각의 역할이 다르다.

eslint-plugin 과 eslint-config 는 한단어만 뒤에 붙일 수 있다는 네이밍 규칙을 가지고 있다.

eslint-plugin-{…}

특정 프레임워크나 도메인과 관련된 규칙을 묶어서 제공하는 패키지

- eslint-plugin-import : JS 에서 다른 모듈을 불러오는 import 와 관련된 다양한 규칙제공
- eslint-plugin-react: JSX 배열에 키를 선언하지 않았어! 와 같이 리액트 관련 규칙들을 가지고 있다.

eslint-config-{…}

내가 원하는 규칙들을 모으고 plugin 을 설치하고 적용하는 것도 만만치가 않기에 이를 묶어서 세트로 제공하는 패키지인 eslint-config 를 설치하면 된다.

- [eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb) : 에어비앤비 개발자 뿐 아니라 500여명의 수많은 개발자들이 유지보수하는 eslint-config
- [@titicaca/triple-config-kit](https://www.npmjs.com/package/@titicaca/eslint-config-triple)
  - 한국 커뮤니티에서 운영되는 eslint-config 중 유지보수가 활발함.
  - eslint-config-airbnb 를 기반으로 하고 있지 않음에도
  - 규칙에 대한 테스트 코드 존재해서 바르게 규칙이 추가되었는지 확인이 가능
  - 그외의 Prettier, Stylelint도 모노레포로 관리해 필요에 따라 다운이 가능하다.
- [eslint-config-next](https://www.npmjs.com/package/eslint-config-next)
  - Next.js 프레임워크 사용하는 프로젝트라면 설치를 권장 !
  - 페이지나 컴포넌트에서 반환하는 JSX 구문 및 \_app, \_document 에서 작성되어 있는 HTML 코드 또한 정적 분석 대상으로 분류해 제공
  - Next.js 기반 웹서비스의 성능향상에 도움 및 웹지표( 웹 서비스 성능 ) 에 영향을 미치는 요소를 분석해 제공하는 기능도 포함

### 8.1.3 나만의 ESLint 규칙 만들기

eslint-config 나 eslint-plugin 에서 제공하고 있지 않지만 조직에서 일괄적으로 고쳐야하는 코드가 있을 수 있는데, 찾아 바꾸기나 pr 을 일일히 확인하는 것은 비효율적이고 실수할 수 있다. 이때 eslint 규칙을 커스터마이징 하거나, 새롭게 추가해보자 !

**이미 존재하는 규칙을 커스터마이징 해서 적용하기**

- 시나리오 : 리액트 17버전 부터 새로운 JSX런타임 덕분에 import React 가 필요하지 않다.
  - import React 가 있는 파일과 없는 파일은 dev server 로 가동한 후 bundle.js를 비교했을 때 크기 차이가 있다. → build 하면 웹팩의 트리쉐이킹이 사용되지 않는 코드를 모두 제거하는데? → 트리쉐이킹에 소요되는 시간을 줄일 수 있으니까 !!
- 기존 규칙 no-restricted-imports 커스터마이징 해보기
  - 금지시킬 모듈과 import 형식, 경고 메세지를 등록해주면 끝!!
  - https://eslint.org/docs/latest/rules/no-restricted-imports#rule-details
  - eslint document 에서 playground를 통해 예제의 오류를 직접 확인 가능하다.
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/136b4da4-78ed-4060-a159-c317e2852dd1/f80201f5-f615-40f9-a5f6-5161d806aadc/Untitled.png)
- 적용할 수 있는 다른 케이스 : lodash 같은 경우 CommonJS 로 작성되어 트리쉐이킹이 안된다고 한다. 따라서 이부분을 위의 규칙으로 막아줄 수 있겠다.

완전히 새로운 규칙 만들기 : new Date 를 금지시키는 규칙

- 시나리오 : new Date() 는 기기의 시간에 종속되어있어서 한국시간을 반환하는 서버의 시간을 사용해야하는 경우 new Date() 의 사용을 막아야한다.
- 규칙 작성과정
  1. 먼저 new Date() 코드를 espree 에서 어떻게 파싱하는 지 살펴보자
     - type: NewExpression
     - callee.name: Date
     - ExpressionStatement.expression.arguments : []
  2. 위의 조건을 만족하는 aspree 노드가 있다면 경고를 주도록 create 함수를 작성하면 된다.
  3. fix 를 키로하는 함수를 작성해서 자동으로 수정하는 코드를 넣어줄 수 있다. 여기에 ServerDate() 로 교체해주는 함수를 넣어주면 된다.

eslint-plugin 배포해보기

- 규칙은 묶음으로 배포가 가능하다.
- yo + generate-eslint 를 활용해서 eslint-plugin을 구성할 환경을 구성하자
- rules 폴더에 규칙 코드를 작성하자
- tests 폴더에 규칙을 테스트하는 코드를 작성하자
- npm publish 로 배포해서 원하는 곳에 사용하자

### 8.1.4 주의할점

prettier 와의 충돌

- Prettier : HTML, CSS, 마크다운, JSON 등 다양한 언어에서 코드의 포맷팅을 도와줌
- ESLint : 자바스크립트에서 작동하며 코드의 잠재적 문제가능성을 분석

이 둘이 충돌할 때 해결방법

1. Prettier 에서 제공하는 규칙을 어기지 않게 ESLint 에서는 규칙을 끄기

   → 이 경우에 ESLint 를 적용하는 작업과 코드의 포매팅을 서로 다른 패키지에서 발생시킴

2. 자바스크립트 타입스크립트에는 ESLint 를 그외는 Prettier 에 맡기는것

   → 자바스크립트에 필요한 prettier 의 규칙은 eslint-plugin-prettier 로 적용해주면 된다.

규칙에 대한 예외 처리, 그리고 react-hooks/no-exhaustive-deps

- 특정 줄만 제거하거나, 파일전체를 제외하거나, 범위에 걸쳐 제외시킨다.
- 리액트 개발자들이 의존성 배열에 `eslint-disable-line no-exhaustive-deps` 같은 규칙을 많이 사용하는데, 의존성 배열이 너무 길어지거나 빈배열을 넣을때 발생하는 경고를 무시하기 위함이다. 그러나 규칙은 존재하는 이유가 있기 때문에 다시 한번 질문을 해보고, 필요없는 규칙은 off 해서 끄는 것이 좋다.
  - 의존성 배열이 너무 긴경우 : useEffect 를 쪼개서 가독성과 안정성을 확보해야한다.
  - 마운트 시점에 한번 실행하고 싶은 경우 : 컴포넌트의 상태값과 별개의 부수 효과가 되어 상태와 불일치가 일어날 수 있다.

ESLint 버전 충돌

패키지의 ESLint 버전이 app 의 ESLint 버전과 다른 경우에 충돌이 나서 오류가 발생할 수 있다. 이때 peerDependencies 로 명시해서 충돌을 방지 해야한다.

### 8.1.5 정리

- 이미 ESLint 를 잘쓰고 있는 개발자도 한번 현재 설치되어있는 eslint-config 가 무엇인지, 어떤 규칙을 가지는지 살펴보면 좋다. 모든 규칙에는 이유가 있고, 그 이유때문에 개발자들에게 인정받고 설치가 되는 것이다 .
- 조직의 eslint 규칙을 모아서 eslint-config 를 제작하고 조직을 탄탄히 다지는 도구로 사용될 수 있다.

## 8.2 리액트 팀이 권장하는 리액트 테스트 라이브러리

테스트는 프로그램이 코딩을 한 의도대로 작동하는 지 확인하는 일련의 작업

- 테스트의 목적
  - 처음 설계하는 대로 프로그램 작동하는 지 확인
  - 버그 사전 방지
  - 잘못된 작동에 의한 비용 절감
  - 수정 후에도 예외케이스 없이 잘 작동하는 지 확인
- 백앤드 테스트
  - 화이트 박스 테스트 , 주로 AUI( Application User Interface )에서 주로 수행
  - 서버나 DB 에서 원하는 데이터를 올바르게 가져오는지
  - 데이터 수정 간 교착 상태내 경쟁이 발생하지 않는지
  - 데이터 손실이 없는지
  - 특정 상황에서 장애가 발생하지 않는지
- 프론트앤드 테스트
  - 블랙박스 형태, 의도대로 동작되는 지 확인
  - 비즈니스 로직이나 모든 경우의 수를 고려해야 한다.
    - 테스팅 라이브러리도 다양하다. 유닛테스트 , e2e 테스트 등 …

### 8.2.1 React Testing Library란?

- 리액트 테스팅 라이브러리는 DOM Testing Library 를 기반으로 한다.
  - DOM Testing Library : Node.js 같은 환경에서도 HTML 과 DOM 을 사용할 수 있게 해준다.
    - 실제 리액트 컴포넌트를 브라우저에서 랜더링하지 않고도 리액트 컴포넌트가 원하는 대로 랜더링 되는 지 확인할 수 있다.
- 컴포넌트 뿐아니라 Provider, 훅 등 리액트를 구성하는 다양한 요소들을 테스트할 수 있다.

### 8.2.2 자바스크립트 테스트의 기초

- 기본적인 테스트 코드를 작성하는 방식
  1. 테스트할 함수나 모듈을 선정
  2. 함수나 모듈이 반환하길 기대하는 값을 적는다
  3. 함수나 모듈의 실제 반환 값을 적는다
  4. 3번의 기대에 따라 2번이 일치하는지 확인한다.
  5. 기대하는 결과이면 테스트는 성공이고, 기대와 다르면 에러가 발생한다.
- 어설션 라이브러리 : 테스트 결과를 확인할 수 있게 도와주는 라이브러리
  - Node.js 에는 기본적으로 assert 를 지원
  - should.js, expect.js, chai 등 다양한 라이브러리
- 테스팅 프레임워크 : 어설션 라이브러리를 내장해 테스트를 수행, 코드 작성자에게 정보도 제공한다
  - 좋은 테스트 코드는 다양한 코드가 작성되고 통과되는 것 뿐 아니라 테스트가 무엇을 하는지 일목요연하게 보여주는 것도 중요한데, 그 역할을 한다.
  - 종류 : Jest,Mocha,Karma,Jasmine 등 …
    - Jest : 메타에서 만들어 널리 쓰임, 오픈소스 재단으로 관리 주체 바뀜
      - 실행 시 전역스코프에 기본값을 넣어줌으로 import 하지않고 test, expect 등의 함수를 사용할 수 있다.

### 8.2.3 리액트 컴포넌트 테스트 코드 작성하기

컴포넌트 테스트 순서

1. 컴포넌트 랜더링하기
2. 컴포넌트에서 특정 액션 수행하기
3. 컴포넌트 랜더링과 2번 액션을 통해 기대하는 결과와 실제 결과 비교

일반적인 시나리오

- 원하는 값,속성 지닌 HTML 요소가 있는 지
- 방법
  - getBy : 인수의 조건에 맞는 요소를 반환한다. 조건에 맞는게 없거나 여러개면 에러, 복수개를 찾을 때는 getAllBy 사용
  - findBy : getBy와 유사하나 비동기로 찾음 . 복수개 가능
  - queryBy: 인수의 조건에 맞지않으면 null, 여러개면 에러

테스트파일 위치

- 같은 디렉토리에서 `*.test.{name}.(jsx|tsx)`를 준수하면 된다.

정적 컴포넌트 테스트

- 별도의 상태가 존재하지 않아 항상 같은 결과를 반환하기 때문에, 쉽다
- 원하는 컴포넌트 랜더링 > 테스트 원하는 요소를 찾아 테스트 수행하기

```jsx
// LinkComponent.js
import React from "react";

const LinkComponent = () => {
  const links = [
    { id: 1, href: "https://example.com", text: "Link 1" },
    { id: 2, href: "https://example.org", text: "Link 2" },
    { id: 3, href: "https://example.net", text: "Link 3" },
  ];

  return (
    <div data-testid="link-component">
      {links.map((link) => (
        <a key={link.id} href={link.href} data-testid={`link${link.id}`}>
          {link.text}
        </a>
      ))}
    </div>
  );
};

export default LinkComponent;
```

```jsx
// LinkComponent.test.js
import React from "react";
import { render, cleanup, getByTestId } from "@testing-library/react";
import LinkComponent from "./LinkComponent";

// 각각의 테스트 전에 실행되는 코드
beforeEach(() => {
  // render 함수를 사용하여 컴포넌트를 렌더하고, 결과를 변수에 저장합니다.
  render(<LinkComponent />);
});

// 여러 테스트를 그룹화하는 describe 블록
describe("LinkComponent Tests", () => {
  // 특정 테스트 케이스를 설명하는 it 블록
  it("renders LinkComponent correctly", () => {
    // getByTestId 함수를 사용하여 특정 data-testid 속성을 가진 엘리먼트를 찾습니다.
    const linkComponentElement = getByTestId("link-component");

    // 특정 엘리먼트에 대한 검증을 수행합니다.
    expect(linkComponentElement).toBeInTheDocument();
  });

  it("contains three links with correct hrefs", () => {
    // getByTestId 함수를 사용하여 각 링크 엘리먼트를 찾습니다.
    const link1Element = getByTestId("link1");
    const link2Element = getByTestId("link2");
    const link3Element = getByTestId("link3");

    // 각 링크에 대한 href 속성을 확인합니다.
    expect(link1Element).toHaveAttribute("href", "https://example.com");
    expect(link2Element).toHaveAttribute("href", "https://example.org");
    expect(link3Element).toHaveAttribute("href", "https://example.net");
  });
});
```

동적 컴포넌트 테스트

- 사용자 액션이 있는 동적인 컴포넌트를 테스트하는 방법 이다.
- 사용자 작동을 재현한 뒤에 DOM에 기댓값이 반영됐는지 확인한다.
  - 사용자 동작을 흉내내기 위해 userEvent 의 메소드를 사용
  - jest.spyOn 에 함수를 등록하면, 모킹함수를 만들고 이 함수에 대한 실행정보를 알 수 있다.

비동기 컴포넌트 테스트

fetch가 실행되는 컴포넌트를 예로 비동기 컴포넌트 테스트 방법을 알아보자

- 간단하게 Promise를 활용해서 응답을 반환하는 모킹함수를 만들 수 있다. → 모든 시나리오를 커버하지 못한다.
- MSW(Mock Service Worker) : Node.js 나 브라우저에서 모두 사용가능 한 모킹 라이브러리, https 나 XMLHttp Request 요청을 가로채는 방식으로 작동한다. 즉 fetch 의 모든 기능을 그대로 사용하면서 응답을 모킹하는 것이다.

  ```jsx
  // setupServer 로 요청을 가로채서 모킹데이터를 반환하게 한다
  const server = setupServer(
    rest.get("/todos/:id", (req, res, ctx) => {
      const todoId = req.params.id;
      // 경우에 따른 응답 ...
    })
  );

  //서버를 가동하고 각 테스트마다 서버 기본설정으로 리셋하고, 전체 테스트가 끝나면 서버를 종료한다
  beforeAll(() => server.listen());
  afterEach(() => server.resetHandlers());
  afterAll(() => server.close());

  //setupServer 에서 정상적인 응답 모킹을 했고 실제 사용시 server.use() 를 활용해서 서버기본 작동을 덮어쓰게 할 수 있다.

  it("서버 요청시 에러 발생하면 에러 문구 노출", async () => {
    server.use(
      rest.get("/todos/:id", (req, res, ctx) => {
        return res(ctx.status(503));
      })
    );
  });
  ```

### 8.2.4 사용자 정의 훅 테스트하기

- 사용자 훅을 테스트 하려면 ? 훅이 포함된 컴포넌트를 테스트 하면될까 ? 그렇게 되면 모든 테스트 케이스를 커버하지 못할 수 있다. → react-hooks-testing-library
- react-hooks-testing-library 사용하기
  - 리액트 18 부터는 @testing-library/react 에 통합됨
- 예시 : useEffectDebugger 라는 컴포넌트의 props 를 console 을 찍어주는 커스텀 훅이 있다고 하자.

  ```jsx
  // ❓ 523 pg process.env.NODE_ENV 를 development 로 변경하는 게 타입스크립트 읽기 전용 속성으로 간주하기 때문이라고 했는데,

  describe("useEffectDebugger", () => {
    afterAll(() => {
      //eslint-diable-nextline @typescript-eslint/ban-ts-comment
      //@ts-ignore
      process.env.NODE_ENV = "development";
    });

    it("props 가 없으면 호출되지 않는다", () => {
      // 훅을 랜더링 하기 위해 renderHook 사용
      renderHook(() => useEffectDebugger(componentName));
      expect(consoleSpy).not.toHaveBeenCalled();
    });
  });
  ```

  - 훅의 규칙을 위반 않기 위해 renderHook 내부에 컴포넌트를 만들어 훅의 규칙을 위반하지 않는다.
    - 이 때문에 훅을 두번 실행하려면 renderHook 이 반환하는 unmount 함수를 호출해서 기존 훅을 테스트하기 위해 만들어진 컴포넌트를 제거해야한다.

### 8.2.5 테스트를 작성하기에 앞서 고려해야 할 점

- TDD 로 테스트를 우선 시 할지라도 테스트 커버리지를 100% 하기는 어렵다. 어플리케이션의 취약점과 중요 부분을 파악하는 것이 중요하다 .
- 어플리케이션의 기능 중 결제는 중요한 부분일 텐데, 이 부분을 테스트하기 위해서는, 사용자의 작업과 최대한 유사하게 작업 되어야한다.
  - 입력 > 장바구니 > 주소입력 > 결제 모두 사용자와 최대한 비슷한 입장에서 테스트를 작성하는 것이 필요하다.
- 소프트웨어 품질에 대한 확신을 얻기 위해 작성하는 것

### 8.2.6 그밖에 해볼 만한 여러 가지 테스트

- 유닛테스트 : 각각의 코드나 컴포넌트가 독립적으로 분리된 환경에서 정확히 작동하는지 검증
- 통합테스트 : 유닛 테스트를 통과한 여러 컴포넌트가 묶어서 테스트
- 엔드 투 엔드 테스트 : 실제 사용자 처럼 작동하는 로봇이 전체적인 기능을 확인
  - Cypress
- 유닛 > 통합 > E2E 로 갈수록 테스트 실패지점이 많고 테스트 코드도 복잡하지만, 코드에 대한 자신감의 가능성 또한 커집

### 8.2.7 정리

- 애플리케이션이 비즈니스 요구사항을 충족하는지 확인
- 핵심적인 부분 부터 테스트 코드를 작성하며 소프트 웨어의 품질에 대한 확신 가지기
