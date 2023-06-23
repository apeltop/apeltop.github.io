---
title: eGov 애플리케이션 백그라운드 실행
layout: doc
subtitle: SSH 세션이 끊겨도 서버는 실행되도록 하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 목표

`SSH 이 끊겨도 서버가 실행되도록 한다.`

## 현재 상황

현재는 SSH 세션이 끊기면 서버도 같이 종료되는 상태이다. 이를 해결하기 위해 서버를 백그라운드로 실행시키는 방법을 알아보자.

SSH 세션이 끊기면 서버가 종료되는 이유와 백그라운드 실행하는 간단한 예제를 먼저 살펴보려한다.

### Foreground

아래 구문은 Foreground 로 실행되는 구문이다. 즉 Ctrl + C 를 누르면 프로그램 종료와 동시에 서버도 종료가 된다.

```bash
java -jar sample_webapp.jar
```

### Background

#### &

구문 뒤에 & 를 붙이면 백그라운드 실행이 된다. & 을 붙이고 세션을 종료해도 서버가 종료되지 않는다.
과거 OS 에서는 & 을 붙이고 세션을 종료하면 서버도 종료가 되었는데 현재는 & 를 붙이어도 세션이 끊어져도 유지가 되도록하는게 기본 값이기 때문이다.
혹 과거 OS 이거나 불안함이 있다면 아래 소개할 nohup 을 사용하는 것을 권장한다.

```bash
java -jar sample_webapp.jar &
```

#### nohup

& 실행 구문 앞에 nohup 을 붙이는게 전부이다.
nohup 은 no hang up 의 약자로 세션이 끊겨도 유지가 되도록 하는 명령어이다.
또한 nohup 은 데몬 프로세스로 실행이 되기 때문에 프로세스가 종료되어도 서버가 종료되지 않는다.

```
nohup java -jar sample_webapp.jar &
```

nohup 의 출력과 관련해서는 [쉽게 설명한 nohup 과 &(백그라운드) 명령어 사용법](https://joonyon.tistory.com/entry/쉽게-설명한-nohup-과-백그라운드-명령어-사용법) 을
참고해주길 바란다.

## Shell Script 작성

### startup.sh

아래 구문을 실행하면 sample_webapp.jar 가 백그라운드로 실행된다.

```bash
nohup java -jar /root/sampleapp/sample_webapp.jar &
```

### shutdown.sh

sample_webapp.jar 로 실행되고 있는 프로세스의 pid 를 찾아 kill 하는 구문이다.

```bash
pid=`ps -ef | grep sample_webapp.jar | grep -v grep | awk '{print $2}'`
echo "Killing process $pid"
kill -9 $pid
```

grep -v grep 을 추가한 이유는 grep sample_webapp.jar 을 실행하면 자기 자신의 프로세스도 포함되기 때문이다.

```bash
(base) apeltop% ps -ef | grep sample_webapp.jar
  501 44013     1   0 10:14PM ttys008    0:09.25 /usr/bin/java -jar ../target/sample_webapp.jar
  501 44027 37450   0 10:14PM ttys008    0:00.00 grep sample_webapp.jar
```

```bash
(base) apeltopscript % ps -ef | grep sample_webapp.jar | grep -v grep 
  501 44013     1   0 10:14PM ttys008    0:09.58 /usr/bin/java -jar ../target/sample_webapp.jar
```

## 배포

startup.sh 와 shutdown.sh 는 server1, server2 에 각각 업로드해준다.

server1 구축을 먼저 해놓고 해당 이미지를 만들어 server2 를 만들었으면 더 편하지만 서버 이미지를 생성하는 예제에서 쉘 스크립트 작성 내용이 없어야 더 명확한 글이 될 것 같아 나누어서 진행하였다.

![4-1.png](/assets/img/ncloud-sourcepipeline/4-1.png)

startup.sh 를 실행시키면 권한이 없다고 나온다.

```bash
[root@server1 sampleapp]# ./startup.sh
-bash: ./startup.sh: Permission denied
```

startup.sh 와 shutdown.sh 에 실행 권한을 부여해준다.

```bash
[root@server1 sampleapp]# chmod 755 startup.sh
[root@server1 sampleapp]# chmod 755 shutdown.sh
```

startup.sh 를 실행해서 공인 IP 로 접근하면 실행이 된 것을 확인할 수 있으며 shutdown.sh 를 실행 시 `Killing process 8450` 와 같은 출력이 나오면서 서버가 종료된 것을 확인할
수 있다.

이로써 백그라운드로 서버 운영과 종료 할 수 있는 Shell Script 를 작성하였다. 



