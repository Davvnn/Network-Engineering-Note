# BGP 기초

## 개념

### BGP

BGP(Border Gateway Protocol)는 서로 다른 Autonomous System 사이에서 Network 경로를 교환하기 위해 사용하는 Path Vector Routing Protocol이다.

OSPF와 EIGRP는 하나의 AS 내부에서 최적 경로를 계산하는 IGP이고, BGP는 AS 사이에서 경로를 교환하고 정책을 기준으로 Best Path를 선택하는 EGP이다.

BGP는 인터넷 규모의 많은 Route를 처리해야 하므로 빠른 수렴보다 안정성, 확장성 및 정책 제어에 중점을 둔다.

BGP는 다음과 같은 특징이 있다.
- BGP Neighbor를 자동으로 발견하지 않고 관리자가 직접 설정한다.
- TCP `179` Port를 사용하여 신뢰성 있게 Route를 교환한다.
- 여러 Path Attribute를 순서대로 비교하여 Best Path를 선택한다.
- 처음에는 전체 Route를 교환하고, 이후에는 변경된 Route만 `Partial Update`한다. 
- OSPF와 EIGRP의 `network` 명령은 Interface에서 Protocol을 활성화하고, BGP의 `network` 명령은 Routing Table의 특정 Route를 BGP에 등록한다.  

### Autonomous System

Autonomous System(AS)은 회사나 ISP가 독립적인 Routing 정책으로 관리하는 Network와 Router의 집합이다.  
- 하나의 ISP가 여러 지역의 Router와 IP Network를 하나의 Routing 정책으로 관리하면 전체가 하나의 AS가 된다.  
- 회사가 ISP로부터 할당받은 Public IP와 Default Route만 사용하고 ISP가 해당 Route를 대신 광고하면, 회사는 독립적인 AS로 보이지 않는다. 인터넷에서는 ISP의 AS를 통해 회사 Network에 도달하며, 회사는 ISP에 연결된 고객 Network로 인식된다.
- 회사가 인터넷 주소 관리기관으로부터 Public ASN을 할당받고 자신의 Public IP 대역을 BGP로 직접 광고하면, 인터넷에서는 해당 회사를 독립적인 AS로 인식한다.

AS는 자신을 구분하기 위한 고유한 ASN(Autonomous System Number)을 사용하며, BGP는 이 번호를 통해 Route가 어떤 AS를 거쳐왔는지 확인한다.

예를 들어 AS `65001`과 AS `65002`가 BGP Neighbor를 형성하면 서로 다른 AS 사이의 eBGP Session으로 동작한다.

### eBGP

eBGP(External BGP)는 서로 다른 AS에 속한 Router 사이에서 BGP Route를 교환한다.

eBGP는 일반적으로 직접 연결된 Router끼리 Neighbor를 형성하며, 기본 TTL은 `1`이다.

eBGP로 학습한 Route의 Cisco 기본 Administrative Distance는 `20`이다.

Route를 광고할 때 일반적으로 Next-Hop을 자신의 IP Address로 변경한다.  

### iBGP 
 
iBGP(Internal BGP)는 같은 AS에 속한 Router 사이에서 BGP Route를 교환한다. 
 
iBGP는 외부 AS에서 학습한 BGP Route를 같은 AS 내부의 다른 BGP Router에게 전달하기 위해 사용한다. OSPF나 EIGRP와 같은 IGP를 대체하는 Protocol은 아니다. 
 
기본 TTL은 `255`이다. 
 
iBGP로 학습한 Route의 Cisco 기본 Administrative Distance는 `200`이다. 
 
### Next-Hop 
 
eBGP는 Route를 다른 AS로 광고할 때 Next-Hop을 자신으로 변경한다.

 iBGP는 외부 AS에서 eBGP로 학습한 Route를 같은 AS 내부의 iBGP Neighbor에게 광고할 때, 기존 Next-Hop을 그대로 유지한다.    
- `next-hop-self`를 사용하여 자신을 Next-Hop으로 변경할 수 있다.    
  
### iBGP Route 재광고 규칙  
  
iBGP Neighbor에게 학습한 Route를 다른 iBGP Neighbor에게 광고하지 않는다.    
- 같은 AS 내부에서는 Route를 전달해도 AS_PATH에 ASN이 추가되지 않아 Loop를 확인하기 어렵기 때문에, iBGP로 학습한 Route를 다른 iBGP Neighbor에게 다시 광고하지 않는다.  
  
eBGP로 학습한 Route는 직접 연결된 iBGP Neighbor에게 광고할 수 있지만, 해당 Route를 iBGP로 학습한 Router는 다른 iBGP Neighbor에게 다시 광고하지 않는다.  
```  
R1 --eBGP-- R2 --iBGP-- R3 --iBGP-- R4 --iBGP-- R5  
  
R1 → R2 : eBGP로 Route 전달  
R2 → R3 : iBGP로 Route 전달 가능  
R3 → R4 : 전달 불가  
R4 → R5 : 전달 불가  
``` 

따라서 기본 iBGP 환경에서는 모든 iBGP Router가 서로 직접 Neighbor를 형성하는 Full Mesh 구성이 필요하다. 
 
![](images/15-bgp-fullmesh.png)

iBGP Router가 `n`대일 때 필요한 Session 수는 다음과 같다. 
 
`n × (n - 1) / 2` 
 
예를 들어 iBGP Router가 4대라면 총 6개의 iBGP Session이 필요하다. 
 
대규모 Network에서는 iBGP Session 수를 줄이기 위해 Route Reflector를 사용할 수 있다. 

### BGP Neighbor

BGP는 OSPF나 EIGRP처럼 Multicast Hello를 사용하여 Neighbor를 자동으로 찾지 않는다.

양쪽 Router에서 상대방의 IP Address와 ASN을 직접 설정해야 하며, 상대방 IP까지 IP Reachability가 확보되어 있어야 한다.

설정한 Remote AS가 자신의 AS와 다르면 eBGP, 같으면 iBGP로 동작한다.

### BGP Timer

Cisco BGP의 기본 Keepalive Time은 `60초`이고 Hold Time은 `180초`이다.

Keepalive 메시지를 주기적으로 교환하여 Session을 유지하며, Hold Time 동안 BGP 메시지를 수신하지 못하면 Neighbor Session을 종료한다.

### BGP 메시지

BGP는 Neighbor Session을 형성하고 Route 정보를 교환하기 위해 다음 메시지를 사용한다.
- Open: BGP Version, ASN, Hold Time, Router ID 등의 정보를 교환하여 Session을 시작한다.
- Keepalive: Neighbor Session이 정상적으로 유지되고 있음을 알린다.
- Update: 새로 추가되거나 사라진 Route와 경로 정보를 전달한다.
- Notification: 오류 내용을 알리고 BGP Session을 종료한다.

### BGP Neighbor State

BGP Neighbor는 다음 과정을 거쳐 `Established` 상태까지 진행된다.

1\. Idle: BGP Neighbor 연결을 시작하기 전의 상태이다.  

2\. Connect: Neighbor와 `TCP 3-Way Handshake`가 완료되기를 기다리는 상태이다.  

3\. Active: TCP 연결에 실패하여 다시 연결을 시도하는 상태이다.

4\. OpenSent: TCP 연결 후 Open 메시지를 전송하고 상대방의 Open 메시지를 기다리는 상태이다.

5\. OpenConfirm: 상대방의 Open 메시지를 정상적으로 확인한 후 Keepalive 메시지를 전송하고, 상대방의 Keepalive 또는 Notification 메시지를 기다리는 상태이다.

6\. Established: BGP Neighbor가 정상적으로 형성되어 Update와 Keepalive 메시지를 교환하는 상태이다.

### BGP Router ID

BGP Router ID는 BGP Router를 식별하는 `32-bit` 값이다.

Router ID를 직접 설정하지 않으면 일반적으로 가장 높은 Loopback IP Address를 사용하고, Loopback Interface가 없으면 가장 높은 Active Physical Interface IP Address를 사용한다.

Router ID를 자동으로 선택하면 사용 중인 Interface가 Down되거나 BGP가 재시작될 때 Router ID도 달라질 수 있다. 따라서 운영 환경에서는 Router ID가 항상 동일하게 유지되도록 직접 설정하는 것이 좋다.  
- Router ID가 Interface 상태나 IP 변경의 영향을 받지 않도록 직접 설정하는 것이 좋다.

### BGP Route 광고

BGP는 Connected Route나 IGP Route를 자동으로 광고하지 않는다.

로컬 Route를 BGP로 광고하려면 `network` 또는 `redistribute` 명령을 사용해야 한다.

`network` 명령을 사용하려면 광고하려는 Network와 Subnet Mask가 Local Routing Table의 Route와 정확히 일치해야 한다.

예를 들어 `203.0.113.0/24`를 광고하려면 Local Routing Table에도 동일한 `203.0.113.0/24` Route가 존재해야 한다.
```
R1(config)# router bgp 65001
R1(config-router)# network 203.0.113.0 mask 255.255.255.0
```

`redistribute` 명령은 Local Routing Table에 등록된 특정 종류의 Route 전체를 BGP로 가져와 광고할 때 사용한다.
 
Local Routing Table에 다음과 같은 Static Route가 있다. 
 ``` 
S  203.0.113.0/24 
S  198.51.100.0/24 
``` 
 
다음과 같이 설정하면 Local Routing Table에 등록된 모든 Static Route를 BGP로 가져와 광고한다. 
 ``` 
R1(config)# router bgp 65001 
R1(config-router)# redistribute static 
``` 
 
OSPF Route를 BGP로 가져오려면 다음과 같이 설정한다. 
``` 
R1(config-router)# redistribute ospf 1 
``` 
 
`network`: 광고할 Route를 하나씩 직접 지정한다. 

`redistribute`: Static, Connected, OSPF, EIGRP와 같은 특정 종류의 Route를 BGP로 가져와 광고한다. 

`redistribute`는 여러 Route가 함께 광고될 수 있으므로 `Route-Map`이나 `Prefix-List`를 사용하여 필요한 Route만 제한하는 것이 좋다.

