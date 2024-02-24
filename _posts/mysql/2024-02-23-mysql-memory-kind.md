---
title: MySQL Global buffers, Global caches, Session buffers 
author: sunshine@ptokos.com
categories: [mysql]
---


## 목표
Global buffers, Global caches, Session buffers 에 대해 알아보자.

## Memory Allocation
MySQL 의 메모리는 Global 과 Session 으로 나뉜다. Session 은 Connection 으로 불리기도 한다.

Global Memory 는 모든 스레드에 공유가 된다.

Session Memory 는 클라이언트 스레드마다 할당된다. 스레드 간 공유되지 않는다.
스레드는 요청 당 하나씩 생성되고 요청이 끝나면 제거된다.

## Global buffers
Global buffers 는 서버가 시작될 때 할당되고 종료될 때 해제된다.

### InnoDB Buffer Pool
InnoDB Buffer Pool 은 MySQL 인스턴스에서 메모리를 가장 많이 사용하는 부분이다.
[innodb_buffer_pool_size](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_buffer_pool_size) 로 설정한다.
InnoDB Buffer Pool 은 InnoDB 테이블의 데이터와 인덱스, change buffer, adaptive hash index 등을 저장한다.

### InnoDB log buffer
InnoDB log buffer 는 redo log 를 기록하는 버퍼이다.
innodb_log_buffer_size 로 설정한다. 기본 값은 16MB 이다.

> [redo log](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html) 란 
> 
> 시스템이 비정상적으로 종료되었을 때, 데이터베이스를 복구하는데 사용되는 로그이다. 

InnoDB Architecture 를 보면 redo log 가 어떻게 사용되는지 알 수 있다.
![innodb-architecture-8-0.png](https://dev.mysql.com/doc/refman/8.0/en/images/innodb-architecture-8-0.png)

### Key buffer size
MyISAM 테이블의 인덱스를 저장하는 버퍼이다. 대부분의 경우 InnoDB 를 사용하기 때문에 설정할 필요가 없다.

### Query cache size
Query cache 는 쿼리 결과를 캐싱하는데 사용된다. MySQL 8.0 부터는 제거되었다.
5.7 까지는 disabled 가 기본 값이었고, 5.7.20 이후부터는 deprecated 가 되었다.

## Global caches
Global caches 는 모든 커넥션에 공유된다.

### Table cache
table 의 메타데이터를 캐싱하는데 사용된다. 

table_open_cache 와 table_definition_cache 로 나누어진다.

table_open_cache 는 여러 세션이 동시에 테이블에 접근할 수 있으며 이떄 각 세션은 독집적으로 테이블을 사용할 수 있다.

table_definition_cache 는 테이블의 정의를 캐싱하는데 사용된다.

### Thread cache
스레드를 재사용하는데 사용된다. 스레드를 재사용하면 새로운 스레드를 생성하는데 드는 비용을 줄일 수 있다.

## Session buffers
Session buffers 는 각 세션에만 할당된다.

아래와 같은 버퍼들이 있다.

- sort_buffer_size
- join_buffer_size
- read_buffer_size
- read_rnd_buffer_size

### Binary log cache
트랜잭션이 실행중일 때 변경된 Binary log 를 캐싱하는데 사용된다.

### Temporary tables
GROUP BY, ORDER BY, DISTINCT 등을 사용할 때 임시 테이블을 생성하는데 사용된다. 
1차적으로는 메모리에 생성되는데 maximum size 가 초과하게 되면 디스크에 저장된다.





