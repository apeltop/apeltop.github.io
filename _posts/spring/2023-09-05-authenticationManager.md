---
title: AuthenticationManager 알아보기
author: sunshine@ptokos.com
categories: [ spring,spring-security ]
---

---

- 버전 : spring-boot-starter-security 3.0.2

---

## AuthenticationManagerBuilder

`AuthenticationManagerBuilder 는 문자 그대로 AuthenticationManager 를 생성하는 빌더 클래스이다.`

좀 더 상세히 알아보기 위해 클래스를 열어보면 아래와 같이 설명이 되어있다.

> SecurityBuilder used to create an AuthenticationManager.
> Allows for easily building in memory authentication, LDAP authentication, JDBC based authentication, adding
> UserDetailsService, and adding AuthenticationProvider's.

간단하게 번역하자면 아래와 같다.

> SecurityBuilder 를 사용하여 AuthenticationManager 를 생성하며 쉽게 인메모리 인증, LDAP 인증, JDBC 기반 인증, UserDetailsService 추가,
> AuthenticationProvider 추가를 할 수 있도록 한다.

AuthenticationManagerBuilder 를 사용한 예를 보자면 아래와 같다

```java
private final AuthenticationManagerBuilder authenticationManagerbuilder;
```

```java
Authentication authentication=authenticationManagerbuilder.getObject().authenticate(
        new UsernamePasswordAuthenticationToken(signInRequest.getEmail(),signInRequest.getPw())
        )
```

이를 2단계로 나눌 수 있다.

1. authenticationManagerbuilder.getObject()
2. authenticate() 호출

authenticationManagerbuilder.getObject() 를 하면 어떤 객체가 반환되는지 알아보자.

getObject() 를 눌러 들어가보면 AbstractSecurityBuilder 클래스의 getObject() 를 볼 수 있다.

반환 타입은 제네릭 O 이다.

```java
public final O getObject(){
    if(!this.building.get()){
        throw new IllegalStateException("This object has not been built");
    }
    return this.object;
}
```


AbstractSecurityBuilder 의 구현체는 어떤게 있을까?

- AbstractConfiguredSecurityBuilder
- AuthenticationManagerBuilder
- DefaultPasswordEncoderAuthenticationManagerBuilder.AuthenticationConfiguration
- DefaultPasswordEncoderAuthenticationManagerBuilder.HttpSecurityConfiguration
- HttpSecurity
- WebSecurity

그 중 AuthenticationManagerBuilder 의 class 부분을 살펴보면 아래와 같다.

```java
public class AuthenticationManagerBuilder
        extends AbstractConfiguredSecurityBuilder<AuthenticationManager, AuthenticationManagerBuilder>
```

AuthenticationManagerBuilder 는 AbstractConfiguredSecurityBuilder 를 상속받는다.
그러므로 **AbstractConfiguredSecurityBuilder** 들어가보면 아래와 같이 구현되어 있다. 

```java
public abstract class AbstractConfiguredSecurityBuilder<O, B extends SecurityBuilder<O>>
```

위에서 보았듯 AbstractConfiguredSecurityBuilder 는 O 와 B 를 제네릭으로 받는다.

그러므로 여기서 O 의 타입은 AuthenticationManager 이다. 위에서 궁금했던 getObject 반환 타입은 AuthenticationManager 이다.

그렇다면 위에 있는 인증 로직의 코드는 다음과 같이 줄일 수 있다.

```java
private final AuthenticationManagerBuilder authenticationManagerbuilder;
Authentication authentication=authenticationManagerbuilder.getObject().authenticate()
```

```java
private final AuthenticationManager authenticationManager;
Authentication authentication=authenticationManager.authenticate()
```

아마 AuthenticationManager 를 사용하려고 하면 빈을 찾을 수 없다는 에러가 발생할 것이다.

이때는 간단히 빈을 등록해주면 된다.

```java
@Bean
public AuthenticationManager authenticationManager(AuthenticationConfiguration config)throws Exception{
    return config.getAuthenticationManager();
}
```

코드는 쉬운데 빈으로 **AuthenticationConfiguration** 을 주입 받는다. **AuthenticationConfiguration** 은 처음보는데 또 들어가서 찾아보자.
클래스 선언 부분을 보면 아래와 같다.
`@Configuration` 으로 빈을 등록하는 클래스이다.

```java

@Configuration(proxyBeanMethods = false)
@Import(ObjectPostProcessorConfiguration.class)
public class AuthenticationConfiguration {
```

조금 더 아래를 보면 처음 호기심을 가졌던 **AuthenticationManagerBuilder** 또한 **AuthenticationConfiguration** 에서 등록하고 있는 것을 볼 수 있다.

```java
@Bean
public AuthenticationManagerBuilder authenticationManagerBuilder(ObjectPostProcessor<Object> objectPostProcessor,
        ApplicationContext context){
```

계속해서 파고 들어가니 `authenticationManagerbuilder.getObject()` 이 부분은 단지 `AuthenticationManager` 빈을 가져오는 것이였기 때문에 코드를 줄일 수 있었다.

AuthenticationManager 는 최상위 인터페이스로 ProviderManager 가 상속 받는다.
또한 ProviderManager 는 AuthenticationProvider 를 리스트로 가지고 있어 authenticate() 를 실행한다.
필자의 경우에는 AuthenticationProvider 의 구현체로 DaoAuthenticationProvider 가 사용되고 있다.
