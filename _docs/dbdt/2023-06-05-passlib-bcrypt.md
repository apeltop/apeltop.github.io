---
title: 패스워드 암호화하기
layout: doc
subtitle: passlib 를 사용하여 패스워드를 암호화하기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`평문인 패스워드` 를 해쉬 함수(bcrypt) 를 사용하여 암호화하고자 한다.

## 만들기
### passlib 설치
```bash
pip install passlib[bcrypt]
```

위 구문을 실행하면 2개의 패키지가 설치가 된다.
- bcrypt
- passlib

### 암호화 함수 만들기
utils/security.py 파일을 만들고 아래와 같이 작성한다.

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
```

`CryptContext(schemes=["bcrypt"], deprecated="auto")` 

schemes=["bcrypt"] : bcrypt 암호화를 사용한다.

deprecated="auto" : 지정한 schemes 를 제외한 다른 방식은 사용하지 않는다.

[deprecated 공식 문서](https://passlib.readthedocs.io/en/stable/lib/passlib.context.html)

```python
def get_password_hash(password):
    return pwd_context.hash(password)
    
    
def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)
```


`get_password_hash` 함수는 `password` 를 암호화하여 반환한다.

`verify_password` 함수는 `평문 패스워드` 와 `hashed_password` 를 비교하여 같으면 True 를 반환한다.

### 확인하기
test_security.py 를 만들고 테스트 케이스를 추가한다. 

2가지를 테스트 하였다.
1. 암호화 알고리즘이 bcrypt 인가
2. 평문 패스워드를 암호화했을 때 verify 한가

get_password_hash 함수의 경우 매번 값이 달라지는데 이를 어떻게 테스트하는게 좋을지 고민이 된다. 현재는 해당 메서드 테스트 케이스는 없다.

```python
from unittest import TestCase

from util.security import get_password_hash, pwd_context, verify_password


class SecurityTest(TestCase):
    def test_pwd_context(self):
        self.assertEqual(pwd_context.default_scheme(), 'bcrypt')

    def test_get_password_hash(self):
        self.assertTrue(verify_password('1234', get_password_hash('1234')))

```

get_password_hash 를 호출하면 값은 인자여도 매번 다른 hash 가 다른데 기회가 된다면 이를 설명하는 글을 적어보도록 해보겠다.
우선 매번 hash 값이 달라지는 이유는 자동으로 salt 를 붙이기 때문인데 이를 통해 달라져도 verify 하는데 문제가 없다. 
