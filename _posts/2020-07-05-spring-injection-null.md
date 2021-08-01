---
layout: post
title: 주입받은 객체가 왜 null일까? 
categories: spring
tags: [SPRING, SPRING MVC]
---

스프링 빈이 아닌 객체에 스프링빈을 사용하여야 하면, 스프링 컨텍스트에서 관리하지 않기 때문에 주입으로는 해결할 수가 없다. 
그래서 스프링에서 주입이 된 bean을 들고 올 수 있게 ApplicationContext를 사용할 수 있다.  

​
1. 문제 코드

- 아래처럼 MutlThread라는 객체에서 BeanOne을 주입받아 호출을 하면 NullpointerException이 발생한다. 
MultiThread를 제외하고 모두 스프링 빈인 상태이다. (@RestController, @Component) 

![Spring_bean_null](/assets/images/spring/Spring_bean_null.png)


2. 해결 방법

- 왼쪽처럼 ApplicationContextAware를 구현하는 BeanUtil이라는 클래스를 하나 만들어서, 스프링빈을 들고 오게 하는 방법이 있다.


​![Spring_beanutil](/assets/images/spring/Spring_beanutil.png)


​

ApplicationContextAware 설명

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContextAware.html

 
ApplicationContextAware (Spring Framework 5.2.7.RELEASE API)
All Superinterfaces: Aware All Known Implementing Classes: AbstractAtomFeedView , AbstractCachingViewResolver , AbstractController , AbstractDetectingUrlHandlerMapping , AbstractFeedView , AbstractHandlerMapping , AbstractHandlerMapping , AbstractHandlerMethodAdapter , AbstractHandlerMethodMapping ,...

docs.spring.io

Interface to be implemented by any object that wishes to be notified of the ApplicationContext that it runs in.