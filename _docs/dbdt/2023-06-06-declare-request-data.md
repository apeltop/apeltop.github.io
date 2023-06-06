---
title: Pydantic model 예제 데이터 설정하기
layout: doc
subtitle: docs 에서  
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`docs 에서 예제 데이터를 보여주도록 하기`

## 결과물
### 변경 전
![23-1.png](/assets/img/dbdt/23-1.png)

### 변경 후
![23-2.png](/assets/img/dbdt/23-2.png)

## 만들기
참고 링크 [Declare Request Example Data](https://fastapi.tiangolo.com/tutorial/schema-extra-example/)

### SignUpRequest 에 예제 Schema 추가
기존의 SignUpRequest 에 아래 코드를 추가해주면 된다.

```python
class Config:
    schema_extra = {
        "example": {
            "username": "Gi Ju",
            "email": "communication@ptokos.com",
            "nickname": "apeltop",
            "plain_password": "L1ySZZbd)j",
        }
    }
```

그러므로 전체 코드는 아래와 같다.

```python
class SignUpRequest(BaseModel):
    username: str
    email: EmailStr
    nickname: str
    sign_up_status: str = 'WAITING_EMAIL_VERIFICATION'
    plain_password: str

    @property
    def hashed_password(self):
        return get_password_hash(self.plain_password)

    class Config:
        schema_extra = {
            "example": {
                "username": "Gi Ju",
                "email": "communication@ptokos.com",
                "nickname": "apeltop",
                "plain_password": "L1ySZZbd)j",
            }
        }
```

이로써 필요에 따라 schema_extra 의 값을 설정하여 API 문서에 이해를 도울 수 있게 되었다.

