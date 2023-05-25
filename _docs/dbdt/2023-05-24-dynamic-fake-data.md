---
title: Faker 에 property 추가하기 
layout: doc
subtitle: DynamicProvider 를 사용해 원하는 데이터 포멧을 추가하기
author: sunshine@ptokos.com
tags: [ dbdt ]
comments: true
---

## 만들고자 하는 것

`Job 데이터를 사용하고 싶다.`

Faker 에서 제공하는 데이터도 폭이 정말 넓지만 모든 케이스를 만족시키기는 어렵다.
참가자 목록 데이터가 필요한데 그 중 직업 데이터가 필요하다. 
이를 위해 DynamicProvider 를 사용하여 직업 데이터를 추가해보자.


## 참고 문서
공식 문서 [How to create a Dynamic Provider](https://faker.readthedocs.io/en/master/#how-to-create-a-dynamic-provider) 를 참고하여 개발하였다.

## API 만들기

### 반환하고자하는 데이터 포멧

```json
{
  "users": [
    {
      "user_id": ...,
      "name": ...,
      "email": ...,
      "date_of_application": ...,
      "job": ...
    }
  ]
}
```

### 코드 작성

#### DynamicProvider 추가
provider_name 는 fake.## 에 쓰일 이름이다.

```python
fake = Faker()
job_provider = DynamicProvider(
    provider_name="job",
    elements=["Student", "Data Analyst", "Data Engineer", "Data Scientist", "Machine Learning Engineer",
              "Backend Engineer", "Frontend Engineer", "Fullstack Engineer", "DevOps Engineer", "Cloud Engineer"],
)
```

#### /api/challenge/applicant/list 추가

```python
@app.get('/api/challenge/applicant/list')
async def challenge_user_list():
    return {
        'users': [{
            'user_id': fake.uuid4(),
            'name': fake.name(),
            'email': fake.email(),
            'date_of_application': fake.date_time_this_month(),
            'job': fake.job(),
        } for _ in range(100)]
    }
```

## 결과 확인

/docs 로 접근하여 확인한다.

일부 데이터
```json
{
  "users": [
    {
      "user_id": "915f8526-cc1f-47da-ab21-1aec45682904",
      "name": "Jennifer Morse",
      "email": "ortizjesus@example.com",
      "date_of_application": "2023-05-04T18:58:41",
      "job": "Data Analyst"
    },
    {
      "user_id": "aa63d2e4-73a9-44f3-afbf-89f4656d0948",
      "name": "Sarah Jenkins",
      "email": "lbutler@example.com",
      "date_of_application": "2023-05-14T23:36:23",
      "job": "Cloud Engineer"
    },
    
    ...
    
    ]
}
```

![8-1.png](/assets/img/dbdt/8-1.png)


