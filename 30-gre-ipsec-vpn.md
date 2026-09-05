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

