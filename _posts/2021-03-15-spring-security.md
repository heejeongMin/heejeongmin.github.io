---
layout: post
title: 스프링 시큐리
categories: spring
tags: [SPRING, SPRING MVC]
---

### Spring Security? 

---

#### 인증(Authentification)과 인가(Authorization) 기능을 제공하는 프레임워크
    
   - 인증(Authentification) : 유효한 사용자인지 확인
   - 인가(Authorization) : 유효한 사용자가 수행할 수 있는 권한을 확인


#### Spring Security의 필터기반 동작 

Spring Security에서 FilterChainProxy를 제공하는데 해당 프록시가 을 통해 인증을 위한 여러 필터들을 사용할 수 있다.
(참고 : https://docs.spring.io/spring-security/site/docs/5.4.5/reference/html5/#servlet-architecture)

![Spring_security_filter_chain](/assets/images/spring/Spring_security_filter_chain.png)


#### Spring Security의 중요 컴포넌트들

- **SecurityContextHolder** : 인증된 사용자들의 정보를 담고 있음. ThreadLocal을 default 로 사용하게 되어 있어, 동일 thread에서 SecurityContext가 항상 같은 사용자의 정보를 내보낼 수 있도록 되어 있다.
                                          FilterChainProxy에서 자체적으로 요청된 작업이 끝나면, SecurityContext를 비우는 작업을 하고 있다. 
- **SecurityContext** : 현재 인증된 사용자의 인증 정보를 가지고 있으며, SecurityContextHolder에서 가지고 올 수 있다. 
- **Authentification** : 사용자의 로그인 정보를 가지고 있는 컴포넌트로 SecurityContext에서 현재 사용자에 대한 정보를 가지고 올 수 있다. 
- **AuthentificationManager** : Spring Security 필터가 인증을 어떻게 하는지 정의한 API

![Spring_security_context](/assets/images/spring/Spring_security_context.png)

참고 자료 : https://docs.spring.io/spring-security/site/docs/5.4.5/reference/html5/#servlet-authentication-abstractprocessingfilter


## Spring Security + OAuth2 + JWT

---

Spring Framework에서 제공하는 OAuth2를 이용하여 인증을 위한 Authorization Server를 구헝하고, 인증을 위한 토큰은 JWT 토큰 방식으로 처리 가능하다. 

- OAuth2 : 인증을 위한 산업표준 프로토콜
참고 자료 : https://oauth.net/2/
참고 자료 : https://developer.okta.com/blog/2019/08/22/okta-authjs-pkce/?utm_campaign=text_website_all_multiple_dev_dev_oauth-pkce_null&utm_source=oauthio&utm_medium=cpc

    | OAuth 이전 token이 노출되는 이전의 문제점 | OAuth 표준 인증을 통한 강화된 인증처리 |
    | :---- | :---- |
    | ![Spring_security_token_exposure_problem](/assets/images/spring/Spring_security_token_exposure_problem.png) | ![Spring_security_enhanced](/assets/images/spring/Spring_security_enhanced.png)  |
   


- JWT : JSON Web Token의 약자로, 정보를 Json 오브젝트로 안전하게 전송하는 역할로 주로 사용자 인증에 사용된다. 
참고 자료 : https://jwt.io/introduction


- Spring Security structure (https://derekpark.tistory.com/42)
![Spring_security_structure](/assets/images/spring/Spring_security_structure.png)