---
title: 암호화된 패스워드 property 추가하기
layout: doc
subtitle: 회원가입 모델에 암호화된 패스워드 property 추가하기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`평문인 패스워드를 전달 받아 암호화된 패스워드를 저장하는 property 추가하기`

## 만들기
이전 글인 [이메일 유효성 검사](/docs/dbdt/2023-06-05-validation-email/)하기를 이어서 진행한다.

### SignUpRequest 에 password 추가
SignUpRequest 에 plain_password 를 추가한다.

```python
class SignUpRequest(BaseModel):
    username: str
    email: EmailStr
    full_name: str
    plain_password: str
```

### property 추가
꼭 프로퍼티를 추가하지 않고 함수를 추가해도 되지만 프로퍼티를 추가하는 것이 더 깔끔하다.

```python
def get_hash_password():
    return hashed_password(plain_password)
```

hashed_password 라는 프로퍼티를 추가했다.

```python
class SignUpRequest(BaseModel):
    username: str
    email: EmailStr
    full_name: str
    plain_password: str

    @property
    def hashed_password(self):
        return get_password_hash(self.plain_password)
```

### 테스트 케이스 작성하기
[지난 글](/docs/dbdt/2023-06-05-validation-email/)에 만든 test_schemas.py 에 테스트 케이스를 작성한다.
```python
    def test_sign_up_password_hash(self):
        user_dict = {
            'username': 'test',
            'email': 'test@example.com',
            'full_name': 'test',
            'plain_password': 'test',
        }
        user = SignUpRequest(**user_dict)

        self.assertTrue(user.hashed_password is not None)
        self.assertTrue(verify_password(user.dict()['plain_password'], user.hashed_password))
```

이로써 자동으로 compute 되어 관리할 수 있는 property 를 추가하였다.
