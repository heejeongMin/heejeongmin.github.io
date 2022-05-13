---
layout: post
title: Item1 - 생성자 대신 정적 팩터리 메서드를 고려하라
subtitle: Effective Java
categories: java
tags: [JAVA]
---
## 생성자 대신 정적 팩터리 메서드를 고려하라

### 인스턴스 얻는 방법
1. public 생성자
    ```java
   public Dog (String name) {
      this.name = name;
   }
    ```
2. 정적 팩터리 메서드 (static factory method)
    ```java
       public static Boolean valueOf(boolean b) {
          return b ? Boolean.TRUE : Boolean.FALSE;
       }
   ```
 
  
### 정적 팩터리 메서드의 장점
#### 1) 이름을 가질 수 있다.
- 반환될 객체의 특성의 쉽게 묘사할 수 있다. 
      ```java
         BigInteger(int, int, Random) // 생성자
         BigInteger.probablePrime(int, Random) // 정적팩터리 메서드 -> 소수 
      ```
- 오버로딩 생성자를 만들어서 제한을 피할 수 있지만 다른 사람이 사용할때 무엇때문이지 알려면 문서를 보아야하고, 
  잘못 사용될 가능성이 있다. 
- 따라서, `한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 
  각각의 차이를 잘 드러내는 이름을 지어주자. `
   
#### 2)  호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
- 같은 객체가 자주 요청되는 상황이라면 성능 향상에 도움이 되며, 플라이웨이트 패턴(Flyweight pattern)과 유사. 
  ```text
   플라이웨이트 패턴 
   구조 패턴중의 하나로, 인스턴스가 필요할 때마다 매번 생성하는 것이 아니고 가능한 한 공유해서 사용하여 메모리를 절약하는패턴
   (있으면 사용하고 없으면 새로 만들기) 
   참고 블로그 : https://sorjfkrh5078.tistory.com/145
  ```
- 인스턴스 통제 클래스 (instance-controlled-class)
  - 싱글톤으로 사용 가능
  - 인스턴스화 불가하게 만들 수 있다. 
  - 불변 값 클래스에서 동일한 인스턴스는 단 하나뿐임을 보장 (a.equals(b))
  - 플라이웨이트 패턴의 근간
  - 열거 타입은 인스턴스가 하나만 만들어짐을 보장 (enum 클래스)
  
#### 3) 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
- API를 만들 때 이 유연성을 응용하면 구현 클래스는 공개하지 않고도 그 객체를 반환할 수 있어, API를 작게 유지 할 수 있다.
  ```java
    Collections.emptyList();
    Collections.emptyMap();
  ```
  - 프로그래머가 API를 사용하기 위해 익혀야 하는 개념의 수와 난이도를 낮춰줌. 

#### 4) 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
- 반환 타입이 하위 클래스이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
  ```java
    EnumSet.of(E);
  ```
  ![EnumSet](/assets/images/java/Java_EnumSet.png)


#### 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
```java
  public static Pizza create(Class clazz) {
    if(clazz == null || clazz == PepperoniPizza.class) {
      return new PepperoniPizza();
    } else if (clazz == HawaiianPizza.class) {
      return new HawaiianPizza();
    }
  }
```

### 정적 팩터리 메서드의 단점
#### 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 메서드를 만들 수 없다.
   ![Inheritance](/assets/images/java/Java_static_factory_method_not_inherited.png)
    - 정적 팩토리 메서드를 제외하고 상속이 가능한 모습
    - 하지만 일부러 오버라이딩 하지 못하게 의도하고 컴포지션을 사용하도록 하는 것이라면 장점이 될 수 있다. 

#### 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
- 생성자가 없으니, 사용자는 정ㅈ거 팩터리 메서드 방식 클래스를 통해 인스턴스화 하는 방법을 찾아야 한다. 
- 메서드 이름을 좀 더 명확하게 짓고, API문서를 통해 보완할 수 있다. 
  - from 
    - 매개번수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드 
    ```java
        Date date = Date.from(instance);
    ```
  - of
    - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
    ```java
        Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
    ```
  - valueOf
    - from, of의 더 자세한 버전
    ```java
        BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
    ```    
  - instance / getInstance
    - 매개변수로 명시한 인스턴스를 반환하지만 같은 인스턴스임은 보장하지 않는다. 
    ```java
        Dog pancho = Dog.getInstance(breed);
    ```    
  - create / newInstance
    - instance, getInstance와 유사하지만 매번 새로운 인스턴스를 생성해 반환 함을 보장
    ```java
        Object newArray = Array.newInstance(classObject, arrayLen);
    ```
  - getType
    - getInstance 같으나, 생성할 클래스가 아닌 다른 클래스에 팰터리 메서드를 정의할 때 사용. 
    - "Type"이 팩터리 메서드가 반환할 객체의 타입
    ```java
        FileStore fs = Files.getFileStore(path);
    ```
  - newType
    - newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드 정의시 사용
    - "Type"이 팩터리 메서드가 반환할 객체의 타입
    ```java
        BufferedReader br = Files.newBufferedReader(path);
    ```
  - type
    - getType, newType의 간결한 버전 
    ```java
        List<Complaint> litany = Collections.list(legacyLitany);
    ```

### 결론
- 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 
- 그럼에도 정적 팩터리를 사용하는게 유리한 경우가 더 많음으로 public 생성자를 제공하던 습관이 있다면 고치는 것이 좋다.