---
title: MVC
layout: doc
subtitle: MVC 패턴 맛만보기
author: sunshine@ptokos.com
tags: [design-pattern, JS]
---

## 의의
MVC 패턴은 매우 흔하게 볼 수 있는 패턴이다. 윈도우 폼, 웹, 앱 등에서 볼 수 있다.
유명하고 빈번하게 사용된다는 것은 간단하면서 매우 중요하다는 것을 의미한다고 볼 수 있다. 

MVC 는 Model View Controller 의 앞 글자를 따서 만든 이름이다.

Controller, Model, View 를 각각 분리하자는 것이다. 
장점은 model 과 view 가 각각의 책임이 명확해지고 모듈처럼 재사용할 수 있게 된다.(물론 코드를 잘 짜야한다) 
단점은 model 크기가 너무 커져 관리하기가 어렵게 된다는 점이다.

### Model
model 은 특정 로직을 한 묶음으로 묶은 것이라고 볼 수 있다. 묶음이 큰 묶음일 수 있고 작은 묶음일 수 있다.

예부터 보면 대부분의 서비스에서는 회원 기능이 있다. DB 에 쿼리를 통해 값을 처리하는 AuthModel 을 만든다고 하면 아래와 같은 기능들이 필요할 것이다.

#### AuthModel
- 특정 ID 를 가진 회원 조회하기
- 마지막 로그인 일자 기록하기
- 휴먼 계정 조회하기
- 전화번호를 기준으로 조회하기
- ...

AuthModel 은 회원과 관련된 기능을 담고 있다. model 이라고 꼭 DB 를 사용해야하는 것은 아니다. 예를 들어 JWT 토큰을 발행하는 로직이 들어갈 수도 있는 것이다. 

AuthModel 을 만들었을 때의 장점은 회원 관련된 기능을 찾거나 사용할 때 AuthModel 안에서 찾아서 사용하면 된다. 그리고 AuthModel 안에서만 사용할 로직이 있다면 외부에서 접근을 못하게 숨기고 사용할 수 있다는 것이다.

즉 다시 정리하자면 model 은 로직의 묶음이라고 볼 수 있다.

밑에서 다시 언급하겠지만 model 은 controller 에서만 사용할 수 있다.


### View
view 는 말 그래도 화면에 보여지는 것이다. view 가 분리되지 않는 예는 아래로 들 수 있다.
view 를 분리하지 않아 아래 예제처럼 작성되었다면 로그인 화면 디자인이 변경되었을 때 server 코드가 변경이 되어야한다.

view 가 독립적으로 분리되어있다면 html 파일만 변경하면 되는 것이다.

뷰가 분리된 상황
```
return 'main.html'
```

뷰가 분리 안된 상황
```
if 로그인이 안되어있을 때
    return '<input name='ID' /> <input name='PW' />
else 로그인이 되었을 때
    return '<table>...
```

### Controller
controller 는 model 에서 디테일한 로직을 처리하고 view 에 값을 전해주는 것이다.

controller 는 큰 범위를 controller 하는 것이다.

예를 들어 controller 에서 로그인 관련된 로직을 나열하면 어떠한 로직을 처리하는지 가독성이 떨어지게 된다.
그렇지만 controller 에서 model 에 있는 로그인 함수를 사용하여 로그인이 성공하면 user 객체를 반환하고 로그인이 실패했으면 null 을 반환한다면
controller 에서는 아래와 같이 처리하면 된다.

controller 와 model 을 분리하지 않았을 때
```
// AuthController
if 디비 연결 == null
    예외 발생
else
    if db.exec(ID 로 조회) == null
        return 'login.html'
    if db.exec(ID & PW 로 조회) == null
        비밀번호 실패 횟수 +1
        return 'login.html'
    else
        return 'main.html'
```
 

controller 와 model 을 분리했을 때
```
// AuthController
if authModel.login(USER_ID, USER_PW) == null
    return 'login.html'
else
    return 'main.html'
```

![Alt text](/assets/img/design-pattern/mvc-1.png)


## 예제 코드
예제로는 DB 에는 사용자 정보가 있는데 만 나이로 법률이 개정이 되면서 2살 줄이는 것이다. 조건은 한국사람만 해당이 된다.

만약 controller 가 model 과 view 를 분리되지 않는다면 아래와 같은 코드 일 것이다.
```javascript
const database = {
    user: [
        {
            name: 'foo',
            age: '20',
            country: 'kr',
            lastModified: '2022-06-11T11:40:38.736Z',
        },
        {
            name: 'bar',
            age: '30',
            country: 'kr',
            lastModified: '2022-06-10T11:40:38.736Z',
        },
        {
            name: 'zoo',
            age: '40',
            country: 'uk',
            lastModified: '2022-06-19T11:40:38.736Z',
        },
    ]
};

const Controller = () => {
    const users = database.user;
    const usersByCountryKr = [];
    for (const user of users) {
        if (user.country === 'kr') {
            usersByCountryKr.push(user);
        }
    }

    for (const userKr of usersByCountryKr) {
        userKr.age -= 2;
        userKr.lastModified = new Date().toISOString();
    }

    for (const user of database.user) {
        console.log(`이름 : ${user.name}`)
        console.log(`나이 : ${user.age}`)
        console.log(`국가 : ${user.country}`)
        console.log(`최종 수정 일자 : ${user.lastModified}`)
        console.log(`-------------`)
    }
}
```

위 코드는 아래와 같이 리펙토링?!이 가능하다. 패턴 이해를 돕기위해 코드를 펼쳤으니 이해 해주시기를 바란다.  
```javascript
const Controller = () => {
    database.user
        .filter(({country}) => country === 'kr')
        .map(user => {
            user.age -= 2;
            user.lastModified = new Date().toISOString();
        });

    for (const user of database.user) {
        console.log(`이름 : ${user.name}`)
        console.log(`나이 : ${user.age}`)
        console.log(`국가 : ${user.country}`)
        console.log(`최종 수정 일자 : ${user.lastModified}`)
        console.log(`-------------`)
    }
}
```


MVC 패턴이 적용된 코드
```javascript
const database = {
    user: [
        {
            name: 'foo',
            age: '20',
            country: 'kr',
            lastModified: '2022-06-11T11:40:38.736Z',
        },
        {
            name: 'bar',
            age: '30',
            country: 'kr',
            lastModified: '2022-06-10T11:40:38.736Z',
        },
        {
            name: 'zoo',
            age: '40',
            country: 'uk',
            lastModified: '2022-06-19T11:40:38.736Z',
        },
    ]
};

const Model = {
    getUserList: () => {
        return database.user;
    },
    selectByCountry: (countryId) => {
        return database.user
            .filter(({country}) => country === countryId);
    },
    incrementAge: (users, amount) => {
        for (const user of users) {
            user.age += amount;
            user.lastModified = new Date().toISOString();
        }
    }
}

const View = {
    userList: (users) => {
        for (const user of users) {
            console.log(`이름 : ${user.name}`)
            console.log(`나이 : ${user.age}`)
            console.log(`국가 : ${user.country}`)
            console.log(`최종 수정 일자 : ${user.lastModified}`)
            console.log(`-------------`)
        }
    }
}

const Controller = () => {
    const usersByCountryKr = Model.selectByCountry('kr')
    Model.incrementAge(usersByCountryKr, -2);
    View.userList(Model.getUserList());
}


Controller();
```

controller 부분은 아래와 같이 사용할 수도 있다.
```javascript
const Controller = () => {
    Model.incrementAge(Model.selectByCountry('kr'), -2);
    View.userList(Model.getUserList());
}
```


긴 설명보다는 좋은 코드가 훨씬 도움이 되는데 나름 예제를 고심해서 작성했지만 이해의 도움이 되었을지는 모르겠습니다.
조금이나마 도움이 되셨기를 바래봅니다! 
