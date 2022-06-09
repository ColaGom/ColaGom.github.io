---
layout: post
title: "Spring: gradle kotlin dsl jar파일 빌드"
categories: [Dev, Spring]
tags: [spring, gradle, jar]

---

# 문제

kotlin dsl gradle 아래처럼 작성 후 build task 진행시 jar가 생성되지 않는문제가 발생

```kotlin
tasks {
  named<Jar>() {
    enabled = false
  }

  named<BootJar>("bootJar") {
    archiveName = "mercury"
  }
}
```

# 해결방안

kotlin dsl 의 `named` 함수는 inline이며 `name`인자를 넘기지않으면 단순히 Jar type의 Task(BootJar task)를 리턴하므로 정상 동작을 하지않는다.

```kotlin
named<Jar>("jar") {
  enabled = false
}

named<BootJar>("bootJar") {
  archiveName = "mercury"
}
```

# 참조

[https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/#packaging-executable.and-plain-archives](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/#packaging-executable.and-plain-archives)
