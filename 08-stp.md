# STP / RSTP / MSTP

## 개념

### STP

STP(Spanning Tree Protocol)는 Switch 간에 이중화 Link가 구성된 Layer 2 Network에서 Loop가 발생하지 않도록 방지하는 프로토콜이다.

Layer 2 Header에는 Packet과 같은 TTL이 없기 때문에 Loop가 발생하면 Frame이 계속 돌면서 Broadcast Storm이 일어나 운행 중인 장비가 과부하로 꺼질 수 있다.

STP는 Switch들이 BPDU(Bridge Protocol Data Unit)를 교환하여 Root Bridge를 선출하고, 목적지까지의 경로 중 하나만 Forwarding 상태로 사용한다. 나머지 이중화 경로는 Blocking 상태로 전환하여 Loop를 방지한다.

만약 현재 사용 중인 Link에 장애가 발생하면 Blocking 상태였던 Link는 Forwarding 상태로 전환된다.

### Root Bridge

Root Bridge는 STP Topology의 기준이 되는 Switch이다.

Root Bridge는 가장 낮은 Bridge ID를 가진 Switch로 선출된다. Bridge ID는 Bridge Priority와 MAC Address를 기준으로 결정한다.
- Bridge Priority가 가장 낮은 Switch가 Root Bridge로 선출된다.
- Priority가 같으면 MAC Address가 가장 낮은 Switch가 Root Bridge로 선출된다.
- Priority는 일반적으로 `4096` 단위로 설정한다.

### STP Port

STP Port는 다음 과정을 거쳐 Forwarding 상태로 전환된다.
1\. Blocking: BPDU만 수신하며 Frame 전달과 MAC Address 학습은 하지 않는다.

2\. Listening: BPDU를 송수신하지만 Frame 전달과 MAC Address 학습은 하지 않는다.

3\. Learning: BPDU를 송수신하고 MAC Address를 학습하지만 Frame은 전달하지 않는다.

4\. Forwarding: BPDU를 송수신하고 MAC Address를 학습하며 Frame을 전달한다.
- Disabled: BPDU와 Frame을 처리하지 않으며 MAC Address도 학습하지 않는다.

### STP Port 역할

Root Port는 Non-Root Switch에서 Root Bridge까지 가장 낮은 Path Cost를 가진 인터페이스이다. Switch마다 하나의 Root Port를 선택한다.

Designated Port는 각 Network Segment에서 Root Bridge 방향으로 가장 좋은 경로를 제공하는 인터페이스이다. Root Bridge의 모든 동작 중인 Port는 Designated Port가 된다.

Blocking Port는 Loop를 방지하기 위해 일반 Frame을 전달하지 않는 인터페이스이다. 하지만 다른 스위치들이 보내는 BPDU는 계속 수신하여 STP Topology의 변경을 확인한다.

### STP Convergence Timer

Traditional STP는 Port가 Blocking 상태에서 Forwarding 상태로 전환되는 데 30초 이상이 걸릴 수 있다.
- 기존 BPDU 정보가 만료될 때까지 기다려야 하는 경우 Max Age Timer로 최대 20초가 소요될 수 있다. 
- 이후 Listening 상태에서 15초, Learning 상태에서 15초 동안 머무른 후 Forwarding 상태로 전환된다.
- 따라서 상황에 따라 최대 약 50초가 소요될 수 있으며, 해당 Timer 값은 변경할 수 있다.

### RSTP

RSTP(Rapid Spanning Tree Protocol)는 Traditional STP보다 빠르게 Network Topology를 수렴하도록 개선된 프로토콜이다.

RSTP도 BPDU를 교환하여 Root Bridge를 선출하고, 이중화 경로 중 하나를 Discarding 상태로 전환하여 Layer 2 Loop를 방지한다.

Traditional STP는 Forward Delay Timer(Listening과 Learning)를 기다린 후 Port를 Forwarding 상태로 전환하지만, RSTP는 스위치들끼리 Proposal과 Agreement BPDU를 교환하여 Loop가 없음을 확인하면 Port를 빠르게 Forwarding 상태로 전환한다.

### RSTP Port

RSTP Port는 Discarding, Learning 및 Forwarding를 사용한다.
- RSTP는 Traditional STP의 Blocking, Listening 및 Disabled 상태를 Discarding 상태로 통합했다.

1\. Discarding: BPDU를 송수신하지만 Frame 전달과 MAC Address 학습은 하지 않는다.

2\. Learning: BPDU를 송수신하고 MAC Address를 학습하지만 Frame은 전달하지 않는다.

3\. Forwarding: BPDU를 송수신하고 MAC Address를 학습하며 Frame을 전달한다.

### RSTP의 Port 역할

Root Port는 Root Bridge까지 가장 좋은 경로를 가진 Port이다.

Designated Port는 각 Network Segment에서 Root Bridge까지 가장 좋은 경로를 제공하는 Port이다.

Alternate Port는 Root Port를 대신할 수 있는 예비 Discarding Port이다.

Backup Port는 Designated Port를 대신할 수 있는 예비 Discarding Port이다.
- Backup Port는 주로 Hub와 같이 하나의 Network Segment에 여러 Port가 연결된 환경에서 발생한다. 현재는 Hub를 거의 사용하지 않기 때문에 실제 Network에서 볼 일은 많지 않다.

### RSTP Convergence

1\. 스위치는 자신의 Designated Port를 통해 Proposal Bit가 포함된 BPDU를 이웃 Switch로 전송한다.

2\. BPDU를 수신한 Switch는 Root Bridge ID, Root Path Cost, Sender Bridge ID 및 Port ID를 비교한다.

3\. 더 우수한 BPDU를 수신하면 BPDU에 포함된 Root 정보를 받아들이고, 해당 BPDU를 수신한 Port를 Root Port로 선택한다.

4\. Switch는 임시 Loop를 방지하기 위해 Root Port를 제외한 Non-Edge Designated Port를 Discarding 상태로 전환한다.
- 이 과정을 Sync라고 한다.
- Edge Port는 Sync 대상에서 제외된다.

5\. Sync가 완료되면 Agreement Bit가 포함된 BPDU를 상위 Switch로 전송한다.

6\. Agreement BPDU를 수신한 Switch는 해당 Designated Port를 빠르게 Forwarding 상태로 전환한다.

7\. 같은 과정이 하위 Switch에서도 반복되면서 전체 RSTP Topology가 빠르게 수렴한다.

RSTP는 Traditional STP처럼 Listening과 Learning 상태에서 각각 15초의 Timer를 기다리지 않는다.
- 하지만 상대 Switch가 RSTP를 지원하지 않으면 해당 Port는 Traditional STP 방식으로 동작한다.

RSTP의 Link Type
- Point-to-Point: Switch 간에 Full-Duplex로 연결된 Link이다.
- Shared: Hub와 같이 여러 장비가 하나의 Segment에 Half-Duplex로 동작하는 Link이다.
- Edge: PC나 Server와 같은 단말이 연결된 Port이다.

RSTP Sync는 Point-to-Point Link에서만 동작한다.

### PVST+ / Rapid PVST+


