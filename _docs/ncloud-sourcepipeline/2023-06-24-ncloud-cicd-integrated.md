---
title: 통합본 - NCloud LB & SourcePipeline 구축하기
layout: doc
subtitle: 특정 브랜치에 커밋 시 자동 배포하기
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

## 목표
`main 브랜치에 커밋 시 로드 밸런서로 구축되어있는 서버 2대에 배포하기`

![9-1.png](/assets/img/ncloud-sourcepipeline/9-1.png)

여기에서 CI/CD 부분 을 구축해보려 한다.

![source-pipeline-1600-kr_1684717797806.svg](https://xv-ncloud.pstatic.net/images/product/04-source-pipeline-1600-kr_1684717797806.svg)

출처 [Ncloud SourcePipeline](https://www.ncloud.com/product/devTools/sourcePipeline)

이 글은 파트 별로 나누어 쓴 글을 합친 통합본이다.
- [eGov 프로젝트 구축하기](/docs/ncloud-sourcepipeline/2023-06-18-setup-egov/)
- [Server 1 만들기](/docs/ncloud-sourcepipeline/2023-06-13-create-server-1/)
- [Server 2 만들기](/docs/ncloud-sourcepipeline/2023-06-18-create-server-2/)
- [eGov 애플리케이션 백그라운드 실행](/docs/ncloud-sourcepipeline/2023-06-18-egov-background/)
- [로드 밸런서 구축](/docs/ncloud-sourcepipeline/2023-06-19-load-balancer/)
- [SourceCommit 에 Git 연동](/docs/ncloud-sourcepipeline/2023-06-21-source-commit/)
- [SourceBuild 를 통해 배포 파일 생성](/docs/ncloud-sourcepipeline/2023-06-22-source-build/)
- [SourceDeploy 를 통해 배포하기](/docs/ncloud-sourcepipeline/2023-06-23-source-deploy/)
- [SourcePipeline 을 통해 CI/CD 통합 관리하기](/docs/ncloud-sourcepipeline/2023-06-24-source-pipeline/)

예제에서 사용한 키워드를 간략하게 하면 다음과 같다.

- Server
- Server Image
- Load Balancer
- SourceCommit
- SourceBuild
- SourceDeploy
- SourcePipeline

## 예제에 필요한 자료
각 포스팅의 예제를 따라하면 다 만들고 구축해볼 수 있으나 소스 코드를 직접 만들지 않고 클라우드 관련만 빠르게 따라해보고 싶은 경우가 있을 수 있기에 자료를 올려놓았다.
소스 코드 빌드 과정 등이 있기 때문에 소스 코드를 직접 만들어야하는데 이 또한 따라하지 않고 싶다면 밑 Git Repository 에 들어가 fork 한 뒤 사용하면 된다.

Git Repository: [simple-egov-boot](https://github.com/apeltop/simple-egov-boot)

jar
- [v1](https://github.com/apeltop/simple-egov-boot/blob/main/jar/sample_webapp-v1.jar)
- [v2](https://github.com/apeltop/simple-egov-boot/blob/main/jar/sample_webapp-v1.jar)


# eGov 프로젝트 구축하기
## 목표
`내부 아이피` `UUID` `버전` 을 출력하도록 한다.

![1-1.png](/assets/img/ncloud-sourcepipeline/1-1.png)



## 만들기
### eGov 이란
우선 간단하게 [전자정부프레임워크(eGov)](https://www.egovframe.go.kr/home/main.do) 을 설명하고 진행하겠다.

![1-2.png](/assets/img/ncloud-sourcepipeline/1-2.png)

스프링 프레임워크는 하나의 프로젝트로 구성된 것이 아니라 여러개의 모듈로 구성이 되어있다.
필요에 따라서 Spring 모듈을 선택 및 추가하여 사용할 수 있다.

![spring_architecture.png](https://www.tutorialspoint.com/spring/images/spring_architecture.png)

전자정부프레임워크는 스프링 프레임워크 기반으로 만들어진 프레임워크이다.
차이점이라고 한다면 전자정부프레임워크에서 공통 컴포넌트를 제공하여 사용할 수 있다는 것이다.
공통된 UI 컴포넌트를 제공하기도하지만 보안, 로그인, 권한, 통계, 메일, SMS, 파일처리 등의 공통된 기능을 제공한다.

아래는 [전자정부프레임워크 4.1 일반로그인 가이드](https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:com:v4.1:uat:일반로그인) 중 일부 화면이다.

![1-3.png](/assets/img/ncloud-sourcepipeline/1-3.png)

egovframework.com.~ 처럼 전자정부프레임워크에서 제공하는 컴포넌트가 있다. 가이드를 보면 DAO 관련해서 필수적으로 상속을 받아야하는 클래스가 있는 등 여러가지의 가이드가 있다.
더 자세히 다루기에는 클라우드를 중점인 글이기에 무리가 있어보인다. 이제 프로젝트를 구성해보겠다.

### Egov 프로젝트 초기 셋팅
[전자정부프레임워크 다운로드](https://www.egovframe.go.kr/home/sub.do?menuNo=4)에 접속해서 필요한 버전을 다운로드 받는다.
다운로드 된 폴더 안에 이클립스를 확인할 수 있는데 이를 열어준다. 그 후 프로젝트를 새로 만들 때 eGovFrame Boot Web Project 로 생성한다.

![1-4.png](/assets/img/ncloud-sourcepipeline/1-4.png)

이름에서 유추할 수 있듯이 eGovFrame Boot Web Project 는 Spring Boot 기반으로 만들어진 프로젝트이다.

위 과정을 생략하고 초기 프로젝트를 다운로드 받기를 원한다면 필자가 만들어둔 [simple-egov-boot](https://github.com/apeltop/simple-egov-boot/tree/674321867ecb14c7349fcd6817e8016d8f45bfaf) 에 들어가 다운로드 받으면 된다.

빌드 과정도 생략하고 jar 로 테스트하고 싶은 경우는 [simple-egov-boot-jar](https://github.com/apeltop/simple-egov-boot/tree/main/jar) 에 들어가 sample_webapp-v1.jar 파일을 다운로드 받아 실행하면 된다.

### 정보 반환하는 컨트롤러 만들기
필자는 src/main/java/egovframework/sample 패키지를 만들고 **SimpleInfoController.java** 파일을 추가하였다.
그 후 코드를 추가해준다.

![1-5.png](/assets/img/ncloud-sourcepipeline/1-5.png)


```java
package egovframework.sample;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.net.InetAddress;
import java.util.UUID;

@RestController
public class SimpleInfoController {
    private final UUID uuid = UUID.randomUUID();
    private final String v = "v1";

    @GetMapping("/simple_info.do")
    public String simpleInfo() throws Exception {
        String ip = InetAddress.getLocalHost().getHostAddress();
        return ip + " " + uuid + " " + v;
    }
}
```

코드는 매우 간단하다. **/simple_info.do** 에 접속하면 내부 아이피, UUID, 버전을 반환한다.
내부 아이피는 서버 2대에 배포했을 때 어떤 서버인지 알기위함이다. UUID 의 경우는 서버가 껐다 켜지면 새로운 UUID 가 생성될 것이다.
그러기에 UUID 가 달라진다는 것은 서버가 껐다 켜졌다는 것을 알 수 있다. 버전은 서버가 어떤 버전인지 알기 위함이다.

필자의 경우는 프로젝트 생성만 전자정부프레임워크가 제공하는 이클립스를 사용했다. 그 뒤에 작업은 IntelliJ 를 사용했다.
maven package 를 실행하면 target 폴더 안에 *.jar 파일이 생성이 된다.

![1-6.png](/assets/img/ncloud-sourcepipeline/1-6.png)

### jar 파일 실행하기
jar 파일을 실행하면 서버 실행 로그가 출력이 되는 것을 확인할 수 있다.

```bash
java -jar sample_webapp.jar
```

# Server 1 만들기
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

![2-2.png](/assets/img/ncloud-sourcepipeline/2-2.png)

# Server 2 만들기
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


# eGov 애플리케이션 백그라운드 실행
## 목표

`SSH 이 끊겨도 서버가 실행되도록 한다.`

## 현재 상황

현재는 SSH 세션이 끊기면 서버도 같이 종료되는 상태이다. 이를 해결하기 위해 서버를 백그라운드로 실행시키는 방법을 알아보자.

SSH 세션이 끊기면 서버가 종료되는 이유와 백그라운드 실행하는 간단한 예제를 먼저 살펴보려한다.

### Foreground

아래 구문은 Foreground 로 실행되는 구문이다. 즉 Ctrl + C 를 누르면 프로그램 종료와 동시에 서버도 종료가 된다.

```bash
java -jar sample_webapp.jar
```

### Background

#### &

구문 뒤에 & 를 붙이면 백그라운드 실행이 된다. & 을 붙이고 세션을 종료해도 서버가 종료되지 않는다.
과거 OS 에서는 & 을 붙이고 세션을 종료하면 서버도 종료가 되었는데 현재는 & 를 붙이어도 세션이 끊어져도 유지가 되도록하는게 기본 값이기 때문이다.
혹 과거 OS 이거나 불안함이 있다면 아래 소개할 nohup 을 사용하는 것을 권장한다.

```bash
java -jar sample_webapp.jar &
```

#### nohup

& 실행 구문 앞에 nohup 을 붙이는게 전부이다.
nohup 은 no hang up 의 약자로 세션이 끊겨도 유지가 되도록 하는 명령어이다.
또한 nohup 은 데몬 프로세스로 실행이 되기 때문에 프로세스가 종료되어도 서버가 종료되지 않는다.

```
nohup java -jar sample_webapp.jar &
```

nohup 의 출력과 관련해서는 [쉽게 설명한 nohup 과 &(백그라운드) 명령어 사용법](https://joonyon.tistory.com/entry/쉽게-설명한-nohup-과-백그라운드-명령어-사용법) 을
참고해주길 바란다.

## Shell Script 작성

### startup.sh

아래 구문을 실행하면 sample_webapp.jar 가 백그라운드로 실행된다.

```bash
nohup java -jar /root/sampleapp/sample_webapp.jar &
```

### shutdown.sh

sample_webapp.jar 로 실행되고 있는 프로세스의 pid 를 찾아 kill 하는 구문이다.

```bash
pid=`ps -ef | grep sample_webapp.jar | grep -v grep | awk '{print $2}'`
echo "Killing process $pid"
kill -9 $pid
```

grep -v grep 을 추가한 이유는 grep sample_webapp.jar 을 실행하면 자기 자신의 프로세스도 포함되기 때문이다.

```bash
(base) apeltop% ps -ef | grep sample_webapp.jar
  501 44013     1   0 10:14PM ttys008    0:09.25 /usr/bin/java -jar ../target/sample_webapp.jar
  501 44027 37450   0 10:14PM ttys008    0:00.00 grep sample_webapp.jar
```

```bash
(base) apeltopscript % ps -ef | grep sample_webapp.jar | grep -v grep 
  501 44013     1   0 10:14PM ttys008    0:09.58 /usr/bin/java -jar ../target/sample_webapp.jar
```

## 배포

startup.sh 와 shutdown.sh 는 server1, server2 에 각각 업로드해준다.

server1 구축을 먼저 해놓고 해당 이미지를 만들어 server2 를 만들었으면 더 편하지만 서버 이미지를 생성하는 예제에서 쉘 스크립트 작성 내용이 없어야 더 명확한 글이 될 것 같아 나누어서 진행하였다.

![4-1.png](/assets/img/ncloud-sourcepipeline/4-1.png)

startup.sh 를 실행시키면 권한이 없다고 나온다.

```bash
[root@server1 sampleapp]# ./startup.sh
-bash: ./startup.sh: Permission denied
```

startup.sh 와 shutdown.sh 에 실행 권한을 부여해준다.

```bash
[root@server1 sampleapp]# chmod 755 startup.sh
[root@server1 sampleapp]# chmod 755 shutdown.sh
```

startup.sh 를 실행해서 공인 IP 로 접근하면 실행이 된 것을 확인할 수 있으며 shutdown.sh 를 실행 시 `Killing process 8450` 와 같은 출력이 나오면서 서버가 종료된 것을 확인할
수 있다.

이로써 백그라운드로 서버 운영과 종료 할 수 있는 Shell Script 를 작성하였다. 

# 로드 밸런서 구축

## 목표
`로드 밸런서를 사용해 서버 2대에 나누어 처리하도록 하기`

## 만들기
### Load Balancer 란
![5-1.png](/assets/img/ncloud-sourcepipeline/5-1.png)

이미지 출처 : [What is Azure Load Balancer?](https://learn.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)

#### Public Load Balancer
Public Load Balancer 는 외부에서 접속했을 때 Load Balancer 를 통해 서버로 분배하는 방식이다.

#### Internal Load Balancer
Internal Load Balancer 는 외부에서 접속할 수 없고, 내부 요청의 의해 분배가 이루어지는 방식이다.

Public Load Balancer 를 통한 요청이 Internal Load Balancer 를 통해 분배되는 구조로도 구성할 수 있다.

### Load Balancer 구축
Networking > Load Balancer 에 들어간다.

그 후 `로드 밸런서 생성` 을 클릭한다.

![5-2.png](/assets/img/ncloud-sourcepipeline/5-2.png)

로드밸런서 이름을 입력해준다.

#### 네트워크
그 후 네트워크를 Private IP 와 Public IP 중 선택한다.
Private IP 의 경우 위에서 간단히 설명한 Intenal Load Balancer 에 해당된다.
Public IP 의 경우는 Public Load Balancer 에 해당된다.

이번 예제에서는 외부의 요청을 분배하려고 하기 때문에 **Public IP** 를 선택해준다.

#### 로드 밸런서 설정
80 번 포트로 접근을 하되 8080 번 포트로 요청하도록 하였다.
당연히 포트는 각자에 상황에 따라 달리할 수 있다.

#### 로드배런싱 알고리즘
##### Round Robin
단순히 순서대로 요청을 분배하는 방식이다.
순차적으로 서버에 요청을 분배한다. 장점은 단순히 순차 분배이기 때문에 **LB 서버 부하가 제일 적다**.
단점은 **특정 서버의 처리가 오래** 걸리거나 **특정 서버가 더 성능이 좋을 때** 부하 문제가 발생할 수 있다.

이러한 문제를 해결하기 위해 **가중치** 를 부여할 수 있다.
이를 Weighted Round Robin 이라고 한다. 서버 성능이 좋은 서버에 2, 평범한 성능인 서버에 1 을 부여하면 2:1 비율로 요청을 분배한다.
ncloud 에서 제공하는 로드밸런서는 가중치를 부여할 수 없다.

![round_robin_algorithm-1.png](https://www.jscape.com/hubfs/images/round_robin_algorithm-1.png)

이미지 출처 [Comparing Load Balancing Algorithms](https://www.jscape.com/hubfs/images/round_robin_algorithm-1.png)

##### Least Connection
단어 그대로 가장 적은 연결을 가진 서버에 요청을 분배하는 방식이다.

아래 첨부한 사진처럼 라운드 로빈의 경우라면 서버 1에 5, 서버 2에 2, 4, 6 이 연결될 것이다.
그렇지만 Least Connection 을 사용하면 서버 1에 요청이 더 적기 때문에 6의 요청은 서버 1에 할당될 것이다.

세션 수를 저장하고 사용해야하기에 메모리를 사용하나 적은 양이다. 대체로 많은 경우 Least Connection 을 선택한다.
성능의 균형을 어느정도 맞춰줄 수 있기 때문이다.

![least_connections_algorithm_solves_this.png](https://www.jscape.com/hubfs/images/least_connections_algorithm_solves_this.png)
이미지 출처 [Comparing Load Balancing Algorithms](https://www.jscape.com/hubfs/images/least_connections_algorithm_solves_this.png)

![least_connections.png](https://www.jscape.com/hubfs/images/least_connections.png)
이미지 출처 [Comparing Load Balancing Algorithms](https://www.jscape.com/hubfs/images/least_connections.png)

##### Source IP Hash
Source IP Hash 는 Hashing Methods 중 하나이다.
Hashing Methods 는 **URL Hash method**, **Source IP Hash method** 가 있다.

URL Hash method 는 URL 을 해싱하여 요청을 분배하는 방식이다.

Source IP Hash method 는 Source IP, Source Port, Destination IP, Destination Port 등을 해싱하여 요청을 분배하는 방식이다.
Source IP, Destination IP 등 해싱에 필요한 정보가 같다면 같은 서버로 요청이 간다.
Source IP Hash method 의 방식은 수평 확장(Vertical Scaling) 과 같이 서버가 증설되었을 때 기존의 세션이 다른 곳으로 이동하는 문제가 발생한다.

![DTC-source-10.png](https://blogs.infoblox.com/wp-content/uploads/DTC-source-10.png)
이미지 출처 [Source IP Hash Load Balancing for Application Persistency](https://blogs.infoblox.com/community/source-ip-hash-load-balancing-for-application-persistency/)

![5-3.png](/assets/img/ncloud-sourcepipeline/5-3.png)

#### 서버 선택
**다음** 버튼을 클릭하면 서버 추가 화면이 나온다.
여기서 server1, server2 가 목록에 추가되어있다.
![5-4.png](/assets/img/ncloud-sourcepipeline/5-4.png)

server1, server2 모두 적용 서버로 옮기어준다.
![5-5.png](/assets/img/ncloud-sourcepipeline/5-5.png)

#### 설정 확인 후 생성
설정을 확인하고 문제 없으면 **로드 밸런서 생성**을 클릭해준다.

![5-6.png](/assets/img/ncloud-sourcepipeline/5-6.png)

로드밸런서 생성중인 것을 볼 수 있다.

![5-7.png](/assets/img/ncloud-sourcepipeline/5-7.png)

#### 로드밸런서 정보 확인
여기서 설정한 정보를 볼 수 있다. 적용 서버를 보면 server1, server2 가 운영중이라는 것을 볼 수 있다.
여기서 접속 정보를 보면 **sib-17885866.ncloudsib.com** 이 나오는데 여기에 접속하면 server1, server2 에 분배한다.
더 자세한 건 뒤에서 확인해보자.

![5-8.png](/assets/img/ncloud-sourcepipeline/5-8.png)

**로드밸런서 상태 확인**을 클릭하면 아래와 같이 서버포트, L7 Health Check 등 여러 정보를 확인할 수 있다.

![5-9.png](/assets/img/ncloud-sourcepipeline/5-9.png)

여기서 만든 예제는 Classic 기반이라 subnet 정보가 없다. VPC 기반일 경우 subnet 정보도 확인할 수 있다.
subnet 정보를 통해 ACG 를 설정하는 것이 좋다. 외부에서는 server1, server2 에 접속할 수 없고 LB 를 통해서만 접근할 수 있도록 하기 위함이다.

해당 글은 MANVSCLOUD, [2-1. [NCLOUD] 네이버 클라우드에서의 보안 – SERVER (ACG)](https://manvscloud.com/?p=859) 를 참고하면 좋다.

## 로드밸런서 테스트
로드밸런서의 역할과 동작을 확인해보려 한다.

python 기반으로 코드를 간결하게 적어 테스트하였다.
아래 코드는 **requests** 패키지만 추가하면 실행할 수 있다.

```bash
pip install requests
```

### server1, server2 서버 접속
```python
import requests

SERVER_IP1 = 'http://49.50.162.69:8080'
SERVER_IP2 = 'http://101.101.211.102:8080'

res1 = requests.get(f'{SERVER_IP1}/simple_info.do')
print(res1.text)

res2 = requests.get(f'{SERVER_IP2}/simple_info.do')
print(res2.text)
```

#### 결과
```bash
10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
```

각 서버에 접속이 잘되는 것을 확인하였다.

### LB(Load Balancer) 접속
```python
import requests

LB_IP1 = 'http://slb-17885866.ncloudslb.com'
for i in range(10):
    res_LB = requests.get(f'{LB_IP1}/simple_info.do')
    print(i, res_LB.text)
```

#### 결과
```bash
0 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
1 10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
2 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
3 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
4 10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
5 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
6 10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
7 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
8 10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
9 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
```

LB 에 접속하면 server1, server2 에 요청을 번갈아가며 보내는 것을 확인할 수 있다.

### LB 균등 분배 확인
```python
import requests

LB_IP1 = 'http://slb-17885866.ncloudslb.com'

lb_to_server1_count = 0
for i in range(1000):
    try:
        sleep(1)

        res_LB = requests.get(f'{LB_IP1}/simple_info.do', timeout=3)
        print(i, res_LB.text)
        if res_LB.text.find('10.41.151.114'):
            lb_to_server1_count += 1
    except Exception as e:
        print('request error')

print(lb_to_server1_count)
```

```bash
500
```

결과는 500으로 딱 1000개의 절반이다. 일의 자리 정도는 차이날 수 있겠다 싶었는데 딱 떨어진다.
이로써 분배가 잘 되는 것을 확인하였다.

### 서버 1 다운되었을 때
테스트는 1초에 한 번 씩 LB 에 요청을 보내면서 서버 1을 다운시켜보았다.

![5-10.png](/assets/img/ncloud-sourcepipeline/5-10.png)

```bash
0 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
1 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
2 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
3 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
4 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
5 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
6 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
7 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
8 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
9 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
10 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
11 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1

...

192 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
193 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
194 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
195 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
196 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
197 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
198 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
199 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
200 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
201 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
202 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
203 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
```

이 때 LoadBalancer 를 들어가보니 server1 이 **health check** 가 실패한 것을 확인할 수 있다.

그렇기에 LB 에서는 서버1이 health check 를 통과하지 못하였기에 **서버2에만 요청**을 보내는 것을 확인할 수 있다.

![5-11.png](/assets/img/ncloud-sourcepipeline/5-11.png)

192 번 요청부터는 server1, server2 를 다시 오가는 것을 볼 수 있는데 이는 서버1이 health check 를 통과하였기에 다시 요청을 보내는 것이다.

![5-12.png](/assets/img/ncloud-sourcepipeline/5-12.png)

이로써 LB(LoadBalancer) 를 구축하고 분배가 잘 되는지 server1 이 다운되었을 때 server2 로만 요청을 전달하는지 확인하였다.
모두 예상대로 동작하는 것 또한 확인하였다.


# SourceCommit 에 Git 연동 
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

키 위치는 **/Users/`사용자`/.ssh/id_sourcecoomit** 로 입력하였다.

그러면 키 위치에 **id_sourcecoomit** 과 **id_sourcecoomit.pub** 파일이 생성된다.
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


# SourceBuild 를 통해 배포 파일 생성
## 목표
`main 브랜치에 커밋이 되었을 경우 배포 파일인 jar 이 생성되도록 하기`.

![7-1.png](/assets/img/ncloud-sourcepipeline/7-1.png)

## SourceBuild 란
[SourceBuild](https://www.ncloud.com/product/devTools/sourceBuild) 는 이번 포스팅처럼 jar 파일을 만들 수 있고 이미지를 만들어서 저장하고 관리할 수 있다.

## 만들기
Developer Tools > SourceBuild 에 들어간다.
그 후 **빌드 프로젝트 생성**을 클릭한다.

![7-2.png](/assets/img/ncloud-sourcepipeline/7-2.png)

그럼 Object Storage 를 사용중으로 변경해야한다고 나온다.
이미 NCP(Naver Cloud Platform) 에서 Object Storage 를 사용중이라면 넘어가도 된다.
또한 여기서 유의해서 볼 내용은 Object Storage 생성 후 **버킷이 하나** 있어야한다는 것이다.

![7-3.png](/assets/img/ncloud-sourcepipeline/7-3.png)

### Object Storage 에 버킷 생성
Storage > Object Storage 에 들어가서 **이용 신청**을 클릭한다.

![7-4.png](/assets/img/ncloud-sourcepipeline/7-4.png)

![7-5.png](/assets/img/ncloud-sourcepipeline/7-5.png)

그 후 Object Storage 에서 **버킷 생성**을 클릭한다.

![7-6.png](/assets/img/ncloud-sourcepipeline/7-6.png)

버킷 이름을 입력해주고 다음으로 넘어간다.


![7-7.png](/assets/img/ncloud-sourcepipeline/7-7.png)

설정 관리와 암호화 관리를 사용 여부를 선택할 수 있다.
필요에 따라 설정하면 되나 여기서는 그냥 넘어간다.

![7-8.png](/assets/img/ncloud-sourcepipeline/7-8.png)

권한 관리는 현재 예제에서는 사용자가 필자 혼자이기에 넘어간다.

![7-9.png](/assets/img/ncloud-sourcepipeline/7-9.png)

버킷 생성 정보를 확인 후 문제가 없으면 **버킷 생성**을 클릭한다.

![7-10.png](/assets/img/ncloud-sourcepipeline/7-10.png)

이것으로 Object Storage 에 버킷이 하나 생성하였다.

### SourceBuild 에서 빌드 프로젝트 생성
다시 SourceBuild 로 돌아와서 **빌드 프로젝트 생성**을 클릭하면 Object Storage 가 사용중이라고 나온다.

![7-11.png](/assets/img/ncloud-sourcepipeline/7-11.png)

필수 입력 값인 빌드 프로젝트 이름 입력 후 빌드 대상, 리파지토리, 브랜치를 입력 및 선택해준다.

빌드 대상은 이전 글인 [SourceCommit 에 Git 연동](/docs/ncloud-sourcepipeline/2023-06-21-source-commit/) 에서 만든 대상이다.

알림의 경우 궁금해서 **빌드 시작시**, **빌드 종료 시** 알림이 오도록 하였다.
Email, 문자를 선택해서 알림을 받아볼 수 있다.

![7-12.png](/assets/img/ncloud-sourcepipeline/7-12.png)

빌드 시작과 종료될 때 아래와 같이 문자가 온다.

![7-13.png](/assets/img/ncloud-sourcepipeline/7-13.png)

빌드 환경 설정에서 선택할 부분이 많은데 필자는 아래와 같이 선택하였다.

- SourceBuild 에서 관리되는 이미지
- ubuntu 16.04
- java 11-1.0.0
- 2vCpu 4GB 메모리

![7-14.png](/assets/img/ncloud-sourcepipeline/7-14.png)

빌드 런타임 버전 옆에 있는 설치 패키지 목록을 눌러보면 설치되어있는 패키지 목록을 확인할 수 있다.
당연한 말이지만 여기에 사용하고자하는 패키지가 없다면 설치가 필요하다.

![7-15.png](/assets/img/ncloud-sourcepipeline/7-15.png)

빌드 전 명령어 부분에 지워진 부분은 필요 없어서 지웠다.
빌드 명령어 부분은 [eGov 프로젝트 구축하기](/docs/ncloud-sourcepipeline/2023-06-18-setup-egov/) 에서 maven package 를 통해 jar 파일을 생성하였던 것을 명령어로 입력한 것이다.

```bash
mvn -B package - file pom.xml
```

또한 이번 포스팅에서는 도커를 사용하지 않기 때문에 도커 이미지 빌드 설정을 하지 않았다.

![7-16.png](/assets/img/ncloud-sourcepipeline/7-16.png)

업로드 설정에서는 빌드 결과물을 저장하도록 선택하였다.
빌드 결과물 경로는 위에서 maven package 를 통해 jar 파일이 만들어진 경로이다.
target 폴더 안에 jar 파일이 만들어지는데 필자의 예제에서는 sample_webapp.jar 이라는 파일이 만들어지기 때문에 **target/sample_webapp.jar** 를 입력하였다.

그 후 빌드 결과물이 저장될 이전 단계에서 만들었던 Object Storage 의 Bucket 을 선택해준다.
또한 빌드 결과물이 저장될 경로는 /jar 로 입력하였다. 파일 이름은 기존과 똑같이 smaple_webapp.jar 로 저장되도록 하였다.

결과물 백업은 NCloud 설명처럼 구버전의 빌드 결과물을 저장해주는 것이다.
대체로 백업은 좋기 때문에 **백업 사용**하도록 설정하였다.

![7-18.png](/assets/img/ncloud-sourcepipeline/7-18.png)

![7-17.png](/assets/img/ncloud-sourcepipeline/7-17.png)

최종 확인 단계에서 설정 값을 확인하고 문제가 없으면 **생성** 버튼을 누른다.

![7-19.png](/assets/img/ncloud-sourcepipeline/7-19.png)

SourceBuild 에 들어가보면 빌드 프로젝트가 생성된 것을 볼 수 있다.
여기서 프로젝트 이름을 클릭하면 상세보기에 들어가지는데 프로젝트 이름을 클릭하여 이동한다.

![7-20.png](/assets/img/ncloud-sourcepipeline/7-20.png)

그럼 아래와 같이 정보들이 나오는데 우측의 **빌드 시작하기**을 눌러 기대하는대로 동작하는지 확인해보자.

![7-21.png](/assets/img/ncloud-sourcepipeline/7-21.png)

그 후 2번째 탭인 작업결과에 들어가보면 빌드가 진행되는 것을 볼 수 있고 빌드가 완료되면 결과물 업로드까지 되었다고 나온다.

![7-22.png](/assets/img/ncloud-sourcepipeline/7-22.png)

저 초록불을 전적으로 믿으 수 없으니 Object Storage > Bucket Management 에 들어가본다.
**egov-sample-bucket**을 클릭하면 jar, sourcebuild_backup 디렉토리가 생성된 것을 볼 수 있다.

![7-23.png](/assets/img/ncloud-sourcepipeline/7-23.png)

sourcebuild_backup 먼저 들어가보면 uuid 형식의 디렉토리가 생성되었다.
해당 폴더에 들어가보면 jar 파일이 생성된 것을 볼 수 있다.

**모든 빌드 결과물은 zip 파일로 저장이 된다.**

![7-24.png](/assets/img/ncloud-sourcepipeline/7-24.png)

![7-25.png](/assets/img/ncloud-sourcepipeline/7-25.png)

다시 돌아와 jar 디렉토리를 확인해보면 sample_webapp.jar.zip 이 생성된 것을 볼 수 있다.

![7-26.png](/assets/img/ncloud-sourcepipeline/7-26.png)

이로써 SourceBuild 를 통해 main 브랜치에 커밋이 될 때마다 빌드가 되고 빌드 결과물이 Object Storage 에 업로드 되도록 구축하였다.


# SourceDeploy 를 통해 배포하기
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

파일 배포의 경우 input 의 placeholder 를 보면 **sample_webapp.jar.zip** 을 기준으로 작성하라고 나와있다. 그러기에 **target/sample_webapp.jar** 를 입력하였다.
SourceBuild 를 통해 생성된 sample_webapp.jar.zip 을 다운로드 받아보면 아래와 같이 저장되어있기 때문이다.

![8-35.png](/assets/img/ncloud-sourcepipeline/8-35.png)

파일 배포를 말로 다시 풀면 sample_webapp.jar.zip 파일의 압축을 풀어 target/sampel_webapp.jar 를 /root/sampleapp 폴더에 저장하라는 것이다.

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

![8-33.png](/assets/img/ncloud-sourcepipeline/8-11.png)

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



# SourcePipeline 을 통해 CI/CD 통합 관리하기
## 목표
`SourceCommit, SourceBuild, SourceDeploy 를 사용하는 프로레스를 즉 파이프라인을 구축해보자`

![9-1.png](/assets/img/ncloud-sourcepipeline/9-1.png)

## SourcePipeline 이란
[SourcePipeline](https://www.ncloud.com/product/devTools/sourcePipeline) 은 SourceCommit, SourceBuild, SourceDeploy 를 통합 관리하는 서비스이다.
즉 배포에 필요한 일련의 과정을 자동화할 수 있게 해준다. **SourceCommit**, **SourceBuild**, **SourceDeploy** 는 각각 정해진 일이 명확하다.
SourcePipeline 은 배포 전략에 따라 어떤 흐름으로 처리하게 할지 정해주는 역할을 한다.

이번 글에서는 SourceCommit -> SourceBuild(배포 파일 생성) -> SourceDeploy(dev stage) -> SourceDeploy(real stage) 의 흐름으로 만들어볼 것이다.

## 만들기
Developer Tools > SourcePipeline 에 들어간다.
그 후 **파이프라인 생성** 을 클릭한다.

![9-2.png](/assets/img/ncloud-sourcepipeline/9-2.png)

파이프라인 이름을 입력해준다.

![9-3.png](/assets/img/ncloud-sourcepipeline/9-3.png)

파이프라인 구성 단계에서 **작업 추가**를 클릭한다.

![9-4.png](/assets/img/ncloud-sourcepipeline/9-4.png)

타입이 SourceBuild 인 작업을 추가해준다.

![9-5.png](/assets/img/ncloud-sourcepipeline/9-5.png)

그러면 아래와 같이 sample-build 작업이 추가된다.

![9-6.png](/assets/img/ncloud-sourcepipeline/9-6.png)

타입이 SourceDeploy 인 dev stage 작업을 추가해보자.

![9-7.png](/assets/img/ncloud-sourcepipeline/9-7.png)

그리고 타입이 SourceDeploy 인 real stage 작업을 추가해보자.

![9-8.png](/assets/img/ncloud-sourcepipeline/9-8.png)

그러면 아래와 같이 3개의 작업이 생성이 된다.
![9-9.png](/assets/img/ncloud-sourcepipeline/9-9.png)

파이프라인은 관로라는 뜻으로 A에서 B로 흘러가는 것을 의미한다.
그러기에 작업을 만들었지만 작업의 흐름을 만들어줘야한다.

![pipeline.png](https://images.unsplash.com/photo-1635145613344-3e59b1e8afd0?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1587&q=80)

sample-build 작업 왼쪽의 **+** 버튼을 클릭한다.

![9-10.png](/assets/img/ncloud-sourcepipeline/9-10.png)

SourceBuild 의 선행 작업은 없기에 **선행작업 없음**을 선택해주고 생성한다.

![9-11.png](/assets/img/ncloud-sourcepipeline/9-11.png)

그럼 오른쪽 작업에 sample-build 작업이 추가된다.

![9-12.png](/assets/img/ncloud-sourcepipeline/9-12.png)

이번엔 dev-deploy 작업의 왼쪽 **+** 버튼을 누른다.

![9-13.png](/assets/img/ncloud-sourcepipeline/9-13.png)

SourceBuild 를 선행작업으로 선택하고 생성한다.

![9-14.png](/assets/img/ncloud-sourcepipeline/9-14.png)

그럼 오른쪽 작업에 sample-build 작업과 연결이 된 dev-deploy 작업이 추가된다.

![9-15.png](/assets/img/ncloud-sourcepipeline/9-15.png)

real-deploy 작업은 dev-deploy 를 선행작업으로 선택하고 생성한다.

![9-16.png](/assets/img/ncloud-sourcepipeline/9-16.png)

그럼 아래와 같이 3개의 작업이 연결된 파이프라인이 생성된다.

![9-17.png](/assets/img/ncloud-sourcepipeline/9-17.png)

이제 Trigger 를 설정해야한다. SourceCommit 에서 push 가 발생하면 SourcePipeline 을 통해 배포가 진행되도록 설정하였다.

![9-18.png](/assets/img/ncloud-sourcepipeline/9-18.png)

최종 정보가 맞으면 파이프라인을 생성한다.

![9-19.png](/assets/img/ncloud-sourcepipeline/9-19.png)

SourcePipeline 에 보면 sample-**pipeline** 이 추가된 것을 볼 수 있다.
파이프라인을 클릭해서 상세보기로 들어가보자

![9-20.png](/assets/img/ncloud-sourcepipeline/9-20.png)

여기서 **파이프라인 실행하기**를 눌러 제대로 동작하는지 확인해보자.

![9-21.png](/assets/img/ncloud-sourcepipeline/9-21.png)

그 전에 예상 했던대로 동작하는지 확인해보기 위해 [로드 밸런서 구축](/docs/ncloud-sourcepipeline/2023-06-19-load-balancer/) 에서 만들었던 파이썬 기반 모니터링을 통해 살펴보자.
1초 간격으로 1000번 실행되도록 했기 때문에 파일을 실행 후 **파이프라인 실행**하기를 누른다.

그 후 **작업결과** 탭으로 이동해보면 빌드부터 진행중인 것을 볼 수 있다.

![9-22.png](/assets/img/ncloud-sourcepipeline/9-22.png)

또한 SourceBuild 서비스에 들어가 작업 결과를 보면 열심히 빌드 중인 것을 볼 수 있다.

![9-23.png](/assets/img/ncloud-sourcepipeline/9-23.png)

빌드 작업이 끝나면 그 뒤에 작업인 dev-deploy 작업이 진행된다.

![9-24.png](/assets/img/ncloud-sourcepipeline/9-24.png)

dev-deploy 작업이 끝나면 real-deploy 작업이 진행된다.

![9-25.png](/assets/img/ncloud-sourcepipeline/9-25.png)

SourceBuild 와 마찬가지로 SourceDeploy 에 들어가서 작업 결과를 보면 dev, real 각각 시나리오가 진행되는 것을 볼 수 있다.

![9-26.png](/assets/img/ncloud-sourcepipeline/9-26.png)

3개의 작업 모두 성공적으로 진행이 되었다.

![9-27.png](/assets/img/ncloud-sourcepipeline/9-27.png)

모니터링 기록을 통해 살펴보자.

빌드 과정과 deploy-dev 에서 배포 직전까지는 server1, server2 요청이 번갈아가며 진행된 것을 볼 수 있다.

```bash
```bash
0 10.41.151.114 484a4266-3207-4ad8-9ac7-da15127b0a3d v1
1 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
...
144 10.41.151.114 484a4266-3207-4ad8-9ac7-da15127b0a3d v1
145 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
```

server2 에 배포가 진행되는 잠깐 동안에는 2번 정도 요청이 실패하고 LB 가 server1 로 요청을 보내는 것을 볼 수 있다.

```bash
request error
148 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
request error
150 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
151 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
152 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
153 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
154 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
```

server2 가 배포가 완료되어 헬스 체크가 통과한 후 다시 server1 과 server2 요청이 번갈아가며 진행된다.
이 때 server2 의 UUID 가 바뀐 것 또한 확인할 수 있다.

```bash
166 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
167 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
168 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
169 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
170 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
171 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
172 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
173 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
174 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
```

이번엔 real-deploy 단계에서 서버가 켜질 대 잠깐 동안 서버가 응답하지 않는 것을 볼 수 있다.
LB 는 다시 서버2로만 요청을 보내고 있다.

```bash
request error
request error
177 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
178 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
179 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
180 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
181 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
182 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
183 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
184 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
185 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
186 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
187 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
188 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
189 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
```

서버1이 헬스 체크 통과한 후에는 다시 서버1과 서버2로 요청이 번갈아가며 진행된다.

```bash
190 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
191 10.41.126.159 e6a5cbc8-d6d5-44b8-b622-5693b376433e v1
192 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
193 10.41.126.159 e6a5cbc8-d6d5-44b8-b622-5693b376433e v1
194 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
195 10.41.126.159 e6a5cbc8-d6d5-44b8-b622-5693b376433e v1
196 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
213 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
```


## 테스트
v 를 v1 -> v2 로 변경하고 Commit 과 Push 를 진행하면 파이프라인이 가동이 된다.

```java
private final String v = "v1";
```

```java
private final String v = "v2";
```

![9-28.png](/assets/img/ncloud-sourcepipeline/9-28.png)

배포를 진행할 때 모니터링을 하였는데 출력을 통해 제대로 배포가 되었는지 확인해보자.

우선 번갈아가며 요청을 보내는 것을 확인할 수 있다.

```bash
0 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
1 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
2 10.41.151.114 69510160-e08d-4fc9-9e38-b548ecde0547 v1
3 10.41.151.114 69510160-e08d-4fc9-9e38-b548ecde0547 v1
4 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
```

서버2 가 기동되면서 에러를 반환한다.
그 후 서버 1로만 요청이 간다.
```bash
request error
132 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
133 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
134 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
```

서버2 에 요청이 가고 응답 결과를 보니 **v2** 를 반환하는 것을 확인해볼 수 있다.

```bash
153 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
154 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
155 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
156 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
157 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
```

서버1 이 기동될 때 에러를 반환한다.
v2 를 반환하는 서버2 로만 요청이 가는 것을 확인할 수 있다.

```bash
request error
159 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
160 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
161 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
162 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
163 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2

```

서버1 이 헬스 체크를 통과하면 다시 서버1과 서버2로 요청이 번갈아가며 진행된다.
모두 v2 를 반환해 새로운 버전이 적용된 것을 확인할 수 있다.

```bash
175 10.41.126.159 2a106a00-be53-459c-a38c-62191256babe v2
176 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
177 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
178 10.41.126.159 2a106a00-be53-459c-a38c-62191256babe v2
179 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
180 10.41.126.159 2a106a00-be53-459c-a38c-62191256babe v2
181 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
182 10.41.126.159 2a106a00-be53-459c-a38c-62191256babe v2
183 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
```

