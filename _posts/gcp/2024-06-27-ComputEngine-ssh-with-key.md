---
title: Compute Engine SSH Key 를 이용한 접속
author: sunshine@ptokos.com
categories: [GCP, ComputeEngine]
---

## 목표
> Compute Engine 에 SSH Key 를 이용하여 접속하는 방법을 알아보자.

## 상황
Compute Engine 인스턴스를 생성하면 SSH 를 클릭하면 브라우저로 접속할 수 있다.
그러나 브라우저 환경 말고도 접속할 수 있는 환경이 필요하기도 하다. 또한 파일 업로드 또한 브라우저를 통해서 업로드할 수 있도록 지원하지만 **매우 느리다**.
간단한 파일을 가능하지만 용량이 큰 파일을 업로드할 때는 불편하다. 그러기에 **SSH Key** 를 이용하여 접속하는 방법을 알아보자.

![compute-engine-ssh-key-1.png](/assets/img/gcp/computeEngine-ssh-with-key-1.png)

## 구축하기

### Compute Engine 생성
서버 스펙만 낮추고 생성하였다. 

![compute-engine-ssh-key-2.png](/assets/img/gcp/computeEngine-ssh-with-key-2.png)

일정 시간을 기다리면 인스턴스가 **Running** 상태로 변경된다.

![compute-engine-ssh-key-1.png](/assets/img/gcp/computeEngine-ssh-with-key-1.png)

### SSH 접속할 수 있는지 확인
ssh key 를 통한 접속 전에 **22번** 포트로 접속할 수 있는 상태인지 확인하는 것이 좋다.
그래야 미리 문제가 있는지 확인할 수 있고 문제가 있을 경우 비교적 명쾌히 해결할 수 있기 때문이다.

맥 환경이라 nc 명령어를 사용하여 포트가 열려있는지 확인하였다. 만약 결과가 바로 안 나오고 시간이 지연되면 포트가 열려있지 않은 것이다.

```bash
nc -zv 34.47.101.130 22 
```

![compute-engine-ssh-key-3.png](/assets/img/gcp/computeEngine-ssh-with-key-3.png)

왜 아무런 설정을 하지 않았는데 22번 포트를 사용할 수 있을까?

Compute Engine 인스터스를 생성할 때 Network 를 **default** 로 설정하였다. 
해당 내용은 **VPC network** 에서 확인할 수 있다.

![compute-engine-ssh-key-6.png](/assets/img/gcp/computeEngine-ssh-with-key-6.png)

default 를 클릭해서 **FIREWALLS** 탭을 보면 vpc-firewall-rules 가 있다.
이를 펼쳐보면 각 규칙이 보이는데 이중에서 **default-allow-ssh** 가 있다.
이 규칙을 통해서 22번 포트가 열려있는 것이다. 
만약 이 규칙이 적용되지 않기를 원한다면 새로운 VPC 를 만들기 또는 규칙의 우선순위를 조정 및 새로운 규칙 생성을 통해 설정할 수 있다.

![compute-engine-ssh-key-4.png](/assets/img/gcp/computeEngine-ssh-with-key-4.png)

참고 [Pre-populated rules in the default network](https://cloud.google.com/firewall/docs/firewalls#more_rules_default_vpc)

### SSH KEY 생성
SSH Key 를 생성하기 위해서는 **ssh-keygen** 명령어를 사용한다.
passphrase 는 입력하지 않고 엔터로 넘어가도록 하였다.

```bash
ssh-keygen -t rsa -C "apeltop" -f ~/.ssh/ssh_test_id_rsa
```

-C 는 주석을 달 수 있도록 하는데 이 값이 **username** 이 된다.

![compute-engine-ssh-key-7.png](/assets/img/gcp/computeEngine-ssh-with-key-7.png)

### SSH KEY 등록
인스턴스 Edit 화면에서 SSH Keys 섹션에서 **+Add Item** 을 클릭한다.
생성된 **rsa.pub** 파일을 복사하여 Compute Engine 인스턴스에 등록한다.

![compute-engine-ssh-key-8.png](/assets/img/gcp/computeEngine-ssh-with-key-8.png)

### SSH 접속
이제 접속을 해볼 것이다. 그럼 apeltop 이라는 사용자로 접속이 되는 것을 볼 수 있다.

```bash
ssh -i ~/.ssh/ssh_test_id_rsa apeltop@34.47.101.130
```

![compute-engine-ssh-key-9.png](/assets/img/gcp/computeEngine-ssh-with-key-9.png)














