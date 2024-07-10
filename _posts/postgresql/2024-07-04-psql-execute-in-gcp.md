---
title: psql 로 GCP 에서 SQL 실행하기
author: sunshine@ptokos.com
categories: [mysql]
---

## 목표
GCP 에서 SQL 을 실행할 때 psql 을 사용하는 방법에 대해 알아보자.

## 상황
sql 파일만 전달 받고 디비를 새로 구축해야한다. Datagrip IDE 를 통해 sql 를 통해 구축하려 하였다.
그러나 COPY 명령어가 실행이 되지 않았다. 찾아보니 COPY 명령어를 사용하면 속도가 빠르다고 데이터를 넣을 때 사용하는 것으로 파악되었다.

## 방법
2가지 방법이 떠올랐다. 

1. COPY 문법을 Insert 구문으로 변환
2. psql 을 사용해서 sql 파일을 실행

사실 1번 방법을 선택해서 코드를 공유하고 싶은 욕구가 살짝 느껴졌으나 가능하면 2번이 쉽고 빠르다고 생각되어 2번 방법을 선택하였다.

### psql 실행
#### DB 접속
우선 GCP 콘솔에 접속해서 Cloud Shell 를 실행한다.

instance name 을 확인 후 아래 명령어를 실행한다.
아래 명령어를 입력 후 접속할 유저를 user 인자 값으로 넣고 비밀번호를 입력하면 된다.
```bash
gcloud sql connect myinstance --user=postgres
```

[Connect to Cloud SQL for PostgreSQL from Cloud Shell](https://cloud.google.com/sql/docs/postgres/connect-instance-cloud-shell)

#### psql 실행
우선 psql 를 파일로 실행시킬 것이다. 이유는 대략 300MB 정도 되는 sql 파일이기 때문이다.

브라우저 환경인 Cloud Shell 에서도 파일 업로드를 지원하는데 작은 파일일 경우에만 사용하는 것을 추천한다.
속도가 꽤나 느리기 때문이다. 

가장 좋은 방법은 Cloud Storage 에 파일을 옮긴 후 Cloud Shell 에서 다운로드 받는 것이다.
Cloud Storage 에 파일을 올린 후 아래와 같은 명령어를 입력해준다.

필자의 경우는 대략 2초 정도 걸렸다. 굉장히 빠른 속도를 보여준다.

```bash
gcloud storage mv gs://mybucket/myfile.sql .
```

그 후 아래 명령어로 psql 을 실행시킨다.

-U 는 유저명, --host 는 호스트명, -d 는 DB 이름, -f 는 파일명을 의미한다. 

```bash
psql -U postgres --host=HOST -d DB_NAME -f myfile.sql 
```

그럼 파일이 실행되면서 결과가 출력된다.

이렇게 sql 파일만 받고 디비를 구축해야하는 상황에서 psql 을 사용해서 쉽게 구축할 수 있었다.
