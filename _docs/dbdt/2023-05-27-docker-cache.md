---
title: Docker Cache 이용하기
layout: doc
subtitle: COPY 위치를 조절해 Docker Cache 를 이용하기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`코드만 변경 시 의존성 패키지는 캐쉬로 처리하여 빠르게 빌드하기`

fastapi 공식 문서 중 [FastAPI in Containers - Docker](https://fastapi.tiangolo.com/deployment/docker) 중 [Docker Cache](https://fastapi.tiangolo.com/deployment/docker/#docker-cache) 를 읽다보니 수정할 부분이 바로 보여 적용해보았다.

## Dockerfile 수정하기

### 기존 Dockerfile
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

### 개선점 찾기

환경 설정 등을 제외하고 작업 목록을 나열하면 아래와 같다. 아래 작업은 대략 20초 가량 소요된다.
- COPY . .
- RUN apk add ...
- RUN pip install ...

[Docker Cache](https://fastapi.tiangolo.com/deployment/docker/#docker-cache) 에 따르면 Docker 는 각 단계 별로 증분한다고 설명하고 있다.
즉 layer 위에 layer 가 쌓이는 형식으로 이미지가 만들어진다는 것이다.
마지막으로 만들어진 컨테이너 이미지에서 변경되지 않았다면 새로 만드는 것이 아니라 재사용한다.
파일이 변경된 것을 확인한 다음 step 부터는 캐쉬가 적용되지 않는다. 해당 작업이 뒤에 어떻게 영향을 미치지 모르기 때문이다.

즉 파일 변경이 잦지 않은 부분은 위에서 먼저 실행되도록 하는 것이 캐쉬 사용할 수 있다는 것이다.

기존 Dockfile 을 보면 COPY . . 이 상단에 위치하여 매번 이미지를 만들 때마다 패키지 설치를 진행한다.

출력 로그를 보면 매번 설치하는 것을 확인할 수 있다.
```bash
#6 [2/5] WORKDIR /app
#6 CACHED

#7 [3/5] COPY . ./
#7 DONE 0.6s

#8 [4/5] RUN apk add mysql-client gcc musl-dev mariadb-connector-c-dev
#8 0.156 fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/main/aarch64/APKINDEX.tar.gz
#8 0.685 fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/community/aarch64/APKINDEX.tar.gz
#8 1.813 (1/21) Installing libgcc (12.2.1_git20220924-r10)
#8 6.447 (20/21) Installing mariadb-client (10.11.3-r0)

...

#8 6.968 (21/21) Installing mysql-client (10.11.3-r0)
#8 7.002 Executing busybox-1.36.0-r9.trigger
#8 7.018 OK: 230 MiB in 59 packages
#8 DONE 7.1s

#9 [5/5] RUN pip install --no-cache-dir -r requirements.txt
#9 1.591 Collecting faker==18.9.0
#9 1.658   Downloading Faker-18.9.0-py3-none-any.whl (1.7 MB)
#9 1.878      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.7/1.7 MB 7.8 MB/s eta 0:00:00
#9 1.930 Collecting fastapi==0.95.2
#9 1.940   Downloading fastapi-0.95.2-py3-none-any.whl (56 kB)
#9 1.943      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 57.0/57.0 kB 20.4 MB/s eta 0:00:00
#9 1.961 Collecting httptools==0.5.0
#9 1.972   Downloading httptools-0.5.0-cp310-cp310-musllinux_1_1_aarch64.whl (426 kB)
#9 1.994      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 426.4/426.4 kB 20.2 MB/s eta 0:00:00
#9 12.54

...

#9 12.54 [notice] A new release of pip is available: 23.0.1 -> 23.1.2
#9 12.54 [notice] To update, run: pip install --upgrade pip
#9 DONE 12.7s
```

### 개선 Dockerfile
```bash
FROM python:3.10-alpine

# Allow statements and log
ENV PYTHONUNBUFFERED True

ENV APP_HOME /app
WORKDIR $APP_HOME

RUN apk add mysql-client gcc musl-dev mariadb-connector-c-dev

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

COPY . ./

ENV PORT 8080
CMD exec uvicorn main:app --host 0.0.0.0 --port=${PORT}
```

`COPY . .` 를 하단으로 내리고 `COPY requirements.txt requirements.txt` 를 추가하였다. 이 파일은 pip 패키지 설치 시 필요하기 때문이다.

그 결과 소스 코드만 수정 후 이미지 만드는 로그를 보면 캐쉬가 적용된 것을 확인할 수 있다.
```bash
#6 [2/6] WORKDIR /app
#6 CACHED

#7 [3/6] RUN apk add mysql-client gcc musl-dev mariadb-connector-c-dev
#7 CACHED

#8 [4/6] COPY requirements.txt requirements.txt
#8 CACHED

#9 [5/6] RUN pip install --no-cache-dir -r requirements.txt
#9 CACHED

#10 [6/6] COPY . ./
#10 DONE 0.3s
```

현재 사용하는 패키지 수가 적기 때문에 패키지를 설치해도 20초 가량 밖에 안 걸렸지만 규모가 있는 프로젝트에선느 몇 분을 아낄 수 있는 상황이다.
이미지 빌드는 빈번하게 이루어지는데 몇 분이 쌓인다고 생각하면 몇 시간이 절약할 수 있는데 이는 엄청난 효율을 제공한다. 

