## 1. 리액트에서 발생하는 크로스 사이트 스크립팅(XSS)

XSS란 웹사이트 개발자가 아닌 제3자가 웹사이트에 악성 스크립트를 삽입해 실행할 수 있는 취약점을 의미한다.

### dangerouslySetInnerHTML prop

이름에서 알 수 있듯이 특정 브라우저 DOM의 innerHTML을 특정한 내용으로 교체할 수 있는 방법이다.

```jsx
function App() {

	return <div dangerouslySetInnerHTML={{ __html: 'First &middot; Second' }} />
}
```

dangerouslySetInnerHTML의 위험 성은 인수로 받는 문자열에 제한이 없다는 것이다.

```jsx
const html = `<span><svg/onload=alert(origin)></span>'

function App() {
	return <div dangerouslySetInnerHTML={{ __html: html }} />
}
```

alert이 실행 될 것이며 넘겨주는 문자열 값은 한 번 더 검증이 필요하다

### useRef를 활용한 직접 삽입

useRef를 통해 직접 DOM에 접근해서 innerHTML로 삽입하면 동일한 문제가 발생한다.

```jsx
const html = `<span><svg/onload=alert(origin)></span>'

function App() {
	const divRef = useRef<HTMLDivElement>(null)

	useEffect(() => {
		if (divRef.current) {
			divRef.current.innerHTML = html
		}
	})
	
	return <div ref={divRef} />
}
```

위 코드 또한 alert이 나타나게 된다.

### 리액트에서 XSS 문제를 피하는 방법

가장 확실한 방법은 제3자가 삽입할 수 있는 HTML을 안전한 HTML 코드로 한 번 치환하는 것이다. 이러한 과정을 새니타이즈(sanitize) 또는 이스케이프(escape)라고 하는데, 라이브러리를 사용하는 것이 좋다.

- [DOMpurify](https://github.com/cure53/DOMPurify)
- [sanitize-html](https://github.com/apostrophecms/sanitize-html)
- [js-xss](https://github.com/leizongmin/js-xss)

<aside>
💡 리액트의 JSX 데이터 바인딩

```jsx
const html = `<span><svg/onload=alert(origin)></span>'

function App() {
	return <div>{html}</div>
}
```

리액트는 기본적으로 XSS를 방어하기 위해 이스케이프 작업이 존재한다.

위의 코드를 실행하면 alert이 실행되지 않는다. 따라서 원본값을 그대로 실행하기 위해 별도의 함수인 dangerouslySetInnerHTML가 존재하는 것이다.

</aside>

## 2. getServerSideProps와 서버 컴포넌트를 주의하자.

서버 사이드 렌더링과 서버 컴포넌트는 성능 이점을 가져다 줌과 동시에 서버라는 개발 환경을 프런트엔드 개발자에게 쥐어준 셈이 됐다. 즉, 브라우저에 정보를 내려줄 때는 조심해야 한다.

```jsx
export default function App({ cookie }: { cookie: string }) {
	if (!validatieCookie(cookie)) {
		Router.replace()
		return null
	}
}

export const getServerSideProps = async (ctx: GetServerSidePropsContext) => {
	const cookie = ctx.req.headers.cookie || ''

	return {
		props: {
			cookie,
		}
	}
}

```

이 코드는 쿠키를 가져온 다음에 클라이언트에서 유효성에 따라 작업을 처리하는 코드이다. 정상적으로 동작하겠지만 보안 관점에서는 좋지 못하다. 

props 값은 모두 사용자의 HTML에 기록되고 전역 변수로 등록되어 스크립트로 충분히 접근할 수 있는 보안 위협에 노출되는 값이 된다. 또 리다이렉트도 클라이언트에서 처리해 성능에 좋지 못하다.

```jsx

export default function App({ token }: { token: string }) {
	const user = JSON.parse(window.atob(token.split('.')[1]))
	const user_id = user.id
}

export const getServerSideProps = async (ctx: GetServerSidePropsContext) => {
	const cookie = ctx.req.headers.cookie || ''
	
	const token = validateCookie(cookie)

	if (!token) {
		return {
			redirect: {
				destination: '/404',
				permanet: false,
			}
		}
	}
	return {
		props: {
			token,
		}
	}
}
```

필요한 토큰만 제한적으로 변환했고, 리다이렉트 또한 서버에서 처리했다.

## **3. <a> 태그의 값에 적절한** 제한을 둬야 한다.

<a> href에 javascript::로 시작하는 코드를 넣어 onClick 이벤트와 같이 별도 이벤트 헨들러만 작동시키기 위한 용도로 사용할 때 가 있다.

```jsx
function App() {
	function handleClick() {
		console.log('hello');
	}
	
	return (
		<>
			<a href="javascript:;" onClick={handleClick}>
			링크
			</a>
		</>
	)
}
```

이렇게 하면 onClick만 실행되는데 페이지 이동이 없이 핸들러만 작동 시키고 싶다면 button 태그를 사용하는 것이 좋다.

왜냐하면 href가 작동하지 않은 것이 아니라 javascritp:;만 실행된 것이다. **즉 href내에 다른 자바스크립트 코드가 존재한다면 이를 실행한다는 뜻**이다.

```jsx
function isSafeHref(href: string) {
	let isSafe = false
	
	try {
		const url = new URL(href)
		
		if['http:', 'https:'].includes(url.protocol)) {
			isSafe = true
		}
	} catch {
		isSafe = false
	}

	return isSafe
}

function App() {
	const unsafeHref = "javascript:alert('hello');"
	const safeHref = 'https://www.naver.com'
	
	return (
		<>
			<a href={isSafeHref(unsafeHref) ? unsafeHref : '#'} onClick={handleClick}>
			위험한 href
			</a>
			<a href={isSafeHref(safeHref) ? safeHref : '#'} onClick={handleClick}>
			안전한 href
			</a>
		</>
	)
}
```

## 4. HTTP 보안 헤더 설정하기

보안 헤더란 브라우저가 렌더링하는 내용과 관련된 보안 취약점을 미연에 방지하기 위해 브라우저와 함께 작동하는 헤더를 말한다.

### Strict-Transport-Security

Strict-Transport-Security 응답 헤더는 모드 사이트가 HTTPS를 통해 접근해야 하며, 만약 HTTP로 접근하는 경우 이러한 모든 시도는 HTTPS로 변경되게 한다.

`Strict-Transport-Security: max-age=<expire-time>; includeSubDomains`

### X-XSS-Protection

이는 비표준 기술로, 구형 브라우저와 사파리에서만 제공되는 기능이다.

XSS 취약점이 발견되면 페이지 로딩을 중단하는 헤더다. 이 헤더를 전적으로 믿어서는 안되며, 반드시 페이지 내부에서 XSS에 대한 처리가 존재하는 것이 좋다.

### X-Frame-Options

페이지를 frame, iframe, embed, object 내부에서 렌더링을 허용할지를 나타낼 수 있다.

```jsx
X-Frame-Options: DENY // 만약 위와 같은 프레임 관련 코드가 있다면 무조건 막는다.
X-Frame-Options: SAMEORIGIN // 같은 origin에 대해서만 프레임을 허용한다.
```

### Permissions-Policy

웹사이트에서 사용할 수 있는 기능과 사용할 수 없는 기능을 명시적으로 선언하는 헤더다.

다양한 브러우저의 기능이나 API(GPS 등)를 선택적으로 활성화하거나 필요에 따라서는 비활성화 할 수 있다.

```jsx
// 모든 geolocation 사용을 막는다.
Permissions-Policy: geolocation=()

// geolocation을 페이지 자신과 몇 가지 페이지에 대해서만 허용한다.
Permissions-Policy: geolocation=(self "https://a.yceffort.kr" "https://b.ycefforct.kr")

// 카메라는 모든 곳에서 허용한다.
Permissions-Policy: camera=*;

// pip 기능을 막고, geolocation은 자신과 특정 페이지만 허용하며, 카메라는 모든 곳에서 허용한다.
Permissions-Policy: picture-in-picture:(), geolocation=(self "https://a.yceffort.kr" "https://b.ycefforct.kr"), camera=*;

```

### X-Content-Type-Options

MIME을 먼저 알아야하는데, MIME(Multipurpose Internet Mail Extensions)는 Content-type의 값으로 사용된다. 원래는 메일을 전송할 때 사용하던 인코딩 방식으로, 현재는 Content-type에서 대표적으로 사용되고 있다.

X-Content-Type-Options 란 Content-type 헤더에서 제공하는 MIME 유형이 브라우저에 의해 임의로 변경되지 않게 하는 헤더이다. 즉, Content-type: text/css 헤더가 없는 파일은 브라우저가 임의로 CSS로 사용할 수 없다. **웹서버가 브라우저에 강제로 이 파일을 읽는 방식을 지정하는 것**이 바로 이 헤더다.

### Referrer-Policy

Referer라는 헤더는 현재 요청을 보낸 페이지의 주소가 나타난다. 만약 링크를 통해 들어왔다면 해당 링크를 포함하고 있는 페이지 주소가 다른 도메인에 요청을 보낸다면 해당 리소스를 사용하는 페이지 주소가 포함된다.

(참고로 Referer과 Referrer-Policy의 철자가 다른 이유는 Referer라는 오타가 이미 표준으로 등록된 이후에 뒤늦게 오타임을 발견했기 때문)

**출처에 대한 이야기 (https://yceffort.kr)**

- scheme: HTTPS 프로토콜을 의미한다.
- hostname: [yceffort.kr](http://yceffort.kr)이라는 호스트명을 의미한다.
- port: 443 포트를 의미한다.

https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy

### Content-Security-Policy

콘텐츠 보안 정책은 XSS 공격이나 데이터 삽입 공격과 같은 다양한 보안 위협을 막기 위해 설계됐다.

***-src**

font-src, img-src, script-src 등 다양한 src를 제어할 수 있는 지시문이다.

```jsx
Content-Security-Policy: font-src <source>;
```

font의 src로 가져올 수 있는 소스를 제한할 수 있다. 선언된 font 이외의 소스는 모두 차단된다.

**form-action**

form-action은 폼 양식으로 제출할 수 있는 URL을 제한할 수 있다.

```jsx
<meta http-equiv="Content-Security-Policy" content="form-action 'none'" />
export default function App() {
	function handleFormAction() {
		alert('form action!')
	}
	
	return (
		<div>
			<form action={handleFormAction} method="post">
				<input type="text" name="name" value="foo" />
				<input type="submit" id="submit" value="submit" />
			</form>
		</div>
	)
}
```

이 컴포넌트에서 submit을 눌러 form을 제출하면 콘솔에 다음과 같은 에러 메시지가 출력되며 동작하지 않는다.

> Refused to send form data to ‘****’ because it violates the following Content Security Policy directive: “form-action= ‘none’”.
> 

form-action 을 none으로 해서 거절된 것이다.

### 보안 헤더 설정하기

**[Next.js](https://nextjs.org/docs/pages/api-reference/next-config-js/headers)**

Next.js에서는 애플리케이션 보안을 위해 HTTP 경로별로 보안 헤더를 적용할 수 있다.

```jsx
const securityHeaders = [
	{
		key: 'key',
		value: 'value',
	}
]

module.exports = {
	async headers() {
		return [
			{
				/* 모든 주소에 설정한다. */
				source: '/:path*',
				headers: securityHeaders
			}	
		]
	}
}
```

**NGINX**

정적인 파일을 제공하는 NGINX의 경우 다음과 같이 경로별로 add_header지시자를 사용해 원하는 응답헤더를 더 추가할 수 있다.

```jsx
location / {
	# ...
	add_header X-XSS-Protection "1; mode=block";
	add_header Content-security-Policy "default-src 'self'; script-src 'self'; child-src e_m; style-src 'self' example.com; font-src 'self';";
	#...
}
```

## 5. 취약점이 있는 패키지의 사용을 피하자

package.json에 어떤 패키지가 있는지 정도는 파악할 수 있지만 package-lock.json의 모든 의존성을 파악하는 것은 사실상 불가능에 가깝다.

- Next.js: 12.0.0 ~ 12.0.4 와 11.1.0 ~ 11.1.2 버전 사이에 URL을 잘못 처리하는 버그가 있어 서브 가동이 중지될 수 있는 버그가 있었다.
- React: 초기버전인 0.0.1 ~ 0.14.0 버전 사이에 XSS 취약점이 있었다.
- react-dom: 16.{0,1,2,3,4}.{0,1} 버전의 react-dom/server에 XSS 취약점이 있었다.

## 6. OWASP Top 10