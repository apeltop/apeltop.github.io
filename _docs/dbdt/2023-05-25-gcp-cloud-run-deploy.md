---
title: FastAPI 배포하기
layout: doc
subtitle: GCP 에 FastAPI 배포하기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`지금까지 만든 FastAPI 를 GCP 에 배포하기`

## 배포하기
### Local Test
배포하기 전에 로컬에서 테스트를 해보는 것이 더 좋다. 그런데 FastAPI 를 바로 실행시켜서 확인하기보다 Docker 로 배포하여 테스트하는 것이 바람직한 방법이다.
#### Dockerfile 생성

PYTHONUNBUFFERED 는 Python 의 버퍼링을 끄는 것이다.
stdout 및 stderr 에 저장되지 않고 바로 출력이 되는데 이로써 충돌이 발생하였을 때 일부 로그 손실을 방지할 수 있다.
그렇지만 버퍼링을 끄게 되면 성능에 미미한하지만 영향을 미친다고 한다.

[What is the use of PYTHONUNBUFFERED in docker file?](https://stackoverflow.com/questions/59812009/what-is-the-use-of-pythonunbuffered-in-docker-file)

작업 단계
- python:3.9 이미지를 사용
- 환경 변수 PYTHONUNBUFFERED 를 True 로 설정
- 현재 디렉토리의 모든 파일을 /app 에 복사
- requirements.txt 를 사용하여 의존성 설치
- uvicorn 을 사용하여 FastAPI 실행

```Dockerfile
FROM python:3.9-slim

# Allow statements and log
ENV PYTHONUNBUFFERED True

ENV APP_HOME /app
WORKDIR $APP_HOME
COPY . ./

RUN pip install --no-cache-dir -r requirements.txt

ENV PORT 8080
CMD exec uvicorn main:app --host 0.0.0.0 --port=${PORT}
```

#### 로컬에서 Dockerfile 실행하기
Intellij 에서 [Google Cloud Code](https://plugins.jetbrains.com/plugin/8079-google-cloud-code) 을 설치한다.

그러면 New Configuration 에서 Cloud Run 을 선택할 수 있다.
그 중에서 _Run Locally_ 를 선택한다.
![11-1.png](/assets/img/dbdt/11-1.png)

별 달리 설정할 것 없다. 하단의 OK 를 눌러 준비한다.
![11-2.png](/assets/img/dbdt/11-2.png)

그 후 실행을 시키면 아래와 같이 출력이 되면서 실행이 된다. 
로컬에서 Docker 가 실행 중이여야 돌아간다.
```bash
Validating Docker settings...
Preparing Google Cloud SDK (this may take several minutes for first time setup)...

Initializing Minikube. Minikube may take several minutes to initialize for the first time...

Checking status of cloud-run-dev-internal Minikube cluster...
Status of Minikube api-server is RUNNING
Enabling GCP auth addon...

GCP auth addon successfully enabled

Starting the Cloud Run development session...
/Users/apeltop/Library/Application Support/cloud-code/...
Generating tags...
 - server -> server:latest
Checking cache...
 - server: Not found. Building
Starting build...
Found [cloud-run-dev-internal] context, using local docker daemon.
Building [server]...
Target platforms: [linux/arm64]
#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 295B 0.0s done
```

```bash
Port forwarding service/server in namespace default, remote port 8080 -> http://127.0.0.1:8080
Listing files to watch...
 - server
Press Ctrl+C to exit
Not watching for changes...
[server-container] INFO:     Started server process [1]
[server-container] INFO:     Waiting for application startup.
[server-container] INFO:     Application startup complete.
[server-container] INFO:     Uvicorn running on http://127.0.0.1:8080 (Press CTRL+C to quit)
[server-container] INFO:     127.0.0.1:42196 - "GET / HTTP/1.1" 200 OK
[server-container] INFO:     127.0.0.1:42196 - "GET /favicon.ico HTTP/1.1" 404 Not Found
[server-container] INFO:     127.0.0.1:42196 - "GET / HTTP/1.1" 200 OK
[server-container] INFO:     127.0.0.1:42196 - "GET /docs HTTP/1.1" 200 OK
[server-container] INFO:     127.0.0.1:42196 - "GET /openapi.json HTTP/1.1" 200 OK
[server-container] INFO:     127.0.0.1:42196 - "GET /api/challenge/applicant/list HTTP/1.1" 200 OK
```

### GCP 에 배포하기
이 포스팅에서는 Intellij 에 의존해서 배포를 진행하지만 Command Line 으로도 배포가 가능하다.
```gcloud run deploy ...```
[Deploying a new revision of an existing service](https://cloud.google.com/run/docs/deploying#revision)

Local Test 를 진행할 때와 달리 Deploy 를 선택해준다.

![11-1.png](/assets/img/dbdt/11-1.png)

project 선택하고 Image repository 를 설정해준다.
또한 서버 스펙 또한 설정할 수 있는데 필요에 따라 설정을 해주면 된다.
또한 Service Account 는 GCP Cloud Run 초기 설정하기에서 [Service Account 생성](/docs/dbdt/2023-05-25-gcp-cloud-run-init#service-account-생성) 만들었던 cloud-run-server@~ 계정을 입력해준다.

![11-3.png](/assets/img/dbdt/11-3.png)

그후 배포를 진행하면 아래와 같이 배포가 진행된다.
```bash
Preparing Google Cloud SDK (this may take several minutes for first time setup)...

Creating skaffold file: ...

Configuring image push settings in ...

/Users/apeltop/Library/Application ...
Generating tags...
 - gcr.io/dbdt/server -> gcr.io/dbdt/server:latest
Checking cache...
 - gcr.io/dbdt/server: Not found. Building
Starting build...
Building [gcr.io/dbdt/server]...
Pushing code to gs://dbdt_cloudbuild/source/...
time="2023-05-25T23:20:47+09:00" level=warning msg="Skipping venv/.Python. Only relative symlinks are supported." subtask=-1 task=DevLoop
time="2023-05-25T23:20:47+09:00" level=warning msg="Skipping venv/bin/python. Only relative symlinks are supported." subtask=-1 task=DevLoop
Logs are available at 
https://storage.cloud.google.com/dbdt_cloudbuild/...
starting build "1807be8b-..."

FETCHSOURCE
Fetching storage object: gs://dbdt_cloudbuild/source/...
Copying gs://dbdt_cloudbuild/source/...
```

```bash
"/Users/apeltop/Library/Application Support/google...
Deploying container to Cloud Run service [server] in project [dbdt] region [asia-northeast3]
Deploying new service...
Setting IAM Policy....................done
Creating Revision...........................................................................................................done
Routing traffic.....done
Done.
Service [server] revision [server-00001-kam] has been deployed and is serving 100 percent of traffic.
Service URL: https://... 
```

### 배포 확인하기
Cloud Run 에 들어가 리스트를 확인해보면 server 라는 서비스가 생성된 것을 확인할 수 있다.

![11-4.png](/assets/img/dbdt/11-4.png)

그리고 해당 서비스를 클릭하면 아래와 같이 서비스의 상태를 확인할 수 있다.

![11-5.png](/assets/img/dbdt/11-5.png)



