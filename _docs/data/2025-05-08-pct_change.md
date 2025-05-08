---
title: pct_change
layout: doc
subtitle: pandas pct_change 소개
author: sunshine@ptokos.com
tags: [DATA, pandas]
comments: true
---

### pct_change 소개
pct_change 는 **"전일 대비 수익률 (Percentage Change)"** 를 자동으로 계산하는 Pandas의 내장 함수입니다.
이 함수는 현재 값이 이전 값에 비해 얼마나 변화했는지를 백분율로 나타내는 데 사용됩니다.

### 수식: pct_change() 계산 방법
pct_change()의 기본 수식은 다음과 같습니다:

$\text{pct_change} = \frac{\text{현재 값} - \text{이전 값}}{\text{이전 값}}$

현재 값: 현재 행의 값

이전 값: 바로 이전 행의 값

결과: 변화율을 백분율로 계산 (소수점으로 표현)

### 사용법1 - 기본
```python
import pandas as pd

data = {'price': [100, 105, 102, 108, 110]}
df = pd.DataFrame(data)
df['return'] = df['price'].pct_change()
print(df)
```

### 출력 결과
```
   price   return
0    100      NaN
1    105  0.05000  # (105 - 100) / 100 = 0.05 (5%)
2    102 -0.02857  # (102 - 105) / 105 = -0.02857 (-2.857%)
3    108  0.05882  # (108 - 102) / 102 = 0.05882 (5.882%)
4    110  0.01852  # (110 - 108) / 108 = 0.01852 (1.852%)
```

### 사용법 2 - 기간 설정
```python
df['5day_return'] = df['price'].pct_change(periods=5)

print(df)
```

> periods = 5 는 **5일 전**의 값과 비교하여 변화율을 계산합니다. 만약 5일 전의 값이 없다면 NaN으로 표시됩니다.

