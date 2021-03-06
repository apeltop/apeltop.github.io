---
title: 리펙토링
layout: doc
subtitle: 나열되어있는 기능을 wrapping 함으로 어떠한 순서로 작동되고 있는지 파악할 수 있도록하기
author: sunshine@ptokos.com
tags: selenium
---

리펙토링이라고 하기에는 사실 거창하기는 하지만 따라하시기 편하도록 한 개의 파일 안에서 코드 작성을 이어가고 있었습니다. 
코드를 작성할 때는 당연히 이해가 가지만 조금만 시간이 지나도 이게 어떤 기능을 의미하는지 잊어버릴 때가 있습니다. 
이를 방지하기 위해서는 함수로 만들어 함수 이름으로 유추하게 하던지 아니면 주석을 달면 됩니다. 
때에 따라서 정답은 달리지지만 오늘은 함수로 기능을 묶어 맨 하단에 함수 이름을 통해 어떠한 기능을 수행하는지 알 수 있게되면 성공입니다.

아래 코드는 클릭을 하기는 하는데 저 xpath 가 무엇을 의미하는지 알기가 쉽지 않습니다.
```
click(driver.find_element(By.XPATH, '/html/body/div[1]/header/div[2]/div[2]/form/div/ul/li[1]/a'))
```

함수로 묶어 아래로 표현하면 알 수가 있게 됩니다.
```
def move_to_login_page():
    click(driver.find_element(By.XPATH, '/html/body/div[1]/header/div[2]/div[2]/form/div/ul/li[1]/a'))
```

로그인 관련 부분을 수정해보겠습니다.
기존의 코드는 아래와 같이 나열되어 있습니다. 
이 경우 사실 어떤 행위를 하는지 나중에 봐도 알 수는 있겠지만 3줄이 연달아 있는 것이 불편합니다. 
이를 함수로 묶어 두고 사용하면 훨씬 코드를 작성할 때 가독성이 좋게 될 것 입니다. 
팝업 닫는 코드까지 추가하였습니다. 이는 로그인 단계에서 처리하는 것이 맞다고 생각하기 때문입니다. 

#### 기존 코드
```
send_keys(driver.find_element(By.ID, 'userId'), os.environ.get('ID'))
send_keys(driver.find_element(By.NAME, 'password'), os.environ.get('PW'))
send_keys(driver.find_element(By.NAME, 'password'), Keys.ENTER)
```

#### 변경 코드
```
class SendKeyInfo:
    def __init__(self, by, value, text):
        self.by = by
        self.value = value
        self.text = text

def login(id_info: SendKeyInfo, pw_info: SendKeyInfo):
    send_keys(driver.find_element(id_info.by, id_info.value), id_info.text)
    send_keys(driver.find_element(pw_info.by, pw_info.value), pw_info.text)
    send_keys(driver.find_element(pw_info.by, pw_info.value), Keys.ENTER)

    sleep(1)
    close_windows_if_not_main_url('common.do?method=main')

login(id_info=SendKeyInfo(By.ID, 'userId', os.environ.get('ID')),
        pw_info=SendKeyInfo(By.NAME, 'password', os.environ.get('PW')))
```


## 수정된 하단 코드
코드 변경한 부분을 모두 서술하지는 않겠습니다. 각 기능을 함수로 묶으면 아래와 같이 코드 작성을 할 수 있습니다.  나중에 코드를 보더라도 어떠한 단계를 나누어서 실행되고 있는지 알 수 있게됩니다.
```
move_to_login_page() # 로그인 페이지로 이동

login(id_info=SendKeyInfo(By.ID, 'userId', os.environ.get('ID')),
pw_info=SendKeyInfo(By.NAME, 'password', os.environ.get('PW'))) # 로그인 진행

open_lotto_buy_popup() # 로또 구매를 위한 팝업 열기

switch_to_lotto_iframe() # 로또 게임 iframe 으로 이동

lotto_num_clicks() # 무작위로 발행된 번호를 가지고 해당하는 숫자 버튼 클릭

buy_lotto() # 구매 진행

save_screenshot() # 구매내역 스크린샷

browser_quit() # 브라우저 종료
```

각 단계를 더 합칠 수 있는 부분이 있지만 이번 경우에는 세분화해서 나열하는 것이 더욱 이해가 좋을 것 같다고 보여집니다. 
이처럼 코드를 나열하기보다 함수로 묶어 가독성을 높이는 간단한 리펙토링을 해보았습니다.


## 전체 코드
```
from datetime import datetime
import random
from time import sleep

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By

from dotenv import load_dotenv
import os

from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

load_dotenv()

LOTTO_MIN_NUM = 1
LOTTO_MAX_NUM = 45


def create_driver():
    return webdriver.Chrome(service=Service('./driver/chromedriver_mac64_m1_97'))


driver = create_driver()
driver.get('https://dhlottery.co.kr/common.do?method=main')


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


def send_keys(element, text):
    element.send_keys(text)


def click(element):
    element.click()


def execute_script(script):
    driver.execute_script(script)


def switch_to_last_opened_window():
    driver.switch_to.window(driver.window_handles[-1])


def switch_to_iframe(iframe):
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.TAG_NAME, "iframe"))
    )

    driver.switch_to.frame(iframe)


def pick_numbers():
    picks = set([])
    while 6 != len(picks):
        n = random.randint(LOTTO_MIN_NUM, LOTTO_MAX_NUM)
        picks.add(n)

    return picks


def save_screenshot():
    driver.save_screenshot(f'{datetime.now().strftime("%Y%m%d_%H%M%S")}.png')


class SendKeyInfo:
    def __init__(self, by, value, text):
        self.by = by
        self.value = value
        self.text = text


def login(id_info: SendKeyInfo, pw_info: SendKeyInfo):
    send_keys(driver.find_element(id_info.by, id_info.value), id_info.text)
    send_keys(driver.find_element(pw_info.by, pw_info.value), pw_info.text)
    send_keys(driver.find_element(pw_info.by, pw_info.value), Keys.ENTER)

    sleep(1)
    close_windows_if_not_main_url('common.do?method=main')


def move_to_login_page():
    close_windows_if_not_main_url('common.do?method=main')
    click(driver.find_element(By.XPATH, '/html/body/div[1]/header/div[2]/div[2]/form/div/ul/li[1]/a'))


def open_lotto_buy_popup():
    execute_script('goLottoBuy(2);')
    sleep(1)
    switch_to_last_opened_window()


def switch_to_lotto_iframe():
    switch_to_iframe(driver.find_elements(By.TAG_NAME, 'iframe')[0])


def lotto_num_clicks():
    for num in pick_numbers():
        execute_script(f'$("#check645num{num}").prop("checked", true);')


def buy_lotto():
    click(driver.find_element(By.ID, 'btnSelectNum'))

    click(driver.find_element(By.ID, 'btnBuy'))
    execute_script('closepopupLayerConfirm(true);')
    sleep(2)


def browser_quit():
    driver.quit()


move_to_login_page()  # 로그인 페이지로 이동

login(id_info=SendKeyInfo(By.ID, 'userId', os.environ.get('ID')),
      pw_info=SendKeyInfo(By.NAME, 'password', os.environ.get('PW')))  # 로그인 진행

open_lotto_buy_popup()  # 로또 구매를 위한 팝업 열기

switch_to_lotto_iframe()  # 로또 게임 iframe 으로 이동

lotto_num_clicks()  # 무작위로 발행된 번호를 가지고 해당하는 숫자 버튼 클릭

buy_lotto()  # 구매 진행

save_screenshot()  # 구매내역 스크린샷

browser_quit()  # 브라우저 종료
```
