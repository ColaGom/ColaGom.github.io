---
layout: post
title: "Spring ELK Marker를 활용하여 특정 로그만 Logstash appender로 수집하기"
categories: [Dev, DevOps, Logging]
tags: [logstash, ELK]

---

# Marker

slf4j에 포함된 interface이며 다양한 용도로 활용가능하다.

대표적으로 turboFilter의 MarkerFilter가 있으며 전체 로그를 대상으로 marker filtering을 적용하고싶으면 이를 활용하면 된다.

본글에서는 전체 로그 대상이 아닌 단일 appender를 대상으로 marker filtering을 적용하는 방법을 정리한다

# Single Appender Filter?

```xml

<appender name="log-stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
  <destination>~~~</destination>

  <encoder class="net.logstash.logback.encoder.LogstashEncoder">
    <customFields>{"service":"my-service"}</customFields>
  </encoder>
</appender>
```

logback의 configuration의 일부분이며 상용 서비스의 경우 다양한 appender가 포함되어있을것이다.

만약 특정 appender `log-stash` 에 filter를 적용하고싶을때 turboFilter를 활용 할 수 없으며 regularFilter인 CustomFilter를 구현하여 따로 적용해야된다.

# Regular Marker Filter

```kotlin
object Markers {
  val Access: Marker by lazy {
    MarkerFactory.getMarker("ACCESS")
  }
}

class AccessMarkerFilter : Filter<ILoggingEvent>() {
  override fun decide(event: ILoggingEvent?): FilterReply {
    return if (event?.marker == Markers.Access) FilterReply.ACCEPT
    else FilterReply.DENY
  }
}
```

위 처럼 간단하게 구현가능하다.

- `FilterReply.ACCEPT`
- `FilterReply.NEUTRAL`
  - 하위 필터가 존재하면 해당 필터의 decide 적용 없으면 ACCEPT
- `FilterReply.DENY`

# 적용

```xml

<appender name="log-stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
  ...
  <filter class="com.example.filter.AccessMarkerFilter"/>
  ...
</appender>
```

```kotlin
logger.trace(Markers.Access, ~~~)
```

위 처럼 access logging을 구현한 곳에서 Marker값을 지정하고 Filter를 적용해두면 logstash에 원하는 accesslog만 수집되는 것을 확인 할 수 있다.
