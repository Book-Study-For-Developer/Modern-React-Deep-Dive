## 1. app 디렉터리의 등장

- \_document: 페이지에 쓰이는 <html>과 <body> 태그를 수정하거나, 서버 사이드 렌더링 시 styled-components 같은 일부 CSS-in-JS를 지원하기 위한 코드를 삽입하는 제한적인 용도로 사용된다.
- \_app: app은 페이지를 초기화하기 위한 용도로 사용되며, 다음과 같은 작업이 가능하다.
  - 페이지 변경 시에 유지하고 싶은 레이아웃
  - 페이지 변경 시 상태 유지
  - componentDidCatch를 활용한 에러 핸들링
  - 페이지간 추가적인 데이터 삽입
  - global CSS 주입

즉 12버전까지는 페이지 공통 레이아웃을 유지할 수 있는 방법은 **\_app이 유일**했다.

### 라우팅

/app 디렉토리로 라우팅 방식이 변경되었다.

- 12 이하: /pages/a/b.tsx 또는 /pages/a/b/index.tsx 모두 동일한 주소
- 13: /app/a/b 는 /a/b 로 변환되며 파일명은 무시

**layout.js**

```tsx
export default function DashboardLayout({
  children, // will be a page or nested layout
}: {
  children: React.ReactNode;
}) {
  return (
    <section>
      {/* Include shared UI here e.g. a header or sidebar */}
      <nav></nav>

      {children}
    </section>
  );
}
```

\_app과 \_document를 대신해 공통 코드를 삽입할 수도 있게 되었다.

주의할점

- layout은 app 디렉터리 내부에서는 예약어이다. 무조건 layout을 써야하며 다른 목적으로는 사용할 수 없다.
- layout 내부에는 export default가 반드시 있어야 한다.

**pages.js**

```tsx
// `app/page.tsx` is the UI for the `/` URL
export default function Page() {
  return <h1>Hello, Home page!</h1>;
}
```

page도 예약어이며, 다음과 같은 props를 받는다.

- params: 옵셔널 값으로 동적 라우트 파라미터를 사용할 경우 해당 파라미터에 값이 들어온다. `ex) […id]`
- searchParams: URL에서 ?a=1 과 같은 query에 해당한다.

주의 사항

- 무조건 page.(js,ts,jsx,tsx)을 써야하며 다른 목적으로는 사용할 수 없다.
- export default가 반드시 있어야 한다.

**error.js**

```tsx
"use client";

export default function Error({ error, reset }: { error: Error & { digest?: string }; reset: () => void }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

에러 바운더리는 클라이언트에서만 작동하므로 클라리언트 컴포넌트여야한다.

(https://nextjs.org/docs/app/building-your-application/routing/error-handling#handling-errors-in-layouts)

## 리액트 서버 컴포넌트

### 기존 리액트 컴포넌트와 서버 사이드 렌더링의 한계

한계점

- **자바스크립트 번들 크기가 0인 컴포넌트를 만들 수 없다:** 게시판 등 사용자가 작성한 HTML에 위험한 태그를 제거하기 위해 사용되는 유명한 sanitize-html라이브러리를 사용한다고 가정하자
  ```tsx
  import sanitizeHtml from "sanitize-html"; // 206k (63.3K gzipped)

  function Board({ text }: { text: string }) {
    const html = useMemo(() => sanitizeHtml(text), [text]);

    return <div dangerouslySetInnerHtml={{ __html: html }} />;
  }
  ```
  만약 저 라이브러리를 렌더링한 결과만 클라이언트에 제공하게 된다면 라이브러리를 다운로드 해도 되지 않아도 된다.
- **백엔드 리소스에 대한 직접적인 접근이 불가능하다:** 클라이언트에서 백엔드 데이터에 접근하려면 REST API와 같은 방법이 일반적이다. 이는 항상 백엔드에서 데이터를 제공해줘야 한다. 그러나 직접 가져올 수 있다면 수고로움이 줄어들 것이다. (Q. 프론트에서 DB에 직접 접근하는 것이 맞을까?)
  성능 관점에서 볼 때 단계가 하나 줄어든 셈이므로 이점을 가지고 있다.
- **자동 코드 분할이 불가능하다:** 개발자는 항상 코드 분할을 해도 되는 컴포넌트인지를 유념하고 개발해야 하기 때문에 이를 누락하는 경우가 발생할 수 있고 if문 전까지 어떤 컴포넌트를 불러올지 결정할 수 없다.
- **연쇄적으로 발생하는 클라이언트와 서버의 요청을 대응하기 어렵다:** 요청이 연달아 컴포넌트를 렌더링한다고 할때 최초 컴포넌트의 요청과 렌더링이 끝나기 전까지는 하위 컴포넌트의 요청과 렌더링이 끝나지 않는다는 큰 단점이 있다.
  서버에서 수행한다면 모두 서버에서 렌더링이 일어나기에 클라이언트에서는 반복적으로 요청을 수행하지 않아도 된다. 그에 따라 효율적으로 컴포넌트를 렌더링할 수 있다.
- **추상화에 드는 비용이 증가한다:** 리액트는 탬플릿 언어로 설계되지 않았다. 탬플릿 언어란 HTML에 특정 언어의 문법을 집어넣어 사용할 수 있는 것을 의미한다. 복잡한 추상화에 따른 결과물을 연산하는 작업을 서버에서 수행하게 된다면, 클라이언트에서는 속도가 빨라지고 서버에서는 클라이언트로 전송되는 결과물도 가벼워질 것이다.

### 서버 컴포넌트란?

서버 컴포넌트란 하나의 언어, 하나의 프레임워크, 그리고 하나의 API와 개념을 사용하면서 서버와 클라이언트 모두에서 컴포넌트를 렌더링할 수 있는 기법을 의미한다.

클라이언트 컴포넌트는 서버 컴포넌트를 import 할 수 없다. 그 이유는 **서버 컴포넌트를 불러오게 된다면 서버 컴포넌트를 실행할 방법이 없기 때문**이다.

```tsx
import ServerComponent form './ServerComponent.server'
// x 클라이언트 컴포넌트에서 서버 컴포넌트를 불러오는 것은 불가능
export default function ClientComponent() {
	return (
		<div>
			<ServerComponent />
		</div>
	)
}

// 이 컴포넌트는 서버이거나 클라이언트 일 수도 있다.
import ClientComponent from './ClientComponent'
import ServerComponent form './ServerComponent.server'

export default function ParentServerComponent() {
	return (
		<ClientComponent>
			<ServerComponent />
		</ClientComponent>
	)
}
```

각 컴포넌트의 차이와 제약사항은 다음과 같다.

- 서버 컴포넌트
  - 요청이 오면 그 순간 서버에서 딱 한 번 실행될 뿐이므로 상태를 가질 수 없다. (`useState`, `useReducer` 등의 훅을 사용할 수 없다.)
  - 렌더링 생명주기도 사용할 수 없다. (`useEffect`, `useLayoutEffect`를 사용할 수 없다.)
  - 위의 훅을 사용한 커스텀훅은 사용할 수 없다.
  - 서버에서만 실행되기 떄문에 `window`, `document` 등에 접근할 수 없다.
  - 데이터베이스, 내부 서비스, 파일 시스템 등 서버에만 있는 데이터를 async/await으로 접근할 수 있다. 컴포넌트 자체가 async한 것이 가능하다.
  - 다른 서버 컴포넌트를 렌더링하거나 div, span, p 같은 요소를 렌더링하거나, 혹은 클라이언트 컴포넌트를 렌더링할 수 있다.
- 클라이언트 컴포넌트
  - 브라우저 환경에서만 실행되므로 서버 컴포넌트를 불러오거나, 서버 전용 훅이나 유틸리티를 불러올 수 없다.
  - 클라이언트 컴포넌트가 자식으로 서버 컴포넌트를 갖는 구조는 가능하다. 서버 컴포넌트는 이미 서버에서 만들어진 트리를 가지고 있고, 클라이언트 컴포넌트는 이미 서버에서 만들어진 그 트리를 삽입해서 보여주기 때문에 가능하다.
- 공용 컴포넌트
  - 서버와 클라이언트에서 모두 사용 가능하다. 이는 서버, 클라이언트의 제약사항을 모두 받는 컴포넌트가 된다.

리액트는 기본적으로 다 공용 컴포넌트로 파악하고 대신 클라이언트 컴포넌트를 명시하려면 ‘use client’라고 작성해두면 된다.

### 서버 사이드 렌더링과 서버 컴포넌트의 차이

서버 사이드 렌더링의 목적은 초기에 인터랙션은 불가능하지만 정적인 HTML을 빠르게 내려주는 데 초점을 두고 있다. 그렇기에 여전히 HTML이 로딩된 이후에는 **자바스크립트 코드를 다운로드하고 파싱하고 실행하는데 비용이 든다.**

서버 컴포넌트를 활용해 서버에서 렌더링할 수 있는 컴포넌트는 서버에서 완성해 제공받은 다음, 클라이언트에서 서버 사이드 렌더링으로 초기 HTML을 전달받을 수 있다. 이는 자바스크립트의 양도 줄어들어 브라우저의 부담을 덜 수 있다.

### 서버 컴포넌트는 어떻게 작동하는가?

https://github.com/reactjs/server-components-demo/blob/29ec74365084e1921c6bf49c03266788099449a3/server/api.server.js#L74-L87

```tsx
app.get(
  "/",
  handleErrors(async function (_req, res) {
    await waitForWebpack();
    const html = readFileSync(path.resolve(__dirname, "../build/index.html"), "utf8");
    // Note: this is sending an empty HTML shell, like a client-side-only app.
    // However, the intended solution (which isn't built out yet) is to read
    // from the Server endpoint and turn its response into an HTML stream.
    res.send(html);
  })
);
```

1. 서버가 렌더링 요청을 받는다. 서버가 렌더링 과정을 수행해야 하므로 서버에서 시작된다.
2. 서버에서 받은 요청에 따라 컴포넌트를 JSON으로 직렬화 한다. 이때 서버에서 렌더링할 수 있는 것은 직렬화해서 내보내고, 클라이언트로 표시된 부분은 해당 공간을 플레이스홀더 형식으로 비워두고 나타낸다. 브라우저는 이를 역질렬화해서 렌더링을 수행한다.
3. 브라우저가 리액트 컴포넌트 트리를 구성한다. 브라우저가 서버로 스트리밍으로 JSON 결과물을 받았다면 이 구문을 다시 파싱한 결과물을 바탕으로 트리를 재구성해 컴포넌트를 만들어 나간다.

## Next.js에서의 리액트 서버 컴포넌트

### 새로운 fetch 도입과 getServerSideProps, getStaticProps, getInitialProps의 삭제

getServerSideProps, getStaticProps, getInitialProps 는 fetch 기반으로 요청하도록 변경되었다.

이에 서버 컴포넌트 트리내에 동일한 요청이 있다면 재요청이 발생하지 않도록 요청 중복을 방지했다.

### 정적 렌더링과 동적 렌더링

정적인 라우팅에 대해서는 기본적으로 빌드 임에 렌더링을 미리 해두고 캐싱해 재사용할 수 있게끔 해뒀고, 동적인 라우팅에 대해서는 서버에 매번 요청이 올 때마다 컴포넌트를 렌더링하도록 변경했다.

```tsx
async function getData() {
  const res = await fetch("https://api.example.com/...", { cache: "no-cache" });
  // The return value is *not* serialized
  // You can return Date, Map, Set, etc.

  if (!res.ok) {
    // This will activate the closest `error.js` Error Boundary
    throw new Error("Failed to fetch data");
  }

  return res.json();
}

export default async function Page() {
  const data = await getData();

  return <main></main>;
}
```

동적인 렌더링을 원할 경우 `{ cache: 'no-cache' }` 옵션을 추가해서 요청이 올때 마다 렌더링을 수행하게 하면되고, 정적일 경우엔 제거하면 된다.

만약 동적인 주소이지만 특정 주소에 대해서만 캐싱하고 싶을 경우에는 `generateStaticParams` 를 사용하면 된다.

https://nextjs.org/docs/app/api-reference/functions/generate-static-params

```tsx
// Return a list of `params` to populate the [slug] dynamic segment
export async function generateStaticParams() {
  const posts = await fetch("https://.../posts").then((res) => res.json());

  return posts.map((post) => ({
    slug: post.slug,
  }));
}

// Multiple versions of this page will be statically generated
// using the `params` returned by `generateStaticParams`
export default function Page({ params }) {
  const { slug } = params;
  // ...
}
```

### 캐시와 mutating, 그리고 revalidating

페이지에 revalidate라는 변수를 선언해서 페이지 레벨로 정의하는 것도 가능하다.

1. 최초로 해당 라우트 요청이 올 때는 미리 정적으로 캐시해 둔 데이터를 보여준다.
2. 이 캐시된 초기 요청은 revalidate에 선언된 값만큼 유지된다.
3. 만약 해당 시간이 지나도 일단은 캐시된 데이터를 보여준다.
4. Next.js는 캐시된 데이터를 보여주는 한편, 시간이 경과했으므로 백그라운드에서 다시 데이터를 불러온다.
5. 4번의 작업이 성공적으로 끝나면 캐시된 데이터를 갱신하고, 그렇지 않다면 과거 데이터를 보여준다.

## 서버 액션

[Data Fetching: Server Actions and Mutations](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#forms)

### **[Redirecting](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#redirecting)**

If you would like to redirect the user to a different route after the completion of a Server Action, you can use `[redirect](https://nextjs.org/docs/app/api-reference/functions/redirect)` API. `redirect` needs to be called outside of the `try/catch` block:

app/actions.ts

```tsx
"use server";

import { redirect } from "next/navigation";
import { revalidateTag } from "next/cache";

export async function createPost(id: string) {
  try {
    // ...
  } catch (error) {
    // ...
  }

  // revalidatePath 해당 주소의 캐시를 즉시 업데이트
  revalidateTag("posts"); // Update cached posts 추가해둔 태그의 요청을 모두 초기
  redirect(`/post/${id}`); // 특정 주소로 리다이렉트
}
```

### 서버 액션 사용 시 주의할 점

- 서버 액션은 클라이언트 컴포넌트 내에서 정의될 수 없다. 클라이언트 컴포넌트에서 서버 액션을 쓰고 싶을 때는 ‘use server’로 서버 액션만 모여있는 파일을 별도로 import 해야 한다.
- 서버 액션으 import하는 것 뿐만 아니라, props 형태로 서버 액션을 클라이언트 컴포넌트에 넘기는 것 또한 가능하다. 즉 서버에서만 실행될 수 있는 자원은 반드시 파일 단위로 분리해야 한다.

## Next.js 13코드 맛보기

### getStaticProps와 비슷한 정적인 페이지 렌더링 구현해보기

```tsx
export async function fetchPostById(id: number | string, options?: RequestInit): Promise<Post> {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, options);
  const result: Post = await response.json();

  return result;
}

// https://github.com/wikibook/react-deep-dive-example/blob/main/chapter11/next13/app/ssg/%5Bid%5D/page.tsx
import { fetchPostById } from "#services/server";

export async function generateStaticParams() {
  return [{ id: "1" }, { id: "2" }, { id: "3" }, { id: "4" }];
}

export default async function Page({ params }: { params: { id: string } }) {
  const data = await fetchPostById(params.id);

  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-medium text-gray-100">{data.title}</h1>
      <p className="font-medium text-gray-400">{data.body}</p>
    </div>
  );
}
```

`generateStaticParams`를 통해 [id]로 사용할 값을 객체 배열로 모아두고 별다른 옵션 없이 fetch하게 되면 **미리 모든 html이 빌드 타임에 생성된다.**

`revalidate`를 써주게 되면 캐시 유효시간이 지나게 되면 이전의 캐시를 무효화하고 새로운 페이지를 빌드하게 된다.
