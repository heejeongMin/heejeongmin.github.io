---
layout: post
title: VPC 네트워크 계층
categories: cloud
tags: [AWS, CLOUD]
---
- public/private cloud
- VPC (네트워크 격리)
- VPC Peering (여러 환경을 연결하는 네트워크 확장)

    ![VPC_intro](/assets/images/cloud/VPC_intro.png)

## VPC

---

- AWS 계정 전용 가상 네트워크
- Region 레벨 서비스
- IPv4, IPv6 주소 범위 안에 존재
- 점유 리소스에 대한 특정 CIDR 범위를 생성할 수 있다. (서브넷)
- 보통 소규모 단일 애플리케이션인 경우 VPC한개를 사용하지만 보통은 다중 VPC, 복수 계정(meshlabs, meshtools)을 사용한다. 기본적으로 계정당 5개의 vpc가 가능하나, 요청하면 늘릴 수 있다고 한다.
- 서브넷을 사용한 VPC 분리

    ex) 만일 VPC가 CIDR /22 인 VPC라면 총 1,024개의 IP를 가질 수 있는데 이 아이피를 해당 VPC 하위에 있는 서브넷들이 나눠가지게 된다. 따라서 동일 VPC안에 서브넷의 아이피 범위가 같아질 수 없다. 

    VPC 의 CIDR/22 인 예에서 서브넷이 4개가 하위에 존재한다고 하면, 한 서브넷이 가질 수 있는 최대 IP 갯수는 251이다.  1024/4 = 256 인데, 한 서브넷이 최대 251개 씩만 가질 수 있는 이유는, 각 서브넷 마다 5개의 IP를 예약 해 놓기 때문이다.



[https://www.ipaddressguide.com/cidr](https://www.ipaddressguide.com/cidr)


- CIDR (Classless Inter-Domain Routing) 
[https://www.nakjunizm.com/2020/01/29/Cidr/](https://www.nakjunizm.com/2020/01/29/Cidr/)
 ![VPC_CIDR](/assets/images/cloud/VPC_CIDR.png)
    
## 라우팅 테이블

---

- VPC 리소스 간에 트래픽을 연결하는데 필요
- VPC를 생성하면 기본 라우팅 테이블이 생성이되고, 사용자 지정 라우팅 테이블도 새로 생성할 수 있다. 
처음 생성되는 기본 라우팅 테이블에 로컬 경로 10.0.0.0/16 은 변경 할 수 없다. (해당 라우팅 테이블에 추가는 가능함)
- 모든 서브넷에는 연결된 라우팅 테이블이 존재해야한다. 각 서브넷에 대해 사용지 지정 라우팅 테이블을 사용하는 것이 권고됨.

## 서브넷

---

- 퍼블릭 서브넷과 프라이빗 서브넷이 존재한다. 퍼블릭 서브넷은 인터넷 연결이 가능하고, 프라이빗 서브넷은 인터넷 연결이 불가능하다. 서브넷은 생성하면 기본적으로 프라이빗서브넷으로 생성이되고, 해당 서브넷과 연결된 라우팅 테이블에서 인터넷 게이트웨이를 붙여주어야 퍼블릭 서브넷이 된다.
    - 인터넷 게이트웨이
        - 인터넷 라우팅 트래픽을 연결된 VPC에 전달해주고,
        - 퍼블릭 IPv4주소가 할당된 인스턴스에 대해 네트워크 주소 변환(NAT)을 수행하는데 있다.
    - NAT 게이트웨이 ([https://www.notion.so/Chapter-3-3-13-3-23-e18c03aa8e494323a41e8d7f5ae67a2c](https://www.notion.so/Chapter-3-3-13-3-23-e18c03aa8e494323a41e8d7f5ae67a2c))
        - 프라이빗 서브넷의 인스턴스가 인터넷 또는 다른 AWS 서비스로의 아웃바운드 트래픽을 시작하게해준다.
- CIDR범위가 작은 서브넷보다는 큰 서브넷이 나은데, 추후 사용가능한 IP 가 부족한 경우 서브넷에 IP를 추가할 수 없기 때문이다.

 ![VPC_subnet_1](/assets/images/cloud/VPC_subnet_1.png)
 ![VPC_subnet_2](/assets/images/cloud/VPC_subnet_2.png)


## 인스턴스의 생명 주기

---

 ![VPC_instance_lifecycle](/assets/images/cloud/VPC_instance_lifecycle.png)


## ENI (Elastic Network Interface) 

---

탄력적 네트워크 인터페이스

- 가상 네트워크 인터베이스로 네트워크의 어댑터 같은 존재
- EC2 인스턴스 하나당 ENI 한개는 default로 존재.
- 동일한 가용영역 안에서 EC2 인스턴스 간에 ENI를 옮길 수 있는데, 옮겨지면 다음 정보를 같이 가지고 간다.
    - 프라이빗 IP
    - 탄력적 IP
    - MAC 주소
- 여러 ENI를 하나의 인스턴스에 붙이는 경우의 유용한점
    - 관리 네트워크 생성 
    - 만약에 두개가 붙어있다고 하면, ENI1, ENI2 가 있는 경우, EN1은 퍼블릭 트래픽 처리, EN2는 벡엔드 관리 트래픽을 처리하여, 엑세스 제어가 필요한 별도의 서브넷과의 트래픽을 처리 할 수 있다고 한다.
    - VPC에서 네트워크 및 보안 어플라이언스 사용
    - 별도의 서브넷에 있는 워크로드 / 역할로 이중 홈 인스턴스 생성
    - 저예산 고가용성 솔루션 생성


## EIP (Elastic IP)

---

탄력적 IP 주소

- 고정 퍼블릭 (IPv4) 주소로 인스턴스 또는 네트워크 인터페이스에 연결 할 수 있다. (IPv6에 대한 탄력적 아이피주소는 현재 지원하지 않는다고 한다.) 만약 장애가 나는경우 해당 IP주소를 다른 인스턴스에 매핑하여 신속하게 대처를 할수 있기도 하다고 함. 이때 인스턴스 대신 ENI에 연결하면 ENI의 모든 속성을 한번에 다른 인스턴스로 이동할 수 있는 장점이 있닥 ㅗ한다.
- 리전당 5개가 허용되지만, soft limit이기 때문에 요청하면 더 사용할 수 있다.
- EIP를 할당하여 사용하면 비용이 없지만, 할당하는데 인스턴스가 stop상태로 사용하지 않는 상태이면 비용이 든다.

## 네트워크 보안

---

### 보안그룹

- AWS 리소스에 대한 인바운드 및 아웃바운드 트래픽을 제어하는 가상 방화벽 .
- 인스턴스 레벨로 적용됨
- 모든 IP프로토콜, 포트, 주소로 제한 할 수 있다.
- 규칙은 상태 저장이다. 즉, 인바운드에서 허용되어서 들어오면 해당 트래픽은 아웃바운드를 체크하지 않는다. 그리고 "허용"만 설정할 수 있다.
- 처음 생성하면 기본적으로 모든 아웃바운드 트래픽 허용 / 모든 인바운드 차단으로 생성된다.
- 다른 인스턴스에 대해 서로 다른 규칙을 설정하도록 할 수 있다.
    - 보안 그룹의 체인 : 상위 티어에서 하위 티어로만 흐룬뒤 반드시 반대로 흐르도록 인바운드와 아웃바운드 규칙 설정.

 ![VPC_security_group_1](/assets/images/cloud/VPC_security_group_1.png)
 ![VPC_security_group_2](/assets/images/cloud/VPC_security_group_2.png)
 ![VPC_security_group_3](/assets/images/cloud/VPC_security_group_3.png)


### NACL (Network Access Control List)

- 서브넷 레벨
- 인스턴스 레벨의 보안그룹과 달리 상태를 저장하지 않는다. 즉, 인바운드/아웃바운드 모두 허용/거부 여부를 체크한다.
- 처음 생성하면, 기본적으로 모든 인바운드/아웃바운드 "거부" 상태로 생성된다.

 ![VPC_nacl](/assets/images/cloud/VPC_nacl.png)
 ![VPC_sg_nacl](/assets/images/cloud/VPC_sg_nacl.png)
 
 
---
 ![VPC_test_1](/assets/images/cloud/VPC_test_1.png)

 ![VPC_test_2](/assets/images/cloud/VPC_test_2.png)


