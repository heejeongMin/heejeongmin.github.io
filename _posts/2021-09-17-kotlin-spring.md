---
layout: post
title: Kotlin + Spring Boot 시작하기
subtitle: Kotlin + Spring Boot 시작하기
categories: kotlin
tags: [KOTLIN]
---

Kotlin의 프레임워크로 Spring boot 를 선택할 수도 있고, 
Ktor를 선택할 수도 있다. 가벼운 어플리케이션은 Ktor를 사용할 수 있지만 
어플리케이션이 크거나, 향후 인테그레이션을 생각하면 Spring을 사용하는 것을 추천한다고 한다. 


Spring Initialzr 활용하여 프로젝트를 시작할 수 있다. 

![SpringInitializr1.png](/assets/images/kotlin/SpringInitializr1.png)

![SpringInitializr2.png](/assets/images/kotlin/SpringInitializr2.png)
language를 그동안은 Java로 선택해 왔는데, Kotlin을 선택하여 준다. 

![SpringInitializr3.png](/assets/images/kotlin/SpringInitializr3.png)


생성된 프로젝트 구조를 보면, main/java 가 아니라 이제 main/kotlin인 것을 볼 수 있다. 
파일의 확장자도 .kt 이다. 
 
![projectStructure.png](/assets/images/kotlin/projectStructure.png)


gradle 파일을 살펴보려면, build.gradle.kts 를 보면 되는데, 
의존성에 kotlin을 위한 의존성이 추가된 것이 보인다. 

![projectStructure-gradle.png](/assets/images/kotlin/projectStructure-gradle.png)


DemoApplication을 보면, 자바는 public static main 메서드를 찾지만, kotlin의 경우 
DemoApplciation클래스 밖에 있는 최상위 function을 시작점으로 찾게된다. 

![projectStructure-startingPoint.png](/assets/images/kotlin/projectStructure-startingPoint.png)


