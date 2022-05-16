---
title: 팝업 닫기
layout: doc
subtitle: 필요한 URL 을 제외한 팝업 닫기
author: sunshine@ptokos.com
tags: selenium
---

이번 글에서는 팝업 닫기를 해보려합니다. 팝업 닫기는 상황에 따라 여러 방법이 가능합니다.
1. 메인 URL 을 가지고 메인 페이지만 남기기
2. 팝업 URL 을 가지고 메인 페이지만 남기기

위 1, 2 번 뿐만 아니라 더 많은 접근 방법이 가능하지만 이번 글에서는 1 번에 해당하는 방식을 사용해보려 합니다.

[동행복권 홈페이지](https://dhlottery.co.kr/common.do?method=main) 들어가보면 메인페이지와 팝업 창이 나타납니다. 
이 팝업은 일시적이겠지만 꼭 팝업을 다뤄야하기 때문에 바로 다루어 보려고합니다. 
설명절 관련 팝업이기 때문에 대부분의 경우는 팝업이 없을 것이라고 생각이 듭니다. 
그런 경우는 어렵지 않으니 가볍게 아래 코드를 보고 넘어가도 될 것 같습니다.

![Alt text](/assets/img/lotto-automation/2-1.png)


## 1. 메인, 팝업 페이지의 핸들러 확인하기
핸들러는 각 페이지별로 고유한 ID 를 의미한다고 볼 수 있습니다. 
페이지별 ID 를 가지고 화면 전환 등을 할 수 있습니다. 
설명이 와닿지 않을 수 있지만 오늘 예제와 나중에 다룰 부분에 다루기 때문에 금방 개념을 알 수 있을 것입니다.


## 2. 핸들러 목록 출력하기
```
def get_main_handler():
    for window_handle in driver.window_handles:
        driver.switch_to.window(window_handle)

        print(f'현재 핸들러 name : {driver.current_window_handle}')
        print(f'현재 url :  {driver.current_url}')
        print('')

get_main_handler()
```

위 코드를 실행시키면 아래와 같이 출력이 됩니다.
![Alt text](/assets/img/lotto-automation/2-2.png)

>핸들러 name 은 계속 달라집니다.

## 3. 메인 페이지가 아닌 창 닫기
URL 에 common.do?method=main 이 포함되어있지 않으면 닫도록 하였습니다.
driver.switch_to.window(window_handle) 가 필요한 이유는 driver 는 하나의 윈도우를 가리키고 있기 때문에 팝업 윈도우를 가리키도록 변경하고 창을 닫도록 하는 것입니다.


```
def get_main_handle(main_url):
    for window_handle in driver.window_handles:
        driver.switch_to.window(window_handle)

        print(f'현재 핸들러 name : {driver.current_window_handle}')
        print(f'현재 url :  {driver.current_url}')
        print('')

        if main_url in driver.current_url:
            return window_handle
    raise '메인 핸들러 찾지 못함'


def close_windows_if_not_main_url(main_url):
    main_handle = get_main_handle(main_url)

    for window_handle in driver.window_handles:
        if main_handle != window_handle:
            driver.switch_to.window(window_handle)
            driver.close()

    driver.switch_to.window(main_handle)


close_windows_if_not_main_url('common.do?method=main')
  
```


간략하게 팝업 컨트롤을 보았습니다. 
메소드를 분리하고 파일을 분리해야 맞지만 더욱 간편한 이해를 돕기 위해 하나의 파일 안에서 작성하고 있습니다. 
제가 예제로 적는 코드는 실무에서 쓰기 좋은 코드라기 보다는 이해가 편한 코드라고 봐주시면 좋을 것 같습니다.
