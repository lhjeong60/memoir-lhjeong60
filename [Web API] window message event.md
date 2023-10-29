프론트 개발을 하다보면 피치 못하게 `window.open`으로 새창을 열때가 있다.

그럴때 기존 페이지와 새로 생긴 window와 어떻게 데이터를 주고 받는지 알아보자.

## [Window: message event](https://developer.mozilla.org/en-US/docs/Web/API/Window/message_event)

window에는 message라는 이벤트가 있는데, 이는 window가 message를 수신했을 때 이벤트가 발생한다. 
(이 이벤트는 버블링 되지 않고, 취소될 수 없다.)

그렇다면 message가 수신될 때는 뭘까? 바로 다른 window에서 `postMessage`를 호출했을때를 말한다.

## [Window.postMessage](https://developer.mozilla.org/ko/docs/Web/API/Window/postMessage)
단순하게 말하면 말그대로 메시지를 보내는 함수다.

``` js
targetWindow.postMessage(message, targetOrigin, [transfer]);
```
이 함수가 실행되는 window가 보내는 쪽, `targetWindow`가 받는 쪽이다.
`targetWindow`를 어떻게 아냐고? `window.open`으로 열었다면 `window.opener`가 `targetWindow`일 것이다.

message에 보내고 싶은 데이터를 넣으면 알아서 직렬화 된다고 하니 안심하고 message를 보내자. 
([The structured clone algorithm](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) 이용해 직렬화)

그럼 `targetOrigin`은?

`"*"`이거나 URI로 지정해야한다.

한가지 알아둬야할 주의사항이 있는데, 이 `postMessage` 함수는 `targetWindow`의 스키마, 호스트, 포트까지 모두 `targetOrigin`과 같아야 정상적으로 실행된다. 아무래도 보안 때문에 걸려있는 제약으로 보인다.

최종적으로 아래와 같이 사용할 수 있겠다.
```js
const message = {
    isSuccess: true,
    // blah blah blah
}

window.opener.postMessage(message, "https://www.mem0ir.com")
```

받는 window에서는 message 이벤트에 핸들러를 등록해 처리하면 된다.
``` js
window.addEventHandler("addEventListener", function(e) {
    // 보냈던 message는 event.data에 담겨있다.
    if (e?.data?.isSuccess) {
        childWindow.close();
    }
})

```


## 왜 이런 걸 알아보게 되었는가?
최근 직장에서 NICE 본인인증 연동할 일이 생겼다.

[연동가이드]([링크](https://www.niceapi.co.kr/#/apis/guide?ctgrCd=0100&prdId=38&prdNm=%ED%9C%B4%EB%8C%80%ED%8F%B0%EB%B3%B8%EC%9D%B8%ED%99%95%EC%9D%B8))를 보면 알겠지만 프론트엔드 입장에서는 크게 할 일이 없다. 
지정된 값들을 form 태그에 담고, form 태그의 target을 window.open으로 열린 window의 이름으로 지정하면 우리가 자주 사용하는 PASS 인증 팝업창이 뜨게 된다.

본인인증이 모두 완료되면, 팝업 window에서 미리 지정해둔 우리측 서버 URL(이하 `returnURL`)로 인증 정보를 담아 HTTP 요청을 보내게 된다.
이 부분이 좀 특이한데 요청을 보내는 방식이 팝업 window의 URL 주소를 `returnURL`로 redirect 시켜버리는 방식으로 진행된다.

내가 겪었던 문제는, 아무런 처리도 해놓지 않으면 본인인증이 완료되어도 팝업창이 자동으로 닫히지 않는다는 사실..!
위에서 말한 `returnURL`로 리다이렉트된 팝업창에서 머물러 있기 때문이다.

## 어떻게 해결했냐?
서버에서 `returnURL`의 응답으로 html을 내려주기로 했다. 리다이렉트이기 때문에 팝업창을 닫는 코드를 html로 내려주면 만사 OK이기 때문이다.

사실 아래와 같이 팝업창 자신을 닫기만 해도 문제는 해결된다.
```
<html>
    ...
    <script>
        window.close();
    </script>
</html>
```

하지만 본인인증이 완료된 시점을 기존 페이지에서 알아야하고, 본인인증 완료된 직후에 인증정보가 필요할 수도 있기 때문에 팝업창에서 기존 페이지로 데이터 전달이 필요했던 것이었다.

따라서, 팝업창에서 기존페이지로 메시지(성공여부, 필요 인증정보 등)를 전송하고, 기존 페이지에서 메시지 여부에 따라 팝업 window를 끄는 방식으로 문제를 해결할 수 있었다.


## 마치며..
나이스 본인인증 연동은 많이들 사용하는 서비스인것 같은데도 개발 연동 관련 레퍼런스가 없었어서 꽤나 당황했었는데, 잘 해결되어서 다행이다.