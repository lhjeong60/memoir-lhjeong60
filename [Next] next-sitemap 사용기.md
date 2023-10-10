MEMOIR.의 sitemap 파일을 만들기위해 next-sitemap를 사용했다.

## sitemap 이란?
> 사이트맵은 사이트에 있는 페이지, 동영상 및 기타 파일과 그 관계에 관한 정보를 제공하는 파일입니다.
>
> 출처 : [사이트맵 알아보기](https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview?hl=ko)

뭐 대략 검색엔진이 쉽게 페이지들을 탐색하기 위한 일종의 도구 같은거라고 볼 수 있겠다.

없어도 크게 상관없지만 없는 것보단 있는게 낫다고 언급하고 있다.
~~(실제로 500페이지 이하의 웹 서비스는 필요없다고도 하지만... MEMOIR.는 월클 예정이니까?!)~~

## next-sitemap
이 라이브러리는 빌드시점에 page들을 탐색해서 sitemap.xml 파일을 public 폴더에 넣어준다.
[next-sitemap github](https://github.com/iamvishnusankar/next-sitemap)를 참고하면 아주 손쉽게 사용할 수 있다.

같이 따라해보자?

### 1. 설치
`yarn add next-sitemap` `npm install next-sitemap` 등등

### 2. 설정 파일 만들기
root 경로에 `next-sitemap.config.js` 파일 생성 후, 아래와 같이 저장한다.

``` js
/** @type {import('next-sitemap').IConfig} */
module.exports = {
  siteUrl: process.env.SITE_URL || 'https://example.com',
  generateRobotsTxt: true, // (optional)
  // ...other options
}
```
당연히 `process.env.SITE_URL` 이나 `https://example.com`은 입맛에 바꿔야겠지?
여러 옵션들이 굉장히 많아보이니, 추후에 필요해지면 써보도록 하자


### 3. 사이트맵 생성!
사이트맵은 빌드가 완료된 후에 생성되기 때문에 아래와 같이 `package.json` 파일에 postbuild 명령어를 지정해주자
``` json
{
  "scripts": {
      ...
      "build": "next build",
      "postbuild": "next-sitemap"
      ...
  }
}
```
이런 식으로 지정하면 빌드 후에 알아서 `next-sitemap.config.js`을 읽어서 sitemap을 만들어준다.
설정 파일이름은 다른 식으로 해도 되는데 그럴 경우 postbuild 명령어에 별도 옵션으로 파일이름을 지정해줘야한다. 
(귀찮으니 시키는대로 하자)

위 설정을 모두 진행하고 빌드를 진행하면 public 폴더에 짠!하고 sitemap.xml 파일이 생성된다.


## SSR 대응은 어떻게...?
위 방식으로 만들어진 sitemap.xml 파일을 뒤져보면 알겠지만, dynamic route에 대한 URL들은 모두 제외되어 있다.
URL이 어떻게 될지 모르니 알 수가 없었겠지, 어찌보면 당연한 결과다.

이를 위해서는 동적으로 URL 목록들을 만들어서 뱉어줘야한다.
당연하게도 이미 github에 필요한 설명들이 있어서 참고하여 진행해보자. 
[next-sitemap : Generating dynamic/server-side-sitemaps](https://github.com/iamvishnusankar/next-sitemap#generating-dynamicserver-side-sitemaps)


### 원리
기본적으로 생성된 sitemap.xml은 파편화(이런 표현이 맞는진 모르겠지만)된 sitemap-0.xml을 호출한다.
그와 동일하게 sitemap.xml에서는 다른 xml 파일, 예를 들어 server-sitemap-index.xml을 호출하고 이 응답으로 최신의 URL 목록들을 뱉어주면 만사 OK다.
(게다가 이 server-sitemap-index.xml은 따로 서버 내에 파일을 저장하는 것이 아닌 xml 데이터 자체로 응답을 준다.)

정리하자면 
sitemap.xml -> server-sitemap-index.xml 호출 -> next 서버의 server-sitemap-index.xml 라우트에서 최신 URL 목록 만들어서 응답
(최신 URL 목록 만들기에는 우리 백엔드로의 요청이 동반된다.)


### 따라해볼까?
- app directory
``` ts
// app/server-sitemap-index.xml/route.ts
import { getServerSideSitemapIndex } from 'next-sitemap'

export async function GET(request: Request) {
  // url 목록 불러오기
  const urls: string[] = await fetch('https//example.com/api')

  return getServerSideSitemapIndex(urls)
}
```

- pages directory
``` ts
// pages/server-sitemap-index.xml/index.tsx
import { getServerSideSitemapIndexLegacy } from 'next-sitemap'
import { GetServerSideProps } from 'next'

export const getServerSideProps: GetServerSideProps = async (ctx) => {
  // url 목록 불러오기
  const urls: string[] = await fetch('https//example.com/api')

  return getServerSideSitemapIndexLegacy(ctx, urls)
}

// Default export to prevent next.js errors
export default function SitemapIndex() {}
```

### 마무리 설정

``` js
// next-sitemap.config.js

/** @type {import('next-sitemap').IConfig} */
module.exports = {
  siteUrl: 'https://example.com',
  generateRobotsTxt: true,
  exclude: ['/server-sitemap-index.xml'], // <= exclude here
  robotsTxtOptions: {
    additionalSitemaps: [
      'https://example.com/server-sitemap-index.xml', // <==== Add here
    ],
  },
}
```


## 결과
시키는 대로만 하면 다 잘 되더이다?

![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/ge20d84dg80ba.png)

