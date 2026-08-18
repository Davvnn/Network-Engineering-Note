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


## 동작 원리

### TCP 연결 설정

1\. 클라이언트는 서버에게 TCP 연결을 요청하기 위해 SYN Segment를 전송한다.

2\. 서버는 해당 TCP Port에서 연결을 기다리고 있다면 SYN과 ACK가 설정된 Segment를 클라이언트에게 전송한다.

3\. 클라이언트는 SYN-ACK를 수신하고 ACK Segment를 서버에게 전송한다.

4\. 마지막 ACK까지 정상적으로 전달되면 TCP Connection State가 `ESTABLISHED`가 되고 Data를 전송할 수 있다.

```
Client → Server: SYN
Server → Client: SYN-ACK
Client → Server: ACK
```

### TCP Data 전송

1\. Application Data에 TCP Header가 추가되어 TCP Segment가 생성된다.

2\. TCP Header에는 Source Port, Destination Port 및 Sequence Number 등의 정보가 포함된다.

3\. 송신 장비는 TCP Segment를 목적지 장비에게 전송한다.

4\. 수신 장비는 Destination Port를 확인하여 해당 Port를 사용하는 Application에 Data를 전달한다.

5\. 수신 장비는 Sequence Number를 확인하여 Data가 올바른 순서로 도착했는지 확인한다.

6\. Data를 정상적으로 수신하면 다음으로 받을 Sequence Number를 Acknowledgment Number로 지정하여 ACK를 전송한다.

7\. 송신 장비는 ACK를 확인하고 다음 Data를 전송한다.

### TCP 재전송

1\. 송신 장비는 TCP Segment를 전송한 후 ACK를 기다린다.

2\. 일정 시간 안에 ACK를 받지 못하면 TCP Segment가 손실된 것으로 판단한다.

3\. 송신 장비는 ACK를 받지 못한 TCP Segment를 다시 전송한다.

4\. 수신 장비는 Sequence Number를 확인하여 재전송된 TCP Segment를 올바른 위치에 재조립한다.

5\. 중복된 TCP Segment를 수신하면 Sequence Number를 확인하여 중복 Data를 제거한다.

### TCP 연결 종료

1\. 클라이언트는 더 이상 전송할 Data가 없다는 것을 알리기 위해 FIN Segment를 서버에게 전송한다.

2\. 서버는 클라이언트의 FIN을 수신하고 ACK를 전송한다.

3\. 서버도 Data 전송을 완료하면 클라이언트에게 FIN Segment를 전송한다.

4\. 클라이언트는 서버의 FIN을 수신하고 마지막 ACK를 전송한다.

```
Client → Server: FIN
Server → Client: ACK
Server → Client: FIN
Client → Server: ACK
```

마지막 ACK를 전송한 장비는 일정 시간 동안 `TIME-WAIT` 상태를 유지할 수 있다.

상황에 따라 ACK와 FIN이 하나의 TCP Segment로 전송될 수 있으므로 항상 정확히 네 개의 Segment가 전송되는 것은 아니다.

### UDP

1\. Application은 전달할 Data를 생성한다.

2\. UDP Header에 Source Port와 Destination Port가 추가되어 UDP Datagram이 생성된다.

3\. UDP Datagram은 IP Packet 안에 Encapsulation된다.

4\. 출발지 장비는 별도의 연결 설정 과정 없이 UDP Datagram을 목적지 장비로 전송한다.

5\. 목적지 장비는 UDP Header의 Destination Port를 확인한다.

6\. 목적지 장비는 해당 Port를 사용하는 Application에 Data를 전달한다.

7\. 응답이 필요한 경우 목적지 Application은 새로운 UDP Datagram을 생성하여 출발지 장비에 전송한다.

UDP 자체에는 Acknowledgment가 없으므로 UDP Datagram을 전송한 것만으로는 목적지 Application이 Data를 정상적으로 수신했는지 확인할 수 없다.

---

## 예시 및 구성도

PC1
- IP Address: 192.168.1.10/24

Server1
- IP Address: 192.168.1.20/24
- TCP Service Port: 443
- UDP Service Port: 5000

### TCP

![](images/05-tcp-operation.png)

PC1이 Server1의 TCP Port `443`에 연결한다.

1\. PC1은 Source Port를 `50000`, Destination Port를 `443`으로 지정한 SYN Segment를 Server1에게 전송한다.

```
PC1 → Server1

SYN
Source Port: 50000
Destination Port: 443
Sequence Number: 1000
```

2\. Server1은 TCP Port `443`에서 연결을 기다리고 있는 것을 확인하고 SYN-ACK Segment를 PC1에게 전송한다.

```
Server1 → PC1

SYN, ACK
Source Port: 443
Destination Port: 50000
Sequence Number: 5000
Acknowledgment Number: 1001
```

3\. PC1은 SYN-ACK를 수신하고 ACK Segment를 Server1에게 전송한다.

```
PC1 → Server1

ACK
Source Port: 50000
Destination Port: 443
Sequence Number: 1001
Acknowledgment Number: 5001
```

4\. 3-Way Handshake가 완료되면 PC1과 Server1의 TCP Connection State가 `ESTABLISHED`가 된다.

5\. PC1은 Sequence Number가 `1001`인 `100 Byte`의 Data를 Server1에게 전송한다.

```
PC1 → Server1

Sequence Number: 1001
Data Length: 100 Byte
```

6\. Server1은 Data를 정상적으로 수신하고 Acknowledgment Number를 `1101`로 지정한 ACK를 PC1에게 전송한다.

```
Server1 → PC1

Acknowledgment Number: 1101
```

7\. Acknowledgment Number가 `1101`이라는 것은 `1100`번 Byte까지 정상적으로 수신했으며, 다음에는 `1101`번 Byte를 기대한다는 의미이다.

8\. 통신이 완료되면 PC1과 Server1은 FIN과 ACK를 주고받아 TCP Connection을 종료한다.

### UDP

![](images/05-udp-operation.png)

PC1이 Server1의 UDP Port `5000`으로 Data를 전송한다.

1\. PC1은 Source Port를 `53000`, Destination Port를 `5000`으로 지정하여 UDP Datagram을 생성한다.

```
UDP 192.168.1.10:53000 → 192.168.1.20:5000
```

2\. PC1은 별도의 연결 설정 과정 없이 UDP Datagram을 Server1에게 전송한다.

3\. Server1은 UDP Header의 Destination Port가 `5000`인 것을 확인한다.

4\. Server1은 UDP Port `5000`을 사용하는 Application에 Data를 전달한다.

5\. 응답이 필요한 경우 Server1의 Application은 PC1에게 새로운 UDP Datagram을 전송한다.

```
UDP 192.168.1.20:5000 → 192.168.1.10:53000
```

6\. UDP Datagram이나 응답이 손실되어도 UDP 자체적으로는 재전송하지 않는다.

---

## 명령어

### Cisco

```
R1# show tcp brief all
```

Cisco 장비에 생성된 TCP Connection과 Connection State를 확인한다.

```
R1# show tcp
```

TCP Connection의 Sequence Number, Acknowledgment Number, Window Size 및 재전송 정보를 확인한다.

```
R1# show udp
```

Cisco 장비에서 사용 중인 UDP Socket 정보를 확인한다.

`show tcp`와 `show udp`는 Cisco 장비 자신이 출발지 또는 목적지가 되는 TCP와 UDP 정보를 확인한다. 장비를 단순히 통과하는 모든 TCP와 UDP 통신을 표시하는 것은 아니다.

### Windows

```
PC1> netstat -ano
```

현재 장비의 TCP Connection, UDP Port 및 Process ID를 확인한다.

```
PC1> netstat -ano | findstr TCP
```

현재 장비의 TCP Connection을 확인한다.

```
PC1> netstat -ano | findstr UDP
```

현재 장비에서 사용 중인 UDP Port를 확인한다.

```
PC1> netstat -ano | findstr :443
```

Port `443`과 관련된 정보를 확인한다.

```
PC1> netstat -ano | findstr LISTENING
```

현재 장비에서 TCP 연결 요청을 기다리고 있는 Port를 확인한다.

```
PC1> tasklist | findstr <PID>
```

`netstat -ano`에서 확인한 Process ID를 사용하는 Program을 확인한다.

### Windows PowerShell

```
PS> Test-NetConnection 192.168.1.20 -Port 443
```

목적지의 TCP Port `443`까지 TCP 연결을 설정할 수 있는지 확인한다.

```
PS> Get-NetTCPConnection
```

현재 장비의 TCP Connection과 Connection State를 확인한다.

```
PS> Get-NetUDPEndpoint
```

현재 장비에서 사용 중인 UDP Port를 확인한다.

---

## Troubleshooting

1\. TCP 또는 UDP 통신이 실패하면 Source IP Address, Destination IP Address, Protocol, Source Port 및 Destination Port가 올바른지 확인한다.

2\. 목적지 장비에서 해당 TCP 또는 UDP Port를 사용하는 Application이 실행되고 있는지 확인한다.

```
PC1> netstat -ano | findstr :443
```

3\. TCP 연결이 실패하면 3-Way Handshake가 어느 단계에서 중단되는지 확인한다.
- SYN을 전송했지만 SYN-ACK를 받지 못함
- SYN-ACK를 받았지만 마지막 ACK가 전달되지 않음
- SYN을 전송한 후 RST를 수신함

4\. SYN을 전송했지만 SYN-ACK를 받지 못하면 목적지에서 연결 요청을 받지 못했거나 응답이 출발지까지 전달되지 못했을 수 있다.

5\. SYN을 전송한 후 RST를 수신하면 목적지 TCP Port에서 연결을 기다리는 Application이 없거나 연결 요청이 거부되었을 수 있다.

6\. TCP Connection State를 확인한다.
- SYN-SENT: SYN을 전송하고 SYN-ACK를 기다리는 상태
- SYN-RECEIVED: SYN-ACK를 전송하고 마지막 ACK를 기다리는 상태
- ESTABLISHED: TCP 연결이 정상적으로 설정된 상태
- CLOSE-WAIT: 상대방의 FIN을 수신하고 연결 종료를 기다리는 상태
- TIME-WAIT: 연결 종료 후 일정 시간 동안 연결 정보를 유지하는 상태

7\. TCP Segment의 재전송이 반복되면 ACK가 돌아오지 않거나 특정 TCP Segment가 손실되고 있을 수 있다.

8\. CLOSE-WAIT 상태가 계속 증가하면 Application이 TCP Connection을 정상적으로 종료하지 않고 있을 수 있다.

9\. UDP는 TCP처럼 Connection State와 Acknowledgment가 없으므로 UDP Datagram을 전송한 것만으로 통신 성공 여부를 판단하기 어렵다.

10\. UDP 응답이 없다면 목적지 UDP Port가 올바른지 확인하고, 해당 Port를 사용하는 Application이 실행되고 있는지 확인한다.

11\. TCP 또는 UDP Port를 다른 Application이 이미 사용하고 있다면 새로운 Application이 같은 Port를 사용하지 못할 수 있다.

12\. TCP Connection이 `ESTABLISHED` 상태이지만 서비스가 동작하지 않는다면 TCP 연결 이후의 Application 동작을 확인해야 한다.

---

## 실무 질문

TCP와 UDP는 어떤 역할을 하는가?
- Transport Layer에서 Port Number를 사용하여 출발지 Application의 Data를 목적지 장비의 Application까지 전달한다.

TCP와 UDP가 사용하는 Protocol Number는 무엇인가?
- TCP는 `6`, UDP는 `17`을 사용한다.

TCP와 UDP의 가장 큰 차이점은 무엇인가?
- TCP는 연결을 설정하고 Data의 수신 확인, 순서 관리 및 재전송 기능을 제공한다. UDP는 연결 설정과 이러한 기능 없이 UDP Datagram을 바로 전송한다.

Port Number는 어떤 용도로 사용하는가?
- 장비에서 실행 중인 여러 Application과 서비스를 구분하기 위해 사용한다.

TCP 3-Way Handshake의 순서는 어떻게 되는가?
- 클라이언트가 SYN을 전송하고 서버가 SYN-ACK로 응답한 후 클라이언트가 ACK를 전송한다.

Sequence Number는 어떤 역할을 하는가?
- 전송되는 Byte의 순서를 구분하여 수신 장비가 Data를 올바른 순서로 재조립할 수 있도록 한다.

Acknowledgment Number는 무엇을 의미하는가?
- 상대방에게 다음으로 받기를 기대하는 Byte의 Sequence Number를 알려준다.

TCP Segment가 손실되면 어떻게 되는가?
- 일정 시간 안에 ACK를 받지 못하거나 Duplicate ACK를 여러 번 수신하면 손실된 TCP Segment를 재전송한다.

TCP FIN과 RST의 차이는 무엇인가?
- FIN은 TCP Connection을 정상적인 절차에 따라 종료할 때 사용하고, RST는 TCP Connection을 즉시 종료하거나 연결 요청을 거부할 때 사용한다.

UDP Datagram이 손실되면 어떻게 되는가?
- UDP 자체적으로는 손실을 확인하거나 재전송하지 않는다. 필요한 경우 Application에서 확인과 재전송 기능을 구현해야 한다.

UDP는 항상 TCP보다 빠른가?
- UDP는 연결 설정과 수신 확인 과정이 없어 처리 과정이 단순하지만, 실제 통신 속도는 Application과 네트워크 상태에 따라 달라지므로 항상 TCP보다 빠르다고 할 수는 없다.

TCP Connection이 `ESTABLISHED` 상태라는 것은 무엇을 의미하는가?
- TCP 3-Way Handshake가 완료되어 두 장비가 TCP Data를 주고받을 수 있는 상태라는 의미이다.
````
