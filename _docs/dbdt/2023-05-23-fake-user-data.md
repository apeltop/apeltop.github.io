---
title: Faker 사용해서 회원 목록 반환하기
layout: doc
subtitle: 가짜 회원 정보 반환하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것

`DB 연동 전 회원 정보를 반환하는 API`

회원 가입 기능을 만들고 실제 DB 정보를 반환해도 되지만 그럼 규모가 커지기 때문에 우선 가짜 정보지만 반환하고 화면에서 사용할 수 있도록 하면 더 수월한 작업이 가능하다.
이렇게 우선 만들어 놓고 화면과 연동하며 필요 정보를 더 구체화하고 시나리오에 따라서 분명 추가적인 개발이 필요하기 때문이다. 가짜 데이터를 사용하면 전체적인 그림을 빠르게 볼 수 있단느 장점이 있다.

## 로직 구상

[Faker](https://github.com/joke2k/faker) 라이브러리를 사용하여 가짜 회원 정보를 반환하는 API 를 만들어보자.

## API 만들기

### 반환하고자하는 데이터 포멧

```json
{
  "users": [
    {
      "user_id": ...,
      "name": ...,
      "email": ...,
      "image_url": ...,
      "register_date": ...
    }
  ]
}
```

### 코드 작성

Faker 를 설치한다.

```python
pip install Faker
```

Faker 를 사용할 수 있도록 import 한다.

```python
from faker import Faker
fake = Faker()
```

/api/user/list 로 접근 시 100명의 가짜 회원 정보가 반환되도록 한다.

```python
@app.get('/api/user/list')
async def user_list():
    return {
        'users': [{
            'user_id': fake.uuid4(),
            'name': fake.name(),
            'email': fake.email(),
            'image_url': fake.image_url(),
            'register_date': fake.date_time_this_month(),
        } for _ in range(100)]
    }
```

## 결과 확인

/docs 로 접근하여 확인한다.

![6-1.png](/assets/img/dbdt/6-1.png)

![6-2.png](/assets/img/dbdt/6-2.png)


