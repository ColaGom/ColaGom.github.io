---
layout: post
title: "Spring: Mapstruct SubclassMapping 사용법"
categories: [Dev, Spring]
tags: [spring, mapstruct]

---

![mapstruct](/assets/img/220608_1_1.png)

# Mapstruct?

class mapping을 구현하기위해 주로 `ModelMapper`를 사용했었는데 퍼포먼스 이슈(특히 nested + polymorphic collection)가 있어서 대체제를 찾던중 codegen 방식의 `MapStruct`를 발견하여 채택

reflection 기반의 `ModelMapper`와 달리 annotation processing 과정에서 Mapper class를 생성해주는 방식이라 퍼포먼스가 훌륭하다.

> 현재 기준 최신버전인 1.5.x.Final 기준으로 작성되었으며 전버전(1.4.x.Final) 대비 많은 기능(`SubclassMapping`, `Map to bean`, Conditional Mapping 등) 이 추가되고 codegen 과정의 최적화가 진행되었다.(추천)
>

# codegen?

```kotlin
class UserEntity(
  val name: String,
  val email: String
)

class UserDto(
  val name: String,
  val email: String
)

@Mapper
interface UserMapper {
  fun map(entity: UserEntity): UserDto
  fun map(dto: UserDto): UserEntity
}

//Genereted code
public class UserMapperImpl implements UserMapper {
  @Override
  public UserDto map(UserEntity entity) { ... }
  @Override
  public UserEntity map(UserDto dto) { ... }
}
```

generated code를 보면 `@Mapper` annotation을 붙인 interface의 signature에 따라 적절한 mapping 함수를 생성해준다.

# SubclassMapping

### Entity & Dto

```kotlin
@Entity
@Inheritance
class AnimalEntity
class CatEntity : AnimalEntity
class DogEntity : AnimalEntity

// DTO
class Animal
class Cat : Animal
class Dog : Animal
```

### Mapper

```kotlin
/*
* ComponentModel.SPRING을 지정하면 @Component이 추가된다.
* 따라서, bean(DI)를 통해 주입받아 사용 가능하다.
*/
@Mapper(componentModel = MappingConstants.ComponentModel.SPRING)
interface AnimalMapper {
  @SubclassMappings(
    SubclassMapping(source = CatEntity::class, target = Cat::class),
    SubclassMapping(source = DogEntity::class, target = Dog::class)
  )
  fun map(entity: AnimalEntity): Animal
  fun map(entities: List<AnimalEntity>): List<Animal>
}
```

### Service

```kotlin
@Service
class AnimalService(
  private val mapper: AnimalMapper,
  private val repository: AnimalRepository
) {
  fun all(): List<Animal> = repository.findAll().let(mapper::map)
}
```

# 주의사항

> kotlin 환경에서 `isXXX` 또는 `hasXXX`이름을 가지는 boolean property는 mapping시 해당 값이 무시되는 문제가 있다.

### 원인
source.isA() getter 는 target.a property 로 mapping code가 생성되는데 `kotlin/jvm` 구현상 traget의 property 또한 isA로 생성되기때문에 mismatching

### 해결방안

직접 mapping annotation을 작성하거나 is 또는 has 접두사를 사용을 피할 것

```kotlin
data class Foo(
  val isA: Boolean
)
data class Bar(
  val isA: Boolean
)

@Mapper
interface FooBarMapper {
  fun map(foo: Foo): Bar // not work

  @Mapping(target = "isA", source = "a")
  fun map(foo: Foo): Bar
}
```

# Reference

[https://mapstruct.org/documentation/stable/reference/html/](https://mapstruct.org/documentation/stable/reference/html/)
