---
title: 이메일 인증 로직 구현
layout: doc
subtitle: DB 에 인증 정보를 넣고 인증 요청 시 적절한 처리하도록 하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것

`이메일 인증 정보를 만들고 이메일 인증을 처리할 수 있도록 하기`

실제 이메일을 보내는 것은 뒤에 구현할 예정이다.
우선 이메일 인증 정보를 만들고 이메일 인증을 처리할 수 있도록 하기 위한 로직을 만들어보려 한다.

## 기능 만들기

### Model 만들기

상황에 따라 더 많아지겠지만 최소한의 기능을 구현하고자 하기 때문에 간단하다.

- user_id: 인증을 요청한 user 의 id
- code: 인증 코드
- status: 인증 상태
- created_at: 생성 시간
- updated_at: 수정 시간

속성 중에서 재밌는 부분이 있는데 _onupdate_ 이다.
이것은 DB 에서 수정이 일어날 때마다 자동으로 현재 시간으로 업데이트를 해주는 기능이다.

```python
class EmailVerification(Base):
    __tablename__ = "email_verification"

    user_id = Column(
        String(50),
    )
    code = Column(String(36),
                  default=uuid.uuid4,
                  primary_key=True)
    status = Column(String(30))
    created_at = Column(DateTime, default=datetime.now)
    updated_at = Column(DateTime, onupdate=datetime.now)
```

### Schema 만들기

`EmailCreateVerification`, `EmailVerification` 2 가지를 만들었다.
차이는 verification_code 여부인데 이는 생성 시에 default 값으로 만들어지도록 하였기 때문에 `EmailCreateVerification` 를 통해 생성이 되면 `EmailVerification`
이 반환된다.

```python
def get_host():
    env = os.getenv('FASTAPI_ENV')
    if env == 'LOCAL':
        return 'http://localhost:8000'


class EmailCreateVerification(BaseModel):
    user_id: str
    status: str = 'WAITING_EMAIL_VERIFICATION'


class EmailVerification(EmailCreateVerification):
    verification_code: str
```

### VerificationEmailRepository 만들기

#### 기존 crud.py Interface

각 함수별로 첫번째 인자에 db 를 넘겨주어서 사용하도록 하였다.
같은 db 를 사용하는 것인데 매번 넘겨줄 필요 없이 하나의 변수를 참조해서 사용하면 좋겠다는 생각이 든다.
verification email 을 위한 crud 를 만들 때는 repository 를 만들어 보려한다.

```python
def get_blog_hub_sites(db: Session, skip: int = 0, limit: int = 100): pass
def create_blog_hub_site(db: Session, blog_hub: schemas.BlogHubSite): pass
def create_user(db: Session, user: schemas.SignUpInDB): pass
def create_fake_user(db: Session): pass
def authenticate_user(db: Session, email: str, password: str): pass
def delete_test_user(db: Session): spass
```

#### 기본 구조

생성자에 db 를 넣어 사용할 수 있도록 하였다. 이를 DI(Dependency Injection) 이라고 한다.

```python
from sqlalchemy import and_, true
from db import schemas, models


class VerificationEmailRepository:
    def __init__(self, db):
        self.db = db
```

#### 이메일 인증 코드 DB 에 생성하기

create_verification_email 를 사용하면 아래와 같이 데이터를 확인할 수 있다.
![27-1.png](/assets/img/dbdt/27-1.png)

```python
    def create_verification_email(self, verification_email: schemas.EmailCreateVerification):
        db_verification_email = models.EmailVerification(
            **verification_email.dict(),
        )
        self.db.add(db_verification_email)
        self.db.commit()
        self.db.refresh(db_verification_email)

        return db_verification_email
```

#### 이메일 인증 코드 유효성 검사하기

verification_email_by_code 는 직접 실행 시켜보아도 좋지만 손으로 직접하기엔 귀찮으니까 테스트 케이스를 통해서 마구 실행해보자.

로직은 사실 매우 간단하다.
assert 구문이 의아할 수 있으나 이메일 인증 정보 조회의 결과값이 2개 이상인 것은 문제가 있기에 assert 를 통해 문제를 발생시키도록 하였다.
assert 를 통해 최소한의 로직을 검증하는 것도 좋은 방법이다. 이 방법보다 좋은 방법은 DB Table 을 정의할 때 user_id 와 code 를 하나의 unique key 로 묶는 것이다.

의사코드

```
result = select(user_id, code)
assert result.count() < 2
is_valid = result is not None
if is_valid:
    update('DONE')
    
return is_valid
```

실제 코드

```python
    def select_verification_email(self, user_id: str, code: str):
        return self.db \
            .query(models.EmailVerification) \
            .filter(and_(models.EmailVerification.code == code,
                         models.EmailVerification.user_id == user_id))

    def verification_email_by_code(self, user_id: str, code: str):
        result = self.select_verification_email(user_id, code) \
            .filter(models.EmailVerification.status == 'WAITING_EMAIL_VERIFICATION')

        assert result.count() < 2
        is_valid = result.first() is not None
        if is_valid:
            result.update({'status': 'DONE'})
            self.db.commit()

        return is_valid

```

#### 테스트 데이터 삭제하기

```python
    def delete_test_data(self):
        self.db.query(models.EmailVerification).where(models.EmailVerification.user_id.startswith('test-') == true()).delete()
        self.db.commit()
```

### endpoint 추가하기

user_id 와 code 를 GET 파라미터로 받아서 유효성 검사를 하도록 하였다.

```python
@app.get('/api/users/{user_id}/verification_email')
async def user_verification_email(user_id, code, db: Session = Depends(get_db)):
    repository = VerificationEmailRepository(db)
    return {
        'result': repository.verification_email_by_code(user_id, code)
    }
```

## 테스트하기

### 통합 테스트 (integration test) - repository

더 여러 상황을 그려볼 수 있겠지만 이번 포스팅에서는 간단히 아래와 같이 테스트 케이스를 만들었다.

- 이메일 인증 코드 생성
    - 성공
    - 실패
- 이메일 인증 코드 유효성 검사
    - 성공
    - 실패
        - 잘못된 code
        - 잘못된 user_id
        - 잘못된 code & user_id
        - 이미 인증된 code & user_id

#### 기본 설정

`import` 와 `fixture` 설정하였다.

```python
from unittest import TestCase

import pytest
from faker import Faker
from pydantic import ValidationError
from sqlalchemy.orm import Session

from db.repository.verification_email import VerificationEmailRepository
from db.schemas import EmailCreateVerification
from db.database import engine, SessionLocal
from db import models

fake = Faker()

@pytest.fixture(autouse=True, scope='module')
def run_around_tests(db: Session):
    yield


@pytest.fixture(scope="module")
def db():
    models.Base.metadata.create_all(bind=engine)
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

#### 인증 코드 생성 성공

```python
def test_create_verification_email_success(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))

    assert verification_email is not None
```

#### 인증 코드 생성 실패 - user_id 가 없음

```python
def test_create_verification_email_fail(db: Session):
    repository = VerificationEmailRepository(db)
    with pytest.raises(ValidationError):
        repository.create_verification_email(EmailCreateVerification(
            user_id=None,
        ))
```

#### 인증 코드 verify 성공

```python 
def test_verification_email_by_code_success(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))

    db_email = repository.select_verification_email(verification_email.user_id, verification_email.code).first()

    assert repository.verification_email_by_code(verification_email.user_id, verification_email.code) is True
    assert db_email.status == 'DONE'
```

#### 인증 코드 verify 실패 - code 불일치

```python
def test_verification_email_by_code_fail_by_code(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))

    assert repository.verification_email_by_code(verification_email.user_id, 'WRONG_CODE') is False
```

#### 인증 코드 verify 실패 - user_id 불일치

```python
def test_verification_email_by_code_fail_by_user_id(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))

    assert repository.verification_email_by_code('WRONG_USER_ID', verification_email.code) is False
```

#### 인증 코드 verify 실패 - user_id & code 불일치

```python
def test_verification_email_by_code_fail_by_user_id_and_code(db: Session):
    repository = VerificationEmailRepository(db)
    repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))

    assert repository.verification_email_by_code('WRONG_USER_ID', 'WRONG_CODE') is False
```

#### 인증 코드 verify 실패 - 상태 값이 DONE

```python
def test_verification_email_by_code_fail_by_status_done(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))

    assert repository.verification_email_by_code(verification_email.user_id, verification_email.code) is True
    assert repository.verification_email_by_code(verification_email.user_id, verification_email.code) is False
```

### 통합 테스트 (integration test) - http

repository 를 테스트를 했지만 실제로는 http 로 통신을 하기 때문에 http 통신을 테스트 해보자.
repository 의 테스트와 논리는 같다.

#### httpx 설치
http 통신 테스트를 위해 httpx 를 우선 설치해야한다.
```bash
pip install httpx
```

#### 기본 설정

테스트가 모두 종료되면 테스트 데이터를 삭제하기 위해 `run_around_tests` 를 사용하였다.

```python
import pytest
from faker import Faker
from fastapi.testclient import TestClient
from sqlalchemy.orm import Session

from db import models
from db.database import engine, SessionLocal
from db.repository.verification_email import VerificationEmailRepository
from db.schemas import EmailCreateVerification
from main import app

client = TestClient(app)

fake = Faker()


@pytest.fixture(scope="session")
def db():
    models.Base.metadata.create_all(bind=engine)
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


@pytest.fixture(autouse=True, scope='session')
def run_around_tests(db: Session):
    repository = VerificationEmailRepository(db)
    yield
    repository.delete_test_data() 
```

#### 인증 코드 생성 성공

```python
def test_verification_email_success(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))
    params = f'?code={verification_email.code}'
    response = client.get(f'/api/users/{verification_email.user_id}/verification_email{params}')

    assert response.status_code == 200
    assert response.json() == {"result": True} 
```

#### 인증 코드 생성 실패 - user_id 가 없음

```python
def test_verification_email_fail_by_user_id(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))
    params = f'?code={verification_email.code}'

    response = client.get(f'/api/users/WRONG_USER_ID/verification_email{params}')

    assert response.status_code == 200
    assert response.json() == {"result": False} 
```

#### 인증 코드 생성 실패 - code 가 없음

```python
def test_verification_email_fail_by_code(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))
    params = f'?code=WRONG_CODE'

    response = client.get(f'/api/users/{verification_email.user_id}/verification_email{params}')

    assert response.status_code == 200
    assert response.json() == {"result": False}
 
```

#### 인증 코드 생성 실패 - 상태 값이 DONE

```python
def test_verification_email_fail_by_status_done(db: Session):
    repository = VerificationEmailRepository(db)
    verification_email = repository.create_verification_email(EmailCreateVerification(
        user_id=f'test-{fake.uuid4()}',
    ))
    params = f'?code={verification_email.code}'

    client.get(f'/api/users/{verification_email.user_id}/verification_email{params}')
    response = client.get(f'/api/users/{verification_email.user_id}/verification_email{params}')

    assert response.status_code == 200
    assert response.json() == {"result": False}
 ```

이로써 실제 이메일을 전송하기 위한 발판이 마련되었다.



