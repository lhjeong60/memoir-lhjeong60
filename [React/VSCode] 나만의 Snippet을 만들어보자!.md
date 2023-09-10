최근 react-router 관련 강의를 보다가 강의자가 useState를 snippet(이하 스니펫)으로 간편하게 작성하는 것을 보고 충격을 먹었었다.

그동안 나는 react에서 state 하나를 선언하려면 아래와 같이 손수 타이핑 하고 있었다.

``` tsx
const [state, setState] = useState('');
```
고작 1줄이라지만, 같은 짓을 여러번하는 경우가 많아지다보면 티끌모아 빡치기 마련이다.

이런 부분들은 vscode에서 제공하는 사용자 정의 스니펫을 이용하면 훨씬 편하게 개발할 수 있다.
어떻게 나만의 스니펫을 만들어 보자.

## 들어가기 앞서..
내가 평소에 개발을 진행하며 가장 쓰기 귀찮은 것들을 나열해본다.

- ~/MyComponent/index.tsx 와 같이 폴더 이름을 컴포넌트 이름
- useState(하는 김에, useRef)
- custom hook

이 3가지는 꼭 편하게 쓰고 말테다.


## 어떻게 설정하나?
[vscode 공식](https://code.visualstudio.com/docs/editor/userdefinedsnippets#_create-your-own-snippets)을 보면 친절히 나와있다.

먼저 `cmd + shift + P`에서 `configure user snippets`을 선택한다.
![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/203c0hbdgc9i6.png)


그럼 아래와 같이 나오는데, 나에게 맞는 설정으로 시작하자, 내경우 Typescript JSX로 진행했다.
![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/h3ehj9d0c3g2.png)


아래와 같이 기본 json 파일이 나올 것이다.
``` json
{
  // Place your snippets for typescriptreact here. Each snippet is defined under a snippet name and has a prefix, body and
  // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
  // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the
  // same ids are connected.
  // Example:
  // "Print to console": {
  // 	"prefix": "log",
  // 	"body": [
  // 		"console.log('$1');",
  // 		"$2"
  // 	],
  // 	"description": "Log output to console"
  // }
}
```

요기를 이제 내마음대로 주무르면 되는데, 간단히 설명하자면
`prefix`: 스니펫을 사용할 때, 이용할 키워드라 생각하면 된다.
`body`: 실제로 적용되는 코드 조각을 각줄에 따라 문자열 배열로 설정하면 된다.
`description`: 스니펫을 사용할 때, intellisense에서 표시되는 설명이다.

그리고 또 하나 중요한게 바로 `Variables`이다. 
코드조각에서 사용할 수 있는 변수들이 몇가지 있는데, (현재 파일이름이나, 현재 디렉토리 이름 등) 이들을 잘 이용하면 훨씬더 편하게 사용할 수 있다.

[전체 목록](https://code.visualstudio.com/docs/editor/userdefinedsnippets#_variables)

하나하나 설명하기는 힘들고, (나도 잘 모르고) 내 스니펫을 만들어보자!

## 1. 폴더이름으로 컴포넌트 생성하기


변수 중, `TM_DIRECTORY`를 이용하면 현재 파일의 상대경로를 불러올 수 있는데, 이를 마지막 폴더 이름만 가져오려면 몇가지 변형이 필요하다.
[Variable Transform](https://code.visualstudio.com/docs/editor/userdefinedsnippets#_variable-transforms)에서 확인할 수 있듯, 정규식을 이용해 replace 비스무리하게 동작시킬 수 있다.

예를 들어 `~/src/components/Button/index.tsx` 파일에서 스니펫을 이용한다고 하면, 폴더이름을 가져오기 위해 아래와 같이 수정을 거쳐야한다.
`${TM_DIRECTORY}` : `~/src/components/Button/index.tsx`
`${TM_DIRECTORY/.+\\///gi}` : `~/src/components/Button/index.tsx`

대략 설명하자면 `${variable/바꾸고 싶은 것/바뀔 예정 문자열/}`의 느낌으로 이것저것 시도해보면 된다.
참고로 바꾸고 싶은 쪽에서는 regex를 이용하는데 `\/`는 json에서 `\\/`로 사용해야 되는 모양이더라.

아래는 내가 적용한 코드이다.
``` json
{
    "React Functional Component Export Props Interface FolderName": {
    "prefix": "tsrafcei",
    "body": [
      "import React from 'react'",
      "",
      "interface Props {}",
      "",
      "const ${TM_DIRECTORY/.+\\///gi} = (props: Props) => {",
      "  return (",
      "    <div>${TM_DIRECTORY/.+\\///gi}</div>",
      "  )",
      "}",
      "",
      "export default ${TM_DIRECTORY/.+\\///gi}"
    ],
    "description": "React 함수형 컴포넌트, interface Props, Export default, folderName"
  }
}

```


## 2. useState, useRef
useState를 선언할때 가장 빡치는 점은 setState에서 대문자를 섞어줘야한다는 것이다.
이또한 스니펫에서 구현이 가능하다.

``` json
// useState
{
    "prefix": "rstate",
    "body": [
      "const [$1, set${1/(.)/${1:/upcase}/}] = useState($2);",
    ]
}
```
$1, 2, 3, 등등은 탭인덱스라고 보면 되는데, 탭을 누를때마다 커서가 옮겨간다.
위처럼 적용을 하게 되면 처음에는 state, setstate와 같이 둘다 소문자로 입력 되지만, 탭을 눌러 $2로 옮겨가면 대문자로 바뀐다.

useRef는 비교적 쉽다. 기본값을 같이 설정해줘봤다. `${1:기본값}`
``` json
// useRef
{
    "React useRef": {
    "prefix": "rref",
    "body": [
      "const ${1:ref} = useRef(${2:null});",
    ]
  },
}
```


## 3. custom hook
커스텀 훅 또한 사실 크게 어려운 것이 없다. `useMyHook/index.tsx`와 같이 이용한다면 동일하게 폴더이름으로 가져오면 끝이다.

``` json
{
    "React Hook": {
    "prefix": "rhk",
    "body": [
      "import React from 'react';",
      "",
      "const ${TM_DIRECTORY/.+\\//$1/gi} = () => {",
      "}",
      "export default ${TM_DIRECTORY/.+\\//$1/gi};"
    ]
  }
}
```

## 결론
개발자는 귀찮은 것을 줄이는 것을 귀찮아하지 말아야한다고들 하더라.
아직 개발자가 되려면 멀었다.

