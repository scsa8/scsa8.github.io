---
layout: post
title: "[SPRING] 생성자 주입 vs 필드 주입 vs 수정자 주입"
description: "이젠 스프링에서 거의 기본 룰이라고 볼 수 있지만 그래도 다시금 정리해본다."
categories: 
  - SPRING
tags: ["Dev. Sol", "SPRING"]
author: "Dev. Sol"
---

## DI

이번 포스팅의 결과부터 얘기하자면 스프링 프레임워크에서 필드 주입보다는 생성자 주입을 더 권장한다.
이를 이해하기 위해 3가지 주입 방법부터 알아보자.

### 생성자 주입(Constructor Injection)
스프링에서 공식적으로 권장하는 방식. 스프링 프레임워크 4.3 이후 부터는 의존성 주입으로부터 클래스를 완벽하게 분리할 수 있다. 단일 생성자인 경우에는 @Autowired 어노테이션도 붙이지 않는다. 다만 생성자가 2개 이상인 경우, 생성자에 @Autowired를 붙여주어야 한다.<br>
아래는 컨트롤러에서 서비스를 주입한 예이다. 단일 생성자이므로 @Autowired를 붙이지 않았고, Lombok을 통해 생성자를 만들었다.

```java
@RestController
@AllArgsConstructor
@Slf4j
@Api(value="rest/v1/users", consumes = "application/json")
public class UserController {

    private final UserService userService;

    @ApiOperation(httpMethod = "POST", value = "user 추가", produces = "application/json", consumes = "application/json")
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO userDTO) {
        log.debug("[POST] /users #Create new user", "/rest/v1");
        return ResponseEntity.ok(userService.createUser(userDTO));
    }

}
```

### 필드 주입(Field Injection)
스프링을 처음 입문할때 가장 많이 쓰는 것 같다. @Autowired 어노테이션만 붙여주면 자동으로 의존성이 주입된다. IDE를 사용하다보면 @Autowired 어노테이션을 코드스멜로 Detect하는 경우가 많을 정도로 피해야하는 것이 이젠 기본 룰이다. 최근 사내 스프링을 감싼 솔루션도 버전업을 하면서 모두 필드 주입에서 생성자 주입으로 바꿨다.
아래는 컨트롤러에서 서비스를 주입을 필드 주입을 통해 한 예이다. 
```java
@RestController
@Slf4j
@Api(value="rest/v1/users", consumes = "application/json")
public class UserController {

    @Autowired
    private UserService userService;

    @ApiOperation(httpMethod = "POST", value = "user 추가", produces = "application/json", consumes = "application/json")
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO userDTO) {
        log.debug("[POST] /users #Create new user", "/rest/v1");
        return ResponseEntity.ok(userService.createUser(userDTO));
    }

}
```

### 수정자 주입(Setter Injection)
 Setter를 이용한 주입 방법이다. 수정자 주입은 다양한 상황에서 유용하다. 컴포넌트가 의존성을 컨테이너로 노출하지만, 기본 의존성을 제공할 때는 일반적으로 수정자 주입이 의존성 주입에 가장 좋은 방법이라고 한다. 수정자 주입의 또 다른 장점은 인터페이스에서 모든 의존성을 선언할 수 있다는 것이다. 다만 객체 생성 시 의존관계를 주입하지 않아도 객체 생성이 가능하기 때문에 NullPointerException이 발생할 수 있어 객체의 참조가 null인지를 확인해야한다.
```java
@RestController
@Slf4j
@Api(value="rest/v1/users", consumes = "application/json")
public class UserController {

    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    @ApiOperation(httpMethod = "POST", value = "user 추가", produces = "application/json", consumes = "application/json")
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO userDTO) {
        log.debug("[POST] /users #Create new user", "/rest/v1");
        return ResponseEntity.ok(userService.createUser(userDTO));
    }

}
```

## 생성자 주입을 사용해야 하는 이유

### 순환 참조
오늘도 내일도 스프링을 개발할때에는 항상 여러 컴포넌트 간 의존성이 생긴다. 순환 참조란 A 컴포넌트가 B 컴포넌트를 참조하고 B 컴포넌트가 다시 A 컴포넌트를 참조할 때 발생한다.<br>
생성자 주입을 써야하는 가장 중요한 이유가 여깄다. 수정자 주입이나 필드 주입은 애플리케이션 구동 시 아무런 오류없이 정상적으로 구동된다. 즉, 실제로 해당 컴포넌트가 호출되기 전에는 문제를 발견할 수 없는 것이다. 생성자 주입의 경우에는 BeanCurrentlyCreationException이 발생하며 애플리케이션이 구동되지 않는다.<br>
결과적으로 생성자 주입을 사용해야 애플리케이션 구동 시 미리 오류를 알 수 있고 운영 중의 트러블 발생을 막을 수 있다.

> 그렇다면 왜 이런 차이가 발생할까?

답은 Bean을 주입하는 순서가 다르기 때문이다.<br>
수정자 주입의 경우 주입 받으려는 Bean의 생성자를 호출하여 Bean을 찾거나 Bean 팩토리에 등록하고 그 후에 생성자 인자에서 사용하는 Bean을 찾거나 만든다. 그 이후에 주입하려는 Bean 객체의 수정자를 호출하여 주입한다.<br>
필드 주입의 경우 수정자 주입과 같이 Bean을 생성한 후에 어노테이션이 붙은 필드에 해당하는 Bean을 찾아서 주입하는 방법이다. 즉, 먼저 Bean을 생성한 후에 필드에 대해서 주입한다.<br>
생성자 주입은 생성자로 객체를 생성하는 시점에 필요한 Bean을 주입한다. 먼저 생성자의 인자에 사용되는 Bean을 찾거나 Bean 팩토리에서 만든다. 그 후에 찾은 인자 Bean으로 주입하려는 Bean의 생성자를 호출한다. 즉, 먼저 Bean을 생성하지 않는다. 

그렇기 때문에 생성자 주입에서만 순환 참조가 문제가 되었던 것이다. 객체 생성 시점에 빈을 주입하기 때문에 순환 참조하는 객체가 생성되지 않은 상태에서 그 빈을 참조하기 때문에 Exception을 뱉는다. 이처럼 순환 참조를 피하기 위해 생성자 주입을 권장하는 것이다.

### 용이한 테스트 코드 작성
DI의 핵심은 관리되는 클래스가 DI 컨테이너에 의존성이 없어야 한다는 것이다. 즉, 독립적으로 인스턴스화가 가능한 POJO여야 한다는 것이다. DI 컨테이너를 사용하지 않고도 유닛테스트에서 인스턴스화 시킬 수 있고, 각각 나눠서 테스트도 가능하다. 필드 주입을 사용하면 필요한 의존성을 가진 클래스를 곧바로 인스턴스화 시킬 수 없다.

### 불변성(Immutability)
생성자 주입에서는 final을 선언하여 불변성을 보장하지만 필드 주입은 불가능하기 때문에 불변성을 보장할 수 없다.<br>
따라서 필드 주입의 경우 런타임 시점에 변경할 수 있다. 즉 중간에 변경이 가능하기 때문에 예기치 못한 사이드 이펙이나 NullPointerException이 발생할 것이다. 하지만 생성자 주입을 사용한다면 이와 같은 상황을 컴파일 시점에 방지할 수 있다.

## 정리
생성자 주입을 권장하는 이유는
> 순환 참조를 방지한다
> 테스트 코드 작성이 편리하다
> 불변성이 보장된다

