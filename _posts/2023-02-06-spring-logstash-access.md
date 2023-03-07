---
layout: post
title: "Spring access log Logstash 수집하기"
categories: [Dev, DevOps]
tags: [logstash, ELK]

---

spirng boot + ELK stack 구현 시 http access logging을 빠르게 구현하고 싶을때 적용 할 수 있는 방법 정리

# 의존성

```kotlin
implementation("org.slf4j:slf4j-api")
implementation("net.logstash.logback:logstash-logback-encoder")
implementation("ch.qos.logback:logback-access")
implementation("ch.qos.logback:logback-classic")
```

LogStashAppender, Encoder를 사용하기때문에 위 의존성을 프로젝트에 추가하여야한다. 현재 프로젝트의 의존성을 확인하여 적절한 버전 명시 필요

*`dependencyManagement`를 사용하여 의존성 중복문제를 발생하지 않도록 작업하는것을 추천한다.*

# logback-access.xml 작성

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <appender name="access"
            class="net.logstash.logback.appender.LogstashAccessTcpSocketAppender">
    <destination>{LOGSTASH ADDRESS}</destination>
    <encoder class="net.logstash.logback.encoder.LogstashAccessEncoder"/>
  </appender>

  <appender-ref ref="access"/>
</configuration>
```

# TomcatContext Pipeline 추가

```kotlin
@Configuration
class AccessLogConfig {

  @Bean
  fun addLogbackValve() = TomcatContextCustomizer { context ->
    javaClass.getResourceAsStream("/logback-access.xml").use {
      Files.createDirectories(
        (context.catalinaBase.toPath()
          .resolve(LogbackValve.DEFAULT_CONFIG_FILE)).parent
      )

      Files.copy(
        it, context.catalinaBase.toPath()
          .resolve(LogbackValve.DEFAULT_CONFIG_FILE)
      )
    }

    LogbackValve().let {
      it.isQuiet = true
      context.pipeline.addValve(it)
    }
  }
}
```

위 작업을 완료하면 아래 필드들이 수집되는것이 확인가능하다.

TomcatContextPipeline을 추가하여 동작하도록 구현한 방식이라 서비스 영역의 커스텀이 불가한 단점이 존재하지만 특별히 커스텀이 불필요한 서비스의경우 빠르게 작업하기 좋다.

```
"@timestamp"
"@version"
"message"
"method"
"protocol"
"status_code"
"requested_url"
"requested_uri"
"remote_host"
"content_length"
"elapsed_time"
```

# 참조

[https://github.com/logfellow/logstash-logback-encoder](https://github.com/logfellow/logstash-logback-encoder)
