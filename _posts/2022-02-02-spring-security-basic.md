---
layout: post
title: "Spring: Security 인가설정 - Basic"
categories: [Dev, Spring]
tags: [spring, security]

---

# SecurityConfig

Spring-security 의존성 설정 후 `WebSecurityConfigurerAdapter`을 상속하여 기본적인 인증, 인가 처리에대한 설정을 할 수 있다.

### Authorization config

`configure(HttpSecurity)` 함수를 오버라이드하여 설정한다.

여러 메소드를 chaining하는 방법으로 설정이 진행되며

***chaining 순서 그대로 인가 정책의 우선순위가 결정***되므로

좁은범위 → 넓은범위 순으로 정책을 작성해야된다.

```kotlin
override fun configure(http: HttpSecurity) {
      http
          .authorizeHttpRequests()
          .antMatchers("/shop/pay").hasRole("USER")  // 1
          .antMatchers("/shop/**").hasAnyRole("GUEST", "USER") // 2
          .antMatchers("/admin/**").hasRole("ADMIN")
          .anyRequest()
          .authenticated()
          .and()
          .formLogin()
  }
```

shop/pay 경로에는 UESR 권한이 요구되며 그 외 shop 하위경로는 GUEST, USER 모두 접근가능한 설정에 대한 예시이다. 이 때 ***1,2 의 순서가 변경되면 shop의 모든 하위경로 /shop/** 에 GUEST, USER 모두 접근가능한 정책이 선 반영***되어 /shop/pay 의 USER 권한이 요구되는 정책은 무시된다.

### inMemoryAuthentication

테스트를위한 Authentication 정보 설정

```kotlin
override fun configure(auth: AuthenticationManagerBuilder) {
    with(auth) {
        inMemoryAuthentication().withUser("user").password("{noop}1234").roles("USER")
        inMemoryAuthentication().withUser("guest").password("{noop}1234").roles("GUEST")
        inMemoryAuthentication().withUser("admin").password("{noop}1234").roles("ADMIN")
    }
}
```

password의 {} 구문은 password encoding에 대한 내용이며 noop은 따로 적용된 encoding이 없다는 표시이다.
