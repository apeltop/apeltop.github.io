---
title: DB 서버 Scale Up
layout: doc
subtitle: DB 서버 Scale Up 을 통해 성능을 향상시키기
author: sunshine@ptokos.com
tags: [locust, performance]
comments: true
---


## 기존 상황
JPA Batch Size 를 조정하니 응답 시간은 효과적으로 줄었으나 사용자 수가 증가하면서 DB CPU 사용량이 100% 를 달성하였다.

## Scale Up
### DB
- vCPUs: 1 -> 2
- Memory: 1.7 GB -> 8 GB


### 성능 테스트 결과
#### 기존
![techcollection-performance-improve-2-3.png](/assets/img/techcollection-performance-improve/2-3.png)


#### Scale Up 후
그래프만 보면 성능이 더 가파르게 안 좋아지는 것처럼 보이나 기존 그래프의 간격을 보면 실제로는 성능이 향상되었다.
기존에는 100명의 사용자에 도달했을 때 Response Time 이 6초, 10초 걸리는 응답 값이 있었던 반면 지금은 늦어도 2초 내로 응답이 완료되었다.

![techcollection-performance-improve-3-1.png](/assets/img/techcollection-performance-improve/3-1.png)

CPU 사용량 또한 60%를 조금 윗도는 정도이다.

![techcollection-performance-improve-3-2.png](/assets/img/techcollection-performance-improve/3-2.png)

## 결론
동시 사용자가 100명을 처리할 수 있도록 되었다. 그러나 150명만 되어도 버겁다. 
서버 스펙을 2배 이상 올렸지만 성능이 2배 이상 향상되지 않았다.



