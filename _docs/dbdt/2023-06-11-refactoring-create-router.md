---
title: 리팩터링 - router 분리하기
layout: doc
subtitle: main 에 있는 각각의 router 를 별도의 모듈로 분리하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`main 에 모든 endpoint 가 있는데 이를 각각의 router 로 분리하여 관리하고 싶다.`

## 리펙터링 전과 후
![30-1.png](/assets/img/dbdt/30-1.png)

## 리펙터링 하기
우선 이전 글에서 [리팩토링 - api prefix 추가](/docs/dbdt/2023-06-11-refactoring-add-api-prefix/) 에서 /api/v1/ 으로 관리하도록 하였다.
여기서 또 문제는 매번 /api/v1/ 을 붙여한다는 점이다. 또 매번 /users, /blogs 등을 붙여야한다. 물론 이런 오타 같은 경우 테스트 케이스를 통해 관리할 수도 있겠지만 그럴 필요 없이 하나의 라우터로 묶으면 편하게 관리할 수 있다.

### router 만들기
기존의 main.py 에 있는 코드를 단순히 router 로 분리한다.

코드가 달라진 부분이라면 **@app**.get -> **@router**.get 정도이다.

#### /router/users.py
APIRouter 에서 prefix 로 /api/v1/users 를 설정하였기 때문에 해당 모듈에서는 '/' 으로만 설정하여도 /api/v1/users 시 호출된다.

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session

from db.database import get_db
from db.repository.verification_email import VerificationEmailRepository
from db.schemas import SignUpRequest
from util.faker import fake
from util.security import verify_password, get_password_hash

router = APIRouter(
    prefix='/api/v1/users',
    tags=['users'],
)


@router.get('/')
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


@router.get('/login')
async def user_login():
    return {
        "pw": get_password_hash('1234'),
        "verify_password": verify_password('1234', "$2b$12$Py0FA0WreFXgNidDKqtrF.V.oC82gG7tWjDhlSsr0XlyC6J9p05Me")
    }


@router.post('/')
async def user_create(user: SignUpRequest):
    # TODO : check if user already exists
    # TODO : send email verification
    # TODO : create user

    return user


@router.get('/{user_id}/verification_email')
async def user_verification_email(user_id, code, db: Session = Depends(get_db)):
    repository = VerificationEmailRepository(db)
    return {
        'result': repository.verification_email_by_code(user_id, code)
    }
```

### /routers/blogs.py
```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session

from db import crud
from db.database import get_db

router = APIRouter(
    prefix='/api/v1/blogs',
    tags=['challenge'],
)


@router.get('/')
async def blog_data_list(db: Session = Depends(get_db)):
    sites = crud.get_blog_hub_sites(db)
    return {
        'blogs': sites,
    }
```

### /router/challenges.py
```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session

from db.database import get_db
from db.repository.verification_email import VerificationEmailRepository
from db.schemas import SignUpRequest
from util.faker import fake
from util.security import verify_password, get_password_hash

router = APIRouter(
    prefix='/api/v1/challenges',
    tags=['challenge'],
)


@router.get('/applicants')
async def challenge_user_list():
    return {
        'users': [{
            'user_id': fake.uuid4(),
            'name': fake.name(),
            'email': fake.email(),
            'date_of_application': fake.date_time_this_month(),
            'job': fake.job(),
        } for _ in range(100)]
    }


@router.get('/posts')
async def challenge_post_list():
    return {
        'posts': [{
            'post_id': fake.uuid4(),
            'title': fake.sentence(),
            'content': fake.text(),
            'date_of_post': fake.date_time_this_month(),
            'reference_url': fake.url(),
            'post_url': fake.url(),
        } for _ in range(100)]
    }
```

### main 에서 router 추가하기
```python
from routers import users, challenges, blogs

...

app.include_router(users.router)
app.include_router(challenges.router)
app.include_router(blogs.router)
```

간단하지만 확실히 깔끔해진 코드를 볼 수 있다. 각각 router 파일을 만들었기 때문에 관리의 용이성도 증가하였다.
혹자는 /api/v1 도 중복된 문자열인데? 라고 생각할 수 있다. 옳은 지적이다. 하지만 지금은 v2 가 없기 때문에 이렇게 관리할 것이다.
추후 v2 가 생긴다면 v1, v2 라우터를 만들어서 관리하도록 할 것이다.  

