---
title: MySQL Isolation Level 
author: sunshine@ptokos.com
categories: [mysql]
---


## 목표
MySQL 의 Isolation Level 에 대해 알아보자.

## Isolation Level
- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE

|                     | DIRT-READ                  | NON-REPEATABLE READ        | PHANTOM READ                |
|-----------------------|------------------------------|-------------------------------|-------------------------------|
| READ UNCOMMITTED | O                             | O                                  | O                                  |
| READ COMMITTED    | X                             | O                                  | O                                  |
| REPEATABLE READ | X                             | X                                  | O                                  |
| SERIALIZABLE        | X                             | X                                  | X                                  |



### READ UNCOMMITTED
READ UNCOMMITTED 는 **dirty read** 가 발생할 수 있다.

dirty read 란 다른 트랜잭션이 commit 되지 않은 데이터를 읽는 것을 말한다.

즉 A 트랜잭션에서 데이터 DATA-A 를 수정하고 commit 을 하지 않아도 B 트랜잭션에서 **A 트랜잭션에서 변경한 DATA-A 를 읽을 수 있다**.

### READ COMMITTED
READ COMMITTED 는 dirty read 가 발생하지 않으며 **consistent read** 가 보장된다.

이와 같이 다른 트랜잭션과 다르게 데이터를 읽을 수 있는 이유는 **undo log** 를 사용하기 때문이다.
데이터를 변경하게 되면 undo log 에 변경 전 데이터를 저장하고 변경 후 데이터를 저장한다.
이때 다른 트랜잭션이 변경 전 데이터를 읽게 되면 undo log 에서 데이터를 읽게 된다.

READ COMMITTED 설정으로 인해 영향을 받는게 생긴다.

첫 번째로는 UPDATE, DELETE 를 사용할 때 레코드에 lock 을 걸게되는데 where 절이 실행되고 일치하지 않는 record 의 lock 을 해제하게 된다.

두 번째로는 UPDATE 할 때 해당 record 가 이미 lock 이 걸려있다면 최신 commit 버전 데이터를 읽는다. 
where 절에 일치하다면 record 를 다시 읽고 lock 또는 대기한다. 이를 **semi-consistent read** 라고 한다.

READ COMMITTED 는 **NON-REPEATABLE READ** 가 발생할 수 있다.

**NON-REPEATABLE READ** 란 같은 트랜잭션에서 같은 데이터를 읽었을 때 다른 결과가 나오는 것을 말한다.

1. A 트랜잭션에서 WHERE name = 'apeltop' 을 조회한다. - 0 row
2. B 트랜잭션에서 UPDATE set name = 'apeltop' 실행 후 commit 한다.
3. A 트랜잭션에서 WHERE name = 'apeltop' 을 조회한다. - 1 row

### REPEATABLE READ
REPEATABLE READ 는 READ COMMITTED 에서 발생하는 NON-REPEATABLE READ 가 발생하지 않는다.
InnoDB 엔진에서는 REPEATABLE READ 가 **기본** Isolation Level 이다.

이것이 가능한 이유는 InnoDB 가 **MVCC(Multi-Version Concurrency Control)** 를 사용하기 때문이다.
**MVCC** 는 **undo log** 를 사용하여 데이터에 대하여 **여러 버전**을 가질 수 있게 된다.
이러한 특징으로 인해 Isolation Level 에 따라 다르게 데이터를 읽을 수 있다.

REPEATABLE READ 는 undo log 의 **TRX_ID**(Transaction ID) 를 사용하여 데이터를 읽게 된다.
현재 트랜잭션보다 **작은 TRX_ID 를 가진 데이터는 읽을 수 있지만** 큰 TRX_ID 를 가진 데이터는 읽을 수 없다.

그래서 READ COMMITTED 에서 발생하는 NON-REPEATABLE READ 가 발생하지 않는다.

그러나 REPEATABLE READ 는 **PHANTOM READ** 가 발생할 수 있다.

PHANTOM READ 란 같은 트랜잭션에서 같은 데이터를 읽었을 때 **유령(phantom) 과 같이 row 가 추가되거나 삭제**되는 것을 말한다.

**SELECT ... FOR UPDATE** 또는 **SELECT ... LOCK IN SHARE MODE** 을 사용하면 **PHANTOM READ** 가 발생할 수 있다.
이유는 **undo log** 에 **lock 을 걸 수 없기 때문에** undo log 에서 데이터를 가져오지 못하고 최신 commit 버전 데이터를 읽게 된다.

**Gap Lock** 을 사용하면 PHANTOM READ 가 발생하지 않는다.
Gap Lock 은 **인덱스의 범위를 lock** 하는 것을 말한다. 그래서 데이터 삽입을 제한할 수 있다.
예를 들어 WHERE BETWEEN 1 AND 10 을 사용하면 1 ~ 10 까지의 범위를 lock 하게 된다.

**Next Key Lock** 을 사용하면 PHANTOM READ 가 발생하지 않는다.
Next Key Lock 은 **Gap Lock 과 Record Lock 을 합친 것**을 말한다.
즉 Next Key Lock 은 WHERE BETWEEN 1 AND 10 을 사용하면 1 ~ 10 까지의 범위를 lock 하고 10 까지의 record 를 lock 하게 된다.

InnoDB 엔진은 REPEATABLE READ 를 사용할 때 **Gap Lock** 과 **Next Key Lock** 을 사용해서 PHANTOM READ 가 발생하지 않도록 한다.


### SERIALIZABLE
SERIALIZABLE 은 REPEATABLE READ 에서 발생하는 PHANTOM READ 가 발생하지 않는다.
그러므로 가장 격리성이 높은 Isolation Level 이다. 그만큼 **성능이 떨어진다**.
읽기 작업 시 lock 을 걸기 때문에 성능이 떨어진다.

InnoDB 는 **순수 SELECT 를 암시적으로 SELECT ... FOR SHARE 로 변경**한다.

참고 자료
- [17.7.2.3 Consistent Nonlocking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)
- [17.3 InnoDB Multi-Versioning](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)
- [17.7.2.1 Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)






