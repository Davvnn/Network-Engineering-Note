# SVI / Inter-VLAN Routing

## 개념

### SVI (Switched Virtual Interface)

SVI(Switched Virtual Interface)는 Switch에 구성된 VLAN을 Layer 3 Interface로 사용하기 위해 생성하는 논리적인 Interface이다.

SVI에 IP Address를 설정하여 VLAN에 속한 단말들의 Default Gateway로 사용한다.

예를 들어 VLAN 10의 Network가 `192.168.10.0/24`라면 다음과 같이 구성할 수 있다.
- VLAN 10 SVI: `192.168.10.1/24` 
- PC1: `192.168.10.10/24`
- PC1 Default Gateway: `192.168.10.1`

대부분의 SVI는 Backbone Switch에 구성하고, 서로 다른 VLAN 대역 간의 통신은 Backbone Switch에서 Routing한다.

Layer 2 Switch에서도 SVI를 설정하며, 해당 SVI는 SSH 접속, Ping, SNMP 등 Switch를 관리하기 위한 관리용 IP Address로 사용한다.
- 해당 Layer 2 Switch에서 사용하는 관리용 SVI의 Default Gateway는 Backbone Switch에 구성된다.

### Inter-VLAN Routing

VLAN들은 서로 다른 Broadcast Domain으로 동작하기에 서로 다른 VLAN과 통신할 수 없다.

Inter-VLAN Routing은 Layer 3 Switch를 사용하여 서로 다른 VLAN 사이의 Traffic을 Routing하게 해준다.

Layer 3 Switch에서는 각 VLAN에 SVI를 생성하고 `ip routing`을 활성화하여 Inter-VLAN Routing을 구성한다.

### Router-on-a-Stick

Router-on-a-Stick은 Router의 하나의 Physical Interface에 여러 Subinterface를 생성하여 Inter-VLAN Routing을 한다.

Switch와 Router에 사이의 Link를 Trunk를 구성하고, Router의 각 Subinterface에는 VLAN별 802.1Q Encapsulation과 Default Gateway IP Address를 설정한다.
```
R1(config)# interface gi0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0  

R1(config)# interface gi0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0  
```
모든 VLAN Traffic이 하나의 물리적인 Link를 사용하기 때문에 작은 Network에서 주로 사용한다.

Router-on-a-Stick은 Layer 3 Switch가 사용되기 전에 사용하던 Inter-VLAN Routing 방식이다.

현재는 Layer 3 Switch의 SVI를 사용하여 Inter-VLAN Routing을 구성하기 때문에 실무에서는 일반적으로 많이 사용하지 않는다. 

하지만 Layer 3 Switch가 없고 Layer 2 Switch와 Router만 사용하는 SOHO와 같은 소규모 Network 환경에서는 사용할 수 있다.

---
