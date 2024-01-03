---
title: firebase storage 용량 제한 및 type 제한하기
author: sunshine@ptokos.com
categories: [javascript, firebase]
---

## 목표
> 로그인한 사용자만 파일 업로드 가능
> 
> 파일 크기 10MB 제한
> 
> Content-type 제한

## rule 설정 
기본 설정으로 모든 read, write 권한을 허용했다고 가정하자.

```
rules_version = '2';

service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if true;
    }
  }
}
```

### 로그인한 사용자만 파일 업로드 가능

```
allow read: if request.auth != null;
allow write: if request.auth != null;
```

### 파일 크기 10MB 제한

```
allow write: if request.resource.size < 10 * 1024 * 1024;
```

### Content-type 제한

```
allow write: if request.resource.contentType.matches('image/.*');
```

## 최종
```
rules_version = '2';

service firebase.storage {
  match /b/{bucket}/o {

    match /{allPaths=**} {
       allow read: if request.auth != null;
       
       allow write: if request.auth != null 
          && request.resource.size < 10 * 1024 * 1024
          && request.resource.contentType.matches('image/.*');
    }
  }
}
```


혹 10MB 이상 이미지가 필요한 경우는 아래 이미지를 다운로드 받으면 11.6 MB 로 사용할 수 있다.
![11.6MB.jpg](/assets/img/firebase/firebase-storage-validation-rule.jpeg)

## 참고 문서

[Storage Data Validation](https://firebase.google.com/docs/storage/security#data_validation)
