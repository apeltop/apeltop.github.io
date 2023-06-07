---
title: alembic 사용해서 db 마이그레이션 하기
layout: doc
subtitle: DB Table 의 컬럼 변경 등에 능동적으로 대응하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것

`model 에 새로운 column 이 추가되었을 때 DB 에 반영하기`

현재는 한 번 model 을 통해 Table 이 만들어지고나면 model 에 column 을 추가해도 반영이 되지 않는다.
직접 column 을 추가하던지 Table 을 drop 하면 적용이 된다. 즉 엄청 불편하다. 이를 해결하기 위해 alembic 을 사용한다.

## 만들기

### alembic 설치

```bash
pip install alembic
```

### alembic init

```bash
alembic init alembic
```

alembic 폴더가 생성되고 다음과 같은 파일들이 생성된다.

- alembic.ini
- alembic
    - env.py
    - script.py.mako
    - versions
    - README

![25-1.png](/assets/img/dbdt/25-1.png)

![25-2.png](/assets/img/dbdt/25-2.png)

### alembic 설정

alembic.ini 파일을 열어서 **sqlalchemy.url** 를 수정한다.

```ini
sqlalchemy.url = ~
```

![25-3.png](/assets/img/dbdt/25-3.png)

그 후 alembic/env.py 파일을 열어서 **target_metadata** 를 수정한다.
기존에는 target_metadata 가 None 으로 설정되어있는데 이를 수정해준다.

```python
target_metadata = None
```

```python
target_metadata = models.Base.metadata 
```

### alembic revision

설정 후 revision 을 기록한다.

--autogenerate 를 사용하지 않을 수 있는데 이 경우에는 revision 을 기록할 때마다 직접 변경된 내용을 기록해줘야 한다.
필자는 autogenerate 를 사용하였다.

```bash
alembic revision --autogenerate
```

그럼 alembic/versions 폴더에 파일이 생성된다.

### Column 추가하기

테스트 할 겸 model 에 있는 User 에 hased_password1 이라는 column 을 추가해준다.

```python
class User(Base):
    __tablename__ = "user"
    
    ...
    
    hased_password1 = Column(String(100))
```

그 후 다시 revision 을 기록한다.

```bash
alembic revision --autogenerate
```

그럼 다음과 같이 파일이 생성된다.

![25-4.png](/assets/img/dbdt/25-4.png)

파일을 보면 upgrade 와 downgrade 를 볼 수 있으며 새롭게 추가된 hased_password1 과 관련된 작업이 보인다.

이를 적용하기 위해서 아래와 같이 upgrade 를 해주면 된다. head 를 보니 git 같은 느낌을 받는다.

```bash
alembic upgrade head
```

실행 후 DB 에 접속해서 확인해보면 hased_password1 column 이 추가된 것을 확인할 수 있다.

![25-5.png](/assets/img/dbdt/25-5.png)

반대로 다운그레이드를 하고 싶다면 다음과 같이 실행하면 된다.
```bash
alembic downgrade -1
```



