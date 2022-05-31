---
title: 환경 설정 및 구성
layout: doc
subtitle: 환경 설정 후 구글 브라우저 띄우기
author: sunshine@ptokos.com
tags: selenium
---

프로젝트 선정 이유
---
---
저는 복권을 하지는 않지만 개발하면서 꽤 중요한 순간과 연관이 되어있습니다. 처음으로 코딩을 배우며 스스로 만들어 본 프로젝트가 복권 번호를 랜덤으로 발행하여 해당 등수를 나타내주는 것이였습니다. 그리고 객체지향을 배우며 왜 상속을 해야하는지.. 다형성이 왜 중요한지 도저히 이해할 수 없는 기간에 한국 복권과 미국의 복권(파워볼) 이 비슷해보이지만 번호의 범위, 등수 선정 등 세세한 규칙들이 다른 것을 보고 Lotto Class 를 KoLotto 와 EnLotto 클래스가 상속 받고 세세한 규칙이 다른 부분은 Lotto 클래스에서 abstract 로 해놓으면 되겠구나 등이 떠오르며 객체지향의 달콤함을 맛보았던 기억이 있습니다. 이런 경험 덕분인지 Selenium 을 가지고 어떤 것을 만들어볼까 했을 때 복권이 떠올랐습니다. 이 복권 구매 자동화를 가지고 좋은 정보들을 나눌 수 있을지 살펴볼 때 꽤나 만족스러운 환경이여서 선정하였습니다.


필자 프로젝트 환경 구성
---
```
언어 - python3.9
라이브러리 - selenium
브라우저 - chrome
```
>이번에 만들어볼 프로젝트는 다양한 라이브러리가 필요로 하지 않으니 conda 등을 사용하지 않겠습니다. 

## 1. 드라이버 다운로드
selenium 을 사용하기 위해서는 브라우저 드라이버가 필요합니다. 다운로드 홈페이지는 아래와 같습니다.

[Chrome](https://sites.google.com/chromium.org/driver/, "chrome driver link")

[Edge](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/, "edge driver link")

[firefox](https://github.com/mozilla/geckodriver/releases, "firefox driver link")

[Safari](https://webkit.org/blog/6900/webdriver-support-in-safari-10/, "safari driver link")

>이 글은 크롬을 사용합니다. 다른 브라우저를 사용하셔도 대부분의 selenium 사용 방법은 똑같지만 브라우저 option 설정 등이 좀 다르다고 볼 수 있습니다.


## 2. 브라우저 버전 확인
로컬 PC 의 브라우저 버전에 맞게 드라이버를 다운로드 받아야하기 때문에 [버전](chrome://settings/help)을 먼저 확인해야합니다.
![Alt text](/assets/img/lotto-automation/1-1.png)

## 3. 드라이버 프로젝트에 Import
저의 경우는 dirver 디렉토리에 다운로드 받은 드라이버를 넣었습니다.

## 4. selenium 설치
```
pip install selenium
```

## 5. Sample 코드 작성
```
from selenium import webdriver
from selenium.webdriver.chrome.service import Service

driver = webdriver.Chrome(service=Service('./driver/chromedriver_mac64_m1_97'))
driver.get('https://www.google.com')
```

## 6. 실행 확인
실행 시 아래와 같이 구글 창이 나타나게됩니다.
![Alt text](/assets/img/lotto-automation/1-2.png)





