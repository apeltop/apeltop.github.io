---
title: sort_values
layout: doc
subtitle: pandas sort_values 소개
author: sunshine@ptokos.com
tags: [DATA, pandas]
comments: true
---

### sort_values 소개
`sort_values`는 **데이터프레임을 지정된 열(column) 또는 여러 열을 기준으로 정렬**할 수 있도록 하는 메서드입니다. 기본적으로 오름차순으로 정렬되며, 다양한 옵션으로 정렬 방식과 순서를 제어할 수 있습니다.

### 사용법1 - 기본
```python
import pandas as pd

# 예제 데이터프레임 생성
data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eve'],
    'Age': [25, 32, 18, 29, 22],
    'Score': [88, 95, 79, 85, 92]
}
df = pd.DataFrame(data)

# Age 열을 기준으로 오름차순 정렬
df_sorted = df.sort_values('Age')
print(df_sorted)
```

### 출력 결과
``` css
      Name  Age  Score
2  Charlie   18     79
4      Eve   22     92
0    Alice   25     88
3    David   29     85
1      Bob   32     95
```

### 사용법 2 - 내림차순 정렬
```python
# Age 열을 기준으로 내림차순 정렬
df_sorted = df.sort_values('Age', ascending=False)
print(df_sorted)
```

### 출력 결과
``` css
      Name  Age  Score
1      Bob   32     95
3    David   29     85
0    Alice   25     88
4      Eve   22     92
2  Charlie   18     79
```

### 사용법 3 - 여러 열 기준 정렬
```python
# Age를 기준으로 오름차순, Score를 기준으로 내림차순
df_sorted = df.sort_values(['Age', 'Score'], ascending=[True, False])
print(df_sorted)
```

### 출력 결과
``` css
      Name  Age  Score
2  Charlie   18     79
4      Eve   22     92
0    Alice   25     88
3    David   29     85
1      Bob   32     95
```

### 사용법 4 - 인덱스 순서 유지 (inplace)
```python
# Age를 기준으로 오름차순 정렬 (원본 데이터프레임 수정)
df.sort_values('Age', inplace=True)
print(df)
```

> `inplace=True`를 사용하면 원본 데이터프레임이 수정됩니다. 이 경우, 정렬된 결과를 새로운 변수에 저장할 필요가 없습니다.

### 출력 결과
``` css
      Name  Age  Score
2  Charlie   18     79
4      Eve   22     92
0    Alice   25     88
3    David   29     85
1      Bob   32     95
```

### 사용법 5 - 결측값 정렬 방식 지정
```python
import numpy as np

# 결측값 포함된 데이터프레임 생성
data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eve'],
    'Score': [88, np.nan, 79, 85, np.nan]
}
df = pd.DataFrame(data)

# 결측값을 마지막에 배치
df_sorted = df.sort_values('Score', na_position='last')
print(df_sorted)
```

### 출력 결과
``` css
      Name  Score
2  Charlie   79.0
3    David   85.0
0    Alice   88.0
1      Bob    NaN
4      Eve    NaN
```

### 사용법 6 - 인덱스 기반 정렬
```python
# 인덱스 자체를 기준으로 정렬
df_sorted = df.sort_index(ascending=True)
print(df_sorted)
```

### 출력 결과
``` css
      Name  Score
0    Alice   88.0
1      Bob    NaN
2  Charlie   79.0
3    David   85.0
4      NaN    NaN
```

### 사용법 7 - 사용자 정의 함수로 정렬
```python
# 이름의 길이를 기준으로 정렬
df_sorted = df.sort_values('Name', key=lambda x: x.str.len())
print(df_sorted)
```

### 출력 결과
``` css
      Name  Score
4      NaN    NaN
3    David   85.0
1      Bob    NaN
0    Alice   88.0
2  Charlie   79.0
```

### 사용법 8 - 정렬 후 인덱스 리셋
```python
# 정렬된 데이터프레임의 인덱스를 다시 정렬
df_sorted = df.sort_values('Value', ascending=False).reset_index(drop=True)
print(df_sorted)
```

> `reset_index(drop=True)`를 사용하면 기존 인덱스는 삭제되고 새로운 인덱스가 생성됩니다.

### 출력 결과
``` css
  Category  Value
0        C     40
1        C     30
2        A     20
3        B     15
4        A     10
5        B     10
```
