---
layout: post
title: Kotlin Ktor로 간단한 HTTP 목서버 만들기
subtitle: Kotlin + Ktor
categories: kotlin
tags: [KOTLIN, KTOR]
---

연동 테스트를 하고 싶은데 아직 연동을 하지 못하는 경우가 있어서 간단히 목서버를 만들어서 다음을 검증하고 싶었다. 
1. Request 가 잘 가는지 
2. Response 가 내가 만든 객체로 잘 받아지는지 
3. 응답을 받은 이후 로직이 잘 수행이 되는지 

python으로 목서버를 만드는 경우도 보았는데, 그동안 관심이 있었던 코틀린을 공부겸 활용해보고자 Kotlin + Ktor를 활용하여 목서버를 만들어보았다. 
사용 IDE는 인텔리제이이다. 

1. Spring은 Spring Initializer가 있는데 코틀린도 Ktor Project Generator라는 것이 있다. 
    1. ktor.io 사이트에 접근 
        - https://start.ktor.io
    2. Project 이름을 정하고 세팅을 한다. 
        - Gradle을 Groovy가 아닌 코틀린으로 작성이 가능하다고 하여 궁금하여 코틀린으로 선택
        - 엔진도 Apache가 아닌 Netty 
        - plugin에 Routing도 일단 추가
        ![ktor.io.png](/assets/images/kotlin/ktor.io.png)
    3. 생성을 마치고 나면 zip파일이 다운로드 되는데 원하는 경로에 풀고 인텔리제이로 띄우면 다음과 같은 프로젝트 구조와 
       코틀린으로 생성된 build.gradle을 볼수 있다. 파일 확장자는 .kt이다. 
        ![project_structure.png](/assets/images/kotlin/project_structure.png)

    4. Application.kt & Routing.kt
        - embeddedServer 에서는 코드 안에서 파라미터로 서버정보를 받아서 빨리 실행해 볼수 있게 해주는데, Netty로 이루어져있다고 한다. 
          (https://ktor.io/docs/create-server.html#embedded-server)
        - configureRouting 은 Routing.kt 에 구현체를 바라보게 되어 있다. 
        ![Application_Routing.png](/assets/images/kotlin/Application_Routing.png)
