---
title: Strategy
layout: doc
subtitle: Strategy 패턴 맛만보기
author: sunshine@ptokos.com
tags: [design-pattern, JS]
---

## 간략 정의
Strategy Pattern 은 전략 패턴으로 불리기도 합니다. 조심스레 전략 패턴을 아주 간략히 요약해보면 아래와 같이 요약할 수 있을 것 같습니다. 
> 추상형을 상속 받은 클래스를 만들어 쉽게 전략을 바꿀 수 있다

Strategy Pattern 의 장점은 추상형을 상속 받음으로 행동을 명시한다는 것입니다. 
추상형을 사용하여 필요한 기능만 사용가능함으로 자연스레 캡슐화가 됩니다. 전략 패턴을 가지고 현재 작성하지는 않았지만 Factory Pattern 을 사용하여 더욱 유용한 코드를 작성할 수 있습니다.

이번에 살펴볼 예제 다이어그램입니다.
![Alt text](/assets/img/design-pattern/strategy-1.png)

## 예제 코드
아주 극단적으로 냄새나는 코드를 작성해보고 점차 수정해보겠습니다.

### Step 1
🤮
각 클래스 안에서는 구매를 뜻하는 buy 와 결과를 확인을 의미하는 checkDrawResult 함수가 있습니다.  
```javascript
class KrLotto {
    buyKr() {
        console.log('Buy Kr Lotto');
    }

    checkDrawResultKr() {
        console.log('Check Kr Draw Result');
    }
}

class EnLotto {
    buyEn() {
        console.log('Buy En Lotto');
    }

    checkDrawResultEn() {
        console.log('Check En Draw Result');
    }
}
```

### Step 2
각 Lotto 클래스 안의 함수 이름 통일하기

```javascript
class KrLotto {
    buy() {
        console.log('Buy Kr Lotto');
    }

    checkDrawResult() {
        console.log('Check Kr Draw Result');
    }
}

class EnLotto {
    buy() {
        console.log('Buy En Lotto');
    }

    checkDrawResult() {
        console.log('Check En Draw Result');
    }
}
```

### Step 3
interface 를 만들고 상속 받기
```javascript
interface ILotto {
    buy: () => void;
    checkDrawResult: () => void;
}

class KrLotto implements ILotto {
    buy(): void {
        console.log('Buy Kr Lotto');
    }

    checkDrawResult(): void {
        console.log('Check Kr Draw Result');
    }
}

class EnLotto implements ILotto {
    buy(): void {
        console.log('Buy En Lotto')
    }

    checkDrawResult(): void {
        console.log('Check En Draw Result');
    }
}

const krLotto = new KrLotto();
krLotto.buy(); // Buy Kr Lotto

const enLotto = new EnLotto();
enLotto.buy(); // Buy En Lotto
```

### Step 4
장점 확인해보기

아래 코드는 에러 없이 출력이 됩니다. 그렇지만 krLotto 에서는 something 호출이 가능하고 enLotto 에서는 호출이 되지 않는 것을 바람직하지 않습니다.

```javascript
const krLotto = new KrLotto();
krLotto.buy();
krLotto.something();

const enLotto = new EnLotto();
enLotto.buy();
```

아래 코드는 krLotto.something(); 에서 에러가 발생합니다. 이유는 something 함수가 있을 수도 있고 없을 수도 있기 때문입니다. 
createLotto 를 통해서 사용할 수 있는 것은 ILotto 에서 명시된 함수 밖에 없습니다. 
이럴 경우 나중에 각 Lotto 클래스에 금액 지급하는 함수를 추가가 필요한 상황이 발생한다면 ILotto 에 금액 지금 함수를 명시함으로써 각 Lotto 클래스에서 코드를 작성하라고 친절히 에러로 알려줄 것입니다.

```javascript
function createLotto(country: 'kr' | 'en') {
    return country === 'kr' ? new KrLotto() : new EnLotto();
}

const krLotto = createLotto('kr');
krLotto.buy();
krLotto.something(); // 에러 발생

const enLotto = createLotto('en');
enLotto.buy();
```

## 결론
Strategy Pattern 은 매우 유용하고 사용 할 일이 많은 것 같습니다. 이 패턴의 최대 장점이라고 보는 것은 캡슐화입니다.
외부에서 접근할 수 있는 함수를 명확히 할 수 있기 때문입니다. 또한 전략이라는 단어처럼 필요에 따라 간단하게 전략을 바꿀 수 있다는 것입니다.
복잡하게 특정 함수에 로직을 추가하는 것이 아니라 새로운 클래스를 만들고 기능을 구현하고 적은 코드로 새로 만든 클래스를 생성하여 반환할 수 있도록하면 되기 때문입니다.
하나의 로직 안에서 값들이 막 변경이 되어 특정 값을 반환하는 것이 아니라 **클래스 생성 및 반환**만하면 내부 로직은 해당 클래스에서 처리하기 때문입니다.
이러한 이유로 Strategy Pattern 은 중요한 것 같습니다.
