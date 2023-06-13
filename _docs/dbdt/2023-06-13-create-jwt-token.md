---
title: JWT 토큰 만들기
layout: doc
subtitle: JWT 토큰 만들고 검증하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`인증 정보를 JWT 로 만들어 사용하도록 하고 싶다.`

## 만들기

FastAPI 의 공식 문서 중 [oAuth2 with Password (and hashing), Bearer with JWT tokens](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/) 를 참고하여 만들었다.
### 패키지 설치
아래 pip 구문대로 설치를 진행하면 2가지가 설치가 된다.

- cryptography
- python-jose

```python
pip install "python-jose[cryptography]"  
```

### JWT 토큰 만들기
util/security.py 에 추가한다.

#### 기본 정보 setup
- secret_key : 암호화 키
- algorithm : 암호화 알고리즘
- access_token_expire_minutes : 토큰 만료 시간

secret_key 의 경우 임의로 생성해서 사용해도 된다. 필자는 `openssl` 명령어를 사용하여 랜덤한 값을 사용했다.
```bash
openssl rand -hex 32
```

```python
SECRET_KEY = "1fe606aaf688d41ba969e0cab278698dfff6db46465542f703dd183035dfdc5d"
ALGORITHM = "HS512"
ACCESS_TOKEN_EXPIRE_MINUTES = 60 * 24
```

[Handle JWT tokens](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/#handle-jwt-tokens)


#### 토큰 생성 함수
```python
from datetime import timedelta, datetime
from passlib.context import CryptContext
from jose import JWTError, jwt
from model.jwt import TokenData
```

만료 시간을 따로 주지 않으면 기본 값으로 1시간을 부여하였다.

```python
def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=60)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt
```

`create_access_token` 을 사용해서 출력해보면 아래와 같이 . 이 2개로 3영역이 나누어진 문자열이 출력이 된다.
```
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ0ZXN0IiwibmFtZSI6ImFwZWx0b3AiLCJleHAiOjE2ODY2Mzk3NjR9.aQAbnwZw_btAV_oUB8a7A7ttW7AhmvWwm1SN1gMW0Y8m8hz8Blil4aMnr-hWjf1v5yp9jZeWOX2cojdPZlXl5w 
```

이 값을 가지고 [jwt.io](https://jwt.io) 에 접속하여 디코딩을 해보면 아래와 같이 header 와 payload 부분을 볼 수 있다.
SECRET_KEY 가 있어야만 실제로 유효한지 확인할 수 있다.

![31-1.png](/assets/img/dbdt/31-1.png)

#### 토큰 검증 함수
토큰 검증은 secret_key 를 가지고 검증을 하게 된다. 
또한 유효하다면 payload 의 값을 반환하도록 한다.

```python
def decode_access_token(token: str, secret_key: str = SECRET_KEY, algorithm: str = ALGORITHM):
    try:
        return jwt.decode(token, secret_key, algorithms=[algorithm])
    except JWTError:
        raise JWTError
```

### 테스트 케이스 작성
테스트 케이스는 간단하게 3개 작성하였다.

- 토큰 생성
- 토큰 검증 성공
- 토큰 검증 실패

```python
    def test_access_token(self):
        token = create_access_token({'sub': 'test', 'name': 'apeltop'})
        header = jwt.get_unverified_header(token)

        self.assertEqual(header, {
            'alg': 'HS512',
            'typ': 'JWT'
        })

    def test_decode_token_success(self):
        data = {'sub': 'test', 'name': 'apeltop'}
        token = create_access_token(data)
        payload = decode_access_token(token)

        self.assertEqual(payload['sub'], data['sub'])
        self.assertEqual(payload['name'], data['name'])

    def test_decode_token_fail(self):
        data = {'sub': 'test', 'name': 'apeltop'}
        token = create_access_token(data)

        with self.assertRaises(JWTError) as e:
            decode_access_token(token, secret_key='WRONG_TOKEN')
```

이제 실제 로그인 성공했을 때 토큰을 부여하기 위한 작업 밑바탕은 된 것이다.

