---
title: Cloud Run Outbound IP 고정하기
author: sunshine@ptokos.com
categories: [GCP]
---
![Alt text](/assets/img/gcp/cloud-run-static-ip/2.png)

Cloud Run 은 Serverless 로서 동적 IP 이다. 계속해서 inbound, outbound IP 가 달라지게 된다.
이러한 특성은 여러 문제를 야기한다. Cloud Run 에서 DB 를 연결한다면 DB 는 Allow IP 를 특정할 수 없게 된다.
필자의 경우 서비스에서 문자 발송 API 를 사용하는데 문자 발송 API 는 사전에 등록된 IP 에서 요청이 들어와야 발송을 해주었다.
어쩔 수 없이 Cloud Run 를 사용하지 못하고 App Engine 을 사용했다. 계속해서 Cloud Run 으로 변경하고 싶은 마음이 들어 문자 발송만 App Engine 에 올려놓고 사용할까 싶었다.
그런데 검색을 하다보니 Serverless 제품이지만 outbound IP 를 고정할 수 있다는 것을 알게 되었다. 
사실 이전에 검색했으면 나왔을텐데 당시 '당연히 IP 는 변할 수 밖에 없지' 라고 생각하고 검색조차 하지 않았었다.
돌이켜보면 정말 미련한 행동이였다. 그렇지만 이런 경험을 통해 다음엔 더 나은 모습을 보여줄 수 있을 것 같다.

Serverless 의 outbound IP 를 고정하는데 사용되는 네트워크 기술은 [Serverless VPC Access connector](https://cloud.google.com/vpc/docs/serverless-vpc-access), [Cloud NAT](https://cloud.google.com/nat/docs/overview) 이다.

한문장으로 NAT 를 소개하면 사설 IP -> 공인 IP 로 변경하는 것이다.
여러개의 호스트가 있지만 모든 외부와의 통신을 NAT 를 거치게 하면 외부에서는 공인 IP 의 정보를 받게된다.

약간의 여담이지만 Google Cloud NAT 는 일반적인 방법의 NAT 와 다르다.
일반적인 NAT 는 NAT Router 에서 테이블표를 가지고 있다. 
그렇지만 Google Cloud NAT 는 각 인스턴스가 사설 IP, 포트를 가지고 있다. 이렇게 했을 때 장점은 NAT 를 거칠 때 정보 변환하는 과정이 필요가 없어진다는 것이다.

#### Google Cloud NAT
![Alt text](/assets/img/gcp/cloud-run-static-ip/1.svg)
[이미지 출처](https://cloud.google.com/nat/docs/overview)

#### 일반적인 NAT
![Alt text](/assets/img/gcp/cloud-run-static-ip/3.jpeg)
[이미지 출처](https://brunch.co.kr/@sangjinkang/61)


## Serverless Product 에 Outbound IP 고정하기
아래 이미지는 Cloud Run 의 outbound 요청을 Serverless VPC -> Cloud NAT -> Third party 로 보내는 다이어그램이다.
이번 포스팅에서는 아래처럼 구성해볼 것이다.

![Alt text](/assets/img/gcp/cloud-run-static-ip/2.png)
[이미지 출처](https://medium.com/google-cloud/provisioning-cloud-run-with-cloud-nat-using-terraform-e6b8d678eb85)


### 1. 서브넷 만들기
이 서브넷은  Serverless VPC Access connector, Cloud NAT, Cloud Router 가 위치하게 될 것이다.

서브넷 안에 해당 VPC 가 위치한다면 다른 컴퓨팅 자원(VPC, Compute Engine, Kubernetes 등)에서 사용하기 위한 정적 IP 를 사용하지 못하게하는 효과가 있다.

VPC 와 VPC 는 연결하기 위해서는 추가적인 작업이 필요하기 때문이다.

서브넷 생성

- 이름 : cloud-run-subnet
- range : 10.124.0.0/28
- network : default
- region : asia-northeast3

```
gcloud compute networks subnets create cloud-run-subnet \
--range=10.124.0.0/28 --network=default --region=asia-northeast3
```

### 2. Serverless VPC Access connector 만들기
Serverless VPC Access connector 생성

- 이름 : cloud-run-connector
- region : asia-northeast3
- subnet-project : PROJECT-ID
- subnet : cloud-run-subnet(서브넷 생성 때 사용한 이름)
```
gcloud compute networks vpc-access connectors create cloud-run-connector \
  --region=asia-northeast3 \
  --subnet-project=PROJECT-ID \
  --subnet=cloud-run-subnet
```


### 3. NAT(Network Address Translation) 구성하기 
#### 3.1 라우터 만들기
라우터 생성

- 이름 : cloud-run-router
- network : default
- region : asia-northeast3

```
gcloud compute routers create cloud-run-router \
  --network=default \
  --region=asia-northeast3
```

#### 3.2 정적(static) IP 생성하기
정적 IP 생성

- 이름 : cloud-run-origin-ip
- region : asia-northeast3

```
gcloud compute addresses create cloud-run-origin-ip --region=asia-northeast3
```

#### 3.3 NAT 만들기
NAT 생성
- 이름 : cloud-run-nat
- router : cloud-run-router(라우터 생성 때 사용한 이름)
- region : asia-northeast3
- nat-custom-subnet-ip-ranges : cloud-run-subnet(서브넷 생성 때 사용한 이름)
- nat-external-ip-pool : cloud-run-origin-ip(정적 IP 생성 때 사용한 이름)

```
gcloud compute routers nats create cloud-run-nat \
  --router=cloud-run-router \
  --region=asia-northeast3 \
  --nat-custom-subnet-ip-ranges=cloud-run-subnet \
  --nat-external-ip-pool=cloud-run-origin-ip
```

### 4. Cloud Run 과 Serverless VPC Access connector 연결하기
Cloud Run 과 VPC 연결하기
- 이름 : production(Cloud Run 에 배포한 이름)
- image : asia.gcr.io/~
- vpc-connector : cloud-run-connector(VPC 생성 때 사용한 이름)
- vpc-egress : all-traffic

```
gcloud run deploy production \
   --image=IMAGE_URL \
   --vpc-connector=cloud-run-connector \
   --vpc-egress=all-traffic
```


위 작업을 모두 마치면 Cloud Run 의 외부 엔드포인트에는 고정 IP 가 전달될 것이다.
