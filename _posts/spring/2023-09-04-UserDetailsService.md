---
title: UserDetailsService 알아보기
author: sunshine@ptokos.com
categories: [ spring,spring-security ]
---

---

- 버전 : spring-boot-starter-security 3.0.2

---

## UserDetailsService
UserDetailsService 는 인터페이스로 매우 간단하다. 

loadUserByUsername 메소드만 구현하면 된다. 

```java
package org.springframework.security.core.userdetails;

public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

이렇게 간단한 인터페이스를 구현하면 어디에서 어떻게 사용되는지 궁금할 것이다. 이제 그 부분을 살펴보자.

### DaoAuthenticationProvider
DaoAuthenticationProvider 는 AuthenticationProvider 인터페이스를 구현한 클래스이다.
또한 UserDetailsService 를 구현한 클래스를 필요로 한다.

DaoAuthenticationProvider 의 retrieveUser 메소드를 보면 아래와 같이 UserDetailsService 를 통해 User 정보를 가져오는 것을 볼 수 있다.

```java
UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
```

여기서 this.getUserDetailsService() 는 UserDetailsService 를 구현한 구현체이다.
여기서 loadUserByUsername 를 호출한 후 UserDetails 를 반환한다.

### UserDetails
UserDetails 은 아래와 같이 명세되어있다.
이 인터페이스를 상속 받은 객체에서 권한 관리를 할 수 있도록 한다.  

```java
public interface UserDetails extends Serializable {
	Collection<? extends GrantedAuthority> getAuthorities();
	String getPassword();
	String getUsername();
    
	boolean isAccountNonExpired();
	boolean isAccountNonLocked();
	boolean isCredentialsNonExpired();
	boolean isEnabled();
}
```

UserDetails 는 상속 받으면 은근히 구현할게 많은데 이건 또 어디서 쓰이는지 알아야겠다.

AbstractUserDetailsAuthenticationProvider 의 authenticate 를 보면 아래와 같은 코드가 있다.
```java
try {
    this.preAuthenticationChecks.check(user);
    additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
}
```

this.preAuthenticationChecks 의 구현체는 DefaultPreAuthenticationChecks 이다.
DefaultPreAuthenticationChecks 의 check 함수를 보면 아래 3가지를 통해 체크한다. 

- isAccountNonLocked
- isEnabled
- isAccountNonExpired

이러한 기능들은 꽤 흔히 만날 수 있다. 비활성화 게정, 잠긴 계정, 만료된 계정 등등 이다. 
구현을 강제해서 좀 귀찮긴하지만 역시 스프링 답게 확장성이 좋다.

```java
private class DefaultPreAuthenticationChecks implements UserDetailsChecker {
		@Override
		public void check(UserDetails user) {
			if (!user.isAccountNonLocked()) {
				AbstractUserDetailsAuthenticationProvider.this.logger
						.debug("Failed to authenticate since user account is locked");
				throw new LockedException(AbstractUserDetailsAuthenticationProvider.this.messages
						.getMessage("AbstractUserDetailsAuthenticationProvider.locked", "User account is locked"));
			}
			if (!user.isEnabled()) {
				AbstractUserDetailsAuthenticationProvider.this.logger
						.debug("Failed to authenticate since user account is disabled");
				throw new DisabledException(AbstractUserDetailsAuthenticationProvider.this.messages
						.getMessage("AbstractUserDetailsAuthenticationProvider.disabled", "User is disabled"));
			}
			if (!user.isAccountNonExpired()) {
				AbstractUserDetailsAuthenticationProvider.this.logger
						.debug("Failed to authenticate since user account has expired");
				throw new AccountExpiredException(AbstractUserDetailsAuthenticationProvider.this.messages
						.getMessage("AbstractUserDetailsAuthenticationProvider.expired", "User account has expired"));
			}
		}
	}
```

아래 코드 중에서 this.preAuthenticationChecks.check(user); 는 살펴보았다.
이제 additionalAuthenticationChecks 를 알아보자.

```java
try {
    this.preAuthenticationChecks.check(user);
    additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
}
```

additionalAuthenticationChecks 를 들어가보면 아래와 같이 구현되어 있다.

이 부분에서 비밀번호가 일치하는지 여부를 확인한다. 일치하지 않을 경우 BadCredentialsException 을 발생시킨다.

```java
	@Override
	@SuppressWarnings("deprecation")
	protected void additionalAuthenticationChecks(UserDetails userDetails,
			UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
		if (authentication.getCredentials() == null) {
			this.logger.debug("Failed to authenticate since no credentials provided");
			throw new BadCredentialsException(this.messages
					.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
		}
		String presentedPassword = authentication.getCredentials().toString();
		if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
			this.logger.debug("Failed to authenticate since password does not match stored value");
			throw new BadCredentialsException(this.messages
					.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
		}
	}
```

Spring Security 를 사용해서 인증을 구현하다보면 왜 loadUserByUsername 를 통해 username 를 가지고만 조회하는지를 알 수 없다.
원래라면 id, pw 를 가지고 단순히 DB 일치여부만 판단하면 되기 때문이다. 대충 보면 스프링은 설정이 귀찮지만 들어가보면 언제나 아름답다.


