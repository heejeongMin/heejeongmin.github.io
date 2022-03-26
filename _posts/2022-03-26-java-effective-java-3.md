---
layout: post
title: Item3 - private 생성자나 열거 타입으로 싱글턴임을 보장하라
subtitle: Effective Java
categories: JAVA
tags: [JAVA]
---
## private 생성자나 열거 타입으로 싱글턴임을 보장하라

- 싱글턴(singleton) 
  - 정의 : 인스턴스를 오직 하나만 생성할 수 있는 클래스 
  - 생성 
    1. 생성 자는 private 으로 감춰두고, public static 멤버가 final필드인 방식
       - private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한번 호출된다.
       - API에 명백히 해당 객체가 싱글턴임이 드러난다. 
          ```java
           public class Elvis {
             public static final Elvis INSTANCE = new Elvis();
             private Elvis() { ... }
        
             public void leaveTheBuilding() { ... }
           }
          ```
       
    2. 생성자는 private 으로 감춰두고, 정적 팩터리 메서드를 public static으로 제공하는 방식
       - Elvis.getInstance는 항상 같은 객체를 참조
       - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다. 
       - 참조 공갑자로 사용할 수 있다. (메서드 레퍼런스)
       ```java
        public class Elvis {
          private static final Elvis INSTANCE = new Elvis();
          private Elvis() { ... }
          public static Elvis getInstance() { return INSTANCE; }
       
          public void leaveTheBuilding() { ... }
        }
       ```
    3. 열거 타입 방식의 싱글턴을 만드는 방식
       - 더 간결하고, 추가 노력 없이 직렬화 할 수 있고 리플렉션 공격에도 안전하다. 

       ```java
        public enum Elvis {
          INSTANCE;
        
          public void leaveTheBuilding() { ... }
        }
       ```
  - 1,2번 방식의 주의 할 점 
      - 보장할 수 없는 딱 한가지 예외상황이 리플렉션을 통해서 setAccsible(true) 로 들어오는 경우로,
        문제 상황을 피하려면 두 번째 객체 생성 시도시 예외를 던지게 해야한다. 
      - 직렬화 시 Serializable만 구현한다고 되지 않으며, readResolve메서드를 구현해야 한다. 
        그렇지 않으면 직렬화된 인스턴스를 역직렬화 할때 새로운 인스턴스가 만들어진다.
        ```java
        private Object readResolve(){
          return INSTANCE;
        }
        ```
  - 3번 방식의 주의 할 점
    - 만들려는 싱글턴이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다. 
