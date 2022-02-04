---
layout: post
title: "Spring: FilterProxy, FilterChain 기본"
categories: [Dev, Spring]
tags: [spring, security]
---

## 들어가며

![overview](/assets/img/filter-chain.jpeg)
FilterProxy에 대한 개략적인 정리

## FilterProxy?

수신된 Request에 대해 config(matcher, ...)를 토대로 적합한 FilterChain을 생성해준다.

> request, response시 하나의 filterChain을 통과하며 ***filter chaining***이 발생한다. 따라서, filter의 구현에서 servlet의 내용을 수정하는 경우 이후 filter에서도 수정된 내용이 그대로 적용되는 것을 명심해야된다.

### Filter Chaining

위의 이해를 돕기위해 A,B,C의 필터 구현이 아래와 같을경우

```java
fun doFilter(request:~, response:~, chain: FilterChain) {
  log("before : $filterName")
  chain.doFilter(request, response)
  log("after : $filterName")
}
```

아래와 같이 log가 출력된다.

```java
before : A
before : B
before : C
~~~
after : C
after : B
after : A
```

## DefaultSecurityFilters

```java
WebAsyncManagerIntegrationFilter
SecurityContextPersistenceFilter
HeaderWriterFilter
CsrfFilter
LogoutFilter
UsernamePasswordAuthenticationFilter
DefaultLoginPageGeneratingFilter
DefaultLogoutPageGeneratingFilter
BasicAuthenticationFilter
RequestCacheAwareFilter
SecurityContextHolderAwareRequestFilter
AnonymousAuthenticationFilter
SessionManagementFilter
ExceptionTranslationFilter
FilterSecurityInterceptor
```

Security 활성화시 기본적으로 생성되는 Filter이며 Security Config에 따라 Filter의 동작과 FilterChain에 포함되는 Filter 종류가 달라진다.
