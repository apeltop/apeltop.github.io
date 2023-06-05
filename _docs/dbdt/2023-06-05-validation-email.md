---
title: 이메일 유효성 검사하기
layout: doc
subtitle: email-validator 를 사용하여 이메일 유효성 검사하기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`이메일이 유효한지 검사(validation)`

이메일 주소 형식이 맞는지 검사하는 것은 매우 빈번한 상황이다.
이메일이 필요한 곳에서는 꼭 필요로 하기 때문이다. 이메일 유효 검사를 위해서 다양한 regex 표현식이 있지만 이번 포스팅에서는 이를 직접 만들지 않고 `email-validator` 를 사용하여 이메일 유효 검사를 해보고자 한다.

## 만들기
### email-validator 설치
```bash
pip install email-validator
```

### SignUpRequest 생성
db/schemas.py 파일에 SignUpRequest 추가한다.
회원가입 시 받을 객체이다.

```python
from pydantic import BaseModel, EmailStr

...

class SignUpReqeust(BaseModel):
    username: str
    email: EmailStr
    full_name: str
```

### API 추가
```python
@app.post('/api/users')
async def user_create(user: SignUpReqeust):
    return user
```

docs 에 접근해서 email 이 user@example.com 인 값으로 테스트해보면 정상적으로 반환이 된다.

![21-1.png](/assets/img/dbdt/21-1.png)

![21-2.png](/assets/img/dbdt/21-2.png)

그러나 email 을 user@examplecom 으로 변경해서 요청하면 422 가 Status Code 로 반환되며 에러 메시지가 반환이 된다. 

![21-3.png](/assets/img/dbdt/21-3.png)

![21-4.png](/assets/img/dbdt/21-4.png)

### 테스트 케이스 작성
매번 docs 에 접근해서 테스트를 해볼 수는 없기에 테스트 케이스를 작성하는 것이 좋다.

테스트하고자하는 것은 이메일 유효성 검사 **성공**과 **실패**이다.
우선 간단한 데이터를 가지고 테스트하고 추후 필요에 따라 테스트 데이터를 추가하면 된다.

_db/test_schemas.py_ 파일 추가한다.

```python
from unittest import TestCase

from pydantic.error_wrappers import ValidationError

from db.schemas import SignUpRequest


class SchemasTest(TestCase):
    def test_sign_up_email_validation_fail(self):
        user_dict = {
            'username': 'test',
            'email': 'test',
            'full_name': 'test',
        }
        with self.assertRaises(ValidationError) as e:
            user = SignUpRequest(**user_dict)
        self.assertEqual(e.exception.errors()[0]['loc'][0], "email")
        self.assertEqual(e.exception.errors()[0]['msg'], "value is not a valid email address")

    def test_sign_up_email_validation_success(self):
        user_dict = {
            'username': 'test',
            'email': 'test@example.com',
            'full_name': 'test',
        }
        user = SignUpRequest(**user_dict)
        self.assertEqual(user.dict(), user_dict)
```

email-validation 을 사용해서 간단하게 이메일 유효성 검사하는 로직을 추가해보았다.
