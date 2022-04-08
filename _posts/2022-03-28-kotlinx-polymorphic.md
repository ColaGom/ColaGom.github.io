---
layout: post title: "Kotlinx-serialization: PolymorphicSerializer"
categories: [Dev, Android]
tags: [android, kmm, kotlinx]

---

# 들어가며

본 글에서는 kotlinx-serialization을 사용할 때 Custom Serializer - `PolymorphicSerializer` 를 사용하는 방법에 대한 정리.

retrofit에서 CustomConvertor를 추가하는 내용과 동일한 목적을 가진다.

# PolymorphicObject

```kotlin
interface IPolymorhicType {
  val type: String
}

data class First(
  override val type: String = "F",
  ...
) : IPolymorhicType

data class First(
  override val type: String = "S",
  ...
) : IPolymorhicType
```

위 처럼 주어진 type에 따라 First 또는 Second property를 가지는 model이 있을때 적용하는 사용하는 방법

# Add Serializable

```kotlin
@Serializable
data class First(...) : IPolymorhicType

@Serializable
data class Second(...) : IPolymorhicType
```

# Serializer implements

```kotlin
object MySerializer :
  JsonContentPolymorphicSerializer<IPolymorhicType>(IPolymorhicType::class) {
  override fun selectDeserializer(element: JsonElement) = when (element.jsonObject["type"]?.jsonPrimitive?.content) {
    "F" -> First.serializer()
    else -> Second.serializer()
  }
}

@Serializable
data class Parent(
  @Serializable(with = MySerializer::class)
  val data: IPolymorhicType,
  ...
)
```
