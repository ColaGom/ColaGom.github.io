---
layout: post
title: "StartService vs BindService 차이"
category: Android
tags: [service]
---
> StartService 와 BindService 차이 정리

### Overview

![start vs bind]({{ "/assets/images/start_bind_1.png" | absolute_url }})
![Lifecycle of BoundService]({{ "/assets/images/start_bind_2.png" | absolute_url }})

### Lifecycle

`startService`는 **process lifecycle**에 따라 동작한다. 즉, client에 의해(`stopService`) 또는 스스로(`stopSelf`) 종료하기 전까지는 `Service`는 정지 하지 않는다.

`bindService`의 경우 bind된 **Component(as Activity)의 lifecycle**에 따라 동작하며 bind된 component가 존재 하지 않는 경우 (= 모두 unbind되었을 때) 정지한다.

### Callback method

`startService`의 경우 `onCreate → onStartCommand` 함수가 호출되며 종료 될 때 `onDestory`가 호출된다. `onBind`, `onUnbind` 함수는 호출되지 않는다. 즉, client에서 `ServiceConnection`을 구현했더라도 `onServiceConnected`, `onServiceDisconnected`등의 함수는 호출되지 않는다.

`bindService`는 `onStartCommand`는 호출되지 않으며 Service가 최초로 binding되는 경우 onBind가 호출되고 service가 실행 중인경우 조건에 따라 onBind 혹은 onRebind가 호출된다.

모든 컴포넌트가 unbind되고 service또한 종료되었을 때 `onDestroy`가 호출된다.

### Conclusion

결국 목적에 맞게끔 service를 구현하여 사용하는것이 중요하며 Component와 lifecycle을 함께해야되는 경우 BindServcie(a.k.a bound service)를 그게 아니면 StartService를 활용하면 될것이다.

### Reference

[Services overview | Android Developers](https://developer.android.com/guide/components/services)