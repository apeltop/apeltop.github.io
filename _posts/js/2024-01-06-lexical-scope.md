---
title: JS Lexical Scope 
author: sunshine@ptokos.com
categories: [javascript]
---

## Lexical Scope
Lexical Scope 는 **정적 스코프** _static scope_ 라고도 한다.

> In simple terms, the lexical scope is the scope of a variable or function based on where it is defined in the source code. 
> 
> 간단히 말해서, lexical scope 는 변수나 함수가 소스코드에서 어디에 정의되었는지에 따라 결정된다.
> 
> [Lexical Scope in JavaScript](https://www.geeksforgeeks.org/lexical-scope-in-javascript/)

쉽게 말해 함수가 **어디에 정의**되었는지에 따라서 함수가 접근할 수 있는 변수의 범위가 결정된다.

### 예제

```javascript
var str = "ORIGINAL";

function changeValue() {
    var str = "CHANGED";
    printA();
}

function printA() {
    console.log(str);
}

changeValue(); // ORIGINAL
printA(); // ORIGINAL
```

위 코드를 잠시 생각해보면 changedValue() 에서는 CHANGED 를 출력할 것 같다.
이유는 printA 에서 str 을 찾을 때 호출한 함수에서 찾을 것 같기 때문이다.

그러나 **Lexical Scope** 에 의해서 printA 에서는 전역 변수 str 을 찾게 된다.
