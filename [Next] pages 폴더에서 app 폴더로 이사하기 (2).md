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