---
title: SendGrid 를 통해 이메일 보내기
layout: doc
subtitle: SendGrid 템플릿을 사용한 이메일 보내기 
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`SendGrid` 를 통해 이메일을 보내고 싶다. 템플릿 또한 사용하여 보내고자한다.


## GCP SendGrid 연동
### SendGrid 구독
GCP Console 에서 Marketplace 를 검색하여 SendGrid 를 설치한다.

![19-1.png](/assets/img/dbdt/19-1.png)

검색 후 3개의 결과가 나온다. 이 중 2번째인 SendGrid Email API 를 선택한다.

![19-2.png](/assets/img/dbdt/19-2.png)

그럼 아래와 같은 화면이 나오게 된다. 여기서 아래로 스크롤을 하거나 _VIEW ALL PLANS_ 를 클릭해준다.

![19-3.png](/assets/img/dbdt/19-3.png)

여러 플랜이 나오는데 이 중 Free 를 선택해준다.
**Free Plan** 의 경우 달마다 12,000 건을 발송할 수 있다. 

![19-4.png](/assets/img/dbdt/19-4.png)

Free Plan 을 클릭하게되면 구독 화면이 나오게된다.

![19-5.png](/assets/img/dbdt/19-5.png)

결제 계정을 선택하고 몇 개의 약관에 동의하면 구독할 수 있다.

![19-6.png](/assets/img/dbdt/19-6.png)

SUBSCRIBE 를 클릭하면 아래와 같이 구독 신청이 완료되었다고 알려준다. 그 후 _SING UP WITH SENDGRID_ 를 클릭해준다. 

![19-7.png](/assets/img/dbdt/19-7.png)

그럼 별다른 조취를 취할 것 없이 가입이 되었다고 알려준다.

![19-8.png](/assets/img/dbdt/19-8.png)

안내 사항처럼 다시 GCP 화면으로 새로고침을 진행해주면 **VIEW ALL PLANS** 가 아니라 **MANAGE ON PROVIDER** 가 나오게 된다.

![19-9.png](/assets/img/dbdt/19-9.png)

MANAGE ON PROVIDER 를 클릭하게 되면 SendGrid 홈페이지로 이동하게 되며 기초 정보를 입력하게 한다. 

![19-10.png](/assets/img/dbdt/19-10.png)

가입 후 Settings > API Keys 로 이동한다.

![19-11.png](/assets/img/dbdt/19-11.png)

Full Access 가 아닌 Restricted Access 를 선택한다. 그 후 Mail Send 만 권한을 부여한다.

![19-12.png](/assets/img/dbdt/19-12.png)

그럼 API KEY 가 생성되는데 지금 이후로는 다시 확인할 수 없으니 어딘가에 저장을 해놓는다.

![19-13.png](/assets/img/dbdt/19-13.png)

Email API > Dynamic Templates 로 이동한다. 그 후 _Create a Dynamic Template_ 를 클릭해주고 템플릿을 생성해준다.

![19-14.png](/assets/img/dbdt/19-14.png)

그럼 비어있는 템플릿이 하나가 생성이 된다. 여기서 **Add Version** 을 클릭해준다. 

![19-15.png](/assets/img/dbdt/19-15.png)

그럼 여러가지 상황에 대한 이메일 템플릿이 있다. 필자는 회원가입 시 이메일 인증을 하고자하는 상황이기 때문에 Verification Email Template 를 선택했다.

![19-16.png](/assets/img/dbdt/19-16.png)

Template 를 선택하면 Design Editor 와 Code Editor 둘 중에 선택하라고 나온다.
지금은 템플릿을 사용한 이메일 전송 여부만 확인하고 싶기에 우선 Design Editor 를 선택해준다. 

![19-17.png](/assets/img/dbdt/19-17.png)

그럼 화면을 미리 볼 수 있고 수정도 할 수 있다. 수정은 나중에 하기로 하고 우선 저장 후 나갔다.

![19-18.png](/assets/img/dbdt/19-18.png)

템플릿 안에 버전이 하나 생성된 것을 확인할 수 있다.

![19-19.png](/assets/img/dbdt/19-19.png)

설정을 하고 있다보면 자꾸 상단에 알림이 나온다. 읽어보니 필요해서 Create a sender identity 를 클릭하였다.
만약 보이지 않는다면 계정 아이콘 클릭 후 Setup Guide 에 들어가면 Create a sender identity 영역이 보일 것이다.. 

![19-20.png](/assets/img/dbdt/19-20.png)

Sender 정보를 입력해준다.

![19-21.png](/assets/img/dbdt/19-21.png)

그럼 인증 이메일이 발송되는데 이메일을 확인하고 인증을 완료해준다.

![19-22.png](/assets/img/dbdt/19-22.png)

Single Sender Verification 에 정보가 등록된 것을 확인할 수 있다.

![19-23.png](/assets/img/dbdt/19-23.png)

이제 SendGrid 사용을 위한 준비가 끝났다. 사용만 남았다.

## 이메일 전송 코드 작성
### 인증 Key 준비(.env)
```bash
SENDGRID_API_KEY=SG.~.~
```

### 텍스트만 전송
```python
# using SendGrid's Python Library
# https://github.com/sendgrid/sendgrid-python
import os
from sendgrid import SendGridAPIClient
from sendgrid.helpers.mail import Mail

from dotenv import load_dotenv

load_dotenv()

message = Mail(
    from_email='communication@ptokos.com',
    to_emails='sunshine@ptokos.com',
    subject='Sending with Twilio SendGrid is Fun',
    html_content='<strong>and easy to do anywhere, even with Python</strong>',
)


try:
    sg = SendGridAPIClient(os.getenv('SENDGRID_API_KEY'))
    response = sg.send(message)
    print(response.status_code)
    print(response.body)
    print(response.headers)
except Exception as e:
    print(e.body)
```

이메일을 확인해보면 성공적으로 전송된 것을 확인할 수 있다.

![19-24.png](/assets/img/dbdt/19-24.png)

### 템플릿을 이용한 전송

템플릿을 사용해야하다보니 template_id   를 설정해줘야한다.
template_id 는 Template 를 들어가면 d- 로 시작하는 아이디를 확인할 수 있다. 해당 아이디를 사용하면 된다.

```python
# using SendGrid's Python Library
# https://github.com/sendgrid/sendgrid-python
import os
from sendgrid import SendGridAPIClient
from sendgrid.helpers.mail import Mail

from dotenv import load_dotenv

load_dotenv()

message = Mail(
    from_email='communication@ptokos.com',
    to_emails='sunshine@ptokos.com',
    subject='Sending with Twilio SendGrid is Fun',
)

message.template_id = 'd-38...'
try:
    sg = SendGridAPIClient(os.getenv('SENDGRID_API_KEY'))
    response = sg.send(message)
    print(response.status_code)
    print(response.body)
    print(response.headers)
except Exception as e:
    print(e.body)
```

그럼 성공적으로 템플릿으로 보내진 것을 확인할 수 있다.

![19-25.png](/assets/img/dbdt/19-25.png)


## 데이터 연동해서 이메일 전송하기
앞 단계를 진행했다면 이메일이 전송되는 것까지 모두 확인하였을 것이다.
이메일마다 사용자 이름, 인증 URL 등 다른 정보를 주어야한다. 이를 진행해보려한다.

### 새로운 Template 만들기
앞선 부분에서 만든 Template 는 Deisgn Editor 를 사용했다. Code Editor 를 사용해야 템플릿에 데이터를 넣을 수 있다.

템플릿 선택까지 동일하게 진행하고 Code Editor 를 선택해준다. 그러면 아래와 같이 코드가 보일 것이다.

상단에 **TEST DATA** 를 클릭해서 Json 형식으로 데이터를 입력해준다.

![19-27.png](/assets/img/dbdt/19-27.png)

다시 **CODE** 를 클릭해 Jane 부분을 찾아 {{name}} 으로 치환하였다. {{ }} 으로 감싸면 나중에 데이터를 사용할 수 있도록 선언하는 부분이다.

![19-26.png](/assets/img/dbdt/19-26.png)

Template 저장 후 나가서 확인해보면 기존의 버전이 ACTIVE 인 것을 확인할 수 있다.  

![19-28.png](/assets/img/dbdt/19-28.png)

새롭게 변경한 템플릿을 ACTIVE 로 변경해준다.

![19-29.png](/assets/img/dbdt/19-29.png)

### 코드

dynamic_template_data 에 dict 형식으로 데이터를 입력해주면 된다.

```python
# using SendGrid's Python Library
# https://github.com/sendgrid/sendgrid-python
import os
from sendgrid import SendGridAPIClient
from sendgrid.helpers.mail import Mail

from dotenv import load_dotenv

load_dotenv()

message = Mail(
    from_email='communication@ptokos.com',
    to_emails='sunshine@ptokos.com',
    subject='Sending with Twilio SendGrid is Fun'
)
message.template_id = 'd-38...'
message.dynamic_template_data={
    'name': 'sunshine'
}
try:
    sg = SendGridAPIClient(os.getenv('SENDGRID_API_KEY'))
    response = sg.send(message)
    print(response.status_code)
    print(response.body)
    print(response.headers)
except Exception as e:
    print(e.body)
```


### 메일 확인
이메일에 Jane 이 아닌 sunshine 으로 온 것을 확인하였다.

![19-30.png](/assets/img/dbdt/19-30.png)


