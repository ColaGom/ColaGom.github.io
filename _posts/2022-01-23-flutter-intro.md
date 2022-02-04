---
layout: post
title: "Flutter: Widget & Element"
categories: [Dev, Flutter]
tags: [flutter]
---

# Flutter?

> *Flutter transforms the app development process. Build, test, and deploy beautiful mobile, web, desktop, and embedded apps from a single codebase.*
>

dart로 작성한 하나의 프로젝트를 flutter cross-compiler를 활용하여 멀티 플랫폼 런타임을 생성해주는 프레임워크.

~~아직까진~~ 점유율이나 생태계측면에서 성장하고있는 단계인것으로 보여지며 작성된 ***flutter widget들을 프레임워크별로 최적화된 rendering과정을 통해 나타내는 방식이라 성능도 우수한편이다.***

본 글에서는 이러한 일련의 과정들을 알아보고 정리해보고자 한다.

# Widget?

> ***Describes the configuration for an [Element](https://api.flutter.dev/flutter/widgets/Element-class.html).***
>

Flutter framework의 핵심 클래스.

## Widget은 유저 인터페이스의 일부에 불변 설명이다. ~~(무슨말이야..?)~~

풀어서 설명하자면 일반적인 Widget의 경우 모두 불변상태이며 이를 활용해 Element infalting을 진행한다.

가변상태의 Widget이 필요의 경우 StatefuleWidget을 사용한다.

## Element와 연결된다

Flutter Render(Element) tree를 만드는 과정에서 모든 Widget은 Element로 연결되며 하나의 트리를 생성하는 과정에서 하나의 위젯이 여러곳에 위치 할 수 있다.

## 요약

- Widget은 UI의 구현체이고 불변 상태를 가진다.
- 즉, 이를 통해 Render tree를 구성할 때 중복 객체(Element) 생성을 막는 등 다양한 내부 내부 메커니즘을 구현하는데 사용된다.

# Element?

> ***An instantiation of a [Widget](https://api.flutter.dev/flutter/widgets/Widget-class.html) at a particular location in the tree.***
>

Widget은 UI의 구성중 일부를 나타낸다. 즉, 일반적으로 한 화면은 여러 Widget의 계층형 집합이며 하나의 부모 위젯에 여러 자식위젯이 존재하는 등의 구현이 가능하다.

따라서, 한 화면에는 여러개의 동일한 위젯이 존재 할 수 있으며 이들은 구현에따라 state만 다르거나(StatefulWidget) state 또한 동일한 Widget 일 수 있다.

매우 높은 실시간성을 요구하는 현재의 프론트엔드환경에서 이러한 화면의 구성들은 언제든지 빠르게 업데이트 되는 경우가 많다. 따라서, 보다 탄력적으로 대응하기위해 실제 UI구현체인 Widget은 재사용하고 Element를 통해 각 Widget의 location 정보를 갱신한다.

## Element lifecycle

- Widget.createElement를 통해 초기화를 진행
- 생성된 Element는 마운트 과정을 통해 지정된 위치의 트리에 추가
  - 해당 시점에서 마운트된 위치에따라 화면에 노출 될 수 있다(active)
- UI가 업데이트 되는 시점에 비활성화 조건에 성립되면 Element는 비활성화 처리된다. 주로 부모 Element에서 deactivateChild를 통해 비활성화 된다.
- 해당 Element가 다시 활성화 되면 Owner의 deactive element 목록에서 이를 제거하고 다시 재사용한다.
- ...

*추상적인 설명이고 디테일한 부분은 많이 찾아봐야될듯하다.*

# 결론

![flutter-elements](/assets/img/220123-2-1.png)

Element는 Widget과 RenderObject를 연결한다.

Widget은 Element로 변환되며 Element는 계층적으로 위젯의 구조와 구현에 따라 적절한 Render Tree를 생성한다. 즉, Widget tree와 Element tree 구조는 동일하나 Render tree의 경우 동일하지 않게 생성되는점을 이해하자. (Element가 초기화 될 때 active상태로 화면에 노출되고 불변 element(stateless)면 Render tree에 포함되지 않는다.)

# Reference

[Widget class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/Widget-class.html)

[Element class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/Element-class.html)
