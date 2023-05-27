---
title: MySQL 연동 후 Dockerfile 수정하기
layout: doc
subtitle: MySQL 연동 후 Docker 로 실행시켜 확인해보면 에러가 발생하는데 필요 패키지 설치하기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`Docker 에서 실행되도록 하기`


## Trouble Shooting
### 에러 원인 파악
이전글인 [SqlAlchemy + GCP MySQL 연결하기](/docs/dbdt/2023-05-26-gcp-mysql-connnect/) 을 완성 후 Docker 로 실행시켜보면 아래와 같은 에러가 발생한다.

```bash
#8 1.686 Collecting mysqlclient==2.1.1
#8 1.693   Downloading mysqlclient-2.1.1.tar.gz (88 kB)
#8 1.698      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 88.1/88.1 kB 70.0 MB/s eta 0:00:00
#8 1.712   Preparing metadata (setup.py): started
#8 2.076   Preparing metadata (setup.py): finished with status 'error'
#8 2.079   error: subprocess-exited-with-error
#8 2.079   
#8 2.079   × python setup.py egg_info did not run successfully.
#8 2.079   │ exit code: 1
#8 2.079   ╰─> [16 lines of output]
#8 2.079       mysql_config --version
#8 2.079       /bin/sh: mysql_config: not found
#8 2.079       mariadb_config --version
#8 2.079       /bin/sh: mariadb_config: not found
#8 2.079       mysql_config --libs
#8 2.079       /bin/sh: mysql_config: not found
#8 2.079       Traceback (most recent call last):
#8 2.079         File "<string>", line 2, in <module>
#8 2.079         File "<pip-setuptools-caller>", line 34, in <module>
#8 2.079         File "/tmp/pip-install-5p_nz853/mysqlclient_15728ef5fe914e1781afa09dd721b927/setup.py", line 15, in <module>
#8 2.079           metadata, options = get_config()
#8 2.079         File "/tmp/pip-install-5p_nz853/mysqlclient_15728ef5fe914e1781afa09dd721b927/setup_posix.py", line 70, in get_config
#8 2.079           libs = mysql_config("libs")
#8 2.079         File "/tmp/pip-install-5p_nz853/mysqlclient_15728ef5fe914e1781afa09dd721b927/setup_posix.py", line 31, in mysql_config
#8 2.079           raise OSError("{} not found".format(_mysql_config_path))
#8 2.079       OSError: mysql_config not found
#8 2.079       [end of output]
#8 2.079   
#8 2.079   note: This error originates from a subprocess, and is likely not a problem with pip.
#8 2.079 error: metadata-generation-failed
``` 

에러 출력을 읽어보면 `mysql_config not found` 라는 메시지가 보인다. 

### 해결 방법
에러 출력을 토대로 stackoverflow 를 찾아본 결과 추가적으로 필요한 패키지를 설치해야한다는 것을 알게되었다.

[error when docker build python django app in alpine linux](https://stackoverflow.com/questions/52158937/error-when-docker-build-python-django-app-in-alpine-linux)

#### Dockerfile 수정
```bash
RUN apk add mysql-client gcc musl-dev mariadb-connector-c-dev
```

#### Dockerfile 전체
```bash
FROM python:3.10-alpine

# Allow statements and log
ENV PYTHONUNBUFFERED True

ENV APP_HOME /app
WORKDIR $APP_HOME
COPY . ./

RUN apk add mysql-client gcc musl-dev mariadb-connector-c-dev
RUN pip install --no-cache-dir -r requirements.txt

ENV PORT 8080
CMD exec uvicorn main:app --host 0.0.0.0 --port=${PORT}
```
