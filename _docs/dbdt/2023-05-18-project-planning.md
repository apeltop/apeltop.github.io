---
title: 프로젝트 구성
layout: doc
subtitle: front-end, back-end 구성하기
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

## front-end
밑바닥부터 만들기에는 너무 방대하고 리소스가 많이 든다. 그러기에 [Creative Tim](https://www.creative-tim.com) 에 있는 템플릿을 구매해서 사용하기로 하였다.

대략적인 느낌은 아래와 같다.
![2-1.png](/assets/img/dbdt/2-1.png)
![2-2.png](/assets/img/dbdt/2-2.png)

템플릿이여도 기술 스택은 **React + Typescript** 기반이다.

이를 토대로 코드를 입히며 개발하려한다.


## back-end
백엔드는 회사에서 주로 사용하는 것이 express 였기 때문에 익숙한 것으로 할까 싶었지만 데분당태 클럽에서 시작되었고 교육을 목표로 한 그룹이다.
그렇기에 python 기반으로 해서 나중에 교육용으로 써도 되고 실제로 기능 추가 등을 할 수 있으면 좋겠다는 생각이 들어 python 기반인 [FastAPI](https://fastapi.tiangolo.com) 를 선택했다.

나중에 인공지능 등이 추가될 때 python 코드일 경우 확장성 또한 좋기 때문이다.





