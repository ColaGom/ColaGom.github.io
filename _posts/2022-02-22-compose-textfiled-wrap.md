---
layout: post
title: "Android: Compose TextField wrap content"
categories: [Dev, Android]
tags: [android, compose]

---

# TextField wrap content

TextField UI의 경우 일반적으로 Fixed width로 디자인되나, 간혹 wrap_content처럼 작성한 text에 따른 width를 가지는 구현이 필요할 때가있다.

이 때, EditText에서는 wrap_content attribute를 활용하여 해결되지만 Compose의 TextField에서는 따로 방법이 없어 해결한 방법에 대한 정리

# Compose TextField

Compose의 TextField에서 사용하는 `Composable`인 `CoreTextField`의 구현을 살펴보면 Paragraph width값으로 layout width를 계산하고있으며, 이때 `modifier`의 `Intrinsic`값을 활용하는것을 알 수 있다.

따라서, 해당 값을 수정하여 원하는 layout을 구현 할 수 있다.

```kotlin
BasicTextField(
	modifier = Modifier.width(IntrinsicSize.Min),
	...
)
```
