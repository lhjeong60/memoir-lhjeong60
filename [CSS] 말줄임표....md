텍스트가 정해진 너비를 넘어가게 되면 ... 하고 말 줄임표를 적용해야할 때가 있다.

그럴때는 ellipsis 라는 키워드로 구글에 검색하면 되는데, 매번 검색하기 귀찮아 적어놓으려한다.


## CSS

``` css
.말줄임표 {
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
}
```

사실 이게 끝이다.

포인트는 `overflow: hidden` 과 `white-space: nowrap`를 같이 지정해줘야 한다.