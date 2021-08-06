---
layout: post
title: Filter & Interceptor는 언제 작동할까 ?
categories: spring
tags: [SPRING, SPRING MVC]
---

### 작동순서 :  Filter → Interceptor → AOP

공통 프로세스 관리를 위해 스프링에서 사용할 수 있는 기능으로는 다음 3가지가 있다. 
하는 역할이 비슷한데 언제/어떻게 작동하는지는 다음과 같다. 

1. **Filter** :
DispatcherServlet 이전에 실행되며 스프링의 자원에 도달하기 이전에 처리된다. 주로 인증처리 / URL에 따른 처리 / HTTP 요청과 응답을 변경가능하다. 필터가 여러개인 경우, FilterRegistration에 등록하여 순서를 정할 수 있고, filterChain에 의해서 순서대로 실행된다. 

2. **Interceptor** :
스프링 컨텍스트내부에서 요청과 응답에 대해 처리하며, 스프링의 모든 빈객체에 접근할 수 있다. 주로 로그인체크 / 권한체크 / 실행시간 / 로그확인 등등에 사용된다. 
postHandle 을 통해서 response에 객체를 추가할 수는 있지만, filter와 다르게 interceptor 변경 작업을 할 수 없다.

3. **AOP** :
주로 로깅/트랜잭션/에러처리 등 비지니스단의 메서드에서 좀더 세밀하게 조정하고 싶을때

![Spring_filter_interceptor](/assets/images/spring/Spring_filter_interceptor.png)



- 참고 자료 : https://goddaehee.tistory.com/154
- 참고 자료 : https://javacan.tistory.com/tag/FilterChain
- 참고 자료 : http://blog.naver.com/PostView.nhn?blogId=kbh3983&logNo=220937324670&parentCategoryNo=&categoryNo=34&viewDate=&isShowPopularPosts=false&from=postView
- 참고 자료 : https://stackoverflow.com/questions/35856454/difference-between-interceptor-and-filter-in-spring-mvc





