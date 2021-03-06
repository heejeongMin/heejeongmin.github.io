---
layout: post
title: JPA 영속성 컨텍스트
categories: JAVA
tags: [JAVA, JPA]
---

### 영속성 컨텍스트 (persistence context)

---

- 개념 : Entity를 보관/관리하는 환경으로, Entity Manager를 생성할때 1개가 생성되는 논리적인 개념이다.
- 특징 
    - 영속성 컨텍스트에서 관리되는 Entity는 식별자 값으로(@Id) 구분하기 때문에 값이 반드시 있어야 한다. (Id 생성 전략이 IDENTITY를 사용하는 경우의 예를 뒤에서 확인하면 좀더 이해가 됨)
    - 영속성 컨텍스트에서 관리되는 Entity는 트랜잭션이 커밋되면 데이터베이스에 반영된다. 
    - 영속성 컨택스트에서 관리되는 Entity에 대해서는 다음과 같은 기능을 사용 가능하다
        - 1차 캐시
        - 동일성 보장
        - 쓰기 지연
        - 변경 감지
        - 지연 로딩
    - EntityManagerFactory : EntityManager = 1 : n 관계이고, EntityManager : Persistence Context는 1:1


### 1차 캐시 & 쓰기 지연 & 변경감지(DirtyChecking)

---
![Java_jpa_persistence_context](/assets/images/java/Java_jpa_persistence_context.png)

- 1차캐시
    - 영속 컨텍스트 안에 있는 1차캐시에 엔티티가 있으면 해당 엔티티를 재사용함. 
    1. 조회를 시도
    2. 영속 컨텍스트 안의 1차 캐시에 해당 엔티티가 있는지 확인하고 있으면 캐시에 있는 엔티티를 반환
             - 없다면 DB에서 조회하여 1차 캐시에 넣어두고 반환
             - persist/save, 변경감지를 통한 업데이트는 1차 캐시를 참조하고 쓰기지연 장소에 쿼리를 저장해 놓지만, 삭제의 경우 쓰기 지연 장소에는 쿼리를 저장하지만 영속성 컨텍스트에서는 제거된다. 
    
    3. 영속 컨텍스트 엔티티의 값변경, 트랜잭션  commit
    
    4. 엔티티와 스냅샷을 비교
    
    5. 다름이 발견되면 쓰기지연 장소에 update SQL 생성 (dirty checking) 
            → 이렇게 트랜잭션이 종료되어 커밋이 되어야 되는 시점에 쓰기지연 장소에 모아둔 SQL을 DB로 보내는 것을 쓰기 지연이라고함.
            - JPQL은 sql로 변환되어 데이터베이스에 접근하기 때문에 자동으로 flush가(영속성 컨텍스트의 변경내역을 데이터베이스에 반영하는 것) 호출되어 바로 커밋된다.
   
    6. DB로 쓰기지연 SQL 저장소에 있던 SQL들을 DB로 보내고, 커밋
            - 기본적으로 update는 모든 필드를 없데이트하게 되어있다. 변경된 값만 update하려면 @org.hibernate.annotations.DynamicUpdate를 사용해야한다. 
            - 보통 컬럼이 30개 이상이면 다이나믹업데이트가 더 빠르다고 하지만 직접 테스트 해보고 적용하는 것이 권고됨.
- 변경감지 (DIrty Checking)
        - 영속 컨텍스트에서 관리되고 있는 엔티티의 값을 변경하면, 트랜잭션이 커밋되는 시점에 1차 캐시에 있는 스냅샷과 비교하여 다르면 Update를 진행한다. 
### 동일성 보장

---

- 1차 캐시에 들어 있는 식별자가 같은 엔티티를 n번 조회해서 n개의 엔티티에 담아서 == 비교하여도 동일함을 보장한다.
    - SQL mapper를 사용하여 DB에서 두번 조회해서 두개의 인스턴스에 담으면 == 비교하면 동일하지 않음.
    - 동일성(identity) : 참조값 비교 ==
    - 동등성(equality) : 인스턴스가 가지고 있는 값 비교 equals()

### 지연 로딩

---

연관된 객체를 사용하는 시점에 적절한 sql을 실행하게 되는데, 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룬다고 하여 지연로딩이라고 함.
