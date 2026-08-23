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

## 동작 원리

### SVI를 이용한 Inter-VLAN Routing
- PC1: `192.168.10.10/24`
- PC1 Default Gateway (VLAN 10): `192.168.10.1`
- Server: `192.168.20.10/24`
- Server Default Gateway (VLAN 20): `192.168.20.1`

1\. PC1은 목적지 `192.168.20.10`이 자신의 Network에 없다는 것을 확인한다.

2\. PC1은 Packet을 Default Gateway인 VLAN 10 SVI로 전송하기 위해 SVI의 MAC Address를 ARP로 확인한다.
- SVI를 생성하면 Switch가 해당 SVI에 MAC Address를 할당하며, Vendor와 장비에 따라 여러 SVI가 같은 MAC Address를 사용할 수 있다. 하지만 ARP는 VLAN별로 따로 동작하기 때문에 같은 MAC Address를 사용해도 문제가 발생하지 않는다.

3\. PC1은 Destination MAC Address를 VLAN 10 SVI의 MAC Address로 설정하여 Frame을 전송한다.

4\. Layer 3 Switch는 VLAN 10 SVI로 전달된 Frame의 Layer 2 Header를 제거하고 Destination IP Address를 확인한다.

5\. Routing Table에서 `192.168.20.0/24` Network가 VLAN 20 SVI에 연결되어 있는 것을 확인한다.

6\. Layer 3 Switch는 ARP Table에서 Server의 MAC Address를 확인한다.

7\. Layer 3 Switch는 Source MAC Address를 VLAN 20 SVI의 MAC Address로, Destination MAC Address를 Server의 MAC Address로 설정하여 새로운 Frame을 생성한다.

8\. 새로운 Frame을 VLAN 20에 속한 Server로 전달한다.

9\. Server가 응답할 때도 Default Gateway인 VLAN 20 SVI로 Packet을 전송하고 Layer 3 Switch가 VLAN 10으로 Routing한다.

Routing 과정에서 Source IP Address와 Destination IP Address는 유지되지만 Source MAC Address와 Destination MAC Address는 변경된다.

### Router-on-a-Stick 동작

1\. PC1은 다른 VLAN에 있는 목적지로 Packet을 전송하기 위해 자신의 Default Gateway로 Frame을 전송한다.

2\. Switch는 Router 방향의 Trunk Port로 Frame을 전달하면서 VLAN Tag를 추가한다.

3\. Router의 Subinterface는 Frame을 수신하고 Routing Table에서 목적지 Network를 확인한다.

4\. Router는 목적지 VLAN Tag가 포함된 새로운 Frame을 생성하여 동일한 Physical Interface로 다시 전송한다.

4\. Router는 목적지 Network에 연결된 Subinterface를 선택하고, 해당 Subinterface의 설정에 따라 VLAN Tag를 추가하여 동일한 Physical Interface로 Frame을 전송한다.

5\. Switch는 VLAN Tag를 확인하여 목적지 VLAN으로 Frame을 전달하고, Access Port에서 해당 Frame의 VLAN Tag를 제거한 후 Untagged 형식으로 목적지 단말로 전송한다.

---

