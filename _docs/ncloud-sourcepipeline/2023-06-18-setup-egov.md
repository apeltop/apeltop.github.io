---
title: eGov 프로젝트 구축하기
layout: doc
subtitle: 내부 ip 와 버전을 출력하는 간단한 프로젝트를 구축하기
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

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













