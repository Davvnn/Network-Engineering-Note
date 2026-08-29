# VRF / VRF-Lite / Route Leaking

## 개념

### VRF

VRF(Virtual Routing and Forwarding)는 하나의 Router에서 여러 개의 독립적인 Routing Table을 사용하는 기술이다.

각 VRF는 자신의 Routing Table, CEF Table 및 ARP Table을 별도로 관리한다.

따라서 하나의 Router를 여러 개의 논리적인 Router처럼 사용할 수 있으며, 서로 다른 VRF에서는 같은 IP Address를 중복으로 사용할 수도 있다.

### Routing Table

Global Routing Table은 VRF가 적용되지 않은 Interface에서 사용하는 기본 Routing Table이다.
```
R1# show ip route
```

특정 VRF의 Routing Table은 VRF 이름을 지정하여 확인한다.
```
R1# show ip route vrf FINANCE
```
Global Routing Table과 각 VRF의 Routing Table은 서로 독립적이다.

### VRF Interface

VRF는 Layer 3 Interface에 적용한다.

하나의 Layer 3 Interface는 하나의 VRF에만 포함될 수 있다.

Router는 Packet이 들어온 Interface의 VRF를 확인하고 해당 VRF의 Routing Table에서 Destination Route를 검색한다.

### VRF-Lite

VRF-Lite는 MPLS 없이 하나의 Router나 Layer 3 Switch에서 여러 개의 VRF를 사용하는 방식이다.

일반적으로 회사 내부에서 부서, 고객, 관리 Network 및 Guest Network의 Routing Table을 분리할 때 사용한다.
- VRF-Lite: MPLS 없이 장비 내부에서 Routing Table을 분리한다.
- MPLS L3VPN: MPLS Label과 MP-BGP를 사용하여 서로 떨어져 있는 지점의 같은 VRF를 통신사망을 통해 하나처럼 연결하는 방식이다.

둘 다 VRF를 분리해서 사용한다는 점은 비슷하지만, VRF-Lite는 MPLS 없이 VRF를 연결하고 MPLS L3VPN은 통신사 MPLS망을 통해 여러 지점의 VRF를 연결한다.

### VRF 생성 방식

Cisco IOS에서는 VRF를 생성하는 방식에 따라 Interface에 적용하는 명령어가 다르다.

기존 IPv4 전용 방식은 `ip vrf`로 VRF를 생성하고 `ip vrf forwarding`으로 Interface에 적용한다.
```
R1(config)# ip vrf FINANCE
R1(config)# interface gi0/0
R1(config-if)# ip vrf forwarding FINANCE
```

Address-Family 방식은 `vrf definition`으로 VRF를 생성하고 `vrf forwarding`으로 Interface에 적용한다.
```
R1(config)# vrf definition FINANCE
R1(config-vrf)# address-family ipv4
R1(config-vrf-af)# exit-address-family

R1(config)# interface gi0/0
R1(config-if)# vrf forwarding FINANCE
```
- `ip vrf forwarding`: `ip vrf`로 생성한 기존 IPv4 전용 VRF에 사용한다.
- `vrf forwarding`: `vrf definition`으로 생성한 VRF에 사용하며 IPv4와 IPv6 Address-Family를 지원한다.

두 방식 모두 Interface에 VRF를 적용하면 기존 IP Address가 제거되므로 VRF를 적용한 후 IP Address를 다시 설정해야 한다.

### VRF별 Routing

각 VRF는 자신의 Static Route와 Routing Protocol을 가질 수 있다.
```
R1(config)# ip route vrf FINANCE 0.0.0.0 0.0.0.0 10.0.10.2
```
위 명령어는 `FINANCE` VRF의 Routing Table에만 Default Route를 등록한다.

OSPF도 VRF별로 별도의 Process를 구성할 수 있다.
```
R1(config)# router ospf 10 vrf FINANCE
```

VRF는 Global Routing Table과 분리된 별도의 Routing Table에서 Route 정보를 관리한다.  

### Route Leaking

Route Leaking은 서로 분리된 VRF 사이에서 필요한 Route만 공유하여 통신할 수 있도록 하는 기능이다.

기본적으로 VRF끼리는 서로의 Routing Table을 확인할 수 없기 때문에 직접 통신할 수 없다.

Route Leaking으로 통신하려면 요청 방향뿐만 아니라 응답 방향의 Route도 필요하다.

### Route Distinguisher

RD(Route Distinguisher)는 서로 다른 VRF에서 사용하는 동일한 IP Prefix를 MP-BGP에서 서로 다른 VPN Route로 구분하기 위한 값이다.
- MP-BGP는 서로 다른 VRF 사이에서 VPN Route 정보를 전달하는 기능이다.
- VPN Route는 특정 VRF에 속한 Route를 의미한다.  
- MP-BGP라는 별도의 명령어가 있는 것은 아니다. `router bgp`에서 VRF별 Address Family를 설정하면 MP-BGP로 동작한다.  

Route를 MP-BGP로 전달하면 여러 VRF의 Route가 VPN Routing Table에서 관리한다. 이때 VRF 이름은 MP-BGP로 전달되지 않으므로 같은 Prefix를 구분할 수 없다.  

예를 들어 두 VRF에서 모두 `192.168.10.0/24`를 사용하더라도 RD를 추가하면 서로 다른 VPN Route로 구분할 수 있다.
- `65000:10:192.168.10.0/24`
- `65000:20:192.168.10.0/24`

```
R1(config)# vrf definition FINANCE
R1(config-vrf)# rd 65000:10
```
단순히 VRF-Lite로 Routing Table만 분리할 때는 RD가 실제 Packet 전달에 사용되지 않지만, MP-BGP로 VRF Route를 교환하거나 Route Leaking을 구성할 때 사용한다.

### Route Target

RT(Route Target)는 MP-BGP에서 어떤 VRF Route를 내보내고 가져올지 결정하는 BGP Extended Community 값이다.
- `route-target export`: VRF Route에 RT 값을 설정하여 내보낸다.
- `route-target import`: 지정한 RT 값이 설정된 Route를 가져온다.

```
FINANCE
route-target export 65000:10
route-target import 65000:100

SHARED
route-target export 65000:100
route-target import 65000:10
```

위 설정은 자신의 Route를 RT `65000:10`으로 내보내고, RT `65000:100`이 설정된 Route를 가져온다.
각 VRF는 자신의 `Export` 값을 상대방 VRF의 `Import` 값과 맞춰야 한다.

### RD와 RT의 차이

RD는 동일한 IP Prefix를 서로 다른 VPN Route로 구분하고, RT는 어떤 VRF가 해당 Route를 가져올지 결정한다.

- RD: Route를 구분한다.
- RT: Route를 Import하거나 Export한다.

### MP-BGP를 이용한 VRF Route Leaking

#### RD와 RT 설정

각 VRF에 RD를 설정하고, Route를 공유할 수 있도록 자신의 `Export` 값을 상대방 VRF의 `Import` 값과 맞춘다.
```
R1(config)# vrf definition FINANCE
R1(config-vrf)# rd 65000:10
R1(config-vrf)# address-family ipv4
R1(config-vrf-af)# route-target export 65000:10
R1(config-vrf-af)# route-target import 65000:100
R1(config-vrf-af)# exit-address-family

R1(config)# vrf definition SHARED
R1(config-vrf)# rd 65000:100
R1(config-vrf)# address-family ipv4
R1(config-vrf-af)# route-target export 65000:100
R1(config-vrf-af)# route-target import 65000:10
R1(config-vrf-af)# exit-address-family
```
- `FINANCE`가 Export한 RT `65000:10`을 `SHARED`가 Import한다.
- `SHARED`가 Export한 RT `65000:100`을 `FINANCE`가 Import한다.

#### VRF Route를 MP-BGP에 등록

각 VRF의 Connected Route를 MP-BGP에 등록한다.
```
R1(config)# router bgp 65000

R1(config-router)# address-family ipv4 vrf FINANCE
R1(config-router-af)# redistribute connected
R1(config-router-af)# exit-address-family

R1(config-router)# address-family ipv4 vrf SHARED
R1(config-router-af)# redistribute connected
R1(config-router-af)# exit-address-family
```
- `address-family ipv4 vrf`: 각 VRF의 BGP Address Family로 들어간다.
- `redistribute connected`: VRF의 Connected Route를 MP-BGP에 등록한다.

---
