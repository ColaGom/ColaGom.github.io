---
layout: post
title: "Spring: JPA 테스트(더미) 데이터 설정"
categories: [Dev, Spring]
tags: [spring, jpa]
---

# 들어가며

이번 글에서는 테스트용 데이터 초기화를 위한 방법 중 spring, hibernate에서 자동으로 인식하는 sql file(import, schema, data)를 통한 초기화 방법과 `EventListener`를 활용하여 코드를 통한 초기화 방법에대해 정리한다.

# import.sql

Hibernate를 통해 scheme생성을 진행하는 경우 (`create`, `create-drop` property 사용) import.sql 파일이 존재하면 초기화를 진행한다.

# schema.sql, data.sql

Spring boot 실행시 scheme.sql, data.sql 파일이 존재하면 실행한다. (hibernate 초기화 이전)

> 즉, 테스트용 profile이 존재하고 hibernate를 통해 DB scheme를 생성하는 프로젝트에서는 data.sql을 사용하면 scheme가 생성되지않은 상태라 Exception이 발생한다.
>

# EventListener

Spring component를 구현하고 원하는 Event class의 `EventListener` annotation을 등록해두면 해당 이벤트에 대한 리스너 등록이 가능하다.

Spring에서 제공하는 여러 `ApplicationEvent`가 있지만 이번글에서는 `ApplicationReadyEvent` 가 발생하는 시점에 테스트용 데이터 초기화를 진행한다.

```java
@Profile(...)
@Component
@RequiredArgsConstructor
public class EntityInitializer {

  private final MyRepository myRepository;

  @EventListener
  public void init(ApplicationReadyEvent event) {
    myRepository.saveAll(
      List.of(...)
        );
  }
}
```

정해진 profile(for test)에서만 동작하도록 Profile annotation도 함께 추가
