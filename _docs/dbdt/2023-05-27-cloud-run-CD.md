---
title: Cloud Run CD 구축하기
layout: doc
subtitle: main 브랜치에 push 시 Cloud Run 에 자동 배포되도록 하기  
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`main 브랜치에 push 시 Cloud Run 에 자동 배포되도록 하기`

## CD(Continuous Deployment) 구축하기
Cloud Run 에 들어가서 SET UP CONTINUOUS DEPLOYMENT 를 클릭한다.  

![16-1.png](/assets/img/dbdt/16-1.png)

Cloud Build 를 통한 Image 를 사용해야하기 때문에 Artiface Registry API 를 허용해줘야한다.

그 후 Repository Provider 에 Github 을 선택하고 Repository 를 선택해주면 된다. 선택 후 NEXT 버튼을 클릭해준다.
![16-2.png](/assets/img/dbdt/16-2.png)

Branch 는 regex 로 설정이 가능하기 때문에 필요에 따라서 설정하면 된다.
이번 포스팅에서는 main 일 때 배포되도록 설정한다.

Build Type 도 선택해야하는데 Dockerfile 를 통해 배포하도록 하였기 때문에 Dockerfile 을 선택해주고 Dockerfile 의 위치를 입력해주면 된다.
필자의 경우 /server 라는 폴더 안에 파일이 있기 때문에 /server/Dockerfile 이다. 정보를 다 입력헀다면 SAVE 버튼을 클릭해준다.

![16-3.png](/assets/img/dbdt/16-3.png)

이 때 생성된 부분은 Cloud Build > Triggers 를 들어가서 확인해볼 수 있다.
그럼 아래 그림과 같이 생성된 것을 확인할 수 있다.
![16-6.png](/assets/img/dbdt/16-6.png)

Cloud Run 에서 CD 설정을 완료하면 설정한 내용을 바탕으로 배포를 진행해주는데 에러가 발생했다. 
다급히 log 를 확인해봤다.

![16-4.png](/assets/img/dbdt/16-4.png)

이 region 에서는 빌드를 실행할 수 없다고 나온다. 매우 당황스럽지만 찾아보니 그럴 수도 있다고 안내는 해주고 있다..

[Restricted regions for some projects](https://cloud.google.com/build/docs/locations#restricted_regions_for_some_projects)

![16-5.png](/assets/img/dbdt/16-5.png)

그래서 Cloud Build > Triggers 에서 만들어진 trigger 에 들어가 Region 을 asia-northeast3(Seoul) -> global 로 변경하였다. 

![16-7.png](/assets/img/dbdt/16-7.png)

![16-8.png](/assets/img/dbdt/16-8.png)

Cloud Build > Triggers 에서 Run 을 클릭해서 테스트 해본다.

![16-6.png](/assets/img/dbdt/16-6.png)

Build 내역은 Cloud Build > History 에서 확인할 수 있다.

![16-9.png](/assets/img/dbdt/16-9.png)

Build 열에 해당하는 부분을 클릭하면 진행 상황을 볼 수 있다.
시간을 좀 기다리면 Build, Push, Deploy 단계를 거쳐 배포가 완료된다.

![16-10.png](/assets/img/dbdt/16-10.png)

Cloud Run 의 Revisions 을 확인해보면 새로운 버전이 배포된 것을 확인할 수 있다.(바로 캡쳐를 못해서 35분 전입니다..ㅎ)
![16-11.png](/assets/img/dbdt/16-11.png)








