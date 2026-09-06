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

### Recursive Routing

Recursive Routing은 GRE Tunnel Destination으로 가는 경로가 다시 GRE Tunnel Interface를 가리키는 문제이다.

여기서 Tunnel Destination은 상대방 Router의 Tunnel Interface IP Address가 아니라, GRE Packet을 실제로 전달할 상대방 Router의 WAN IP Address이다.

예를 들어 R1의 Tunnel Destination이 `198.51.100.2`인 경우, R1은 해당 IP Address로 가는 경로를 Routing Table에서 확인한다. 정상적인 경우에는 물리적인 WAN Interface를 통해 ISP로 Packet을 전달해야 한다.

그러나 Dynamic Routing Protocol을 통해 `198.51.100.2`로 가는 경로를 `Tunnel0`으로 학습하면, GRE Tunnel을 생성하기 위해 다시 GRE Tunnel을 사용하려는 문제가 발생한다.

이를 방지하기 위해 Tunnel 내부에서는 본사와 지사의 내부 LAN Network만 광고하고, Tunnel Destination의 Public IP Address와 Underlay Network는 광고하지 않아야 한다.

가장 확실한 방법은 Tunnel Destination에 대한 `/32` Static Route를 설정하여 해당 Traffic이 물리적인 WAN Interface를 통해 전달되도록 하는 것이다.

### IPsec

IPsec(Internet Protocol Security)은 IP Packet을 암호화하고 인증하여 안전하게 전달하는 기술이다.

IPsec은 다음 보안 기능을 제공한다.
- Authentication(인증): 신뢰할 수 있는 VPN Peer인지 확인한다.
- Integrity(무결성): 전송 중 Packet이 변경되거나 조작되지 않았는지 확인한다.
- Confidentiality(기밀성): Packet 내용을 암호화하여 다른 장비가 확인하지 못하도록 한다.
- Anti-Replay(재전송 공격 방지): 공격자가 이전 Packet을 다시 전송하는 것을 방지한다.

### AH

AH(Authentication Header)는 Packet의 인증과 무결성을 제공하지만 암호화는 제공하지 않는다.
- 무결성은 Packet이 전송되는 중간에 변경되거나 조작되지 않았는지 확인하는 기능이다.
- Packet을 암호화하지 않기 때문에 TCP/UDP Header와 Data의 내용을 확인할 수 있다.

![](images/30-ipsec-ah.png)
- AH Transport Mode에서는 기존 IP Header와 TCP/UDP Header 사이에 AH Header가 추가된다.
- AH Tunnel Mode에서는 원본 IP Packet 앞에 AH Header와 새로운 IP Header가 추가된다.
- AH는 IP Protocol Number `51`을 사용한다.

AH는 Data와 IP Header 일부의 무결성을 확인한다. 하지만 NAT 장비가 Source 또는 Destination IP Address를 변경하면 AH의 무결성 확인에 실패할 수 있기 때문에 NAT와 함께 사용하기 어렵다.

### ESP

ESP(Encapsulating Security Payload)는 Packet의 암호화, 인증 및 무결성을 제공한다.

![](images/30-ipsec-esp.png)
- ESP Header는 암호화된 Data 앞에 추가되고, ESP Trailer와 ESP Authentication 정보는 뒤에 추가된다.
- ESP Transport Mode에서는 기존 IP Header를 유지하고 TCP/UDP Header와 Data를 암호화한다.
- ESP Tunnel Mode에서는 원본 IP Packet 전체를 암호화하고 새로운 IP Header를 추가한다.
- ESP는 IP Protocol Number `50`을 사용한다.

일반적인 IPsec VPN에서는 암호화를 제공하지 않는 AH보다 암호화와 무결성을 함께 제공하는 ESP를 사용한다.

### Transport Mode

Transport Mode는 기존 IP Header를 유지하고 IP Packet의 Payload를 보호하는 방식이다.

기존 IP Header는 암호화되지 않기 때문에 Router는 Source와 Destination IP Address를 확인하여 Packet을 Forwarding할 수 있다.

### Tunnel Mode

Tunnel Mode는 원본 IP Packet 전체를 보호하고 새로운 Outer IP Header를 추가하는 방식이다.

새로운 Outer IP Header에는 VPN 장비의 Public IP Address가 Source와 Destination으로 설정된다.

ISP Router는 새로운 Outer IP Header의 Destination IP Address를 확인하여 상대방 VPN 장비까지 Packet을 Forwarding한다. 원본 IP Address, TCP/UDP Header 및 Data는 ESP로 암호화되어 있기 때문에 확인할 수 없다.

일반적인 Site-to-Site IPsec VPN에서는 서로 다른 Network의 원본 IP Packet 전체를 보호하기 위해 주로 Tunnel Mode를 사용한다.
