---
layout: post
title: "Spring: Querydsl 환경설정 및 용례"
categories: [Dev, Spring]
tags: [spring, querydsl]

---

# QueryDsl?

HQL(JPQL) 또는 native SQL을 사용 할 때 런타임에서만 query validation이 가능한 문제가 있다. JPA 2.0 부터 Criteria Query API가 제공되긴하지만 복잡한 쿼리를 작성하기에는 한계점이 있고 작성한다 해도 가독성이 매우 떨어지는 형태가 된다.

이러한 ORM framework의 문제점들을 해결하는데 목적을두는 라이브러리.

# Setup dependency

```gradle
implementation 'com.querydsl:querydsl-jpa'
annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa"
annotationProcessor "jakarta.persistence:jakarta.persistence-api"
annotationProcessor "jakarta.annotation:jakarta.annotation-api"
```

# Dsl Config

```java
@RequiredArgsConstructor
@Configuration
public class QuerydslConfig {

  private final EntityManager em;

  @Bean
  public JPAQueryFactory queryFactory() {
    return new JPAQueryFactory(em);
  }
}
```

# UseCase

연관관계가 없는 두 entity(`User`, `UserEmail`)을 조회해서 DTO로 결과값을 만들어보기(일반적으로 `OneToOne` relation이 존재하겠지만 join을 활용하는 예시를 위해 설정)

### Entity

```java
@Entity
public class User {
	@Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private long id;
	private long userId;
  ...
}

@Entity
public class UserEmail {
	@Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private long id;

	...
}
```

### DTO

```java
import com.querydsl.core.annotations.QueryProjection;

public class UserDto {

  private long id;
  private String name;
  private String email;

  public MemberDto() {
  }

  @QueryProjection
  public MemberDto(long id, String name, String email) {
    this.id = id;
    this.name = name;
    this.email = email;
  }
}
```

### UseCase

```java

@Repository
public class UserRepository extends QuerydslRepositorySupport {
  private final JPAQueryFactory qf;

  public UserDto get(long userId) {
    return qf.select(new QUserDto(user.id, user.name, uesrEmail.email))
      .from(uesr)
      .join(userEmail).on(user.id.eq(userEmail.userId))
      .where(user.id.eq(userId))
      .fetchOne();
  }
}
```
