![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/5h7cc5f54jjd4.png)



## AccessControlListNotSupported
React 웹을 S3 + Cloudfront로 배포할때 마주한 AccessControlListNotSupported 에러를 어떻게 해결할 수 있는지 알아보자.
사실 Cloudfront는 상관없으니 S3에만 해당되는 얘기가 되겠다.

## 상황
문제가 발생한 프로젝트의 상황은 아래와 같다.
1. github action의 `jakejarvis/s3-sync-action@v0.5.1` 이용해서 빌드된 결과물을 s3 sync하여 배포
2. github action에서 사용하는 IAM 계정은 S3에 대한 거의 대부분의 권한을 가짐
3. 배포에 사용되는 S3는 `ACL 비활성화(버킷 소유자 적용)`, `퍼블릭 액세스 차단 활성화`, `CloudFront 읽기 버킷 정책`으로 설정

## 해결..?
AccessControlListNotSupported 에러로 검색을 하면 거의 대부분이 ACL을 활성화하고, 퍼블릭 액세스 차단을 비활성화하여 문제를 해결하고 있다. 분명 그런식으로 해결해도 되겠지만, 왜 나는 그러기가 싫었을까..

## 어떻게?
나의 github action yaml 파일에 aws s3 sync 명령어의 인자로 `--acl public-read --follow-symlinks --delete`를 작성해놨었다.
![image](https://d1ccleacxg8gcm.cloudfront.net/lhjeong60/images/d1jbad5jdb26.png)

각각 설명하자면

- `--acl public-read` : 업로드 하는 객체에 대해 공개 읽기(public-read) ACL을 적용
- `--follow-symlinks` : sync 하려는 파일에 심볼릭 링크가 있을 경우, 링크가 아니라 링크가 가리키는 실제 파일을 업로드
- `--delete` : 원본(로컬)의 파일 목록과 동기화 (sync) 후, 버킷에만 남아있는 파일을 삭제

사실 `--acl public-read` 이게 제일 중요한데, 내가 올리려는 파일의 ACL을 수정하려고 하면 에러가 발생했던 것이다. 특히 해당 버킷의 경우에는 `ACL 비활성화(버킷 소유자 적용)`으로 적용되어있기 때문에 충돌이 있었으리라.
### 사건 해결~
해당 옵션을 `--acl bucket-owner-full-control`으로 수정하면 문제 해결!