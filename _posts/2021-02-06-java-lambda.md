---
layout: post
title: Java 8 Lambda
categories: java
tags: [JAVA]
---

### 함수형 인터페이스

- 추상메서드를 딱 하나만 가지고 있는 인터페이스
- @FunctionalInterface 어노테이션을 가지고 있는 인터페이스

    Static & Default Method
    인터페이스에 추상메서드 외에 Static, Default 메서드를 사용할 수 있게 되었다. Default 메서드의 큰 장점은 해당 인터페이스를 구현하는 구현체에서는 필수로 override하지 않아도 된다는 점이다. 

    자바의 함수형 프로그래밍
    - 함수를 일급객체로 사용할 수 있다. (First-class citizen) 
       : 객체를 파라미터로 전달, 반환, 변수에 담을 수 있다. 
       : 자바의 Method는 일급객체가 아니지만 람다는 일급객체이다. 
    - 상태가 없다.
       : 함수 밖의 값을 사용하지도 않고, 그 값을 변경하지도 않는다. 

- 기본제공 함수형 인터페이스
    - Function<T, R>
    - BiFunction<T, U, R>
    - Consumer<T>
    - Supplier<T>
    - Predicate<T>
    - UnaryOperator<T>
    - BinaryOperator<T>

### 람다 표현식

- 표현식
    - () → {}
    - 인자가 1 개인 경우는 () 생략 가능
    - 인자 타입 생략 가능. 컴파일러가 추론
    - 바디가 한줄인 경우 {} 및 return 키워드 생략 가능
- 변수캡쳐
    - final이거나 effective final 인 경우만 참조한다. (concurrency 문제 방지)
    - 익명 클래스는 새로 스콥을 만들지만 람다는 람다를 감사고있는 스콥과 같아 쉐도잉 하지 않는다.

### 메소드 레퍼런스

- 객체레퍼런스::인스턴스메서드
- 타입::스태틱메서드
- 타입::인스턴스 메서드
- 타입::new