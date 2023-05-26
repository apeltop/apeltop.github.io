---
title: SqlAlchemy + GCP MySQL 연결하기
layout: doc
subtitle: SqlAlchemy 을 사용해서 GCP MySQL 연결하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`SqlAlchemy` 을 사용해서 `GCP MySQL` 에 연결하기

## SqlAlchemy 을 사용해서 GCP MySQL 연결하기
### MySQL Schema 만들기
dbdt 라는 이름의 schema 를 만든다.
Collation 은 `utf8mb4_general_ci` 로 설정한다. 이 collation 은 이모티콘이 가능하다는 특징이 있다.

![14-1.png](/assets/img/dbdt/14-1.png)

### MySQL DB 계정 만들기
dbdt 스키마에 server 라는 User 를 만든다.
필요한 권한에 따라 부여한다. 때로 특정 예제를 보면 모든 권한을 주도록하는데 이게 편할지 모르겠지만 또 연습이니까 괜찮다고 생각할지 모르겠지만 개인적으로는 연습부터 권한을 제한하는 것이 좋다고 생각한다.

![14-2.png](/assets/img/dbdt/14-2.png)

### Cloud SQL 에 User 추가하기
GCP Cloud SQL 에서는 MySQL 에 User 를 추가해도 Cloud SQL 에서 User 추가하지 않으면 접속이 되지 않는다.
password 는 따로 기록해둬야한다.

![14-3.png](/assets/img/dbdt/14-3.png)

### SqlAlchemy + GCP Cloud SQL(MySQL)
공식 문서 tutorial 을 참고하였다.

[SQL (Relational) Databases](https://fastapi.tiangolo.com/tutorial/sql-databases/#sql-relational-databases)

#### 설치
```bash
pip install sqlalchemy
```

필자는 mysql 을 사용하기에 추가적으로 `mysqlclient` 설치하였다.
```bash
pip install mysqlclient
```

#### 연결
##### db/database.py
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "mysql+mysqldb://DB_USER:DB_PW@DB_IP:DB_PORT/DB_NAME"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()
```

#### 모델, 스키마, crud
##### db/models.py
```python
from sqlalchemy import Column, Integer, String

from .database import Base


class BlogHubSite(Base):
    __tablename__ = "blog_hub_sites"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    name = Column(String(20), unique=True, index=True)
    site_url = Column(String(200))
    image_url = Column(String(200))
```

#### db/schemas.py
```python
from pydantic import BaseModel


class BlogHubSite(BaseModel):
    id: int
    name: str
    site_url: str
    image_url: str

    class Config:
        orm_mode = True
```

#### db/crud.py
```python
from sqlalchemy.orm import Session
from . import models, schemas


def get_blog_hub_sites(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.BlogHubSite).offset(skip).limit(limit).all()


def create_blog_hub_site(db: Session, blog_hub: schemas.BlogHubSite):
    db_blog_hub = models.BlogHubSite(
        name=blog_hub.name,
        site_url=blog_hub.site_url,
        image_url=blog_hub.image_url
    )
    db.add(db_blog_hub)
    db.commit()
    db.refresh(db_blog_hub)
    return db_blog_hub
```

#### 데이터 추가하기
기존에는 변수에 데이터를 저장하여 반환하였는데 이를 DB 로 옮기려한다.

##### import 및 db 추가
```python
...
from fastapi import FastAPI, Depends
from fastapi.staticfiles import StaticFiles
from sqlalchemy.orm import Session

from db import models, crud
from db.database import engine, SessionLocal
from util.date import get_current_kst, get_today_deadline_kst

models.Base.metadata.create_all(bind=engine)

...

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
        
```

##### 데이터 추가하기
아래 데이터는 한 번만 DB 에 추가되면 되니 서버 로딩될 때 한 번 실행되고 그 후에는 주석처리하였다.
```python
create_blog_hub(db, models.BlogHub(name='데이터리안(Datarian)', site_url='https://blog.hackle.io', image_url='/static/data-blog/datarian.webp'))
create_blog_hub(db, models.BlogHub(name='핵클(hackle)', site_url='https://datarian.io', image_url='/static/data-blog/hackle.jpg'))
```

##### 기존 코드
```python
@app.get('/api/blogs')
async def blog_data_list():
    return {
        'blogs': [
            {
                'name': '데이터리안(Datarian)',
                'site_url': 'https://blog.hackle.io',
                'image_url': '/static/data-blog/datarian.webp',
            },
            {
                'name': '핵클(hackle)',
                'site_url': 'https://datarian.io',
                'image_url': '/static/data-blog/hackle.jpg',
            },
        ]
    }
```

##### DB 데이터 반환하는 코드
```python
@app.get('/api/blogs')
async def blog_data_list(db: Session = Depends(get_db)):
    sites = crud.get_blog_hub_sites(db)
    return {
        'blogs': sites,
    }
```
