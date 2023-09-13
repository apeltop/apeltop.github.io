---
title: Promise.all 을 사용해 처리 속도 높이기
author: sunshine@ptokos.com
categories: [javascript, enhancement]
---

마주친 상황은 크롤링을 통해 얻은 정보를 GCP Storage 에 올려야하는데 수십 ~ 수백 개의 json 파일과 이미지 파일을 올려하는 상황이다.

우선 처음 코드는 GCP Storage 공식 문서에 있는 [Upload a directory to a bucket](https://github.com/googleapis/nodejs-storage/blob/main/samples/uploadDirectory.js) 를 가져와 사용하였다.

```typescript
for await (const filePath of getFiles(directoryPath)) {
    try {
        const dirname = path.dirname(directoryPath);
        const destination = path.relative(dirname, filePath);

        await bucket.upload(filePath, {destination});

        console.log(`Successfully uploaded: ${filePath}`);
        successfulUploads++;
    } catch (e) {
        console.error(`Error uploading ${filePath}:`, e);
    }
}
```

위 코드의 경우 json 파일 291개와 이미지 파일 289개를 올리는데 12분 41초가 소요되었다.
- json : 5:28.966 (m:ss.mmm)
- image : 7:13.189 (m:ss.mmm)
- total : **12:41.155** (m:ss.mmm)

굉장히 오랜 시간이 걸리는데 이를 속도를 줄이고 싶다.

```
289 files uploaded to tech-collector-crawling-bucket successfully.
upload json files: 5:28.966 (m:ss.mmm)
```

```
291 files uploaded to tech-collector-crawling-bucket successfully.
upload images: 7:13.189 (m:ss.mmm)

```

### Promise.all
순차적으로 하나씩 파일을 업로드하는 것이 아니라 병렬적으로 업로드를 진행하면 속도를 높일 수 있을 것이다.

아래처럼 tasks 에 작업할 것을 담아두고 Promise.all 을 통해 병렬적으로 실행하도록 하였다.

```typescript
const tasks = [];

for await (const filePath of getFiles(directoryPath)) {
    tasks.push(async () => {
        try {
            const dirname = path.dirname(directoryPath);
            const destination = path.relative(dirname, filePath);

            await bucket.upload(filePath, {destination});
            console.log(`Successfully uploaded: ${filePath}`);
            successfulUploads++;
        } catch (e) {
            console.error(`Error uploading ${filePath}:`, e);
        }
    })
}

await Promise.all(tasks.map(task => task()));

```

실행 결과는 아래와 같이 획기적으로 줄었다.

- json : 10.080s
- image : 32.311s
- total : **42.391s**

```
291 files uploaded to tech-collector-crawling-bucket successfully.
upload json files: 10.080s
```

```
293 files uploaded to tech-collector-crawling-bucket successfully.
upload images: 32.311s
```


### 결론
기존에 대략 13분 가량 걸리던 것이 42초로 줄었다. 퍼센트로 따지면 **96%** 가량 줄었다.
