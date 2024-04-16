---
title: GCP Oracle DB 를 Compute Engine 으로 연결하기 (스택 선정) - 1
author: sunshine@ptokos.com
categories: [gcp]
---

## 해결 방법
> 저렴한 비용으로 Compute Engine 을 사용해 Oracle DB 에 연결하기.

## 주어진 상황
> 굉장히 작은 프로젝트에서 Oracle DB 를 사용해야한다.

### 방안 1. AWS RDS 에서 Oracle DB 사용하기
Azure 와 GCP 에서는 Oracle 을 제공하지 않는다.
반면 AWS RDS 에서는 Oracle DB 를 제공하고 있다.

![1.png](/assets/img/gcp/gcp-oracle-compute-engine-1.png)

instance configuration 에서 가장 작은 인스턴스 타입이 db.m5.large 이다.
이 인스턴스 타입은 2 vCPU, 8GB RAM 을 사용한다.

![2.png](/assets/img/gcp/gcp-oracle-compute-engine-2.png)

아래와 같이 가장 저렴하게 하기 위해 Single-AZ 로 구성하였다.

![3.png](/assets/img/gcp/gcp-oracle-compute-engine-3.png)

월에 176.21 USD 이기 때문에 한화로 1400원을 가정하면 약 246,000원이다.

![4.png](/assets/img/gcp/gcp-oracle-compute-engine-4.png)

### 방안 2. Oracle Cloud 사용하기
Oracle Cloud 에서는 Oracle DB 를 제공하고 있다. [Why Oracle Cloud Infrastructure over Amazon Web Services](https://www.oracle.com/cloud/oci-vs-aws/) 를 보면 Oracle Cloud 를 사용하는게 더 좋은 것을 나열하고 있다.
물론 단점까지 고려한다면 AWS RDS 를 사용하는 것이 더 좋을 수도 있다. 그러나 아무래도 Oracle 회사에서 제공하기에 더 좋은 면도 있는 것 같다.

[Pricing Calculator](https://www.oracle.com/cloud/costestimator.html) 를 사용해보면 최소 비용으로 약 190 USD 이다.
이를 한화로 계산하면 약 260,000원이다.

![5.png](/assets/img/gcp/gcp-oracle-compute-engine-5.png)

### 방안 3. Compute Engine 사용하기
Compute Engine 에 Oracle DB 를 설치하고 사용하는 방법이다.
가격은 ec-medium 인스턴스 타입을 사용하면 월 45,000 원 정도이다. 

![6.png](/assets/img/gcp/gcp-oracle-compute-engine-6.png)

### 선택
만약 제공해야하는 서비스가 신뢰도가 중요하다면 AWS RDS 와 Oracle Cloud 를 고려했을 것 같다.
그러나 이번 경우에는 신뢰도보다는 거의 테스트 용도로 간단하게 사용하는 것이기에 **Compute Engine** 을 사용하기로 결정했다.

