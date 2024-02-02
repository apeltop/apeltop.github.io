---
title: JPA Batch Size 설정
layout: doc
subtitle: JPA Batch Size 설정을 통해 성능을 향상시키기
author: sunshine@ptokos.com
tags: [locust, performance]
comments: true
---


## 기존 상황
동접자 100명까지 늘려가며 성능 테스트를 진행해보자.

### 테스트 환경
#### Server
로컬에서 진행하였다. 
로컬에서 진행 후 배포한 서버에 대한 성능 테스트를 진행하려한다.

#### DB
- vCPUs: 1
- Memory: 1.7 GB

![techcollection-performance-improve-2-1.png](/assets/img/techcollection-performance-improve/2-1.png)

아래 차트를 통해 알 수 있는 내용을 간략히 추려보자.
- Total Requests per Second 는 30~40 사이로 유지되고 있다.
- Response Time 은 계속해서 증가하고 있다.
- 사용자 30명부터 Total Requests per Second 가 30~40 사이로 유지되고 있다. 이는 성능이 저하되고 있다는 것을 의미한다.

응답하신이 500ms 로 시작해서 3000ms 까지 증가하고 있다. 이는 성능이 저하되고 있다는 것을 의미한다.

![techcollection-performance-improve-2-2.png](/assets/img/techcollection-performance-improve/2-2.png)

## 개선
JPA 쿼리가 단건 쿼리로 계속해서 발생하고 있기에 이를 Batch Size 설정을 통해 해결해보자.

### Batch Size 설정
```
default_batch_fetch_size: 100
```

### 성능 테스트 결과
60명까지는 Response Time 도 굉장히 낮은 수치로 유지가 되고 있으나 60명 이후로는 조금씩 증가하다가 100명 이후로는 급격히 응답 시간이 증가하고 Total Requests per Second 가 20명 부근으로 떨어지는 것을 볼 수 있다.

![techcollection-performance-improve-2-3.png](/assets/img/techcollection-performance-improve/2-3.png)

CPU 사용량을 보면 피크가 2개 생겼다.

![techcollection-performance-improve-2-4.png](/assets/img/techcollection-performance-improve/2-4.png)

첫 번째는 Batch Size 적용 전이고 두 번째는 Batch Size 적용 후이다.
이것을 통해 알 수 있는 것은 Batch Size 를 적용하기 전에는 CPU 사용량이 60% 퍼센트 정도로 운용이 되면서 성능이 저하되고 있었으나 Batch Size 를 적용하고 나서는 CPU 사용량이 100% 를 달성하였다.

Batch Size 를 적용한게 100% 가 육박하는 이유는 Request 수가 증가하였기 때문이다.
Batch Size 적용 전에는 초당 보내는 Request 수가 30~40개 정도였으나 Batch Size 적용 후에는 60~70개 정도이기 때문이다.








