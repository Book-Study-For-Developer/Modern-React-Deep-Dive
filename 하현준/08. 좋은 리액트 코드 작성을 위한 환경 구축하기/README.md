# 8장 좋은 리액트 코드 작성을 위한 환경 구축하기

## ESLint를 활용한 정적 코드 분석

### ESLint 살펴보기

ESLint는 어떻게 자바스크립트 코드를 읽어서 분석하는 것일까?

-   자바스크립트 코드를 문자열로 읽는다.
-   자바스크립트 코들르 분석할 수 있는 파서로 코드를 구조화한다.
-   구조화한 트리를 AST(Abstract Syntax Tree)라 하며, 이 구조화된 트리를 기준으로 각종 규칙과 대조한다.
-   규칙과 대조했을 때 이를 위반한 코드를 알리거나 수정한다.

자바스크립트 파서 중에 ESLint는 기본적으로 `[espree](https://github.com/eslint/espree)`를 사용한다.

Parser가 동작하는 방식은 [여기서](https://denar90.github.io/eslint.github.io/parser/) 확인이 가능하다.

```tsx
{
  "type": "Program",
  "start": 0,
  "end": 22,
  "body": [
    {
      "type": "FunctionDeclaration",
      "start": 0,
      "end": 22,
      "id": {
        "type": "Identifier",
        "start": 9,
        "end": 14,
        "name": "hello"
      },
      "generator": false,
      "expression": false,
      "params": [
        {
          "type": "Identifier",
          "start": 15,
          "end": 18,
          "name": "str"
        }
      ],
      "body": {
        "type": "BlockStatement",
        "start": 20,
        "end": 22,
        "body": []
      }
    }
  ],
  "sourceType": "module"
}
```

타입스크립트를 사용할 경우엔 타입스크립트의 **타입 정보가 추가되게 된다**.

**[no-debugger에 대한 규칙](https://github.com/eslint/eslint/blob/main/lib/rules/no-debugger.js)**은 debugger 사용을 제한하는 규칙이다.

```tsx
/**
 * @fileoverview Rule to flag use of a debugger statement
 * @author Nicholas C. Zakas
 */

"use strict";

//------------------------------------------------------------------------------
// Rule Definition
//------------------------------------------------------------------------------

/** @type {import('../shared/types').Rule} */
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
            unexpected: "Unexpected 'debugger' statement.",
        },
    },

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

create가 실제 코드에서 문제점을 확인하는 곳이고, AST 트리를 순회하면서 특정 조건을 만족하는 코드를 확인해 node에 리보트하는 방식으로 작성되어 있다.

### eslint-plugin과 eslint-config

**eslint-plugin**

eslint-plugin은 규칙들을 모아놓은 패키지이다. 예를 들어 리액트 관련 규칙을 제공하는 `eslint-plugin-react`는 리액트에서 필요한 규칙들을 만들어서 제공해주는 것이다. (`[react/jsx-key](https://github.com/jsx-eslint/eslint-plugin-react/blob/master/lib/rules/jsx-key.js#L66)`와 같이 key를 안적으면 경고해주는 단순한 규칙이라도 create함수가 매우 방대)

**eslint-config**

특정 프레임워크나 도메인과 관련된 규칙을 묶어서 제공하는 패키지이다. 이러한 패키지중에 대표적인 것들이 몇개 있다.

`[eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb)`

이름 그대로 에어비엔비에서 만들었으며 에어비엔비의 개발자 뿐만 아니라 다른 개발자도 같이 참여하여 만든 eslint-config이다.

`[@titicaca/triple-config-kit](https://www.npmjs.com/package/@titicaca/eslint-config-triple)`

스타트업 개발사인 트리플에서 개발하고 있으며 다른 패키지와 다른 특징이 있다.
Prettier와 StyleLint도 같이 모노레포 형태로 개발되어 있어서 필요에 따라 설치해서 사용할 수 있고, 자체적인 eslint-config를 구축할때 도움이 된다.

`[eslint-config-next](https://www.npmjs.com/package/eslint-config-next)`

자바스크립트 코드를 정적으로 분석하는 것 뿐만 아니라 JSX, \_app, \_document에서 작성된 HTML코드, 핵심 웹 지표도 같이 분석해주는 기능이 포함되어 있다.

### 나만의 ESLint 규칙 만들기

조직 내부에서 필요하는 규칙 또는 코드를 일괄로 변화해야 할 때 나만의 ESLint규칙을 생성해서 빠르고 쉽게 수정할 수 있다.

**이미 존재하는 규칙을 커스터마이징해서 적용하기: import React를 제거하기 위한 ESLing 규칙 만들기**

리액트 17버전부터는 import React구문이 필요 없어졌다.

```tsx
// 이 컴포넌트를 사용하는 것이 100개가 있다고 가정
import React from "react";

export function Component1() {
    return <div>Hello World!</div>;
}
```

이를 남겨두는 것은 코드를 실행할때 크기 차이가 존재한다.

번들링을 진행할 때에는 크기가 똑같아지지만 그럼에도 제거하는 것이 좋다. → 트리쉐이킹이 일어날 때 해당 import 구문을 모두 확인하게 되므로 없애어 소요되는 시간을 줄일 수 있기 때문이다.

```tsx
module.exports = {
    rules: {
        "no-restricted-imports": [
            "error",
            {
                paths: [
                    {
                        name: "react",
                        importNames: ["default"],
                        message:
                            "import React from 'react'는 react 17부터 더 이상 필요하지 않습니다. 필요한 것만 react로부터 import해서 사용해주세요",
                    },
                ],
            },
        ],
    },
};
```

**완전히 새로운 규칙 만들기: new Date를 금지시키는 규칙**

new Date()는 기기에 종속된 현재 시간이어서 기기의 시간을 바꾸면 new Date()가 반환하는 시간 또한 변경된다.

-   new Date 대신 ServerDate를 만들어 이 함수만 사용이 가능
-   new Date(’2022-01-01’)같이 인자를 받는 것은 사용이 가능

```tsx
{
  "type": "Program",
  "start": 0,
  "end": 10,
  "body": [
    {
      "type": "ExpressionStatement",
      "start": 0,
      "end": 10,
      "expression": {
        "type": "NewExpression",
        "start": 0,
        "end": 10,
        "callee": {
          "type": "Identifier",
          "start": 4,
          "end": 8,
          "name": "Date"
        },
        "arguments": []
      }
    }
  ],
  "sourceType": "module"
}
```

-   `ExpressionStatement.expression`은 어떤 표현이 들어가있는지 확인
-   `ExpressionStatement.expression.type` 은 해당 표현이 어떤 타입인지 (생성자 이므로 NewExpression)
-   `ExpressionStatement.expression.callee` 은 생성자의 이름을 나타냄
-   `ExpressionStatement.expression.arguments` 은 생성자에 전달하는 인수

```tsx
/**
 * @fileoverview yceffort
 * @author yceffort
 */
"use strict";

// ------------------------------------------------------------------------------
// Rule Definition
// ------------------------------------------------------------------------------

/**
 *
 * @type {import('eslint').Rule.RuleModule}
 */
module.exports = {
    meta: {
        type: "suggestion",
        docs: {
            description: "disallow use of the new Date()",
            recommended: false,
        },
        fixable: "code",
        schema: [],
        messages: {
            message:
                "new Date()는 클라이언트에서 실행시 해당 기기의 시간에 의존적이라 정확하지 않습니다. 현재 시간이 필요하다면 ServerDate()를 사용해주세요.",
        },
    },
    create: function (context) {
        return {
            NewExpression: function (node) {
                if (
                    node.callee.name === "Date" &&
                    node.arguments.length === 0
                ) {
                    context.report({
                        node,
                        messageId: "message",
                        fix: function (fixer) {
                            return fixer.replaceText(node, "ServerDate()");
                        },
                    });
                }
            },
        };
    },
};
```

위의 조건을 만족하는 eslint-plugin을 직접 만들었다.

### 주의할 점

**Prettier와의 충돌**

Prettier는 코드의 포메팅을 도와주는 도구로 자바스크립트 뿐만 아니라 HTML, CSS등 다양한 언어에도 적용이 된다.

1. 서로 규칙이 충돌되지 않게 끔 규칙을 잘 선언하기
2. 자바스크립트나 타입스크립트는 ESLint에 그외는 Prettier에게 맡기기 (추가적으로 필요하면 `eslint-plugin-prettier`를 사용)

**규칙에 대한 예외 처리, 그리고 react-hooks/no-exhaustice-deps**

특정 규칙을 임시로 지우고 싶을 때 eslint-disable- 을 사용하면 된다.

리액트에서 가장 많이 사용하는 `eslint-dsiable-line no-exhaustive-deps` 가 있는데 이는 위험하고 잠재적인 버그를 발생시킬 수 있다.

-   괜찮다고 임의로 판단하는 경우: **가장 위험한 경우,** 해당 변수를 어디서 어떻게 선언할지 다시 고민해봐야 한다.
-   의존성 배열이 너무 긴 경우: 의존성 배열이 길다는 것은 useEffect 함수가 매우 크다는 것이고, 이는 쪼개서 관리해야 한다.
-   마운트 시점에 한 번만 실행하고 싶은 경우: 의도적으로 []로 모든 의존성을 제거해 마운트 시점에만 실행하는데, 이는 **함수형 컴포넌트 패러다임과는 맞지 않을 가능성이 있다.**
    빈 배열이 있다는 것은 컴포넌트의 상태값과는 별개의 부수 효과가 되어 상태 불일치가 일어날 수 있다. (다시 한번 생각해보고 적절한 위치로 옮겨라)

즉, 임시로 지우지 말고 정말 필요없다면 규칙을 없애는 것이 오히려 낫다.

## 리액트 팀이 권장하는 리액트 테스트 라이브러리

테스트란 **프로그램이 의도한대로 동작하는지 확인하는 일련의 작업을 말한다.**

### React Testing Library 란?

리액트 테스팅 라이브러리란 DOM Testing Library를 기반으로 만들어진 테스팅 라이브러리이다. DOM Testing Library는 jsdom을 기반으로 되어 있다.

jsdiom은 순수하게 자바스크립트로 작성된 라이브러리이다. 즉, 실제 리액트 컴포넌트를 렌더링하지 않고도, 확인할 수 있다.

### 자바스크립트 테스트의 기초

```tsx
function sum(a, b) {
    return a + b;
}

let actual = sum(1, 2);
let expected = 3;

if (expected !== actual) {
    throw new Error(`${expected} is not equal to ${actual}`);
}

actual = sum(2, 2);
expected = 4;

if (expected !== actual) {
    throw new Error(`${expected} is not equal to ${actual}`);
}
```

1. 테스트할 함수나 모듈을 선정한다.
2. 함수나 모듈이 반환하길 기대하는 값을 적는다.
3. 함수나 모듈의 실제 반환 값을 적는다.
4. 3번의 기대에 따라 2번의 결과와 일치하는지 확인한다.
5. 기대하는 결과를 반환한다면 테스트는 성공이며, 만약 기대와 다른 결과를 반환하면 에러를 던진다.

Node의 assert를 사용한 코드

```tsx
const assert = require("assert");

function sum(a, b) {
    return a + b;
}

assert.equal(sum(1, 2), 3);
```

좋은 테스트 코드는 다양한 테스트 코드가 작성되고 통과하는 것 뿐만 아니라 어떤 테스트가 무엇을 테스트하는지 일목요연하게 보여주는 것도 중요하다.

### 리액트 컴포넌트 테스트 코드 작성하기

리액트에서 컴포넌트 테스트는 다음과 같은 순서로 진행된다.

-   컴포넌트를 렌더링한다.
-   필요하다면 컴포넌트에서 특정 액션을 수행한다.
-   컴포넌트 렌더링과 2번 액션을 통해 기대환 결과와 실제 결과를 비교한다.

테스트 코드를 작성해보자 (원본 소스는 [여기서](https://github.com/wikibook/react-deep-dive-example/blob/main/chapter8/react-test/src/App.tsx) 확인이 가능하다.)

```tsx
// App.test.tsx
import { render, screen } from "@testing-library/react";

import App from "./App";

test("renders learn react link", () => {
    render(<App />);
    // 'learn react'라는 문자열을 가진 DOM을 찾는다.
    const linkElement = screen.getByText(/learn react/i);
    // linkElement요소가 document내부에 있는지 확인한다.
    expect(linkElement).toBeInTheDocument();
});

// App.tsx
import logo from "./logo.svg";
import "./App.css";
import StaticComponent from "./components/StaticComponent";

function App() {
    return (
        <div className="App">
            <header className="App-header">
                <img src={logo} className="App-logo" alt="logo" />
                <p>
                    Edit <code>src/App.tsx</code> and save to reload.
                </p>
                <a
                    className="App-link"
                    href="https://reactjs.org"
                    target="_blank"
                    rel="noopener noreferrer"
                >
                    Learn React
                </a>
            </header>
        </div>
    );
}

export default App;
```

-   `getBy__`: 인수의 조건에 맞는 요소를 반환하며, 없거나 두개 이상이면 에러를 발생시킴 → 복수개를 찾고 싶으면 `getAllBy__`
-   `findBy__`: Promise를 반환하며 비동기로 찾는다. 마찬가지로 복수개를 찾고 싶으면 `findAllBy__`를 사용하면 된다.
-   `queryBy__`: 인수의 조건에 맞는 요소를 반환하는 대신 찾지 못하면 null을 반환한다. 복수개를 찾고 싶을 땐 `queryAllBy__`를 사용한다.

\*.test.{t|s}jsx만 준수한다면 번들링에서 제외되므로 해당 파일에서 테스트 코드를 작성하면 된다.

**정적 컴포넌트**

[왼쪽](https://github.com/wikibook/react-deep-dive-example/blob/main/chapter8/react-test/src/components/StaticComponent/index.tsx)은 정적 컴포넌트 [오른쪽](https://github.com/wikibook/react-deep-dive-example/blob/main/chapter8/react-test/src/components/StaticComponent/index.test.tsx)은 테스트 코드이다.

```tsx
import { memo } from "react";

const AnchorTagComponent = memo(function AnchorTagComponent({
    name,
    href,
    targetBlank,
}: {
    name: string;
    href: string;
    targetBlank?: boolean;
}) {
    return (
        <a
            href={href}
            target={targetBlank ? "_blank" : undefined}
            rel="noopener noreferrer"
        >
            {name}
        </a>
    );
});

export default function StaticComponent() {
    return (
        <>
            <h1>Static Component</h1>
            <div>유용한 링크</div>

            <ul data-testid="ul" style={{ listStyleType: "square" }}>
                <li>
                    <AnchorTagComponent
                        targetBlank
                        name="리액트"
                        href="https://reactjs.org"
                    />
                </li>
                <li>
                    <AnchorTagComponent
                        targetBlank
                        name="네이버"
                        href="https://www.naver.com"
                    />
                </li>
                <li>
                    <AnchorTagComponent
                        name="블로그"
                        href="https://yceffort.kr"
                    />
                </li>
            </ul>
        </>
    );
}
```

```tsx
import { render, screen } from "@testing-library/react";

import StaticComponent from "./index";

beforeEach(() => {
    render(<StaticComponent />);
});

describe("링크 확인", () => {
    it("링크가 3개 존재한다.", () => {
        const ul = screen.getByTestId("ul");
        expect(ul.children.length).toBe(3);
    });

    it("링크 목록의 스타일이 square다.", () => {
        const ul = screen.getByTestId("ul");
        expect(ul).toHaveStyle("list-style-type: square;");
    });
});

describe("리액트 링크 테스트", () => {
    it("리액트 링크가 존재한다.", () => {
        const reactLink = screen.getByText("리액트");
        expect(reactLink).toBeVisible();
    });

    it("리액트 링크가 올바른 주소로 존재한다.", () => {
        const reactLink = screen.getByText("리액트");

        expect(reactLink.tagName).toEqual("A");
        expect(reactLink).toHaveAttribute("href", "https://reactjs.org");
    });
});

describe("네이버 링크 테스트", () => {
    it("네이버 링크가 존재한다.", () => {
        const naverLink = screen.getByText("네이버");
        expect(naverLink).toBeVisible();
    });

    it("네이버 링크가 올바른 주소로 존재한다.", () => {
        const naverLink = screen.getByText("네이버");

        expect(naverLink.tagName).toEqual("A");
        expect(naverLink).toHaveAttribute("href", "https://www.naver.com");
    });
});

describe("블로그 링크 테스트", () => {
    it("블로그 링크가 존재한다.", () => {
        const blogLink = screen.getByText("블로그");
        expect(blogLink).toBeVisible();
    });

    it("블로그 링크가 올바른 주소로 존재한다.", () => {
        const blogLink = screen.getByText("블로그");

        expect(blogLink.tagName).toEqual("A");
        expect(blogLink).toHaveAttribute("href", "https://yceffort.kr");
    });

    it("블로그는 같은 창에서 열려야 한다.", () => {
        const blogLink = screen.getByText("블로그");
        expect(blogLink).not.toHaveAttribute("target");
    });
});
```

-   beforeEach: 각 테스트(it)를 수행하기 전에 실행하는 함수
-   describe: 비슷한 속성을 가진 테스트를 하나의 그룹으로 묶는 역할을 한다.
-   it: test와 완전히 동일하며, test의 축약어이다. (이유는 사람이 읽기 쉽게 하기위해 describe … it)
-   testId: testId는 리액트 테스팅 라이브러리의 예약어, DOM 요소에 testId 데이터셋을 선언해 두면 테스트 시에 `getByTestId`, `findByTestId` 등으로 선택할 수 있다.
    querkySelector([data-testid=”${yourId}”])와 동일한 역할을 한다.

**동적 컴포넌트**

상태값을 관리하는 컴포넌트는 순수한 무상태 컴포넌트보다 테스트가 더 복잡하다.

```tsx
import { useState } from "react";

export function InputComponent() {
    const [text, setText] = useState("");

    function handleInputChange(event: React.ChangeEvent<HTMLInputElement>) {
        const rawValue = event.target.value;
        const value = rawValue.replace(/[^A-Za-z0-9]/gi, "");
        setText(value);
    }

    function handleButtonClick() {
        alert(text);
    }

    return (
        <>
            <label htmlFor="input">아이디를 입력하세요.</label>
            <input
                aria-label="input"
                id="input"
                value={text}
                onChange={handleInputChange}
                maxLength={20}
            />
            <button onClick={handleButtonClick} disabled={text.length === 0}>
                제출하기
            </button>
        </>
    );
}
```

-   input은 최대 20자까지, 한글 입력만 가능하도록 제한한다.
-   버튼은 글자가 있을때만 able처리가 된다.
-   버튼을 클릭하면 alert창을 띄운다.

```tsx
import { fireEvent, render } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

import { InputComponent } from ".";

describe("InputComponent 테스트", () => {
    const setup = () => {
        const screen = render(<InputComponent />);
        const input = screen.getByLabelText("input") as HTMLInputElement;
        const button = screen.getByText(/제출하기/i) as HTMLButtonElement;
        return {
            input,
            button,
            ...screen,
        };
    };

    it("input의 초기값은 빈 문자열이다.", () => {
        const { input } = setup();
        expect(input.value).toEqual("");
    });

    it("input의 최대길이가 20자로 설정되어 있다.", () => {
        const { input } = setup();
        expect(input).toHaveAttribute("maxlength", "20");
    });

    it("영문과 숫자만 입력된다.", () => {
        const { input } = setup();
        const inputValue = "안녕하세요123";
        userEvent.type(input, inputValue);
        expect(input.value).toEqual("123");
    });

    it("아이디를 입력하지 않으면 버튼이 활성화 되지 않는다.", () => {
        const { button } = setup();
        expect(button).toBeDisabled();
    });

    it("아이디를 입력하면 버튼이 활성화 된다.", () => {
        const { button, input } = setup();

        const inputValue = "helloworld";
        userEvent.type(input, inputValue);

        expect(input.value).toEqual(inputValue);
        expect(button).toBeEnabled();
    });

    it("버튼을 클릭하면 alert가 해당 아이디로 뜬다.", () => {
        const alertMock = jest
            .spyOn(window, "alert")
            .mockImplementation((_: string) => undefined);

        const { button, input } = setup();
        const inputValue = "helloworld";

        userEvent.type(input, inputValue);
        fireEvent.click(button);

        expect(alertMock).toHaveBeenCalledTimes(1);
        expect(alertMock).toHaveBeenCalledWith(inputValue);
    });
});
```

-   setup 함수: 모든 테스트에 필요한 컴포넌트를 렌더링한 뒤에 필요로 하는 값을 리턴해준다.
-   userEvent.type: 사용자가 타이핑하는 것을 흉내 내는 메소드이다.
-   jest.spyOn(window, ‘alert’).mockImplementation():
    -   jest.spyOn: 어떠한 특정 메서드를 오염시키지 않고 실행이 됐는지, 어떤 인수로 실행됐는지 등 **실행과 관련된 정보만 얻고 싶을 때 사용한다.**
    -   mockImplementation: 해당 메서드에 대한 모킹 구현을 도와준다. 모의 함수로 구현된 함수이지만 이 함수가 실행됐는지 등의 정보는 확인할 수 있도록 도와준다.

**비동기 이벤트가 발생하는 컴포넌트**

```tsx
import { MouseEvent, useState } from "react";

interface TodoResponse {
    userId: number;
    id: number;
    title: string;
    completed: false;
}

export function FetchComponent() {
    const [data, setData] = useState<TodoResponse | null>(null);
    const [error, setError] = useState<number | null>(null);

    async function handleButtonClick(e: MouseEvent<HTMLButtonElement>) {
        const id = e.currentTarget.dataset.id;

        const response = await fetch(`/todos/${id}`);

        if (response.ok) {
            const result: TodoResponse = await response.json();
            setData(result);
        } else {
            setError(response.status);
        }
    }

    return (
        <div>
            <p>{data === null ? "불러온 데이터가 없습니다." : data.title}</p>

            {error && (
                <p style={{ backgroundColor: "red" }}>에러가 발생했습니다</p>
            )}

            <ul>
                {Array.from({ length: 10 }).map((_, index) => {
                    const id = index + 1;
                    return (
                        <button
                            key={id}
                            data-id={id}
                            onClick={handleButtonClick}
                        >
                            {`${id}번`}
                        </button>
                    );
                })}
            </ul>
        </div>
    );
}
```

-   데이터를 불러오는 데 성공하면 응답값 중 하나를 노출하지만, 실패하면 에러 문구를 노출하는 컴포넌트이다.

```tsx
import { fireEvent, render, screen } from '@testing-library/react'
import { rest } from 'msw'
import { setupServer } from 'msw/node'

import { FetchComponent } from '.'

const MOCK_TODO_RESPONSE = {
  userId: 1,
  id: 1,
  title: 'delectus aut autem',
  completed: false,
}

const server = setupServer(
  rest.get('/todos/:id', (req, res, ctx) => {
    const todoId = req.params.id

    if (Number(todoId)) {
      return res(ctx.json({ ...MOCK_TODO_RESPONSE, id: Number(todoId) }))
    } else {
      return res(ctx.status(404))
    }
  }),
)

beforeAll(() => server.listen())
// 각 테스트가 진행될 때마다 서버를 종료시킨다.
**afterEach(() => server.resetHandlers());**
afterAll(() => server.close())

beforeEach(() => {
  render(<FetchComponent />)
})

describe('FetchComponent 테스트', () => {
  it('데이터를 불러오기 전에는 기본 문구가 뜬다.', async () => {
    const nowLoading = screen.getByText(/불러온 데이터가 없습니다./)
    expect(nowLoading).toBeInTheDocument()
  })

  it('버튼을 클릭하면 데이터를 불러온다.', async () => {
    const button = screen.getByRole('button', { name: /1번/ })
    fireEvent.click(button)

    const data = await screen.findByText(MOCK_TODO_RESPONSE.title)
    expect(data).toBeInTheDocument()
  })

// 서버에 실패한 경우 임의로 상태를 변경하므로 리셋이 필요하다.
  it('버튼을 클릭하고 서버요청에서 에러가 발생하면 에러문구를 노출한다.', async () => {
    server.use(
      rest.get('/todos/:id', (req, res, ctx) => {
        return res(ctx.status(503))
      }),
    )

    const button = screen.getByRole('button', { name: /1번/ })
    fireEvent.click(button)

    const error = await screen.findByText(/에러가 발생했습니다/)
    expect(error).toBeInTheDocument()
  })
})
```

fetch 요청을 하는 것과 동일하게 네트워크 요청을 수행하고, 중간에 MSW가 감지하여 미리 준비한 모킹 데이터를 제공하는 방식이다.

### 사용자 정의 훅 테스트하기

훅이 들어가 있는 컴포넌트를 만들 때 → 테스트 코드 작성 외에 작업이 더 추가된다는 점.

훅이 들어있는 컴포넌트일 때 → 훅이 모든 테스트 케이스를 커버하지 못할 경우 또 다른 테스트 가능한 컴포넌트를 찾아야 함

이를 해결 하기 위한 라이브러리 `[react-hooks-testing-library](https://www.npmjs.com/package/@testing-library/react-hooks)` 가 있다.

useEffectDebugger 훅은 컴포넌트명과 props를 인수로 받아 **해당 컴포넌트가 어떤 props의 변경으로 인해 리렌더링됐는지 확인해 주는 일종의 디버거 역할**을 한다.

-   최초 컴포넌트 렌더링 시에는 호출하지 않는다.
-   이전 props를 useRef에 저장해 두고, 새로운 props를 넘겨받을 때마다 이전 props와 비교해 무엇이 렌더링을 발생시켰는지 확인한다.
-   이전 props와 신규 props의 비교는 리액트의 원리와 동일하게 Object.is를 활용해 얕은 비교를 수행한다.
-   process.env.NODE_ENV === ‘production’ 인 경우에는 로깅을 하지 않는다.

```tsx
import { useEffect, useRef } from "react";

export type Props = Record<string, unknown>;

export const CONSOLE_PREFIX = "[useEffectDebugger]";

export default function useEffectDebugger(
    componentName: string,
    props?: Props
) {
    const prevProps = useRef<Props | undefined>();

    useEffect(() => {
        // process.env.NODE_ENV === ‘production’ 인 경우에는 로깅을 하지 않는다.
        if (process.env.NODE_ENV === "production") {
            return;
        }

        const prevPropsCurrent = prevProps.current;

        if (prevPropsCurrent !== undefined) {
            // 이전 props와 현재 props들의 모든 key를 가져온다.
            const allKeys = Object.keys({ ...prevProps.current, ...props });

            // 키들을 통해서 props를 얕은 비교를 통해 같은지 확인
            const changedProps: Props = allKeys.reduce<Props>((result, key) => {
                const prevValue = prevPropsCurrent[key];
                const currentValue = props ? props[key] : undefined;

                // 같지 않다면 전에 값과 현재값을 담은 result를 내보내줘 무엇이 렌더링 되었는지 확인시켜준다.
                if (!Object.is(prevValue, currentValue)) {
                    result[key] = {
                        before: prevValue,
                        after: currentValue,
                    };
                }
                return result;
            }, {});

            if (Object.keys(changedProps).length) {
                // eslint-disable-next-line no-console
                console.log(CONSOLE_PREFIX, componentName, changedProps);
            }
        }

        prevProps.current = props;
    });
}
```

이 훅은 props가 변경되는 것만 확인할 수 있고, 부모 컴포넌트의 리렌더링으로 바뀌는 경우에는 확인이 불가능 하다.

useEffectDebugger훅을 사용한 테스트 코드

```tsx
import { renderHook } from "@testing-library/react";

import useEffectDebugger, { CONSOLE_PREFIX } from "./useEffectDebugger";

const consoleSpy = jest.spyOn(console, "log");
const componentName = "TestComponent";

describe("useEffectDebugger", () => {
    afterAll(() => {
        // eslint-disable-next-line @typescript-eslint/ban-ts-comment
        // @ts-ignore
        process.env.NODE_ENV = "development";
    });

    it("props가 없으면 호출되지 않는다.", () => {
        renderHook(() => useEffectDebugger(componentName));

        expect(consoleSpy).not.toHaveBeenCalled();
    });

    it("최초에는 호출되지 않는다.", () => {
        const props = { hello: "world" };

        renderHook(() => useEffectDebugger(componentName, props));

        expect(consoleSpy).not.toHaveBeenCalled();
    });

    it("props가 변경되지 않으면 호출되지 않는다.", () => {
        const props = { hello: "world" };

        const { rerender } = renderHook(() =>
            useEffectDebugger(componentName, props)
        );

        expect(consoleSpy).not.toHaveBeenCalled();

        rerender();

        expect(consoleSpy).not.toHaveBeenCalled();
    });

    it("props가 변경되면 다시 호출한다.", () => {
        const props = { hello: "world" };

        const { rerender } = renderHook(
            ({ componentName, props }) =>
                useEffectDebugger(componentName, props),
            {
                initialProps: {
                    componentName,
                    props,
                },
            }
        );

        const newProps = { hello: "world2" };

        rerender({ componentName, props: newProps });

        expect(consoleSpy).toHaveBeenCalled();
    });

    it("props가 변경되면 변경된 props를 정확히 출력한다", () => {
        const props = { hello: "world" };

        const { rerender } = renderHook(
            ({ componentName, props }) =>
                useEffectDebugger(componentName, props),
            {
                initialProps: {
                    componentName,
                    props,
                },
            }
        );

        const newProps = { hello: "world2" };

        rerender({ componentName, props: newProps });

        expect(consoleSpy).toHaveBeenCalledWith(
            CONSOLE_PREFIX,
            "TestComponent",
            {
                hello: { after: "world2", before: "world" },
            }
        );
    });

    it("객체는 참조가 다르다면 변경된 것으로 간주한다", () => {
        const props = { hello: { hello: "world" } };
        const newProps = { hello: { hello: "world" } };

        const { rerender } = renderHook(
            ({ componentName, props }) =>
                useEffectDebugger(componentName, props),
            {
                initialProps: {
                    componentName,
                    props,
                },
            }
        );

        rerender({ componentName, props: newProps });

        // 이후 호출
        expect(consoleSpy).toHaveBeenCalled();
    });

    it("process.env.NODE_ENV가 production이면 호출되지 않는다", () => {
        // eslint-disable-next-line @typescript-eslint/ban-ts-comment
        // @ts-ignore
        // 환경변수 값 강제로 주입
        process.env.NODE_ENV = "production";

        const props = { hello: "world" };

        const { rerender } = renderHook(
            ({ componentName, props }) =>
                useEffectDebugger(componentName, props),
            {
                initialProps: {
                    componentName,
                    props,
                },
            }
        );

        const newProps = { hello: "world2" };

        rerender({ componentName, props: newProps });

        expect(consoleSpy).not.toHaveBeenCalled();
    });
});
```

**use**로 시작하는 사용자 정의 훅임에도 훅의 규칙을 위반하는 경고메시지를 출력하지 않는다.

그 이유는 `renderHook` 함수 내부에서 컴포넌트를 생성하고 그 컴포넌트 내에서 훅을 실행하기 때문이다.

### 그 밖에 해볼 만한 여러 가지 테스트

-   유닛 테스트(Unit Test): 각각의 코드나 컴포넌트가 독립적으로 분리된 환경에서 의도된 대로 정확히 작동하는지 검증하는 테스트
-   통합 테스트(Integration Test): 유닛 테스트를 통과한 여러 컴포넌트가 묶여서 하나의 기능으로 정상적으로 작동하는지 확인하는 테스트
-   엔드 투 엔드 테스트(End to End Test): 흔히 E2E 테스트라하며, 실제 사용자처럼 작동하는 로봇을 활용해 애플리케이션의 전체적인 기능을 확인하는 테스트

애플리케이션이 **비지니스 요구사항을 충족하는지 확인**하는 것을 위해 테스트를 진행하는 것이다. 조금씩 핵심적인 부분부터 테스트 코드를 작성해보자
