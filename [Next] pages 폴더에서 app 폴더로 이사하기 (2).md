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

### Step 1. 작성 중 ....
