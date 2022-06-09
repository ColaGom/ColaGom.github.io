---
layout: post
title: "KMM: Ktor client 설정"
categories: [Dev, KMM]
tags: [kmm, ktor]

---

# Ktor-client

Http-client 라이브러리, kotlin & coroutine를 기반으로 구축되었으며 가볍고 강력한 비동기 통신을 지원한다.

본 글에서는 ktor-client 설정에 대하여 정리

> 2.0.0-beta 기준
>

# Setup

`HttpClient`는 정의는 크게 아래 3가지 조합으로 이뤄진다.

### HttpClientPlugin

`Config`와 `Plugin`이 분리되어있으며 `install(PLUGIN, CONFIG initalizer)` 를 통해 HttpClient에 적용 할 수 있다.

Plugin에서 install 및 prepare 함수를 구현하여 설정값(CONFIG)에 따라 interceptor등을 추가하여 원하는 동작을 구현한다.

### HttpEngine

HttpEngine은 interface이며 각 플랫폼별 HttpEngine provider를 정의하여 사용하면 된다.

대표적으로 `CIO`, `OkHttp`, `IOS` 등이 있다.

# Example in koin

```kotlin
single {
  OkHttp.create() // HttpEngine
}

single {
  HttpClient(get()) {
    install(ContentNegotiation) {
      json()
    }

    install(Logging) {
      logger = object : Logger {
        override fun log(message: String) {
          localLogger.log(message)
        }
      }

      level = LogLevel.ALL
    }

    install(HttpTimeout) {
      val timeout = 30000L
      connectTimeoutMillis = timeout
      requestTimeoutMillis = timeout
      socketTimeoutMillis = timeout
    }
  }
}
```

# Reference

[Ktor: Build Asynchronous Servers and Clients in Kotlin](https://ktor.io/)
