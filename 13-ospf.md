# OSPF

## 개념

OSPF(Open Shortest Path First)는 Link-State 기반의 동적 라우팅 프로토콜이다. 각 Router는 LSA(Link-State Advertisement)를 교환하여 LSDB(Link-State Database)를 구성하고, Dijkstra의 SPF(Shortest Path First) 알고리즘으로 목적지까지의 최적 경로를 계산한다.

OSPF의 특징은 다음과 같다.
- IP Protocol Number `89`를 사용하며 TCP나 UDP를 사용하지 않는다.
- 같은 Area의 Router들은 동일한 LSDB를 유지한다.
- Cost를 Metric으로 사용한다.
- AD 값은 `110`이다.
- `224.0.0.5`와 `224.0.0.6` Multicast Address를 사용한다.
- Cisco 장비에서는 OSPF Packet을 일반적으로 DSCP `CS6`로 표시한다.

`224.0.0.5`는 모든 OSPF Router가 수신하고, `224.0.0.6`은 DR과 BDR이 수신한다.

OSPF Process ID는 Router 내부에서만 의미가 있는 값이다. 따라서 서로 연결된 Router의 Process ID가 달라도 OSPF Neighbor를 형성할 수 있다. 하나의 Router에서 여러 OSPF Process를 사용하면 각 Process는 별도의 LSDB를 유지하며, Process 간에 Route를 공유하려면 Redistribution이 필요하다.

### Router ID

Router ID는 OSPF Router를 식별하는 32 Bit 값이다. IPv4 Address 형식으로 표시하지만 실제 Interface에 할당된 IP Address일 필요는 없다.

Router ID는 OSPF Process가 시작될 때 다음 순서로 결정된다.

1\. `router-id` 명령어로 직접 설정한 값

2\. Up 상태인 Loopback Interface 중 가장 높은 IP Address

3\. Up 상태인 Physical Interface 중 가장 높은 IP Address

OSPF Domain 안에서 Router ID는 중복되지 않아야 한다. Router ID가 중복되면 LSA가 충돌하여 Neighbor 및 Route 학습 문제가 발생할 수 있다. 

운영 중 Router ID를 변경하면 OSPF Process가 재시작되면서 Neighbor 관계가 일시적으로 끊기고 LSDB가 다시 구성된다. IOS Version에 따라 Router ID 변경 후 `clear ip ospf process` 명령어를 추가로 실행해야 하거나 별도의 명령어 없이 Router-ID가 변경되어 Neighbor이 끊킬 수 있으므로 작업 전에 확인해야 한다.

### OSPF Cost

OSPF는 목적지까지 누적된 Cost중에서 가장 낮은 경로를 선택한다.
```
Cost = Reference Bandwidth / Interface Bandwidth
```

Cisco 장비의 기본 Reference Bandwidth는 `100 Mbps`이다.
- FastEthernet 100 Mbps: `100 / 100 = Cost 1`
- GigabitEthernet 1 Gbps: `100 / 1000 = 0.1 → Cost 1`

OSPF Cost는 최소값이 `1`이므로 기본 설정에서는 FastEthernet과 GigabitEthernet이 모두 Cost `1`로 계산될 수 있다. 따라서 현재 회사 Network에서 사용하는 가장 빠른 Link를 기준으로 Reference Bandwidth를 변경해야 Link 속도에 따른 Cost를 제대로 구분할 수 있다.

Reference Bandwidth는 OSPF Domain의 모든 Router에서 동일하게 설정해야 한다. 또한 Interface Cost는 `ip ospf cost` 명령어를 사용하여 Interface 단위로 직접 설정할 수 있다.

### OSPF Area

OSPF는 규모가 큰 Network를 여러 Area로 나누어 LSA의 Flooding 범위와 SPF 계산 범위를 줄인다. 각 Area는 별도의 LSDB를 유지하며, 다른 Area의 구성은 알지 못한다.

Area 0은 OSPF를 Backbone Area라고 부른다. 모든 Non-Backbone Area는 Area 0과 연결되어야 하며, 다른 Area 간 Traffic은 Area 0을 통해 전달된다.

### OSPF Router 역할

Backbone Router: Area 0에 속한 Router이다.

ABR(Area Border Router): Area 0과 Non-Backbone Area를 연결하는 Router이다.

ASBR(Autonomous System Boundary Router): Static Route 및 다른 Routing Protocol의 Route를 OSPF로 
Redistribution하는 Router이다.

### OSPF Packet

OSPF는 다섯 가지 Packet을 사용한다.

1\. Hello: Neighbor를 탐색하고 관계를 유지한다.

2\. DBD(Database Description): LSDB에 있는 LSA Header 목록을 요약하여 전달한다.
- Point-to-Point 환경에서는 연결된 두 Router가 서로 DBD Packet을 교환한다.  
- Broadcast 환경에서는 DROther가 DR 및 BDR과 DBD Packet을 교환하지만, DROther끼리는 교환하지 않는다. 

3\. LSR(Link-State Request): 없거나 오래된 LSA의 내용을 요청한다.

4\. LSU(Link-State Update): 하나 이상의 전체 LSA를 전달한다.

5\. LSAck(Link-State Acknowledgment): 수신한 LSA에 대한 확인 응답을 전송한다.

Hello Packet에는 Network Mask, Hello/Dead Interval, Area ID, Area Type, Router Priority, DR/BDR 정보가 포함된다. Neighbor를 형성하려면 양쪽의 Hello Parameter가 서로 일치해야 한다.

DBD Packet은 LSA Type, Link State ID, Advertising Router 및 Sequence Number 등이 포함된 LSA Header만 전달한다. 상대 Router는 DBD와 자신의 LSDB를 비교하고 필요한 LSA를 LSR로 요청한다.

### OSPF Neighbor State

![](images/13-ospf-neighbor-state.png)

1\. Down: Neighbor로부터 Hello Packet을 수신하지 못한 상태이다.

2\. Attempt: NBMA 환경에서 수동으로 지정한 Neighbor에게 Hello Packet을 보내고 응답을 기다리는 상태이다.
- 일반적인 Ethernet Broadcast 및 Point-to-Point 환경에서는 Attempt 상태가 나타나지 않는다.
- NBMA는 여러 Router가 같은 IP 대역을 공유하지만 Broadcast와 Multicast가 자동으로 전달되지 않는 환경이다. 일반적인 OSPF 환경에서는 Hello Packet을 `224.0.0.5`로 전송하지만, NBMA 환경에서는 수동으로 지정한 Neighbor에게 Unicast로 전송한다.

3\. Init: 상대방의 Hello Packet은 받았지만, 양방향 Hello 통신은 아직 확인되지 않은 상태이다.

4\. 2-Way: 양쪽 Router가 서로의 Hello Packet에서 자신의 Router ID를 확인한 상태이다.

5\. ExStart: Master와 Slave를 결정하고 DBD Sequence Number를 협상하는 상태이다.
- DBD Sequence Number는 DBD Packet을 구분하기 위한 순서 번호이다. DBD Packet을 여러 번 나누어 보내면 Master가 `100`, `101`, `102`처럼 번호를 증가시키고 Slave는 같은 번호로 응답한다. 이를 통해 어떤 DBD Packet을 주고받았는지 확인하고 누락이나 중복을 방지한다.

6\. Exchange: DBD Packet으로 서로의 LSA Header 목록을 교환하는 상태이다.

7\. Loading: 필요한 LSA를 LSR로 요청하고 LSU로 수신하는 상태이다.

8\. Full: LSDB 동기화가 완료된 상태이다.

Point-to-Point Link에서는 연결된 두 Router가 Full 상태까지 Neighbor 관계를 형성한다. 하지만 Broadcast Network에서는 DROther가 DR 및 BDR과만 Full Adjacency를 형성하고, 다른 DROther와는 2-Way 상태를 유지한다.
- 2-Way는 두 Router가 Hello Packet을 주고받아 서로를 Neighbor로 확인한 상태이다.
- DROther끼리는 Neighbor 관계를 형성하지만 DBD를 교환하거나 LSDB를 직접 동기화하지 않는다.
- 대신 DR을 통해 LSA를 전달받아 같은 Area의 Router들과 동일한 LSDB를 유지한다.

### LSA Type

Type 1 Router LSA: 모든 OSPF Router가 생성하며 자신이 연결된 Link, Neighbor, Network와 Cost를 같은 Area에 광고한다.
- 각 Router가 자신과 연결된 Neighbor 및 Network와 해당 경로의 Cost를 같은 Area에 광고한다.

Type 2 Network LSA: DR이 생성하며 같은 Network에 연결된 OSPF Router 목록과 Subnet Mask를 같은 Area에 광고한다.
- DR이 자신이 속한 OSPF Network의 대역과 자기와 연결된 OSPF Router 목록을 같은 Area에 광고한다.  

Type 3 Summary LSA: ABR이 생성하며 다른 Area의 Network와 해당 Network까지의 Cost를 자신의 Area에 광고한다.
- ABR이 자신을 통해 다른 Area의 OSPF Network로 갈 수 있다는 정보를 자신의 Area에 광고한다. 

Type 4 ASBR Summary LSA: ABR이 생성하며 다른 Area에 있는 ASBR의 Router ID와 해당 ASBR까지의 Cost를 자신의 Area에 광고한다.
- ABR이 자신을 통해 다른 Area에 있는 ASBR로 갈 수 있다는 정보를 자신의 Area에 광고한다.
- 만약 ASBR이 같은 Area에 있으면 Type 1 LSA를 통해 경로를 확인할 수 있으므로 Type 4 LSA는 필요하지 않다.

Type 5 AS External LSA: ASBR이 생성하며 Redistribution된 외부 Network와 External Metric을 OSPF Domain에 광고한다.
- ASBR이 OSPF 외부 Network와 External Metric을 OSPF Domain에 광고한다.
- Static, EIGRP, BGP 등의 Route를 OSPF로 Redistribution할 때 생성된다.

Type 6 Group Membership LSA: OSPF 환경에서는 거의 사용되지 않는다.

Type 7 NSSA External LSA: NSSA 내부 ASBR이 생성하며 NSSA로 Redistribution된 외부 Route를 광고한다.
- NSSA 내부 ASBR이 외부 Network를 NSSA에 광고하며 ABR에서 Type 5 LSA로 변환되어 다른 Area에 전달될 수 있다.


ASBR이 외부 Route를 Redistribution하면 Type 5 LSA를 생성한다. ASBR과 다른 Area에 있는 Router가 해당 ASBR까지 도달할 수 있도록 ABR은 Type 4 LSA를 생성한다. ASBR과 같은 Area에 있는 Router는 Type 1 LSA로 ASBR의 위치를 알 수 있으므로 Type 4 LSA가 필요하지 않다.

NSSA 내부에서 Redistribution된 Route는 Type 7 LSA로 생성된다. Type 7 LSA는 NSSA 내부에서만 Flooding되며 ABR은 Type 7 LSA를 Type 5 LSA로 변환하여 다른 Area에 광고한다.

ABR은 연결된 Area마다 별도의 LSDB를 유지하고 하나의 Routing Table을 사용한다. 한 Area의 Type 1과 Type 2 LSA를 다른 Area로 그대로 전달하지 않고, 해당 LSA로 계산한 Network 정보를 Type 3 LSA로 요약하여 광고한다.

### DR and BDR

Broadcast와 NBMA Network에서는 모든 Router가 DR과 BDR을 선출한다.
- 모든 Router가 서로 Full Adjacency를 형성하면 LSA 교환량이 증가하여 과부하가 발생할 수 있기 때문이다.

DR과 BDR은 다음 순서로 선출된다.

1\. OSPF Interface Priority가 가장 높은 Router

2\. Priority가 같으면 Router ID가 가장 높은 Router

Priority의 기본값은 `1`이며, Priority를 `0`으로 설정한 Interface는 DR이나 BDR로 선출될 수 없다.

DR/BDR 선출은 `Non-Preemptive` 방식이다. DR이 선출된 이후 더 높은 Priority나 Router ID를 가진 Router가 연결되어도 기존 DR을 교체하지 않는다. DR에 장애가 발생하면 BDR이 DR로 승격되고 새로운 BDR을 선출한다.

### Stub Area

Stub Area는 외부 Route를 나타내는 Type 5 LSA와 외부 ASBR의 위치를 나타내는 Type 4 LSA를 차단하여 LSDB와 Routing Table을 단순화하는 Area이다.
- Type 1과 Type 2 LSA를 사용하여 Area 내부 Route를 학습한다.
- Type 3 LSA를 사용하여 다른 OSPF Area의 Network를 학습한다.
- Type 4와 Type 5 LSA는 차단한다.
- ABR은 Stub Area 내부에 `0.0.0.0/0` Type 3 Default Route를 자동으로 광고한다.
- Stub Area의 모든 내부 Router는 ABR이 광고한 `0.0.0.0/0` Default Route를 routing table에 추가하고, 목적지 Route가 없으면 자신의 Default Route를 사용하여 ABR로 패킷을 전달한다.  
- ABR은 자신이 알고 있는 외부 Route를 사용하여 패킷을 ASBR 방향으로 전달한다.

### Totally Stubby Area

Totally Stubby Area는 Stub Area의 기능에 더해 다른 Area의 Network를 나타내는 Type 3 LSA까지 차단하는 Area이다.
- Type 1과 Type 2 LSA를 사용하여 Area 내부 Route를 학습한다.
- Type 3, Type 4, Type 5 LSA를 차단한다.
- ABR은 Type 3 Default Route를 광고한다.
- 다른 OSPF Area와 External Network로 가기 위해서는 ABR이 광고한 Type 3 Default Route를 통해 나갈 수 있다.
- 다른 Area의 네트워크를 학습하지 않으므로 Stub Area보다 LSDB와 Routing Table을 더 작게 유지할 수 있다.

Stub Area에서 Totally Stubby Area가 되기 위해서는 `no-summary`를 ABR에 설정해야 한다. 이후 ABR은 다른 Area에서 받은 Type 3 LSA를 내부 Router에 광고하지 않고, 대신 `0.0.0.0/0` Type 3 Default Route를 광고한다.
- `no-summary`는 해당 Area의 Route가 밖으로 나가는 것을 차단하는 것이 아니라, 다른 Area의 일반 Type 3 LSA가 해당 Area로 들어오는 것만 차단한다.

### NSSA and LSA Type 7

NSSA(Not-So-Stubby Area)는 Stub Area처럼 다른 Area에서 들어오는 Type 4와 Type 5 LSA를 차단하지만, NSSA 내부에서 외부 Route를 Redistribution할 수 있는 Area이다.
- Type 1과 Type 2 LSA를 사용하여 NSSA 내부 Route를 학습한다.
- Type 3 LSA를 사용하여 다른 OSPF Area의 Network를 학습한다.
- 다른 Area에서 들어오는 Type 4와 Type 5 LSA는 차단한다.
- NSSA 내부 ASBR의 Redistribution은 허용한다.
- NSSA 내부에서 Redistribution된 외부 Route는 Type 7 LSA로 생성된다.
- Type 7 LSA는 NSSA 내부에서만 Flooding된다.

일반 Stub Area에서는 Type 5 LSA를 사용할 수 없고 Area 내부에서도 외부 Route를 Redistribution할 수 없다. 하지만 NSSA는 Type 5 LSA 대신 Type 7 LSA를 사용하여 이를 해결한다.

### Type 7에서 Type 5로의 변환

NSSA 내부 ASBR이 외부 Route를 Redistribution하면 Type 7 LSA가 생성된다. NSSA의 ABR은 다른 Area에서도 해당 외부 Route를 사용할 수 있도록 Type 7 LSA를 Type 5 LSA로 변환한다.
- NSSA 내부 ASBR이 외부 Route를 OSPF로 Redistribution한다.
- ASBR이 해당 외부 Route를 Type 7 LSA로 생성한다.
- Type 7 LSA가 NSSA 내부에 Flooding된다.
- NSSA의 ABR이 변환 대상 Type 7 LSA를 수신한다.
- ABR이 Type 7 LSA를 Type 5 LSA로 변환한다.
- 변환된 Type 5 LSA가 Area 0과 Type 5 LSA를 허용하는 다른 Normal Area로 전달된다.
- Area 0의 Router는 Type 1 LSA를 통해 변환 ABR의 위치를 알 수 있으므로 Type 4 LSA가 필요하지 않다.
- 다른 Normal Area의 ABR은 Area 0에서 Type 5 LSA를 통해 External Network와 변환 ABR의 Router ID를 확인하고, Type 1 LSA를 통해 변환 ABR까지의 경로를 확인한다. 이후 변환 ABR까지의 경로를 Type 4 LSA로 만들어 자신의 Area에 광고한다.

하나의 NSSA에 ASBR과 ABR이 동시에 존재할 수 있다. ASBR은 외부 Route를 NSSA로 Redistribution하여 Type 7 LSA를 생성하고, ABR은 Type 7 LSA를 Type 5 LSA로 변환하여 다른 Area에 광고한다.

### NSSA의 Default Route 동작

일반 NSSA는 Stub Area와 달리 ABR이 Default Route를 자동으로 광고하지 않는다.

- 일반 NSSA는 Type 3 LSA를 통해 다른 OSPF Area의 Network를 학습한다.
- 다른 Area에서 생성된 Type 5 LSA는 NSSA로 들어오지 않는다.
- Default Route가 필요하면 별도의 Default Route 광고 설정이 필요하다.
- Cisco IOS에서 일반 NSSA의 Default Route는 Type 7 LSA로 광고되며 Routing Table에는 일반적으로 `O*N2`로 표시된다.

NSSA에 Default Route가 없더라도 Type 3 LSA로 학습한 다른 OSPF Area의 Network에는 통신할 수 있다. 그러나 Type 5 LSA로만 알려진 외부 목적지는 별도의 Default Route나 구체적인 Route가 없으면 통신할 수 없다.

### NSSA의 Default Route 동작

일반 NSSA는 Stub Area와 달리 ABR이 Default Route를 자동으로 광고하지 않는다.
- 일반 NSSA는 Type 3 LSA를 통해 다른 OSPF Area의 Network를 학습한다.
- 다른 Area에서 생성된 Type 5 LSA는 NSSA로 들어오지 않는다.
- Default Route가 필요하면 별도의 Default Route 광고 설정이 필요하다.
- Cisco IOS에서 일반 NSSA의 Default Route는 Type 7 LSA로 광고되며 Routing Table에는 일반적으로 `O*N2`로 표시된다.

NSSA에 Default Route를 광고하면 NSSA 내부 Router는 0.0.0.0/0 Type 7 Default Route를 학습한다. 이후 Routing Table에 목적지 Route가 없는 패킷은 Default Route를 통해 ABR로 전달한다. NSSA 내부 Router는 다른 Area의 External Route를 구체적으로 알지 못하지만, ABR은 Area 0을 통해 해당 Route를 알고 있으므로 패킷을 목적지로 전달할 수 있다.   

```
router ospf 1
 area 2 nssa default-information-originate
```
Area 2를 NSSA로 설정하고, `0.0.0.0/0` Default Route를 Area 2 내부에 Type 7 LSA로 광고한다.

### Totally NSSA

Totally NSSA는 Totally Stubby Area의 기능과 NSSA의 Redistribution 기능을 함께 사용하는 Area이다.
- Type 1과 Type 2 LSA를 사용하여 Area 내부 Route를 학습한다.
- Type 3, Type 4, Type 5 LSA를 차단한다.
- ABR은 Type 3 Default Route를 광고한다.
- NSSA 내부 ASBR이 생성한 Type 7 LSA는 허용한다.
- Type 7 LSA는 ABR에서 Type 5 LSA로 변환되어 다른 Area에 전달될 수 있다.
- 해당 Area 외부 OSPF Network는 Type 3 Default Route를 통해 ABR 방향으로 전달한다.

NSSA에서 Totally NSSA가 되기 위해서는 `no-summary`를 ABR에 설정해야 한다. 이후 ABR은 다른 Area에서 받은 Type 3 LSA를 내부 Router에 광고하지 않고, 대신 `0.0.0.0/0` Type 3 Default Route를 광고한다.

### OSPF Route Summarization

OSPF Route Summarization은 여러 상세 Route를 하나의 Summary Route로 묶어 다른 Area에 광고하는 기술이다.

- Routing Table과 LSDB의 Route 및 LSA 수가 줄어든다.
- Route가 변경이 되도 다른 Area까지 전파되는 범위를 줄어든다.
- 같은 Area 내부에서는 Type 1과 Type 2 LSA를 통해 Route를 그대로 공유하고, 다른 Area로 광고할 때만 ABR이 Route를 요약한다.
- Summarization은 ABR 또는 ASBR에서 수행한다.
- ABR은 한 Area의 여러 내부 Route를 요약하여 다른 Area에 Type 3 LSA로 광고한다.
```
area 1 range 10.1.0.0 255.255.252.0
```

- ASBR은 OSPF로 Redistribution되는 External Route를 요약하여 Normal Area에서는 Type 5 LSA로, NSSA에서는 Type 7 LSA로 광고한다.
- ASBR에서는 `summary-address` 명령어를 사용한다.
```
summary-address 10.11.11.0 255.255.255.128
```

Summary Route를 생성하면 ABR 또는 ASBR의 Routing Table에 Summary 범위의 `Null0` Discard Route가 생성된다. 목적지에 대한 실제 Route가 Routing Table에 있으면 해당 Route로 전달하지만, 목적지가 Summary 범위에만 포함되고 실제 Route가 없으면 Traffic은 `Null0`로 전달되어 폐기된다. 
```
R1# show ip route ospf

O     10.1.0.0/22 is a summary, 00:05:21, Null0
O     10.1.0.0/24 [110/20] via 10.0.12.1, 00:05:21, GigabitEthernet0/1
O     10.1.1.0/24 [110/20] via 10.0.12.1, 00:05:21, GigabitEthernet0/1
```

`10.1.0.0/22`는 `10.1.2.0/24`와 `10.1.3.0/24`까지 포함한다. 하지만 Routing Table에는 `10.1.2.0/24` 또는 `10.1.3.0/24`로 가는 Route가 없다. 이 상태에서 `10.1.2.10`이나 `10.1.3.10`으로 패킷을 보내면 해당 패킷은 `Null0`로 전달되어 폐기된다.

### OSPF Route Filtering

OSPF Route Filtering은 특정 Route가 다른 Area에 광고되것을 제한하는 기능이다. 

ABR Filtering은 Area 사이에 전달되는 Type 3 LSA를 제한한다.

`area 1 range 10.1.0.0 255.255.0.0 not-advertise`는 Area 1에서 `10.1.0.0 255.255.0.0`를 다른 Area에 광고하지 않는다. `not-advertise`는 큰 Network 범위를 한꺼번에 차단할 때 사용한다.
```
R3(config)# router ospf 1
R3(config-router)# area 1 range 10.1.0.0 255.255.0.0 not-advertise
```

`area filter-list prefix`는 Prefix List를 사용하여 특정 Type 3 LSA를 필터링하며, `in`은 지정한 Area로 들어오는 Route를 제한하고 `out`은 지정한 Area에서 나가는 Route를 제한한다.
```
ip prefix-list BLOCK35 seq 5 deny 10.1.35.0/24
ip prefix-list BLOCK35 seq 10 permit 0.0.0.0/0 le 32

router ospf 1
 area 2 filter-list prefix BLOCK35 in
```
- 다른 Area의 `10.1.35.0/24` Type 3 LSA가 Area 2로 들어오는 것을 차단한다.
- 나머지 Prefix는 Area 2로 들어오는 것을 허용한다.

```
ip prefix-list BLOCK35 seq 5 deny 10.1.35.0/24
ip prefix-list BLOCK35 seq 10 permit 0.0.0.0/0 le 32

router ospf 1
 area 1 filter-list prefix BLOCK35 out
```
- Area 1의 `10.1.35.0/24`가 다른 Area에 Type 3 LSA로 광고되는 것을 차단한다.
- 나머지 Prefix는 다른 Area로 광고되는 것을 허용한다.

ASBR은 Route Map을 사용하여 외부 Route가 OSPF로 Redistribution되는 것을 제한한다.
- 허용된 Route만 Type 5 또는 Type 7 LSA로 생성된다.
- 차단된 Route는 OSPF Domain에 광고되지 않는다.
- Route Map을 사용하여 External Metric과 Metric Type을 설정할 수 있다.
- Route를 차단하려면 ACL이나 Prefix List에서 대상을 `permit`으로 일치시키고 `route-map deny`에서 차단한다.

예를 들어 `10.11.16.0/24`는 OSPF로 Redistribution하지 않고, 나머지 Static Route는 E1 Metric `100`으로 Redistribution한다.
```
ip prefix-list BLOCK16 seq 5 permit 10.11.16.0/24

route-map STATIC-TO-OSPF deny 10
 match ip address prefix-list BLOCK16

route-map STATIC-TO-OSPF permit 20
 set metric 100
 set metric-type type-1

router ospf 1
 redistribute static subnets route-map STATIC-TO-OSPF
```
