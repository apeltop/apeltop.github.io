---
title: prod 환경에서 성능 테스트
layout: doc
subtitle: Spring 내에서 데이터를 캐싱하여 성능을 향상시키기
author: sunshine@ptokos.com
tags: [ locust, performance ]
comments: true
---

## 목표

디비 스케일업을 제외하고 아래 3가지 방법을 사용하여 실제 production 환경에서 성능을 테스트하기

- 기존 코드
- JPA Batch Size
- Cache 사용

## 성능 테스트 결과

### 환경

- Cloud : Google Cloud Platform
- Product : [Cloud Run](https://cloud.google.com/run?hl=en) - 완전 관리형 컨테이너 서비스
- CPU : 2
- Memory : 1 GiB
- Maximum number of instances : 10
- Maximum concurrent requests per instance : 10

### 기존

100명까지 사용자가 늘리면서 성능을 테스트한 결과이다.

- Total Requests per Second : 20
- 평균 Response Time : 2초 -> 6초

Total Request per Second 를 보면 10 부근에서 20으로 올라가는데 이는 관리형 컨테이너 서비리스 Cloud Run 에서 오토스케일링을 통해 성능 향상이 이루어진 것으로 보인다.

![techcollection-performance-improve-5-1.png](/assets/img/techcollection-performance-improve/5-1.png)

테스트 진행하는 동안 Container Instance 는 3개가 만들어졌다.
![techcollection-performance-improve-5-2.png](/assets/img/techcollection-performance-improve/5-2.png)

### JPA Batch Size 조정

동시 접속자가 100명이 되니 Total Requests per Second 가 100 -> 20 으로 급격하게 떨어졌다. 최대 Response Time 은 그와 반대로 급격하게 증가하였다.

- Total Requests per Second : 100
- 평균 Response Time : 0.5초 -> 1.5초

![techcollection-performance-improve-5-3.png](/assets/img/techcollection-performance-improve/5-3.png)

Container Instance 는 최대 크기인 10개 까지 만들어졌다.

[JPA Batch Size 설정](/docs/techcollection-performance-improve/2024-02-02-jpa-batch-size/) 에서 확인하였듯 DB 문제이지만 응답 시간이 길어져
instance 에서 처리해야하는 request 가 많아져 instance 크기가 늘어난 것이다.

이를 통해 유의하게 얻을 수 있는 정보는 DB 문제로 인해 응답 시간이 길어지면 Cloud Run 서비스에서는 instance 를 늘려서 처리하려고 한다는 것이다.
연쇄적인 문제가 발생할 수 있다는 것이다.

[[Cloud Run] About instance autoscaling in Cloud Run services](https://cloud.google.com/run/docs/about-instance-autoscaling#splits)

![techcollection-performance-improve-5-4.png](/assets/img/techcollection-performance-improve/5-4.png)

### Cache 사용

Total Requests per Second 가 JPA Batch Size 조정 한 경우인 100에서 300 으로 대략 3배의 성능 향상이 이루어졌다.
차트 처음에 Response Time 이 높은 이유는 서버리스라 서버가 기동되면서 느린 응답을 반환한 것으로 보인다.

- Total Requests per Second : 300
- 평균 Response Time : 0.5초 -> 0.5초

![techcollection-performance-improve-5-5.png](/assets/img/techcollection-performance-improve/5-5.png)

대략 70 명 사용자가 될 때 RPS 가 250 으로 그 이후로는 크게 증가하지 않은 부분이 아쉽다.
그렇지만 안정적인 성능을 보여주고 있다. 평균, 최대 응답 시간이 굉장히 안정적으로 유지되고 있다.

![techcollection-performance-improve-5-6.png](/assets/img/techcollection-performance-improve/5-6.png)

심지어 instance 를 9개를 사용해 최대인 10개까지 사용하지도 않았다.

### DB 사용량 비교

3개의 솟은 부분은 보이는데 이는 순서대로 아래와 같다.

1. 기존
2. JPA Batch Size 조정
3. Cache 사용

여기서 당연하면서도 재밌는 것은 Cache 를 사용했을 때 성능이 가장 좋지만 DB 사용량이 가장 적다는 것이다.

![techcollection-performance-improve-5-7.png](/assets/img/techcollection-performance-improve/5-7.png)

## 결론

기존 환경에서 Cache 로 성능을 향상시켰다.

수치로 보면 Total Requests per Second 가 20 -> 300 으로 1500% 이상 성능이 향상되었다.
응답 시간 또한 6초 -> 0.5초로 줄였다.

물론 CPU 수를 늘리고 instance 수를 늘리면 성능을 더 향상시킬 수 있겠지만 사이드 프로젝트로 가장 효율적인 방법을 찾다보니 Cache 를 적용한 성능으로 만족하려한다.

한편으로는 Next.js 를
사용해서 [ISR](https://nextjs.org/docs/pages/building-your-application/data-fetching/incremental-static-regeneration)(
Incremental Static Regeneration) 를 사용하면 성능이 얼마나 향상될지 궁금하다.
사실 이 방법이 가장 깔끔해보이기는 한다. 이유는 스프링에서는 스케쥴링으로 Cache 를 직접 관리해야하기 때문이다.
그래야 새로운 데이터가 생겼을 때 Cache 를 갱신할 수 있기 때문이다.

