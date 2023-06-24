---
title: SourceBuild 를 통해 배포 파일 생성
layout: doc
subtitle: SourceBuild 를 통해 배포인 jar 파일 생성
author: sunshine@ptokos.com
tags: [ncloud]
comments: true
---

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




























