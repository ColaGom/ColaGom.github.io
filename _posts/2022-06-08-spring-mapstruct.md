---
layout: post
title: "Spring: Mapstruct 예제(+SubclassMapping)"
categories: [Dev, Spring]
tags: [spring, mapstruct]

---

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

`isXXX` 형태의 boolean을 사용하는경우 mapping이 동작하지않는 문제가 있다.

이는 source.isA() getter ⇒ target.a property 로 mapping code가 구현되도록 되어있는데 kotlin/jvm 구현상 traget의 property 또한 isA 이기 때문에 mapping code가 생성되지않는다.

### 해결방안

직접 mapping annotation을 명시해줘야된다

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
