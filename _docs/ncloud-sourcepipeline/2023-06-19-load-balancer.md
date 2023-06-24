---
title: 로드 밸런서 구축
layout: doc
subtitle: server1 과 server2 에 reqeust 를 분배하도록 하기
author: sunshine@ptokos.com
tags: [ncloud]
comments: true
---

## 목표
`로드 밸런서를 사용해 서버 2대에 나누어 처리하도록 하기`

## 만들기
### Load Balancer 란
![5-1.png](/assets/img/ncloud-sourcepipeline/5-1.png)

이미지 출처 : [What is Azure Load Balancer?](https://learn.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)

#### Public Load Balancer
Public Load Balancer 는 외부에서 접속했을 때 Load Balancer 를 통해 서버로 분배하는 방식이다.

#### Internal Load Balancer
Internal Load Balancer 는 외부에서 접속할 수 없고, 내부 요청의 의해 분배가 이루어지는 방식이다.

Public Load Balancer 를 통한 요청이 Internal Load Balancer 를 통해 분배되는 구조로도 구성할 수 있다.

### Load Balancer 구축
Networking > Load Balancer 에 들어간다.

그 후 `로드 밸런서 생성` 을 클릭한다.

![5-2.png](/assets/img/ncloud-sourcepipeline/5-2.png)

로드밸런서 이름을 입력해준다.

#### 네트워크
그 후 네트워크를 Private IP 와 Public IP 중 선택한다. 
Private IP 의 경우 위에서 간단히 설명한 Intenal Load Balancer 에 해당된다.
Public IP 의 경우는 Public Load Balancer 에 해당된다.

이번 예제에서는 외부의 요청을 분배하려고 하기 때문에 **Public IP** 를 선택해준다.

#### 로드 밸런서 설정
80 번 포트로 접근을 하되 8080 번 포트로 요청하도록 하였다. 
당연히 포트는 각자에 상황에 따라 달리할 수 있다.

#### 로드배런싱 알고리즘
##### Round Robin
단순히 순서대로 요청을 분배하는 방식이다.
순차적으로 서버에 요청을 분배한다. 장점은 단순히 순차 분배이기 때문에 **LB 서버 부하가 제일 적다**.
단점은 **특정 서버의 처리가 오래** 걸리거나 **특정 서버가 더 성능이 좋을 때** 부하 문제가 발생할 수 있다.

이러한 문제를 해결하기 위해 **가중치** 를 부여할 수 있다.
이를 Weighted Round Robin 이라고 한다. 서버 성능이 좋은 서버에 2, 평범한 성능인 서버에 1 을 부여하면 2:1 비율로 요청을 분배한다.
ncloud 에서 제공하는 로드밸런서는 가중치를 부여할 수 없다.

![round_robin_algorithm-1.png](https://www.jscape.com/hubfs/images/round_robin_algorithm-1.png)

이미지 출처 [Comparing Load Balancing Algorithms](https://www.jscape.com/hubfs/images/round_robin_algorithm-1.png)

##### Least Connection
단어 그대로 가장 적은 연결을 가진 서버에 요청을 분배하는 방식이다.

아래 첨부한 사진처럼 라운드 로빈의 경우라면 서버 1에 5, 서버 2에 2, 4, 6 이 연결될 것이다.
그렇지만 Least Connection 을 사용하면 서버 1에 요청이 더 적기 때문에 6의 요청은 서버 1에 할당될 것이다.

세션 수를 저장하고 사용해야하기에 메모리를 사용하나 적은 양이다. 대체로 많은 경우 Least Connection 을 선택한다.
성능의 균형을 어느정도 맞춰줄 수 있기 때문이다.

![least_connections_algorithm_solves_this.png](https://www.jscape.com/hubfs/images/least_connections_algorithm_solves_this.png)
이미지 출처 [Comparing Load Balancing Algorithms](https://www.jscape.com/hubfs/images/least_connections_algorithm_solves_this.png)

![least_connections.png](https://www.jscape.com/hubfs/images/least_connections.png)
이미지 출처 [Comparing Load Balancing Algorithms](https://www.jscape.com/hubfs/images/least_connections.png)

##### Source IP Hash
Source IP Hash 는 Hashing Methods 중 하나이다. 
Hashing Methods 는 **URL Hash method**, **Source IP Hash method** 가 있다.

URL Hash method 는 URL 을 해싱하여 요청을 분배하는 방식이다.

Source IP Hash method 는 Source IP, Source Port, Destination IP, Destination Port 등을 해싱하여 요청을 분배하는 방식이다.
Source IP, Destination IP 등 해싱에 필요한 정보가 같다면 같은 서버로 요청이 간다.
Source IP Hash method 의 방식은 수평 확장(Vertical Scaling) 과 같이 서버가 증설되었을 때 기존의 세션이 다른 곳으로 이동하는 문제가 발생한다.

![DTC-source-10.png](https://blogs.infoblox.com/wp-content/uploads/DTC-source-10.png)
이미지 출처 [Source IP Hash Load Balancing for Application Persistency](https://blogs.infoblox.com/community/source-ip-hash-load-balancing-for-application-persistency/)

![5-3.png](/assets/img/ncloud-sourcepipeline/5-3.png)

#### 서버 선택
**다음** 버튼을 클릭하면 서버 추가 화면이 나온다.
여기서 server1, server2 가 목록에 추가되어있다.
![5-4.png](/assets/img/ncloud-sourcepipeline/5-4.png)

server1, server2 모두 적용 서버로 옮기어준다.
![5-5.png](/assets/img/ncloud-sourcepipeline/5-5.png)

#### 설정 확인 후 생성
설정을 확인하고 문제 없으면 **로드 밸런서 생성**을 클릭해준다.

![5-6.png](/assets/img/ncloud-sourcepipeline/5-6.png)

로드밸런서 생성중인 것을 볼 수 있다. 

![5-7.png](/assets/img/ncloud-sourcepipeline/5-7.png)

#### 로드밸런서 정보 확인
여기서 설정한 정보를 볼 수 있다. 적용 서버를 보면 server1, server2 가 운영중이라는 것을 볼 수 있다.
여기서 접속 정보를 보면 **sib-17885866.ncloudsib.com** 이 나오는데 여기에 접속하면 server1, server2 에 분배한다. 
더 자세한 건 뒤에서 확인해보자.

![5-8.png](/assets/img/ncloud-sourcepipeline/5-8.png)

**로드밸런서 상태 확인**을 클릭하면 아래와 같이 서버포트, L7 Health Check 등 여러 정보를 확인할 수 있다.

![5-9.png](/assets/img/ncloud-sourcepipeline/5-9.png)

여기서 만든 예제는 Classic 기반이라 subnet 정보가 없다. VPC 기반일 경우 subnet 정보도 확인할 수 있다.
subnet 정보를 통해 ACG 를 설정하는 것이 좋다. 외부에서는 server1, server2 에 접속할 수 없고 LB 를 통해서만 접근할 수 있도록 하기 위함이다.

해당 글은 MANVSCLOUD, [2-1. [NCLOUD] 네이버 클라우드에서의 보안 – SERVER (ACG)](https://manvscloud.com/?p=859) 를 참고하면 좋다.

## 로드밸런서 테스트
로드밸런서의 역할과 동작을 확인해보려 한다.

python 기반으로 코드를 간결하게 적어 테스트하였다.
아래 코드는 **requests** 패키지만 추가하면 실행할 수 있다.

```bash
pip install requests
```

### server1, server2 서버 접속
```python
import requests

SERVER_IP1 = 'http://49.50.162.69:8080'
SERVER_IP2 = 'http://101.101.211.102:8080'

res1 = requests.get(f'{SERVER_IP1}/simple_info.do')
print(res1.text)

res2 = requests.get(f'{SERVER_IP2}/simple_info.do')
print(res2.text)
```

#### 결과
```bash
10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
```

각 서버에 접속이 잘되는 것을 확인하였다.

### LB(Load Balancer) 접속
```python
import requests

LB_IP1 = 'http://slb-17885866.ncloudslb.com'
for i in range(10):
    res_LB = requests.get(f'{LB_IP1}/simple_info.do')
    print(i, res_LB.text)
```

#### 결과
```bash
0 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
1 10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
2 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
3 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
4 10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
5 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
6 10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
7 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
8 10.41.126.159 239785bf-20d5-4ca4-8223-99a4c449abe0 v1
9 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
```

LB 에 접속하면 server1, server2 에 요청을 번갈아가며 보내는 것을 확인할 수 있다.

### LB 균등 분배 확인
```python
import requests

LB_IP1 = 'http://slb-17885866.ncloudslb.com'

lb_to_server1_count = 0
for i in range(1000):
    try:
        sleep(1)

        res_LB = requests.get(f'{LB_IP1}/simple_info.do', timeout=3)
        print(i, res_LB.text)
        if res_LB.text.find('10.41.151.114'):
            lb_to_server1_count += 1
    except Exception as e:
        print('request error')

print(lb_to_server1_count)
```

```bash
500
```

결과는 500으로 딱 1000개의 절반이다. 일의 자리 정도는 차이날 수 있겠다 싶었는데 딱 떨어진다.
이로써 분배가 잘 되는 것을 확인하였다.

### 서버 1 다운되었을 때
테스트는 1초에 한 번 씩 LB 에 요청을 보내면서 서버 1을 다운시켜보았다.

![5-10.png](/assets/img/ncloud-sourcepipeline/5-10.png)

```bash
0 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
1 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
2 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
3 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
4 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
5 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
6 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
7 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
8 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
9 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
10 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
11 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1

...

192 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
193 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
194 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
195 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
196 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
197 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
198 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
199 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
200 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
201 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
202 10.41.126.159 a8289e7f-b9cd-460c-a437-7d53dbadfd30 v1
203 10.41.151.114 9478b18b-d362-4979-ae76-56cbeb942165 v1
```

이 때 LoadBalancer 를 들어가보니 server1 이 **health check** 가 실패한 것을 확인할 수 있다. 

그렇기에 LB 에서는 서버1이 health check 를 통과하지 못하였기에 **서버2에만 요청**을 보내는 것을 확인할 수 있다.

![5-11.png](/assets/img/ncloud-sourcepipeline/5-11.png)

192 번 요청부터는 server1, server2 를 다시 오가는 것을 볼 수 있는데 이는 서버1이 health check 를 통과하였기에 다시 요청을 보내는 것이다.

![5-12.png](/assets/img/ncloud-sourcepipeline/5-12.png)

이로써 LB(LoadBalancer) 를 구축하고 분배가 잘 되는지 server1 이 다운되었을 때 server2 로만 요청을 전달하는지 확인하였다.
모두 예상대로 동작하는 것 또한 확인하였다.



