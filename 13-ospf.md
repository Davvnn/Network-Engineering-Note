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

