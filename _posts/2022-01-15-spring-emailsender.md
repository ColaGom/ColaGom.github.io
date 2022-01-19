---
layout: post
title: "Spring : 이메일 전송 구현 via smtp"
categories: [Dev, Spring]
tags: [spring, email]
---

## 들어가며

smtp protocol을 활용하여 spring project에서 메일을 전달하는 기본 방법에 대해 정리.
smtp 서버를 직접 구축하거나 smtp를 지원하는 메일 서비스를 통해 구현 가능하다.

### JavaMailSender?

`springframework.mail`의 `MailSender`를 구현한 클래스이다.

`MailMessage` interface를 구현한 `MimeMessage` 클래스를 인자로 받아 메일을 전송한다.

application config가 적용된 구현체를 bean을 통해 주입 받을 수 있다.

## 의존성 추가

```gradle
implementation 'org.springframework.boot:spring-boot-starter-mail'
```
{: file="build.gradle" }
### 설정

```yml
mail:
    host: smtp.gmail.com # your smtp host
    port: 587
    username: '{smtp-username}'
    password: '{smtp-password}'
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
```
{: file="application.yml" }
### 메일 전송

```java
@RequiredArgsConstructor
@RestController
public class FooController {

    private final JavaMailSender mailSender;

    public void sendEmailExample() {
        SimpleMailMessage message = new SimpleMailMessage();
				message.setFrom("from-email");
				message.setTo("to-email");
				message.setSubject("제목");
				message.setText("내용");

        mailSender.send(message);
    }
}
```

### 참고
- [Sending Email with Spring mail](https://docs.spring.io/spring-framework/docs/1.2.x/reference/mail.html)
