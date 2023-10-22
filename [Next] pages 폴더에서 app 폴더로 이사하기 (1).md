그동안 next.js 13 버전을 쓰면서도 pages 폴더 라우팅 방식을 그대로 고집하고 있었는데, 이제는 슬슬 app 라우팅 방식으로 변경해야 할 때가 온 것이다.

변경하는 일이 크게 어렵지 않을 것 같아 보였는데, 생각보다 쉽지 않아 보여서 과정을 기록해두려 한다.

### 공식 가이드
Next 공식 문서에 pages 폴더에서 app 폴더로 migrate 하는 방법이 나와있다.
[migrating-from-pages-to-app](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration#migrating-from-pages-to-app)
기본적으로 이 문서를 따라가며(번역해가며) 진행할 예정이다.

먼저 공식 문서를 살펴 보자

## Step 1 : app 폴더 생성
일단은 next 버전을 업데이트 한다. ((Next.js 버전 13.4 이상 필요))
```
npm install next@latest
```
그리고  `src/app` 폴더를 만든다. 참 쉽다 그죠~?


## Step 2 : root layout 생성
`app` 폴더 내부에 `app/layout.tsx` 새 파일을 만듭니다.

이 파일에서 적용된 레이아웃은 `app` 폴더 내부에서 공통적으로 적용된다.

이 root layout에 관한 몇가지 설명을 번역해보자면,
- `app` 폴더는 반드시 이 `app/layout.tsx` 파일을 포함해야 한다.
- `app/layout.tsx`은 반드시 `<html>`와 `<body>` 태그를 포함해야한다. (next에서 자동으로 생성하지 않기 때문에)
- 이 `app/layout.tsx` 파일이 기존의 `pages/_app.tsx`과 `pages/_document.tsx` 파일을 대체한다.
- `app/layout.tsx`은 `.js`, `.jsx`, `.tsx` 파일들로 작성 가능하다.

그리고 step 3에 나올 예정이지만, `next/head`가 deprecated 될 에정이기 때문에, 이전에 `pages/_document.tsx`에 있던 `<Head>` 컴포넌트 내의 내용들을 [Metadata API](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)로 변경해야 한다.


``` tsx
// Metadata API example
import { Metadata } from 'next'
 
export const metadata: Metadata = {
  title: 'Home',
  description: 'Welcome to Next.js',
}
```

`_document.tsx`와 `_app.tsx`의 내용들을 바로 `app/layout.tsx` 파일에 복사해도 되지만, 이 `app/layout.tsx`는 `app` 폴더에만 영향을 미치고 `pages`폴더에는 아무 영향 없기 때문에 migration을 진행하는 동안에는 `_document.tsx`와 `_app.tsx` 그대로 두는게 좋다.

### `getLayout()` 패턴은 어떻게?
app 라우팅에서는 [nesting Layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#nesting-layouts)를 적용하면 된다.

#### Before
``` tsx
// components/DashboardLayout.js
export default function DashboardLayout({ children }) {
  return (
    <div>
      <h2>My Dashboard</h2>
      {children}
    </div>
  )
}
```

``` tsx
// pages/dashboard/index.js
import DashboardLayout from '../components/DashboardLayout'
 
export default function Page() {
  return <p>My Page</p>
}
 
Page.getLayout = function getLayout(page) {
  return <DashboardLayout>{page}</DashboardLayout>
}
```

#### After
``` tsx
// app/dashboard/page.js
export default function Page() {
  return <p>My Page</p>
}
```

``` tsx
// app/dashboard/DashboardLayout.js
'use client' // this directive should be at top of the file, before any imports.
 
// This is a Client Component
export default function DashboardLayout({ children }) {
  return (
    <div>
      <h2>My Dashboard</h2>
      {children}
    </div>
  )
}
```

``` tsx
// app/dashboard/layout.js
import DashboardLayout from './DashboardLayout'
 
// This is a Server Component
export default function Layout({ children }) {
  return <DashboardLayout>{children}</DashboardLayout>
}
```
기존의 레이아웃에서 사용자와 상호작용하지 않는 부분은 서버 컴포넌트인 layout 파일로 이동시켜 자바스키립트 번들 사이즈를 줄일 수 있다.

참고로 root layout이 아닌 layout 파일에서는 `<html>`와 `<body>` 태그를 포함하면 안된다.


## 다음 이 시간에..
생각보다 쓸 내용이 많아서 step 3 부터는 다음 글에서 확인해보자..