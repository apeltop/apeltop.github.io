---
title: DB 에 User Insert 하기
layout: doc
subtitle: 회원 정보 저장하기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`회원 정보를 받아 DB 에 넣기`

**유효성 검사는 여기서 진행하지 않음.**

## 만들기

### model 추가
db/models.py 에 User 모델을 추가한다.

테이블 이름은 user 로 하였으며 저장하는 정보는 다음과 같다.

id 는 따로 설정하지 않고 기본 값으로 uuid 를 사용하도록 하였다.

- id: uuid4 로 생성
- username: 사용자 이름
- email: 이메일
- nickname: 닉네임
- sign_up_status: 회원가입 상태
- hashed_password: 비밀번호

```python
import uuid

...

class User(Base):
    __tablename__ = "user"

    id = Column(
        String(36),
        default=uuid.uuid4,
        primary_key=True,
        index=True,
        nullable=False,
    )
    username = Column(String(20))
    email = Column(String(30))
    nickname = Column(String(20))
    sign_up_status = Column(String(30))
    hased_password = Column(String(100))
```

### schema 추가
db/schemas.py 에 스키마를 추가한다.

기본적으로 사용자 정보를 사용할 때 쓸 User 를 만들고 회원 가입 시에만 필요한 정보를 추가로 받는 SignUpInDB 를 만든다.

```python
class User(BaseModel):
    username: str
    email: EmailStr
    nickname: str
    sign_up_status: str = 'WAITING_EMAIL_VERIFICATION'
    
class SignUpInDB(User):
    hased_password: str
```

### crud 중 c(create) 추가
db/curd.py 에 create_user 함수를 추가한다.

```python
def create_user(db: Session, user: schemas.SignUpInDB):
    db_user = models.User(
        **user.dict(),
    )
    db.add(db_user)
    db.commit()
    db.refresh(db_user)

    return db_user
```


### 테스트 케이스 작성하기
test_models.py 를 만들고 테스트 케이스를 작성한다.

디비가 필요한 테스트 케이스가 계속해서 생겨날 것이기 때문에 db 를 테스트 케이스마다 작성할 수는 없기에 fixture 를 이용하였다.
fixture 를 통해 테스트 케이스에 db 를 넘겨주도록 하였다. fixture 의 범위를 module, class, session 등으로 설정 가능하다.
그 중에서 여기서는 module 로 설정하였다.

```python
import pytest
from faker import Faker
from sqlalchemy.orm import Session

from db import models
from db.crud import create_user
from db.database import engine, SessionLocal
from db.schemas import SignUpRequest, SignUpInDB


@pytest.fixture(scope="module")
def db():
    models.Base.metadata.create_all(bind=engine)
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


fake = Faker()


def test_sign_up(db: Session):
    sign_up_request = SignUpRequest(
        username=fake.user_name(),
        email=fake.email(),
        nickname=fake.name(),
        plain_password=fake.password(),
    )

    sign_up_user = SignUpInDB(**sign_up_request.dict(),
                              hased_password=sign_up_request.hashed_password)

    created_user = create_user(db, sign_up_user)

    # e.g id: 065ae0ac-50d3-4648-81d7-35acd566f16b
    assert len(created_user.id.split('-')) == 5
```

DB 를 확인해보면 성공적으로 값이 들어왔다.
![24-1.png](/assets/img/dbdt/24-1.png)

여기서 문제는 테스트 데이터인데 테스트 데이터는 테스트 케이스가 끝나면 지워져야 한다.
크게 방법은 2가지를 생각해볼 수 있다.

1. 테스트 케이스 별로 각자가 만든 데이터는 각자가 지우기
2. 전체 또는 국지적으로 테스트 실행이 종료되면 한 번에 삭제하기

1번도 가능하지만 그럼 테스트 케이스마다 중복된 코드가 많아진다. 또한 return 값이 필요없지만 return 을 만들어 사용해야하기도 한다.

그러기에 2번이 대체로 낫다. 그렇다면 한 번에 테스트 데이터를 지우려면 어떻게 지워야 하는가? 테스트 데이터라는 것을 알려주면 된다.

예를 들어 어떤 값에 test- 를 붙이는 것이다. 

#### after & before
테스트가 실행 전(before) 과 후(after) 에 특정한 행동을 하기 위해서는 아래와 같은 fixture 가 필요하다.
이 코드는 test_models.py 에 추가했다. 

```python
@pytest.fixture(autouse=True, scope='module')
def run_around_tests():
    print('before')
    yield
    print('after')
```

위 코드를 추가 후 테스트 케이스를 돌리고 로그를 보면 scope 가 module 이기 때문에 현재 1개만 실행된다.
PASSSED 전후에 before 와 after 가 로그가 출력된다.

```python
before
PASSED                                   [  9%]
after
```

#### 테스트 사용자 삭제 코드 추가
`test-` 로 시작하는 데이터 삭제하도록 하는 쿼리이다.

crud.py 에 추가하였다.

```python
from sqlalchemy import true

...

def delete_test_user(db: Session):
    db.query(models.User).where(User.username.startswith('test-') == true()).delete()
    db.commit()
```

다시 test_models.py 로 돌아와 코드를 추가한다.
```python

@pytest.fixture(autouse=True, scope='module')
def run_around_tests(db: Session):
    yield
    delete_test_user(db)
```

그럼 테스트 실행 후 테스트 데이터가 삭제된 것을 확인해볼 수 있다.

현재 유효성 검사, 이메일 인증 단계 등이 없지만 우선 하나하나 작은 기능을 덧입혀 나가보려한다.
