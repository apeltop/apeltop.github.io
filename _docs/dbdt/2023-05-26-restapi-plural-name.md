---
title: RESTAPI 복수형 명사 사용하기
layout: doc
subtitle: RESTAPI 복수형 명사로 Schema 알아보기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`/list 제거하기`

## /api/user/list vs /api/users
[Faker 사용해서 회원 목록 반환하기](/docs/dbdt/2023-05-23-fake-user-data/) 에서 `/api/user/list` 인 endpoint 를 만들었다.
그러나 user 와 같은 resource 뒤에 /list 를 붙이는 것은 좋은 디자인이 아니다.

/user/list 와 같이 디자인을 할 경우 endpoint 가 하나 더 생기는 반면 /users 와 같이 디자인을 할 경우 endpoint 가 하나 더 생기지 않는다.

복수명 명사를 써야하는 직관적인 이유 중에 하나는 아래와 같다.
```
GET  /users          <---> users 
POST /users          <---> users.push(data)
GET  /users/1        <---> users[1]
PUT  /users/1        <---> users[1] = data
GET  /users/1/posts  <---> users[1].posts
POST /users/1/posts  <---> users[1].posts.push(data) 
```

