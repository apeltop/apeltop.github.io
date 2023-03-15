---
title: GCP Cloud Function ffmpeg troubleshooting
author: sunshine@ptokos.com
categories: [expo]
---

## 해결 방법
>  Node Version 18 -> 16 으로 변경

## 문제 상황
Cloud Function 에서 ffmpeg 사용하려고하니 _Error: ffmpeg was killed with signal SIGSEGV_ 이 발생
![1.png](/assets/img/gcp/Cloud-Function-ffmpeg-troubleshooting/1.png)

### 원인 파악

#### 1. ffmpeg 설치 확인
우선 Cloud Function 은 Serverless 제품으로 apt-get 으로 설치를 할 수 없다.
그렇기에 우선 [Cloud Function 에서 System Package](https://cloud.google.com/functions/docs/reference/system-packages) 로 제공하는지 확인해야한다.
![2.png](/assets/img/gcp/Cloud-Function-ffmpeg-troubleshooting/2.png)

Cloud Function 에서 ffmpeg 패키지를 제공하는 것을 확인할 수 있다.

#### 2. ffmpeg 연결 확인
Cloud Function 위에서 ffmpeg 가 실행되니 ffmpeg 를 찾을 수 없다는 에러가 출력되었다.
```
Error: Cannot find ffmpeg
    at /layers/google.nodejs.yarn/yarn_modules/node_modules/fluent-ffmpeg/lib/processor.js:136:22
    at /layers/google.nodejs.yarn/yarn_modules/node_modules/fluent-ffmpeg/lib/capabilities.js:123:9
    at wrapper (/layers/google.nodejs.yarn/yarn_modules/node_modules/async/dist/async.js:271:20)
```
![3.png](/assets/img/gcp/Cloud-Function-ffmpeg-troubleshooting/3.png)

서버에 내장되어있는 ffmpeg 가 불러와지지 않는다는 것이기 때문에 [글들을 찾다보니](https://stackoverflow.com/questions/62652721/nodejs-fluent-ffmpeg-cannot-find-ffmpeg-for-firebase-cloud-functions) path 를 설정해줘야한다는 것을 알게되었다.
 
#### 3. ffmpeg path 설정
@ffmpeg-installer/ffmpeg 에서 path 를 불러와 ffmpeg.setFfmpegPath(path) 사용해 설정해주면 된다.

```typescript
import ffmpeg from 'fluent-ffmpeg';
import {path} from '@ffmpeg-installer/ffmpeg';

ffmpeg.setFfmpegPath(path);
```

#### 4. ffmpeg 실행
ffmpeg 는 찾았으나 _Error: ffmpeg was killed with signal SIGSEGV_ 가 발생한다.
```
Error: ffmpeg was killed with signal SIGSEGV
    at ChildProcess.<anonymous> (/workspace/node_modules/fluent-ffmpeg/lib/processor.js:180:22)
    at ChildProcess.emit (node:events:513:28)
```
![1.png](/assets/img/gcp/Cloud-Function-ffmpeg-troubleshooting/1.png)

사실 이 에러로 정말 한 10시간은 보낸 것 같다. 
Cloud Function 2세대를 사용해서 그런가해서 1세대로 테스트하고 비동기처리 도중에 서버 리소스가 반환이 되나 싶어 여러 테스트를 진행하는 등 꽤 갈피를 못잡고 헤맸다.

### 해결 작업
#### Cloud Function Node Version 변경
Cloud Function 에서 node18 을 제공하길래 별 생각 없이 환경을 구성하였다.
문득 로컬에서는 돌아가고 Cloud Run 에서 돌아가는데 뭐가 다를까 싶은 마음에 로컬 환경처럼 Cloud Function 의 Runtime 을 node16 으로 변경하였다.
그랬더니 정상적으로 ffmpeg 가 실행되었다.

### 결론
해결하니 안도감이 들면서 왜 이제 이게 생각이 났는지 그리고 고생한거에 비해서 node 버전 문제여서 허탈하면서 그 덕분에 여러 문서를 읽고 더 테스트하느라 재미는 있었고 여러 감정이 복잡미묘했다.
그렇지만 종종 느끼지만 이런게 개발자의 나름 재미라고 생각한다. 결국 어려운 길 돌고돌아 쉬운 해답을 찾는 것.
