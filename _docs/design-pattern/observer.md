---
title: Observer Pattern
layout: doc
subtitle: Observer 패턴
author: sunshine@ptokos.com
tags: [design-pattern, JS]
---

## 간략 정의

> Publishers + Subscribers = Observer Pattern
>
> `발행자 + 구독자 = 옵저버 패턴`
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

> The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically.
> 
> `옵저버 패턴은 객체들 사이에서 일대다 관계를 정의하는 것이다. 그래서 한 객체가 상태가 변경되면, 그 객체에 의존하는 모든 객체들이 알림을 받고 자동으로 업데이트 된다.`
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

### Keyword
**Subject Object** : `데이터 관리`

**Observer Object** : `Subject 를 구독 시 Subject 의 데이터 변경되었을 때 데이터 전달 받음.`

**Subscribe** : `Observer 를 Subject 에 등록`

**Unsubscribe** : `Observer 를 Subject 에서 제거`

**Notify** : `Subject 의 데이터 변경 시 Observer 에게 알림`

### 장점
#### Loose Coupling (낮은 결합)
> the only thing the subject knows about an observer is that it implements a certain interface (the Observer interface)
> 
> `subject는 observer가 특정 인터페이스 (Observer 인터페이스)를 구현한다는 것만 알고 있다.`
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

> We can add new observers at any time
> 
> `우리는 observer를 언제든지 추가할 수 있다.`
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

> We never need to modify the subject to add new types of observers
> 
> `새로운 observer 타입을 추가할 때 subject를 수정할 필요가 없다.`
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

> We can reuse subjects and observers independently of each other
> 
> `우리는 subject와 observer를 독립적으로 재사용할 수 있다.`
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

> Changes to either the subject or an observer will not affect the other.
> 
> `observer나 subject의 변경 사항은 다른 것에 영향을 미치지 않는다.`
>
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020

### 사용되는 곳
> you'll find plenty of examples of the pattern being used in many libraries and frameworks.
> If we look at the Java Development Kit(JDK), for instance, both the JavaBeans and Swing libraries make use of the Observer Pattern.
> The pattern's not limited to Java either; it's used in JavaScript's events and in Cocoa and Swift's Key-Value Observing protocol.
> 
> `너는 많은 라이브러리와 프레임워크에서 패턴을 사용하는 예제를 찾을 수 있다. 
예를 들어 살펴보면 Java Development Kit(JDK)에서 JavaBeans와 Swing 라이브러리가 옵저버 패턴을 사용한다. 
자바로 그치지않고 JavaScript의 이벤트와 Cocoa와 Swift의 Key-Value Observing 프로토콜에서도 사용된다.`
> 
> Eric Freeman, Elisabeth Robson, "Head First Design Patterns, 2nd Edition", O'Reilly Media, Inc., 2020 


### Class Diagram
#### 기존
![Alt text](/assets/img/design-pattern/observer-1.png)

<br />

#### 패턴 적용
![Alt text](/assets/img/design-pattern/observer-2.png)

### Code
책의 예제를 보면 완성된 코드가 아니라 코드의 일부분만 나와있어서 볼 때는 왜 파편적으로 적었지 싶어 코드를 완성시키고 포스팅하려고 보니 너무 길어져 오히려 이해도가 좋지 않을 수 있겠다는 생각이 든다.
그렇지만 개인적으로는 코드가 완성이 되어 머릿속으로 실행시켜보는게 더 이해가 잘 될 수 있다고 보기 때문에 코드를 완성하여 글을 적었다.

책에서는 습도, 온도 등의 데이터를 받아 display 하는 예제이다. 그렇지만 이 글에서는 compute 사용량 모니터링을 가지고 코드를 작성해보았다.

#### 기존
```typescript
function calcAverage(arr: number[]) {
  return arr.reduce((a, b) => a + b, 0) / arr.length;
}

class CurrentConditionDisplay {
  update(cpu, memory, disk) {
    console.log(`Current conditions: ${cpu} cpu usage and ${memory}% memory usage and ${disk}% disk usage`);
  }
}

class StatisticsDisplay {
  private cpuUsages: number[] = [];
  private memoryUsages: number[] = [];
  private diskUsages: number[] = [];

  update(cpu, memory, disk) {
    this.cpuUsages.push(cpu);
    this.memoryUsages.push(memory);
    this.diskUsages.push(disk);

    console.log(`Average conditions: ${calcAverage(this.cpuUsages)} cpu usage and ${calcAverage(this.memoryUsages)}% memory usage and ${calcAverage(this.diskUsages)}% disk usage`);
  }
}


class AlertDisplay {
  update(cpu, memory, disk) {
    if (cpu > 0.8) {
      console.log('CPU usage is too high!');
    }
    if (memory > 0.8) {
      console.log('Memory usage is too high!');
    }
    if (disk > 0.8) {
      console.log('Disk usage is too high!');
    }
  }
}


class ResourceData {
  currentConditionsDisplay = new CurrentConditionDisplay();
  statisticsDisplay = new StatisticsDisplay();
  alertDisplay = new AlertDisplay();

  getCpuUsage() {
    return 0.5;
  }

  getMemoryUsage() {
    return 0.5;
  }

  getDiskUsage() {
    return 0.9;
  }

  resourceChanged() {
    const cpu = this.getCpuUsage();
    const memory = this.getMemoryUsage();
    const disk = this.getDiskUsage();

    this.currentConditionsDisplay.update(cpu, memory, disk);
    this.statisticsDisplay.update(cpu, memory, disk);
    this.alertDisplay.update(cpu, memory, disk);
  }
}

new ResourceData().resourceChanged();
// Current conditions: 0.5 cpu usage and 0.5% memory usage and 0.9% disk usage
// Average conditions: 0.5 cpu usage and 0.5% memory usage and 0.9% disk usage
// Disk usage is too high!
```

#### 패턴 적용
```typescript
interface Observer {
  update(temperature: number, humidity: number, pressure: number): void;
}

interface Subject {
  registerObserver(o: Observer): void;
  removeObserver(o: Observer): void;
  notifyObservers(): void;
}

interface DisplayElement {
    display(): void;
}

function calcAverage(arr: number[]) {
    return arr.reduce((a, b) => a + b, 0) / arr.length;
}

class ResourceData implements Subject {
  private observers: Observer[] = [];
  private cpu!: number;
  private memory!: number;
  private disk!: number;

  setResource(cpu: number, memory: number, disk: number) {
    this.cpu = cpu;
    this.memory = memory;
    this.disk = disk;
    this.resourceChanged();
    console.log('');
  }

  registerObserver(o: Observer) {
    this.observers.push(o);
  }

  removeObserver(o: Observer) {
    const index = this.observers.indexOf(o);
    if (index >= 0) {
      this.observers.splice(index, 1);
    }
  }

  notifyObservers() {
    this.observers.forEach((observer) => {
      observer.update(this.cpu, this.memory, this.disk);
    });
  }

  resourceChanged() {
    this.notifyObservers();
  }
}

class CurrentConditionDisplay implements Observer, DisplayElement {
  private cpu!: number;
  private memory!: number;
  private disk!: number;
  private resourceData: Subject;

  constructor(resourceData: Subject) {
    this.resourceData = resourceData;
    resourceData.registerObserver(this);
  }

  update(cpu: number, memory: number, disk: number) {
    this.cpu = cpu;
    this.memory = memory;
    this.disk = disk;
    this.display();
  }

  display() {
    console.log(`Current conditions: ${this.cpu} cpu usage and ${this.memory}% memory usage and ${this.disk}% disk usage`);
  }
}

class StatisticsDisplay implements Observer, DisplayElement {
  private cpuUsages: number[] = [];
  private memoryUsages: number[] = [];
  private diskUsages: number[] = [];
  private resourceData: Subject;

  constructor(resourceData: Subject) {
    this.resourceData = resourceData;
    resourceData.registerObserver(this);
  }

  update(cpu: number, memory: number, disk: number) {
    this.cpuUsages.push(cpu);
    this.memoryUsages.push(memory);
    this.diskUsages.push(disk);

    this.display();
  }

  display() {
    console.log(`Average conditions: ${calcAverage(this.cpuUsages)} cpu usage and ${calcAverage(this.memoryUsages)}% memory usage and ${calcAverage(this.diskUsages)}% disk usage`);
  }
}

class AlertDisplay implements Observer, DisplayElement {
  private cpu!: number;
  private memory!: number;
  private disk!: number;
  private resourceData: Subject;

  constructor(resourceData: Subject) {
    this.resourceData = resourceData;
    resourceData.registerObserver(this);
  }

  update(cpu: number, memory: number, disk: number) {
    this.cpu = cpu;
    this.memory = memory;
    this.disk = disk;
    this.display();
  }

  display() {
    if (this.cpu > 0.8) {
      console.log('CPU usage is too high!');
    }
    if (this.memory > 0.8) {
      console.log('Memory usage is too high!');
    }
    if (this.disk > 0.8) {
      console.log('Disk usage is too high!');
    }
  }
}

const resourceData = new ResourceData();


new CurrentConditionDisplay(resourceData);
new StatisticsDisplay(resourceData);
new AlertDisplay(resourceData);

resourceData.setResource(0.5, 0.3, 0.2);
resourceData.setResource(0.7, 0.9, 0.2);

// Current conditions: 0.5 cpu usage and 0.3% memory usage and 0.2% disk usage
// Average conditions: 0.5 cpu usage and 0.3% memory usage and 0.2% disk usage
//
// Current conditions: 0.7 cpu usage and 0.9% memory usage and 0.2% disk usage
// Average conditions: 0.6 cpu usage and 0.6% memory usage and 0.2% disk usage
// Memory usage is too high!

```

### 실제 사용 느낌
실제로 패턴 적용 전후로 코드를 작성해보니 확장성 측면에서 크게 다르게 느껴진다.
낮은 결합이라는 것은 결국 확장성과 연관이 되어있는데 Subject 와 Observer 는 서로에 대해 매우 일부만 알고 있다.
알고있는 것이라고 하면 Observer Interface 의 메소드 뿐이다. 그렇기에 데이터를 관리하는 곳과 데이터를 사용하는 곳은 결합도가 매우 낮다.
즉 어떻게 구현되어있는지 관심사가 아니라는 것이다. 나중에 필요에 따라 추가적인 모니터링이 필요할 때 클래스를 만들어 추가하면 된다.

Observer 는 매우 흔하게 사용되고 이미 개발하며 사용되고 있지만 이를 개념적으로 이해하고 사용하는 것은 매우 중요하다. 
모르는 개념은 아니였지만 다시금 글을 읽고 정리하면서 더 명쾌하게 이해가 되었다.
