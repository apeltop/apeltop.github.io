---
title: json 을 Generic 객체로 변환하기
author: sunshine@ptokos.com
categories: [ spring, jackson ]
---

Spring 에서 json 을 Generic 객체로 변환하는 방법을 알아보자.

Foo 가 있고 data 부분을 제네릭 타입으로 받는다고 가정하자. 

```java
import lombok.Data;

@Data
public class Foo<T> {
    private T data;
}

@Data
class A {
    private String a;
}

@Data
class B {
    private String b;
}

```

기본적인 Jackson 의 사용법은 아래와 같다.

```java

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;

import static org.junit.jupiter.api.Assertions.*;

class JacksonGenericTest {
    @Test
    void test() throws JsonProcessingException {
        ObjectMapper objectMapper = new ObjectMapper();
        Foo<A> foo = objectMapper.readValue("{\n" +
                        "  \"data\": {\n" +
                        "    \"a\": \"test-event-id\"\n" +
                        "  }\n" +
                        "}\n" +
                        "\n"
                , Foo.class);
        System.out.println(foo);
    }
}
```

위 코드를 실행시켜 확인해보면 LinkedHashMap 으로 변환된 것을 확인할 수 있다. 의도했던 바는 A 클래스를 받아오는 것이었는데 LinkedHashMap 으로 변환되어서 나온다. 

![jackson-generic-1.png](/assets/img/spring/jackson-generic-1.png)

이는 readValue 에서 `Foo<A>.class` 로 선언을 하지 못하기 때문이다.

이를 ObjectMapper 의 constructParametricType 를 이용하여 해결할 수 있다.

첫번째 인자로 Foo.class 를 넣고 두번째 인자로 Foo 의 제네릭 타입을 넣어주면 된다.

`.constructParametricType(Foo.class, B.class)`

```java
class JacksonGenericTest {
    @Test
    void test() throws JsonProcessingException {
        ObjectMapper objectMapper = new ObjectMapper();
        JavaType javaType = objectMapper.getTypeFactory().constructParametricType(Foo.class, A.class);
        
        Foo<A> foo = objectMapper.readValue("{\n" +
                        "  \"data\": {\n" +
                        "    \"a\": \"test-event-id\"\n" +
                        "  }\n" +
                        "}\n"
                , javaType);
        System.out.println(foo);

    }
}
```

![jackson-generic-2.png](/assets/img/spring/jackson-generic-2.png)


```java
class JacksonGenericTest {
    @Test
    void test() throws JsonProcessingException {
        ObjectMapper objectMapper = new ObjectMapper();
        JavaType javaType = objectMapper.getTypeFactory().constructParametricType(Foo.class, B.class);
        
        Foo<A> foo = objectMapper.readValue("{\n" +
                        "  \"data\": {\n" +
                        "    \"b\": \"test-event-id\"\n" +
                        "  }\n" +
                        "}\n"
                , javaType);
        System.out.println(foo);

    }
}
```

![jackson-generic-3.png](/assets/img/spring/jackson-generic-3.png)

jackson 의 ObjectMappter 를 사용항려 json 을 Generic 객체에도 변환하는 방법을 알아보았다.
