---
title: Singleton
layout: doc
subtitle: Singleton 패턴
author: sunshine@ptokos.com
tags: [design-pattern, JS]
---

## 간략 정의
> The Singleton Pattern ensures a class has only one instance, and provides a global point of access to it
> 
> `싱글톤 패턴은 클래스에 하나의 인스턴스만 가지도록한다. 그리고 전역에서 접근할 수 있도록 제공한다.`
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

싱글톤 패턴은 프로그램에서 하나의 인스턴스만 생성하고 그 인스턴스를 사용하도록 하는 패턴이다.
사용되는 곳은 전역 변수, 전역 인스턴스 등에 사용된다. 자바의 언어로 싱글톤 패턴을 보면 항상 thread-safe 가 나온다.
찰나의 순간에 인스턴스가 생성되지 않은 싱글톤 클래스에 2개 이상의 요청이 들어왔을 때 인스턴스가 2개 이상이 생길 수 있기 때문이다.
그렇기에 thread-safe 를 보장하기 위해 synchronized 를 사용한다. 필자의 경우 javascript 를 기준 언어로 사용하는데 딱히 thread 관련 문제를 찾아보기는 어렵다.
해당 문제를 해결할 수 있는 기법은 몇 가지를 찾아 발생 가능한지 테스트 해보려했으나 2개 이상의 인스턴스가 생기는 경우를 발생시키지 못했다. 
여하튼 thread-safe 문제를 염두해둘 필요는 있다.

혹자의 경우 그냥 전역에서 접근할 거면 변수로 선언하면 되는 것 아닌가 의문을 제기할 수 있다. 
일리있는 말이지만 싱글톤 패턴은 lazy initialization 을 지원한다. 즉 필요한 순간에 생성된다는 것이다. 

### Keyword
getInstance : `새로 생성된 인스턴스 혹은 기존의 인스턴스를 반환한다.`

### Class Diagram
#### 패턴 적용
![Alt text](/assets/img/design-pattern/singleton-1.png)


### Code
#### 기존
이 코드의 경우는 얼마든지 중복적으로 인스턴스를 생성할 수 있다. 
인스턴스 생성을 방지하려면 여러 코드가 덕지덕지 추가되어야할 것이다.

```typescript
class Connection {}

class DBConnection {
  private connection: any;
  constructor() {
    this.connection = new Connection();
  }

  public query(query: string): any {
    this.connection.query(query);
  }

  public close(): void {
    this.connection.close();
  }
}

const conn = new DBConnection();
conn.query('SELECT * FROM USERS');
conn.close();
```

#### 패턴 적용
```typescript
class DBConnection {
  private static instance: DBConnection;
  private connection: any; // this should be the DB connection object
  private constructor() {
    // Initialize the connection to the database here
  }

  public static getInstance(): DBConnection {
    if (!DBConnection.instance) {
      DBConnection.instance = new DBConnection();
    }
    return DBConnection.instance;
  }

  public query(query: string): any {
    // Execute the query using the connection and return the result
  }

  public close(): void {
    // Close the connection to the database
  }
}

const conn = DBConnection.getInstance();
conn.query('SELECT * FROM USERS');
conn.close();
```

### 실제 사용 느낌
싱글톤 패턴의 경우 개념이 간단하다보니 이해를 하는데 어렵지는 않았다. 
좀 익숙해져서 그런지 이번 포스팅을 하면서 다시금 느낀 점은 없다.
