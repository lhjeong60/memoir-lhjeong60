이왕이면 Polling 방식을 선호하지는 않지만, 어쩔 수 없이 써야하는 순간들이 있어서 정리해두려 한다.


## Polling.. 그게 뭔데?
![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/heb6c19d98jd.png)

예를 들어 실시간으로 업데이트 되는 내 계좌 입금내역 페이지를 만든다고 생각해보자.

페이지에서는 입금시점에 맞춰서 입급내역 GET 요청을 보낼 수가 없다. 
사람들이 입금하는 시기는 다 제각각이고 이는 우리 개발자의 관할 밖에서 진행되는 일이기 때문이다.

그럴때 주기적으로 HTTP 요청을 보내는 것으로 실시간 업데이트가 되는 것처럼 보이게 할 수 있다.
이 "주기적으로 요청을 보내는 일"을 Polling이라고 한다.

실제로 API 요청을 보내는 것이기 때문에 당연히 이 방식은 서버의 부담을 동반한다.
(이 때문에 나는 별로 선호하지 않는다.)

## 그거 어떻게 하는건데..
결국엔 "주기적으로 HTTP 요청을 보내는 것"뿐이라 구현이 어렵진 않다.

###  쌩으로 구현
1초에 한번 보내는 코드라고 한다면 아래와 같이 구현할 수 있겠다.

``` ts
let intervalId = -1;
const intervalTime = 1000;

intervalId = setInterval(() => {
    fetch("/some/api").then((res) => {
        if (폴링 중단?) {
            clearInterval(intervalId)
        }
    })
}, intervalTime)

```
쉽지만 버그도 많을 것 같다...

다른 신경쓸 것도 많아보여서 진짜 간단한게 아니면 아래 방법을 이용하려 한다.

### tanstack query (구 react-query)
useQuery의 옵션 중에 enabled와 refetchInterval을 이용하면 쉽게 사용가능하다. [docs](https://tanstack.com/query/v4/docs/react/reference/useQuery)

``` ts

const [polling, setPolling] = useState(true);
const { data } = useQuery({
    queryKey: ["fetchSomething", "polling.."],
    queryFn: () => fetch("/some/api")
    enabled: polling,
    refetchInterval: 1000
})

useEffect(() => {
    // 특정 조건에서 Polling 중단
    if (data.isDone) {
        setPolling(false)
    }
}, [data])

```

아래 velog 글을 아주 많이 참고했다.
[[JS][RxJS] 3가지 방식으로 Polling 구현하기: JS, React-Query, RxJS (feat. abortController)](https://velog.io/@gyutato/3%EA%B0%80%EC%A7%80-%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C-Polling-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-JS-React-Query-RxJS-feat.-abortController)

RxJS를 이용하거나, 요청을 취소(abortController)하는 고급기술도 나오는데 필요해지면 다시 써보도록하자.
