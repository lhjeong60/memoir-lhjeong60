## 이슈

지금 사용하고 있는 MEMOIR. 블로그 서비스는 Next.js로 개발하고 Vercel(무료 플랜)을 이용하여 배포하고 있다.
열심히 개발해서 배포해보니, SSR을 이용하는 페이지 로딩시 거의 1초에 가까운 로딩시간이 발생했다.

![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/e79g4afahe6jc.png)


## 원인 파악

SSR을 이용하는 함수에서 특별히 오래걸릴 일이 없었다.
로컬에서 동작시켰을때도 이질감 없었고, 기껏해야 API 요청/응답이었기 때문에 로직 관련 이슈는 아니라 생각했다.

그럼 무료 플랜이라 Next 서버가 너무 구린가? 
합리적인 의심이었지만 확인할 방법이 없었다.

그런데, 어라라? Washington D.C. ??
위 캡쳐에서 볼 수 있듯, Location이 워싱텅 DC 였다.


## 해결
Vercel 공식 자료에도 SSR의 Execution Time을 줄일 수 있는 방안을 몇가지 제시해놓았는데, 그 중 첫번째가 Location에 관련된 내용이었다.

[How do I lower my Serverless Function execution time?](https://vercel.com/guides/how-do-i-lower-my-serverless-function-execution-time)

결론부터 말하자면 Location 관련 설정 후, 약 50% 가량 Execution Duration이 감소했다.
![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/327j8dg6b793b.png)


## 설정 방법

Vercel의 배포 중인 프로젝트로 들어가서 Settings -> Functions 순으로 진입하면 된다.
![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/6a1ecg49903g.png)

설명에 나와 있는 것처럼, SSR 또는 SSG 함수 내에서 접근하는 자원과 가까이 설정하는 것이 좋으며, 
설정 뒤에는 재배포가 필요하다.