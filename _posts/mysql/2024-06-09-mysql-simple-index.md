---
title: MySQL 단순 index 를 사용했을 때 성능 비교
author: sunshine@ptokos.com
categories: [mysql]
---

## 목표
> MySQL 에서 단순 index 를 사용했을 때 성능을 비교해보자.

## 상황
### event 테이블 행 개수
> 361636 행

### userId 가 가진 event 행 개수
> 211137 행

### 쿼리
```sql
SELECT SUM(TIMESTAMPDIFF(SECOND, startDateTime, endDateTime)) as total_time
     , userUserId                                             as userId
FROM event
WHERE userUserId = '58fe3488-faf0-4fea-8a88-c2d31acc7ce0'
  AND startDateTime between '2024-07-01' and '2024-07-31'
GROUP BY userUserId;
```

### 상황1 - index 사용하지 않은 경우

밑에서부터 위로 읽어나가면서 실행된다.

Full Scan 으로 데이터를 모두 읽고  Filter 에서 조건에 맞는 데이터만 추출한다.

361636 행 -> 8735 행 -> 1 행으로 쿼리문이 실행되었다.

| Operation                        | Rows   | Actual Rows | Total Cost | Actual Total Time  | Actual Startup Time | Raw Desc                          |
|----------------------------------|--------|-------------|------------|-------------------|---------------------|-----------------------------------|
| Aggregate (Group aggregate)      | 4056   | 1           | 37777.29   | 262.499           | 262.499      | sum(timestampdiff(SECON..         |
| Filter                           | 4056   | 8735        | 37371.65   | 260.781           | 0.108        | (‘event’.userUserid = ‘58...     |
| Full Scan (Table scan) table: event; | 365114 | 361636      | 37371.65   | 115.046           | 0.093        |                                   |

![index-사용하지-않은-경우.png](/assets/img/mysql/mysql-simple-index-1.png)


### 상황2 - index 사용한 경우 (startDateTime 기준)
Index Scan 을 통해 데이터를 읽는다. 그 후 일자가 조건에 부합하는지 Filter 를 통해 확인한다.

11171 행 -> 8735 행 -> 1 행으로 쿼리문이 실행되었다.

| Operation                                | Rows | Actual Rows | Total Cost | Actual Total Time  | Actual Startup Time | Raw Desc                           |
|------------------------------------------|------|-------------|------------|-------------------|---------------------|------------------------------------|
| Aggregate (Group aggregate)              | 2114 | 1           | 9726.5     | 37.044            | 37.043     | sum(timestampdiff(SE...            |
| Filter                                   | 2114 | 8735        | 9515.06    | 35.758            | 0.459       | ('event'.userUserid = ...          |
| Index Scan (Index. table: event; index: startDateTime_index over; | 21144 | 11171      | 9515.06    | 32.762            | 0.03         | ('2024-07-31 00:00:00', ...        |

![index-사용한-경우-1.png](/assets/img/mysql/mysql-simple-index-2.png)

### 상황3 - index 사용한 경우 (userUserId + startDateTime 기준)

Index Scan 을 통해 데이터를 읽는다. filter 는 사용하지 않는다.

8735 행 -> 1 행으로 쿼리문이 실행되었다.

| Operation                                                        | Rows | Actual Rows | Total Cost | Actual Total Time | Actual Startup Time | Raw Desc                           |
|------------------------------------------------------------------|------|-------------|------------|-------------------|-------------|------------------------------------|
| Aggregate (Group aggregate)                                      | 17722 | 1          | 9747.36    | 30.522            | 30.522      | sum(timestampdiff(SE...            |
| Index Scan (Index table: event; index: userIdstartDateTime_index over; | 17722 | 8735       | 7975.16    | 29.257            | 0.027       | (userUserId = '58fe34...            |

![index-사용한-경우-2.png](/assets/img/mysql/mysql-simple-index-3.png)


## 결론
상황1(인덱스를 사용하지 않는 경우) 는 Full Scan 으로 데이터를 가져오는데 115.046 초가 걸렸다. 
그러나 인덱스를 사용하는 경우는 32.762, 29.257 초가 걸렸다. 거의 4배에 가까운 성능 향상이 있었다. 

상황 2 & 3 를 비교해보면 시간 차이가 근소하지만 인덱스 사용한 상황 2 가 더 빨랐다.
상황 2 에서는 filter 가 있다. filter 가 있는 이유는 WHERE 조건에서 사용하는 컬럼이 userUserId 와 startDateTime 이기 때문이다.
mysql 엔진은 인덱스를 사용할 수 있는 컬럼이 있을 때 인덱스를 사용해서 먼저 데이터를 가져온다.
그러나 당연히 인덱스를 사용할 수 없는 컬럼이 있을 때는 인덱스를 사용하지 못한다. 그래서 filter 가 있는 것이다.

상황 2 와 3이 차이가 크게 없어보일 수 있으나 데이터가 더 많아질 경우 차이가 점점 더 벌어지게 된다.
그 이유는 위에서 설명하였듯 상황 3은 유저 데이터가 많이 싸여도 userUserId + startDateTime 조합으로 데이터를 가져오는 행은 크게 변하지 않기 때문이다.
그러나 상황 2의 경우는 데이터가 많아질수록 startDateTime 조합으로 데이터를 가져오는 행이 늘어나기 때문에 성능 차이가 발생한다.

이 글은 아주 간단하고도 당연한 내용이지만, 그런 내용을 글로 남기어 보고 싶었다.

