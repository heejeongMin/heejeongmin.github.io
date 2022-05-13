---
layout: post
title: 마이크로서비스는 코드 작성 이상을 의미
subtitle: 스프링 마이크로서비스 코딩 공작소
categories: MICROSERVICE 
tags: [MICROSERVICE]
---
## 마이크로 서비스를 구축할때 고려해야할 서비스가 실행될 환경과 확장 및 회복성에 대한 방법을 고민해야한다.

![microservice_consideration_points.png](/assets/images/microservice/microservice_consideration.png)

### 적정크기
- 마이크로서비스가 과도한 책임을 맡지 않도록 어떻게 적절한 크기로 만들 수 있을까?
- 적절한 크기의 마이크로서비스는 애플리케이션의 신속한 변경과 전체 애플리케이션의 전반적인 장애를 줄일 수 있다.
- 단위가 작을 수록 코드 변경에 따른 복잡성을 낮추고 코드 배포 시간이 줄어드는 유연성을 가질 수 있다.

### 위치투명성
- 다수의 인스턴스가 재빨리 시작하고 종료될 때 서비스 호출에 대한 물리적 상세 정보를 관리할 수 있는 방법 → 아직 잘 이해안가요

### 회복성
- 장애가 발생한 서비스를 우회하고 빨리 실패하는 방법은 무엇인가?
- 장애 발생 시 애플리케이션의 전반적인 무결성을 어떻게 유지할 것인가 ?

### 반복성
- 새로운 인스턴스가 시작될때마다 코드베이스를 동일하게 하는 방법은 무엇인가?

### 확장성
- 분리된 서비스를 여러 서버에 수평적으로 쉽게 분산할 수 있는 장점이 있는 마이크로서비스를 어떻게 원만하게 확장할 수 있는가?
- 비동기 프로세싱과 이벤트를 사용해 서비스간 의존성을 최소화 필요


## 위의 고민사항의 답이 될 수 있는 6가지 패턴
1. 핵심 개발 패턴 (core development patterns)
2. 라우팅 패턴 (routing patterns)
3. 클라이언트 회복성 패턴 (client resiliency patterns)
4. 보안 패턴 (security patterns)
5. 로깅 및 추적 패턴 (logging and tracing patterns)
6. 빌드 및 배포 패턴 (build and deployment patterns)
