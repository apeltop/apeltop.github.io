---
title: Vercel 에서 Next.js 배포 시 갑작스런 에러 발생 
author: sunshine@ptokos.com
categories: [nextjs, js]
---

## 상황
> 아주 간단한 코드 수정을 하고 Push 하였는데 Vercel 에서 Next.js 배포 시 갑작스런 에러 발생

커밋 메시지에서 알 수 있듯이 빌드 시 no-lint 옵션만 활성화하였는데 갑작스런 에러가 발생하였다.

![nextjs-deploy-error-1.png](/assets/img/vercel/nextjs-deploy-error-1.png)

## 원인 파악
원인 파악을 하기위해 로그를 확인하였다.
**@firebase/auth** 를 못 찾겠다는 에러가 보인다.

![nextjs-deploy-error-2.png](/assets/img/vercel/nextjs-deploy-error-2.png)

패키지 수정을 하지 않았는데 이런 에러가 발생하여 로그를 위로 올려보며 자세히 보았다.
그랬더니 의심이 가는 부분이 **Detected 'package-lock.json'** 부분이다.

![nextjs-deploy-error-3.png](/assets/img/vercel/nextjs-deploy-error-3.png)

프로젝트 루트에 보면 **package-lock.json** 파일이 **없기** 때문이다.
없는데 왜 detected 라고 나오는 부분이 이상하였다. 

![nextjs-deploy-error-4.png](/assets/img/vercel/nextjs-deploy-error-4.png)

## 해결
해결 방법을 고민하다가 이 프로젝트는 yarn 을 사용하고 있었기에 **yarn.lock** 파일이 있었고 이 파일을 **레포지토리에 추가**하였다.

빌드 로그를 보니 **yarn.lock** 파일이 **detected** 되었다.

![nextjs-deploy-error-5.png](/assets/img/vercel/nextjs-deploy-error-5.png)

빌드가 정상적으로 **완료**가 되고 **배포**까지 진행이 되었다. 
vercel 에서 어떠한 문제로 이러한 문제가 생겼는지는 모르지만 yarn.lock 또는 package-lock.json 파일을 추가하는 것이 확실히 예외적인 상황을 줄일 수 있다고 느낄 수 있었다.

![nextjs-deploy-error-6.png](/assets/img/vercel/nextjs-deploy-error-6.png)

