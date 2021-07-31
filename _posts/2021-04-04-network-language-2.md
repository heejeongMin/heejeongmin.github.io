---
layout: post
title: 네트워크의 공통언어 Part 2
subtitle: 그림으로 배우는 네트워크 원리
categories: infrastructure
tags: [NETWORK, INFRASTRUCTURE]
---

## 인터넷에서 사용하는 주소와 사설 네트워크에서 사용하는 주소

---

- **공인 IP 주소 (퍼블릭 IP 주소, 물리적 주소)** 
- 인터넷에서 사용하는 IP 주소로, 인터넷에서 통신하기 위해서 반드시 공인된 주소가 필요하다. 
- 중복되지 않도록 관리되며, 인터넷 접속 서비스를 계약하면 할당되게 된다.
- **사설 IP 주소** 
- 사내 네트워크등 사설 네트워크에서 이용하는 IP 주소로 범위는 다음과 같다.
   **- 10.0.0.0 ~ 10.255.255.255** → A class 대역 (40억대)
   **- 172.16.0.0 ~ 172.31.255.255** → B class 대역 (65,000 대 정도)
   **- 192.168.0.0 ~ 192.168.255.255**  → C class 대역 (254대 미만일때 사용가능)
**-** 다른 사설네트워크의 사설 주소와 겹치더라도 해당 네트워크안에 통신에는 문제가 없음
- 사설 네트워크에서 인터넷으로 통신할대는 사설 주소로는 안되고 **NAT(Network Address Translation)**가 필요하다.

![Network_public_private_IP](/assets/images/network/Network_public_private_IP.png)


## 사설 네트워크에서 인터넷으로의 통신

---

1. 목적지가 공인 주소이고 출발지가 사설 네트워크인경우 서버로 요청을 보낼 수는 있음. (outbound)
2. 반대로 목적지가 사설 네트워크이고 출발지가 공인네트워크인 경우 응답을 바로 받을 수 없음. (inbound)
**인터넷에서는 목적지가 사설 주소로된 IP 패킷은 폐기하기 때문이다.  → N 개의 사설망에서는 사설 아이피가 중복될수 있기때문에 !!!!** 
    - 이럴때 **사설 IP를 NAT 주소로 변환**하여 사용한다.
        1. 목적지: 공인주소, 출발지 : 사설주소
        : 출발지의 IP 주소를 변환하여 요청을 보낸다.
        2. 라우터가 출발지의 변환된 주소와 기존 사설주소 모두를 NAT 테이블에 보관해둔다. 
        3. 목적지 : NAT 주소, 출발지 : 공인주소
        : 요청에 대한 응답이 라우터로 돌아오면 NAT 테이블에서 저장해둔 NAT 주소의 원래 주소를 찾아 변환
    - 사설주소 1 개와 공인주소 1개씩 매핑을 하면 공인주소가 너무 많이 필요해지기 때문에 
    여러개의 사설주소를 하나의 글로벌 주소에 대응시키는 주소변환을 
    **NAP(Network Address Port Translation)** 이라고 부른다.
        - **Static NAT (1:1 NAT)** : 공인 IP와 사설 IP가 1:1 매핑되는 방식으로 공인 주소 절약의 효과는 없으나, 주로 사설 IP를 사용하는 서버가 여러가지 역할을 할때 포트포워딩 목적으로 사용한다. 
        * 포트 포워딩 : 하나의 서버에서 여러 서비스를 운영중인 경우, 특정 서비스에 임의의 포트를 지정하여 해당 포트를 통해 특정 서비스의 경로를 지정해주는 역할
        - **Dynamic NAT (N:N NAT)** : 공인 IP 주소 대비, 사설 IP 주소가 더 많을 경우 사용
        - **PAT (Port Address Translation), NAPT (Network Address Port Translation) ( 1 : N)**
        : 공인 IP 주소 1개에 사설 IP 주소 여러개가 매핑되는 방식. 특히 PAT 은 포트 번호까지 지정함.
    - NAT 를 사용하는 또 다른 이유는 **보안**이다.
        1. 라우터를 통해 외부로 트래픽이 나갈 때 사설 IP가 공인 IP 주소로 바뀌므로 공격자가 라우터 안 쪽에 있는 사설 IP 를 알 수 없기 때문에 최정 목적지로의 공격이 어려워져 내부 네트워크 및 호스트를 보호할 수 있다. 

![Network_NAT](/assets/images/network/Network_NAT.png)


참고 
[https://www.stevenjlee.net/2020/07/11/이해하기-nat-network-address-translation-네트워크-주소-변환/](https://www.stevenjlee.net/2020/07/11/%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-nat-network-address-translation-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A3%BC%EC%86%8C-%EB%B3%80%ED%99%98/)



## 데이터가 목적지에 잘 도착했을까 ? (ICMP)


---

 1. IP
        - 데이터를 다른 호스트에 전송은 하지만 제대로 잔달이 되었는지에 대한 여부는 보장하지 않는다. 이런 데이터 전송의 특징을 최선형 (best effort) 라고 하는데, '데이터를 보내기 위해' 최선을 다했지만 안되도 어쩔수 없다는 IP의 특징을 말한다.
 2. ICMP (Internet Control Message Protocol)
        - IP의 best effort틀징을 개선하기위해, 즉, IP에 의한 엔드투엔드 통신이 정상적으로 이루어졌는지 확인하기위한 기능을 가진 ICMP가 개발된다.
        - 주요기능은 **에러리포트**와 **진단기능**이다.
            - 에러리포트 : 어떠한 이유로 IP 패킷이 폐기가 되었다면 ICMP가 폐기된 IP 패킷을 출발지로 에러리포트를 전송하고 이 리포트를 "도달불능 메시지" 라고 부르며, 이 리포트로 통신실패 원인을 알 수 있다.
            - 진단기능 : 주로 ping 커맨드를 사용하여 ICMP 에코 요청/응답 메시지를 보내서 지정한 IP 주소와 통신할 수 있는지 확인한다.
        - 지난 장에 인터넷 계층 : [네트워크 공통언어 Part1](/infrastructure/2021/03/28/network-language-1.html)

## IP 주소와  MAC 주소를 대응시킨다.  (ARP)

---

### TCP/IP에서는 IP 주소를 지정해서 데이터(IP 패킷)을 전송 
→ IP 패킷은 PC나 서버등의 인터페이스까지 전송 
→ PC나 서버등의 인터페이스에서 MAC 주소를 식별을 ARP(Address Resolution Protocol) 를 이용하여 주소해석 을 한다. 

![Network_ip_dest](/assets/images/network/Network_ip_dest.png)

![Network_dest_IP_MAC](/assets/images/network/Network_dest_IP_MAC.png)


### ARP 동작 흐름

- 주소 해석 범위 : 동일 네트워크에 있는 IP 주소
- 이더넷 인터페이스로 접속된 PC등의 기기가 IP 패킷을 송신하고자 목적지 IP 주소를 지정할때 자동으로 ARP가 실행된다.
    - ARP 요청으로 IP 주소에 대응하는 MAC 주소 질의
    - 질의받은 IP 주소를 가진 호스트가 ARP 응답으로 MAC주소를 알려줌
    - IP 주소와 MAC 주소 매핑을 ARP 캐시에 보존


![Network_ARP](/assets/images/network/Network_ARP.png)

## 포트 번호로 애플리케이션에 할당한다.

---

### 포트 번호의 역할

1. TCP/IP의 애플리케이션을 식별하기 위해 식별번호로 포트번호를 사용하는데, 호스트에서 동작하는 애플리케이션에 데이터를 분배하기 위해서는 각각의 애플리케이션을 식별할 수 있어야한다. 
2. 포트번호는 TCP 혹은 UDP의 헤더에 지정한다. 
3. 포트번호는 16비트의 수치로, 지정할 수 있는 범위는 0~65535 이며 범위마다 의미가 다르다.     
    - 0 ~ 1023  : "웰노운포트" 라고도 부르며 서버 애플리케이션 용으로 예약된 포트
    - 1024 ~ 49151 : "등록된 포트"로 자주 이용되는 애플리케이션의 서버 쪽 포트 번호
    - 49152 ~ 65535 : "동적/사설 포트:로 클라이언트 애플리케이션용 포트 번호

    | 프로토콜 |  TCP  |  UDP  | REMARK |
    | :----:  | ----: | ----: | :--------- |
    |HTTP    | 80    | -     | -  |
    |HTTPS   | 44    | -     | -  |
    |SMTP    | 25    | -     | SIMPLE MAIL TRANSFER PROTOCOL   |
    |POP3    | 110   | -     | POST OFFICE PROTOCOL 3 |
    |IMAP4   | 143   | -     | INTERNET MAIL ACCESS PROTOCOL VERSION 4 |
    |FTP     | 20/21 | -     | FILE TRANSFER PROTOCOL |
    |DHCP    | -     | 67/68     | DYNAMIC HOST CONFIGURE PROTOCOL |




## 확실하게 애플리케이션의 데이터를 전송한다.

---

### TCP

#### 신뢰성 있는 애플리케이션 간의 데이터를 전송하기 위한 프로토콜로 TCP 를 사용하면, 애플리캐이션 프로콜에는 신뢰성을 확보하기 위한 구조를 넣을 필요가 없다. 

![Network_tcp](/assets/images/network/Network_tcp.png)


#### 전송절차

   a. **TCP 커넥션 맺기**

   - 데이터를 송수신하는 애플리케이션 간의 통신이 정상적으로 이루어질 수 있는지 먼저 확인하는 절차로, 이 프로세스를 3웨이 핸드쉐이크라고 부른다.

        Client > Server : TCP SYN
        Server > Client : TCP SYN ACK
        Client > Server : TCP ACK

        (SYN : Synchronize Sequence Numbers, ACK : Acknowledgement)

        **[STEP 1]**

        A클라이언트는 B서버에 접속을 요청하는 SYN 패킷을 보낸다. 이때 A클라이언트는 SYN 을 보내고 SYN/ACK 응답을 기다리는SYN_SENT 상태가 되는 것이다.

        **[STEP 2]**

        B서버는 SYN요청을 받고 A클라이언트에게 요청을 수락한다는 ACK 와 SYN flag 가 설정된 패킷을 발송하고 A가 다시 ACK으로 응답하기를 기다린다. 이때 B서버는 SYN_RECEIVED 상태가 된다.

        **[STEP 3]**

        A클라이언트는 B서버에게 ACK을 보내고 이후로부터는 연결이 이루어지고 데이터가 오가게 되는것이다. 이때의 B서버 상태가 ESTABLISHED 이다.

   출처:

   [https://mindnet.tistory.com/entry/네트워크-쉽게-이해하기-22편-TCP-3-WayHandshake-4-WayHandshake](https://mindnet.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-22%ED%8E%B8-TCP-3-WayHandshake-4-WayHandshake)

   ![Network_application_send_data](/assets/images/network/Network_application_send_data.png)


  b. **애플리케이션 간 데이터 송수신**

   - 애플리케이션 데이터에 프로토콜 헤더와 TCP 헤더를 추가하는데 이를 TCP 세그먼트라고 한다. 
   - 이때 애플리케이션의 데이터가 크면 TCP 헤더가 여러개일 수 있고, 분할/조립 방법도 함께 명시해져 있다.
   - 만일 일부 데이터가 제대로 도착하지 않았다면 데이터를 재전송하기도 하며, 네트워크가 혼잡하지면 전송속도를 제어하기도 하는데 이를 플로우 제어라고 한다.

  c. **TCP 커넥션 끊기**


#### TCP 헤더 형식
   - TCP 헤더에서 가장 중요한 것은 포트번호이다. 포트번호를 사용하여 적절한 애플리케이션 프로토콜에 데이터를 배분할 수 있기 때문이다.
   - 그리고 중요한것은, 시퀀스번호와 ACK번호이다.
        - 시퀀스 번호 : 이름처럼 TCP로 전송하는 데이터의 순서를 나타낸다. 데이터가 분할되어있을때 시퀀스 번호를 보고 어떻게 분할 되었는지 알 수 있다.
        - ACK 번호 : 데이터를 올바르게 수신했음을 확인하기 위한 용도이다.

![Network_tcp_header](/assets/images/network/Network_tcp_header.png)


#### 데이터의 분할구조

- MSS(Maximum Segment Size)
    - TCP에서 애플리케이션 데이터를 분할하는 단위
    - 표준 크기는 1460 바이트이고, 이 범위를 넘으면 MSS 단위로 나누어 보내게 된다.
    - 크기의 기준은 전송계층 까지 갔을때 이더넷의 MTU(Maximum Transmission Unit)인 1500바이트를 넘지 않기 위함인것으로 보이고, 이후 5장에 설명이 나오게 된다.

![Network_data_split](/assets/images/network/Network_data_split.png)


## 애플리케이션에 데이터를 배분만 하는경우 ? - UDP

---

TCP 와 같은 확인 절차를 걸치지 않고 PC나 서버등에 도달한 데이터를 적절한 애플리케이션에 배분하는 기능만 있는 프로토콜이 UDP이다. 

- UDP 헤더와 애플리케이션의 데이터를 합쳐 UDP  데이터 그램이라고 부른다.
- UDP는 상대방의 애플리케이션 동작 여부에 상관없이 무조건 UDP 데이터그램으로 데이터를 전송하고, 도달 여부도 알 수 없다.
- TCP 보다 빠르지만 신뢰성이 높지 않다는 단점이 있다.
- 데이터 크기가 커도 TCP 처럼 분할하는 기능이 없기때문에 수신하는 애플리케이션 쪽에서 적절한 크기로 쪼개야한다. 
예를 들어 IP 전화의 경우 IP 전화의 설정에 따라 음성데이터가 분할되지만 전송하는 UDP 에서는 설정에의해서 들어온 데이터에 헤더만 붙여서 그대로 전송하게 된다.

![Network_udp](/assets/images/network/Network_udp.png)



## 네트워크의 전화번호부 - DNS

---

- 상대방의 주소가 숫자로 이루어진 IP 주소로 되어 있기 때문에 사용자 입장에서 이해하기는 어렵다. 그래서, 애플리케이션이 동작하는 서버는 클라이언트 PC 등의 호스트의 사용자가 이해하기 쉽게 **호스트명**을 붙여준다.
- 사용자가 사용하는 URL 주소, 메일주소 등이 호스트명 자체나 호스트 이름을 구분하기 위한 정보를 포함하고 있다.
- 이런 호스트 이름에 대응하는 IP 주소를 자동으로 구해주는 것이 **DNS(Domain Name System)** 의 역할이다.

### DNS 서버

- DNS를 이용하려면 DNS 서버가 필요하서 서버에 미리 사용하고자 하는 호스트명과 IP 주소의 매핑 관계를 등록해 놓아야 한다.
- 매핑관계외에도 여러 정보를 저장하는데 이러한 정보를 리소스 레코드라고 부른다.

    | 종류 |  의미 |  
    | :----:  | :---- |
    | A   | 80    |
    | AAAA   | 44    | 
    | CNAME    | 25    |
    | MX    | 110   |
    | NS   | 143   |
    | PTR     | 20/21 |



### DNS의 동작방법

![Network_dns](/assets/images/network/Network_dns.png)

DNS는 루트를 정점으로 한 계층 구조로 구성되어 있고, 동작 방식은 다음과 같다. 

1. DNS 서버에 필요한 정보(리소스 레코드)를 등록한다. (사용자)
2. 애플리케이션이 동작하는 호스트에 DNS서버의 IP 주소를 설정한다. 애플리케이션을 이용하는 사용자가 호스트 이름을 지정하면 자동으로 DNS서버에 대응하는 IP 주소를 질의하는데 이런 기능은 OS에 내장되어 있고,  DNS리졸버라고 부른다.
3. 처음 질의하여 호스트명에 맞는 IP 주소를 찾을때에는 루트부터 시작해서 계층구조를 내려가며 질의하는데 이 행위를 재귀 질의라고 한다. 
4. 요청이 발생할때마다 재귀질의를 하는것은 아니고 한번 찾은 정보는 한동안 캐시에 보관되게 되고 캐시에 있으면 재귀질의를 시작하지 않고 바로 사용할 수 있다. 

### **DNS 조회의 8단계:**

1. 사용자가 웹 브라우저에 'example.com'을 입력하면, 쿼리가 인터넷으로 이동하고 DNS 재귀 확인자가 이를 수신합니다.
2. 이어서 확인자가 DNS 루트 이름 서버(.)를 쿼리합니다.
3. 다음으로, 루트 서버가, 도메인에 대한 정보를 저장하는 최상위 도메인(TLD) DNS 서버(예: .com 또는 .net)의 주소로 확인자에 응답합니다. example.com을 검색할 경우의 요청은 .com TLD를 가리킵니다.
4. 이제, 확인자가 .com TLD에 요청합니다.
5. 이어서, TLD 서버가 도메인 이름 서버(example.com)의 IP 주소로 응답합니다.
6. 마지막으로, 재귀 확인자가 도메인의 이름 서버로 쿼리를 보냅니다.
7. 이제, example.com의 IP 주소가 이름 서버에서 확인자에게 반환됩니다.
8. 이어서, DNS 확인자가, 처음 요청한 도메인의 IP 주소로 웹 브라우저에 응답합니다.

DNS 조회의 8단계를 거쳐 example.com의 IP 주소가 반환되면, 이제 브라우저가 웹 페이지를 요청할 수 있습니다.

1. 브라우저가 IP 주소로 **[HTTP](https://www.cloudflare.com/ko-kr/learning/ddos/glossary/hypertext-transfer-protocol-http)** 요청을 보냅니다.
2. 해당 IP의 서버가 브라우저에서 렌더링할 웹 페이지를 반환합니다(10단계).

참고 
[https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns/](https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns/)

## 필요한 설정을 자동화 한다. - DHCP

---

TCP/IP를 이용해 통신하기 위해서는 각종 네트워크 기기에 TCP/IP 설정이 올바르게 되어 있어야하는데, 실수를 최소화하기 위해서 설정을 자동화하는 방법이 있는데, 설정을 자동화하는 프로토콜이 DHCP이다. 

DHCP (Dynamic Host Configuration Protocol) 

1. DHCP Discover : PC에서 DHCP서버가 있는지 브로드캐스팅 방식을 통해 DHCP Discover 메시지를 보내게 된다.
2. DHCP Offer : DHCP에서 PC에게 줄 수 있는 IP주소 리스트를 제공해준다.
3. DHCP Request : 리스트중 IP 하나를 선택하여 제공을 요청한다.
4. DHCP Ack : IP 및 네트워크 정보를 할당해주고 임대 기간을 알려준다.

**장점**

- IP를 자동으로 할당해주기 때문에 IP 충돌을 사전에 방지 할 수 있다.

**단점**

- 현재 PC가 DHCP서버에 의존하여 IP 주소를 할당받으므로 DHCP 서버가 다운되면 IP를 받지 못하여 인터넷 사용이 불가능 할 수 있다.

출처:
[https://www.crocus.co.kr/1414](https://www.crocus.co.kr/1414)

![Network_dhcp](/assets/images/network/Network_dhcp.png)
