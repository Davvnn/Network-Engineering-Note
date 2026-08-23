# SVI / Inter-VLAN Routing

## 개념

### SVI

SVI(Switched Virtual Interface)는 Switch에 구성된 VLAN을 Layer 3 Interface로 사용하기 위해 생성하는 논리적인 Interface이다.

SVI에 IP Address를 설정하여 VLAN에 속한 단말들의 Default Gateway로 사용한다.

예를 들어 VLAN 10의 Network가 `192.168.10.0/24`라면 다음과 같이 구성할 수 있다.
- VLAN 10 SVI: `192.168.10.1/24` 
- PC1: `192.168.10.10/24`
- PC1 Default Gateway: `192.168.10.1`

대부분의 SVI는 Backbone Switch에 구성하고, 서로 다른 VLAN 대역 간의 통신은 Backbone Switch에서 Routing한다.

Layer 2 Switch에서도 SVI를 설정하며, 해당 SVI는 SSH 접속, Ping, SNMP 등 Switch를 관리하기 위한 관리용 IP Address로 사용한다.
- 해당 Layer 2 Switch에서 사용하는 관리용 SVI의 Default Gateway는 Backbone Switch에 구성된다.
