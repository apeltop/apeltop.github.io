---
title: firebase admin 설정하기
author: sunshine@ptokos.com
categories: [javascript, firebase, firestore]
---


## 목표
firebase 를 사용하다보면 데이터 구조가 전체적으로 바뀌어야하거나 데이터를 한번에 수정해야할 때가 있다. 이럴때는 firebase console 을 사용하는 것 보다는 firebase admin 을 사용하는 것이 좋다.


### firebase admin 패키지 추가
새로운 프로젝트를 만들어서 firebase admin 패키지를 추가한다.

```bash
yarn add firebase-admin
```

### credential 파일 생성 
firebase console 에서 Project Overview > Project settings > Service accounts 로 이동한다.
![firebase-admin-setup-1.png](/assets/img/firebase/firebase-admin-setup-1.png)

이 화면에서 Generate new private key 버튼을 클릭하면 json 파일이 다운로드 된다. 이 파일을 프로젝트 폴더에 저장한다.

![firebase-admin-setup-2.png](/assets/img/firebase/firebase-admin-setup-2.png)

### firebase admin 설정
```javascript
import {cert, getApps, initializeApp} from 'firebase-admin/app';
import {getFirestore} from "firebase-admin/firestore";

const apps = getApps();
if (!apps.length) {
    initializeApp({
        credential: cert('./apeltop-firebase-adminsdk-....json')
    });
}
```

### firestore 사용하기
#### 컬렌션 전체 조회
```javascript
const db = getFirestore();
const iDoc = await db.collection('itineraries').get();

iDoc.forEach(doc => {
    console.log(doc.id, '=>', doc.data());
});
```

#### document 조회
```javascript
const doc = await db.collection('itineraries').doc('document-id').get();
```

#### document 추가
```javascript
const doc = await db.collection('itineraries').add({
    name: 'new document'
});
```

#### document 수정
```javascript
const doc = await db.collection('itineraries').doc('document-id').set({
    name: 'new name'
}, {merge: true});
```

#### document 삭제
```javascript
const doc = await db.collection('itineraries').doc('document-id').delete();
```









