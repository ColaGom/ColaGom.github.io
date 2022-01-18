---
layout: post
title: "Spring - jackson InvalidDefinitionException"
categories: [Dev, Spring]
tags: [spring, jackson, trouble]
---

### 문제점

![using built-in gradle](/assets/img/220118_1_1.png)

intellij에서 built-in gradle을 사용하고 lombok + jackson을 활용할때 발생하는 이슈

### 원인

위 환경에서 일반 gradle build를 사용하는경우 자동으로 컴파일러 옵션을 활성화 시켜주는데(-parameters) built-in gradle을 사용 할 때는 해당 옵션이 활성화 되지않는다. 따라서, jackson이 deserialize를 실패하여 exception 발생

### 해결책

`-parameters` 컴파일 옵션 추가

![compiler option](/assets/img/220118_1_2.png)
