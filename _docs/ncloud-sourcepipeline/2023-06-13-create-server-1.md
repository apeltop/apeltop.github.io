---
title: Server 1 만들기
layout: doc
subtitle: NCloud 에서 서버 1개 만든 후 Spring Boot 실행하기
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

## 목표
`서버 1개를 생성한 후 스프링을 실행시켜 웹으로 접속해본다.`

### 결과물
![2-2.png](/assets/img/ncloud-sourcepipeline/2-2.png)

## 서버 생성하기
우선 [Naver Cloud Platform](https://www.ncloud.com) 에 접속한다.

그 후 우측 상단에 `콘솔`을 클릭한다.

![2-1.png](/assets/img/ncloud-sourcepipeline/2-1.png)

그럼 대쉬보드가 보이는데 여기서 중요하게 보고 넘어갈 부분은 `Platform` 이다.
Platform 은 `Classic` 과 `VPC` 를 선택하게 되어있다.

![2-3.png](/assets/img/ncloud-sourcepipeline/2-3.png)

간단하게 비교하면 가상 네트워크 사용 여부이다. VPC 는 단어 그대로 Virtual Private Cloud 을 사용할 수 있게 된다.
Classic 은 VPC 를 사용하지 않는다는 의미이다.

아래의 그림과 같이 뭔가 마구 연결되게 된다. Classic IP 가 무작위로 생성이 되기 때문에 권한 제어, 접근 제어 등이 어렵다.
자세한 내용은 [네이버클라우드 기술&경험 VPC 낱낱이 알려드림](https://blog.naver.com/n_cloudplatform/222292617371) 에 쉽게 설명되어있다.

![platform-classic](https://miro.medium.com/max/1158/1*fDwl2BQwX4zBUa_hH7ebBg.png)
출처 [네이버 클라우드 공식 블로그](https://blog.naver.com/n_cloudplatform/222292617371)

VPC 는 클라우드라면 모두 사용되는 부분이기 때문에 VPC 로 만들어서 포스팅을 하려했지만 기업 계정이 아니면 VPC 를 사용할 수 없는 것 같다.
VPC 를 만들고 서버를 생성하려 하니 `서버 생성 제한`이 걸려있다고 해서 해제하려 하니 `Organization` 이 필요하다고 한다. 그래서 또 만들려고 가니 아래와 같이 나오며 생성이 되지 않는다.
즉 VPC 서버를 못 만드는 것으로 이해가 되는데 이게 맞는지 확신은 없다. 만약 맞다면 아쉬운 부분이겠다.

![2-4.png](/assets/img/ncloud-sourcepipeline/2-4.png)

예제를 만드는데 VPC 여야만하는 이유는 없기 때문에 Classic 으로 진행할 것이다. 필자는 Classic 으로 진행하지만 회사에서 사용해야한다면 VPC 를 선택해서 구축하기를 권장한다.

Services > All > Server 를 클릭해준다.

![2-5.png](/assets/img/ncloud-sourcepipeline/2-5.png)

그러면 Server 화면이 나오게 된다. `2세대` 와 `1세대` 를 선택할 수 있다.
이 중에서 **1세대의 Standard** 를 선택할 것이다. 우선 서버 생성을 클릭한다.

![2-6.png](/assets/img/ncloud-sourcepipeline/2-6.png)

서버 종류를 선택할 수 있다. 선호에 따라 또는 상황에 따라 각기 필요한 서버를 선택하면 된다.
이번 예제 에서는 centos 를 선택하였다.

![2-7.png](/assets/img/ncloud-sourcepipeline/2-7.png)

**서버 설정** 단계이다. 여기서 중요한 부분은 서버 타입이다. 
아무래도 과금의 규모가 좌우되기 때문이다. 이번 예제에서는 g1(1세대) > Standard > vCPU 2, 메모리 4GB, SSD 50GB 를 선택하였다. 

![2-8.png](/assets/img/ncloud-sourcepipeline/2-8.png)

다음 단계로는 인증키 설정이다. 여기서 인증키 이름을 입력하고 **인증키 생성 및 저장**을 눌러주면 **pem** 확장자인 파일이 다운로드된다.
후반 작업에서 **관리자 비밀번호**를 알아야할 때 쓰이기 때문에 잘 보관해두어야한다. 

![2-9.png](/assets/img/ncloud-sourcepipeline/2-9.png)

네트워크 접근 설정에서는 ACG(Access Control Group) 를 설정한다.
**+ACG 생성**을 클릭하면 단번에 무엇을 하는지 알 수 있다.

![2-10.png](/assets/img/ncloud-sourcepipeline/2-10.png)

우선 접근 소스는 0.0.0.0/0 과 허용 포트는 80 인 설정을 추가한다. 
이 뜻은 모든 IP 에서 80 포트로 접근이 가능하다는 의미이다. 그 후 생성을 클릭한다.

![2-11.png](/assets/img/ncloud-sourcepipeline/2-11.png)

마지막 단계에서는 설정한 내용을 확인할 수 있다. 여기서 **서버 생성** 버튼을 클릭하면 서버가 생성된다.

![2-12.png](/assets/img/ncloud-sourcepipeline/2-12.png)

**서버 생성** 버튼을 클릭하면 **서버 생성 중**이라는 modal 이 나타나면서 시간이 꽤 걸린다는 것을 알려주고 있다.
또한 2가지를 추가로 안내해주고 있다.

1. 포트포워딩
2. 공인 IP

`포트포워딩`은 간단하게 정리하면 외부에서 내부로 접근할 수 있게 해주는 것이다. 
포트포워딩과 관련해서는 [포트포워딩(Port-Forwarding) 이란?](https://storytown.tistory.com/14) 포스팅을 참고하면 이해에 도움이 될 것 같다.

`공인 IP`는 서버에 접근할 수 있는 IP 주소를 말한다.
공인 IP 를 등록하지 않으면 내부 IP 만 존재해서 외부에서 접속이 불가능하다.

위 작업은 서버 생성 전에 해도되는 작업인데 서버가 만들어지면 작업하려한다. 

![2-13.png](/assets/img/ncloud-sourcepipeline/2-13.png)


서버 생성은 고지해준대로 수 분이 소요된다.

![2-14.png](/assets/img/ncloud-sourcepipeline/2-14.png)

다른 것을 조금 하다보면 운영중을 뜻하는 초록불을 볼 수 있다. 확인 후 포트 포워딩 설정을 클릭해준다.

![2-15.png](/assets/img/ncloud-sourcepipeline/2-15.png)

포트 포워딩을 설정해야 외부에서 SSH 접근이 가능하다. 포트 번호 22번은 SSH 접근을 위한 포트이다.
외부 포트는 개인마다 설정하기 나름인데 이번 포스팅에서는 10022 번으로 설정하였다.

![2-16.png](/assets/img/ncloud-sourcepipeline/2-16.png)

**+추가**까지 눌러줘야 적용이 된다.

![2-17.png](/assets/img/ncloud-sourcepipeline/2-17.png)

그렇지만 포트포워딩을 설정해도 접속할 준비가 다 된것은 아니다. 이유는 ACG 에서 22번 포트를 허용하지 않고 있기 때문이다.
그래서 22번 포트도 허용해주어야한다. 22번의 경우 자신만 사용하거나 특정 대역폭에서 사용하기 때문에 제한 하는 것이 좋다.
간단하게도 **myIP** 를 클릭하면 자신의 IP 를 등록할 수 있다.

![2-32.png](/assets/img/ncloud-sourcepipeline/2-32.png)

이제 외부에서 접근은 할 수 있는데 로그인할 수 있는 정보가 없다.
비밀번호를 설정하거나 고지해준 것이 없지만 `서버 관리 및 설정 변경 > 관리자 비밀번호 확인`을 눌러준다. 

![2-18.png](/assets/img/ncloud-sourcepipeline/2-18.png)

서버 생성 단계 중 **인증키 설정** 단계에서 만들어서 다운로드 되었던 pem 파일을 업로드 한다. 
그 후 **비밀번호 확인**을 눌러준다. 

![2-19.png](/assets/img/ncloud-sourcepipeline/2-19.png)

그럼 아래와 같이 비밀번호가 나타난다. 이 비밀번호를 이용해서 서버에 접근할 수 있다.

![2-20.png](/assets/img/ncloud-sourcepipeline/2-20.png)

필자의 경우는 IntelliJ 라는 IDE 에서 제공하는 SSH 연결 기능을 이용해서 접근하였다.
NCloud 에서도 안내하고 있는 대표적인 SSH 접속 프로그램은 [Putty](https://www.putty.org) 이다.

서버에 접속해서 Java 를 먼저 설치애햔다. jar 로 실행할 것이지만 java 는 설치가 되어야하기 때문이다.

```bash
yum list java*jdk-devel
```

아래 명령어를 입력하면 설치 가능한 Java 버전들이 나타난다.

예시 출력 값
```bash
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.kakao.com
 * extras: mirror.kakao.com
 * updates: mirror.kakao.com
Installed Packages
java-1.8.0-openjdk-devel.x86_64  1:1.8.0.372.b07-1.el7_9     @updates
Available Packages
java-1.6.0-openjdk-devel.x86_64   1:1.6.0.41-1.13.13.1.el7_3    base    
java-1.7.0-openjdk-devel.x86_64   1:1.7.0.261-2.6.22.2.el7_8    base    
java-1.8.0-openjdk-devel.i686   1:1.8.0.372.b07-1.el7_9   updates 
java-11-openjdk-devel.i686   1:11.0.19.0.7-1.el7_9   updates 
java-11-openjdk-devel.x86_64    1:11.0.19.0.7-1.el7_9   updates 
```

![2-21.png](/assets/img/ncloud-sourcepipeline/2-21.png)

목록 중에서 java-1.8.0-openjdk-devel.x86_64 를 설치한다.

```bash
yum install java-1.8.0-openjdk-devel.x86_64
```

위 구문을 실행하면 아래와 같이 설치가 진행된다.

![2-22.png](/assets/img/ncloud-sourcepipeline/2-22.png)

설치가 완료되면 `java -version` 을 입력해서 설치가 제대로 되었는지 확인해본다.

![2-23.png](/assets/img/ncloud-sourcepipeline/2-23.png)

이제 java 를 설치했기 때문에 jar 파일을 실행할 수 있다.

이전 글에서 만들었던 jar 파일을 업로드 해준다.
서버에 파일을 업로드하는 방법은 여러 방법이 있지만 개인적으로는 [FileZilla](https://filezilla-project.org) 를 선호한다.

서버에 접속했던 정보를 토대로 접속하면 된다. 다른 점은 이전에는 SSH 로 접속했지만 파일을 옮기기 위해서는 FTP 로 접속한다는 점이다.
그러나 FTP 로 접속하려하면 접속이 되지 않는다. SFTP 로 접속해야한다. 접속이 성공하면 root 폴더가 나타난다.

![2-24.png](/assets/img/ncloud-sourcepipeline/2-24.png)

sampleapp 폴더를 만들고 그 안에 sample_webapp.jar 파일을 업로드 해준다.

![2-25.png](/assets/img/ncloud-sourcepipeline/2-25.png)

그 후 다시 SSH 로 접속하여 서버를 실행해준다.

```bash
java -jar sample_webapp.jar
```

익숙한 로그들이 올라가고 있는 것을 볼 수 있다.

![2-26.png](/assets/img/ncloud-sourcepipeline/2-26.png)

이제 서버도 실행이 되었겠다 브라우저에서 접속해보고 싶다. 
그러나 공인 IP 가 없어 접속할 수 없다. 공인 IP 는 간단하게 할당이 가능하다.

**Server > Public IP** 로 이동한다. 그 후 **공인 IP 신청**을 클릭한다.

![2-27.png](/assets/img/ncloud-sourcepipeline/2-27.png)

Zone 은 선택할 수 있지만 기본값으로 설정이 되어있고 적용 서버를 선택하면 된다.
현재는 서버 1개 밖에 없어 선택의 여지가 없다.

![2-28.png](/assets/img/ncloud-sourcepipeline/2-28.png)

월에 **약 4000원** 정도 추가 비용이 발생함을 고지해준다. 생성 버튼을 클릭해서 **서버1** 에 공인 IP 를 할당해준다.

![2-29.png](/assets/img/ncloud-sourcepipeline/2-29.png)

아쉽게도 설정 한 가지를 더 해야된다. 그것은 ACG 에서 8080 포트 허용이다.
처음 ACG 설정에서 80 포트만 허용을 해주었기 때문에 8080 포트로 접속이 안된다.

**Server > ACG**로 이동 하여 8080 포트를 허용해준다.

![2-30.png](/assets/img/ncloud-sourcepipeline/2-30.png)

그 후 공인 IP 뒤에 8080인 호스트 뒤에 /simple_info.do 를 호출하면 내부 IP 와 UUID 를 확인할 수 있다.

![2-31.png](/assets/img/ncloud-sourcepipeline/2-31.png)




























