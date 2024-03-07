긴 공백기 끝에 다시 재개..

## Step 3 : `next/head` 대체하기

기존의 `next/head`는 HTML의 `<head>` 태그(`title`, `meta` 등)를 관리하기 위해 쓰인다. 하지만 `app` 폴더에서는 [Metadata](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)를 이용하면 된다.


**before**
```jsx
// pages/index.tsx
import Head from 'next/head'
 
export default function Page() {
  return (
    <>
      <Head>
        <title>My page title</title>
      </Head>
    </>
  )
}
```

**after**
```tsx
// app/page.tsx
import { Metadata } from 'next'
 
export const metadata: Metadata = {
  title: 'My Page Title',
}
 
export default function Page() {
  return '...'
}
```

## Step 4 : 페이지 이사하기
여기부터 정신을 똑바로 차려야 한다.

-  `pages` 폴더에서는 페이지 컴포넌트들이 모두 클라이언트 컴포넌트였던 것에 반해,  `app` 폴더에서는 기본적으로 모두 서버 컴포넌트로 설정된다.
-  기존의 서버사이드에서 데이터를 가져오는 방법들 `getServerSideProps`, `getStaticProps`, `getInitialProps` 등은 모두 새로운 단순한 API로 대체될 예정이다.
- `app` 폴더는 중첩된 폴더와 `page.tsx` 파일을 이용해 라우트를 구성한다.

  ![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/b7aibh56hjcc.png)




공식문서에서는 Step 4 과정을 아래 두 단계로 나누어 진행하라고 권장한다.
1. 기존의 페이지 컴포넌트를 클라이언트 컴포넌트로 변경한다.
2. 1단계에서 만든 클라이언트 컴포넌트를 `app` 폴더 하위의 `page.tsx` 에서 import 해서 사용한다.

#### 1. 클라이언트 컴포넌트 만들기
- `app` 폴더 내부에 클라이언트 컴포넌트를 내보내는 별도의 파일을 만든다. (ex. `app/home-page.tsx`) 파일 최상단의 `'use client'` 지시어를 통해 클라이언트 컴포넌트로 선언할 수 있다.
    - `pages router` 때와 유사하게 첫 페이지 로드시 클라이언트 컴포넌트를 정적인 HTML로 사전 렌더링하는 최적화 단계가 있다.

- 페이지 컴포넌트를 `pages/index.ts`에서 `app/home-page.tsx`로 이동한다.
```tsx
'use client'
 
// This is a Client Component (same as components in the `pages` directory)
// It receives data as props, has access to state and effects, and is
// prerendered on the server during the initial page load.
export default function HomePage({ recentPosts }) {
  return (
    <div>
      {recentPosts.map((post) => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  )
}
```

#### 2. 새로운 페이지 만들기
- `app` 폴더 내부에 `app/page.tsx` 파일을 만든다. 이 파일은 기본적으로 서버 컴포넌트이다.
- 1에서 만들었던 `home-page.tsx` 클라이언트 컴포넌트를 import 한다.
- 만약 기존의 `pages/index.tsx`에 data fetching 부분이 있다면, 이를 새로워진 data fetching API로 변환하여 서버 컴포넌트 내부로 이동해야 한다. (참고 : [data fetching upgrade guide](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration#step-6-migrating-data-fetching-methods))


``` tsx
// Import your Client Component
import HomePage from './home-page'
 
async function getPosts() {
  const res = await fetch('https://...')
  const posts = await res.json()
  return posts
}
 
export default async function Page() {
  // Fetch data directly in a Server Component
  const recentPosts = await getPosts()
  // Forward fetched data to your Client Component
  return <HomePage recentPosts={recentPosts} />
}
```
- 만약 `useRouter`를 이용했다면, 새로운 routing hooks으로 변경 필요 ([참고](https://nextjs.org/docs/app/api-reference/functions/use-router))


Step 5 부터는 다음 시간에....