---
layout: post
title: "Android: Compose Text dp 적용하기"
categories: [Dev, Android]
tags: [android, compose]

---

# Dp for Text

UI 디자인상 sp unit을 적용하지 못하는 영역이 있다.(최대한 피해야되긴하지만..)

하지만, Compose의 Text에서 사용되는 fontSize unit이 sp로 고정되어있어 dp 값을 사용할 수 없다.

이를 해결하기위해 주어진 dp값을 sp값으로 변환하여 사용하는 방법 정리

# Extensions

```kotlin
@Composable
fun TextStyle.fixed() =
  with(LocalDensity.current) { copy(fontSize = Dp(fontSize.value).toSp()) }

val Int.dsp
  @Composable
  get() = with(LocalDensity.current) { Dp(toFloat()).toSp() }
```

# Use

```kotlin
Text(
    ...
    fontSize = 11.dsp
)

Text(
    ...
    style = style.fixed()
)
```
