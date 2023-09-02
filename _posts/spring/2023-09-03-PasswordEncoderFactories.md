---
title: PasswordEncoderFactories, DelegatingPasswordEncoder, PasswordEncoder 알아보기
author: sunshine@ptokos.com
categories: [ spring,spring-security ]
---

---

- 버전 : spring-boot-starter-security 3.0.2

---

## PasswordEncoderFactories
> PasswordEncoderFactories는 PasswordEncoder를 생성하는 팩토리 클래스이다.

사용 예는 아래와 같이 사용할 수 있다.

```java
PasswordEncoderFactories.createDelegatingPasswordEncoder();
```

안에 코드를 들어가보면 아래와 같이 구현되어 있다.

기본 값으로 bcrypt 를 사용하도록 되어 있다.
반환 타입은 **DelegatingPasswordEncoder** 이다.

```java
public static PasswordEncoder createDelegatingPasswordEncoder() {
    String encodingId = "bcrypt";
    Map<String, PasswordEncoder> encoders = new HashMap<>();
    encoders.put(encodingId, new BCryptPasswordEncoder());
    encoders.put("ldap", new org.springframework.security.crypto.password.LdapShaPasswordEncoder());
    encoders.put("MD4", new org.springframework.security.crypto.password.Md4PasswordEncoder());
    encoders.put("MD5", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("MD5"));
    encoders.put("noop", org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance());
    encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
    encoders.put("scrypt", new SCryptPasswordEncoder());
    encoders.put("SHA-1", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("SHA-1"));
    encoders.put("SHA-256", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("SHA-256"));
    encoders.put("sha256", new org.springframework.security.crypto.password.StandardPasswordEncoder());

    return new DelegatingPasswordEncoder(encodingId, encoders);
}
```

### DelegatingPasswordEncoder
DelegatingPasswordEncoder 는 PasswordEncoder 인터페이스를 구현체이다.

PasswordEncoderFactories.createDelegatingPasswordEncoder() 에서는 암호화 목록 중에서 bcrypt 를 기본으로 사용하도록 되어 있다.

delegate 는 단어 그대로 대리자라는 뜻이다. 클래스 이름을 직역하면 PasswordEncoder 대리자이다.
DelegatingPasswordEncoder 를 통해 새로운 PasswordEncoder 를 생성할 수 있다.

encode 메소드는 아래와 같이 구현되어 있다. 4가지의 조합인 것을 볼 수 있다.
- idPrefix
- idForEncode
- idSuffix
- encode 된 평문 비밀번호

```java
@Override
public String encode(CharSequence rawPassword) {
    return this.idPrefix + this.idForEncode + this.idSuffix + this.passwordEncoderForEncode.encode(rawPassword);
}
```

커스텀한 PasswordEncoder 는 아래와 같이 생성할 수 있다.

```java
PasswordEncoder passwordEncoder = new DelegatingPasswordEncoder(CUSTOM_ENCODE_ID, new CUSTOM_PASSWORD_ENCODER());
```

### PasswordEncoder
PasswordEncoder 인터페이스는 아래와 같이 구현되어 있다.

해당 인터페이스를 구현체는 **org.springframework.security.crypto** 패키지에 모여있다.


```java
public interface PasswordEncoder {
    String encode(CharSequence var1);

    boolean matches(CharSequence var1, String var2);
    
    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```
