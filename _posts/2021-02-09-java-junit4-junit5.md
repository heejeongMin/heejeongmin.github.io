---
layout: post
title: Junit4에서 Junit5
categories: java
tags: [java, junit]
---

### Junit4에서 Junit5로 넘어갈때 어떤 변화 ?

- Junit5는 다음과같이 세가지 모듈로 구성되어 있고, JUnitPlatform 위에 Jupiter를 통해서 API를 제공받는다. Junit5에서도 Junit4는 기본적으로 동작하는데, 그 이유는 JUnit Platform 위에 Vintage 구현체를 통해서 가능하다. 
    - JunitPlatform : 테스트를 실행해주는 런처 및 TestEngine API 제공
    - Jupiter : TestEngine API의 구현체로 JUnit 5 제공
    - Vintage : Junit 4와 3을 지원하는 TestEngien 구현체
    - 다음은 실제 Junit 5와 Junit4를 함께 사용했을때 구현체를 서로 달리 사용하는 것을 보여 주는 예제이다. 
        
        → StudyTestJunit4 테스트 클래스에서는 org.junit.Test 사용
        
        → StudyTest 테스트 클래스에서는 org.junit.jupiter.api.Test 사용
        
        ![Java_junit5](/assets/images/java/Java_junit5.png)


   - 스프링 2.2.0 부터 Junit5에 대한 의존성을 org.springframework.boot:spring-boot-starter-test:2.2.0.RELEASE를 통해 기본으로 제공한다. 
  
      ![Java_junit5_dependency](/assets/images/java/Java_junit5_dependency.png)

   - Junit5로 오면서 기존 어노테이션에 변화 및 추가가 있었는데, 기존 어노테이션을 굳이 변경하지 않아도 테스트는 위와 같은 이유로 작동한다. 

        | Junit 4 | Junit 5 | 기능 |
        | :---- | :---- |  :---- |
        | @Before | @BeforeEach | 모든 테스트 인스턴스 실행 전 마다 실행  |
        | @BeforeClass | @BeforeAll | 모든 테스트 인스턴스 실행 전 최초 한번 실행 |
        | @After | @AfterEach | 모든 테스트 인스턴스 실행 후 마다 실행 |
        | @AfterClass | @AfterAll | 모든 테스트 인스턴스 실행 후 최후 한번 실행  |
        |  | @DisplayName | 테스트 인스턴스의 이름 설정  |
        | @Ignore | @Disabled | 테스트 인스턴스 실행하지 않고 무시  |
        | @Category(Class) | @Tag(String) |   |
        | TestRule 구현 클래스 사용 | @RepeatedTest | 테스트의 반복 횟수를 정할 수 있다. |
        | ^^ | ^^ | ^^ ([Junit 4 참고](https://turreta.com/2017/07/15/junit-4-run-test-method-more-than-once/)) |
        | @RunWith(Parameterized.class) | @ParameterizedTest | |
        | ^^ @Parameters, @Parameter 조합 | ^^ | ^^ |
        | ^^ JunitParam 라이브러리 사용 | ^^ | ^^  |

- JunitParam 라이브러리 참고 : https://www.baeldung.com/junit-params
- JunitParam 라이브러리 참고 : https://www.baeldung.com/junit-params
- Junit4 파라미터 사용 참고 : https://code-mentor.com/blog/2016/data-injection-with-junit.html
- Junit5 ParameterizedTest를 사용하면서 함께 사용할 수 있는 인자에 대한 어노테이션
    - @ValueSource
    - @NullSource, @EmptySource, @NullAndEmptySource
    - @MethodSource
    - @CvsSource
    - @CvsFileSource
    - @ArgumentSource
- 파라미터로 넘어온 값들을 객체로 변환하고 싶다면
    - SimpleArgumentConverter를 상속받은 static nested class를 만들어서, @Conver With 어노테이션에 알려줄 수 있다.
    - ArgumentAccessor를 테스트 인스턴스의 인자로 사용하고 하나씩 꺼내와서 변환할 수도 있고,
    - ArgumentsAggregator를 구현한 static nested class를 만들어서 ‘@AggregateWith 의 어노테이션에 알려줄 수도 있다.