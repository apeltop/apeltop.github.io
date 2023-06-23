---
title: SourceDeploy 를 통해 배포하기
layout: doc
subtitle: SourceDeploy 를 통해 dev & real stage 에 따라 배포되도록 하기
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

## 목표
`SourceDeploy 를 통해 수동으로 배포하기보다 자동으로 배포되도록 하기`

![8-1.png](/assets/img/ncloud-sourcepipeline/8-1.png)

## SourceDeploy 이란
[SourceDeploy](https://www.ncloud.com/product/devTools/sourceDeploy) 는 배포 자동화를 위한 서비스이다.
SourceDeploy 를 통해 CD(Continuous Delivery) 를 구현할 수 있다.

여러 스테이지(e.g. real, test, dev 등)에 따라 배포를 다르게 진행할 수 있다. Sub Account 가 있을 경우 [배포 승인 요청](https://guide.ncloud-docs.com/docs/sourcedeploy-use-deploy)을 통해 배포를 진행할 수 있다.

## 만들기
### 목표
배포 전략에는 여러 가지 방법이 있지만 이번 예제에서 만들고 싶은 것은 지금까지 서버 2대를 사용하고 있는데 서버 1대에 먼저 배포를 진행하고 문제가 없을 경우 남은 한 대에 배포를 진행하고 싶다.

> - dev stage 에서는 server2 에 배포
> - real stage 에서는 server1 에 배포


### dev stage 만들기
Developer Tools > SourceDeploy 에 들어간다. 그 후 **배포 프로젝트 생성**을 클릭한다.

![8-2.png](/assets/img/ncloud-sourcepipeline/8-2.png)

프로젝트 이름을 입력해준다.

![8-3.png](/assets/img/ncloud-sourcepipeline/8-3.png)

배포 환경 설정 단계에서는 배포 타겟, 서버를 선택해줘야한다.

![8-4.png](/assets/img/ncloud-sourcepipeline/8-4.png)

배포 stage 의 경우 왼쪽 패널을 보면 dev 로 기본으로 선택되어있기 때문에 적용 서버로 server2 를 보내었다. 

![8-5.png](/assets/img/ncloud-sourcepipeline/8-5.png)

입력한 정보가 맞다면 **배포 프로젝트 생성**을 클릭한다.

![8-6.png](/assets/img/ncloud-sourcepipeline/8-6.png)

그럼 sample-deploy 라는 프로젝트가 생성된 것을 확인할 수 있다.
프로젝트를 선택하면 아래 이미지와 같이 여러 정보들이 나타난다. 

배포 시나리오 에서 **생성** 을 클릭한다.

![8-7.png](/assets/img/ncloud-sourcepipeline/8-7.png)

배포 시나리오 이름을 입력해준다.

![8-8.png](/assets/img/ncloud-sourcepipeline/8-8.png)

배포 과정을 선택해준다. 이번 예제에서는 **순차 배포** 를 선택한다.

![8-9.png](/assets/img/ncloud-sourcepipeline/8-9.png)

배포 파일 위치는 이전글인 [SourceBuild 를 통해 배포 파일 생성](/docs/ncloud-sourcepipeline/2023-06-22-source-build/)에서 Source Build 를 통해 파일 업로드를 진행하였기에 **Source Build** 선택하여 진행한다. 

![8-10.png](/assets/img/ncloud-sourcepipeline/8-10.png)

배포 명령어 설정은 3단계로 나누어져 있다.

- 배포 전 실행
- 파일 배포
- 배포 후 실행

파일 배포의 경우 input 의 placeholder 를 보면 **sample_webapp.jar.zip** 을 기준으로 작성하라고 나와있다. 그러기에 / 만 입력하였다.

배포 후 실행은 서버가 종료하고 시작되도록 하였다. 실제 현업이라면 restart.sh 를 만들어서 관리하는게 더 나을 것이다.

![8-11.png](/assets/img/ncloud-sourcepipeline/8-11.png)

입력한 정보가 맞는지 확인한 후 **배포 시나리오 생성**을 클릭한다.

![8-12.png](/assets/img/ncloud-sourcepipeline/8-12.png)

다시 CloudDeploy 화면으로 돌아오면 배포 시나리오가 생성된 것을 확인할 수 있다.

![8-13.png](/assets/img/ncloud-sourcepipeline/8-13.png)

### real stage 만들기
dev stage 를 만들었으니 real stage 를 만들어보자. 예제에서는 stage 에 따라 다른 액션을 취하지 않기 때문에 같은 방법으로 만든다.

**+** 버튼 눌러 real stage 를 만들어준다.

![8-27.png](/assets/img/ncloud-sourcepipeline/8-27.png)

배포 환경 **생성** 버튼을 눌러준다.

![8-28.png](/assets/img/ncloud-sourcepipeline/8-28.png)

이번엔 서버1 을 적용 서버로 옮기어 준다. 

![8-29.png](/assets/img/ncloud-sourcepipeline/8-29.png)

그 후 **적용** 버튼을 눌러준다.

![8-30.png](/assets/img/ncloud-sourcepipeline/8-30.png)

배포 시나리오 생성을 진행해보자.

![8-31.png](/assets/img/ncloud-sourcepipeline/8-31.png)

배포 시나리오 이름을 입력해준다.

![8-32.png](/assets/img/ncloud-sourcepipeline/8-32.png)

배포 전략 설정, 배포 파일 설정은 이전과 동일하기에 넘어갔다.
배포 명령어 설정 또한 같다.

![8-33.png](/assets/img/ncloud-sourcepipeline/8-33.png)

설정한 정보가 맞다면 **배포 시나리오 생성**을 진행한다.

![8-34.png](/assets/img/ncloud-sourcepipeline/8-34.png)

### SourceDeploy 에이전트 설치
이제 배포할 준비가 끝난걸까? 그렇지 않다. **배포 프로젝트 생성** 단계에서 보았듯 배포 서버에 **SourceDeploy 용 에이전트**를 설치해야한다.

참고 [에이전트 설치 및 관리](https://guide.ncloud-docs.com/docs/devtools-devtools-4-4) 

![8-14.png](/assets/img/ncloud-sourcepipeline/8-14.png)

#### API Key 발급
오른쪽 상단의 계정을 클릭하고 **계정 관리**를 클릭한다.

![8-15.png](/assets/img/ncloud-sourcepipeline/8-15.png)

비밀번호를 입력해준다.

![8-16.png](/assets/img/ncloud-sourcepipeline/8-16.png)

계정 관리 > 인증키 관리 에 들어가서 **신규 API 인증키 생성** 을 클릭한다.

![8-17.png](/assets/img/ncloud-sourcepipeline/8-17.png)

그러면 바로 인증키가 생성되었다고 alert 이 나타난다.

![8-18.png](/assets/img/ncloud-sourcepipeline/8-18.png)

그 후 API 인증키 관리에 보면 인증 키가 생성된 것을 확인할 수 있다.
Secret Key 의 경우 보기를 눌러 확인하면 된다.

![8-19.png](/assets/img/ncloud-sourcepipeline/8-19.png)

#### 에이전트 설치
이제 서버에 SSH 로 접속해 에이전트를 설치해야한다.

서버1 & 서버2 모두 설치가 되어야한다.

[에이전트 설치 및 관리](https://guide.ncloud-docs.com/docs/devtools-devtools-4-4) - 서버 접속 후 작성에서 적혀있는 방법은 아래와 같다.

```bash
echo $'NCP_ACCESS_KEY=accesskey\nNCP_SECRET_KEY=secretkey' > /opt/NCP_AUTH_KEY
chmod 400 /opt/NCP_AUTH_KEY
wget  Agent 다운로드 주소
chmod 755 install
./install
rm -rf install
```

환경(VPC, Classic)과 리전(한국, 싱가포르)에 따라 다운로드 주소가 다르니 참고해서 다운로드 받아야한다.

![8-20.png](/assets/img/ncloud-sourcepipeline/8-20.png)

API 인증키 정보를 가지고 위의 명령어를 입력하면 에이전트가 설치된다.

```bash
echo $'NCP_ACCESS_KEY=accesskey\nNCP_SECRET_KEY=secretkey' > /opt/NCP_AUTH_KEY
chmod 400 /opt/NCP_AUTH_KEY
wget  https://sourcedeploy-agent.apigw.ntruss.com/agent/v2/download/install
chmod 755 install
./install
rm -rf install
```

그럼 아래와 같이 설치가 진행이 된다.

![8-21.png](/assets/img/ncloud-sourcepipeline/8-21.png)

이제 에이전트 설치가 되었으니 배포할 준비가 되었다.

#### 배포 테스트
실제로는 server1 & server2 모두를 확인하면 해야하지만 예제이니 한 가지만 확인해보자.

기존 server2 에 접속해서 정보를 보자.
만약 배포가 성공적으로 진행이 된다면 중간에 UUID 값이 바뀌어 있을 것이다.
재실행되었을 것이기 때문이다.

![8-22.png](/assets/img/ncloud-sourcepipeline/8-22.png)

SourceDeploy 에서 sample-deploy 를 클릭하면 상세 보기 화면이 나온다.
**배포 시작하기** 버튼을 누른다.

![8-23.png](/assets/img/ncloud-sourcepipeline/8-23.png)

작업 결과 탭에 들어가면 배포가 진행되는 것을 볼 수 있다.

![8-24.png](/assets/img/ncloud-sourcepipeline/8-24.png)

조금 기다리면 배포가 완료된 것을 확인할 수 있다.

![8-25.png](/assets/img/ncloud-sourcepipeline/8-25.png)

이제 다시 서버에 접속해서 정보를 보면 UUID 값이 바뀐 것을 확인할 수 있다.

![8-26.png](/assets/img/ncloud-sourcepipeline/8-26.png)

이로써 SourceCommit 으로 소스를 커밋하면 SourceBuild 를 통해 배포 파일이 생성되고 SourceDeploy 를 통해 배포를 진행할 수 있게 되었다.  

































