---
title: firestore document 최대 크기 알아보기
author: sunshine@ptokos.com
categories: [javascript]
---

firestore 는 document database 이다. document 는 json 형태로 저장된다. 
각 DB 마다 document 최대 크기가 다르기에 최대 크기를 확인해보는게 좋다.

firestore 에서 document 의 [최대 크기는 1MB](https://firebase.google.com/docs/firestore/quotas#collections_documents_and_fields) 이다. 

당연히 공식 문서처럼 사용할 수 있겠지만 궁금해서 직접 확인해보았다.

### 에러 발생시키기
```typescript
export const executeMaxSizeOverflow = async () => {
    const labRef = collection(db, 'labs');
    let documentReference = await addDoc(labRef, {});

    try {
        for (let i = 0; i < 1000; i++) {
            await updateDoc(documentReference, {
                [`data${i}`]: randomString(100000),
            });

            console.log(i);
        }
    } catch (e) {
        console.log(e);
    }
}
```

executeMaxSizeOverflow 를 실행시키면 우선 아래와 같이 정보가 저장이 된다.

![firestore-document-max-size-1.png](/assets/img/firebase/firestore-document-max-size-1.png)

data 의 index 를 보면 0~9 번까지만 저장이 되었다.

실행한 결과는 당연히 에러를 발생시킨다.

```
FirebaseError: Document 'projects/.../labs/4IQLTO1WWiC5bVucZwvM' cannot be written because its size (1,100,156 bytes) exceeds the maximum allowed size of 1,048,576 bytes.
```

