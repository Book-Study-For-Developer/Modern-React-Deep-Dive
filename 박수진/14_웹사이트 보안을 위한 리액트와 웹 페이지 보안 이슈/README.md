# 웹사이트 보안을 위한 리액트와 웹페이지 보안 이슈

본 챕터에서은 프론트앤드 개발자가 신경써야할 다양한 보안 이슈를 살펴본다.

## 14.1 리액트에서 발생하는 크로스 사이트 스크립팅(XSS)

**크로스 사이트 스크립팅( Cross-Site Scripting Xss )**
웹 사이트 개발자가 아닌 제 3자가 웹사이트에 악성 스크립트를 삽입해 실행할 수 있는 취약점
스크립트가 실행되는 곳이라면 개발자 처럼 작업을 수행할 수 있으며, 쿠키를 획득해 사용자의 로그인 세션을 탈취하거나 사용자의 데이터를 변경하는 등 각종 위험성이 있다.

**리액트에서 발생할 수 있는 경우**

- dangerouslySetInnerHTML prop
  - 브라우저 DOM에 직접 innerHTML 을 사용자의 입력 등으로 교체를 할 수 있는데, 인수로 받는 문자열에 제한이 없다.
    ```tsx
    const html = `<span><svg/onload=alert(origin)</span>`;
    ```
    다음과 같이 origin 을 alert 로 볼 수 도 있다.
- useRef를 활용한 직접 삽입
  - 이 또한 useRef를 이용해서 DOM 에 직접 접근해서 innerHTML 을 교체하면 의도치않은 화면과 동작을 실행시킬 수 있다.
- \<a> 에 잘못된 href를 삽입하거나 onclick, onload 등 이벤트를 활용하는 등 여러가지 방식의 XSS 가 있지만, 공통점은 웹사이트 개발자가 만들지 않은 코드를 삽입한다는 것이다.

**리액트에서 XSS 문제 피하기**

- 제 3자가 삽입할 수 있는 HTML을 안전한 코드로 한번 치환하는 것
  - 이를 sanitize, escape 라고 하며 DOMpurity, sanitize-html, js-xss 와 같은 라이브러리가 있다.
  - sanitize-html
    - https://www.npmjs.com/package/sanitize-html
    - 위험 요소를 차단하고, 허용할 목록을 작성하는 방식
- 사용자가 콘텐츠를 저장할 때도 이스케이프 과정을 거쳐서 DB에 XSS 위험성이 있는 콘텐츠를 저장될 수 없게 하는 것이 좋다.
  - 이런 과정은 서버에서 실행하면 좋다
- 게시판 같은 예시가 아니라도 쿼리스트링에 있는 내용을 그대로 실행 시, 보안 취약점이 발생할 수 있다.
- 리액트에는 기본적인 이스케이프 작업이 되고 있어서 dangerouslySetInnerHTML 속성이 별도로 존재한다.

## 14.2 getServerSideProps 와 서버 컴포넌트를 주의하자

서버사이드 랜더링과 서버 컴포넌트는 서버라는 개발환경을 프런트앤드 개발자에게 접근가능하게 하는데, 서버에는 일반 사용자에게 노출되면 안되는 정보들이 있어서 브라우저에 정보를 내려줄 때 조심해야한다.

- getServerSideProps가 반환하는 값 또는 서버 컴포넌트에 반환하는 props 는 필요한 값으로 제한 되어야한다
  - getServerSideProps 가 반환하는 props 의 값은 모두 사용자의 HTM에 기록되고,전역변수에 등록되어 스크립트로 충분히 접근할 수 있는 보안 위협에 노출되는 값이 된다.
- 예를 들어 getServerSideProps 에서 쿠키 값을 통째로 반환하지 않고, 클라이언트에서 필요한 token 값만 제한적으로 반환하는 식으로 하는 것이다.
  ```tsx
  ❗️ 예시코드 작성예정
  ```

## 14.3 \<a> 태그의 값에 적절한 제한을 둬야한다

```tsx
function App() {
    return <><a href="javascript:alert('hello')" ></>
}
```

다음과 같이 \<a> 에 자바스크립트가 작동하는 것을 볼 수 있는데, 사용자가 입력한 주소를 넣을 수 있다면 보안이슈로 이어진다. 따라서 `href={isSafeHref(userHref)}` 이렇게 한번 제한해주는 것이 좋다.

## 14.4 HTTP 보안 헤더 설정하기

HTTP 보안 헤더란 브라우저가 랜더링하는 내용과 관련된 보안 취약점을 미연에 방지하기 위해 브라우저와 함께 작동하는 헤더

**보안해더의 종류**

```json
{
  Strict-Transport-Security: max-age=<expire-time>;includeSubDomains,
  //부여한 시간 동안, 모든 사이트가 HTTPS 를 통해 접근하도록 함

  X-XSS-Protection:1,
  //xss 취약점이 발견되면, 페이지로딩을 중단함. Content-Security-Policy 를 지원하지 않는 구형 브라우저에서 사용가능

  X-Frame-Options:DENY,//SAMEORIGIN
  //페이지를 frame, iframe, embed, object 내부에서 랜더링을 허용할 지 나타낼 수 있다.

  Permissions-Policy:,
  //웹사이트에서 사용할 수 있는 기능과 사용할 수 없는 기능을 명시적으로 선언하는 해더 
  

  X-Content-Type-Options
}
```

5.

## 14.5 취약점이 있는 패키지의 사용을 피하자

## 14.6 OWASP Top10
