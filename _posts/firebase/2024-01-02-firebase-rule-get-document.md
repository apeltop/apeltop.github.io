---
title: firestore subCollection 에서 부모 데이터를 가지고 rule 설정하기
author: sunshine@ptokos.com
categories: [javascript, firebase]
---

## 목표
> subCollection 접근할 때 부모 데이터를 가지고 validation 하기

## 결과 코드
```
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
  
    match /COLLECTION_NAME/{documentId} {
      ...
      
      match /picks/{subDocumentId} {
      	allow read, write: if request.auth != null 
        && request.auth.uid in get(/databases/$(database)/documents/COLLECTION_NAME/$(documentId)).data.members;
      }
    }
  }
}
```

## 마주친 상황
현재 데이터 구조는 아래와 같다.
```
collection
  - parentDocument
    - members: {
        - uid1: {...}
        - uid2: {...}
        - uid3: {...}
    }
    - picks (subCollection)
      - pickDocument1
      - pickDocument2
      ...
```

security rule 설정 조건으로 picks 에 접근할 때 현재 로그인한 사용자가 부모 document members 에 있는지 확인해야 한다.


## rule 설정
크게 2가지 방법이 있겠다.

첫번째로는 pick document 에도 members 를 넣어주는 방법이 있다.

두번째로는 부모 document 에서 members 를 가져오는 방법이 있다.

첫번째를 선택하기에는 부모 document 에서 데이터 변경 시 함께 변경을 해줘야하는데 그러기에는 너무 많은 쓰기가 발생한다고 판단된다.
그래서 두번째인 방법인 부모 document 에서 members 를 가져오는 방법을 선택했다.

### 부모 document 에서 members 가져오기
여기서 키워드는 `get` 함수인데 `get` 함수를 사용하면 다른 document 에서 데이터를 가져올 수 있다.

```
request.auth.uid in get(/databases/$(database)/documents/COLLECTION_NAME/$(documentId)).data.members;
```



## 참고 문서

[Access Other Documents](https://firebase.google.com/docs/firestore/security/rules-conditions?hl=en&authuser=0#access_other_documents)
