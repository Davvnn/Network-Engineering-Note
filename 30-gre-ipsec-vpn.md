# GRE Tunnel / IPsec / Site-to-Site VPN

## 개념

### VPN

VPN(Virtual Private Network)은 Internet과 같은 공용 Network를 통해 멀리 떨어진 Network를 안전하게 연결하는 기술이다.

### Site-to-Site VPN

Site-to-Site VPN은 ISP가 제공하는 Internet을 통해 본사와 지사처럼 서로 떨어진 두 Network를 Router나 Firewall로 안전하게 연결하는 방식이다.

일반적인 IPsec Tunnel Mode에서는 VPN 장비가 원본 Packet을 암호화하고 새로운 Outer IP Header를 추가한다.
- 기업에서는 일반적으로 Firewall이나 Router가 VPN 장비 역할을 수행하며, Site-to-Site VPN은 주로 Firewall에 구성한다.

ISP Router는 Outer Destination IP Address를 기준으로 Packet을 상대방 VPN 장비까지 Forwarding한다. 원본 Packet의 내부 IP Address, Port Number 및 Data는 ESP로 암호화되어 있기 때문에 확인할 수 없다.

### GRE Tunnel

GRE(Generic Routing Encapsulation)는 원본 Packet에 GRE Header와 새로운 IP Header를 추가하여 다른 Network로 전달하는 Tunneling Protocol이다.
```
New IP Header | GRE Header | Original IP Packet
```
GRE Tunnel을 생성하면 물리적으로 떨어진 두 Router의 Tunnel Interface를 하나의 가상 Point-to-Point Link로 연결할 수 있다.

물리적으로 Packet은 여러 ISP Router를 통해 전달된다. ISP Router는 새로운 Outer IP Header의 Destination IP Address만 확인하여 GRE Packet을 상대방 Router까지 Forwarding한다.

상대방 Router는 Outer IP Header의 Source와 Destination IP Address, GRE Protocol Number를 자신의 Tunnel 설정과 비교한다. 정보가 일치하면 Outer IP Header와 GRE Header를 제거하고, 내부의 원본 Packet을 Tunnel Interface에서 수신한 것으로 처리한다.
- 따라서 실제로는 ISP Network를 통과하지만, 원본 Packet을 처리하는 Overlay Network에서는 두 Router의 Tunnel Interface가 직접 연결된 것처럼 동작한다.

GRE Tunnel은 물리적으로 멀리 떨어진 두 Router를 가상의 Point-to-Point Link로 연결하고 Routing Protocol의 Hello Message를 전달할 수 있기 때문에, 두 Router는 OSPF나 EIGRP Neighbor를 형성할 수 있다.  

GRE 자체는 Packet을 암호화하지 않기 때문에 보안이 필요한 경우 GRE over IPsec을 구성한다. IPsec Tunnel Mode에서는 GRE Packet을 암호화하고 새로운 Outer IP Header를 추가한다. ISP Router들은 Outer IP Header를 확인하여 Packet을 Forwarding하며, 내부 IP Address, Routing Protocol Message 및 Data는 암호화되어 있기 때문에 확인할 수 없다.
- GRE는 IP Protocol Number `47`을 사용한다.

### Underlay와 Overlay

Underlay Network는 GRE Packet을 실제로 전달하는 물리적인 Network이다.

Overlay Network는 GRE Tunnel을 통해 논리적으로 연결된 가상의 Network이다.
```
Underlay: R1과 R2의 Public IP Address를 연결하는 Internet 경로
Overlay: R1과 R2의 Tunnel Interface를 연결하는 가상 경로
```
GRE Tunnel이 동작하려면 먼저 Underlay Network를 통해 상대방 Tunnel Destination까지 통신할 수 있어야 하며, 이후 그 위에 Overlay Network를 구축할 수 있다.

### GRE Overhead와 MTU

기본 GRE Encapsulation은 새로운 IP Header `20 Byte`와 GRE Header `4 Byte`를 추가한다.

따라서 기본적으로 총 `24Byte`의 Overhead가 발생한다.
```
Original Packet: 1500Byte
GRE Overhead: 24Byte
최종 Packet: 1524Byte
```
Ethernet을 사용하는 Underlay Network의 MTU는 일반적으로 `1500 Byte`이다. 원본 Packet에 GRE Header와 새로운 IP Header를 포함한 `24 Byte`가 추가되면 Ethernet MTU를 초과하여 Fragmentation이 발생할 수 있다. 따라서 GRE Tunnel Interface의 MTU를 `1476 Byte`로 조정하여 Fragmentation을 줄일 수 있다.
```
R1(config-if)# ip mtu 1476
R1(config-if)# ip tcp adjust-mss 1436
```
- `ip mtu 1476`: Ethernet MTU `1500 Byte`에서 GRE Encapsulation으로 추가되는 새로운 IP Header `20 Byte`와 GRE Header `4 Byte`를 제외한 값이다.
- `ip tcp adjust-mss 1436`: Tunnel MTU `1476 Byte`에서 기본 IPv4 Header `20 Byte`와 TCP Header `20 Byte`를 제외한 값이다.
```
Data 1436 Byte
→ TCP Header 20 Byte 추가
= TCP Segment 1456 Byte

→ Original IP Header 20 Byte 추가
= Original IP Packet 1476 Byte

→ GRE Header 4 Byte + New Outer IP Header 20 Byte 추가
= Total GRE Packet 1500 Byte
```
