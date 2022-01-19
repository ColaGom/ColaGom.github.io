---
layout: post
title: "Spring : ControllerAdvice, ExceptionHandler를 활용한 예외처리"
categories: [Dev, Spring]
tags: [spring, exception]
---

## Spring error handling 기본

따로 예외처리를 구현하지 않았을 때, Spring servlet 내부에서 발생한 예외가 servlet container까지 전파되며 servlet container에서는 `BasicErrorController` 를 통해 예외처리를 진행한다.

application config값에 따라 노출하는 내용, 에러 페이지등을 다르게 설정 가능하다.

## @ControllerAdvice

Spring component annotation이며 기본적으로 global-scope에서 발생하는 exception에 대한 예외처리를 제공해주며. basePackage 인자를통해 scope를 별도로 지정 할 수 있다.

```java
@ControllerAdvice(basePackages = ...)
```

Spring-mvc에서는 `ResponseEntityExceptionHandler`를 상속하여 함수 오버라이딩을 통해 보다 편리하게 전역적인 ControllerAdvice를 작성 할 수 있다.

```java
@ControllerAdvice
public class GlobalErrorHandler extends ResponseEntityExceptionHandler {
    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        ...
    }

		@Override
    protected ResponseEntity<Object> handleTypeMismatch(TypeMismatchException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        ...
    }
}
```

## @ExceptionHandler

Component 단위로 함수의 ExceptionHandler annotation확인하여 해당 Component 영역에서의 예외처리기능을 제공

### ControllerAdvice - servlet scope

```java
@ControllerAdvice
public class GlobalErrorHandler {
	@ExceptionHandler(ContraintViolationException.class)
	public void handleConstraintViolationException(ConstraintViolationException e, ServletWebRequest webRequest) {
		...
	}
}
```

### Controller - controller scope

```java
@RestController
public class LoginController {
	...
	@ExceptionHandler(LoginException.class)
	public void handleException(LoginException e, ...) {
		...
	}
}
```

## 결론

전역적으로 사용하는 Exception의 경우 `ControllerAdvice`를 통해 처리하고

Controller scope에서 발생하는 ControllerScopedCustomException의 경우 Controller 내부에서 `ExceptionHandler`를 활용하여 처리한다.
