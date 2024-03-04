---
title: MySQL Index Covering 
author: sunshine@ptokos.com
categories: [mysql]
---


## 목표
MySQL 의 Index Covering 에 대해 알아보자.

## Index Covering
> Index Covering 이란 **SELECT** 문에서 **INDEX** 만으로 **모든 필드**를 조회하는 것을 말한다.

### 시나리오
users 테이블이 아래와 같이 있다.

```sql
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  age INT NOT NULL
);
```

email 인덱스를 생성한다.
```sql
CREATE INDEX idx_email ON users (email);
```

이때 아래와 같은 쿼리를 실행한다.
```sql
SELECT id, name FROM users WHERE email BETWEEN 'a' AND 'd';
```

이 때 실행 계뢱은 아래와 같다.
1. email 인덱스를 사용하여 조회한다. ->  인덱스를 사용해 10만 건 레코드 조회 
2. id, name 필드를 조회한다. -> 약 10만 건 레코드 읽기

여기서 문제는 레코드 읽기 시 I/O 가 발생한다는 것이다. 

#### Index Covering 사용
여기서 Index Covering 을 사용하면 I/O 가 발생하지 않는다.
```sql
SELECT id, email FROM users WHERE email BETWEEN 'a' AND 'd';
```

SELECT 시 가져오는 필드가 name 이 아니라 email 로 변경하였다.
이렇게 되면 인덱스 정보만으로 id 와 email 정보를 가져올 수 있기 때문에 레코드를 읽을 필요가 없다.

id 도 인덱스 정보로 가져올 수 있는 이유는 email 은 Secondary Index 이기 때문이다.
Secondary Index 의 값은 레코드의 주소가 아닌 Primary Key 값이다.
그러기에 Secondary Index 인 email 을 사용하면 Primary Key 값도 가져올 수 있다.

### Index Covering 의 장점과 단점
#### 장점
- I/O 가 발생하지 않는다.
- 레코드를 읽지 않기 때문에 굉장히 빠르다.

#### 단점
- index 를 무리해서 늘리면 크기도 커지고 데이터 저장 및 수정 시 성능이 떨어진다.







