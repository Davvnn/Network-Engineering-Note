# EtherChannel / LACP

## 개념

### EtherChannel

EtherChannel은 여러 개의 물리적인 Ethernet Link를 하나의 논리적인 Link로 묶어 대역폭을 증가시키면서 동시에 이중화를 제공하는 기술이다.

여러 개의 물리적인 Port를 하나의 `Port-Channel` Interface에 포함시킨다. Layer 2 EtherChannel의 경우 STP에서는 여러 개의 물리적인 Link가 아니라 하나의 논리적인 Link로 인식한다.

또한 하나의 Member Port에 장애가 발생해도 나머지 Port를 통해 Traffic을 전달할 수 있다.
- EtherChannel에 포함되는 물리적인 Interface를 Member Port라고 한다.
- EtherChannel에 포함되는 Member Port의 설정은 서로 일치해야 한다.

EtherChannel은 Layer 2 또는 Layer 3 방식으로 구성할 수 있다.

### Port-Channel

Port-Channel은 EtherChannel로 묶인 여러 개의 물리적인 Interface를 하나의 논리적인 Interface로 표현한 것이다.

예를 들어 다음 Interface를 EtherChannel Group `1`로 구성하면:
- `Gi0/1`
- `Gi0/2`

논리적인 Interface인 `Port-channel1`이 생성된다.

```
SW1# show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/1     unassigned      YES unset  up                    up
GigabitEthernet0/2     unassigned      YES unset  up                    up
Port-channel1          unassigned      YES unset  up                    up
```

실제 Traffic은 Member Port를 통해 전달되지만 STP나 Routing Protocol에서는 Port-Channel을 하나의 Interface로 인식한다.

### LACP

LACP(Link Aggregation Control Protocol)는 여러 개의 물리적인 Link를 하나의 논리적인 Link로 묶기 위해 사용하는 표준 Protocol이다.

LACP는 IEEE 표준이기 때문에 Cisco 장비뿐만 아니라 다른 Vendor의 장비와 EtherChannel을 구성할 때도 사용할 수 있다.

LACP를 사용하면 Switch들이 서로 LACP 메시지를 교환하여 EtherChannel을 구성할 수 있는 Port인지 확인한다.

### LACP Active

Active Mode는 상대방에게 LACP 메시지를 전송하여 적극적으로 EtherChannel 구성을 시도한다.

Active Mode는 상대방이 Active 또는 Passive Mode인 경우 EtherChannel을 구성할 수 있다.

### LACP Passive

Passive Mode는 먼저 LACP Negotiation을 시작하지 않고 상대방으로부터 LACP 메시지를 수신하면 응답한다.

Passive와 Passive를 연결하면 양쪽 모두 먼저 LACP Negotiation을 시작하지 않기 때문에 EtherChannel이 구성되지 않는다.

실무에서는 보통 양쪽을 `active` Mode로 구성한다.

### PAgP

PAgP(Port Aggregation Protocol)는 Cisco 전용 EtherChannel Negotiation Protocol이다.

PAgP에는 `Desirable`와 `Auto Mode`가 있다.

Desirable Mode는 LACP의 Active Mode와 같이 적극적으로 PAgP 메시지를 전송하며, Auto Mode는 LACP의 Passive Mode와 같이 상대방으로부터 PAgP 메시지를 수신하면 응답한다.

LACP가 IEEE 표준이기 때문에 일반적으로 다양한 Vendor 환경에서는 LACP를 사용한다. 

### Layer 2 EtherChannel

Layer 2 EtherChannel은 여러 Switchport를 하나의 논리적인 Layer 2 Port-Channel로 구성하는 방식이다.

예를 들어 SW1과 SW2 사이의 Trunk Link 2개를 EtherChannel로 묶을 수 있다.

### Layer 3 EtherChannel

Layer 3 EtherChannel은 여러 Routed Port를 하나의 논리적인 Layer 3 Interface로 구성하는 방식이다.

Member Port에서 `no switchport`를 설정하고 Port-Channel Interface에 IP Address를 설정한다.
```
SW1(config)# interface range gi0/1 - 2
SW1(config-if-range)# no switchport
SW1(config-if-range)# channel-group 1 mode active
SW1(config-if-range)# interface port-channel 1
SW1(config-if)# no switchport
SW1(config-if)# ip address 10.0.12.1 255.255.255.252
```
Layer 3 EtherChannel에서는 Routing을 할 때 각각의 Member Port를 사용하지 않고 `Port-channel1`을 하나의 Layer 3 Interface로 사용한다.


