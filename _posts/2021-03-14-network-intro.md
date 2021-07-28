---
layout: post
title: 네트워크의 기본
subtitle: 그림으로 배우는 네트워크 원리
categories: infrastructure
tags: [NETWORK, INFRASTRUCTURE]
---

## 무엇을 위해서 네트워크를 이용할까 ?

---

PC, 정보 단말기기, 서버 간에 네트워크를 통하여 데이터를 주고 받을 수 있게 한다. 

## 누가 이용할 수 있는 네트워크인가?

---

네트워크 사용 주체를 기반으로 크게 사설네트워크와 공용네트워크 2가지로 나뉜다. 

1. **사설 네트워크 (Private Network)**
- **사설 네트워크란 ?**
특정 집단에만 접속할 수 있도록 제한한 사설 IP 주소로 구축된 네트워크로 회사에서는 인트라넷이라고 부르기도한다. 주로, 비인가자가 접근하지 못하게 하기 위한 보안측면에서 사용한다. 

공개 범위에 따라 인트라넷과, 엑스트라넷으로 나뉘기도 한다. 
- 인트라넷 : 내부(Intra)라는 의미와 Network가 결합된 용어로 Intra + net이 되었다. 
                 : 인트라넷을 운영하는 기관(회사)에 소속된 직원만 접근 가능하다. 
- 엑스트라넷 : 외부(Extra)라는 의미와 Network가 결합된 용어로 Extra + net이 되었다. 
                    : 소속된 직원 + 외부 협력사 및 고객까지 접근할 수 있게 하는 네트워크이다. 
                    : 외부에 어느정도 공개된 네트워크인만큼 보안성/기밀성이 요구되어, VPN, 방화벽과 같은
                      정보보호 제품을 통해 보안할 수 있다.
- **사설 IP 주소 특징**
    - 외부에 공개되지 않아 외부에서 접근이 기본적으로 불가능하다.
    - 일반적으로 많이 사용되는 IP 주소는 "192.168.xx.xx" 이다.
    - 사설 IP주소만으로는 인터넷 직접 연결은 되지 않으며, 라우터 장비에서 개인 IP주소와 글로벌 IP주소를 상호 변환함으로써 인터넷을 이용할 수 있다.

![Network_IP](/assets/images/network/Network_IP.png)
![Network_my_pc](/assets/images/network/Network_my_pc.png)


- 사설 네트워크는 어떻게 구성하는가 ?
    - **LAN (Local Area Network)**
    - 정의 : 근거리에 구성된 네트워크
    - 주체 : 사용자가 직접 구축하고 관리해야한다 (기기배치, 배선, 유선(이더넷), 무선 장비 구비 등)
    - 비용 : 통신요금이 들지는 않지만 기기 구축 및 인건비용이 들어간다.
    - **WAN (Wide Area Network)** 
    - 정의 : 물리적으로 멀리 떨어진 곳의 네트워크를 통해 연결. "물리적으로 멀리"가 상대적인 개념이라 때에 따라 지역 <> 지역, 국가 <> 국가 가 될 수도 있다. 
    - 주체 : 통신업자가 구축하고 관리한다. (KT, SK, LG, 세종텔레콤, 딜라이브 ....)
    - 비용 : 통신업자와의 초기 계약비용 및 통신요금 (종량제/고정)
    - **LAN 과 WAN 의 차이 ?**
        - 예를 들어, 규모가 큰 회사인 경우 여러 지사를 가질 수 있다. 이때 각 지사에서는 각각 LAN을 구축하게된다. 이 지사들 간에 통신도 필요하니, 이때 WAN을 통하여 가능하다.

![Network_lan_wan](/assets/images/network/Network_lan_wan.png)

    - 참고 
     PAN (Personal Area Network)
     CAN (Campus Area Network)
     MAN (Metropolitan Area Network)

2. 공용 네트워크 (Public Network)

- 정의 : 인터넷과 같은 공공회선에 액세스 할 때 글로벌 IP 주소로 구축된 네트워크이다. 사용자의 PC가 라우터를 통하지 않고 인터넷에 직접 연결되어 있는 경우 PC의 IP 주소를 공인 IP 주소라고 부른다.
    - 공인 IP 주소의 특징
        - 외부에 공개되어 있기 때문에 인터넷에 연결된 다른 PC로부터 접근이 가능. 따라서 방화벽 등의 보안프로그램 설치가 권장된다
        - 인터넷 사용자의 로컬 네트워크를 식별하기 위해 ISP(인터넷 서비스 공급자)가 제공하는 IP

## 네트워크의 네트워크

---

인터넷이란 전 세계의 다양한 조직이 관리하는 네트워크가 연결되어 있는 것이다. 이 조직의 네트워크를 AS(Autonomous System)라고 부른다. AS 네트워크를 제공하는 기업/조직들을 인터넷 서비스 프로바이더(ISP : Internet Service Provider) 라고한다 

Autonomous System :[https://www.youtube.com/watch?v=CT4xaXLcnpM](https://www.youtube.com/watch?v=CT4xaXLcnpM)

![Network_autonomous_system](/assets/images/network/Network_autonomous_system.png)


ISP들은 많지만, 이 중 최상위 군을 Tier1이라고 부른다. Tier1에 속하지 않는 ISP들은 인터넷을 사용하려면 Tier1 을 통해야 한다. 

[https://en.wikipedia.org/wiki/Tier_1_network](https://en.wikipedia.org/wiki/Tier_1_network)
[https://unifiedh.medium.com/왜-인터넷은-근본부터-글러먹었는가-코로나19와-한국-인터넷의-해외접속-장애-그리고-넷플릭스-전쟁에-관한-이야기-ae27826e7fc8](https://unifiedh.medium.com/%EC%99%9C-%EC%9D%B8%ED%84%B0%EB%84%B7%EC%9D%80-%EA%B7%BC%EB%B3%B8%EB%B6%80%ED%84%B0-%EA%B8%80%EB%9F%AC%EB%A8%B9%EC%97%88%EB%8A%94%EA%B0%80-%EC%BD%94%EB%A1%9C%EB%82%9819%EC%99%80-%ED%95%9C%EA%B5%AD-%EC%9D%B8%ED%84%B0%EB%84%B7%EC%9D%98-%ED%95%B4%EC%99%B8%EC%A0%91%EC%86%8D-%EC%9E%A5%EC%95%A0-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EB%84%B7%ED%94%8C%EB%A6%AD%EC%8A%A4-%EC%A0%84%EC%9F%81%EC%97%90-%EA%B4%80%ED%95%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0-ae27826e7fc8)

![Network_tier](/assets/images/network/Network_tier.png)
![Network_tier_wikipedia](/assets/images/network/Network_tier_wikipedia.png)


ISP 의 라우터에 바로 접속 하는 방법

고정회선 

전용선 : 통신 속도는 보장되나 비용이 비쌈

전화회선 : 저가로 인터넷에 접속할 수 있다 (모뎀?)

광케이블 /케이블 TV : 고속으로 인터넷 접속 가능

모바일 회선
      휴대전화 : 4G, LTE
      WiMAZ / WiMAX2 ([https://ko.wikipedia.org/wiki/와이맥스](https://ko.wikipedia.org/wiki/%EC%99%80%EC%9D%B4%EB%A7%A5%EC%8A%A4))
      무선랜 (Wi-Fi)

## 애플리케이션간의 통신 방식

---

1. 클라이언트 서버 애플리케이션 
: 우리가 익숙한 클라이언트<>서버가 네트워크를 통해 데이터를 요청/응답을 통하여 서로 주고 받는 형태 (양방향)
2. 피어투피어 애플리케이션 (Peer-to-Peer)
: P2P라고도 하며, 서버를 거치지 않고, 클라이언트끼리 직접 데이터를 주고 받는 형태를 말한다. 
: 요청하는 컴퓨터와 서비스를 제공하는 컴퓨터가 따로 있는 것이 아니라, 서로 연결되어 다수의 컴퓨터에게 데이터를 요청하고, 여러대에서 데이터를 제공해 줄 수 있다면 동시에 여러곳에서 받을 수 있다. 
[https://academy.binance.com/ko/articles/peer-to-peer-networks-explained](https://academy.binance.com/ko/articles/peer-to-peer-networks-explained)
![Network_application](/assets/images/network/Network_application.png)


## 통신에서 사용하는 언어

---

- 네트워크 아키텍쳐

    우리가 같은 언어로 대화를 하는 것처럼 컴퓨터끼리 통신에도 서로 이해할수 있는 언어로 통신을 해야하는데 그것을 네트워크 아키텍쳐라고 한다. 종류는 여러가지가 있지만, TCP/IP가 현재 주요 사용되고, 네트워크의 공통 언어이다. 네트워크 아키텍쳐는 프로토콜 스택, 프로토콜 스위트라고도 불린다. 

    종류 :  TCP/IP, OSI, Microsoft METBEUI, Apple Appletalk ... 

- 프로토콜
우리가 같은 언어로 대화할때 해당 언어의 문자 표기법, 발음, 문법 등이 있는 것처럼, 컴퓨터의 통신언어인 네트워크 아키텍쳐에도 규칙이 있는데 그것을 프로토콜이라고 한다. 
[https://namu.wiki/w/TCP/IP](https://namu.wiki/w/TCP/IP)

![Network_OSI](/assets/images/network/Network_OSI.png)

- TCP/IP
프로토콜 중 가장 중요한 역할을 하는, TCP와 IP의 합성어로 데이터의 흐름관리, 정확성 확인 및 패킷의 목적지 보장을 담당한다. 데이터의 정확성확인은 TCP, 패킷의 목적지까지의 전송은 IP가 한다.
    - IP (Internet Protocol)
    : 인터넷에서 사용 되는 주소
    : 패킷 전달 여부를 보증하지 않고, 패킷을 보낸 순서와 받는 순서가 다를 수 있다.
    - TCP (Transmission Control Protocol) 
    : 데이터 전달 규칙을 정의 하고 있다. 
    : IP위에서 동작하는 프로토콜로, 데이터의 전달을 보증하고 보낸 순서대로 받게 해준다.
    - packet : 데이터의 형식화된 블록으로 네트워크를 통해 전송되는 데이터 조각

## 클라우드

---

지금까지 위에서 설명한 내용을 직접 구축/관리하지 않고, 인터넷을 통해 서버의 기능을 사용할 수 있게 한 것을 클라우드 서비스라고 한다. 기존 방법은 온프레미스 (on-premis)라고 한다. 

서버의 어느 부분까지를 사용자가 이용할수 있느냐에 따라 3가지로 나뉜다. 

1. Iaas (Infrastructure as a Service)
: CPU/메모리/스토리지와 같은 **하드웨어** 부분만 제공
2. Paas (Platform as a Service)
: 하드웨어 + OS/미들웨어 **플랫폼** 제공
3. Saas (Software as a Service)
: **애플리케이션** 제공

![Network_aws](/assets/images/network/Network_aws.png)

참고 자료

[https://www.tabmode.com/windows10/private-public-network.html#gsc.tab=0](https://www.tabmode.com/windows10/private-public-network.html#gsc.tab=0)

[https://papadino.tistory.com/12](https://papadino.tistory.com/12)

[https://namu.wiki/w/인터넷 서비스 제공사업자](https://namu.wiki/w/%EC%9D%B8%ED%84%B0%EB%84%B7%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%EC%A0%9C%EA%B3%B5%EC%82%AC%EC%97%85%EC%9E%90)

[https://m.blog.naver.com/lego7407/221806770611](https://m.blog.naver.com/lego7407/221806770611)

[https://medium.com/@rlatla626/tcp-ip-정리-204e8a986d98](https://medium.com/@rlatla626/tcp-ip-%EC%A0%95%EB%A6%AC-204e8a986d98)

[https://enlqn1010.tistory.com/9](https://enlqn1010.tistory.com/9)