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

