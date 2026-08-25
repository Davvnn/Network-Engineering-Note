# OSPF

## 개념

### OSPF

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

#### OSPF Router 역할

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


