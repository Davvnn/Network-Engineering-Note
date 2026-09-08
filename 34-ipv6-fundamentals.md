# IPv6 Fundamentals

## 개념

### IPv6

IPv6(Internet Protocol Version 6)는 IPv4 Address 부족 문제를 해결하기 위해 만들어진 Network Layer Protocol이다.

IPv4는 `32bit` Address를 사용하지만 IPv6는 `128bit` Address를 사용한다.

```
IPv4: 192.168.10.10
IPv6: 2001:db8:10::10
```

IPv6는 다음 특징을 가진다.
- Broadcast를 사용하지 않고 Multicast를 사용한다.
- NDP를 사용하여 Neighbor와 Default Gateway를 확인한다.
- SLAAC를 사용하면 DHCP Server 없이 Address를 자동으로 생성할 수 있다.
- IPv4와 별도의 IPv6 Routing Table을 사용한다.

### IPv6 Address 표기법

IPv6 Address는 `16bit`씩 8개의 영역으로 나누어 16진수로 표시한다.
```
2001:0db8:0000:0000:0000:0000:0000:0010
```

각 영역 앞의 `0`은 생략할 수 있다.
```
2001:db8:0:0:0:0:0:10
```

연속된 `0` 영역은 `::`로 한 번만 줄일 수 있다.
```
2001:db8::10
```
- 하나의 IPv6 Address에서 `::`는 한 번만 사용할 수 있다.

### Prefix Length

IPv6는 IPv4의 Subnet Mask 대신 Prefix Length를 사용한다.
```
2001:db8:10::1/64
```

일반적인 LAN Network에서는 `/64` Prefix를 사용한다.
```
Network Prefix: 처음 64bit
Interface ID: 나머지 64bit
```

### IPv6 Address Type

#### Global Unicast Address

Global Unicast Address는 IPv4의 Public IP Address와 비슷하게 Internet이나 외부 Network에서 Routing할 수 있는 Address이다.

일반적으로 `2000::/3` 범위를 사용한다.
```
2001:db8:10::10/64
```

#### Link-Local Address

Link-Local Address는 같은 Link에 연결된 장비끼리 통신할 때 사용하는 Address이다.

`FE80::/10` 범위를 사용하며 IPv6가 활성화되면 Interface에 자동으로 생성된다.
```
FE80::1
```
- Link-Local Address는 Router를 넘어 전달되지 않는다.

다음 기능에서 주로 사용한다.
- NDP를 통한 Neighbor 확인
- Default Gateway
- OSPFv3 Neighbor 형성
- 같은 Link의 Router 간 통신

#### Unique Local Address

ULA(Unique Local Address)는 사내 Network와 같은 내부 환경에서 사용하는 IPv6 Address이다.

`FC00::/7` 범위이며 일반적으로 `FD00::/8` 범위에서 생성하여 사용한다.
```
FD00:10:10::1/64
```

#### Multicast Address

IPv6 Multicast Address는 하나의 Packet을 여러 Receiver에게 전달할 때 사용한다.

`FF00::/8` 범위를 사용한다.

대표적인 IPv6 Multicast Address는 다음과 같다.
- `FF02::1`: 같은 Link의 모든 IPv6 장비
- `FF02::2`: 같은 Link의 모든 IPv6 Router
- `FF02::5`: 모든 OSPFv3 Router
- `FF02::6`: OSPFv3 DR과 BDR

IPv6에는 Broadcast Address가 없으며 필요한 장비를 선택하여 Multicast로 전달한다.

#### Anycast Address

Anycast는 여러 장비의 Interface에 동일한 IPv6 Address를 설정하고, Routing Table을 기준으로 가장 가까운 장비에 Packet을 전달하는 방식이다.

### IPv6 Header

IPv6 기본 Header는 `40Byte`의 고정된 크기를 사용한다.

```
IPv6 Header | Extension Header | TCP/UDP Header | Data
```

주요 Field는 다음과 같다.
- Traffic Class: Traffic의 우선순위를 표시한다.
- Flow Label: 같은 Flow의 Packet을 구분한다.
- Payload Length: IPv6 Header 뒤에 있는 Data의 크기를 표시한다.
- Next Header: TCP, UDP, ICMPv6 또는 Extension Header를 표시한다.
- Hop Limit: Router를 통과할 수 있는 횟수이며 IPv4의 TTL과 같은 역할을 한다.
- Source Address: 출발지 IPv6 Address이다.
- Destination Address: 목적지 IPv6 Address이다.

IPv6 Router는 전달 중인 Packet을 Fragmentation하지 않는다.

Packet이 다음 Link의 MTU보다 크면 Router는 ICMPv6 Packet Too Big Message를 Source에 전송한다. Source는 해당 정보를 이용하여 Packet 크기를 줄인다.

### ICMPv6

ICMPv6는 IPv4의 ICMP와 비슷한 역할을 하지만, IPv6에서 사용하는 별도의 Protocol이다.

ICMPv6는 NDP, Router Advertisement 및 Path MTU Discovery 등에 사용되므로 Firewall이나 ACL에서 모두 차단하면 IPv6 통신에 문제가 발생할 수 있다.

### NDP

NDP(Neighbor Discovery Protocol)는 같은 Link의 Neighbor, MAC Address, Router 및 Network Prefix를 확인하는 Protocol이다.

IPv4의 ARP, Router Discovery 및 일부 ICMP 기능을 대신하며 ICMPv6 Message를 사용한다.

NDP는 다음 Message를 사용한다.
- RS(Router Solicitation): Host가 Router에 RA를 요청한다.
- RA(Router Advertisement): Router가 Prefix와 Default Gateway 정보를 전달한다.
- NS(Neighbor Solicitation): IPv6 Address에 해당하는 MAC Address를 확인한다.
- NA(Neighbor Advertisement): 자신의 MAC Address 정보를 응답한다.
- Redirect: 더 적절한 Next-Hop을 Host에 알려준다.
```
IPv4: ARP Request → ARP Reply
IPv6: Neighbor Solicitation → Neighbor Advertisement
```

### DAD

DAD(Duplicate Address Detection)는 자신이 사용하려는 IPv6 Address를 다른 장비가 이미 사용하고 있는지 확인하는 기능이다.

Host는 해당 Address를 사용하기 전에 NS Message를 전송한다.

다른 장비에서 NA Message가 오지 않으면 중복되지 않은 Address로 판단하고 사용한다.

### SLAAC

SLAAC(Stateless Address Autoconfiguration)는 Host가 Router의 RA Message를 이용하여 자신의 IPv6 Address를 자동으로 생성하는 방식이다.

1\. Host가 Link-Local Address를 생성한다.

2\. DAD를 통해 중복 여부를 확인한다.

3\. Host가 Router를 찾기 위해 RS Message를 전송한다.

4\. Router는 Network Prefix와 Default Gateway 정보가 포함된 RA Message를 전송한다.

5\. Host는 RA의 Prefix와 자신이 생성한 Interface ID를 결합하여 Global Unicast Address를 생성한다.
```
RA Prefix: 2001:db8:10::/64
Interface ID: ::100
생성된 Address: 2001:db8:10::100/64
```

### DHCPv6

DHCPv6는 IPv6 Host에 Address, DNS Server 및 Domain Name 등의 정보를 제공한다.

DHCPv6는 다음 UDP Port를 사용한다.
- DHCPv6 Client: UDP Port `546`
- DHCPv6 Server: UDP Port `547`

기본 Message 교환 과정은 다음과 같다.
```
Solicit → Advertise → Request → Reply
```
- IPv6 Host의 Default Gateway는 DHCPv6가 아니라 Router의 RA Message를 통해 학습한다.

### Stateless DHCPv6

Stateless DHCPv6는 Host가 SLAAC로 IPv6 Address와 Default Gateway를 설정하고, DHCPv6 Server에서 DNS Server와 Domain Name 등의 추가 정보만 받는 방식이다.

Router는 RA Message의 O Flag를 설정하여 Host가 추가 정보를 DHCPv6 Server에서 받도록 알린다.

Cisco Router에서는 다음 명령어로 O Flag를 설정한다.
```
R1(config-if)# ipv6 nd other-config-flag
```

### Stateful DHCPv6

Stateful DHCPv6는 DHCPv6 Server가 Host의 IPv6 Address와 DNS 등의 정보를 할당하고 Binding 정보를 관리하는 방식이다.

Router는 RA Message의 M Flag를 설정하여 Host가 IPv6 Address를 DHCPv6 Server에서 받도록 알린다.

Cisco Router에서는 다음 명령어로 M Flag를 설정한다.
```
R1(config-if)# ipv6 nd managed-config-flag
```

### RA Flag

RA Message의 M Flag와 O Flag에 따라 Host의 Address 설정 방식이 달라진다.
```
M=0, O=0: SLAAC 사용
M=0, O=1: Stateless DHCPv6 사용
M=1, O=1: Stateful DHCPv6 사용
```

### OSPFv3

OSPFv3는 IPv6 Route를 교환할 수 있는 Link-State Routing Protocol이다.

OSPFv3의 기본 동작은 OSPFv2와 비슷하지만 다음 차이가 있다.
- IPv6 Link-Local Address를 사용하여 Neighbor를 형성한다.
- Interface에서 OSPFv3를 활성화한다.
- Router ID는 IPv4 형식의 `32bit` 값을 사용한다.
- IPv6 Multicast Address `FF02::5`와 `FF02::6`을 사용한다.
- IPv4와 동일하게 IP Protocol Number `89`를 사용한다.

Router ID는 IPv4 Address처럼 보이지만 장비를 구분하기 위한 값이며 실제 IPv4 통신에 사용하는 Address일 필요는 없다.

### IPv6 BGP

BGP에서 IPv6 Route를 교환하려면 MP-BGP(Multiprotocol BGP)의 IPv6 Address Family를 사용한다.

BGP Neighbor를 설정한 후 `address-family ipv6 unicast`에서 해당 Neighbor를 활성화해야 한다.
```
IPv4 Route: address-family ipv4 unicast
IPv6 Route: address-family ipv6 unicast
```

IPv6 BGP도 기존 BGP와 동일하게 TCP Port `179`를 사용한다.

### Dual Stack

Dual Stack은 하나의 Interface와 장비에서 IPv4와 IPv6를 동시에 사용하는 방식이다.

```
IPv4 Address: 192.168.10.1/24
IPv6 Address: 2001:db8:10::1/64
```

IPv4와 IPv6는 각각 별도의 Routing Table과 Protocol Stack을 사용한다.
