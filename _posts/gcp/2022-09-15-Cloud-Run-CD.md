---
title: Cloud Run 에 GitHub 연동하여 CD 구축하기
author: sunshine@ptokos.com
categories: [GCP]
---

### Cloud Run
Google Cloud Platform(이하 GCP) 는 Serverless 제품을 2가지 가지고 있다.
- Cloud Run
- Cloud Functions

Cloud Run 은 Fully Managed 로서 한 번의 배포로 작업이 끝날 수 있도록 많은 기능을 제공한다.
예를 들어 autoscaling, 지역별 이중화(Redundancy), 보안(Security) 등 많은 편의 기능을 제공한다.

Cloud Run 의 대표적인 특징 2가지이다.
#### Runtime 제한이 없다
Cloud Run 을 좀 더 입체감 있게 소개하기 위해서 Cloud Functions 와 차이점을 가지고 설명해보려한다.

Cloud Run 은 Container Image 로 배포되어 언어 사용에 제한이 없다.

반면에 Cloud Function 은 [언어의 제한](https://cloud.google.com/functions/docs/concepts/execution-environment)이 있다.
![Alt text](/assets/img/gcp/cloud-run-cd/cloud-function-runtime.png)

#### 시스템 패키지를 사용할 수 있다.
Cloud Run 은 Docker 를 사용하기 때문에 Dockerfile 에서 패키지 설치 구문을 추가할 수 있다.
예를 들어 아래 같이 apt-get 을 사용할 수 있다.
```dockerfile
RUN apt-get update -y && apt-get install -y \
  graphviz \
  && apt-get clean
```

반면에 Cloud Function 은 시스템 패키지를 사용할 수 없다.

### 1. Express 서버 구성
[예제 코드 github](https://github.com/apeltop/cloud-run-nodejs)

우선 express 환경이 구성되어있다고 가정하고 router 코드만 보겠습니다.
아래 router 는 GET / 접근 시 서버가 시작된 시간과 node version 을 출력한다.
```javascript
var express = require('express');
var router = express.Router();

const startUpDate = new Date(Date.now());

router.get('/', function(req, res, next) {
    res.json({
        startUpDate,
        version: process.version
    });
});

module.exports = router;
```
![Alt text](/assets/img/gcp/cloud-run-cd/1.png)


### 2. Image 생성
#### 2.1 Dockerfile 생성
프로젝트 루트에 Dockerfile 빈 파일을 생성한다.

#### 2.2 
아래 과정을 설명하면 아래와 같다.
1. Node 16 버전을 가져오기
2. app directory 를 생성하고 이동하기
3. package.json 파일을 container 디렉토리에 복사하기
4. dependencies 설치하기
5. 코드를 container image 로 복사하기
6. 서버 시작하기

```dockerfile
FROM node:16

WORKDIR /app

COPY package.json .

RUN yarn install

COPY . .

CMD [ "yarn", "start" ]

```

### 3. Cloud Build 를 통해 Image 생성하기
여기서 PROJECT_ID 는 GCP 프로젝트 ID 를 의미한다.
gcr.io/PROJECT_ID/production 에서 production 은 Cloud Storage 에 저장될 디렉토리 이름이다. 
즉 바뀌어도 무방하다

```
gcloud builds submit --tag gcr.io/PROJECT_ID/production --project PROJECT_ID
```

프로젝트 처음 만들고 실행할 경우에 위 구문을 실행시키면 아래와 같이 출력이 되면서 cloudbuild 를 사용할건지 물어본다.
여기서 y 를 눌러주면 자동으로 처리해준다.
![Alt text](/assets/img/gcp/cloud-run-cd/2.png)

터미널에서 보이듯 log 를 볼 수 있다는 링크를 클릭하면 Cloud Build 이 열리면서 아래와 같이 Status 가 빙글빙글 돌고있는 것을 확인할 수 있다.
![Alt text](/assets/img/gcp/cloud-run-cd/3.png)

여기서 Build 에 해당하는 열을 클릭하면 열심히 빌드하고 있는 것을 볼 수 있다.
![Alt text](/assets/img/gcp/cloud-run-cd/4.png)
![Alt text](/assets/img/gcp/cloud-run-cd/5.png)

빌드가 완료되면 터미널은 Image 를 확인할 수 있는 URL 을 반환한다. 여기에서 반환된 URL(gcr.io 로 시작하는) 것은 기록해둘 필요가 있다.
뒤이어 사용한다.
![Alt text](/assets/img/gcp/cloud-run-cd/6.png)


### 4. Cloud Run 배포
GCP console 에서 Cloud run 을 처음 들어가면 아래와 같은 화면일 것이다.(API Enabled 를 했다면)
![Alt text](/assets/img/gcp/cloud-run-cd/7.png)

여기서 Create Service 를 클릭해준다.
![Alt text](/assets/img/gcp/cloud-run-cd/8.png)

Container image URL 에 3번 단계에서 확인한 gcr.io/~ 를 입력한다.
![Alt text](/assets/img/gcp/cloud-run-cd/9.png)

지역은 서울로 해준다. (당연히 이 부분은 선택사항이다.)
![Alt text](/assets/img/gcp/cloud-run-cd/10.png)

테스트이기 때문에 Maximum number of instance 를 100 -> 5로 조정하고 Authentication 은 누구든지 접속할 수 있도록 하였다. 
![Alt text](/assets/img/gcp/cloud-run-cd/11.png)

위 설정을 마쳤으면 생성 버튼을 클릭한다.

빙글빙글 돌며 열심히 배포 진행을 하고있다.
![Alt text](/assets/img/gcp/cloud-run-cd/12.png)

배포가 완료되면 ✅ 가 된다.
![Alt text](/assets/img/gcp/cloud-run-cd/13.png)

상단에 URL 을 클릭해주면 로컬에서와 같이 출력이 된다.
![Alt text](/assets/img/gcp/cloud-run-cd/14.png)

### 5. Github 과 여동하기
상단의 SET UP CONTINUOUS DEPLOYMENT 을 클릭해준다.
![Alt text](/assets/img/gcp/cloud-run-cd/15.png)

Repository Provider 를 Github 을 선택하고 아래의 Repository 를 선택해준다.
![Alt text](/assets/img/gcp/cloud-run-cd/16.png)
![Alt text](/assets/img/gcp/cloud-run-cd/17.png)


트리거 대상이 될 branch 를 입력하고 Build type 은 Dockerfile 을 선택한다. 그리고 저장 버튼을 눌러준다.
![Alt text](/assets/img/gcp/cloud-run-cd/18.png)
![Alt text](/assets/img/gcp/cloud-run-cd/19.png)

그 후 Cloud Build -> Triggers 를 가보면 Trigger 가 하나 생성되어있을텐데 이것은 바로 전에 했던 작업으로 생성된 것이다.
![Alt text](/assets/img/gcp/cloud-run-cd/20.png)

Trigger 상세 보기에 들어가면 설정들이 잘 되어있는 것을 확인할 수 있다.
![Alt text](/assets/img/gcp/cloud-run-cd/21.png)

### 6. 코드 수정 후 master branch 에 push 하기
기존 코드와는 json body 만 달라졌다. app_version 이 추가되었다.
코드 commit 후 push 를 한다.

```javascript
var express = require('express');
var router = express.Router();

const startUpDate = new Date(Date.now());

router.get('/', function(req, res, next) {
  res.json({
    startUpDate,
    node_version: process.version,
    app_version: '1.0.1'
  });
});

module.exports = router;
```

Cloud Build -> History 를 보면 빌드가 진행되고 있음을 볼 수 있다.
![Alt text](/assets/img/gcp/cloud-run-cd/22.png)


![Alt text](/assets/img/gcp/cloud-run-cd/23.png)


![Alt text](/assets/img/gcp/cloud-run-cd/24.png)


Cloud Run 에 다시 들어가 보면 새로운 Revision 이 생기고 새로운 Revision 이 활성화되어있는 것을 볼 수 있다.
![Alt text](/assets/img/gcp/cloud-run-cd/25.png)

아까 방문했었던 URL 을 새로고침해보면 새로운 버전의 데이터가 정상적으로 반환되는 것을 볼 수 있다.
![Alt text](/assets/img/gcp/cloud-run-cd/26.png)


이번 포스팅에서는 Cloud Run 과 Github 를 연동해서 특정 브랜치에 push 가 될 경우 자동으로 배포되는 예제를 통해 알아보았다.
