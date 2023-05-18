---
title: 데이터 블로그 목록 API
layout: doc
subtitle: 추천 데이터 블로그 목록 반환하는 API 만들기
author: sunshine@ptokos.com
tags: [dbdt]
comments: true
---

## API 만들기
### 목적
> 데이터 블로그 목록을 반환하고자 한다.

> 현재는 mock 데이터이지만 추후 DB 와 연동할 계획이다.

### 데이터 Structure
- blogs
  - blog[ ]
    
- blog
  - name
  - site_url
  - image_url

### 코드
#### 데이터 블로그 목록 반환하는 endpoint 추가하기

main.py 에 아래와 같이 코드를 추가한다.

단순히 /api/blogs/data 를 GET 으로 요청 시 2개의 블로그 목록을 json 으로 반환한다.

/api/blogs/data 로 만든 이유는 추후 데이터 블로그 챌린지 이외에도 생겨날 경우 데이터 블로그 이외에도 참고할 블로그를 등록할 수 있기 때문이다.
그때 단순히 /api/blogs/{name} 으로 변경하고 해당 블로그의 정보를 반환하도록 하면 되기 때문이다.

미리 확장을 많이 하기보다 확장할 수 있도록 하는 것이 중요하다.

```python
@app.get('/api/blogs/data')
async def blog_data_list():
    return {
        'blogs': [
            {
                'name': '데이터리안(Datarian)',
                'site_url': 'https://blog.hackle.io',
                'image_url': '/static/data-blog/datarian.webp',
            },
            {
                'name': '핵클(hackle)',
                'site_url': 'https://datarian.io',
                'image_url': '/static/data-blog/hackle.jpg',
            },
        ]
    }
```

응답 값이 제대로 나오는지 확인하기 위해 /doc 를 들어가본다.
![4-1.png](/assets/img/dbdt/4-1.png)

그럼 하단에 /api/blogs/data 가 추가된 것을 확인할 수 있다.
해당 row 를 클릭하면 테스트할 수 있도록 영역이 펼쳐진다.

![4-2.png](/assets/img/dbdt/4-2.png)

펼쳐진 영역을 보면 _Try it out_ 이 있는데 이를 클릭해본다. 

![4-3.png](/assets/img/dbdt/4-3.png)

그럼 Execute 라는 버튼이 나오는데 이를 클릭해본다.

![4-4.png](/assets/img/dbdt/4-4.png)

그럼 Response headers & body 을 볼 수 있다.

**response body**
```json
{
  "blogs": [
    {
      "name": "데이터리안(Datarian)",
      "site_url": "https://blog.hackle.io",
      "image_url": "/static/data-blog/datarian.webp"
    },
    {
      "name": "핵클(hackle)",
      "site_url": "https://datarian.io",
      "image_url": "/static/data-blog/hackle.jpg"
    }
  ]
}
```

#### static 이미지 사용하기
제공하는 정보 중에 image_url 이라는 정보가 있다. 이는 이미지 URL 을 의미한다.
일단 경로를 주었지만 현재 해당 이미지를 불러올 수 없다. 데이터 블로그 소개 이미지 같은 경우 정적 이미지이기 때문에 static 으로 관리하려한다.

##### 이미지 추가
![4-5.png](/assets/img/dbdt/4-5.png)


##### static 폴더 mount 

mount 하는 코드는 매우 간단하다. 기존 코드에서 2줄만 추가하면 된다.
```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")
```

##### 이미지 확인
/static/data-blog/hackle.jpg 로 접근하면 이미지를 확인할 수 있다.

![4-6.png](/assets/img/dbdt/4-6.png)


