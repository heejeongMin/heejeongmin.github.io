---
layout: post
title: 네트워크의 공통언어 Part 1
subtitle: 그림으로 배우는 네트워크 원리
categories: network
tags: [NETWORK, INFRASTRUCTURE]
---

호스트 : TCP/IP로 통신하는 PC와 스마트폰, 각종 네트워크 기기전반을 호스트라고 부른다. 

## 데이터를 전송하는 역할을 하는 계층 (TCP/IP 모델 기준)

---

### 네트워크 접근 계층
---
- **역할** 
같은 네트워크 안에서 데이터를 전송. 하나의 네트워크는 라우터와 레이어3 스위치로 구획되는 범위 또는 레이어 2 스위치로 구성하는 범위이다. 
[(네트워크를 만드는 것 : 구체적인 네트워크 구성 사진)](/infrastructure/2021/03/21/network-gadgets.html)
- 프**로토콜**
1. 유선 이더넷
2. 무선 LAN(Wi-Fi)
3. PPP (Point-to-Point Protocol)
     ****점대점 프로토콜(영어: Point-to-Point Protocol, PPP)는 네트워크 분야에서 두 통신 노드 간의 직접적인 연결을 위해 일반적으로 사용되는 데이터 링크 프로토콜이다. 점대점 프로토콜은 인증, 암호화[1]를 통한 전송 및 데이터 압축 기능을 제공한다.
[https://ko.wikipedia.org/wiki/점대점_프로토콜](https://ko.wikipedia.org/wiki/%EC%A0%90%EB%8C%80%EC%A0%90_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)
- **데이터 전송 방법**
    - 레이어2 스위치에 연결된 PC의 인터페이스에서 같은 레이어2 스위치에 연결된 다른 PC의 인터페이스까지 데이터를 전송할 수 있다. 이때 '0', '1'의 디지털 데이터를 전기신호 등의 물리적 신호로 변환해 전송 매체로 전달해 간다.
    - 프로토콜이 통신 상대와 같아야할 필요는 없음

![Network_osi_network](/assets/images/network/Network_osi_network.png)
![Network_osi_network_intro](/assets/images/network/Network_osi_network_intro.png)


### 인터넷 계층
---
- **역할**
많은 네트워크 사이에서 데이터를 전송하는 역할. 이때 네트워크끼리 연결하고 데이터를 전송하는 기기가 **라우터.**
- **프로토콜**
1. IP : 엔드투엔드 통신에 사용하는 프로토콜
2. ICMP : IP를 도와주는 프로토콜
3. ARP : IP를 도와주는 프로토콜
- **데이터 전송 방법**
1. 라우팅 : 라우터에 의한 네트워크 간 전송
2. 엔투엔드 통신 : 원격지 네트워크에 연결된 PC 간의 통신
- **IPv4 와 IPv6 차이**
    - IPv4 (Internet Protocol version 4)
    기기가 인터넷에 접근할 때, 고유한 숫자로 구성된 주소를 할당해 준고, 한 기기에서 다른 기기로 데이터를 전송할때 데이터 패킷은 반드시 양쪽 기기의 아이피 주소를 가지고 있어야 한다.
    - IPv6 (Internet Protocol version 6)
    차세대 인터넷 프로토콜로 IPv4를 보조, 궁극적으로는 대체하기 위해 등장함. 현재 사용되고 있는 IPv4는 32-bit 주소를 사용하고 있어서 2^32 만큼의 값만 가질 수 있고, 4.29억개 이지만 현재 할당 할 수 있는 고유한 주소가 모두 할당이 되어 사용할 수 있는 고유주소 고갈 상태이다. 반면 IPv6는 128-bit 주소를 사용할 수 있어더 2^128 만큼의 주소를 가질 수 있다.
    - 그 밖에 IPv6 장점 .. [https://www.thousandeyes.com/learning/techtorials/ipv4-vs-ipv6#:~:text=IPv4 uses a 32-bit address for its Internet addresses.&text=IPv6 utilizes 128-bit Internet,the number of IPv4 addresses](https://www.thousandeyes.com/learning/techtorials/ipv4-vs-ipv6#:~:text=IPv4%20uses%20a%2032%2Dbit%20address%20for%20its%20Internet%20addresses.&text=IPv6%20utilizes%20128%2Dbit%20Internet,the%20number%20of%20IPv4%20addresses).

![Network_osi_internet_intro](/assets/images/network/Network_osi_internet_intro.png)


## 애플리케이션의 동작을 준비하는 계층 (TCP/IP 계층 기준)

---

### 트랜스포트층

- **역할** 
데이터를 적절한 애플리케이션에 배분하는 역할로 우리가 당연한 것처럼 PC로 네트워크를 통해 복수의 애플리케이션을 사용할 수 있는 이유이다.
- **프로토콜과 전송 방법**
1. TCP (Transmission Control Protocol)
   -  어떤 이유로 데이터가 유실되더라도 유실된 데이터를 검출하여 다시 보내기 때문에 신뢰성이 매우 높다. 
2. UDP (User Diagram Oriented Protocol)
  -  TCP처럼 에러 검출은 하지만 재전송하지는 않기때문에 신뢰성은 떨어지지만 TCP 보다 빠르다.
    - TCP, UDP 차이
    [http://guru99.com/tcp-vs-udp-understanding-the-difference.html#:~:text=TCP is a connection-oriented,speed of UDP is faster&text=TCP does error checking and,but it discards erroneous packets](http://guru99.com/tcp-vs-udp-understanding-the-difference.html#:~:text=TCP%20is%20a%20connection-oriented,speed%20of%20UDP%20is%20faster&text=TCP%20does%20error%20checking%20and,but%20it%20discards%20erroneous%20packets).
    - TCP, UDP 사용 실례 
    [https://stackoverflow.com/questions/5330277/what-are-examples-of-tcp-and-udp-in-real-life#:~:text=Real life examples of both,AM](https://stackoverflow.com/questions/5330277/what-are-examples-of-tcp-and-udp-in-real-life#:~:text=Real%20life%20examples%20of%20both,AM))%2C%20Wi%2DFi.

### 애플리케이션층

- **역할**
애플리케이션의 기능을 실행하기 위한 데이터의 형식과 처리 절차 등을 결정한다.
- **프로토콜**
1. HTTP : 웹브라우저
2. SMTP : 전자메일 소프트웨어
3. POP3 : 전자메일 소프트웨어
4. DHCP : 애플리케이션 통신을 준비하기 위한 프로토콜 (애플리케이션 층에 포함된 프로토콜이라고 해서 반드시 사용되는 것은 아니다)
5. DNS : 애플리케이션 통신을 준비하기 위한 프로토콜 (애플리케이션 층에 포함된 프로토콜이라고 해서 반드시 사용되는 것은 아니다)
- 데이터 전송 방식
단순한 '0','1'이 아닌, 문자와 이미지등 사람이 인식할 수 있는 데이터로 표현한다.

![Network_osi_application](/assets/images/network/Network_osi_application.png)

## 데이터 송수신 규칙

---

### 프로토콜의 제어정보 '헤더'를 만든다

![Network_data_transport](/assets/images/network/Network_data_transport.png)

- 통신 주체인 애플리케이션이 데이터를 주고받게 하려면, 복수의 프로토콜을 조합할 필요가 있고, TCP/IP에서는 각 계층의 4개의 프로토콜을 조합한다. 조합할 때 기능을 실현하기 위한 제어정보(헤더)가 필요하다. 
ex ) IP 헤더의 경우 출발지와 도착지의 IP 주소가 헤더 값에 지정됨.
    - 캡슐화 - 헤더를 추가하는 작업
    - 역캡슐화 - 헤더를 보고 프로토콜을 처리 후, 헤더를 제거하고 다른 프로토콜에 처리 넘김

![Network_fcs](/assets/images/network/Network_fcs.png)

 - FCS(Frame Check Seqeunce) - 에러 체크를 위한 정보
- 상위 계층에서 하위 계층으로 내려가면서 여러 프로토콜의 헤더가 추가되는 캡슐화가 진행되고, 마지막에는 물리적인 신호로 데이터 전체를 변환해 전송 매체로 보내게 된다. 

![Network_data_transport_2](/assets/images/network/Network_data_transport_2.png)
![Network_data_receiving](/assets/images/network/Network_data_receiving.png)

1. 물리적인 신호를 0, 1 데이터로 되돌린다
2. **이더넷 헤더**를 참조하여 자기 앞으로 온 데이터인지 확인하고 맞으면 이더넷헤더, FCS를 제거하고 IP 헤더처리를 상위 계층으로 넘김
3. **IP 헤더**를 참조해 자기 앞으로 데이터인지 확인하고 맞으면 IP 헤더를 제고하고 TCP 로 헤더 처리를 넘김
4. **TCP 헤더**를 참조하여 어느 애플리케이션의 데이터인지 확인하고, TCP 헤더를 제거하고 웹서버 애플리케이션으로 데이터 처리를 넘김
5. 웹서버 애플리케이션에서 HTTP 헤더와 데이터를 처리한다. 

## 데이터를 부르는 방법은 다양하다

---

각 계층의 데이터를 부르는 호칭이 따로 있는데 각각 헤더가 추가되면서 완성이 되면 부르는 이름.

1. 애플리케이션 층 : HTTP 메시지
2. 트랜스포트층 : TCP - 세그먼트, UDP - 데이터 그램
3. 인터넷층 : 패킷 또는 데이터그램
4. 네트워크 인터페이스 층 : 프레임

- 라우터가 동작하는 계층과 어떤 데이터를 전송?

    인터넷 계층에서, IP 패킷을 전송

- 레이어2 스위치가 동작하는 계층과 어떤 데이터를 전송?

    네트워크 계층에서 이더넷 프레임 전송

![Network_how_to_call_data](/assets/images/network/Network_how_to_call_data.png)

## IP 주소

---

- TCP/IP에서 통신 상대가 되는 호스트를 식별하기 위한 식별 정보이다.
- 데이터 전송시 데이터에 (TCP 헤더 + HTTP 헤더 + 웹브라우저의 데이터)  IP 헤더를 추가해 IP 패킷을 만든다.
- IP 헤더에는 반드시 출발지와 도착지의 IP 주소가 필요하다.
- IP 주소는 호스트 자체가 아니라 정확하게는 호스트의 인터페이스를 식별한다. 
- 한개의 노트북 PC에 유선 이더넷, 무선 LAN 인터페이스가 같이 탑재된 경우가 많고, 인터페이스 마다 IP 를 설정할 수 있기 때문이다.

![Network_decide_whom_to_call](/assets/images/network/Network_decide_whom_to_call.png)

![Network_what_is_ip](/assets/images/network/Network_what_is_ip.png)

### IP 주소 표기 방법

- 현재 사용하고 있는 IPv4 는 32 비트 임으로 0 과 1 이 32개 나열이된다. 하지만 사람의 입장에서 너무 기니, 8비트씩 10진수로 변환하여 . 으로 구분한것이 우리가 보는 IP 주소이다.

 - 왜 0~255 까지인가 ? 
[https://www.howtogeek.com/341307/how-do-ip-addresses-work/#:~:text=The reason each number can,number the octet can reach](https://www.howtogeek.com/341307/how-do-ip-addresses-work/#:~:text=The%20reason%20each%20number%20can,number%20the%20octet%20can%20reach). 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5ad7389d-1e7e-407c-855f-3042977b795b/_2021-03-27__4.28.36.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5ad7389d-1e7e-407c-855f-3042977b795b/_2021-03-27__4.28.36.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c72ac23c-0697-4bb3-8c39-11a1471ac3f8/_2021-03-27__4.37.06.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c72ac23c-0697-4bb3-8c39-11a1471ac3f8/_2021-03-27__4.37.06.png)

### IP 로 데이터 전송 시 목적지의 개수에 따른 전송 방식

1. **유니캐스트**
    - 단 한곳으로 데이터를 전송
    - TCP/IP 통신의 대부분이 유니캐스트
    - IP 주소 구성의 특징
        - 네트워크부와 호스트부로 나뉜다. (네트워크주소/호스트주소 라고도 함)
        - 네트워크부와 호스트부는 가변이며, 서브넷 마스크로 식별이 가능하다. (아래 설명되어 있음)
        - 왜 192.168.xx.xx 인가?

        [https://docs.oracle.com/cd/E19504-01/802-5753/planning3-18471/index.html#:~:text=The IP address is a,bit fields separated by periods.&text=The bytes of the IP,part and the host part](https://docs.oracle.com/cd/E19504-01/802-5753/planning3-18471/index.html#:~:text=The%20IP%20address%20is%20a,bit%20fields%20separated%20by%20periods.&text=The%20bytes%20of%20the%20IP,part%20and%20the%20host%20part)

        1. 같은 네트워크 상에 있는 모든 PC에 프레임을 전송 / 수신
        2. 각각의 PC는 목적지의 MAC 주소와 자신의 랜 카드 MAC 주소를 비교
        3. 서로 다르다면, CPU에 보내지 않고 받은 프레임 폐기 ⇒ CPU 성능 저하 X

            동일하다면, CPU 위에 프레임 올려서 통신

        .

2. **브로드캐스트**
    - 같은 네트워크 상의 모든 호스트에 똑같은 데이터 전송
    - IP 주소 구성
        - 255.255.255.255 로 정해져 있다. 즉 모든 비트가 1 인 주소이다.
        - 192.168.1.255 인 경우, 192.168.1 까지가 네트워크부인 경우, 마지막 255는 호스트부가 모두 1인경우인데, 255.255.255.255 이외에 이렇게 브로드캐스트 IP를 표현하기도 한다.
    - ARP (Address Resolution Protocol): 두 PC 간에 처음으로 통신하는 경우 상대의 MAC 주소를 알아내기 위해 IP 주소를 MAC으로 바꾸는 과정
    - 주로, IP 주소는 알지만 MAC 주소를 모를 경우에 사용
        1. 같은 네트워크 상에 있는 모든 PC로 프레임 전송 / 수신
        2. 패킷 받은 모든 LAN 카드가 패킷을 CPU로 전송 ⇒ CPU 성능 부하
3. **멀티캐스트**
    - 정의 : 특정 그룹에 포함되는 호스트에 똑같은 데이터를 전송
    - IP 주소 구성
        - 244.0.0.0 ~ 239.255.255.255 로 범위가 정해져 있음.
            - 244.0.0.2 의 경우 미리 정해진 멀티캐스트 IP 주소로, 같은 네트워크 상에 있는 모든 라우터 그룹.
            - 239로 시작하면 사용자가 자유롭게 그룹을 정할 수 있다
            - [https://blog.naver.com/hts0128/220025741723](https://blog.naver.com/hts0128/220025741723)
    - UDP 사용하여 전송하기에 신뢰성 보장이 안된다.
    - 라우터와 스위치가 멀티캐스트를 지원해야 가능한 방식.
    - 원하는 그룹에만 프레임송신하여 적은 네트워크 부하

### 서브넷 마스크

- 32 비트의 IP 주소의 어디까지가 네트워크부인지 명시한 것이 서브넷 마스크이다.
- IP 주소처럼 32비트 이기 때문에 동일하게 0과 1이 32개 나열되고, 알아보기 쉽게 8비트씩 10진수로 변환하고 . 으로 구분한다.
- 1은 네트워크부를 나타내고, 0은 호스트부를 나타낸다. 따라서 연속한 1과 연속한 0으로만 구성된다.
    1. 192.168.1.1 255.255.255.0 —>  IP 주소 앞 세개가 네트워크 부, 마지막이 호스트 부
    2. 192.168.1.1/24 —> 1번과 동일한데, 서브넷 마스크를 연속한 1의 개수가 몇개인지 표시하는 **프리픽스 표기법**이다. 24개이내 8*3 임으로 앞 세자리가 네트워크부
    3. 192.168.1.0/24 —> 호스트부인 마지막 0이, 모두 비트 0으로 채워진 경우이면 네트워크 자체를 식별하기 위한 용도이다. 구성도에 주로 사용된다.

## 네트워크에 접속하는 두 단계

---

1. 물리적 접속
    - TCP/IP 계층의 인터페이스 계층
    - 물리적 신호를 주고 받을 수 있게 하는 것으로 다음과 같다.
        - 이더넷 인터페이스에 LAN 케이블을 삽입
        - 무선 LAN 액세스 포인트에 접속
        - 휴대전화 기지국 전파 포착
2. 논리적 접속
    - TCP/IP 계층의 인터넷 계층
    - 물리적 접속이 이루어진 후에 논리적인 접속으로 IP 주소 설정이 필요하고, 공통언어로 사용하고 있는 TCP/IP에서는 IP 주소를 통하여 통신한다.
        - 호스트  IP 주소가 192.168.1.1/24 인경우, 192.168.1.0/24 를 네트워크부로 가지는 IP 는 통신할 수 있다는 의미