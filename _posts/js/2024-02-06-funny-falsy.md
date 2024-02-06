---
title: js == 트릭  
author: sunshine@ptokos.com
categories: [javascript]
---


## 유명한 자바스크립트의 == 의 함정

![funny-falsy-1.png](/assets/img/js/js-falsy-1.jpeg)
 
== 은 암시적 형변환으로 인해 예상치 못한 결과를 가져올 수 있다.
몇 가지의 예제를 통해 살펴보자.

[Abstract Equality Comparison](https://262.ecma-international.org/6.0/#sec-abstract-equality-comparison) 을 보면 조건들이 나와있다.

> 0 == "0" // true

0 == "0" 의 경우 숫자형과 문자열의 비교이니 문자열이 숫자형으로 변환된다.

0 == ToNumber("0") 이 되고 ToNumber("0") 은 0 이다.

그러므로 0 == 0 이 되어 true 가 된다.

[Abstract Equality Comparison](https://262.ecma-international.org/6.0/#sec-abstract-equality-comparison) 6번 조건 참고
---

> ["12", "34"] == "12,34" // true

배열의 경우 Object 이기 때문에 ToPrimitive 를 통해 원시 타입으로 변환된다.

[ToPrimitive](https://262.ecma-international.org/6.0/#sec-toprimitive) 을 보면 valueOf 를 호출하고 그 결과가 원시 타입이 아니면 toString 을 호출한다.
배열의 valueOf 는 원시 타입을 반환하지 않기 때문에 toString 을 호출한다.

---

> 0 == [] // true

배열의 경우는 원시 타잆이 아니여서 ToPrimitive 를 통해 원시 타입으로 변환된다.
위에서 본 것처럼 배열은 toString 을 호출한다. 

그러므로 0 == "" 로 변환된다.

0 == "" 는 숫자와 문자열의 비교이기 때문에 문자열이 숫자형으로 변환된다.

즉 o == ToNumber("") 이 되고 ToNumber("") 은 0 이다.

[ToNumber Applied to the String Type](https://262.ecma-international.org/5.1/#sec-9.3.1) 참고

---

> "0" == [] // false 

그렇다면 왜 삼단 논법이 성립하지 않을까?

위에서 살펴본 것 처럼 우선 [] 은 "" 으로 변환된다.

우선 "0" == "" 으로 변환된다.

그러면 문자열과 문자열의 비교이기 때문에 그대로 비교된다.

그래서 false 가 된다.

---

> false == null // false

[Abstract Equality Comparison](https://262.ecma-international.org/6.0/#sec-abstract-equality-comparison) 8번 조건에 따르면 Type(x) 가 Boolean 일 때 ToNumber(x) 로 변환된다.

그러므로 0 == null 이 된다.

null 은 [Abstract Equality Comparison](https://262.ecma-international.org/6.0/#sec-abstract-equality-comparison) 1~11번 조건에 맞지 않기 때문에 12번 조건인 false 를 반환한다.


---

> false == undefined // false

이 경우도 마찬가지로 0 == undefined 가 되고 undefined 는 [Abstract Equality Comparison](https://262.ecma-international.org/6.0/#sec-abstract-equality-comparison) 1~11번 조건에 맞지 않기 때문에 12번 조건인 false 를 반환한다.

---

## 결론
> === 를 사용하자.
