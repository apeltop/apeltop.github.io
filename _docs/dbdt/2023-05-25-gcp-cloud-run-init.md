---
title: GCP Cloud Run 초기 설정하기
layout: doc
subtitle: 테스트 Image 를 사용해서 Cloud Run 환경 구축하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`GCP Cloud Run 초기 환경 구축`

- Cloud Run 테스트 이미지 배포
- Cloud Run 실행에 필요한 환경 설정

## Cloud Run 이란
Cloud Run 은 Serverless 서비스이다.
Serverless 는 서버를 직접 구축하지 않고 서버를 사용하는 것을 의미한다.
Cloud Run 은 Docker Image 를 사용해서 서버를 구축한다.
Cloud Run 은 Docker Image 를 사용하기 때문에 어떤 언어로든 서버를 구축할 수 있다.


## Cloud Run 환경 구축하기
### Service Account 생성
Cloud Run 을 만들 때 첨부한 사진과 같이 기본 값으로 Compute Engine default serivce account 이다.
이 service account 로도 만들 수 있지만 좋은 습관은 아니다. 구글에서도 이를 권장하지는 않는다.
최소한의 권한만 가진 service account 를 만들어서 사용하는 것이 좋다.

[Service accounts](https://cloud.google.com/run/docs/configuring/service-accounts)

[Service identity](https://cloud.google.com/run/docs/securing/service-identity)

![9-8.png](/assets/img/dbdt/9-8.png)

#### 새로운 Service account 만들기
IAM & Admin > Service accounts > Create service account 

![9-9.png](/assets/img/dbdt/9-9.png)

여기서 Create service account 를 클릭해준다.

![9-10.png](/assets/img/dbdt/9-10.png)

Service account 를 cloud-run-server 로 만들어준다.

![9-11.png](/assets/img/dbdt/9-11.png)

### Test Image 설정
Container 를 사용해 운영되기 때문에 Image 를 사용해야한다.
그런데 우선 Cloud Run 환경을 구축하기 위해 테스트 이미지를 사용하려한다.
_TEST WITH A SAMPLE CONTAINER_ 를 클릭한다.

![9-1.png](/assets/img/dbdt/9-1.png)

### CPU allocation and pricing
request 가 있을 때만 CPU 가 할당되도록 설정하였다.

![9-2.png](/assets/img/dbdt/9-2.png)

### Auto Scaling
필요에 따라 자동으로 instance 수의 수를 0~10 개가 될 수 있도록 설정하였다.

![9-3.png](/assets/img/dbdt/9-3.png)

### Authentication
모든 요청을 허용하도록 설정하였다.

![9-4.png](/assets/img/dbdt/9-4.png)

### Container
기본 설정을 그대로 사용하였다.

![9-5.png](/assets/img/dbdt/9-5.png)

### Capacity
#### Memory
1 GiB

_GiB vs GB_

> 1 GB 와 1 GiB 의 차이는 1 GB = 1,000,000,000 Byte, 1 GiB = 1,073,741,824 Byte 이다. 
> 이러한 차이가 발생하는 이유는 1 GB 는 10 진법으로 인식하여 1000MB 가 된다. 
> 반면에 1 GiB 는 2 진법으로 인식하여 1024MB 가 된다.

#### CPU
2

#### Maximum concurrent requests per instance
직역하면 **인스턴스 당 최대 동시 요청 수**이다.

쉽게 이해하기 위해서 다음 그림을 보자.
concurrency 가 1인 경우 request 가 3개가 동시에 올 경우 1개의 서비스 안에 3 개의 인스턴스에 나누어 전달이 된다.
반면에 concurrency 가 80인 경우 request 가 3개가 동시에 올 경우 1개의 서비스 안에 1개의 인스턴스에서 처리하게 된다.

한 개의 인스턴스에서 얼마나 많은 요청을 처리하게 할지에 따라 설정을 해주면 된다.

![Maximum concurrent requests per instance](https://cloud.google.com/static/run/docs/images/concurrency-diagram.svg)

![9-6.png](/assets/img/dbdt/9-6.png)

### Exeuction environment
Cloud Run 은 1세대와 2세대가 있다.

2세대는 1세대와 다르게 네트워크 파일 시스템, Linux 확장성, CPU & Network 더 나은 성능을 보여준다.
단 2세대는 1세대보다 cold start 가 더 오래 걸린다. 

나중에 어떻게 될지느 모르지만 글 쓰는 시점에서는 Default 는 1세대이다. 
그러나 Cloud Run 에서 job 을 설정하면 기본 값이 2세대이다.

[About execution environments](https://cloud.google.com/run/docs/about-execution-environments)

![9-7.png](/assets/img/dbdt/9-7.png)

### Security
Service account 를 만들었던 cloud-run-server 를 선택해준다.

![9-8.png](/assets/img/dbdt/9-15.png)


### Cloud Run 확인하기
Cloud Run 을 확인하면 다음과 같이 생성된 것을 확인할 수 있다.

![9-12.png](/assets/img/dbdt/9-12.png)

hello 를 클릭해서 들어가보면 URL 이 보인다.

![9-13.png](/assets/img/dbdt/9-13.png)

클릭해서 접속해보면 테스트 IMAGE 를 통해 배포된 화면을 확인할 수 있다.

![9-14.png](/assets/img/dbdt/9-14.png)

이제 Cloud Run 을 사용할 수 있는 환경이 구축되었다.








