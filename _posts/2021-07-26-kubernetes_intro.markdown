---
layout: post
title:  "Kubernetes - Intro"
date:   2021-07-26 21:15:00 +0900
categories: Infrastructure
---
책에서 소개되는 실습 환경의 인프라 구성은 다음과 같고 각 인프라 도구의 소개이다.

1. 쿠버네티스 (Kubernetes)
    - 다수의 컨테이너를 관리하는데 사용 (도커)
    - 컨테이너의 자동 배포, 부하에 따른 동적 확장
    - API 게이트웨이, 서비스 디스커버리, 이벤트 버스, 인증 및 결제 등의 다양한 서비스를 관리할 수 있는 환경 제공

2. 도커 (Docker)
    - 컨테이너 환경에서 독립적으로 애플리케이션을 실행할 수 있도록 컨테이너를 만들고 관리
    - 도커로 애플리케이션 실행 시 운영체제환경에 관계 없이 독립적인 환경에서 일관된 결과 보장
   
3. 젠킨스 (Jenkins)
    - 지속적 통합 (CI : Continuous Integration)과 지속적 배포 (CD: Continuous Deployment) 지원
    - 개발한 프로그램의 빌드, 테스트, 패키지화, 배포를 모두 자동화화여 개발 단계 표준화
    
4. 프로메테우스 (Prometheus)와 그라파나 (Grafana)
    - 모니터링을 위한 도구
    - 프로메테우스 : 상태 데이터 수집
    - 그라파나 : 프로메테우스로 수집한 데이터를 관리자가 보기 좋게 시각화   
    
![image](/assets/image/kubernetes/infra.png)


-------
관련 서적

컨테이너 인프라 환경 구축을 위한 쿠버네티스/도커 [book-link]

[book-link]: https://www.google.com/search?q=%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88+%EC%9D%B8%ED%94%84%EB%9D%BC+%ED%99%98%EA%B2%BD+%EA%B5%AC%EC%B6%95%EC%9D%84+%EC%9C%84%ED%95%9C+%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%2F%EB%8F%84%EC%BB%A4&oq=%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88+%EC%9D%B8%ED%94%84%EB%9D%BC+%ED%99%98%EA%B2%BD+%EA%B5%AC%EC%B6%95%EC%9D%84+&aqs=chrome.1.69i57j0l2.11585j0j7&sourceid=chrome&ie=UTF-8
