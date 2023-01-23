---
title: Decorator
layout: doc
subtitle: Decorator 패턴
author: sunshine@ptokos.com
tags: [design-pattern, JS]
---

## 간략 정의
> The Decorator Pattern attaches additional responsibilities to an object dynamically. 
> Decorators provide a flexible alternative to subclassing for extending functionality.
>
> `Decorator 패턴은 객체에 동적으로 추가적인 책임을 부여한다. Decorator는 기능을 확장하기 위한 서브클래싱의 유연한 대안을 제공한다.`
> 
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

데코레이터 패턴은 기능을 확장해나갈 때 매우 유용한 패턴이다. 
특히 여러 조합들이 가능할 때 데코레이터 패턴이 유용하다. 
상태들을 하나하나 추가해나가며 조합을 만들어가기 때문이다. 

참고한 헤드 퍼스트 디자인 책의 예제는 커피 제조 관련이지만 이 챕터를 읽을 때 네이버에서 샐러디 주문하는 부분이 떠올라 예제 코드를 샐러디 주문으로 코드를 작성해보았다.

![Alt text](/assets/img/design-pattern/decorator-1.png)

<br />

![Alt text](/assets/img/design-pattern/decorator-2.png)


### Keyword
Component : `각 컴포턴트는 그 자체로 사용할 수도 있고 decorator를 통해 감싸질 수 있다.`

ConcreteComponent : `동적으로 행동(behavior)을 추가할 수 있으며 컴포넌트를 확장할 수 있다.`

Decorator : `각 decorator 는 HAS-A 로 컴포넌트를 가진다. instance variable 로 component 를 참조한다.`

ConcreteDecorator : `instance variable 를 가진 decorator 를 상속받으며 새로운 메서드를 추가할 수 있다. 그러나 기존 메서드 실행 전후에 추가되어야한다.`

![Alt text](/assets/img/design-pattern/decorator-5.jpg)
_https://timepasstechies.com/decorator-design-pattern-java-real-world-example/#jp-carousel-798_


### Class Diagram
#### 기존
![Alt text](/assets/img/design-pattern/decorator-3.png)

<br />

#### 패턴 적용
![Alt text](/assets/img/design-pattern/decorator-4.png)

### Code
기존 코드에서는 각 조합별로 클래스가 나뉘어 있지만 패턴을 적용하여 각 토핑을 선택해서 조합할 수 있도록 데코레이터 패턴을 적용하였다.

#### 기존
```typescript
abstract class Bowl {
  private _description = 'Unknown bowl';


  getDescription(): string {
    return this._description;
  }

  set description(value: string) {
    this._description = value;
  }

  abstract cost(): number;
}

class SaladBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Salad Bowl';
  }

  cost() {
    return 4300;
  }
}

class RiceBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Rice Bowl';
  }

  cost() {
    return 5000;
  }
}

class RiceWithChickenBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Rice with Chicken Bowl';
  }

  cost() {
    return 6700;
  }
}

class SaladWithChickenBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Salad with Chicken Bowl';
  }

  cost() {
    return 6000;
  }
}

class SaladWithEggBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Salad with Egg Bowl';
  }

  cost() {
    return 5000;
  }
}

class RiceWithEggBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Rice with Egg Bowl';
  }

  cost() {
    return 5700;
  }
}

class RiceWithChickenAndEggBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Rice with Chicken and Egg Bowl';
  }

  cost() {
    return 7600;
  }
}

class SaladWithChickenAndEggBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Salad with Chicken and Egg Bowl';
  }

  cost() {
    return 6900;
  }
}

class RiceWithSalmonBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Rice with Salmon Bowl';
  }

  cost() {
    return 9000;
  }
}

class SaladWithSalmonBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Salad with Salmon Bowl';
  }

  cost() {
    return 7300;
  }
}

console.log(new SaladBowl().getDescription(), new SaladBowl().cost()); // Salad Bowl 4300
console.log(new RiceBowl().getDescription(), new RiceBowl().cost()); // Rice Bowl 5000
console.log(new RiceWithChickenBowl().getDescription(), new RiceWithChickenBowl().cost()); // Rice with Chicken Bowl 6700
console.log(new SaladWithChickenBowl().getDescription(), new SaladWithChickenBowl().cost()); // Salad with Chicken Bowl 6000
console.log(new SaladWithEggBowl().getDescription(), new SaladWithEggBowl().cost()); // Salad with Egg Bowl 5000
console.log(new RiceWithEggBowl().getDescription(), new RiceWithEggBowl().cost()); // Rice with Egg Bowl 5700
console.log(new RiceWithChickenAndEggBowl().getDescription(), new RiceWithChickenAndEggBowl().cost()); // Rice with Chicken and Egg Bowl 7600
console.log(new SaladWithChickenAndEggBowl().getDescription(), new SaladWithChickenAndEggBowl().cost()); // Salad with Chicken and Egg Bowl 6900
console.log(new RiceWithSalmonBowl().getDescription(), new RiceWithSalmonBowl().cost()); // Rice with Salmon Bowl 9000
console.log(new SaladWithSalmonBowl().getDescription(), new SaladWithSalmonBowl().cost()); // Salad with Salmon Bowl 7300
```

#### 패턴 적용
```typescript
abstract class Bowl {
  private _description = 'Unknown bowl';


  getDescription(): string {
    return this._description;
  }

  set description(value: string) {
    this._description = value;
  }

  abstract cost(): number;
}

abstract class CondimentDecorator extends Bowl {
  abstract bowl: Bowl;
  abstract getDescription(): string;
}

class SaladBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Salad Bowl';
  }

  cost() {
    return 4300;
  }
}

class RiceBowl extends Bowl {
  constructor() {
    super();
    this.description = 'Rice Bowl';
  }

  cost() {
    return 5000;
  }
}

class ChickenTopping extends CondimentDecorator {
  bowl: Bowl;

  constructor(bowl: Bowl) {
    super();
    this.bowl = bowl;
  }

  getDescription(): string {
    return this.bowl.getDescription() + ', Chicken';
  }

  cost(): number {
    return this.bowl.cost() + 1700;
  }
}

class EggTopping extends CondimentDecorator {
  bowl: Bowl;

  constructor(bowl: Bowl) {
    super();
    this.bowl = bowl;
  }

  getDescription(): string {
    return this.bowl.getDescription() + ', Egg';
  }

  cost(): number {
    return this.bowl.cost() + 900;
  }
}

class SalmonTopping extends CondimentDecorator {
  bowl: Bowl;

  constructor(bowl: Bowl) {
    super();
    this.bowl = bowl;
  }

  getDescription(): string {
    return this.bowl.getDescription() + ', Salmon';
  }

  cost(): number {
    return this.bowl.cost() + 4000;
  }
}

class ApelSaladStore {
  constructor() {
    let bowl1 = new SaladBowl();
    bowl1 = new SalmonTopping(bowl1);

    let bowl2 = new RiceBowl();

    bowl2 = new ChickenTopping(bowl2);
    bowl2 = new EggTopping(bowl2);

    console.log(bowl1.getDescription() + ' ' + bowl1.cost()); // Salad Bowl, Salmon 8300
    console.log(bowl2.getDescription() + ' ' + bowl2.cost()); // Rice Bowl, Chicken, Egg 7600
  }
}

new ApelSaladStore();
```

### 실제 사용 느낌
패턴 적용 전후 코드를 작성해보니 확장성 측면이 극과 극 인 것을 느껴볼 수 있었다.
패턴을 적용한 코드에서는 어떠한 토핑도 추가할 수 있고 샐러드 볼의 기본 사이즈 M, L 에 따라 토핑 가격이 달라진다고해도 손쉽게 코드를 수정하여 기능을 추가할 수 있다.
그렇지만 패턴을 적용하지 않는 코드는 만약 사이즈에 따라 토핑 가격이 달라진다고하면 모든 클래스에 코드 추가와 수정이 불가피하다.
또한 최종적으로 상태에 따라 특정 클래스를 찾을 필요 없이 토핑이 추가될 때마다 해당 클래스를 사용하기만하면 상태 값은 추가가 되는 매우 편리한 패턴임을 느껴볼 수 있다.

