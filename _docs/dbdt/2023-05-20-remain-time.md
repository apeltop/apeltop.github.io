---
title: 제출 마감 시간 구하기
layout: doc
subtitle: 제출 마감 시간 로직 작성 및 테스트 케이스 작성
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

## 만들고자 하는 것
`제출 마감 시간을 반환하고 싶다.`

대략적인 화면은 아래와 같다. 아래 이미지는 [tickcounter](https://www.tickcounter.com) 사이트에서 만들었다.

![5-1.png](/assets/img/dbdt/5-1.png)

## 로직 구상
### 주어지는 데이터
- 오늘 datetime
- 마감 datetime

### 반환해야하는 데이터 포멧

```json
{
  "h": 6,
  "m": 47,
  "s": 58
}
```

### 의사 코드
```python

def 현재_시간():
    return 현재_시간

def 오늘_마감_시간():
    return 오늘_자정_시간

def 시간_차이_구하기(현재_시각, 마감_시각):
    return 마감_시각 - 현재_시각
    
시간_차이_구하기(현재_시각(), 오늘_마감_시간()) # {h: 6, m: 47, s: 58}

```

## 코드 작성
### 메소드 추가
우선 현재 시간, 마감 시간을 util 폴더 안에 date.py 파일에 추가하였다.

우선 필요한 모듈을 import 한다.

```python
# util/date.py
from datetime import timezone, timedelta, datetime
```

#### 현재 시각 구하기
KST 는 UTC 에 +9 시간된 시간대이기 때문에 명시하여 timezone 으로 설정하였다.
datetime.now() 로 호출하면 서버에 설정된 timezone 을 따라 생성되기에 나중에 문제가 생길 여지도 있어보이기 때문이다.

```python
def get_current_kst():
    tz_kst = timezone(timedelta(hours=9))
    return datetime.now().astimezone(tz_kst)
```

#### 오늘 자정 시각 구하기
```python
def get_today_deadline_kst():
    return get_current_kst().replace(
        hour=0,
        minute=0,
        second=0,
        microsecond=0,
    ) + timedelta(days=1)
```    

#### 시간 차이 구하기
```python
def get_remain_time(current_time, deadline_time):
    total_s = (deadline_time - current_time).total_seconds()
    return {
        's': int(total_s % 60),
        'm': int((total_s / 60) % 60),
        'h': int(total_s / 60 / 60),
    }
```

### 테스트 케이스 작성하기
각 메소드는 간단하지만 테스트 케이스를 추가해야한다.
혹자는 저정도 메소드에 테스트 케이스를 추가하는 것은 오히려 불필요한 자원을 소모한다고 생각할 수 있겠다.
하지만 역설적으로 간단할수록 테스트를 꼼꼼히 해야한다. 간단하다는 것은 그만큼 많이 사용되는 코드이라는 뜻이기 때문이다.
즉 간단한 곳에서 문제가 생기면 발생하는 문제의 여파 역시 클 수 밖에 없다.

#### 빈 테스트 클래스 생성
```python
# util/test_date.py

class DateTest(TestCase):
    pass
```

#### 필요 모듈 import
```python
from datetime import timezone, timedelta, datetime
from unittest import TestCase

from util.date import get_current_kst, get_today_deadline_kst, get_remain_time
```

#### 현재 시각 구하기 테스트
검사한 항목
- timezone == +9 hours

```python
class DateTest(TestCase):
    def test_get_deadline_kst(self):
        kst = get_current_kst()
        self.assertEqual(kst.tzinfo, timezone(timedelta(hours=9)))
```

#### 오늘 자정 시각 구하기 테스트
검사한 항목
- 마감_시간.hour == 0
- 마감_시간.minute == 0
- 마감_시간.second == 0
- 마감_시간.microsecond == 0
- 마감_시간.tzinfo == +9 hours
- 현재_시각.day + 1 == 마감_시각.day

```python
class DateTest(TestCase):
    ...

    def test_get_deadline_kst(self):
        self.assertEqual(get_today_deadline_kst().hour, 0)
        self.assertEqual(get_today_deadline_kst().minute, 0)
        self.assertEqual(get_today_deadline_kst().second, 0)
        self.assertEqual(get_today_deadline_kst().microsecond, 0)

        self.assertEqual(get_today_deadline_kst().tzinfo, timezone(timedelta(hours=9)))
        self.assertEqual(get_current_kst().day + 1, get_today_deadline_kst().day)
```

#### 시간 차이 구하기 테스트
이번 테스트는 하나의 경우만 테스트 하는 것이 아니라 일, 시, 분, 초가 다른 경우로 테스트하였다.

```python
class DateTest(TestCase):
    ...
    def test_get_remain_time(self):
        remain = get_remain_time(
            datetime(2023, 1, 1, 0, 0, 0, 0, timezone(timedelta(hours=9))),
            datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9))),
        )

        self.assertEqual(remain['h'], 24)
        self.assertEqual(remain['m'], 0)
        self.assertEqual(remain['s'], 0)

        remain = get_remain_time(
            datetime(2023, 1, 1, 8, 0, 0, 0, timezone(timedelta(hours=9))),
            datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9))),
        )

        self.assertEqual(remain['h'], 16)
        self.assertEqual(remain['m'], 0)
        self.assertEqual(remain['s'], 0)

        remain = get_remain_time(
            datetime(2023, 1, 1, 8, 40, 0, 0, timezone(timedelta(hours=9))),
            datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9))),
        )

        self.assertEqual(remain['h'], 15)
        self.assertEqual(remain['m'], 20)
        self.assertEqual(remain['s'], 0)

        remain = get_remain_time(
            datetime(2023, 1, 1, 8, 40, 20, 0, timezone(timedelta(hours=9))),
            datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9))),
        )

        self.assertEqual(remain['h'], 15)
        self.assertEqual(remain['m'], 19)
        self.assertEqual(remain['s'], 40)


```

##### 리펙토링

아래의 테스트는 통과하고 논리적으로는 괜찮지만 중복 코드가 너무 많다. 개인적으로는 나열하고 줄이는 방법을 선호한다.
이 코드는 결국 아래 로직이 전부이고 인자와 결과값만 다를 뿐이다. 그래서 테스트 값과 결과값으로 나누어 볼 수 있다.

```python
# data
datetime(2023, 1, 1, 0, 0, 0, 0, timezone(timedelta(hours=9))),
 datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9)))

# expected
{'h': 24, 'm': 0, 's': 0}
```

이를 배열로 만들어서 테스트를 실행하려한다.
parameterized 를 사용한다.

```pip
pip install parameterized
```

데이터를 배열로 분리하니 테스트하는 로직이 깔끔해졌다.
이렇게 분리할 경우 데이터를 추가하기 쉽고 테스트 로직이 깔끔해진다는 장점이 있다. 
테스트 로직이 깔끔하지 않고 쭉 나열된 테스트라면 중간에 수정을 되어도 확인하기 어려워 테스트의 목적을 잃어버리기 쉽다.

```python
from parameterized import parameterized

...

@parameterized.expand([
        ((
                 datetime(2023, 1, 1, 0, 0, 0, 0, timezone(timedelta(hours=9))),
                 datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9))),
         ), {'h': 24, 'm': 0, 's': 0}),
        ((
                 datetime(2023, 1, 1, 8, 0, 0, 0, timezone(timedelta(hours=9))),
                 datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9))),
         ), {'h': 16, 'm': 0, 's': 0}),
        ((
                 datetime(2023, 1, 1, 8, 40, 0, 0, timezone(timedelta(hours=9))),
                 datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9))),
         ), {'h': 15, 'm': 20, 's': 0}),
        ((
                 datetime(2023, 1, 1, 8, 40, 20, 0, timezone(timedelta(hours=9))),
                 datetime(2023, 1, 2, 0, 0, 0, 0, timezone(timedelta(hours=9))),
         ), {'h': 15, 'm': 19, 's': 40}),
    ])
    def test_get_remain_time(self, test_input, expected):
        current_time, deadline = test_input

        remain = get_remain_time(
            current_time,
            deadline,
        )

        self.assertEqual(remain['h'], expected['h'])
        self.assertEqual(remain['m'], expected['m'])
        self.assertEqual(remain['s'], expected['s'])

```

오늘 추가된 로직과 파일은 아래와 같다.

![5-2.png](/assets/img/dbdt/5-2.png)


테스트를 실행한 결과이다.

![5-3.png](/assets/img/dbdt/5-3.png)
