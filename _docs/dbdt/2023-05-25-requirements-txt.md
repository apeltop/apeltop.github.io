---
title: requirements.txt 생성하기
layout: doc
subtitle: pip 리스트가 담겨있는 requirements.txt 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`pip 리스트가 담겨있는 requirements.txt 생성`

## requirements.txt 이란
requirements.txt 는 pip 리스트가 담겨있는 파일이다.
서버를 구축할 때 어떤 패키지를 사용하는지 알 수 있도록 requirements.txt 를 생성한다.

## requirements.txt 생성하기
```bash
pip freeze > requirements.txt
```

## requirements.txt 내용
```
anyio==3.6.2
click==8.1.3
exceptiongroup==1.1.1
Faker==18.9.0
fastapi==0.95.2
h11==0.14.0
httptools==0.5.0
idna==3.4
iniconfig==2.0.0
packaging==23.1
parameterized==0.9.0
pluggy==1.0.0
pydantic==1.10.7
pytest==7.3.1
python-dateutil==2.8.2
python-dotenv==1.0.0
PyYAML==6.0
six==1.16.0
sniffio==1.3.0
starlette==0.27.0
tomli==2.0.1
typing_extensions==4.5.0
uvicorn==0.22.0
uvloop==0.17.0
watchfiles==0.19.0
websockets==11.0.3
```

## requirements.txt 설치하기
requirements.txt 를 가지고 pip 를 설치하고 싶다면 아래 명령어를 사용한다.
```bash
pip install -r requirements.txt
```
