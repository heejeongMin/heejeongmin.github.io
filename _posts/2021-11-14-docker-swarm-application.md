---
layout: post
title: 스웜을 이용한 실전 애플리케이션 개발
subtitle: 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문
categories: docker
tags: [DOCKER, CONTAINER, INFRASTRUCTURE]
---

Chapter04 스웜을 이용한 실전 애플리케이션 개발
---

- 이 장을 이해하고 나면 컨테이너 중심의 애플리케이션 개발에 대한 기초적인 기술을 습득할 수 있다.

## 웹 애플리케이션 구성
### 아키텍처
- 오케스트레이션 시스템으로 도커 스웜을 사용 (도커 스웜 클러스터 구축 참고 : https://heejeongmin.github.io/docker/2021/11/07/docker-container_best_practice.html)
   ![architecture.png](/assets/images/docker/ch04/architecture.png)  
   * nignx : 애플리케이션 프론트엔드 서버 및 API 앞단에 리버스 프록시 역할을 한다. 캐싱, 백엔드에 대한 유연한 라우팅, 접근 로그 생성등의 역할을 한다.
- overlay 네트워크를 미리 같춘다. (overlay 네트워크 : 컨테이너를 배치된 호스트와 상관없이 마치 같은 네트워크에 속하는 것처럼 다룬다.)
   ```shell script
      docker exec -it manager docker network create --driver=overlay --attachable crane-overlay-network
   ```
   
### MySql 서비스 구축
1. MySql 마스터-슬레이브 구조 구축
#### 데이터베이스 컨테이너 구성
    - MySQL 컨테이너는 도커 허브에 등록된 최신 이미지를 사용한다. 
    - 마스터-슬레이브 컨테이너는 두 역할을 모두 수행할 수 있는 하나의 이미지로 생성한다. 
         - 별도의 Dockerfile대신 환경 변수 값으로 두 역할을 수행하는 이미지를 만든다.  
    - MYSQL_MASTER 환경 변수의 유무에 따라 해당 컨테이너가 마스터로 동작할지, 슬레이브로 동작할지가 정해진다. 
    - replicas 값을 조절해 슬레이브를 늘릴 수 있게 한다. 이때 마스터, 슬레이브 모두 일시정지(다운타임)을 허용한다. 
       
#### 인증 정보
**Master**  

| 환경 변수 이름 | 내용 | d
| :-------------- | :-------------- | 
| MYSQL_ROOT_PASSWORD | root 사용자 패스워드 |
| MYSQL_DATABASE | crane 어플리케이션에서 사용할 데이터베이스 |
| MYSQL_USER | 데이터베이스 사용자명 |
| MYSQL_PASSWORD | 패스워드 |

**Slave**  

| 환경 변수 이름 | 내용 |  값 | 
| :-------------- | :-------------- | :-------------- | 
| MYSQL_MASTER_HOST | 마스터의 호스트명 | master |
| MYSQL_ROOT_PASSWORD | root 사용자 패스워드  | password |
| MYSQL_DATABASE | crane 어플리케이션에서 사용할 데이터베이스 | crane-mysql |
| MYSQL_USER | 데이터베이스 사용자명 | crane |
| MYSQL_PASSWORD | 패스워드 | crane |
| MYSQL_REPL_USER | 마스터에 등록할 레플리케이션 사용자 | repl |
| MYSQL_REPL_PASSWORD | 레플리케이션 사용자 패스워드 | crane |


-------
관련 서적

도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문 [link]

[link]: http://www.yes24.com/Product/Goods/70893433
