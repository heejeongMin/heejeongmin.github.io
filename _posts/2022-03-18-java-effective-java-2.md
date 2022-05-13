---
layout: post
title: Item2 - 생성자에 매개변수가 많다면 빌더를 고려하라
subtitle: Effective Java
categories: java
tags: [JAVA]
---
## 생성자에 매개변수가 많다면 빌더를 고려하라

- 정적 팩터리와 생서자에 동일한 제약은 선택적 매개변수가 많다면 적절히 대응하기 어렵다는 점이고, 해결하기 위한 방법은 다음과 같다. 

### 점층적 생성자 패턴 (telescoping constructor pattern)  
- 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다. 
- 보통 사용자가 원치않는 매개변수까지 포함하기 쉬워, 그런 매개 변수에도 값을 지정해 주어야 한다. 
- 필드 수가 늘어나면 그 만큼 시그니처가 다른 생성자가 늘어나게 되고, 매개변수를 맞게 넣었는지도 주의해야 한다. 
- 실수로 매개변수의 순서를 바꿔서 넣는 경우 찾기 어려운 버그를 유바할 수도 있다. 

  ```java
    public class NutritionFacts {
  
      private final int servingSize;
      private final int servings;
      private final int calories;
      private final int fat;
      private final int sodium;
      private final int carbohydrate;

      private NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
      }
  
      private NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
      }
  
      private NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
      }
    
      private NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
      }
  
      private NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
      }
    }
  ```

### 자바빈즈 패턴 (JavaBeans pattern)
- 매개변수가 없는 생성자로 객체를 만든 후 세터 메새드를 호출하여 값을 설정하는 방법이다.
- 점층적 생성자 패턴과 달리 가독성은 올라가지만, 심각한 단점은 객체 하나를 만들기위해 여러 메서드를 호출하게 되며 완성되기 전까지 일관성이 보장되지 않는다. 
- DDD에서도 단순 setter는 지양하는 것을 권고하고 있음. 
```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```
```java
public class NutritionFacts {

  private int servingSize = -1;
  private int servings = -1;
  private int calories;
  private int fat;
  private int sodium;
  private int carbohydrate;

  private NutritionFacts() {
  }

  public void setServingSize(int servingSize) {
    this.servingSize = servingSize;
  }

  public void setServings(int servings) {
    this.servings = servings;
  }

  public void setCalories(int calories) {
    this.calories = calories;
  }

  public void setFat(int fat) {
    this.fat = fat;
  }

  public void setSodium(int sodium) {
    this.sodium = sodium;
  }

  public void setCarbohydrate(int carbohydrate) {
    this.carbohyrate = carbohyrate;
  }
}
```
### 빌더 패턴 (Builder pattern)
- 점층적 생성자 패턴의 안정성과 자바 빈즈 패턴의 가독성을 모두 가진 패턴.
- 파이썬과 스칼라에 있는 명명된 선택적 매개변수 (named optional parameters) 모방 
- 잘못된 매개변수를 일찍 발견하려면 빌더로부터 매개변수를 복사한 뒤 해당 객체 필드 검사 추가 할 수 있다. 
```java
public class NutritionFacts {

  private int servingSize;
  private int servings;
  private int calories;
  private int fat;
  private int sodium;
  private int carbohydrate;

  public static class Builder {

    private final int servingSize;
    private final int servings;
    private int calories;
    private int fat;
    private int sodium;
    private int carbohydrate;

    public Builder(int servingSize, int servings) {
      this.servingSize = servingSize;
      this.servings = servings;
    }

    public Builder calories(int val) {
      this.calories = val;
      return this;
    }

    public Builder fat(int fat) {
      this.fat = fat;
      return this;
    }

    public Builder sodium(int sodium) {
      this.sodium = sodium;
      return this;
    }

    public Builder carbohydrate(int carbohydrate) {
      this.carbohydrate = carbohydrate;
      return this;
    }

    public NutritionFacts build() {
      return new NutritionFacts(this);
    }
  }

  public NutritionFacts(Builder builder) {
    servingSize = builder.servingSize;
    servings = builder.servings;
    calories = builder.calories;
    fat = builder.fat;
    sodium = builder.sodium;
    carbohydrate = builder.carbohydrate;
  }

  public static void main(String[] args) {
    NutritionFacts facts =
        new NutritionFacts.Builder(240, 8)
            .calories(100)
            .sodium(35)
            .build();
  }
}
```
- 계층 구조를 가질때 유용하게 사용해 볼 수 있는 빌더 패턴 예시
- 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 `공변환 타이핑(convariant return typing)` 이라 한다. 
```java
public abstract class Pizza {

  final Set<Topping> toppings;

  public enum Topping {
    HAM,
    MUSHROOM,
    ONINON,
    PEPPER,
    SAUSAGE
  }

  //재귀적 타입 한정
  abstract static class Builder<T extends Builder<T>> {
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

    public T addTopping(Topping topping){
      toppings.add(Objects.requireNonNull(topping));
      return self();
    }

    abstract Pizza build();

    protected abstract T self(); //하위 클래스에서 형변환 없이 메서드 연쇄를 지원할 수 있다.
  }

  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone();
  }



  public static class NyPizza extends Pizza {
    private final Size size;

    public enum Size {
      SMALL,
      MEDIUM,
      LARGE
    }

    public static class Builder extends Pizza.Builder<Builder> {
      private final Size size;

      public Builder(Size size) {
        this.size = Objects.requireNonNull(size);
      }

      @Override
      public NyPizza build() {
        return new NyPizza(this);
      }

      @Override
      protected Builder self(){
        return this;
      }
    }

    private NyPizza(Builder builder){
      super(builder);
      this.size = builder.size;
    }
  }

  public static void main(String[] args) {
    NyPizza nyPizza =
            new NyPizza.Builder(SMALL)
                    .addTopping(Topping.SAUSAGE)
                    .addTopping(Topping.ONINON)
                    .build();
  }
}
```
- 장점
  - 빌더 하나로 여러 객체를 순회하면서 만들 수 있다. 
  - 넘기는 매개변수에 따라 다른 객체를 만들 수 도 있다. 
  - 특정 필드는 빌더가 알아서 채우도록 할 수도 있다. 
  - (대개는) 불변의 인스턴스를 반환하여 일관성을 유지한다. 
- 단점
  - 객체를 만들려면 빌더부터 만들어야 하는데, 점층적 생성자 패턴보다 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다. 

### 결론 
- 생성자나 정적 팩터리가 처리해야할 매개변수가 많다면 빌더 패턴을 선택하는 것이 낫다. 
- 매개 변수 중 필수가 아니거나 같은 타입이면 특히 더 그렇다. 
- 점층적 생성자보다 코드를 읽고 쓰기가 편하고, 자바 빈즈보다 안전하다. 


