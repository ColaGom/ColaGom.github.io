---
layout: post
title: "Android: ComposeView inside CoordinatorLayout (ComposeView CoordinatorLayout에서 사용하기)"
categories: [Dev, Android]
tags: [android, compose]

---

![compose-with-coordinator](/assets/img/220206_1_1.jpeg)

# ComposeView inside CoordinatorLayout

100% `ComposeView`로 작성한 앱은 발생하지 않을 문제이긴하나 위 레이아웃 구조 처럼 `AndroidView`와 `ComposeView`를 섞어서쓰는경우 `ComposeView`에서 발생한 scrollView 이벤트를 `CoordiantorLayout`에 전파하는 방법에대해 정리

# Touch propagation

### AndroidView

```java
main doCallback
- MessageQueue.next
 - InputEventRecevier.dispatchInputEvent()
  - ViewRoot.enqueueEvent()
   - View.dispatchPointerEvent()
    - DecorView.dispatchTouchEvent()
     - Activity.dispatchTouchEvent()
      - ViewGroup.dispatchTouchEvent()
       - TargetView.onInterceptTouchEvent()
        - if nested widget THEN startNestedScroll()
```

`dispatchTouchEvent`, `onInterceptTouchEvent`, `onTouchEvent` 의 chaning 으로 처리되는 touch event 자세한 내용은 skip

### ComposeView dispatchTouch

```java
main doCallback
- MessageQueue.next
 - InputEventRecevier.dispatchInputEvent()
  - ViewRoot.enqueueEvent()
   - View.dispatchPointerEvent()
    - DecorView.dispatchTouchEvent()
     - Activity.dispatchTouchEvent()
      - ViewGroup.dispatchTouchEvent()
       - AndroidComposeView
        - dispatchTouchEvent
        - handleTouchEvent
        - PointerInputEventProcesser.process
        - ...
```

`AndroidComposeView` 까지의 touch propagation과정은 동일하나 이후 Compose framework에서 자체적으로 제공되는 touchEvent 처리방식을 통해 이벤트 처리가 진행된다. 해당 방식과 AndroidView의 touchEvent를 처리하는 방식은 ***data struct부터 touch event를 처리하는 value 등 다른게 구현된 부분이 대부분이며 AndroidView와의 compatibility를 지원하지 않는다.***

# 문제점

> *ComposeView의 nestedScroll 동작이 필요하다.*
>

# 해결방안 (proto typing LazyColumn)

> ***단순 가능여부를 판단하기위한 테스트이며 실제 프로덕트에 적용하면 사이드 발생가능성이...***
>

가장 좋은방법은 100% `ComposeView`를 활용하여 구현하는것이다. (마이그레이션은..?)

### dispatch compose nested scroll event

`LazyColumn`, `Pager`등에서 사용되는 `NestedScrollConnection`와 `NestedScrollingChildHelper`를 구현하여 테스트 진행.

***Compose의 touch event를 처리하는데 사용하는 값(consumed, available 등)은 AndroidView와 반대로 계산된다. 따라서, -값으로 childHelper에 전달.***

```kotlin
val listState = rememberLazyListState()
val childHelper = remember {
    NestedScrollingChildHelper(requireView()).apply {
        isNestedScrollingEnabled = true
    }
}
if (listState.isScrollInProgress) {
    DisposableEffect(Unit) {
        childHelper.startNestedScroll(SCROLL_AXIS_VERTICAL)

        onDispose {
            childHelper.stopNestedScroll()
        }
    }
}
val scrollConnection = remember {
    object : NestedScrollConnection {
        override suspend fun onPostFling(
            consumed: Velocity,
            available: Velocity
        ): Velocity {
            childHelper.dispatchNestedFling(
                -consumed.x,
                -consumed.y,
                false
            )
            return super.onPostFling(consumed, available)
        }

        override fun onPostScroll(
            consumed: Offset,
            available: Offset,
            source: NestedScrollSource
        ): Offset {
            childHelper.dispatchNestedScroll(
                -consumed.x.toInt(),
                -consumed.y.toInt(),
                -available.x.toInt(),
                -available.y.toInt(),
                null
            )
            return super.onPostScroll(consumed, available, source)
        }

        override fun onPreScroll(available: Offset, source: NestedScrollSource): Offset {
            childHelper.dispatchNestedPreScroll(
                -available.x.toInt(),
                -available.y.toInt(),
                reusableIntPair,
                null
            )
            return super.onPreScroll(available, source)
        }

        override suspend fun onPreFling(available: Velocity): Velocity {
            childHelper.dispatchNestedPreFling(
                -available.x,
                -available.y,
            )
            return super.onPreFling(available)
        }
    }
}
```

> AppBar, BottomNav behaviour등은 문제없이 동작하는 부분은 확인하였으나 ExpandableToolbar등의 동작은 확인하지 못하였다.
>
