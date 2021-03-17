---
layout: post
title: "[SPRING] Spring AOP (1)"
description: "Spring AOP에 대해서 알아보고 실무 예제를 살펴보자"
categories: 
  - SPRING
tags: ["Dev. Sol", "SPRING", "AOP"]
author: "Dev. Sol"
---
Spring AOP에 대해서 2편에 걸쳐서 알아보려한다.
1편에는 간단하게 개념을 소개하고 실제 사용 예제를 살펴보고<br>
2편에는 더 다양한 기능을 살펴보려 한다.

# AOP란?

AOP는 Aspect Oriented Programming의 약자로 그대로 표현하면 "측면/양상 지향 프로그래밍"이라고 할 수 있다.<br>

> 그러면 측면/양상 지향 프로그래밍이 뭔데?<br>

아래 예시를 보자

```java
class Sol {
    void before() {
        System.out.println("before");
    }

    void a() {
        before();
        System.out.println("Hello");
        after();
    }

    void b() {
        before();
        System.out.println("Bye");
        after();
    }

    void after() {
        System.out.println("before");
    }
}

class Hoon {
    void before() {
        System.out.println("before");
    }

    void c() {
        before();
        System.out.println("Hoon");
        after();
    }

    void after() {
        System.out.println("before");
    }
}
```

위와 같이 before(), after()가 여기저기서 중복 사용되고 흩어져 있으면 코드 변경이 필요한 경우 매우 귀찮아질 수 밖에 없다.

AOP는 중복되는 코드를 분리하여 각각의 메소드가 본연의 역할에 충실하도록 해주는 개념이다. 여기서 바로 "중복되는 코드"가 aspect라고 보면 된다.

### 프록시 패턴
Spring AOP는 AOP를 구현하기 위해 프록시 패턴을 사용한다. 

AOP 대상인 클래스의 빈이 만들어질때 Spring AOP가 프록시를 만들고 해당 클래스 대신 프록시를 빈으로 등록한다. 
이렇게 되면 해당 클래스가 사용되는 시점에 프록시를 사용되게 된다.

# AOP 적용 예제

내가 실무에 적용했던 경험을 통해 설명하겠다.<br>
설명하기 전 왜 이렇게 사용하게 되었는지 약간의 스토리텔링이 필요해 보이기에 잠깐 설명하고 넘어가겠다. 이런 내용이 필요 없는 사람은 스크롤해서 스킵하자.

> 난 항상 비즈니스 로직 validation에 대해 고민이 많았다. 여기저기 산재해 있는 중간 중간 들어가있는 validation 로직은 때로는 매우 지저분해 보이고 중간중간 정책 변경이 있을 때,
> 수정하기도 매우 조심스럽고 오래된 레거시 코드에는 불필요한 validation 로직까지 들어있는 경우가 많다.<br>
> 그래서 항상 고민했던게 "예쁘게 validation을 하고 싶다" 였다. 그러다가 AOP를 공부했고, 서비스를 호출하기 전에 할 수 있는 validation은 따로 관리하는게 좋겠다고 생각해서 validation 클래스를 AOP로 뺐다.
> 이렇게 하면 validation 로직을 수정하기 더 수월하고, 테스트 코드 작성 또한 용이해진다.
> 정답은 없다. 이럼에도 불구하고 어쩔 수 없이 서비스 로직 안에 들어가야 할 경우가 반드시 생기기 때문에... 여전히 고민 중인 문제다.

아래 예제를 보자

```java
@Service
@AllArgsConstructor
@Slf4j
@Transactional
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    @Override
    public UserDTO createUser(UserDTO dto) {
        User user = UserConverter.INSTANCE.toEntity(dto);
        user = userRepository.save(user);
        return UserConverter.INSTANCE.toDto(user);
    }
}
```

위와 같은 UserService를 상속하는 UserServiceImpl이 있다고 하자.
간단한 예제라 대단한 validation 로직은 없지만 어쨌든 AOP를 사용해서 dto 검증을 해보자.

```java
package com.sol.solapp.user.service.validator;

import com.sol.solapp.common.exception.ServiceException;
import com.sol.solapp.common.exception.SolException;
import com.sol.solapp.common.exception.code.ErrorCode;
import com.sol.solapp.common.util.ValidateUtil;
import com.sol.solapp.user.rest.dto.UserDTO;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class UserServiceValidator {

    @Before("execution(* com.sol.solapp.user.service.UserService.createUser(..)) && args(dto)")
    public void createUser(UserDTO dto) {
        Optional.ofNullable(dto.getFirstName()).orElseThrow(() -> new ServiceException(ErrorCode.PARAMETER_EMPTY));
        Optional.ofNullable(dto.getLastName()).orElseThrow(() -> new ServiceException(ErrorCode.PARAMETER_EMPTY));
        Optional.ofNullable(dto.getEmail()).orElseThrow(() -> new ServiceException(ErrorCode.PARAMETER_EMPTY));

        if(!ValidateUtil.isValidEmail(dto.getEmail())) {
            throw new ServiceException(ErrorCode.PARAMETER_EMPTY);
        }
    }
}
```
위와 같이 @Aspect 어노테이션으로 AOP를 사용할 것이라고 스프링에 알려주고 createUser라는 method를 만든다.<br>
UserService의 createUser 전에 이 메소드를 항상 호출하고 싶으므로 @Before("execution(* com.sol.solapp.user.service.UserService.createUser(..)) && args(dto)") 와 같이 어노테이션을 달아준다.<br>
@Before은 ~전에 호출하라는 의미이고, excution은 ~전에의 목적이 되는 method를 의미한다(패키지명부터 method명까지 모두 써줘야한다). args는 해당 method도 들어오는 parameter들을 써주면 된다.

이렇게 하면 UserServiceImpl 외부에서 createUser를 호출하면 UserServiceValidator의 createUser부터 실행된다. 주의할 점은 같은 클래스 내에서 호출할 때에는 적용되지 않는다라는 것이다.

# 정리

이처럼 AOP를 이용하면 더 가독성 있는 코드를 만들 수 있고 유지보수도 쉬워진다. 어떻게 보면 사실상 프록시 패턴을 쉽게 사용할 수 있도록 만들어졌다고도 할 수 있을 것 같다.

다음 포스팅에서는 더욱 다양한 어노테이션과 활용할 수 있는 기능들을 알아보겠다.

