---
layout: post
title: Item8- finalizer와 cleaner 사용을 피하라
subtitle: Effective Java
categories: java
tags: [JAVA]
---
## finalizer와 cleaner 사용을 피하라

키워드 : finalizer, cleaner

### finalizer
1. 예측할 수 없고 상황에 따라 위험할 수 있어서 일반적으로 불필요 (오작동, 낮은 성능, 이식성 문제의 원인)
2. Java 9부터 deprecated API로 지정되어 cleaner를 대안으로 사용.


### cleaner
1. finalized 보다는 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요함 
```java
cleaner.register(this, state);
cleaner.clean();
```

문제점 
- finalizer와, cleaner는 실행 시점을 예측할 수 없기 때문에 제때 실행되어야 하는 작업에는 사용할 수 가 없다. (ex. 파일열기)
- finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.
- finalizer를 사용한 객체를 생성하고 파괴할 때 AutoCloseable 객체를 생성하고 try-with-resources로 닫는 것보다 시간이 수십 배 더 걸릴 수 있어서, GC효율을 떨어트린다.
- 생성자나 직렬화 과정에서 예외가 발생하면 생성되다 만 객체의 하위 클래스의 finalizer가 수행될 수 있다. (피하려면 final 사용)

보완
- AutoCloseable을 구현하고, 인스턴스를 다 사용하고 나면 close 메서드를 호츨해준다. 하지만 예외 발생시 무시 되는 것을 막기 위해 
  일반적으로  try-with-resources 를 사용한다.  (아래 예시)

언제쓰나
- 자원의 소유자가 close 메서드를 호출하지 않는 다음과 같은 자바 라이브러리들
  - FileInputStream, FileOutputStream, ThreadPoolExecutor
- 네이티브 피어(native peer)와 연결된 객체에서 사용
  - 네이티브 피어: 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체. (자바 객체가 아니기 때문에 GC가 알지 못함)
  (https://stackoverflow.com/questions/48260485/what-is-a-native-peer)


예시
```java
import java.lang.ref.Cleaner;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Room implements AutoCloseable {
  private static final Cleaner cleaner = Cleaner.create(); //cleaner 생성
  private final State state;  // 방의 상태
  private final Cleaner.Cleanable cleanable;

  public Room(int numJunkPiles){
    state = new State(numJunkPiles);
    cleanable = cleaner.register(this, state);
  }

  @Override
  public void close(){
    cleanable.clean();
  }

  private static class State implements Runnable {
    int numJunkPiles; //쓰레기 수
    State(int numJunkPiles) {
      this.numJunkPiles = numJunkPiles;
    }

    @Override
    public void run() {
      log.info("방 청소");
      numJunkPiles = 0;
    }
  }
}
```

- try-with-resources (cleaner의 역할은 단순 안전망 ... )
```java
package com.workbook.crane.common.configuration;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Adult {

  //Audult
  public static void main(String[] args) {
    try (Room room = new Room(7)) { //방청소 출력
      log.info("안녕~");//안녕 출력
    }
  }
}
```

- cleaner만 믿기
```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Teenager {

  public static void main(String[] args) {
    new Room(99); //방청소 출력이 되지 않음 (명시적으로 room.close()를 호출해야면 해준다. 
    System.out.println("아무렵"); //아무렴 출력
  }
}
```

