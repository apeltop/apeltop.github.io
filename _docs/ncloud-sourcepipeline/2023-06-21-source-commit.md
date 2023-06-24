---
title: SourceCommit 에 Git 연동
layout: doc
subtitle: SourceCommit 에 git repository 를 연동하기
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

## 목표
`SourceCommit 에 Git 연동 후 SourceCommit git 사용하기`

![6-1.png](/assets/img/ncloud-sourcepipeline/6-1.png)

## SourceCommit 이란
[SourceCommit](https://www.ncloud.com/product/devTools/sourceCommit) 은 Git 을 사용할 수 있게 해준다.
기존의 Git 과 다른 점은 [File Safer](https://guide.ncloud-docs.com/docs/security-security-6-2) 를 제공하여 악성코드 감염 여부 확인 등을 할 수 있다.
File Safer 의 경우 유료 서비스이지만 실제 회사에서 구축한다면 충분히 필요한 서비스이다. 문제가 터지는 것보다 영양제를 챙겨먹는 것이 낫다.

NCP(Naver Cloud Platform) 의 Sub Account 에 리포지토리 접근 권한 설정이 가능하다는 장점도 있다.
Github 환경이 아니라 하나의 리포지토리를 만들어 SourceCommit 과 연동하고 직원들과 SourceCommit 을 통해 공유하는 것도 하나의 방법이겠다.

## 만들기
### 기존 Git 과 SourceCommit 연동 
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

### SourceCommit 리포지토리 사용하기
기존 git 에 push 를 하면 SourceCommit 에서 연동되어 보이지 않는다.
즉 SourceCommit 이 기존 리포지토리 연동할 때는 기존 정보를 가져올 뿐 동기화를 하지 않는다는 것이다.
그러므로 SourceCommit 에서 관리하는 Git 을 사용하려면 SourceCommit 리포지토리를 clone 하여 사용해야한다.

**GIT 계정 / Git SSH 설정** 버튼 클릭한다. 

더 자세한 내용은 공식 문서 [Git 클라이언트 사용](https://guide.ncloud-docs.com/docs/sourcecommit-use-client) 을 참고하면 된다.

![6-10.png](/assets/img/ncloud-sourcepipeline/6-10.png)

그 후 GIT SSH 탭에 들어간다.

![6-11.png](/assets/img/ncloud-sourcepipeline/6-11.png)

SSH 키가 있어야 하기에 `ssh-keygen` 을 통해 키를 생성한다.

```bash
ssh-keygen
```

그럼 키를 만들 위치와 암호를 입력해야하는데 암호 없이 진행하였다.

키 위치는 /Users/`사용자`/.ssh/id_sourcecoomit 로 입력하였다.

그러면 키 위치에 id_sourcecoomit 과 id_sourcecoomit.pub 파일이 생성된다.
그 중 id_sourcecoomit.pub 파일을 열어서 내용을 복사하여 붙여넣는다.

![6-12.png](/assets/img/ncloud-sourcepipeline/6-12.png)

그 후 등록을 하면 퍼블릭 키가 등록이 된다.
여기서 SSH 키는 뒤 단계에서 사용할 것이기에 따로 기록해둔다.

![6-13.png](/assets/img/ncloud-sourcepipeline/6-13.png)

.ssh/config 파일을 수정한다.

```bash
vi ~/.ssh/config
```

아래 내용을 추가한다.
User 에 해당하는 부분이 SSH 키 이름을 입력해주면 된다.
```bash
Host devtools.ncloud.com
User [콘솔에 등록된 SSH 키 값]
IdentityFile ~/.ssh/id_sourcecommit
```

그 후 프로젝트를 새로만들 때 아래 구문을 사용하여 Clone 을 해준다.
.ssh/config 설정을 통해 알아서 SSH 키를 사용하여 Clone 을 하게 된다.

```bash
git clone [ssh Git URL, 예: ssh://devtools.ncloud.com/**********.git]
```

![6-14.png](/assets/img/ncloud-sourcepipeline/6-14.png)

새로 만들어진 프로젝트를 IDE 를 통해 다시 열어 Git > Manage Remotes 를 클릭해본다.

![6-15.png](/assets/img/ncloud-sourcepipeline/6-15.png)

그럼 원격 저장소 URL 이 SourceCommit 리포지토리 URL 로 변경된 것을 확인할 수 있다.

![6-16.png](/assets/img/ncloud-sourcepipeline/6-16.png)

이제는 SourceCommit 리포지토리와 연동된 프로젝트로 작업할 것이다. 











