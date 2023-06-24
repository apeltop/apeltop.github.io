---
title: SourcePipeline 을 통해 CI/CD 통합 관리하기
layout: doc
subtitle: SourceCommit 에 git repository 를 연동하기
author: sunshine@ptokos.com
tags: [ncloud]
comments: true
---

## 목표
`SourceCommit, SourceBuild, SourceDeploy 를 사용하는 프로레스를 즉 파이프라인을 구축해보자`

![9-1.png](/assets/img/ncloud-sourcepipeline/9-1.png)

## SourcePipeline 이란
[SourcePipeline](https://www.ncloud.com/product/devTools/sourcePipeline) 은 SourceCommit, SourceBuild, SourceDeploy 를 통합 관리하는 서비스이다.
즉 배포에 필요한 일련의 과정을 자동화할 수 있게 해준다. **SourceCommit**, **SourceBuild**, **SourceDeploy** 는 각각 정해진 일이 명확하다.
SourcePipeline 은 배포 전략에 따라 어떤 흐름으로 처리하게 할지 정해주는 역할을 한다.

이번 글에서는 SourceCommit -> SourceBuild(배포 파일 생성) -> SourceDeploy(dev stage) -> SourceDeploy(real stage) 의 흐름으로 만들어볼 것이다.

## 만들기
Developer Tools > SourcePipeline 에 들어간다.
그 후 **파이프라인 생성** 을 클릭한다.

![9-2.png](/assets/img/ncloud-sourcepipeline/9-2.png)

파이프라인 이름을 입력해준다.

![9-3.png](/assets/img/ncloud-sourcepipeline/9-3.png)

파이프라인 구성 단계에서 **작업 추가**를 클릭한다.

![9-4.png](/assets/img/ncloud-sourcepipeline/9-4.png)

타입이 SourceBuild 인 작업을 추가해준다. 

![9-5.png](/assets/img/ncloud-sourcepipeline/9-5.png)

그러면 아래와 같이 sample-build 작업이 추가된다.  

![9-6.png](/assets/img/ncloud-sourcepipeline/9-6.png)

타입이 SourceDeploy 인 dev stage 작업을 추가해보자.

![9-7.png](/assets/img/ncloud-sourcepipeline/9-7.png)

그리고 타입이 SourceDeploy 인 real stage 작업을 추가해보자.

![9-8.png](/assets/img/ncloud-sourcepipeline/9-8.png)

그러면 아래와 같이 3개의 작업이 생성이 된다.
![9-9.png](/assets/img/ncloud-sourcepipeline/9-9.png)

파이프라인은 관로라는 뜻으로 A에서 B로 흘러가는 것을 의미한다.
그러기에 작업을 만들었지만 작업의 흐름을 만들어줘야한다.

![pipeline.png](https://images.unsplash.com/photo-1635145613344-3e59b1e8afd0?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1587&q=80)

sample-build 작업 왼쪽의 **+** 버튼을 클릭한다. 

![9-10.png](/assets/img/ncloud-sourcepipeline/9-10.png)

SourceBuild 의 선행 작업은 없기에 **선행작업 없음**을 선택해주고 생성한다.

![9-11.png](/assets/img/ncloud-sourcepipeline/9-11.png)

그럼 오른쪽 작업에 sample-build 작업이 추가된다.

![9-12.png](/assets/img/ncloud-sourcepipeline/9-12.png)

이번엔 dev-deploy 작업의 왼쪽 **+** 버튼을 누른다. 

![9-13.png](/assets/img/ncloud-sourcepipeline/9-13.png)

SourceBuild 를 선행작업으로 선택하고 생성한다.

![9-14.png](/assets/img/ncloud-sourcepipeline/9-14.png)

그럼 오른쪽 작업에 sample-build 작업과 연결이 된 dev-deploy 작업이 추가된다.

![9-15.png](/assets/img/ncloud-sourcepipeline/9-15.png)

real-deploy 작업은 dev-deploy 를 선행작업으로 선택하고 생성한다.

![9-16.png](/assets/img/ncloud-sourcepipeline/9-16.png)

그럼 아래와 같이 3개의 작업이 연결된 파이프라인이 생성된다.

![9-17.png](/assets/img/ncloud-sourcepipeline/9-17.png)

이제 Trigger 를 설정해야한다. SourceCommit 에서 push 가 발생하면 SourcePipeline 을 통해 배포가 진행되도록 설정하였다.

![9-18.png](/assets/img/ncloud-sourcepipeline/9-18.png)

최종 정보가 맞으면 파이프라인을 생성한다.

![9-19.png](/assets/img/ncloud-sourcepipeline/9-19.png)

SourcePipeline 에 보면 sample-**pipeline** 이 추가된 것을 볼 수 있다.
파이프라인을 클릭해서 상세보기로 들어가보자

![9-20.png](/assets/img/ncloud-sourcepipeline/9-20.png)

여기서 **파이프라인 실행하기**를 눌러 제대로 동작하는지 확인해보자.

![9-21.png](/assets/img/ncloud-sourcepipeline/9-21.png)

그 전에 예상 했던대로 동작하는지 확인해보기 위해 [로드 밸런서 구축](/docs/ncloud-sourcepipeline/2023-06-19-load-balancer/) 에서 만들었던 파이썬 기반 모니터링을 통해 살펴보자.
1초 간격으로 1000번 실행되도록 했기 때문에 파일을 실행 후 **파이프라인 실행**하기를 누른다.

그 후 **작업결과** 탭으로 이동해보면 빌드부터 진행중인 것을 볼 수 있다.

![9-22.png](/assets/img/ncloud-sourcepipeline/9-22.png)

또한 SourceBuild 서비스에 들어가 작업 결과를 보면 열심히 빌드 중인 것을 볼 수 있다.

![9-23.png](/assets/img/ncloud-sourcepipeline/9-23.png)

빌드 작업이 끝나면 그 뒤에 작업인 dev-deploy 작업이 진행된다.

![9-24.png](/assets/img/ncloud-sourcepipeline/9-24.png)

dev-deploy 작업이 끝나면 real-deploy 작업이 진행된다.

![9-25.png](/assets/img/ncloud-sourcepipeline/9-25.png)

SourceBuild 와 마찬가지로 SourceDeploy 에 들어가서 작업 결과를 보면 dev, real 각각 시나리오가 진행되는 것을 볼 수 있다.

![9-26.png](/assets/img/ncloud-sourcepipeline/9-26.png)

3개의 작업 모두 성공적으로 진행이 되었다.

![9-27.png](/assets/img/ncloud-sourcepipeline/9-27.png)

모니터링 기록을 통해 살펴보자.

빌드 과정과 deploy-dev 에서 배포 직전까지는 server1, server2 요청이 번갈아가며 진행된 것을 볼 수 있다.

```bash
```bash
0 10.41.151.114 484a4266-3207-4ad8-9ac7-da15127b0a3d v1
1 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
...
144 10.41.151.114 484a4266-3207-4ad8-9ac7-da15127b0a3d v1
145 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
```

server2 에 배포가 진행되는 잠깐 동안에는 2번 정도 요청이 실패하고 LB 가 server1 로 요청을 보내는 것을 볼 수 있다.

```bash
request error
148 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
request error
150 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
151 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
152 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
153 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
154 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
```

server2 가 배포가 완료되어 헬스 체크가 통과한 후 다시 server1 과 server2 요청이 번갈아가며 진행된다.
이 때 server2 의 UUID 가 바뀐 것 또한 확인할 수 있다.

```bash
166 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
167 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
168 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
169 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
170 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
171 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
172 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
173 10.41.126.159 3d01349c-97e5-400f-8feb-06038c574109 v1
174 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
```

이번엔 real-deploy 단계에서 서버가 켜질 대 잠깐 동안 서버가 응답하지 않는 것을 볼 수 있다.
LB 는 다시 서버2로만 요청을 보내고 있다.

```bash
request error
request error
177 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
178 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
179 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
180 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
181 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
182 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
183 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
184 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
185 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
186 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
187 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
188 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
189 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
```

서버1이 헬스 체크 통과한 후에는 다시 서버1과 서버2로 요청이 번갈아가며 진행된다. 

```bash
190 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
191 10.41.126.159 e6a5cbc8-d6d5-44b8-b622-5693b376433e v1
192 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
193 10.41.126.159 e6a5cbc8-d6d5-44b8-b622-5693b376433e v1
194 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
195 10.41.126.159 e6a5cbc8-d6d5-44b8-b622-5693b376433e v1
196 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
213 10.41.151.114 c7463702-54b5-439e-972c-fc23f7d5f199 v1
```


## 테스트
v 를 v1 -> v2 로 변경하고 Commit 과 Push 를 진행하면 파이프라인이 가동이 된다. 

```java
private final String v = "v1";
```

```java
private final String v = "v2";
```

![9-28.png](/assets/img/ncloud-sourcepipeline/9-28.png)

배포를 진행할 때 모니터링을 하였는데 출력을 통해 제대로 배포가 되었는지 확인해보자.

우선 번갈아가며 요청을 보내는 것을 확인할 수 있다.

```bash
0 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
1 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
2 10.41.151.114 69510160-e08d-4fc9-9e38-b548ecde0547 v1
3 10.41.151.114 69510160-e08d-4fc9-9e38-b548ecde0547 v1
4 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
```

서버2 가 기동되면서 에러를 반환한다.
그 후 서버 1로만 요청이 간다.
```bash
request error
132 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
133 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
134 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
```

서버2 에 요청이 가고 응답 결과를 보니 **v2** 를 반환하는 것을 확인해볼 수 있다. 

```bash
153 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
154 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
155 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
156 10.41.126.159 dabf808b-cf63-40b5-b401-3ad56a5195ce v1
157 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
```

서버1 이 기동될 때 에러를 반환한다.
v2 를 반환하는 서버2 로만 요청이 가는 것을 확인할 수 있다.

```bash
request error
159 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
160 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
161 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
162 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
163 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2

```

서버1 이 헬스 체크를 통과하면 다시 서버1과 서버2로 요청이 번갈아가며 진행된다.
모두 v2 를 반환해 새로운 버전이 적용된 것을 확인할 수 있다.

```bash
175 10.41.126.159 2a106a00-be53-459c-a38c-62191256babe v2
176 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
177 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
178 10.41.126.159 2a106a00-be53-459c-a38c-62191256babe v2
179 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
180 10.41.126.159 2a106a00-be53-459c-a38c-62191256babe v2
181 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
182 10.41.126.159 2a106a00-be53-459c-a38c-62191256babe v2
183 10.41.151.114 67f549c1-a6a1-4c79-b269-23627dcd839b v2
```

















