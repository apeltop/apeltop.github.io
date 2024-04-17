---
title: GCP Oracle DB 를 Compute Engine 으로 연결하기 (VM 사용) - 2
author: sunshine@ptokos.com
categories: [gcp]
---

## 목표
> Compute Engine 에 Oracle DB 를 설치하고 연결하기. 

## 구축하기
### Compute Engine 생성
Compute Engine 은 Debian 과 e2-medium 인스턴스 타입을 사용했다.
e2-medium 인스턴스 타입은 1 vCPU, 4GB RAM 을 사용한다.

_중요한 것은 1521 포트 inbound 를 열어주어야한다._

![1.png](/assets/img/gcp/gcp-oracle-compute-engine-2-1.png)

### Oracle DB 설치
#### 회원가입
[oracle](https://www.oracle.com) 공식 홈페이지에 회원가입을 한다.
registry 를 oracle 에서 제공하기에 가입을 해야한다.

#### Docker 설치
SSH 로 먼저 접속을 해야한다. 필자는 Compute Engine 화면에 보이는 SSH 버튼을 통해 접속했다.

도커 공식 문서인 [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/) 에 자세히 나온다.
명령어만 적어놓으면 다음과 같다.
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Oracle DB Image 다운로드 & 실행
Oracle 에 가입한 정보를 사용해서 인증한다.

```bash
docker login container-registry.oracle.com 
```

Oracle DB Image 를 다운로드한다.
```bash
docker pull container-registry.oracle.com/database/free:latest
```

-dit 옵션은 **detach**, **interactive**, **tty** 옵션을 의미한다.
이 옵션을 사용하면 컨테이너가 백그라운드에서 실행되고, 터미널을 사용할 수 있다.

```bash
docker run -dit -p 1521:1521 --name oracle_db container-registry.oracle.com/database/free:latest
```

![2.png](/assets/img/gcp/gcp-oracle-compute-engine-2-2.png)

#### Oracle DB 환경 설정
아래와 같이 명령어를 입력한다.

```bash
docker exec -it oracle_db bash -c "source /home/oracle/.bashrc; sqlplus /nolog"
```

아래 화면과 같이 SQL> 이 나오면 성공이다.

![3.png](/assets/img/gcp/gcp-oracle-compute-engine-2-3.png)

sys 계정으로 접속한다.

```bash
connect sys as sysdba;
```

"_ORACLE_SCRIPT"=true" 는 11g 문법을 사용할 수 있도록 한다. 이것을 사용하지 않으면 USER 생성할 때 C## 이라는 접두어를 사용해야한다.

```sql
alter session set "_ORACLE_SCRIPT"=true;
```

아래 USER_NAME 과 PASSWORD 를 바꾸어서 USER 를 생성하면 된다.
또한 권한은 여기서는 모든 권한을 주도록했는데 이는 상황에 따라 달리 주면 된다.

```sql
create user USER_NAME identified by PASSWORD;
GRANT ALL PRIVILEGES TO USER_NAME;
```

설정은 모두 끝났다.

혹 SID 를 알아야하는 경우 아래 명령어를 입력하면 된다.

```sql
SELECT instance FROM V$INSTANCE;
```

![4.png](/assets/img/gcp/gcp-oracle-compute-engine-2-4.png)











