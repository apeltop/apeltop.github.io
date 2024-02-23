---
title: MySQL Caching SHA-2 Pluggable Authentication 에 대해 알아보기
author: sunshine@ptokos.com
categories: [mysql]
---


## 목표
mysql 을 사용하다보면 사용자의 비밀번호를 저장할 때 SHA-2 를 사용하는데 이때 caching_sha2_password 라는 인증 플러그인을 사용한다. 이 플러그인에 대해 알아보자.

### 사용자 인증 방식
[Pluggable Authentication](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication.html) 를 참고해서 정리해보면 아래와 같다.

Native Pluggable Authentication
> 단순히 비밀번호를 비교하는 방식

SHA-256 Pluggable Authentication
> SHA-256 알고리즘을 사용해서 비밀번호를 비교하는 방식

Caching SHA-2 Pluggable Authentication
> SHA-256 알고리즘을 사용해서 비밀번호를 비교하는 방식이지만, 캐시를 사용해서 성능을 향상시킨 방식
> 추가적인 기능 제공

Client-Side Cleartext Pluggable Authentication
> 클라이언트에서 비밀번호를 암호화하지 않고 서버로 전송하는 방식
> 
> 서버가 클라이언트 비밀번호를 수신하도록 하는 경우 필요(PAM, LDAP 등)

PAM Pluggable Authentication
> Unix 비밀번호 또는 LDAP 디렉토리와 같은 종류의 인증 방법 사용 가능
> 
> Enterprise Edition 에서만 사용 가능

Windows Pluggable Authentication
> Windows 에서 외부 인증을 사용할 수 있도록 하는 방법
> 
> Enterprise Edition 에서만 사용 가능

LDAP Pluggable Authentication
> LDAP 를 사용하여 외부 인증을 사용할 수 있도록 하는 방법
> 
> Enterprise Edition 에서만 사용 가능

Kerberos Pluggable Authentication
> Kerberos 를 사용하여 외부 인증을 사용할 수 있도록 하는 방법
> 
> Enterprise Edition 에서만 사용 가능

No-Login Pluggable Authentication
> proxy account 로만 접속 가능하도록 하는 방법

Socket Peer-Credential Pluggable Authentication
> Unix 소켓 파일을 사용하여 인증하는 방법


## Caching SHA-2 Pluggable Authentication
MySQL 8.0 부터는 caching_sha2_password 가 기본 인증 플러그인으로 사용된다.

사용자 비밀번호를 저장할 때 20byte 의 salt 를 추가하고 최소 5000번 이상 반복하여 암호화한다.
이로 비밀번호가 탈취 당하여도 brute-force 공격을 어렵게 만든다.

caching_sha2_password 는 [SCRAM(Salted Challenge Response Authentication Mechanism)](https://en.wikipedia.org/wiki/Salted_Challenge_Response_Authentication_Mechanism) 을 사용하여 인증한다.
SCRAM 은 비밀번호에 주로 사용되는 Challenge-Response authentication 방식이다. 

SCRAM 은 아래와 같은 방식으로 동작한다.
> SaltedPassword = H(password, salt, iteration-count) = PBKDF2(HMAC, password, salt, iteration-count, output length of H)

여기서 iteration-count 는 반복 횟수이다. 이 값이 클수록 보안이 높아지지만, 인증 시간이 길어진다.
MYSQL 서버에서는 **caching_sha_password_digest_rounds** 값이 이 값을 설정한다.
기본 값으로 5000 이 설정되어 있다. 

[New Default Authentication Plugin : caching_sha2_password](https://dev.mysql.com/blog-archive/mysql-8-0-4-new-default-authentication-plugin-caching_sha2_password/) 에 설명이 잘 되어있다.

1. Server 에서 Client 에게 Nonce 를 보낸다.
2. Client 는 Nonce 와 비밀번호를 가지고 SHA-256 기반 Challenge-Response 를 생선하고 Server 에게 전송한다.
3. Server 는 Client 으로부터 받은 값으로 비교한다.

![caching_sha2_password](https://dev.mysql.com/blog-archive/mysqlserverteam/wp-content/uploads/2018/01/fast.jpg)

더 자세히로 보면 caching_sha2_password 를 2가지로 나눌 수 있다.
**FAST mode** 와 **COMPLETE mode** 이다. 

**FAST mode** 는 Nonce 를 보내고 Client 가 Nonce 와 비밀번호를 가지고 SHA-256 기반 **Challenge-Response** 를 생성해서 Server 에게 전송한다.

**COMPLETE mode** 는 Client 에서 **caching_sha_password_digest_rounds** 값만큼 반복해서 암호화하여 Server 에게 전송한다.
Server 는 값 일치여부만 판단한다. COMPLETE mode 는 클라이언트의 CPU 사용량이 높아진다. 

참고 [A Tale of Two Password Authentication Plugins...](https://dev.mysql.com/blog-archive/a-tale-of-two-password-authentication-plugins/)

### JAVA 라이브러리로 들여다보기
mysql-connector-java-8.0.28.jar 의 **NativeAuthenticationProvider** 코드 중 일부를 보면 아래와 같다.

```java
List<AuthenticationPlugin<NativePacketPayload>> pluginsToInit = new LinkedList();
pluginsToInit.add(new MysqlNativePasswordPlugin());
pluginsToInit.add(new MysqlClearPasswordPlugin());
pluginsToInit.add(new Sha256PasswordPlugin());
pluginsToInit.add(new CachingSha2PasswordPlugin());
pluginsToInit.add(new MysqlOldPasswordPlugin());
pluginsToInit.add(new AuthenticationLdapSaslClientPlugin());
pluginsToInit.add(new AuthenticationKerberosClient());
pluginsToInit.add(new AuthenticationOciClient());
```

다양한 plugin 을 제공한다. 이 중에서 **CacheSha2PasswordPlugin** 을 들어가서 보자.

**nextAuthenticationStep** 메소드를 보면 아래와 같다.

```java
if (this.stage == CachingSha2PasswordPlugin.AuthStage.FAST_AUTH_SEND_SCRAMBLE) {
                    this.seed = fromServer.readString(StringSelfDataType.STRING_TERM, (String)null);
                    toServer.add(new NativePacketPayload(Security.scrambleCachingSha2(StringUtils.getBytes(this.password, this.protocol.getServerSession().getCharsetSettings().getPasswordCharacterEncoding()), this.seed.getBytes())));
                    this.stage = CachingSha2PasswordPlugin.AuthStage.FAST_AUTH_READ_RESULT;
                    return true;
                }
```

조건문 안에를 한 줄 씩 살펴보자.

서버로 nonce 를 전달 받는 부분이다.
```java
this.seed = fromServer.readString(StringSelfDataType.STRING_TERM, (String)null); 
```

Security.scrambleCachingSha2 메소드를 사용해서 비밀번호를 암호화한다.

```java
toServer.add(new NativePacketPayload(Security.scrambleCachingSha2(StringUtils.getBytes(this.password, this.protocol.getServerSession().getCharsetSettings().getPasswordCharacterEncoding()), this.seed.getBytes())));
```

**scrambleCachingSha2** 를 보면 비밀번호화 nonce 를 가지고 SHA-256 알고리즘을 사용해서 암호화한다.
최종적으로는 **xor** 를 진행해서 반환한다. 이는 위에 Server 와 Client 관련되어 첨부한 이미지와 같다.

```java
    public static byte[] scrambleCachingSha2(byte[] password, byte[] seed) throws DigestException {
        MessageDigest md;
        try {
            md = MessageDigest.getInstance("SHA-256");
        } catch (NoSuchAlgorithmException var7) {
            throw new AssertionFailedException(var7);
        }

        byte[] dig1 = new byte[CACHING_SHA2_DIGEST_LENGTH];
        byte[] dig2 = new byte[CACHING_SHA2_DIGEST_LENGTH];
        byte[] scramble1 = new byte[CACHING_SHA2_DIGEST_LENGTH];
        md.update(password, 0, password.length);
        md.digest(dig1, 0, CACHING_SHA2_DIGEST_LENGTH);
        md.reset();
        md.update(dig1, 0, dig1.length);
        md.digest(dig2, 0, CACHING_SHA2_DIGEST_LENGTH);
        md.reset();
        md.update(dig2, 0, dig1.length);
        md.update(seed, 0, seed.length);
        md.digest(scramble1, 0, CACHING_SHA2_DIGEST_LENGTH);
        byte[] mysqlScrambleBuff = new byte[CACHING_SHA2_DIGEST_LENGTH];
        xorString(dig1, mysqlScrambleBuff, scramble1, CACHING_SHA2_DIGEST_LENGTH);
        return mysqlScrambleBuff;
    }
```








