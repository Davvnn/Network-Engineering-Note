# TCP / UDP

## 개념

TCP와 UDP는 Transport Layer에서 동작하며, 출발지 장비의 Application Data를 목적지 장비의 Application까지 전달하는 프로토콜이다. TCP와 UDP는 Port Number를 사용하여 Application을 구분한다.

Application Data에 TCP Header가 추가되면 TCP Segment가 되고, UDP Header가 추가되면 UDP Datagram이 된다. 이후 IP Header가 추가되어 IP Packet으로 Encapsulation된다.
- TCP Protocol Number: `6`
- UDP Protocol Number: `17`

### Port Number

Port Number는 장비에서 실행 중인 Application이나 서비스를 구분하는 번호이다.

TCP와 UDP는 각각 `0`부터 `65535`까지의 Port Number를 사용한다.
- Well-Known Port: `0~1023` HTTP, HTTPS, SSH처럼 널리 알려진 표준 서비스가 사용하는 Port 범위이다.
- Registered Port: `1024~49151` 특정 Application이나 서비스를 위해 등록하여 사용하는 Port 범위이다.
- Dynamic/Private Port: `49152~65535` 클라이언트가 서버에 접속할 때 운영체제로부터 임시로 할당받는 port 범위이다.

서버는 일반적으로 서비스에 Well-Known Port Number를 사용하고, 클라이언트는 운영체제가 임시로 할당한 Dynamic Port Number를 사용한다. 예를 들어 PC1이 Server1의 TCP Port `443`에 접속하면 다음과 같이 Port Number가 지정될 수 있다.

```
TCP 192.168.1.10:50000 → 192.168.1.20:443
```
- Source Port: `50000`
- Destination Port: `443`

Server1이 PC1에게 응답할 때는 Source Port와 Destination Port의 방향이 반대로 변경된다.
```
TCP 192.168.1.20:443 → 192.168.1.10:50000
```

TCP와 UDP의 Port Number는 서로 별도로 관리되어 하나의 장비에서 TCP Port `5000`과 UDP Port `5000`을 동시에 사용할 수 있다.

### TCP

TCP(Transmission Control Protocol)는 Data를 전송하기 전에 3-Way Handshake를 통해 상대방과 연결을 설정하는 Connection-Oriented 프로토콜이다.

TCP는 Sequence Number와 Acknowledgment Number를 사용하여 Data의 순서와 수신 여부를 확인한다. TCP Segment가 손실되면 재전송하고, 순서가 바뀌어 도착하면 올바른 순서로 재조립하며, 중복된 Segment는 제거한다. 또한 Flow Control과 Congestion Control을 통해 전송량을 조절하고, Checksum으로 오류를 확인한다. 통신이 완료되면 FIN과 ACK를 주고받아 연결을 종료한다.

![](images/05-tcp-opening.png)
![](images/05-tcp-closing.png)

TCP Flags
- SYN: TCP 연결 설정 요청
- ACK: TCP Segment를 정상적으로 수신했다는 응답
- FIN: TCP 연결을 정상적으로 종료하기 위한 요청
- RST: TCP 연결을 즉시 종료하거나 연결 요청을 거부

### TCP Header

![](images/05-tcp-header.png)

TCP Header의 기본 크기는 `20 Byte`이며, TCP Option이 포함되면 Header의 크기가 증가할 수 있다.

TCP Header의 구성 요소
- Source Port: 출발지 Application의 Port Number
- Destination Port: 목적지 Application의 Port Number
- Sequence Number: 전송하는 Data의 시작 Byte 번호
- Acknowledgment Number: 다음에 수신하기를 기대하는 Byte 번호
- Header Length: TCP Header의 길이
- Flags: TCP Segment의 연결 상태
- Window Size: 수신 장비가 다음 ACK를 보내기 전까지 추가로 받을 수 있는 Data의 크기
- Checksum: TCP Header와 Data에 오류가 있는지 확인하는 값  
- Options: TCP의 추가 기능 정보

### UDP

UDP는 TCP와 달리 통신 상대방과 연결을 설정하지 않고 즉시 Data를 전송하는 Connectionless 프로토콜이며, TCP보다 작은 Header를 사용한다.

UDP는 Acknowledgment를 통한 수신 확인, 손실된 UDP Datagram의 재전송, 도착 순서 보장, Flow Control 및 Congestion Control 기능을 제공하지 않는다. 하지만 Checksum을 통해 전송 중 발생한 오류를 확인할 수 있다. TCP는 기본적으로 `1:1` 통신을 사용하지만, UDP는 `1:1` 통신뿐만 아니라 Broadcast와 Multicast를 통한 `1:many` 통신도 지원한다.

### UDP Header

![](images/05-udp-header.png)

UDP Header의 크기는 항상 `8 Byte`이다.

UDP Header 구성 요소
- Source Port: 출발지 Application의 Port Number
- Destination Port: 목적지 Application의 Port Number
- Length: UDP Header와 Data를 포함한 전체 길이
- Checksum: UDP Header와 Data의 오류 확인

![](images/05-udp-header.png)














