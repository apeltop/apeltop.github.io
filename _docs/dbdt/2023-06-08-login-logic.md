---
title: 로그인 로직 만들기
layout: doc
subtitle: DB 를 참조해 로그인 여부 판단하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것

`user 테이블을 참조해 로그인 여부를 판단하는 로직을 만들기`

## 만들기

### fake user 만들기

테스트를 위한 사용자를 많은 곳에서 만들어야하는데 매번 코드를 펼치기엔 코드가 난잡해지기 때문에 따로 **create_fake_user** 라는 함수를 만들어서 사용하도록 하였다.

```python
def create_fake_user(db: Session):
    sign_up_request = SignUpRequest(
        username=f'test-{fake.user_name()}',
        email=fake.email(),
        nickname=fake.name(),
        plain_password=fake.password(),
    )

    sign_up_user = SignUpInDB(**sign_up_request.dict(),
                              hashed_password=sign_up_request.hashed_password)

    created_user = create_user(db, sign_up_user)
    return sign_up_request, created_user
```

### 로그인 로직 만들기

우선 email 로 정보를 받아 user 를 찾은 후 user 의 password verify 를 통해 로그인 여부를 판단하도록 하였다.
[패스워드 암호화하기](/docs/dbdt/2023-06-05-passlib-bcrypt/) 에서 살펴보았듯 평문인 비밀번호를 해쉬 값으로 변환하면 매번 결과 값이 달라진다.
그러기에 단순 비교 연산자로는 사용할 수 없다. 따라서 **verify_password** 함수를 통해 비교하도록 하였다.

```python
def authenticate_user(db: Session, email: str, password: str):
    user = db.query(models.User).filter(models.User.email == email).first()
    if not user:
        return None
    if not verify_password(password, user.hashed_password):
        return None

    return user
```

여기서 일치하는 회원 정보가 있을 때 DB 에서 가져온 User 를 반환하고 있다.
여기에는 비밀번호가 담겨있다. 이것을 반환하며 사용하는 것은 바람직하지 못하다.
어떤 실수를 해서 중요 정보가 반환될지 모르기 때문이다. 그러기에 필요한 정보만 반환을 하도록 개선해보려한다.

![26-1.png](/assets/img/dbdt/26-1.png)

#### 민감 정보를 제외한 user 정보 반환

schemas 를 보면 User 와 UserInDB 라는 두 개의 모델이 있다.
UserInDB 는 현재는 비밀번호만 담겨있지만 추후에는 민감한 정보가 더 추가될 수 있다.
이전 코드에서는 UserInDB 를 반환했기 때문에 민감한 정보가 노출되었다. 그러므로 User 를 반환하도록 개선해보자.

```python
def authenticate_user(db: Session, email: str, password: str):
    user = db.query(models.User).filter(models.User.email == email).first()
    if not user:
        return None
    if not verify_password(password, user.hashed_password):
        return None

    return schemas.User(**user.to_dict())
```

이 코드는 *에러*가 난다. 이유는 to_dict 이 없기 때문이다.
이를 해결하기 위해 User 모델에 to_dict 함수를 추가해보자.

```python
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
    hashed_password = Column(String(100))

    def to_dict(self):
        return {field.name: getattr(self, field.name) for field in self.__table__.c}
```

그 후 다시 테스트를 해보면 정상적으로 동작하며 민감 정보도 없는 것을 확인할 수 있다.

![26-2.png](/assets/img/dbdt/26-2.png)

### 테스트하기

테스트는 4가지만 진행하려한다. 혹자는 4번과 같이 왜 둘 다 틀렸을 때까지 테스트를 해야하는지 의문이 생길 수 있다.
사실 대부분의 경우 email 또는 pw 가 잘못되었을 때 동작을 잘 하기에 email 과 pw 모두 잘못되었을 때도 None 을 반환할 것이다.
그렇지만 테스트 케이스는 가능한 많은 경우를 대변하는 것이 좋다. 개발자 대부분 말도 안되는 실수를 경험하였거나 본 경험이 있을 것이다.
그렇기에 중복이고 의미 없는 테스트 케이스는 오히려 독이 되지만 여러 상황에 대하여 테스트 케이스가 많이 있는 것은 좋다.

1. 로그인 성공
2. 로그인 실패 - email X, pw O
3. 로그인 실패 - email O, pw X
4. 로그인 실패 - email X, pw X

#### 로그인 성공
테스트 조건 중에서 **hashed_password** 가 있는지도 확인하도록 하였다.

```python
def test_login_success(db: Session):
    sign_up_user, created_user = create_fake_user(db)
    user = authenticate_user(db, sign_up_user.email, sign_up_user.plain_password)

    assert user is not None
    assert hasattr(user, 'hashed_password') is False
```

#### 로그인 실패 - email X, pw O

```python
def test_login_fail_by_email(db: Session):
    sign_up_user, created_user = create_fake_user(db)
    user = authenticate_user(db, 'WRONG_EMAIL', sign_up_user.plain_password)

    assert user is None
```

#### 로그인 실패 - email O, pw X

```python
def test_login_fail_by_password(db: Session):
    sign_up_user, created_user = create_fake_user(db)
    user = authenticate_user(db, sign_up_user.email, 'WRONG_PASSWORD')

    assert user is None
```

#### 로그인 실패 - email X, pw X

```python
def test_login_fail_by_all(db: Session):
    create_fake_user(db)
    user = authenticate_user(db, 'WRONG_EMAIL', 'WRONG_PASSWORD')

    assert user is None
```
