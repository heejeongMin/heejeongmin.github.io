---
layout: post
title: Java 8 Stream
categories: java
tags: [JAVA]
---

## Stream?

- 데이터를 담고 있는 저장소
- 스트림 데이터는 오직 한번만 처리 가능
- 스트림이 처리하는 데이터 소스는 변경되지 않는다. (Functional in nature)
- 병렬처리가 가능하다

### Stream's operation

- Intermediate Operation
    - Stateless / Stateful (sorted 와 같이 source를참조해야 하는 경우)
- Terminate Operation

Lazy Invocation of Intermediate Operation
Terminate Operation 없이 Intermediate Operation을 사용하게 되면 정의만 할 뿐 실제로 실행 되지 않는다. Terminate Operation을 함께 사용하여 실제 결과를 도출 해 내야할때 실행된다. 

### Stream Creation

- Stream.empty()
- Stream.of()
- Stream.<T>builder()
- Stream.generate()
- Stream.iterate()
- Arrays.stream()
- collection.stream()
- splitAsStream()
- Files.lines()
- Random.double
- parallelStream()
- parallel()

parallel() vs parallelStream()
source가 collection인 경우 parallelStream() 을 생성하여 사용할 수 있고, isParallel() 메서드로 병렬로 생성한 스트림인지 여부 확인 가능
다시 한 쓰레드에서 처리되는 스트림으로 변경하고 싶으면 해당 스트림에 sequenctial() 메서드 사용

### Stream for Premitives

- IntStream
- LongStream
- DoubleStream

IntStream
String을 .chars()를 통하여 스트림으로 변경할 수 있는데, charstream은 따로 존재하지 않아서 IntStream이 사용된다.

### Stream API

- 찾기
    - filter()
    - findAny()
    - findFirst()
    - anyMatch
    - allMatch
    - nonMatch
- 모으기
    - collect()
        - Collertors를 파라미터로 사용하여 모을 수 있다. 
          
          ex) Collectors.toList(), Collectors.groupingBy(), Collectors.collectingAndThen() ...

    - sum()
    - max()
    - reduce()
    - count()
- 거르기
    - filter()
    - range(int startinclusive, int endExclusive)
    - rangeClosed(int startInclusive, int endclusive)
    - skip()
    - limit()
    - 
- 변경하기
    - map
    - flatMap()

    map() vs flatMap()
    collection인 경우 flatMap()을 사용하면 꺼내올 수 있다. map은 다른 타입으로 1:1 매칭시켜 내보내기 위한 용도

### Parallel sort

sort()는 단일 스레드, parallelSort()는 멀티 스레드에서 정렬을 해준다. 처리하는 데이터 양이나 cpu 에따라 다르지만 보통 parallelSort()가 더 빠르다고 함.