---
layout: post
title: Item7 - 다 쓴 객체의 참조를 해제하라
subtitle: Effective Java
categories: java
tags: [JAVA]
---
## 다 쓴 객체의 참조를 해제하라
키워드 : Obsolete Reference, Garbage Collection, Memory Leak
1. 
2. 정의
   1. Obsolete Reference : 다시 참조 되지 않을 객체 
   2. Garbage Collection : Obsolete Reference 상태인 객체를 회수함. 
   3. Memory Leak : 메모리가 할당 되었지만, 더 이상 메모리가 필요없는 객체에 대한 메모리가 릴리즈 되지 않는 현상
   
3. 예시
   ```java
   public class Stack {
       private Object[] elements; // !!전역변수!! 
       private int size = 0; // !!인덱스 용도!!
       private static final int DEFAULT_INITIAL_CAPACITY = 16;
   
       public Stack(){
           elements = new Object[DEFAULT_INITIAL_CAPACITY];
       }
   
       public void push (Object e) {
           ensureCapacity();
           elements[size++] = e; // !!0번에 넣은 후 size 1로 증가!!
       }
   
       public Object pop() {
           if (size == 0)
              throw new EmptyStackException();
           return elements[--size]; // !!size 0번으로 변경 후 0번 객체 꺼냄!!
        }
   
       /**
        * 원소를 위한 공간을 적어도 하나 이상 확보한다.
        * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
        */
        private void ensureCapacity() {
           if (elements.length == size)
               elements = Arrays.copyOf(elements, 2 * size + 1);
           }
    }
    ```
   1. 위의 예시가 obsolete reference를 만들어 내는 이유
      1. Object[] elements 가 클래스의 인스턴스 변수로 관리되고 있다. 
         그래서 Stack이라는 인스턴스가 생성될때 heap 메모리에 올라가게 된다. 
         ![java_variable_memory.png](/assets/images/java/java_variable_memory.png)

      2. heap 메모리가 올라가면 올라가면 원래는 GC 대상이지만, 위의 예시에서는 pop을 하면서 element를 반환하지만 
         Object[] elements 배열에서 해당 객체 참조를 여전히 가지고 있기 때문에 가비지 컬렉터가 회수하지 못하게 된다. 
         반면 size (인덱스)는 이미 줄어들었기 때문에 더 이상 참조 되지 않게 되면서 obsolete reference 로 남게 되지만,  
         GC가 회수 하지 않기 때문에 메모리 누수로 이어진다.

4. 해결방법
   1. 불필요하게 메모리가 할당 되어 있는 객체의 참조 끊기
      ```java
       public Object pop() {
          if (size == 0)
             throw new EmptyStackException();
          Object result = elements[--size];
          elements[size] = null; //다 쓴 객체의 참조 해제
          return result;
       }
      ```
      - 만약 null처리를 한 객채를 참조를 시도하면 NullPointerException이 발생하게 되는데, 이는 참조 해제를 하기 전에 해당 객체를 참조함으로써 
        의도하지 않은 잘못 된 값을 반환하게 하는 것보다 프로그램의 오류를 조기에 발견할 수 있게 해준다. 
      - 하지만 객체의 참졸르 null 처리 하는 것은 예시 처럼 메모리르 직접관리하는 예외적인 케이스에만 해당되고, 일반적으로 프로그래머가 일일히 null처리 해줄 필요는 없다.  
        다만, 자연스럽게 참조 해제를 하는 가장 좋은 방법은 참조를 담은 변수의 유효범위를 고려하는 것이다.   
        예를 들어 메서드 영역에 선언된 지역변수는 메서드 호출이 끝나고 나면 모두 GC 대상이 된다.  

### 캐시
- 메모리 누수를 일으키는 주범. 보통 Map을 의미하는데 캐시에 넣어두고 지우는 것을 잊는 경우가 많기 때문이고, 프로그래머 대신 자동으로 비워주는 WeakHashMap을 쓸 수 있다.
- 숙제 : hard/soft/weak/phantom reference

---
참고 :  
https://stackoverflow.com/questions/6843102/what-is-meant-by-obsolete-reference-in-java#:~:text=An%20obsolete%20reference%20is%20simply,termed%20as%20unintentional%20object%20retention.

https://k9e4h.tistory.com/389

https://d2.naver.com/helloworld/329631

https://en.wikipedia.org/wiki/Memory_leak

https://www.youtube.com/watch?v=YijcBaS4cu8