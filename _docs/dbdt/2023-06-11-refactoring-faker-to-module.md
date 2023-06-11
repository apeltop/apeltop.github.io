---
title: 리팩터링 - faker 분리
layout: doc
subtitle: faker 를 별도의 모듈로 분리하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것
`faker 를 사용하는 부분이 꽤 되는데 매번 중복된 코드를 작성하고 있던 것을 모듈로 분리하여 사용하도록 하고 싶다.`

개인적으로는 막 코드를 작성하고 리펙터링하기를 선호한다. 그런데 현재 임계점이 찼기 때문에 리팩터링 작업을 진행해보려한다.

## 리팩터링 하기
### faker 모듈 만들기
util/faker.py 를 만든다.

```python
from faker import Faker
from faker.providers import DynamicProvider

fake = Faker()

job_provider = DynamicProvider(
    provider_name="job",
    elements=["Student", "Data Analyst", "Data Engineer", "Data Scientist", "Machine Learning Engineer",
              "Backend Engineer", "Frontend Engineer", "Fullstack Engineer", "DevOps Engineer", "Cloud Engineer"],
)

fake.add_provider(job_provider)
```

### faker 모듈 사용하도록 하기
`fake = Faker()` 를 기준으로 검색하였고 모두 삭제 후 `from util.faker import fake` 를 추가하였다.

main.py 의 경우 아래와 같이 코드가 간결해졌다.

![28-1.png](/assets/img/dbdt/28-1.png)

다른 곳에서도 마찬가지로 코드가 간결해졌다.

![28-2.png](/assets/img/dbdt/28-2.png)



