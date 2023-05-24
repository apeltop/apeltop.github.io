---
title: GCP 에 MySQL 만들기
layout: doc
subtitle: MySQL 서버 구축
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`GCP 에 외부에서 접속할 수 있는 DB 만들기`

## 왜 GCP 인가
3대 클라우드 기업은 AWS, Azure, GCP 이다.
이번 프로젝트에서는 DB, Serverless Server, Storage 정도라 사실 어떤 서비스를 써도 크게 상관이 없다.
클라우드 3사 모두 1년 동안 일부 무료로 제공하거나 크레딧을 제공하기 때문에 어떤 서비스를 사용해도 크게 상관이 없다.
물론 점유율을 생각해서 AWS 를 사용하는 것이 범용성에는 좋겠지만 개인적으로는 GCP 를 많이 사용했고 괜찮은 사용자 경험을 주기 때문에 주로 사용한다.

## GCP 에서 MySQL 만들기
### project 생성
GCP 는 계층 구조를 아래와 같은 이미지로 구성되어있다.

![cloud-hierarchy.svg](https://cloud.google.com/resource-manager/img/cloud-hierarchy.svg)


DBDT 이라는 이름인 프로젝트를 생성한다.

![7-1.png](/assets/img/dbdt/7-1.png)

### instance 생성
SQL 을 클릭해서 들어간다.

![7-2.png](/assets/img/dbdt/7-2.png)

생성한 인스턴스가 없기 때문에 아래와 같은 화면이 나온다. 여기서 **CREATE INSTANCE** 를 클릭한다.

![7-3.png](/assets/img/dbdt/7-3.png)

MySQL, PostgreSQL, SQL Server 를 선택할 수 있게 되어있는데 MySQL 을 선택한다.

![7-4.png](/assets/img/dbdt/7-4.png)

한 번도 생성한 적이 없기 때문에 Compute Engine API 를 허용해줘야한다. 이는 처음만 해주면 된다.

![7-5.png](/assets/img/dbdt/7-5.png)

#### instace 설정

instance ID 는 dbdt 라고 설정하였다.
**패스워드는 GENERATE 를 누르고 따로 기록해두었다.**

![7-6.png](/assets/img/dbdt/7-6.png)

Version 은 MySQL 8.0 으로 설정하였다.

![7-7.png](/assets/img/dbdt/7-7.png)

region 과 zone 은 asia-northeast3 (서울) 로 설정하였다.
필요에 따라서는 HA(High Availability) 를 위해 Multiple zones 을 설정하면 된다.

![7-8.png](/assets/img/dbdt/7-8.png)

Machine Type 은 가장 낮은 스펙인 Shared core 에 1 vCPU, 0.6 GB 를 설정하였다.
이 때 가격은 한 달의 16,321.83 이 예상된다.

![7-16.png](/assets/img/dbdt/7-16.png)

Storage Type 은 SSD 10GB 로 구성하였다. 

![7-9.png](/assets/img/dbdt/7-9.png)

외부에서 접속할 수 있도록 Public IP 로 설정하였다.
New network 부분에서 0.0.0.0/0 을 설정하지 않으면 [SQL Auth Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy) 를 통해서만 접근이 가능하다. 

![7-10.png](/assets/img/dbdt/7-10.png)

백업 설정도 해주었다.

![7-11.png](/assets/img/dbdt/7-11.png)

Query Insights 는 느린 쿼리를 식별하고 최적화하는 데 도움이 되는 기능이다.
그러므로 enabled 로 설정하였다.

![7-13.png](/assets/img/dbdt/7-13.png)

최종적인 설정은 아래와 같다.

![7-14.png](/assets/img/dbdt/7-14.png)

생성을 시작하면 아래와 같이 좀 기다리라고 나오고 만들어진다.

![7-15.png](/assets/img/dbdt/7-15.png)

인스턴스가 만들어지고나면 IP 주소와 connection name 을 확인할 수 있다.

![7-17.png](/assets/img/dbdt/7-17.png)

## 연결 확인
PyCharm 을 통해서 연결을 확인해보니 연결이 되는 것을 확인할 수 있다.

![7-18.png](/assets/img/dbdt/7-18.png)



