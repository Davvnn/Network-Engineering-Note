# HSRP / VRRP

## 개념

### HA, Redundancy 및 Fault Tolerance

HA(High Availability)는 장비에 장애가 발생해도 서비스를 계속 제공할 수 있도록 구성하는 것이다.

Redundancy는 같은 역할을 수행하는 장비나 Link를 여러 개 구성하는 것이며, Fault Tolerance는 하나의 장비에 장애가 발생해도 다른 장비가 그 역할을 이어받도록 하는 것이다.

FHRP(First Hop Redundancy Protocol)는 Default Gateway를 이중화하여 Network의 가용성을 높이는 Protocol이다.

### FHRP

FHRP(First Hop Redundancy Protocol)는 여러 Gateway 장비가 하나의 Virtual IP Address를 공유하여 Client의 Default Gateway를 이중화하는 Protocol이다.

Client는 Virtual IP Address를 Default Gateway로 사용한다.

Active Router 또는 Link에 장애가 발생하면 Backup Router가 Virtual IP Address와 Virtual MAC Address를 이어받아 Traffic을 전달한다.

대표적인 FHRP는 다음과 같다.
- HSRP: Cisco 전용 Protocol이다.
- VRRP: 표준 Protocol이다.
- GLBP: Cisco 전용 Protocol이며 Gateway Load Balancing을 제공한다.

### Virtual IP Address

Virtual IP Address는 Client가 Default Gateway로 사용하는 가상의 IP Address이다.
```
R1 실제 IP Address: 192.168.10.2
R2 실제 IP Address: 192.168.10.3
Virtual IP Address: 192.168.10.1  
```
Client는 R1이나 R2의 실제 IP Address가 아니라 Virtual IP Address인 `192.168.10.1`을 Default Gateway로 사용한다.

### Virtual MAC Address

Client가 Virtual IP Address에 대해 ARP Request를 전송하면 HSRP의 Active Router 또는 VRRP의 Master Router가 Virtual MAC Address로 응답한다.
```
192.168.10.1 → Virtual MAC Address
```
새로운 Active Router는 Gratuitous ARP를 전송하고, Switch들은 Virtual MAC Address를 새로운 Active Router 방향의 Interface로 다시 학습한다.

### HSRP

HSRP(Hot Standby Router Protocol)는 Cisco에서 만든 Default Gateway 이중화 Protocol이다.

HSRP Group에서 Active 장비가 Traffic을 전달하고 Standby 장비는 Active 장비의 상태를 확인하면서 대기하다가 장애 발생 시 Active 역할을 이어받는다. 
- Active: 현재 Default Gateway 역할을 수행한다.
- Standby: Active 장비에 장애가 발생하면 역할을 이어받는다.
- Listen: Active나 Standby는 아니지만 HSRP Hello Message를 수신한다.

### HSRP State

HSRP는 다음 상태를 거쳐 Active 또는 Standby 장비를 결정한다.

1\. Initial: HSRP가 아직 시작되지 않은 상태이다.

2\. Learn: Virtual IP Address와 현재 Active 장비의 정보를 확인하는 상태이다.

3\. Listen: Active나 Standby가 되지 않고 다른 HSRP 장비의 Hello Message를 듣고 있는 상태이다.

4\. Speak: Hello Message를 주고받으며 Active와 Standby 장비를 선출하는 상태이다.

5\. Standby: Active 장비에 장애가 발생하면 대신 Active가 되기 위해 대기하는 상태이다.

6\. Active: Virtual IP Address와 Virtual MAC Address를 사용하여 실제 Traffic을 전달하는 상태이다.

### HSRP Version

HSRP Version 1은 Group Number `0~255`를 사용하며 IPv4 Multicast Address `224.0.0.2`로 Hello Message를 전송한다.
- HSRP Version 1 Virtual MAC: `0000.0c07.acXX`

HSRP Version 2는 Group Number `0~4095`를 사용하며 IPv4 Multicast Address `224.0.0.102`로 Hello Message를 전송한다.
- HSRP Version 2 Virtual MAC: `0000.0c9f.fXXX`

두 Version 모두 HSRP Control Message를 주고받을 때 UDP Port `1985`를 사용한다.

HSRP Version 1과 Version 2는 서로 호환되지 않으므로 같은 Group의 장비는 동일한 Version을 사용해야 한다.

### VRRP

VRRP(Virtual Router Redundancy Protocol)는 여러 Vendor의 장비에서 사용할 수 있는 표준 Default Gateway 이중화 Protocol이다.

하나의 VRRP Group에서 Master 장비가 Traffic을 전달하고 Backup 장비는 Master 장비의 장애에 대비한다.

### VRRP State

VRRP는 다음 세 가지 State를 사용한다.

1\. Initialize: VRRP가 시작되지 않았거나 Interface가 Down 상태이다.

2\. Backup: Master가 주기적으로 보내는 Advertisement Message를 수신하면서 대기하며, 일정 시간 동안 Message가 수신되지 않으면 Master로 전환된다.

3\. Master: Virtual IP Address와 Virtual MAC Address를 사용하여 Traffic을 전달한다.

### VRRP Advertisement

VRRP Master는 Advertisement Message를 전송하여 자신이 정상적으로 동작하고 있음을 Backup 장비에 알린다.
- IPv4 Multicast Address: `224.0.0.18`
- IP Protocol Number: `112`
- Virtual MAC Address: `0000.5e00.01XX`
- 기본 Advertisement Interval: `1초`

VRRP는 TCP나 UDP를 사용하지 않고 IP Protocol Number `112`를 직접 사용한다.

### Priority

HSRP와 VRRP는 Priority가 높은 장비를 Gateway 역할로 선출한다.

두 장비의 Priority가 같으면 일반적으로 Interface IP Address가 높은 장비가 선출된다.

#### HSRP Priority

HSRP의 기본 Priority는 `100`이다.
```
R1 Priority: 110
R2 Priority: 100
```
R1의 Priority가 더 높으므로 Active 장비로 선출된다.

#### VRRP Priority

VRRP의 기본 Priority도 `100`이다.

Virtual IP Address를 자신의 실제 Interface IP Address로 사용할 수 있다. 

사용하는 IP Address Owner는 Priority `255`를 사용하며 VRRP Group에서 Master가 된다.  

Virtual IP Address를 실제 Interface IP Address로 가지고 있는 장비를 IP Address Owner라고 한다.  
```
R1 실제 IP Address: 192.168.10.1
R2 실제 IP Address: 192.168.10.2
Virtual IP Address: 192.168.10.1  
```  
- R1에 장애가 발생하면 R2가 Virtual IP Address를 이어받아 Master가 된다.
- IP Address Owner인 R1이 복구되면 Priority `255`로 Master 역할을 다시 가져온다.
- VRRP는 preempt가 기본적으로 활성화되어 있다.  

### Preempt

Preempt는 더 높은 Priority를 가진 장비가 현재 Gateway 역할을 가져오는 기능이다.

Priority `110`인 R1이 장애 후 복구되어도 Preempt가 없으면 Priority `100`인 R2가 계속 Active 역할을 수행할 수 있다.
- HSRP: Preempt가 기본적으로 비활성화되어 있으므로 직접 설정한다.
- VRRP: Preempt가 기본적으로 활성화되어 있다.

### HSRP Timer

HSRP의 기본 Timer는 다음과 같다.
- Hello Time: `3초`
- Hold Time: `10초`

Active 장비는 기본적으로 3초마다 Hello Message를 전송한다.

Standby 장비가 Hold Time 동안 Hello Message를 받지 못하면 Active 장비에 장애가 발생한 것으로 판단하고 Active 역할을 이어받는다.

### VRRP Timer

VRRP Master는 기본적으로 1초마다 Advertisement Message를 전송한다.

Backup 장비가 Master Down Interval 동안 Advertisement Message를 받지 못하면 Master 장비에 장애가 발생한 것으로 판단하고 새로운 Master를 선출한다.

실제 Master Down Interval은 Advertisement Interval과 Priority를 기준으로 계산한다.

### Object Tracking

HSRP와 VRRP는 Interface나 IP SLA 등의 상태를 추적하여 Priority를 자동으로 낮출 수 있다.
- Interface Tracking: Uplink Interface의 Up/Down 상태를 추적한다.
- IP SLA Tracking: 특정 IP Address까지 정상적으로 통신할 수 있는지 추적한다.

Gateway 장비의 Client 연결 Interface가 정상이어도 외부 Uplink에 장애가 발생하면 Traffic을 외부로 전달할 수 없다.

Object Tracking을 사용하면 Uplink 장애 시 Priority를 낮추어 정상적인 Uplink를 가진 Backup 장비가 Gateway 역할을 가져가도록 할 수 있다.
```
R1 기본 Priority: 110
Uplink 장애 시 감소 값: 20
R1 변경 Priority: 90
R2 Priority: 100
```
R1의 Priority가 `90`으로 낮아지면 Priority `100`인 R2가 Gateway 역할을 가져간다.

### GLBP

GLBP(Gateway Load Balancing Protocol)는 여러 Gateway 장비가 하나의 Virtual IP Address를 공유하면서 Traffic을 분산하여 전달하는 Cisco 전용 FHRP이다.

HSRP와 VRRP는 하나의 Group에서 Active 또는 Master 장비만 Traffic을 전달하지만, GLBP는 여러 장비가 동시에 Traffic 전달에 참여할 수 있다.

GLBP는 하나의 Virtual IP Address에 여러 개의 Virtual MAC Address를 사용한다.

### AVG와 AVF

GLBP는 AVG와 AVF 역할을 사용한다.
- AVG(Active Virtual Gateway): Virtual IP Address에 대한 Client의 ARP Request에 응답하고, 여러 AVF의 Virtual MAC Address 중 하나를 Client에게 알려준다.
- AVF(Active Virtual Forwarder): 자신에게 할당된 Virtual MAC Address로 들어오는 Traffic을 실제로 전달한다.
- 하나의 GLBP Group에는 하나의 AVG와 여러 AVF가 존재할 수 있다.
- AVG 역할을 수행하는 장비도 AVF로서 Traffic 전달에 참여할 수 있다.

### VLAN별 Gateway 분산

HSRP와 VRRP도 VLAN마다 Active 또는 Master 장비를 다르게 지정하여 Traffic을 분산할 수 있다.
```
VLAN 10: R1 Active, R2 Standby
VLAN 20: R2 Active, R1 Standby
```

STP를 함께 사용한다면 각 VLAN의 STP Root Bridge와 Active Gateway를 같은 Switch로 맞추는 것이 좋다.
```
VLAN 10: SW1 STP Root + HSRP Active
VLAN 20: SW2 STP Root + HSRP Active
```
기존 Standby Multi-layer Switch를 특정 VLAN의 Active Gateway로 설정한다면, 해당 VLAN의 STP Root Bridge도 같은 Switch로 지정하는 것이 좋다. 이렇게 하면 단말 Traffic이 다른 Switch를 불필요하게 거치지 않고 Active Gateway로 전달될 수 있다.
