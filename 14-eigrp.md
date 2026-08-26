# EIGRP

## 개념

EIGRP(Enhanced Interior Gateway Routing Protocol)는 Neighbor로부터 받은 Route 정보를 기반으로 최적 경로와 Backup 경로를 계산하는 Advanced Distance-Vector Routing Protocol이다. OSPF처럼 전체 Network의 Link-State 정보를 구성하지 않고 Neighbor가 광고한 Route 정보를 기반으로 경로를 계산하므로 `Routing by Rumor`라고도 표현한다.

EIGRP의 특징은 다음과 같다.
- IP Protocol Number `88`을 사용하며 TCP나 UDP를 사용하지 않는다.
- IPv4에서는 `224.0.0.10` Multicast Address를 사용한다.
- Internal EIGRP Route의 AD 값은 `90`, Redistribution된 External EIGRP Route의 AD 값은 `170`이다.
- Neighbor를 처음 형성할 때는 전체 EIGRP Route 정보를 교환하고, 이후에는 Route가 변경될 때 변경된 정보만 `Partial Update`로 전달한다.
- 경로에서 가장 낮은 Bandwidth와 모든 Interface Delay의 합을 기반으로 Metric을 계산하고, Metric이 가장 낮은 경로를 최적 경로로 선택한다. 
- Equal-Cost Load Balancing과 Unequal-Cost Load Balancing을 지원한다.
- Neighbor를 형성하려면 AS Number가 같아야 한다.

Equal-Cost Load Balancing은 같은 목적지로 가는 여러 경로의 Metric이 같을 때 모두 Routing Table에 등록하여 사용하는 방식이다.Unequal-Cost Load Balancing은 `variance`를 설정하여 Metric이 다른 경로도 Routing Table에 등록하여 함께 사용하는 방식이다.

### EIGRP Table

EIGRP는 세 가지 Table을 사용한다.

Neighbor Table: 직접 연결된 EIGRP Neighbor의 IP Address, Interface, Hold Time 및 Uptime 등을 저장한다.
```
R1# show ip eigrp neighbors

EIGRP-IPv4 Neighbors for AS(100)

H   Address         Interface       Hold  Uptime    SRTT   RTO  Q  Seq
                                  (sec)            (ms)       Cnt Num
0   10.0.12.2       Gi0/0             12  00:18:42    8   100  0  25
1   10.0.13.3       Gi0/1             11  00:15:10   10   100  0  31
```

Topology Table: Neighbor에게 학습한 모든 EIGRP Route와 Metric 정보를 저장한다.
```
R1# show ip eigrp topology

EIGRP-IPv4 Topology Table for AS(100)

P 192.168.10.0/24, 1 successors, FD is 28160
        via 10.0.12.2 (28160/25600), GigabitEthernet0/0

P 192.168.20.0/24, 1 successors, FD is 30720
        via 10.0.13.3 (30720/28160), GigabitEthernet0/1

P 192.168.30.0/24, 2 successors, FD is 33280
        via 10.0.12.2 (33280/30720), GigabitEthernet0/0
        via 10.0.13.3 (33280/30720), GigabitEthernet0/1
```
Routing Table: Topology Table에서 선택한 최적 경로인 Successor Route를 등록한다.

### EIGRP Packet

EIGRP는 주요 다섯 가지 Packet을 사용한다.

1\. Hello: Neighbor를 탐색하고 관계를 유지한다.

2\. Update: 새로운 Route나 변경된 Route 정보를 전달한다.

3\. Query: Successor를 잃고 Feasible Successor가 없을 때 Neighbor에게 다른 경로가 있는지 요청한다.

4\. Reply: Query를 보낸 Neighbor에게 대체 경로가 있는지 없는지 응답한다.

5\. ACK: Update, Query, Reply와 같은 Reliable Packet을 정상적으로 수신했음을 확인한다.

Hello Packet은 일반적으로 Multicast로 전송되며 ACK를 요구하지 않는다. 하지만 Update, Query, Reply Packet은 RTP(Reliable Transport Protocol)를 통해 전달되며 ACK를 요구한다.

### EIGRP Neighbor

EIGRP가 활성화된 Interface는 Hello Packet을 전송하여 Neighbor를 탐색한다. Neighbor를 형성하기 위해서는 다음 조건을 충족해야 한다.
- 양쪽 Router의 EIGRP AS Number가 같아야 한다.
- 양쪽 Router의 K-values가 같아야 한다.
- 양쪽 Interface의 IP Address와 통신할 수 있어야 한다.
- Authentication을 사용하면 인증 설정이 같아야 한다.

대부분의 일반적인 Link에서 Hello Interval의 기본값은 `5초`, Hold Time은 `15초`이다. 저속 NBMA Link에서는 일반적으로 `60초`와 `180초`를 사용한다.

EIGRP는 양쪽 Router의 Hello/Hold Interval이 서로 달라도 Neighbor를 형성할 수 있다. 각 Router는 상대방이 알려준 Hold Time 안에 Hello Packet을 포함한 EIGRP Packet을 수신하면 Neighbor 관계를 유지한다.

### EIGRP Metric

EIGRP는 Bandwidth, Delay, Load, Reliability 등의 값을 Metric 계산에 사용할 수 있지만 기본 K-values는 `K1=1, K2=0, K3=1, K4=0, K5=0`이다. 따라서 기본적으로 Bandwidth와 Delay만 Metric 계산에 사용한다.
- Bandwidth: 출발지에서 목적지까지의 경로 중 가장 낮은 Bandwidth를 사용한다.
- Delay: 출발지에서 목적지까지의 경로에 포함된 모든 Interface의 Delay를 합산한다.
```
Metric = 256 × [(10^7 / Lowest Bandwidth) + Sum of Delay]
```
Bandwidth는 `Kbps`, Delay는 `10 Microseconds` 단위로 계산하며, Metric 값이 가장 낮은 경로를 선택한다.

### Successor and Feasible Successor

Successor는 Routing Table에 등록되는 최적 경로이다.
- Successor은 Next-hop IP Address이다.

Feasible Distance(FD)는 현재 Router가 Successor를 통해 목적지까지 도달하는 데 사용하는 전체 Metric이다.

Reported Distance(RD)는 Neighbor가 해당 목적지까지 도달하는 데 사용하는 Metric이다, Advertised Distance(AD)라고도 한다.

Feasible Successor는 Successor에 장애가 발생했을 때 즉시 사용할 수 있는 Loop 없는 Backup 경로이다.

Backup 경로가 Feasible Successor가 되려면 다음 Feasibility Condition을 만족해야 한다.
```
Neighbor의 Reported Distance < 현재 Successor의 Feasible Distance
```

Neighbor의 RD가 현재 Router의 FD보다 작다는 것은 Neighbor가 현재 Router보다 목적지에 더 가까이 있다는 의미이다. 만약 RD가 FD보다 크거나 같으면 현재 Router는 “어? 이 Neighbor의 RD가 내 FD보다 큰데, 그러면 이 Neighbor가 나를 거쳐 목적지로 가고 있을 수도 있겠네. 이 경로를 Backup으로 선택하면 Loop가 발생할 수도 있겠다.”라고 판단한다.  

하지만 RD가 FD보다 작으면 “이 Neighbor는 나보다 목적지에 더 가까우니까 나를 다시 거쳐 가는 경로가 아니겠네.”라고 판단한다. 따라서 해당 경로를 Loop 없는 Backup 경로인 Feasible Successor로 저장한다.  

