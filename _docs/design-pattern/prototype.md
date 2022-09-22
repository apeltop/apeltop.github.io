---
title: Prototype
layout: doc
subtitle: Prototype 패턴 맛만보기
author: sunshine@ptokos.com
tags: [design-pattern, JS]
---

## 간략 정의
> Prototype is a creational design pattern that lets you copy existing objects without making your code dependent on their classes.
> 
>  https://refactoring.guru/design-patterns/prototype

Prototype Pattern 은 GoF 디자인 패턴 중에서 객체 생성 패턴에 속한다. 
Prototype Pattern 은 존재하는 기존의 객체를 클래스의 의존하지 않고 복사하는 패턴이다. 

이번에 살펴볼 예제 다이어그램이다.
![Alt text](/assets/img/design-pattern/prototype-1.png)


## 예제 코드
```java
import java.util.HashMap;
import java.util.Map;

abstract class Lotto implements Cloneable {
    protected String country;

    abstract void addLotto();

    public Object clone() {
        Object clone = null;
        try {
            clone = super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }

        return clone;
    }
}

class KrLotto extends Lotto {
    public KrLotto() {
        this.country = "kr";
    }

    @Override
    void addLotto() {
        System.out.println("KR Lotto 추가되었습니다.");
    }
}


class UsLotto extends Lotto {
    public UsLotto() {
        this.country = "us";
    }

    @Override
    void addLotto() {
        System.out.println("US Lotto 추가되었습니다.");
    }
}

class LottoStore {
    private static Map<String, Lotto> lottoMap = new HashMap<>();

    static {
        lottoMap.put("kr", new KrLotto());
        lottoMap.put("us", new UsLotto());
    }

    public static Lotto getLotto(String country) {
        return (Lotto) lottoMap.get(country).clone();
    }
}

public class Prototype {
    public static void main(String[] args) {
        LottoStore.getLotto("kr").addLotto(); // KR Lotto 추가되었습니다.
        LottoStore.getLotto("us").addLotto(); // US Lotto 추가되었습니다.
    }
}

```

위 코드를 간단하게 보면 main 함수 안에서 LottoStore 에서 kr | us 를 호출하여 Lotto 인스턴스를 가져오고 addLotto 함수를 호출하는 것을 볼 수 있다.
이처럼 Prototype Pattern 은 객체를 직접적으로 호출하지 않고 복사된 객체를 가져와 사용할 수 있다.

혹자는 [Factory Method Pattern](/docs/design-pattern/factory-method/) 과 유사하다고 생각을 할 수 있다. 
Factory Method Pattern 은 new 를 통해 객체를 생성하여 반환하지만  Prototype Pattern 은 객체를 복사하여 반환한다.

Prototype 의 장점은 간단히 아래와 같다.
1. 구체적인 클래스를 직접 사용하지 않고 복사된 객체를 사용할 수 있다.(성능적인 부분도 향상)
2. 반복적인 초기화 코드를 줄일 수 있다.
3. 사전에 구성(configuration)을 해야하는 작업을 할 때 상속 대신에 사용할 수 있다. 
4. 실행 중에 객체 정보를 담은(해당 글에서는 LottoStore) 곳에서 인스턴스를 추가하고 삭제할 수 있다.
5. 서브클래스를 줄일 수 있다.

단점은 아래와같다.
- 내부 복사가 안되는 경우 clone 을 사용하기가 어렵다. 
- 순환 참조일 경우 사용하기 어렵다.


#### 참고 사이트
[https://www.geeksforgeeks.org/prototype-design-pattern/](https://www.geeksforgeeks.org/prototype-design-pattern/)
[https://refactoring.guru/design-patterns/prototype](https://refactoring.guru/design-patterns/prototype)
[https://vegibit.com/javascript-prototype-pattern/](https://vegibit.com/javascript-prototype-pattern/)

## 결론
Prototype Pattern 은 아직 적용해본 적이 없어 깨달음은 없지만 정리해보면서 장점을 알게되었고 추후 기회가 된다면 사용할 수 있을 패턴 같다. 
