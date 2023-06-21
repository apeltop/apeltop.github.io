---
title: SourceCommit 에 Git 연동
layout: doc
subtitle: SourceCommit 에 git repository 를 연동하기
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

## 목표
`뒤이어 작업할 Build, Deploy 를 위한 첫 걸음인 SourceCommit 에 Git 연동해보자`.

![6-1.png](/assets/img/ncloud-sourcepipeline/6-1.png)

## SourceCommit 이란
[SourceCommit](https://www.ncloud.com/product/devTools/sourceCommit) 은 Git 을 사용할 수 있게 해준다.
기존의 Git 과 다른 점은 [File Safer](https://guide.ncloud-docs.com/docs/security-security-6-2) 를 제공하여 악성코드 감염 여부 확인 등을 할 수 있다.
File Safer 의 경우 유료 서비스이지만 실제 회사에서 구축한다면 충분히 필요한 서비스이다. 문제가 터지는 것보다 영양제를 챙겨먹는 것이 낫다.

NCP(Naver Cloud Platform) 의 Sub Account 에 리포지토리 접근 권한 설정이 가능하다는 장점도 있다.
Github 환경이 아니라 하나의 리포지토리를 만들어 SourceCommit 과 연동하고 직원들과 SourceCommit 을 통해 공유하는 것도 하나의 방법이겠다.

## 만들기
Developer Tools > SourceCommit 에 들어간다.
그 후 **외부 리포지토리 복사**를 만든다. 새로운 리토지토리를 만들고 싶은 경우 리포지토리 생성을 하면 된다.

![6-2.png](/assets/img/ncloud-sourcepipeline/6-2.png)

리포지토리 이름과 Git URL 을 입력하고 하단의 **Git 연결 확인**을 누른다. 문제가 없으면 다음으로 넘어간다.

![6-3.png](/assets/img/ncloud-sourcepipeline/6-3.png)

그 다음 단계로는 보안상품 연동이다. 이는 위에서 언급하였던 [File Safer](https://guide.ncloud-docs.com/docs/security-security-6-2) 이다.
이번 포스팅에도 사용해볼까 하다가 File Filter 의 경우 1만회당 기준이라 1회만 사용해도 1~10000회 사용 금액인 10만원을 내야하기에 넘어가겠다..  

![6-4.png](/assets/img/ncloud-sourcepipeline/6-4.png)

![6-5.png](/assets/img/ncloud-sourcepipeline/6-5.png)

설정한 내용이 맞으면 **생성**을 누른다.

![6-6.png](/assets/img/ncloud-sourcepipeline/6-6.png)

SourceCommit 화면을 보면 방금 만든 리포지토리가 생성된 것을 확인할 수 있다.

![6-7.png](/assets/img/ncloud-sourcepipeline/6-7.png)

리포지토리 이름을 클릭하면 상세 정보를 볼 수 있다. 파일을 볼 수 있으며 Commit 도 확인할 수 있다. 

![6-8.png](/assets/img/ncloud-sourcepipeline/6-8.png)

![6-9.png](/assets/img/ncloud-sourcepipeline/6-9.png)




















