---
title: Server 2 만들기
layout: doc
subtitle: NCloud 에서 서버 1개 만든 후 Spring Boot 실행하기
author: sunshine@ptokos.com
tags: [ncloud]
comments: true
---

## 목표
`서버 1과 같은 구성으로 서버 2를 생성한 후 웹으로 접속해본다.`

[Server 1 만들기](/docs/ncloud-sourcepipeline/2023-06-13-create-server-1/)에 이어 Server 2를 만들어보자.

### 결과물
![3-1.png](/assets/img/ncloud-sourcepipeline/3-1.png)

## 만들기
### Server Image
[Server 1 만들기](/docs/ncloud-sourcepipeline/2023-06-13-create-server-1/) 와 같이 서버를 만들 수 있다.
그러나 여기서 문제점은 구동 환경은 같은데 다시 자바를 설치하는 등 중복적인 작업을 해야한다는 것이다. 이와 같은 것을 피할 수 있는 방법으로 서버 이미지가 있다.

참고 [Ncloud 사용 가이드 Server Image](https://guide.ncloud-docs.com/docs/server-serverimage-classic)

서버 이미지는 현재 OS 상태를 저장해서 다른 서버 생성 시 같은 환경으로 만들 수 있도록한다. 이를 이용하면 서버 1과 같은 구성으로 서버 2를 만들 수 있다.

서버 이미지는 크게 2가지로 나눌 수 있다.
- Public Image
- Custom Image

#### Public Image
Public Image 은 말 그대로 공개된 이미지이다. Ncloud 에서도 다양한 운영체제와 프로그램이 설치된 이미지를 제공한다.

참고 [Server Image 의 LAMP](https://guide.ncloud-docs.com/docs/server-lamp)

![3-2.png](/assets/img/ncloud-sourcepipeline/3-2.png)

#### Custom Image
Custom Image 는 사용자가 직접 만든 이미지이다. Public Image 를 기반으로 설치한 프로그램이나 설정을 추가한 후 Custom Image 로 만들 수 있다.

server1 은 centos 의 이미지를 사용하여 만들어졌지만 server2는 server1 의 이미지를 가지고 만들 것이다.

#### Server 1 이미지 생성

Server 에 들어가 기존에 있던 **server1** 을 선택하고 **서버 관리 및 설정 변경**을 클릭한 후 **내 서버 이미지 생성**을 클릭한다.

![3-3.png](/assets/img/ncloud-sourcepipeline/3-3.png)

마우스 호버 시 친절한 설명이 나온다.

![3-4.png](/assets/img/ncloud-sourcepipeline/3-4.png)

서버 이미지 이름은 sample-image-1 로 입력하고 선택사항인 메모에 간단히 입력 후 다음을 누른다.

![3-5.png](/assets/img/ncloud-sourcepipeline/3-5.png)

최종 확인 단계에서 기본 정보를 확인 후 맞다면 **생성**을 클릭한다.

![3-6.png](/assets/img/ncloud-sourcepipeline/3-6.png)

Server > Server Image 에 들어가보면 원본 서버 이름이 server-1 이며 서버 이미지 이름은 **sample-image-1** 인 이미지가 생성중인 것을 볼 수 있다.

![3-7.png](/assets/img/ncloud-sourcepipeline/3-7.png)

생각보다는 금방 이미지가 생성이 되었다.

![3-8.png](/assets/img/ncloud-sourcepipeline/3-8.png)

### Server2 서버 생성
만들어진 서버 이미지를 클릭하고 서버 생성을 클릭해준다.

![3-9.png](/assets/img/ncloud-sourcepipeline/3-9.png)

server1 서버 생성과 다를 것 없이 서버 생성의 필요한 기본적인 정보를 입력해준다.

![3-10.png](/assets/img/ncloud-sourcepipeline/3-10.png)

인증키의 경우 따로 관리하려면 새로운 인증키 생성을 선택하면 된다.
이번 예제에서는 server1-key 를 사용하도록 한다.

![3-11.png](/assets/img/ncloud-sourcepipeline/3-11.png)

ACG(Access Control Group) 의 경우도 server1 과 같은 ACG 를 사용하도록 하였다.

![3-12.png](/assets/img/ncloud-sourcepipeline/3-12.png)

최종 확인 단계에서 정보를 확인하고 맞다면 **서버 생성** 버튼을 클릭한다.

![3-13.png](/assets/img/ncloud-sourcepipeline/3-13.png)

Server 목록을 보면 server-2 가 생성중인 것을 볼 수 있다.

![3-14.png](/assets/img/ncloud-sourcepipeline/3-14.png)

조금 기다리면 server2 가 운영중인 상태 값을 볼 수 있다.

![3-15.png](/assets/img/ncloud-sourcepipeline/3-15.png)

### 공인 IP 등록
server2 에서도 공인 IP 를 할당 받아야 외부에서 접속을 할 수 있기 때문에 공인 IP 를 할당 받도록 한다.

![3-16.png](/assets/img/ncloud-sourcepipeline/3-16.png)

적용 서버 선택에서 server-2 를 선택해준다.

![3-17.png](/assets/img/ncloud-sourcepipeline/3-17.png)

### 포트 포워딩 설정
server1 에 10022 포트를 할당하였기 때문에 server2 에 10023 을 할당하였다.

![3-18.png](/assets/img/ncloud-sourcepipeline/3-18.png)

### 관리자 비밀번호 확인
server1 을 서버 이미지를 만들어서 server2를 만들었다고해서 root 비밀번호까지 같은 것은 아니다.

![3-19.png](/assets/img/ncloud-sourcepipeline/3-19.png)

server2 의 비밀번호를 확인해준다.

![3-20.png](/assets/img/ncloud-sourcepipeline/3-20.png)

### server2 구축 확인
FileZilla 를 이용하여 server2 에 접속해보면 server1 과 같은 구성으로 구축이 되어있는 것을 볼 수 있다.
sampleapp 이라는 디렉토리안에 jar 파일이 위치하고 있다. 만약 서버 생성 시 Custom 서버 이미지를 만들지 않고 다시 환경을 같이 구축한다면 디펙토리 명이 오타가 발생할 수도 있다.
대체로는 그러지 않겠지만 휴먼 에러가 충분히 발생할 수 있다. 또한 jdk version 이 다르게 설치가 되는 등 서버 이미지를 만들어서 관리한다면 같은 환경임을 보장하여 휴먼 에러를 줄일 수 있다.

![3-21.png](/assets/img/ncloud-sourcepipeline/3-21.png)

#### 서버 실행
jdk 를 설치하지 않았음에도 실행이 잘되는 것을 볼 수 있다.
```bash
java -jar sample_webapp.jar
```

![3-22.png](/assets/img/ncloud-sourcepipeline/3-22.png)

#### 서버 접속
할당 받은 공인 IP 뒤에 /simple_info.do 를 호출하면 아래와 같이 출력을 확인할 수 있다.
혹 이 에제를 따라할 경우 v1 이라는 출력을 제외하고는 다 다를 것인데 잘못된 것이 아니라 정상적인 것이다.

내부 IP 또한 ncloud server 목록에서 보았던 것과 일치하게 출력이 되는 것을 확인하였다.

![3-23.png](/assets/img/ncloud-sourcepipeline/3-23.png)

이로써 서버1 과 서버2를 구축하였다.


