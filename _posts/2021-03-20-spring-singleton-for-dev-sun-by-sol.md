---
layout: post
title: "[SPRING] Dev. Sun을 위한 스프링 Singleton Thread-safe에 대하여"
description: "오늘 스코페 스터디 모임 후 Dev. Sun의 스프링 빈이 Thread-safe한지에 대한 의문으로 이번 포스팅을 하게 되었다."
categories: 
  - SPRING
tags: ["Dev. Sol", "SPRING", "Singleton", "Thread-safe"]
author: "Dev. Sol"
---

Dev. Sun의 궁금증!! 스프링 빈의 기본 Scope는 싱글톤이고, 스프링 환경은 멀티 쓰레드인데, 왜 하나의 객체를 여러 쓰레드가 공유하는게 문제가 되지 않아!!?!? 생각보다 답은 간단했다. 하지만 장황하게 설명할 것이다. 공부를 위해..

# 스프링(Spring), 싱글톤(Singleton), 빈(Bean)
디자인 패턴의 하나인 Singleton 패턴이란 생성자가 여러 차례 호출되어도 최초에 생성된 객체를 반환하는 것으로 단 하나의 인스턴스를 생성하는 것을 말한다. 스프링에서 생성되는 빈 또한 싱글톤 패턴으로 스프링은 공통된 객체를 여러번 생성해서 메모리를 낭비하기보다는 하나의 객체를 공유하는 것을 선호한다.

# 스레드 세이프(Thread-safe)
스레드 세이프란 멀티 스레드 환경에서 어떤 함수나 변수, 혹은 객체가 여러 스레드로부터 동시에 접근이 이루어져도 프로그램의 실행에 문제가 없음을 뜻한다.

# 스프링 빈 테스트
자 그럼 개념들을 숙지했으니 스프링 컴포넌트를 하나 만들어서 테스트해보겠다.

```java
package com.sol.solapp.user.component;

import org.springframework.stereotype.Component;

@Component
public class MailSender {

    public int send(int count) {
        System.out.println("send email...");
        return ++count;
    }
}
```
MailSender라는 클래스를 만들고 @Component를 달아주어 스프링 빈으로 정의했다. send라는 메소드는 count를 인자로 받아 카운트를 하나 증가시켜주는 함수이다(mail과 상관없지만.. 예시로 받아들이자).

```java
package com.sol.solapp.user.service.impl;

...//생략

@Service
@AllArgsConstructor
@Slf4j
@Transactional
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    private final MailSender mailSender;

    private final String[] HEADER = { "id", "firstname", "lastname", "email" };

    @Override
    public UserDTO createUser(UserDTO dto) {
        User user = UserConverter.INSTANCE.toEntity(dto);
        user = userRepository.save(user);


        IntStream.range(0,10).parallel().forEach(i -> {
            System.out.println(mailSender.send(i));
        });

        return UserConverter.INSTANCE.toDto(user);
    }
}
```
위 UserService에서는 mailSender를 주입하여 유저를 생성한 후 0부터 9까지 mailSender의 send를 호출하도록 만들었다.<br>
결과는 아래와 같다.
<img src='/assets/images/sol/20210321/spring-bean-test.PNG' />
비동기로 호출되었기 때문에 순서는 보장되지 못하였지만 결과는 1부터 10까지 결과는 보장되었다.

이번에는 MailSender의 멤버 변수로 count를 추가하고 해당 값을 변경시키는 send함수를 하나 더 만든 후 해당 함수를 호출하게 변경했다.
```java
package com.sol.solapp.user.component;

import org.springframework.stereotype.Component;

@Component
public class MailSender {

    private int count = 0;

    public int send(int count) {
        System.out.println("send email...");
        return ++count;
    }

    public int send() {
        return ++count;
    }
}
```
```java
package com.sol.solapp.user.service.impl;
...//생략

@Service
@AllArgsConstructor
@Slf4j
@Transactional
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    private final MailSender mailSender;

    private final String[] HEADER = { "id", "firstname", "lastname", "email" };

    @Override
    public UserDTO createUser(UserDTO dto) {
        User user = UserConverter.INSTANCE.toEntity(dto);
        user = userRepository.save(user);


        IntStream.range(0,10).parallel().forEach(i -> {
            System.out.println(mailSender.send());
        });

        return UserConverter.INSTANCE.toDto(user);
    }
}
``` 
결과는 아래와 같다.
<img src='/assets/images/sol/20210321/spring-bean-test2.PNG' />
4번이나 시도한 이유는.. 결과를 보다시피 의도치 않게 count에 대한 간섭이 없었기 때문이다..<br>
**중요한건 4번째 시도에서 34가 두번 나왔고 40까지 나왔어야 하는데 결과를 보장해주지 못하였다.**<br>

이러한 결과가 나온 이유는 하나의 공유자원(count)를 놓고 여러개의 스레드가 읽기/쓰기를 하면서 데이터 조작 중 문제가 발생하게 된 것이다. 결과적으로 Thread-safe하지 못하다는 것이다. 더 설명하자면 JVM에서 각각의 스레드는 자신의 stack 영역(지역변수)을 가지고 있지만 heap 영역(전역변수)은 스레드간에 공유를 하고 있다. 그래서 상태를 가지는(count를 변수로 가지고 있는) 가변 객체의 경우 문제가 발생한 것이다.

어쨌든, 스프링 빈도 멀티 쓰레드 환경에서 상태가 있는 가변 객체일 경우 Thread-safe 하지 못하다.<br>
그런데 왜 스프링 빈은 싱글톤일까? 잘 생각해보면 우리가 걱정하는 보통의 Entity의 경우에는 스프링 빈으로 관리하지 않는다. 빈으로 관리하는 객체들을 생각해보자. @Component, @Controller, @Service, @Repository가 달리는 객체들은 불변객체들만 있지 Entity나 VO, DTO 같은 가변객체들은 싱글톤으로 관리되지 않는다. Dev. Sun과 헷갈렸던 이유는 우리도 모르게 그러한 가변 객체들도 공유자원이라고 생각해버린 것 같다. 너무 당연히 사용하다보니 순간적으로 착각을 일으켰던 것 같다.

# 정리하며
싱글톤 스코프의 스프링 빈을 Thread-safe하게 관리하려면 불변 객체만을 싱글톤 스코프로 관리해야 한다. 가변 객체를 싱글톤 스코프의 스프링 빈으로 설정하지 않도록 조심할 필요가 있다. 물론, 관행적으로 우리는 그러한 개발을 할 가능성은 매우 적다는 것도 알고 있다. 그래도 혹시 모르니 주의하도록 하자! 그럼 질문의 답을 해보겠다.

질문 : 스프링 빈의 기본 Scope는 싱글톤이고, 스프링 환경은 멀티 쓰레드인데, 왜 하나의 객체를 여러 쓰레드가 공유하는게 문제가 되지 않아!!?!?
답 : 싱글톤은 당연히 Thread-safe하지 못해. 불변 객체라면 문제가 안되는데, 가변 객체라면 문제가 될 수 있지. 우리가 사용하는 VO, DTO, Entity 등의 객체들은 스프링 빈으로 관리를 하지 않아!