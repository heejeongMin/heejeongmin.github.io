---
layout: post
title: Java 8 Optional
categories: java
tags: [JAVA]
---

## Optional instead of Null Reference

null을 리턴하는 상황이거나, null 체크를 개발자가 하지 않고, 참조하는 경우 NullPointerException이 발생할 수 있다. 유연하게 에러를 대처하고자 나온 것이 Optional로 값이 들어 있을 수도 있고, 없을 수도 있는 컨테이너.

### Optional Return 방법

- Optional.empty()
- Optional.of()
- Optional.ofNullable()

of()
null을 가질 수 없어서, null을 반환하는 경우 NPE 발생. null 일 가능성이 있는 경우에는 ofNullable()을 사용하여야 한다.

### Optional에 값 존재 여부 확인

- isPresent()
- isEmpty()

### Optional의 값 참조

- get()
- orElse()
- orElseGet()
- orElseThrow()

orElse() vs orElseGet()
optional의 값의 존재 여부를 떠나 무조건 실행되는 orElse()
optional의 값이 존재하지 않는 경우에만 실행되는 orElseGet()
** web service call이나 db에 qeury를 날리는 경우 성능에 차이를 가지고 올 수 있으니 유념해서 사용해야 한다. 

### Optional 조작 기능 (intermediate operations)

- map()
- flatmap()
- filter()

map() vs flatmap()
Optional 안의 값을 가져오는 map(), 하지만 가져 왔는데 또 다른 Optional 형태라면 flatmap()을 사용하면 unwrap후 실제 값을 반환

### Optional 사용 시 주의사항

- 리턴값으로만 사용하는 것이 권장된다.

    필드, 파라미터 등 타입에 사용하는 경우, null이 들어오게 되면 Optional에서 제공하는 기능을 바로 사용할 수 없고, 실제 이 Optional이 null이 아닌지를 체크해야하기 때문에 사용 의미를 상실하게 된다.

- Optional을 리턴하는 메서드에 null 리턴 금지
- Premitive 타입을 Optional에 사용하지 말고, OptionalInt, OptionalLong 등을 사용하여 불필요한 boxing, unboxing을 피하자
- Collection, Map, Stream Array, Optional 은 다시 Optional로 감싸지 말것
- Serializable 클래스에는 Optional 사용 금지 (NotSerializableExcpetion)

### TO-DO

- [ ]  Optional as return type [https://www.baeldung.com/java-optional-return](https://www.baeldung.com/java-optional-return)
- [ ]  이팩티브 자바 3판, 아이템 55 적절한 경우 Optional을 리턴하라
- [ ]  JDK9 의 추가된 Optional API 기능 ? (or(), ifPresentOrElse(), stream() ... )