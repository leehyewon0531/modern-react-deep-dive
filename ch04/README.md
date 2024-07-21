# [4장] 서버 사이드 렌더링

## 4.1 서버 사이드 렌더링이란?

### 싱글 페이지 어플리케이션 (SPA: Single Page Application)

-   첫 페이지는 모든 데이터 불러와 **빈 HTML에 자바스크립트 코드로 `<body />` 내용을 넣어 렌더링**
-   하나의 페이지에서 모든 작업 처리, 다음 페이지 렌더링에 필요한 정보만 요청해 DOM을 추가, 수정, 삭제

**👍 [장점]**

    최초 로딩 후에는 리소스를 받아올 일이 적어지므로, 훌륭한 UI/UX 제공

**👎 [단점]**

    최초 로딩할 JS 리소스가 커짐

<br/>

> **등장 배경**

-   과거 PHP, JSP 전통적 방식 - 페이지 전환 시 페이지를 처음부터 새로 그려야 한다는 단점 有
-   JS의 성장으로…
    -   자바스크립트 모듈화, 사용자 기기 성능 향상, 인터넷 속도 발전 등으로 JS에서 할 수 있는 일이 다양
    -   프레임워크의 등장으로 JAM 스택이 등장, JS와 마크업(HTML, CSS)을 미리 빌드해 두고 정적으로 사용자에게 제공

<br/>

> 한계점

-   자바스크립트 리소스의 크기와 수가 증가
-   사용자의 기기와 인터넷 속도 등 환경이 크게 개선
-   But, 실제 사용자들이 느끼는 웹 애플리케이션의 로딩 속도는 5년 전과 같거나 더 느림

<br/>

### 서버 사이드 렌더링 (Server-Side Rendering)

-   최초에 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자 화면을 제공
    -   **SPA** - `자바스크립트 번들`에서 렌더링 → 사용자 기기의 성능에 영향
    -   **SSR** - `서버`에서 렌더링 → 비교적 안정적인 렌더링
    -   ‘ **`렌더링의 책임`**’을 어디에 두느냐

**👍 [장점]**

    -   최초 페이지 진입이 비교적 빠르다 (서버가 HTML 처리, HTTP 요청이 비교적 빠름)
    -   검색 엔진과 SNS 공유 등 메타 데이터 제공이 쉽다
    -   누적 레이아웃 이동이 적다 (ex. 중간에 배너가 삽입되어 레이아웃이 이동하는 현상)
    -   사용자의 디바이스 성농에 비교적 자유롭다
    -   보안에 좀 더 안전하다 (서버 단에서 처리하므로, 브라우저에 노출되지 않음) <br/>

**👎 [단점]**

    -   소스코드 작성 시, 항상 서버를 고려해야 한다 (클라이언트 사이드에서만 가능한 window 접근 등)
    - 적절한 서버가 구축되어야 한다 (요청에 대응하는 서버를 하나 더 준비하는 것)
    - 서비스 지연에 따른 문제 (렌더링이 끝날 때까지 어떤 정보도 제공 불가)

### 정리

> **SSR이 만능은 아니다**

-   오히려 관리 포인트가 서버, 클라이언트 두 개로 늘어나는 역효과
-   웹페이지에서 사용자게에 제공하고 싶은 내용이 무엇인지, 어떤 우선순위에 따라 페이지의 내용을 보여줄지를 잘 설계하는 것이 중요

> **SPA vs. SSR**

-   뛰어난 SPA는 MPA보다 낫다 _(ex. gmail)_
    -   최초 페이지에서는 필요한 정보만 최적화하여 노출
    -   이미지 같이 중요도가 떨어지는 리소스는 lazy loading
    -   code splitting - 필요한 코드만 나눠서 번들링하는 기법
-   평균적으로는 SPA가 SSR보다 느리다
    -   SPA가 렌더링, 라우팅 최적화되어 있지 않다면,
    -   보통 클라 < 서버의 성능이 좋으므로 SSR이 빠르다
    -   SSR 라우팅 기법도 존재 - 페인트 홀딩, bfcache, Shared Element Transition

> **현재의 서버 사이드 렌더링 (Next, Remix 등)**

-   (과거) 모든 페이지를 서버에서 렌더링
-   (현재) 최초 웹사이트 진입 시에는 서버 사이드 렌더링 방식으로 서버에서 완성된 HTML을 제공

    -> 이후 라우팅은 서버에서 내려받은 자바스크립트를 바탕으로 SPA처럼 작동

## 4.2 SSR을 위한 리액트 API

-   [**`react-dom/server.js`**](https://react.dev/reference/react-dom/server) 에서 제공
-   Node.js와 같은 서버 환경에서만 실행 가능

### renderToString

-   리액트 컴포넌트를 렌더링해 HTML 문자열로 반환하는 함수 **`string`**
-   **[목적]** 최초 HTML 페이지를 빠르게 그려주는 목적 (인터렉션을 위해서는 별도의 JS 다운/파싱/실행)
-   data-reactroot 속성으로 hydrate 함수에서 리액트 컴포넌트의 루트 엘리먼트를 식별

```jsx
const result = ReactDOMserver.**renderToString**(React.createElement("div", { id: "root" }, <SampleComponent />));
```

### renderToStaticMarkup

-   리액트 컴포넌트를 렌더링해 HTML 문자열로 반환하는 함수 **`string`**
    -   vs. renderToString 리액트에서만 사용하는 추가적인 DOM 속성(ex. data-reactroot)을 만들지 X
    -   결과물인 HTML의 크기를 아주 약간이라도 줄일 수 O
-   **[목적]** hydrate를 수행하지 않는다는 가정 하에, 리액트의 이벤트 리스너가 필요 없는 완전히 순수한 HTML을 만들 때 사용

```jsx
const result = ReactDOMServer.**renderToStaticMarkup**(React.createElement("div", { id: "root" }, <SampleComponent />));
```

### renderToNodeStream

-   리액트 컴포넌트를 렌더링해 HTML을 `ReadableStream`으로 반환하는 함수
-   utf-8로 인코딩된 바이트 스트림이므로, Node.js와 같은 서버 환경에서 사용 가능
-   **[목적]** 큰 크기의 데이터를 청크 단위로 분리해 순차적으로 처리

### renderToStaticNodeStream

-   리액트 컴포넌트를 렌더링해 HTML을 `ReadableStream`으로 반환하는 함수
    -   vs. 리액트 자바스크립트에 필요한 리액트 속성이 제공되지 않음
-   **[목적]** hydrate를 수행하지 않는다는 가정 하에, 리액트의 이벤트 리스너가 필요 없는 완전히 순수한 HTML을 만들 때 사용

|             | renderToString | renderToStaticMarkup | renderToNodeStream | renderToStaticNodeStream |
| ----------- | -------------- | -------------------- | ------------------ | ------------------------ |
| output      | string         | string               | ReadableStream     | ReadableStream           |
| 리액트 속성 | O              | X                    | O                  | X                        |

### hydrate

-   renderToString, renderToNodeStream으로 생성된 HTML 콘텐츠에
-   자바스크립트 핸들러나 이벤트를 붙이는 역할
-   정적으로 생성된 HTML에 이벤트와 핸들러를 붙여 완전한 웹페이지 결과물을 만든다!

```jsx
ReactDOM.render(<App />, rootElement); /* 렌더링 및 JS 붙이는 작업까지 수행 */
ReactDOM.hydrate(<App />, element); /* JS 붙이는 작업 수행 */
```

## Next.js

-   Vercel에서 만든 리액트 기반 프레임워크
-   과거 메타 프로젝트인 `react-page`를 기반으로 하여 디렉터리 기반 라우팅 제공

### 기본 구조

> **디렉터리 기반 라우팅**

-   pages 폴더를 기초로 구성
-   pages/index.tsx → `/`
-   pages/hello.tsx → `/hello`
-   pages/hello/[greeting].tsx → `/hello/world` // greeting: world로 들어옴
-   pages/hello/[…props].tsx // 도 가능하다, 단 항상 string[]로 들어옴

<br/>

> **서버 API 정의**

-   pages/api 폴더 - `/api/hello` 로 호출 가능
-   서버에서 내려주는 데이터를 조합해 BFF(Backend-for-frontend) 형태로 활용
-   CORS 문제 우회하기 위해 사용 → 외부 API 호출 시
-   인증 관련 ex. 쿠키 발행을 위해

```tsx
/* /pages/api/hello.ts */
import type { NextApiRequest, NextApiResponse } from "next";

export default function handler(req: NextApiReqeust, res: NextApiResponse) {
    res.status(200).json({ name: "John Doe" });
}
```

<br/>

### 서버 라우팅 vs. 클라이언트 라우팅

: Next.js는 초기 렌더링은 SSR 방식으로 받아오지만, 이후는 CSR로 진행한다 (**json** + **JS** → DOM)

```html
<a href="/hello"> 서버 라우팅으로 이동 </a>
<Link href="/hello"> 클라이언트 라우팅으로 이동</Link>
```

<br/>

### Data Fetching

: SSR 지원을 위한 데이터 불러오기 전략

| 조건 | 1. pages/ 폴더에 있는 라우팅이 되는 파일에서만 사용 가능 2. 정해진 함수명으로 export 해주어야 함 |
| --- | --- |

-   **getStaticPaths `static`**

    -   서버에서 미리 필요한 페이지를 만들어 제공
    -   접근 가능한 주소를 미리 정의하여 가능한 조합을 빌드 시점에 불러와 페이지로 미리 렌더링

    ```tsx
    export const getStaticPaths: GetStaticPaths = async () => {
        return {
            paths: [{ params: { id: "1" } }, { params: { id: "2" } }],
            fallback: false,
        };
    };
    ```

-   **getStaticProps `static`**
    -   props를 가지고 데이터를 조회해서 페이지를 만들 수 있음
    ```tsx
    export const getStaticProps: GetStaticProps = async ({ params }) => {
        const { id } = params;

        const post = await fetchPost(id);

        return {
            props: { post },
        };
    };

    export default function Post({ post }: { post: Post }) {
        // post가 props로 넘어온다 > 이걸로 페이지 렌더링
    }
    ```

---

-   **getServerSideProps**
    -   서버에서 실행되는 함수로, 페이지 진입 전에 무조건 실행
    -   값에 따라 리다이렉트 혹은 props로 반환 가능
    ```tsx
    export default function Post({ post }: { post: Post}) {

    }

    export const getServerSideProps: GetServerSideProps = async (context) => {
    	const { query: { id: ''}} = context;

    	const post = await fetchPost(id.toString());

    	if (!post) redirect: { destination: '/404' }
    	return {
    		props: { post }
    	}
    }

    ```

(최근에는 그냥 디폴트로 서버 사이드로 굴러감)
<br/>

### 스타일

> **전역 스타일**

-   `_app.tsx` 에 import

<br/>

> **컴포넌트 레벨 CSS**

-   `[name].module.css` 로 생성 후 import
-   다른 컴포넌트와 충돌이 일어나지 않게 고유한 클래스명 제공
-   scss, sass도 동일하게 제공
    ```jsx
    import styles from "./Button.module.css";

    export function Button() {
        return <Button className={styles.button} />;
    }
    ```
<br/>

> **css-in-js**

-   Next.js에서 사용 시, `_document.tsx` 에서 세팅이 필요
-   styled-components의 스타일을 모아 > 고유한 클래스명 부여 > 서버 렌더링 시, React.Context 형태로 제공
-   얘네도 모듈로 묶어서 ..
-   But, styled-component는 클라이언트 사이드로 돌려야 함! (그래서 tailwind 많이 사용)

<br/>

### next.config.js

: Next.js 실행에 필요한 설정을 추가할 수 있는 파일

```jsx
/**
 * @type {import('next').NextConfig}
 */

const nextConfig = {
    basePath: "/docs",
    swcMinify: true, // swc로 코드 압축 여부 (Rust라 빠른데 실험적 기능???)
    async redirects() {
        return [
            {
                source: "/api/about",
                destination: "/",
                permanent: true, // true: 308 false: 307
            },
        ];
    },
    assetPrefix: isProd ? "https://cdn.mydomain.com" : undefined,
};
module.exports = nextConfig;
```
<br/>

> **basePath 설정 시, 라우팅**

-   <Link>나 router.push()를 이용할 때는 그대로 사용한다
      - <Link href="/about">
      - router.push(’/about’)
-   하지만 이건 Next.js에서 제공하는 기능이기에, <a>나 window.location.push 등을 사용할 땐 주의
    -   <a href=”/docs/about/”>

<br/>

## [추가 토론 내용]

-   nextjs에도 미들웨어 개념이 있나?
    -   있다! https://thefirstperson.tistory.com/278
    -   미들웨어도 서버, 클라이언트(라우팅 미들웨어) 개념이 있는가? -> nextjs에 따르면 서버 사이드가 맞다
 
-   redirect vs rewirtes
    -   [https://velog.io/@chhw130/NextJs-nextJs에서-Proxy설정하기](https://velog.io/@chhw130/NextJs-nextJs%EC%97%90%EC%84%9C-Proxy%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)
    -   https://velog.io/@lucky-jun/proxy-nextjs
  
-   ssr이 적용되지 않는 라이브러리 사용할 때 주의
    -   [https://velog.io/@devilng/Next.js-트러블-슈팅-정리-4](https://velog.io/@devilng/Next.js-%ED%8A%B8%EB%9F%AC%EB%B8%94-%EC%8A%88%ED%8C%85-%EC%A0%95%EB%A6%AC-4)
