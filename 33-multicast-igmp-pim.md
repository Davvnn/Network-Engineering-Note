# Multicast / IGMP / PIM 기본

## 개념

### Multicast

Multicast는 하나의 Source가 동일한 Data를 특정 Multicast Group에 가입한 여러 Receiver에게 전달하는 방식이다.

Source는 각 Receiver에게 Packet을 따로 전송하지 않고 Multicast Group Address를 Destination IP Address로 설정하여 전송한다.

### Multicast Address

IPv4 Multicast는 `224.0.0.0/4` 범위의 IP Address를 사용한다.
```
224.0.0.0 ~ 239.255.255.255
```

주요 Address 범위는 다음과 같다.
- `224.0.0.0/24`: 같은 Local Network 내의 Protocol Message에 사용하며, 해당 Multicast Traffic은 다른 Network로 전달되지 않는다.
- `232.0.0.0/8`: SSM(Source-Specific Multicast)에 사용하며, Receiver가 Traffic을 받을 Source를 직접 지정한다.
- `239.0.0.0/8`: 사내 Network처럼 제한된 범위에서 사용하는 Multicast Address이다. 예를 들어 사내 방송을 `239.1.1.1`로 전송하고, 해당 Traffic이 회사 외부로 전달되지 않도록 범위를 제한할 때 사용한다.

### IGMP

IGMP(Internet Group Management Protocol)는 Receiver가 같은 Network에 있는 Multicast Router에게 특정 Multicast Group의 Traffic을 받고 싶다고 알릴 때 사용하는 Protocol이다.

Receiver는 IGMP Membership Report를 전송하여 특정 Multicast Group의 Traffic을 수신하겠다고 Router에 알린다.

Multicast Router는 같은 Network에 있는 Receiver들에게 IGMP Query를 주기적으로 전송하여, Multicast Group에 가입한 Receiver가 계속 존재하는지 확인한다.

IGMP는 IP Protocol Number `2`를 사용한다.

### IGMP Version

#### IGMPv1

IGMPv1은 Multicast Group 가입 기능을 제공하지만 Group에서 즉시 탈퇴하는 Leave Message가 없다.

어떤 Receiver도 응답하지 않으면 Multicast Router는 Timer가 만료된 후 해당 Network의 Multicast Group 가입 정보를 삭제한다.

#### IGMPv2

IGMPv2는 Leave Group Message를 지원한다.

Receiver가 Multicast Group에서 탈퇴하면 Multicast Router는 해당 Group Address로 Group-Specific Query를 전송하여 다른 Receiver가 남아 있는지 확인한다.

#### IGMPv3

IGMPv3는 Receiver가 수신할 Multicast Group과 해당 Traffic을 전송하는 Source를 함께 지정할 수 있다.
```
IGMPv2: Group만 선택
239.1.1.1

IGMPv3: Source와 Group을 함께 선택
192.168.10.10 → 232.1.1.1
```

### IGMP Snooping

IGMP Snooping은 L2 Switch가 Receiver와 Router 사이의 IGMP Message를 확인하여 Multicast Group에 가입한 Port를 학습하는 기능이다.

IGMP Snooping이 없으면 Switch는 Multicast Group에 가입한 Receiver의 위치를 알 수 없으므로, Multicast Traffic을 같은 VLAN의 모든 Port로 Flooding할 수 있다.

IGMP Snooping을 사용하면 Multicast Traffic을 다음 Port에만 전달한다.
- 해당 Multicast Group에 가입한 Receiver Port
- Multicast Router가 연결된 Mrouter Port

### PIM

PIM(Protocol Independent Multicast)은 Router 사이에서 Multicast 전달 경로를 생성하는 Multicast Routing Protocol이다.

PIM은 Unicast Routing Table을 확인하여 Multicast Traffic이 Source 또는 RP 방향의 올바른 Interface로 들어왔는지 확인한다.

PIM은 특정 Routing Protocol만 사용하는 것이 아니라 Static Route, OSPF 및 EIGRP 등으로 생성된 Routing Table을 모두 사용할 수 있기 때문에 Protocol Independent라고 한다.
- PIM은 IP Protocol Number `103`을 사용한다.
- PIM Router는 `224.0.0.13`으로 PIM Message를 전송한다.

### PIM Dense Mode

PIM-DM(Dense Mode)은 Network의 여러 곳에 Receiver가 존재한다고 가정한다.

처음에는 Multicast Traffic을 모든 PIM Interface로 Flooding하고, Receiver가 없는 경로에 Prune Message를 전송하여 Traffic을 차단한다.
- Prune은 Multicast Traffic을 받을 Receiver가 없는 경로로 Traffic이 전달되지 않도록 차단하는 기능이다.

### PIM Sparse Mode

PIM-SM(Sparse Mode)은 Receiver가 Multicast Group에 가입하면, Receiver와 연결된 Router가 PIM Join Message를 전송하여 요청한 경로로만 Multicast Traffic이 전달되도록 한다.

PIM-SM에서는 Source의 Multicast Traffic과 Receiver의 가입 요청이 만나는 지점으로 RP(Rendezvous Point)를 사용한다.
- Multicast Traffic을 보내는 Source와 연결된 Router는 RP에게 Multicast Traffic이 발생한 것을 알리고, Receiver와 연결된 Router는 RP 방향으로 PIM Join Message를 전송한다. 
```
Source → R1 → R2(RP) → R3 → Receiver
```

### PIM SSM

PIM-SSM(Source-Specific Multicast)은 Receiver가 Multicast Group과 Source를 함께 지정하는 방식이다.

예를 들어 Receiver가 Source `192.168.10.10`이 `232.1.1.1` Group으로 전송하는 Traffic을 받으려는 경우 다음과 같이 표시한다.
```
(S,G): (192.168.10.10, 232.1.1.1)

S: Source IP Address
G: Multicast Group Address
```
SSM은 Receiver가 Source를 직접 지정하므로 RP가 필요하지 않다.

### PIM Register와 Register-Stop

First-Hop Router가 Source로부터 최초 Multicast Packet을 수신하면 해당 Packet을 PIM Register Message 안에 Encapsulation한다.

First-Hop Router는 PIM Register Message를 RP에 Unicast로 전송하여 새로운 Multicast Source가 있다는 것을 알린다.

RP는 PIM Register 안의 Multicast Packet을 Receiver 방향의 Shared Tree로 전달하고, Source 방향으로 PIM `(S,G)` Join Message를 전송한다.

RP가 Source로부터 Multicast Packet을 수신하면 First-Hop Router에 Register-Stop Message를 전송한다.

First-Hop Router는 Register-Stop을 수신하면 Multicast Packet을 PIM Register로 Encapsulation하는 것을 중단한다.

### Shared Tree

Shared Tree는 RP를 중심으로 생성되는 공통 Multicast 전달 경로이다.

Multicast Routing Table에서는 `(*,G)`로 표시한다.

```
(*, 239.1.1.1)
```
- `*`는 특정 Source를 지정하지 않았다는 의미이며, `G`는 Multicast Group을 의미한다.

### Source Tree

Source Tree 또는 SPT(Shortest Path Tree)는 RP를 거치지 않고 Receiver 방향 Router가 Source까지 생성한 최단 경로이다.

Multicast Routing Table에서는 `(S,G)`로 표시한다.
```
(192.168.10.10, 239.1.1.1)
```

### SPT Cutover

Last-Hop Router는 처음에 RP를 기준으로 생성된 Shared Tree를 통해 Multicast Traffic을 수신한다.

Last-Hop Router는 Packet의 Source IP Address를 확인한 후 Source 방향으로 PIM `(S,G)` Join Message를 전송하여 더 짧은 SPT를 생성한다.

SPT를 통해 Traffic을 수신하기 시작하면 기존 RP 방향의 경로를 Prune하여 더 이상 사용하지 않는다.

처음에는 RP 방향의 Shared Tree를 통해 Multicast Traffic을 수신하지만, 이후에는 Source에서 Receiver까지 더 효율적인 경로인 SPT를 사용한다.
```
Shared Tree: Source → RP → Receiver
SPT: Source → Receiver 최단 경로
```

### RPF

RPF(Reverse Path Forwarding)는 Multicast Packet이 올바른 Interface로 들어왔는지 확인하는 기능이다.

Router는 Unicast Routing Table을 조회하여 Source 또는 RP로 가는 경로를 확인한다.

Packet이 해당 경로의 Interface로 들어오면 전달하고, 다른 Interface로 들어오면 폐기한다.

---
