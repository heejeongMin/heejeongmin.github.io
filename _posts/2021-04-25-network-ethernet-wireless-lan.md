---
layout: post
title: 이더넷과 무선 LAN
subtitle: 그림으로 배우는 네트워크 원리
categories: infrastructure
tags: [NETWORK, INFRASTRUCTURE]
---

## 네트워크의 기본 구성 및 데이터 전송 방법

- 네트워크의 기본적인 구성은 레이어2 스위치로 하나의 네트워크를 구성하고, 라우터 또는 레이어3 스위치로 각 네트워크를 서로 연결한다.
- **다른 네트워크에 접속된 서버까지의 데이터 전송은 같은 네트워크 내의 전송을 반복**
    - 우선 같은 네트워크상에 있는 라우터로 데이터를 전송한 다음 그 라우터기준으로 같은 네트워크에 있는 라우터로 전송하여 최종적으로는 원하는 목적지에 데이터를 전송
    - 네트워크 내에서 전송을 반복하는 데에 필요한 **프로토콜이 네트워크 인터페이스층에 속한 이더넷과 무선 LAN(WIFI) 이다.**
    - 이더넷으로 연결되면 유선이고, WIFI로 연결되면 무선이다.

![Network_data_transmission_repeat](/assets/images/network/Network_data_transmission_repeat.png)

# 이더넷

---

1. 네트워크 인터페이스층의 프로토콜로 데이터를 전송하는 프로토콜로, 주 목적은 '어디에서 어디까지 데이터를 전송하는가'이다. **이더넷은 같은 네트워크내의 어떤 이더넷 인터페이스에서 다른 이더넷 인터페이스까지의 데이터 전송**을 담당한다. 
2. 레이어2 스위치로 연결된 PC는 동일 네트워크에 있다고 보는데, 이때 이더넷이 두 PC의 이더넷 인터페이스를 이용하여 데이터를 전송하게 된다. 

## 이더넷의 규격

1. 정의 주체는 IEEE802 위원회 (Institute of Electrical and Electronics Engineers)  
    - [https://en.wikipedia.org/wiki/IEEE_802](https://en.wikipedia.org/wiki/IEEE_802)
    - [https://www.ieee802.org/](https://www.ieee802.org/)
    
    ![Network_ethernet_spec](/assets/images/network/Network_ethernet_spec.png)


2. 10Mbps ~ 100Gbps 사이의 속도를 지원

3. 규격 이름 정의

    - 1000BASE-T
        - 숫자 : Mbps 단위의 속도
        - BASE : 베이스밴드 방식 의미인데 이것 말고 사용하는 것은 없음
            - 베이스밴드 방식이란, 0과 1의 직류 신호를 변환없이 수신측에 그대로 보내주는 것을 의미 ([https://m.blog.naver.com/PostView.nhn?blogId=jvioonpe&logNo=220227244403&proxyReferer=https:%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=jvioonpe&logNo=220227244403&proxyReferer=https:%2F%2Fwww.google.com%2F))
        - - 뒤 : 전송매체/물리신호 변환의 특징
    - 가장 많이 사용되는 규격
        - 10BASE-T, 100BASE-T, 1000BASE-T, 10GBASE-T로 모두 RJ-45 이더넷 인터페이스와 UTP 케이블을 사용한다.
            - RJ-45 이더넷 인터페이스
                - UTP 케이블에 맞춰 8개의 단자가 있고, 전기신호가 흐르는 회로를 최대 4쌍 형성할 수 있다.
            - UTP 케이블
                - 가장 일반적으로 사용되며, 8줄의 절연체로 감싼 구리선을 2줄씩 꼬아서 4쌍으로 만든 케이블. 선을 꼬았기 때문에 노이즈의 영향이 억제되는 효과가 있다.

## 이더넷 인터페이스

이더넷이 데이터를 전송하기 위해 이더넷 인터페이스를 사용한다고 하였는데 이더넷 인터페이스는 MAC주소가 있는 인터페이스를 말한다. 

- MAC주소
    - 이더넷 인터페이스를 특정하기 위한 48비트 주소로 앞 24비트는 OUI(벤더 식별코드), 뒤 24비트는 시리얼 넘버이다. (OUI : [http://standards-oui.ieee.org/oui/oui.txt](http://standards-oui.ieee.org/oui/oui.txt))
    - 시리얼 넘버는 각 벤더가 할당하는데,  **MAC주소는 이더넷 인터페이스에 미리 할당되어 있어, 기본적으로 변경할 수 없는 주소**라서 '물리주소' 혹은 '하드웨어 주소'라고도 한다.
    - 표기는 16 진수로 표기함으로 0~9, A~F의 조합으로 나타나며 표기 규칙은 다음과 깥다.
        - 1바이트씩 16진수로 변환하고 '-' 혹은 ':' 으로구분한다.
        - 2바이트씩 16진수로 변환하고 '.'로 구분한다.
        
            ![Network_ethernet_interface](/assets/images/network/Network_ethernet_interface.png)


## 이더넷의 데이터

전송할 데이터에 이더넷 헤더 및 에러 체크를 위한 FCS를 붙여야 한다. 이더넷 헤더 + 데이터 + FCS 까지 합한 전체를 이더넷 프레임이라고 한다. 

- 이더넷 헤더 구성요소
    - **목적지 MAC주소**
    - **출발지 MAC주소**
    - 타입코드
        - 이더넷으로 운반할 데이터를 나타내는데, 현재는 TCP/IP를 사용하기 때문에 IPv4를 나타내는 0x0800이 정해져 있다.

  전송 대상이 되는 데이터는 64~1500 바이트로 크기가 정해져 있고, 이 데이터 크기의 최대값을 MTU (Maximum Transmission Unit) 이라고 부른다. MTU를 넘어가게 되면 데이터를 복수로 분할하여 전송하게 되고 프로토콜은 TCP를 사용한다. 따라서, 이더넷프레임의 크기는 64~ 1518로 정해져 있다. 

![Network_mac_address](/assets/images/network/Network_mac_address.png)
![Network_ethernet_frame](/assets/images/network/Network_ethernet_frame.png)



## 토폴로지 (topology)

- 도형의 연결방식이나 위치 관계에 초점을 둔 학문분야를 가리키는데 네트워크 분야에서 사용되면서 기기를 연결하는 형태를 나타내는 말로 사용되게 되었다.
    - 버스형

        초기 이더넷 토폴지로, 하나의 전송 매체를 복수의 기기가 공유하는 형태. 공유를 어떻게 제어할 것인지는  CSMA/CD라는 방식을 사용하게 된다. 

        - CSMA/CD : Carrier-sense multiple access with collision detection (CSMA/CD)

            [https://en.wikipedia.org/wiki/Carrier-sense_multiple_access_with_collision_detection](https://en.wikipedia.org/wiki/Carrier-sense_multiple_access_with_collision_detection)

            전송 매체의 공유 메커니즘인데, 케이블이 현재 사용중인지를 확인하고 비어있으면 전송하는 형태이다. 문제는 동시에 복수의 호스트가 미었다고 판단하면 여러곳에서 데이터를 전송하게 되어 충돌이 나게 되고, 데이터가 손상되는데 이런 데이터 손상을 감지할 수도 있다. 충돌이 발생하면 다시 전송하게 되는데 다시 충돌을 방지하기 위해 랜덤한시간 동안 대기하게 하여 타이밍을 어긋나게 하는 방식. 

            단 현재 이더넷은 전송 매체를 공유하지 않게 구성되어 CSMA/CD는 필요하지 않다. 

    - 스타형

        레이어2 스위치를 중심으로 하는 토폴로지로 현재 추세이다. 

    - 링형

![Network_topology](/assets/images/network/Network_topology.png)


# 레이어2 스위치

---

- 하나의 네트워크를 구성하는 기기로 여러대가 연결되어 있어도 하나의 네트워크로 간주된다.
- 이더넷 프레임을 전송하는 역할을 하는데, 이더넷 헤더의 MAC주소만 확인하고 전송한다.
- 여러개 기기가 연결 될 수  있기 때문에 여러개의 이더넷 인터페이스가 있을 수 있고, 클라이언트나 서버에 접속하려면 우선 레이어2 스위치와 먼저 만나기 때문에 네트워크의 입구 역할을 한다. 이런 이유로 '액세스 스위치', '스위칭 허브' 라고 부르는 경우도 많다.

![Network_layer2_switch](/assets/images/network/Network_layer2_switch.png)

## 전송 동작 방식

1. 수신한 이더넷 프레임의 출발지 MAC 주소를 MAC 주소 테이블에 등록
2. 목적지 MAC주소와 MAC주소 테이블에서 전송할 포트를 결정해, 이더넷 프레임을 전송.
MAC주소 테이블에 존재하지 않는 경우, 수신 포트를 제외한 모든 포트로 이더넷 프레임을 전송 한다. (플러딩)

![Network_ethernet_frame_send_1](/assets/images/network/Network_ethernet_frame_send_1.png)
![Network_ethernet_frame_send_2](/assets/images/network/Network_ethernet_frame_send_2.png)



## MAC주소 테이블 관리

위의 예처럼 하나의 포트에 반드시 하나의 MAC주소만 등록되지 않는다. 여러대의 스위치가 연결되는 경우 하나의 포트에 복수의 MAC 주소가 등록 될 수 있다. 예를 들어 위 그림의 요청에 보면 포트 3에 D로 보냈지만 보면 C도 있기 때문에 다음 요청에 C 로 보내지는 경우는 동일 포트에 다른 MAC 주소가 연결되게 된다. 

단, MAC 주소의 접속 포트가 달라지는 경우가 있는데 이를 위해 제한 시간이 존재한다. 스위치 기기마다 다르지만 대략 5분정도라고 한다. 

## 전이중 통신과 반이중 통신

- 전이중 통신 : 데이터 송신과 수신을 동시에하는 것
    - 데이터 수신용, 송신용으로 전송 매체를 나누어 사용하면 가능하다.
    - UTP 케이블을 전송매체로 사용하는데 8개의 구리선을 꼬아서 만드는데 2줄씩 짝을 지어 꼬으기 때문에 4줄이 되고, 전송/수신 나눠서 사용하게 된다.
- 반이중 통신 : 송신과 수신을 통시에 할 수 없어 서로 전환하면서 처리. (전송 매체를 공유하는 버스형 토폴로지인 초기 이더넷의 경우)

# 무선 LAN

---

초기에 사용되었던 이더넷을 대체하기 위해 나온 것이 무선 LAN으로 케이블 없이 간편하게 네트워크를 구축할 수 있는 기술이다. 2000년 무렵부터 저렴한 가격의 제품이 등장하면서 점차 보급되기 시작하였다. 무선 LAN을 사용할 수 있는 기기를 가리켜 무선 LAN 클라이언트라고도 하는데 다음 두가지가 필요하다. 

- **무선 LAN 액세스 포인트**
    - 무선 LAN 액세스 포인트는 공유되어 데이터가 전송이 되는데 이를 인프라스터럭처 모드라고 한다.
- **무선 LAN 인터페이스**
    - 인터페이스끼리 직접 주고받는 경우를 애드혹 모드라고 한다.

무선 LAN 을 사용하더라고 목적지 서버는 보통 유선 이더넷으로 되어 있어 완전 무선 LAN으로 통신이 되는 경우는 드물다. 

![Network_wireless_lan](/assets/images/network/Network_wireless_lan.png)


## 무선 LAN 규격

전파의 주파수 대역을 기준으로 규격이 나뉘는데, 이 또한 IEEE802 위원회에 의해 정해진다. 

WI-FI 는 IEEE802.11 이라는 규격인데 보통 규격의 이름보다는 WI-FI로 많이 통용된다.

무선 LAN 기기 끼리 호환이 잘 되지 않아 제조사가 다르면 연결이 되지 않는 경우가 있었는데, wi-fi aliiance라는 단체가 무선 LAN기기 끼리는 상호 접속성을 인증한 브랜드롤 Wi-Fi라고 부른다. 요즘은 상호 접속을 보증한다는 의미보다는 무선 LAN 자체를 가리켜 Wi-Fi라고 부른다. 

## 무선 랜 통신 방법

### 어소시에이션

- 무선 LAN 액세스 포인트에 연결해 무선 LAN링크를 먼저 확립하는 것을 가리키는데 이더넷으로 치면 케이블 배선에 해당.
- 어소시에이션은 무선 LAN의 논리적 그룹을 식별하는 식별 정보인 SSID(Service Set Identifier)가 필요하다. SSID는 때로는 ESSID(External Service Set Identifier)라고도 부른다. 이 SSID는 무선 LAN액세스 포인트에 최대 32 문자의 문자열로 지정되어 있게되고, 한개의 액세스포인트에 복수의 SSID 설정도 가능하다. 반대로, 복수의 액세스 포인트에 1개의 SSID도 가능하다.

![Network_association](/assets/images/network/Network_association.png)

### 전파

- 무선랜의 전송 매체는 전파인데, 액세스 포인트에서 설정한 특정 주파수대의 전파를 채널이라고 부른다. 무선 랜 엑세스 포인트에 복수의 클라이언트가 어소시에이션 하고 있으면, 복수 클라이언트들으 전파를 공유해서 사용한다. (그래서 느려지는군..)
- 복수의 클라이언트가 동시에 데이터를 전파에 실어보내면 수신하는 쪽에서 데이터 재구성을 할 수 없기 때문에 충돌이 일어나게 되고, 이런 충돌을 막기위해 CSMA/CD를 사용하게 된다.  (위에 이더넷 할때 나옴)
    - 전파가 이용중인지 확인 (Carrier Sense)

        이더넷에는 전송매체가 이용중인지 확인하였지만, 무선 랜에서는 전파가 이용중인지 확인하고, 이용중일때는 대기하는 형태이다.

    - 랜덤 시간 대기 (Collision Avoidance)
    전파가 이용가능하면 바로 보내지 않고 대기 하는데, 랜덤한 시간동안 대기하여 충돌을 방지한다.
    - 데이터 송신

        무선 랜에서는 데이터를 수신했으면 응답으로 ACK를 반환하는데, 이 반환도 전파 사용시간에 포함되어 다른 클라이언트의 대기에 포함이 된다. 

## 전송속도

무선 랜은 구격에서 정한 속도대로 통신할 수 없는 경우가 대부분인데, 그 이유는 전파를 돌려 쓰기 때문이다. 이더넷으로 비교하면 하나의 전송매체를 사용하는 것과 같다. 실질적인 통신 속도를 실효속도 혹은 스루풋 (throughput) 이라고 하는데, 규격상 절반정도 나온다고 보아야한다. 

## 보안

인증 : 액세스 포인트에 정식 사용자만 접속할 수 있게 해야한다. 

암호화 : 전파를 엿들어도 데이터의 내용 자체가 새어나가지 못하게 방지

WPA2 : 무선 랜의 보안 규격으로 IEEE802.11i라고 부른다. 암호화에는 AES(Advanced Encryption Standard), 인증에는 IEEE802.1X를 이용한다.