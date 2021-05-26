---
layout: post
title: 스프링 시큐리티(Spring Security) - 기본 개념
description: 스프링 프로젝트들 중 최고난도를 자랑하는 스프링 시큐리티를 시작해보자
tags: 
    - Spring
    - Spring Security
    - Backend
author: Dev. Sol
category: SPRING
---

새로운 프로젝트에 들어가거나 새로 개발을 시작하다보면 항상 기본적으로 구현해야할 내용이 인증과 인가이다.<br>
스프링 시큐리티(Spring Security)는 이러한 인증, 인가 등의 보안을 담당하는 스프링 하위 프레임워크이다.<br>
하지만 해당 내용은 이미 구현이 되어있거나 해당 내용을 잘 아는 개발자가 홀로 담당하는 경우가 많아 한번도 구현해보거나 깊게 들여다본 경우가 없었기에<br>
이번에 제대로 공부해보고 싶은 열망에 포스팅을 하게 되었다.


# 용어 정리
> - 접근 주체(Principal) : 보호된 리소스에 접근하는 대상
> - 인증(Authentication) : 보호된 리소스에 접근한 대상에 대해 누구인지, 애플리케이션의 작업을 수행해도 되는 주체인지 검증하는 과정 => 넌 누구냐!
> - 인가(Authorize) : 해당 리소스에 대해 접근 가능한 권한을 가지고 있는지 확인하는 과정(After Authentication, 인증 이후)  => 니가 할수있는게 뭐냐!
> - 권한 : 어떠한 리소스에 대한 접근 제한, 모든 리소스는 접근 제어 권한이 걸려있음. 인가 과정에서 해당 리소스에 대한 제한된 최소한의 권한을 가졌는지 확인

<br><br>

# 동작 원리
## 아키텍쳐
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FUOabX%2FbtqEJBBNixH%2FPGDv64FTKaBSLzMiiXkA3K%2Fimg.png">

- 스프링시큐리티는 서블릿 필터 체인을 자동으로 구성하고 요청을 거치게 한다. 필터들이 굉장히 많다... 그 중에 필요한 필터들만 사용한다.


## 로그인 인증구조(서버기반 인증)
1. Http 요청이 들어오면 AuthenticationFilter가 http 서블릿 리퀘스트에서 사용자가 보낸 정보를 인터셉트한다.
2. UsernamePasswordAuthenticationToken을 만들어서 인증을 담당할 AuthenticationManager 인터페이스에 위임한다.
3. AuthenticationManager는 아이디 비밀번호 등의 유저 정보를 처리하여 Authentication 객체를 만들고 AuthenticationProvider에서 실제 인증이 일어나게된다.
4. 실제로 인증할 AuthenticationProvider에게 Authentication 객체를 전달하면 UserDetailsService에서 Authentication을 가지고 DB에서 UserDetails라는 객체로 꺼내서 유저 세선을 생성한다.
5. 이렇게 나온 UserDetails를 스프링 시큐리티의 인메모리 저장소인 SecurityContextHolder에 저장을하고 User 세선아이디와 함께 응답을 보낸다.
6. 이후에 요청 쿠키에서 JSESSIONID를 검증 후 유효하면 인증을 하게된다.
<br><br>

# 인증 방식
## 1. 서버 기반 인증
### 작동 원리
- 클라이언트: 서버에게 로그인 요청
- 서버: 세션 생성 및 유지 & 응답


### 인증 구조
<img src='/assets/images/sol/20210525/session_diagram.png'>
1. 로그인 요청
2. 서버에서 세션ID 생성 후 세션을 등록
3. 브라우저에 등록된 세션값 전달
4. 브라우저 쿠키에 세션값 저장
5. 매 요청시 헤더 쿠키에 세션값 담아 요청 (세션값으로 인증)


### 문제점
- 세션은 유저가 인증할 때 이 기록을 서버에 저장한다. 이 경우 유저의 수(동시 접속)가 많으면 서버나 DB에 부하가 큼
- CORS(Cross-Origin Resource Sharing) 쿠키를 여러 도메인에서 관리하는 것이 번거로움
<br>

## 2. 토큰 기반 인증(JWT)
### 배경지식
#### C.I.A
- 기밀성 (Confidentiality) : 정보를 오직 인가된 사용자에게만 허가
- 무결성 (Integrity) : 부적절한 정보 변경이나 파기 없이 정확하고 완전하게 보존
- 가용성 (Availability) : 시기적절하면서 신뢰할 수 있는 정보로 접근과 사용

#### RSA 암호화
- 암호화 - 공개 키(Public Key), 복호화 - 개인키 (Private Key)
- 전자 서명 - 개인 키(Private Key), 서명 검증 - 공개 키(Public Key)

<img src='/assets/images/sol/20210525/rsa_diagram'>

### 작동 원리
- 유저가 아이디와 비밀번호로 로그인을 수행
- 서버측에서 해당 정보 검증
- 계정 정보가 정확하다면 서버측에서 유저에게 signed 토큰을 발급
- 클라이언트에서는 토큰을 저장해두고 요청마다 토큰을 서버에 함께 전달
- 서버에서 토큰을 검증하고 요청에 응답
- 토큰은 HTTP 헤더에 포함시켜서 전달
<br><br>

### 장점
> - 무상태(stateless)
> - 보안성
> - 쿠키를 사용하지 않음
> - 확장성(Extensibility)
>> - 기능 확장 가능
>> - token의 body에 내용을 담을 수 있다. -> 기능 확장
> - CORS 이슈를 해결할 수 있다.(토큰만 유효하면 어디서나 요청이 정상적으로 처리되므로)
> - 웹 표준 기반(JWT <- 표준 토큰)

<br><br>


# Reference 
- https://devuna.tistory.com/55
- https://mangkyu.tistory.com/77
- https://hororolol.tistory.com/420
