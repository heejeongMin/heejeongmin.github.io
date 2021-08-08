---
layout: post
title: VPC 고가용성
categories: cloud
tags: [AWS, CLOUD]
---
- 네트워크 연결
- VPC 엔드포인트
- 로드 밸런싱
- 고가용성

    ![VPC_high_availability](/assets/images/cloud/VPC_high_availability.png)

## VGW (Virtual (Private) Gateway)

---

on-premis에서 aws로 넘어갈때 과도기라던가, DB는 on-premis에 있다던가 하는 이유로 클라우드와 온프레미스를 연결해야하는 경우가 있다고 한다. 이때 사용할 수 있는것이 VGW(Virtual Private Gateway)라고 한다. Private 하긴 하지만 인터넷을 사용하기 때문에 보안이 걱정된다면, AWS Direct Connect를 사용할 수 있다고 한다. AWS Direct Connect를 사용하면, 전용 광케이블을 사용할 수 있다고 한다. 

![VPC_vgw_1](/assets/images/cloud/VPC_vgw_1.png)
![VPC_vgw_2](/assets/images/cloud/VPC_vgw_2.png)

## VPC 피어링

---

- Region, 계정 상관 없이 VPC끼리 1:1 연결이 가능하다.
- 전이적 피어링은 지원되지 않는다. 예를들어 VPC가  A, B, C가 있고, A<>B, B<>C 이렇게 1:1 연결되어 있는 경우, A와 C를 직접 피어링 해주지 않는 이상 연결되지 않는다.
- 피어링 개수 공식 : n(n-1)/2
- 인터넷 게이트웨이 또는 가상 게이트웨이가 필요하지 않다.
- 피어링 하려면
    1. A VPC 가 B VPC 에게 Peering Connection 요청
    2. B가 A의  Peering Connection 요청을 승인
    3. 라우팅 테이블에 서로의 CIDR값과 피어링 커넥션을 넣어준다. 
- VPC마다 각자의 CIDR범위를 가질 수 있기 때문에 CIDR블록이 중복되 않도록 해야한다.

![VPC_peering](/assets/images/cloud/VPC_peering.png)


## TGW(Transit Gatway)

---

- 라우터와 같은 역할을 하며, 500개까지 가지고 있을 수 있다.
- AWS 의 완전 관리형 서비스이다.
- 리전간 피어링도 가능하다.

![VPC_tgw](/assets/images/cloud/VPC_tgw.png)

## VPC EndPoint

---

- AWS를 벗어나지 않고 EC2 인스턴스를 VPC외부 서비스와 프라이빗 하게 연결해야하는 경우 사용할 수 있다.
- IGW, VPN, NAT, 방화벽 프록시 모두 필요하지 않다. 인터넷을 통해 통과하지 않음. 단 동일 리전에 있어야한다.
- AWS 서비스 중 두 가지 유형의 엔드포인트가 존재하는데, 인터페이스 엔드포인트는 VPC내부에 존재하고, ENI가 존재. 하지만 S3(Region레벨 서비스),  DynamoDB는  게이트웨이 엔드테이블이 필요하여 라우팅 테이블에 추가해서 사용해야한다.

![VPC_endpoint_type](/assets/images/cloud/VPC_endpoint_type.png)

![VPC_endpoint](/assets/images/cloud/VPC_endpoint.png)



## ELB (Elastic Load Balancing)

---

- 수신되는 애플리케이션 트래픽을 여러 EC2 인스턴스, 컨테이너 및 IP 주소에 걸쳐 분산하는 관리형 로드 밸런싱 서비스
- EC2가 앞에 존재하여 트래픽을 분산시켜주는데,  EC2를 숨길수 있어 보안적인 측면에서도 좋음
- 바라보고 있는 인스턴스들의 상태를 계속해서 확인하는 health check 기능을 가지고 있음.
- 설정에 따라 AWS 서버 사용율이 높아지면 인스턴스를 늘이는 auto scaling 기능을 가지고 있음.
    - 3분 간격으로 CPU가 45% 이상이면 인스턴스 2개를 증가하고, 25% 미만이면 한개를 제거
    - 인스턴스를 제거하는 경우 다음과 같은 절차를 거친다.
        1. Inbound차단
        2. 이미 들어온 요청은 처리가 될 때까지 대기
        3. 모든 요청이 처리가 된 후 EC2 제거 

![VPC_elb](/assets/images/cloud/VPC_elb.png)


- ELB의 DNS주소를 nslookup 해보면 두개의 ip주소가 나오는데 .. 음 어떤게 AWS의 IP 인지 어떻게 알지 ??

![VPC_elb_two_ip](/assets/images/cloud/VPC_elb_two_ip.png)
   
    1. 우리의 VPC가 오른쪽에 있구요. 왼쪽에 우리에게는 보이지 않지만 AWS가 관리하는 내부VPC가 존재합니다.
    2. ELB를 생성할때  특정AZ의 서브넷을 선택하거든요, AWS내부 VPC의 서브넷도 선택한 AZ와 동일한 곳에 존재합니다. (연두색형광팬)
    3. ELB를 생성하면, 실제로는 동일한 AZ의 AWS 내부 VPC에 AZ단위로 ELB가 생성이 됩니다. (그림의  노랑색 동그라미)
    4. 그리고 실제 우리VPC에 ELB생성시 선택한 서브넷에 그림처럼 ENI가 생성이 되어 실제 서비스에 Direct하게 연결하여 마치 서브넷에 있다라고 생각합니다.
    5. ENI는 네트워크 어뎁터니까 ENI안에도 IP address가 있겠죠. 그래서 희정님이 lookup하니 ip가 2개가 보이는 것입니다.
    6. ELB는 두개의 ENI를 묶어 사용자 요청을 받을수 있게 하나의 엔드포인트를 제공하게 됩니다.


- ELB
    - 역할
        - 단일 가용 영역 또는 여러 가용 영역에 있는 여러 대상 리소스들에게 ( EC2, container, ip 주소)에 자동으로 분산
        - EC2의 가용성을 확인하기 위해 주기적으로 핑을 보내 연결 시도하거나 요청을 보내 테스트한다. 200응답이 와야 정상으로 간주.  —> health check
        - 통합 인증 관리 및 SSL복호화를 지원하여, 사용자가 로드밸런서의 SSL 설정을 중앙 집중식으로 관리할 수 있다.
        - Amazon VPC내에 프로비저닝된 ELB 로드 밸런서는 보안 그룹과 같은 네트워크 보안 그룹을 활용할 수 있다.

![VPC_remove_instance](/assets/images/cloud/VPC_remove_instance.png)


   - 종류
    1. 애플리케이션 로드 밸런서 (ALB : Application Load Balancer)
        - OSI 7 계층 중 애플리케이션계층에서 작동
        - HTTP, HTTPS 통신
        - Path 기반 라우팅 지원
     ![VPC_alb](/assets/images/cloud/VPC_alb.png)

   2. 네트워크 로드 밸런서 (NLB : Network Load Balancer)
        - 클라이언트로 부터 수신되는 트래픽을 수락하고 이 트래픽을 동일한 가용 영역 내 대상 전체로 분산합니다.
        - TCP, TLS, UDP
        - OSI 4 계층에서 작동하며, IP 프로토콜 데이터에 따라 연결을 대상, 즉 amazon EC2 인스턴스, 컨테이너 및 IP주소로 라우팅한다.
        - ALB와 API 호환 가능
    3. 클래식 로드 밸런서 (CLB : Classic Load Balancer)
        - 여러 가용 영역의  EC2 인스턴스 사이에서 기본적인 로드 밸런싱 제공

## 고가용성

---

- 고가용성을 위해 AWS 리전당 2개의 가용영역을 사용하는 건이 권장된다. 한 가용 영역의 애플리케이션에 장애가 발생하면 안됨

![VPC_architecture_diagram_1](/assets/images/cloud/VPC_architecture_diagram_1.png)
![VPC_architecture_diagram_2](/assets/images/cloud/VPC_architecture_diagram_2.png)


## Route 53

---

- Domain Name System (DNS) 서비스

- 도메인 이름을 IP 주소로 변환한다. 

- DNS를 상태 확인 서비스와 결합하여 정상적인 엔드포인트로 트래픽을 라우팅하거나 개별적으로 엔드포인트에 대한 모니터링 / 경보를 설정 할 수 잇음. 

    - 라우팅 옵션
        1. 단순 라우팅 (라운드 로빈) : 다수의 요청을 모든 참여 서버로 최대한 균일하게 분산
        2. 가중치 기반 라운드 로빈 : 각 응답이 처리되는 빈도를 지정하기 위해 리소스 레코드 세트에 가중치 할당할 수 잇고, 이 기능을 사용하면 소프트웨어를 변경한 서버로 소규모 트래픽을 전송하여 A/B 테스트를 수행할 수 있다. 
        3. 지연 시간 기반 라우팅 (LBR) : 여러 AWS 리전의 실제 성능 측정치를 기준으로 가장 빠른 환경을 제공하는  AWS 엔드포인트로 고객을 라우팅
        4. 지리 근접 라우팅 : 사용자와 리소스 사이의 물리적 거리를 기반으로 트래픽 라우팅. 트래픽 흐름 정책을 생성할 때 AWS리전 또는 각 엔드포인트의 위도 및 경도를 지정할 수 있다. 
        5. 지리 위치 라우팅 : 사용자의 지리적 위치에 따라 트래픽을 지원할 리소스를 선택할 수 있고, 콘테츠를 현지화하고 웹 사이트의 일부 또는 전체를 사용자의 언어로 표시할 수 있다.