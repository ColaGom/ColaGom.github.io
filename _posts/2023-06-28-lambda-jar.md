---
layout: post
title: "AWS Lambda, 기본 설정 및 jar 배포하기"
categories: [Dev, DevOps]
tags: [github, github-action]

---

# for 비용 절감

특정 시간마다 API scraping을 진행하는 작업은 구현했다.

이걸 따로 container에서 돌리자니 비용이 아까워서 aws lambda로 배포하여 trigger를 등록하는 형태로 작업했다.

- **duration *10~20 초, 6시간 간격으로 호출***

jar 관련된 셋업이나 gradle kotlin dsl 기준으로 작성된 내용은 없어서 따로 정리.

# Build.gradle.kts

```kotlin
dependencies {
  ...
	implementation("com.amazonaws:aws-lambda-java-core:1.2.2")
	implementation("com.amazonaws:aws-lambda-java-events:3.11.1")
	runtimeOnly("com.amazonaws:aws-lambda-java-log4j2:1.5.1")
}

// packageJar task 등록
tasks.register<Zip>("packageJar") {
    into("lib") {
        from(tasks.jar)
        from(configurations.runtimeClasspath)
    }
}

// dependsOn, task 의존 관계를 추가, 즉 build task 호출 시 packageJar task 진행
tasks.named("build") {
    dependsOn("packageJar")
}
```

# Handler

```kotlin
class Handler : RequestHandler<Map<String,String>, String> {
    override fun handleRequest(input: Map<String,String>?, context: Context?): String? {
        val parameters = input.asParameters()
        return Scraper.scrap(parameters)
    }
}
```

Lambda function 호출 시 사용할 input(parameter) 와 output을 명시하고 작업

# Build jar

```kotlin
./gradlew build
```

*build/distributions/* 경로에 jar파일 생성

---

아래 배포 및 트리거 추가는 AWS console에서도 쉽게 배포가능하다, 편한 방식으로 진행하면된다.

# 배포

aws cli 관련 설정 및 lambda, events 관련 권한을 가진 iam 준비가 필요하다.

### create

```kotlin
aws lambda create-function --function-name api-scrap \
--runtime java11 --handler com.example.Handler \
--role [MY_ROLE_ARN] \
--zip-file fileb://./build/distributions/scraper.zip
```

### update

```kotlin
aws lambda update-function-code --function-name api-scrap \
--zip-file fileb://./build/distributions/scraper.zip
```

# Trigger

### Create rule

원하는것: 6시간 간격으로 호출

```kotlin
aws events put-rule \
--name every-6hours \
--schedule-expression 'rate(6 hours)'
```

결과에서 `EVENT_ARN` 값 획득

### 함수 호출 권한 Grant

```kotlin
aws lambda add-permission \
--function-name api-scrap \
--statement-id scrap-event \
--action 'lambda:InvokeFunction' \
--principal events.amazonaws.com \
--source-arn [EVENT_ARN]
```

### Event에 타겟 추가

```kotlin
targets.json
[
  {
    "Id": "1",
    "Arn": [LAMBDA_FUNCTION_ARN]
  }
]

aws events put-targets --rule every-6hours --targets file://targets.json
```

끗!

# Reference

https://docs.aws.amazon.com/lambda/latest/dg/java-package.html
