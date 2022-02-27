---
layout: post
title: "선언형 UI 개론"
categories: [Dev]
tags: [android, compose]

---

# Declarative vs Imperative

본글에서는 선언형 UI에 대한 정의와 명령형 UI와의 차이점과 장점에대해 정리해보고자한다.

### Simple Login Button

![sample_1](/assets/img/220225_1_1.png)

![sample_2](/assets/img/220225_1_2.png)

현재 로그인상태의 유저의 경우 **profile 이미지와 alarm badge**를 보여주고

비로그인상태면 **로그인 버튼**을 노출하는 간단한 UI를 구현한다고 가정해보자

### Imperative UI

```kotlin
fun updateUI(user: User?) {
	btnLogin.isVisible = false
	imageProfile.isVisble = false
	alarmBadge.isVisible = false

	if(isLoggedIn) {
		imageProfile.isVisible = true
		imageProfile.loadImage(profileUrl)
		if(user.newAlarmCount > 0) {
			alarmBadge.isVisble = true
			alarmBadge.text = user.newAlarmCount
		}
		return
	}

	btnLogin.isVisible = true
}
```

명령형 UI로 작성할때 일반적으로 많이 사용하던 방식이며 View state 초기화 → 현재 상태 반영 순으로 작성했다.

### Declarative UI

```kotlin
@Composable
fun UserUI(user : User?) {
	if(user == null) {
		LoginButton()
		return
	}

	Image(painter = /* painter by profileUrl */)
	AlarmBadge(alarmCount = user.newAlarmCount)
}
```

선언형 UI의 경우 현재 State에 대한 UI만 작성해주면 되는식으로 훨씬 간결해지고 UI 구조를 파악하기도 좋다.

아주 간단한 로직을 구현하는 UI임에도 차이가 크며 구현하는 뷰 로직의 복잡도가 증가 할 수도록 두 방법의 구현 복잡도에는 큰 차이가 발생한다.

> 즉, 선언형 UI의 경우 View의 이전 상태에는 신경쓰지않고 순수하게 View State에 대한 UI만 잘 작성해두면 나머지는 Framework에서 처리해주는 방식이다.
>

# 이 좋은걸 이제서야?

***그럼 진작에 선언형 UI로 프레임워크를 구현하면되는데 왜 이제서야 이렇게 사용되고있는가?*** 많은 이유가있겠지만 필자가 생각하기에 선언형 UI의 특성상 State가 갱신되면 View의 최상위(root)부터 갱신(선언) 되는데 이 때 ***갱신이 필요한 Layout node만 식별하고 갱신하는 메커니즘***을 구현하고 관련 기능을 프레임워크 레벨에서 하는것이 쉽지않으며 해당 메커니즘의 최적화가 부족한 경우 명령형 UI와 비교해서 퍼포먼스 이슈가 크게 발생하므로 이렇게 시간이 오래걸린게 아닌가 싶다. (Reactive, Flutter, Swift 등의 대표적인 선언형 UI 프레임워크들의 초기버전을 생각해보면 퍼포먼스 이슈가 상당했던걸 보면... 아마도?)

# 결론

위에 작성한 차이점말고도 View 캡슐화, 단방향 data flow등 선언형 UI를 사용하는경우 효과적이고 보다 안전하게 UI 코드를 작성할 수 있으며 이 때문에 수년간 Compose framework를 기획하고 오픈소스화하는 노력을 통해 완성도 높은 프레임워크를 구현하였다고 한다.
