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

