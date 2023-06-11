---
title: 리팩토링 - api prefix 추가
layout: doc
subtitle: api 에 버전 추가하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`/api/users 가 아니라 /api/v1/users 로 버전을 추가하고 싶다.`

## 왜 prefix 를 추가해야하는가
지금 당장 만들 때는 /api/users 로 만들어도 불편함을 모르겠지만 나중에 서비스가 운영되면서 로직을 다르게 처리해야하는 경우가 생긴다.
기존의 사용자는 문제 없이 돌아가야하며 업데이트 버전을 사용하는 사람들에게는 다른 로직을 적용해야한다면 어떻게 분기를 처리를 할 수 있을까. 
뭐든 방법은 여러가지이지만 버전을 추가하는 것이 가장 간단하고 편리하다.

FastAPI 공식 문서의 Bigger Applications - Multiple Files 중 [Include the same router multiple times with different prefix](https://fastapi.tiangolo.com/tutorial/bigger-applications/#include-an-apirouter-in-another)를 보면 prefix 로 **/api/v1** 또는 **/api/latest** 를 사용하기를 권하고 있다.

## prefix 사용 에제
### [네이버 map](https://map.naver.com/)
![29-1.png](/assets/img/dbdt/29-1.png)

### [SendBird](https://sendbird.com)
![29-2.png](/assets/img/dbdt/29-2.png)

### [Upbit](https://upbit.com/)
![29-3.png](/assets/img/dbdt/29-3.png)

## 코드에 적용하기
지금 main.py 에서는 7가지의 endpoint 가 있기 때문에 우선 /api -> /api/v1 으로 치환해준다.

![29-4.png](/assets/img/dbdt/29-4.png)

이제는 나중에 현상은 유지하며 버전에 따라 다르게 처리를 해야할 때 v2 로 api 를 관리하면 된다.
