---
title: Factory Method
layout: doc
subtitle: Factory Method 패턴 맛만보기
author: sunshine@ptokos.com
tags: [design-pattern, JS]
---

## 간략 정의
> In class-based programming, the factory method pattern is a creational pattern that uses factory methods to deal with the problem of creating objects without having to specify the exact class of the object that will be created.
>
> https://en.wikipedia.org/wiki/Factory_method_pattern

위키피디아 정의를 빌리면 팩토리 메서드 패턴은 정확한 클래스를 지정하지 않아도 객체를 생성해서 반환하는 패턴이라고 한다. 

정확한 클래스를 지정하지 않는 다는 것은 아래 설명과 코드로 알아보고자한다.


이번에 살펴볼 예제 다이어그램입니다.
![Alt text](/assets/img/design-pattern/factory-method-1.png)

## 상황
이번 예제는 상황에 따라서 다른 커피 머신을 이용해서 커피를 내리는 것을 만들어보려한다.
주문에 따라서 란실리오, 라마르조꼬, 달라코르테의 커피 머신으로 커피 추출과 머신 청소를 하려 한다.

## 예제 코드

### 패턴 적용 전
펼쳐진 코드를 알아야 패턴을 적용했을 때 전후를 비교할 수 있어 펼친 코드를 먼저 보겠다.
만약 객체 생성을 분리해놓지 않으면 중복된 코드를 펼쳐야하고 같은 인터페이스를 사용하는 객체가 추가되었을 때 모든 곳을 찾아다니며 수정해야한다.

```javascript
class CoffeeMachine {
  brandName = 'default';

  constructor(brandName = 'default') {
    this.brandName = brandName;
  }


  pull() {
    console.log(`${this.brandName} 머신에서 커피 추출 중입니다.`);
  }

  cleanMachine() {
    console.log(`${this.brandName} 머신 청소 중입니다.`);
  }
}

class RancilioCoffeeMachine extends CoffeeMachine {
  constructor() {
    super('rancilio');
  }
}

class LaMarzoccoCoffeeMachine extends CoffeeMachine {
  constructor() {
    super('La Marzocco');
  }
}

class DalacorteCoffeeMachine extends CoffeeMachine {
  constructor() {
    super('Dalacorte');
  }
}

function extractCoffee(type: 'rancilio' | 'La Marzocco' | 'Dalacorte') {
  let machine;

  switch (type) {
    case 'rancilio':
      machine = new RancilioCoffeeMachine();
      break;
    case 'La Marzocco':
      machine = new LaMarzoccoCoffeeMachine();
      break;
    case 'Dalacorte':
      machine = new DalacorteCoffeeMachine();
      break;
    default:
      throw Error('지원하지 않는 커피 머신 종류입니다.');
  }

  machine.pull();
}

function cleanMachine(type: 'rancilio' | 'La Marzocco' | 'Dalacorte') {
  let machine;

  switch (type) {
    case 'rancilio':
      machine = new RancilioCoffeeMachine();
      break;
    case 'La Marzocco':
      machine = new LaMarzoccoCoffeeMachine();
      break;
    case 'Dalacorte':
      machine = new DalacorteCoffeeMachine();
      break;
    default:
      throw Error('지원하지 않는 커피 머신 종류입니다.');
  }

  machine.cleanMachine();
}

extractCoffee('rancilio');
cleanMachine('La Marzocco');


```

### 패턴 적용 후
어떻게 보면 단순히 객체 생성을 함수로 추출한 것 뿐이라고 생각할 수 있다. 
혹자는 이게 어떻게 패턴이라고 할 수 있는지 의아해할 수 있을 것 같다. 그렇지만 이를 토대로 응용할 수 있는 부분이 많기 때문에 패턴이라고 불리지 않나 싶다.
이 글은 Factory Method Pattern 을 설명하고 있지만 이를 좀 더 응용하고 발전한 것이 Abstract Factory Pattern 이 있을 것이다.

팩토리 메서드 패턴은 함수 추출의 장점과 매우 유사하다. 예를 들어 DB connection 을 생성하는 함수를 분리해서 사용하고 있었는데 환경 변수에 따라 connection 정보가 달라져야한다고 했을 때 기존에 DB connection 을 호출하고 있었던 부분은 수정하지 않고 내부 로직을 수정하면 된다는 장점이 있다.


```javascript
class CoffeeMachine {
  brandName = 'default';

  constructor(brandName = 'default') {
    this.brandName = brandName;
  }


  pull() {
    console.log(`${this.brandName} 머신에서 커피 추출 중입니다.`);
  }

  cleanMachine() {
    console.log(`${this.brandName} 머신 청소 중입니다.`);
  }
}

class RancilioCoffeeMachine extends CoffeeMachine {
  constructor() {
    super('rancilio');
  }
}

class LaMarzoccoCoffeeMachine extends CoffeeMachine {
  constructor() {
    super('La Marzocco');
  }
}

class DalacorteCoffeeMachine extends CoffeeMachine {
  constructor() {
    super('Dalacorte');
  }
}

class CoffeeMachineFactory {
  getCoffeeMachine(brandType): CoffeeMachine {
    switch (brandType) {
      case 'rancilio':
        return new LaMarzoccoCoffeeMachine();
      case 'La Marzocco':
        return new RancilioCoffeeMachine();
      case 'Dalacorte':
        return new DalacorteCoffeeMachine();
      default:
        throw Error('지원하지 않는 커피 머신 종류입니다.');
    }
  }
}

function extractCoffee(type: 'rancilio' | 'La Marzocco' | 'Dalacorte') {
  const machineFactory = new CoffeeMachineFactory();
  const machine = machineFactory.getCoffeeMachine(type);

  machine.pull();
}

function cleanMachine(type: 'rancilio' | 'La Marzocco' | 'Dalacorte') {
  const machineFactory = new CoffeeMachineFactory();
  const machine = machineFactory.getCoffeeMachine(type);

  machine.cleanMachine();
}

extractCoffee('rancilio');
cleanMachine('La Marzocco');

```


## 결론
Factory Pattern 은 아주 간단하지만 그만큼 중요하고 강력하다. 아마 팩토리 메소드 패턴을 몰라도 함수 추출을 작은 단위로 하던 습관이 있는 개발자라면 이미 사용하고 있는 패턴일 수 있다.

