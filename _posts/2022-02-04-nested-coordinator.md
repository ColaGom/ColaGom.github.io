---
layout: post
title: "Android: Nested Coordinator Layout (중첩 CoordinatorLayout)"
categories: [Dev, Android]
tags: [android]

---

# 들어가며

본 글에서는 중첩된 `CoordinatorLayout`을 사용하는 경우 ChildCoordinator의 unconsumed scroll event를 ParentCoordinator로 전파하는 방법에대하여 정리

# Nested ScrollEvent 전파

짧게 정리하자면 `NestedScrollingChild`에서 발생한 scroll event `NestedScrollingParent`로 전파되며 이때 전달되는 **consumed, unconsumed x,y**값을 활용하여 nested scroll을 구현한다. 이때 `NestedScrollingParent`, `Child` interface가 사용되며 compatibility를 위해 Parent, Parent2, Child1, Child2, Child3등의 interface가 존재한다.

### 예시

![coordinator](/assets/img/220204_1_1.png)

흔히 볼수있는 `CoordinatorLayout` 구조이며 위 구조에서 `RecyclerView(NestedScrollingChild)`의 스크롤 이벤트 발생시 `CoordinatorLayout(NestedScrollingParent)`으로 이벤트가 전파되며 `CoordinatorLayout`에서는 포함된 Chlid View의 `Behavior`에 해당 이벤트를 전파한다.

```java
//in CoordinatorLayout.onStartNestedScroll
final LayoutParams lp = (LayoutParams) view.getLayoutParams();
final Behavior viewBehavior = lp.getBehavior();
if (viewBehavior != null) {
    final boolean accepted = viewBehavior.onStartNestedScroll(this, view, child,
            target, axes, type);
    handled |= accepted;
    lp.setNestedScrollAccepted(type, accepted);
} else {
    lp.setNestedScrollAccepted(type, false);
}
```

# NestedCoordinatorLayout 문제

![nested coordinator](/assets/img/220204_1_2.jpeg)

만약 위처럼 CoordinatorLayout이 중첩된 Layout을 구현한다면 inner Coordinator의 스크롤이 발생해도 outer Coordinator에는 스크롤이 전파되지 않는 문제. 위 예시에서는 이해를 돕기위해 `AppBar`, `BottomNavigation`으로 명시했으나 `CoordinatorLayout.Behavior`을 구현한 많은 컴포넌트가 포함되는 경우가 일반적이다.

# 해결방법

앞서 정리해본 내용을 토대로 생각해보면, CoordinatorLayout은 `NestedScrollingParent`이므로 내부의 `Behaviour`에는 스크롤 이벤트가 정상적으로 전파되나 부모 `CoordinatorLayout`에는 전파되지 않는다.
여기서 자식 `CoordinatorLayout`에서 `NestedScrollingChild`를 구현하면 부모 `CoordinatorLayout`에게 이벤트를 전파 할 수 있다.

```kotlin
class NestedCoordinatorLayout @JvmOverloads constructor(
  context: Context, attrs: AttributeSet? = null
) : CoordinatorLayout(context, attrs), NestedScrollingChild3 {

  private val helper = NestedScrollingChildHelper(this)

  init {
    isNestedScrollingEnabled = true
  }

  override fun isNestedScrollingEnabled(): Boolean = helper.isNestedScrollingEnabled

  override fun setNestedScrollingEnabled(enabled: Boolean) {
    helper.isNestedScrollingEnabled = enabled
  }

  override fun hasNestedScrollingParent(type: Int): Boolean =
    helper.hasNestedScrollingParent(type)

  override fun hasNestedScrollingParent(): Boolean = helper.hasNestedScrollingParent()

  override fun onStartNestedScroll(child: View, target: View, axes: Int, type: Int): Boolean {
    val superResult = super.onStartNestedScroll(child, target, axes, type)
    return startNestedScroll(axes, type) || superResult
  }

  override fun onStartNestedScroll(child: View, target: View, axes: Int): Boolean {
    val superResult = super.onStartNestedScroll(child, target, axes)
    return startNestedScroll(axes) || superResult
  }

  override fun onNestedPreScroll(target: View, dx: Int, dy: Int, consumed: IntArray, type: Int) {
    val superConsumed = intArrayOf(0, 0)
    super.onNestedPreScroll(target, dx, dy, superConsumed, type)
    val thisConsumed = intArrayOf(0, 0)
    dispatchNestedPreScroll(dx, dy, consumed, null, type)
    consumed[0] = superConsumed[0] + thisConsumed[0]
    consumed[1] = superConsumed[1] + thisConsumed[1]
  }

  override fun onNestedPreScroll(target: View, dx: Int, dy: Int, consumed: IntArray) {
    val superConsumed = intArrayOf(0, 0)
    super.onNestedPreScroll(target, dx, dy, superConsumed)
    val thisConsumed = intArrayOf(0, 0)
    dispatchNestedPreScroll(dx, dy, consumed, null)
    consumed[0] = superConsumed[0] + thisConsumed[0]
    consumed[1] = superConsumed[1] + thisConsumed[1]
  }

  override fun onNestedScroll(
    target: View,
    dxConsumed: Int,
    dyConsumed: Int,
    dxUnconsumed: Int,
    dyUnconsumed: Int,
    type: Int,
    consumed: IntArray
  ) {
    dispatchNestedScroll(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, null, type)
    super.onNestedScroll(
      target,
      dxConsumed,
      dyConsumed,
      dxUnconsumed,
      dyUnconsumed,
      type,
      consumed
    )
  }

  override fun onNestedScroll(
    target: View, dxConsumed: Int, dyConsumed: Int, dxUnconsumed: Int,
    dyUnconsumed: Int, type: Int
  ) {
    super.onNestedScroll(target, dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, type)
    dispatchNestedScroll(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, null, type)
  }

  override fun onNestedScroll(
    target: View,
    dxConsumed: Int,
    dyConsumed: Int,
    dxUnconsumed: Int,
    dyUnconsumed: Int
  ) {
    super.onNestedScroll(target, dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed)
    dispatchNestedScroll(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, null)
  }

  override fun onStopNestedScroll(target: View, type: Int) {
    super.onStopNestedScroll(target, type)
    stopNestedScroll(type)
  }

  override fun onStopNestedScroll(target: View) {
    super.onStopNestedScroll(target)
    stopNestedScroll()
  }

  override fun onNestedPreFling(target: View, velocityX: Float, velocityY: Float): Boolean {
    val superResult = super.onNestedPreFling(target, velocityX, velocityY)
    return dispatchNestedPreFling(velocityX, velocityY) || superResult
  }

  override fun onNestedFling(
    target: View,
    velocityX: Float,
    velocityY: Float,
    consumed: Boolean
  ): Boolean {
    val superResult = super.onNestedFling(target, velocityX, velocityY, consumed)
    return dispatchNestedFling(velocityX, velocityY, consumed) || superResult
  }

  override fun startNestedScroll(axes: Int, type: Int): Boolean =
    helper.startNestedScroll(axes, type)

  override fun startNestedScroll(axes: Int): Boolean = helper.startNestedScroll(axes)

  override fun stopNestedScroll(type: Int) {
    helper.stopNestedScroll(type)
  }

  override fun stopNestedScroll() {
    helper.stopNestedScroll()
  }

  override fun dispatchNestedScroll(
    dxConsumed: Int,
    dyConsumed: Int,
    dxUnconsumed: Int,
    dyUnconsumed: Int,
    offsetInWindow: IntArray?,
    type: Int,
    consumed: IntArray
  ) {
    helper.dispatchNestedScroll(
      dxConsumed,
      dyConsumed,
      dxUnconsumed,
      dyUnconsumed,
      offsetInWindow,
      type,
      consumed
    )
  }

  override fun dispatchNestedScroll(
    dxConsumed: Int, dyConsumed: Int, dxUnconsumed: Int, dyUnconsumed: Int,
    offsetInWindow: IntArray?, type: Int
  ): Boolean = helper.dispatchNestedScroll(
    dxConsumed,
    dyConsumed,
    dxUnconsumed,
    dyUnconsumed,
    offsetInWindow,
    type
  )

  override fun dispatchNestedScroll(
    dxConsumed: Int, dyConsumed: Int, dxUnconsumed: Int,
    dyUnconsumed: Int, offsetInWindow: IntArray?
  ): Boolean = helper.dispatchNestedScroll(
    dxConsumed,
    dyConsumed,
    dxUnconsumed,
    dyUnconsumed,
    offsetInWindow
  )

  override fun dispatchNestedPreScroll(
    dx: Int, dy: Int, consumed: IntArray?,
    offsetInWindow: IntArray?, type: Int
  ): Boolean = helper.dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow, type)

  override fun dispatchNestedPreScroll(
    dx: Int,
    dy: Int,
    consumed: IntArray?,
    offsetInWindow: IntArray?
  ): Boolean =
    helper.dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow)

  override fun dispatchNestedPreFling(velocityX: Float, velocityY: Float): Boolean =
    helper.dispatchNestedPreFling(velocityX, velocityY)

  override fun dispatchNestedFling(
    velocityX: Float,
    velocityY: Float,
    consumed: Boolean
  ): Boolean =
    helper.dispatchNestedFling(velocityX, velocityY, consumed)

}
```
