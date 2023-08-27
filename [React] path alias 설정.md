React 뿐만 아니라 js/ts 환경에서 개발을 하다보면 상대경로로 import 영역이 굉장히 지저분해진다.
그래서 `../../` 등을 `@/`와 같이 별명으로 설정하여 개발할 수 있는데, 매번 설정하는 방법을 검색하는게 귀찮아 정리해두려 한다.

## 개발 환경
프로젝트의 생성 템플릿에 따라 설정이 달라진다. (React, Typescript 환경을 가정한다.)

### 1. CRA
방법만 간단히 기술하자면,
1. 최상위 폴더에 `tsconfig.paths.json` 파일 생성
```json
// tsconfig.paths.json

{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

2. 기존 `tsconfig.json` 파일 내에 추가 (아마 여기까지만 하면 에디터의 오류는 없지만 실행 및 빌드가 안될 것이다.)
```json
// tsconfig.json

{
    ...
    "extends": "./tsconfig.paths.json"
}
```

3. `craco` 설치 및 `package.json` 설정
```
npm i -D @craco/craco
```
```json
// package.json
{
    ...
    // start, build 명령을 craco 통해서 실행하게 변경
    "scripts": {
        "start": "craco start",
        "build": "craco build",
        "test": "craco test",
        "eject": "craco eject"
    }
}
```

4. `react-app-alias` 설치
(`craco-alias` 패키지를 일반적으로 사용하지만 더 이상 관리되지 않는 패키지라서 다른 패키지를 이용, [참고](https://github.com/risen228/craco-alias))
```
npm i -D react-app-alias
```


5. 최상위 폴더에 `craco.config.js` 파일 생성
```js
const { CracoAliasPlugin } = require('react-app-alias')

const options = {} // default is empty for most cases

module.exports = {
  plugins: [
    {
      plugin: CracoAliasPlugin,
      options: {}
    }
  ]
}
```

이렇게 하면 끝!

### 2. Vite
이 경우에는 좀 더 간단하다.
1. `tsconfig.json` 설정 추가
```json
// tsconfig.json
{
    "compilerOptions": {
        ...
        "baseUrl": ".",
            "paths": {
              "@/*": ["./src/*"]
            },
        ...
    }
}
```

2. `vite.config.js` 설정
```js
import path from "path"; // 추가
...

export default defineConfig({
  ...
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  ...
});

```
참고로 초기 설정이라면 "path"를 import 하는 곳에서 오류가 날텐데 이는 타입 문제로 `@types/node`를 설치하면 해결된다.