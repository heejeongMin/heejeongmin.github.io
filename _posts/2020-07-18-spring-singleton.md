---
layout: post
title: 스프링 싱글톤 사용시 주의하기
categories: spring
tags: [SPRING, SPRING MVC]
---


스프링의 빈은 싱글톤으로 기본 작동하기 때문에 빈에 전역 변수가 있으면 스레드에 걸쳐 값이 공유가 되게 된다. 
아래 참고한 사이트 들을 기반으로 빈의 Scope을 달리하여 각각 Singleton, Prototype, Request, Session Scope을 테스트 해보았다.

- 참고한 사이트
    
        https://www.baeldung.com/spring-bean-scopes
        https://stackoverflow.com/questions/13802636/bean-marked-with-prototype-scope-not-working-in-spring



## 싱글톤(Singleton) 빈 호출 시 전역 변수 vs 지역 변수 비교 

   - 싱글톤은 항상 사용하던데로 아무런 옵션 없이 사용 가능하다. 
   왼쪽 로그에 보면 BeanService는 호출 시 마다 항상 같은 객체가 반환되는 것을 볼 수 있다. 
   또 cnt는 전역 변수이고, anotherCnt는 지역변수 인데, 누적을 했을 시 anotherCnt는 유지가 되지만 cnt는 계속 변경이 되는 것을 볼 수 있다. 

![Spring_singleton](/assets/images/spring/Spring_singleton.png)


## 프로토타입(Prototype) 빈 호출 시 전역 변수 vs 지역 변수 비교

   - 프로토타입은 new를 해서 사용하는 것처럼 계속 새로운 객체로 사용 할 수 있음 
   - 프로토 타입으로 등록할때 그냥 @Scope("prototype")으로 등록하면 작동을 안해서 더 찾아보니 이렇게 사용하여야한다고 함 : @Scope(value = "BeanDefinition.SCOPE_PROTOTYPE, proxyMode = ScopedProxyMode.TARGET_CLASS)
   - 백기선님의 인프런 스프링 프레임워크 강좌에서는 이 프로토타입으로 등록을 하면 빈을 프록시가 감싸게 되고, 우리는 프록시를 통해서 Bean을 호출하게 되는 것이라고 한다. 나는 프록시 개념을 잘 모르는데 공부해야겠다. 
   - 로그 오른쪽에 보는 것 처럼 BeanService는 항상 다른 메모리 주소값을 가지고 있는 것으로 보아 계속 다른 객체가 반환되게 되고, 그 안에 있는 전역변수 또한 항상 내가 넣은 값으로 유지가 되는 것을 볼 수 있다.

![Spring_prototype](/assets/images/spring/Spring_prototype.png)

- Singleton과 Prototype의 Bean 라이프사이클 차이 
   
    | test | Singleton | Prototype | 
    | :---- | :---- | :---- |
    | Bean요청시   | 스프링이 ApplicationContext에 존재하는 Bean을 반환해준다 | 스프링이 초기화하여 Bean을 반환해 준다  |
    | 사라지는 시점   | ApplicationContext가 destory될때 사라진다    | GC 가 일어날때 사라진다.  |
   

- 참고 사이트(테스트까지 잘되어있어 참고하기 좋았음) : https://nullbeans.com/prototype-vs-singleton-spring-beans-differences-and-uses/#Prototype_beans


## 리퀘스트스콥(RequestScope) 빈 호출 시 전역 변수 vs 지역 변수 비교

- 리퀘스트마다 새로운 객체임
- @RequestScope 어노테이션 혹은 
    @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS) 로 사용 가능
​- 로그 오른쪽에 보는 것 처럼 BeanService는 항상 다른 메모리 주소값을 가지고 있는 것으로 보아 계속 다른 객체가 반환되게 되고, 그 안에 있는 전역변수 또한 항상 내가 넣은 값으로 유지가 되는 것을 볼 수 있다.

![Spring_request_scope](/assets/images/spring/Spring_request_scope.png)

## 세션스콥(SessionScope) 빈 호출 시 전역 변수 vs 지역 변수 비교

- 동일한 세션인 경우에만 같은 Bean을 사용하게 된다.
- @SessionScope 어노테이션 혹은 @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS) 로 사용 가능
- Session을 두개로 테스트 하기 위해 Chrome 에서 먼저 3번 호출하고, 그 다음 IE에서 1번 호출하였다. 
- 로그 오른쪽에 보는 것 처럼 빨간색 안에 들어있는 BeanService는 같은 메모리 주소를 갖고 있고, cnt도 누적이 되고 있는 걸 볼 수 있다. 파란색 안에 들어 있는 BeanService는 Session이 다른 곳이서 호출이 되었기 때문에 메모리 주소가 다른 새로운 객체가 반환이 된것을 볼 수있고, cnt도 초기화가 되었다.

![Spring_session_scope](/assets/images/spring/Spring_session_scope.png)
