# OSI 7 계층
![network_layer](./img/network_layer.png)

## L1 물리 계층 (Physical Layer)
데이터를 전기적인 신호(Bit)로 변환하여 전송하는 공간. 데이터를 전송하는 역할만 담당한다.

## L2 데이터 링크 계층 (Data Link Layer)
데이터 링크 계층은 물리 계층에서 송수신되는 정보를 MAC 주소가 포함된 프레임 단위의 데이터로 전달하는 역할을 담당한다. 

### Frame
L2의 데이터 전송 단위는 **프레임(Frame)**이다. 프레임은 대략 10KB 정도의 데이터이며 

### MAC

L2의 데이터 전송 대상 식별자는 **MAC 주소**이다. 하드웨어 주소, 물리 주소라고 하며 NIC (Network Interface Card) (혹은 LAN) 카드의 고유 식별자이다.

48 bit로 구성되어 있으며 16진수로 표기한다. MAC 주소의 예시는 다음과 같다.
- 5C:1C:2A:3B:4F:55

### NIC
컴퓨터를 네트워크에 연결하는 하드웨어 장치이며, 랜카드 (LAN Card)라고도 불린다. L2에서의 데이터 전송 대상의 식별자인 MAC 주소를 가지고 있다.

### L2 Switch
**L2 Swtich 혹은 스위칭 허브**라고 불리는 장비는 MAC 주소를 기반으로 연결된 기기들을 식별하여 프레임을 전달한다.

## L3 네트워크 계층 (Network Layer)
최적의 경로(Route)와 주소(IP)를 정하고 패킷을 전달해주는 역할을 담당한다. 

### Packet
L3의 데이터 전송 단위는 **Packet**이다. 패킷의 최대 크기는 1500 바이트 (1.4KB) 이며 **MTU (Maximum Transmission Unit)** 이라고 한다.

### IP
![ip](./img/ip.png)
IPv4 기준으로 **32비트 주소 체계**를 사용하며 8bit x 4의 구조를 가진다. 처음 24비트는 Network ID, 마지막 8비트는 Host ID로 구성된다.

Network ID는 시군구 주소, Host ID는 동 호수와 같은 상세 주소 정도로 비유할 수 있다.

## L4 전송 계층 (Transport Layer)
송수신자간의 신뢰성있는 데이터를 주고 받게 해주는 역할을 한다. 데이터의 오류검출, 흐름제어, 중복 검사등을 수행한다. 포트를 열고 TCP와 UDP 프로토콜을 통해 데이터를 전달한다.

## L5 세션 계층 (Session Layer)
데이터 통신을 위한 논리적인 연결을 설정한다. TCP/IP 세션을 만들고 없애는 역할을 담당한다

## L6 표현 계층 (Presentation Layer)
전송하는 데이터의 표현방식을 결정한다. 데이터의 암복호화, 압축, 파일 인코딩 등을 수행한다

## L7 응용 계층 (Application Layer)
최종 사용자와 직접 상호작용하는 서비스나 프로세스가 동작하는 영역이다. HTTP, FTP

