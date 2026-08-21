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


----------------------------------------(Need Review)------------------------------------------------------


### EtherChannel과 STP

EtherChannel을 사용하지 않고 SW1과 SW2 사이에 여러 개의 Layer 2 Link를 연결하면 STP는 Layer 2 Loop를 방지하기 위해 일부 Link를 Blocking 상태로 전환할 수 있다.

하지만 여러 Link를 EtherChannel로 구성하면 STP는 Member Port 각각을 별도의 Link로 보지 않고 하나의 Port-Channel로 인식한다.

따라서 EtherChannel 내부의 개별 Member Port를 STP가 하나씩 Blocking하지 않는다.

Port-Channel 전체가 하나의 STP Port처럼 동작한다.

### Load Balancing

EtherChannel은 하나의 Traffic을 여러 Member Port에 Packet 단위로 번갈아 전송하는 방식이 아니라, 특정 값의 Hash를 계산하여 Traffic을 Member Port에 분산한다.

일반적으로 다음과 같은 정보를 기준으로 Load Balancing을 수행할 수 있다.

- Source MAC Address
- Destination MAC Address
- Source/Destination MAC Address
- Source IP Address
- Destination IP Address
- Source/Destination IP Address
- Layer 4 Port 정보

Load Balancing 방식은 Switch Model과 IOS Version에 따라 지원되는 방식이 다를 수 있다.

예를 들어 Source/Destination IP Address를 기준으로 Load Balancing하는 경우 특정 Source와 Destination 사이의 Traffic은 동일한 Member Port를 사용할 수 있다.

따라서 1Gbps Link 4개를 EtherChannel로 구성했다고 해서 하나의 TCP Session이 반드시 4Gbps의 대역폭을 사용하는 것은 아니다.

여러 Flow가 서로 다른 Member Port로 분산되면서 전체 EtherChannel의 대역폭을 효율적으로 사용할 수 있다.

### LACP System Priority

LACP를 사용하는 장비는 System Priority와 System MAC Address를 이용하여 LACP System ID를 구성한다.

LACP System Priority는 어떤 장비가 Link Aggregation 과정에서 더 높은 우선순위를 가지는지 결정할 때 사용된다.

Priority 값이 낮을수록 우선순위가 높다.

기본값은 일반적으로 `32768`이다.

```
SW1(config)# lacp system-priority 4096
```







////////////////////



## 동작 원리

### LACP EtherChannel 동작 과정

1\. SW1과 SW2 사이에 `Gi0/1`과 `Gi0/2` Link가 연결되어 있다.

2\. 관리자는 SW1의 `Gi0/1`과 `Gi0/2`를 LACP Active Mode로 설정한다.

```
SW1(config)# interface range gi0/1 - 2
SW1(config-if-range)# channel-group 1 mode active
```

3\. SW2의 `Gi0/1`과 `Gi0/2`도 LACP Active Mode로 설정한다.

```
SW2(config)# interface range gi0/1 - 2
SW2(config-if-range)# channel-group 1 mode active
```

4\. SW1과 SW2는 서로 LACP 메시지를 교환한다.

5\. LACP는 각 Member Port의 정보와 설정을 확인하고 EtherChannel을 구성할 수 있는지 확인한다.

6\. 정상적으로 Negotiation이 완료되면 `Gi0/1`과 `Gi0/2`가 하나의 `Port-channel1`로 묶인다.

7\. STP는 `Gi0/1`과 `Gi0/2`를 각각의 Link로 처리하지 않고 `Port-channel1`을 하나의 논리적인 Link로 처리한다.

8\. Traffic은 설정된 Load Balancing 방식에 따라 Member Port로 분산된다.

9\. 만약 `Gi0/1`에 Link 장애가 발생하면 해당 Port는 EtherChannel에서 제외된다.

10\. `Gi0/2`가 계속 Port-Channel의 Member Port로 동작하기 때문에 Traffic은 중단되지 않고 나머지 Link를 통해 전달된다.

11\. 장애가 발생한 `Gi0/1`이 정상 상태로 복구되면 LACP Negotiation을 거친 후 다시 EtherChannel의 Member Port로 참여한다.

### LACP Active와 Passive 동작 과정

1\. SW1의 Port는 `active` Mode로 설정하고 SW2의 Port는 `passive` Mode로 설정한다.

2\. SW1은 LACP 메시지를 SW2로 먼저 전송한다.

3\. SW2는 LACP 메시지를 수신하고 자신의 LACP 정보를 응답한다.

4\. 양쪽 Port의 설정이 EtherChannel을 구성할 수 있는 조건을 만족하면 Port-Channel이 생성된다.

5\. 만약 양쪽 Switch를 모두 `passive` Mode로 설정하면 어느 쪽도 먼저 LACP Negotiation을 시작하지 않는다.

6\. 따라서 LACP EtherChannel이 정상적으로 구성되지 않는다.

### Member Port 장애 발생 과정

1\. SW1과 SW2 사이의 `Gi0/1`과 `Gi0/2`가 `Port-channel1`로 구성되어 있다.

2\. 정상적인 상태에서는 두 Member Port를 통해 여러 Traffic Flow가 분산되어 전달된다.

3\. `Gi0/1`에서 Cable 또는 Interface 장애가 발생하여 Link가 Down된다.

4\. LACP는 해당 Port가 더 이상 정상적으로 사용할 수 없는 것을 확인하고 EtherChannel에서 제외한다.

5\. Port-Channel 자체는 `Gi0/2`가 살아 있기 때문에 Up 상태를 유지한다.

6\. Traffic은 남아 있는 `Gi0/2`를 통해 계속 전달된다.

7\. `Gi0/1`이 복구되면 다시 LACP 정보를 교환하고 정상적인 Member Port로 EtherChannel에 참여한다.

## 예시 및 구성도 (L2 트렁크 예시)

### LACP EtherChannel

![](images/10-etherchannel-lacp.png)

SW1과 SW2 사이에 `Gi0/1`과 `Gi0/2` 두 개의 Link가 연결되어 있다.

두 Link는 LACP를 사용하여 `Port-channel1`로 구성한다.

1\. SW1과 SW2의 `Gi0/1`, `Gi0/2`를 Trunk Port로 설정한다.

2\. SW1과 SW2의 Interface를 LACP Active Mode로 설정하여 `Port-channel1`을 구성한다.

3\. SW1과 SW2는 LACP 메시지를 교환하고 두 Port의 설정이 EtherChannel 구성 조건을 만족하는지 확인한다.

4\. 정상적으로 Negotiation이 완료되면 `Gi0/1`과 `Gi0/2`가 `Port-channel1`의 Member Port로 동작한다.

5\. VLAN Traffic은 `Port-channel1`을 통해 전달되고 실제 Frame은 Load Balancing 결과에 따라 Member Port 중 하나를 통해 전송된다.

6\. 만약 SW1과 SW2 사이의 `Gi0/1`에 장애가 발생하면 해당 Interface는 EtherChannel에서 제외된다.

7\. `Gi0/2`가 계속 정상적으로 동작하고 있기 때문에 `Port-channel1`은 Up 상태를 유지하며 Traffic을 계속 전달한다.

8\. `Gi0/1`이 복구되면 LACP Negotiation을 거친 후 다시 `Port-channel1`의 Member Port로 참여한다.

## 명령어

### Layer 2 LACP EtherChannel 설정

SW1의 `Gi0/1`과 `Gi0/2`를 Trunk로 구성하고 LACP Active Mode를 설정한다.

```
SW1(config)# interface range gi0/1 - 2
SW1(config-if-range)# switchport mode trunk
SW1(config-if-range)# channel-group 1 mode active
```

SW2에도 동일하게 설정한다.

```
SW2(config)# interface range gi0/1 - 2
SW2(config-if-range)# switchport mode trunk
SW2(config-if-range)# channel-group 1 mode active
```

`channel-group 1`을 설정하면 `Port-channel1` Interface가 생성된다.

### Port-Channel 설정

```
SW1(config)# interface port-channel 1
SW1(config-if)# switchport mode trunk
```

Port-Channel Interface에 Trunk 설정을 적용한다.

Allowed VLAN을 제한하려면 다음과 같이 설정할 수 있다.

```
SW1(config)# interface port-channel 1
SW1(config-if)# switchport trunk allowed vlan 10,20,30
```

### LACP Active / Passive 설정

SW1을 Active Mode로 설정한다.

```
SW1(config)# interface range gi0/1 - 2
SW1(config-if-range)# channel-group 1 mode active
```

SW2를 Passive Mode로 설정한다.

```
SW2(config)# interface range gi0/1 - 2
SW2(config-if-range)# channel-group 1 mode passive
```

Active와 Passive 조합이므로 LACP EtherChannel을 구성할 수 있다.

### Static EtherChannel 설정

Negotiation Protocol을 사용하지 않고 EtherChannel을 강제로 구성한다.

```
SW1(config)# interface range gi0/1 - 2
SW1(config-if-range)# channel-group 1 mode on
```

상대방 Switch도 동일하게 `mode on`으로 설정해야 한다.

### Layer 3 LACP EtherChannel 설정

Member Port를 Routed Port로 변경한다.

```
SW1(config)# interface range gi0/1 - 2
SW1(config-if-range)# no switchport
SW1(config-if-range)# channel-group 1 mode active
```

Port-Channel Interface에 IP Address를 설정한다.

```
SW1(config)# interface port-channel 1
SW1(config-if)# no switchport
SW1(config-if)# ip address 10.0.12.1 255.255.255.252
SW1(config-if)# no shutdown
```

상대방 Switch에도 동일한 방식으로 구성한다.

### EtherChannel 상태 확인

```
SW1# show etherchannel summary
```

예시:

```
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Gi0/1(P) Gi0/2(P)
```

- `Po1`: Port-channel1
- `S`: Layer 2 Port-Channel
- `U`: Port-Channel이 정상적으로 사용 중인 상태
- `LACP`: LACP를 사용하여 구성
- `P`: 해당 Interface가 Port-Channel에 정상적으로 Bundled된 상태

### EtherChannel 상세 정보 확인

```
SW1# show etherchannel 1 detail
```

EtherChannel Group과 Member Port의 상세 정보를 확인한다.

### LACP Neighbor 확인

```
SW1# show lacp neighbor
```

상대방 Switch의 LACP System ID와 연결된 Port 정보를 확인한다.

### Port-Channel Interface 확인

```
SW1# show interfaces port-channel 1
```

Port-Channel의 상태, Traffic Counter 및 Interface 정보를 확인한다.

### EtherChannel Member Port 확인

```
SW1# show interfaces gi0/1 etherchannel
```

해당 Interface가 어느 EtherChannel Group에 포함되어 있는지 확인한다.

### Load Balancing 방식 확인

```
SW1# show etherchannel load-balance
```

예시:

```
EtherChannel Load-Balancing Configuration:
        src-dst-ip
```

현재 EtherChannel이 Source/Destination IP Address를 기준으로 Load Balancing하도록 설정되어 있는 것을 확인한다.

### Load Balancing 방식 설정

```
SW1(config)# port-channel load-balance src-dst-ip
```

Source IP Address와 Destination IP Address를 기준으로 Hash를 계산하여 Member Port를 선택하도록 설정한다.

지원되는 방식은 장비에 따라 다를 수 있다.

### LACP System Priority 설정

```
SW1(config)# lacp system-priority 4096
```

LACP System Priority를 설정한다.

낮은 값일수록 높은 우선순위를 가진다.

### LACP Port Priority 설정

```
SW1(config)# interface gi0/1
SW1(config-if)# lacp port-priority 100
```

해당 Interface의 LACP Port Priority를 설정한다.

낮은 값일수록 높은 우선순위를 가진다.

## Troubleshooting

### EtherChannel이 정상적으로 구성되지 않는 경우

1\. EtherChannel이 정상적으로 구성되지 않으면 Port-Channel과 Member Port 상태를 확인한다.

```
SW1# show etherchannel summary
SW1# show interfaces status
```

2\. Member Port가 정상적으로 `P` 상태로 Port-Channel에 포함되어 있는지 확인한다.

```
SW1# show etherchannel summary
```

3\. LACP를 사용하는 경우 양쪽 Switch의 LACP Mode가 올바르게 설정되어 있는지 확인한다.
- Active + Active: 정상
- Active + Passive: 정상
- Passive + Passive: 구성되지 않음

4\. 양쪽 Switch의 Member Port 설정이 동일한지 확인한다.

```
SW1# show running-config interface gi0/1
SW1# show running-config interface gi0/2
```

5\. Layer 2 EtherChannel이라면 Access/Trunk Mode, VLAN, Native VLAN 및 Allowed VLAN 설정이 일치하는지 확인한다.

6\. Member Port의 Speed와 Duplex 설정이 서로 일치하는지 확인한다.

7\. LACP Neighbor가 정상적으로 확인되는지 확인한다.

```
SW1# show lacp neighbor
```

8\. 한쪽 Member Port에 장애가 있다면 Cable과 Interface 상태를 확인한다.

```
SW1# show interfaces gi0/1
```

9\. 설정을 수정한 후 모든 Member Port가 정상적으로 Port-Channel에 포함되었는지 다시 확인한다.

```
SW1# show etherchannel summary
```

## 실무 질문

EtherChannel을 사용하는 이유는 무엇인가?
- 여러 개의 물리적인 Link를 하나의 논리적인 Link로 묶어 대역폭을 증가시키고 Link 이중화를 제공하기 위해 사용한다.

EtherChannel과 Port-Channel의 차이는 무엇인가?
- EtherChannel은 여러 개의 물리적인 Link를 하나로 묶는 기술이고, Port-Channel은 EtherChannel로 생성되는 논리적인 Interface이다.

LACP를 사용하는 이유는 무엇인가?
- 여러 물리적인 Link를 하나의 EtherChannel로 구성하고, 상대방 장비와 LACP 메시지를 교환하여 Link Aggregation 상태를 자동으로 확인하기 위해 사용한다.

LACP Active와 Passive의 차이는 무엇인가?
- Active는 LACP Negotiation을 먼저 시작하고, Passive는 상대방으로부터 LACP 메시지를 수신하면 응답한다.

LACP에서 Passive와 Passive를 연결하면 EtherChannel이 구성되는가?
- 구성되지 않는다. 양쪽 모두 먼저 LACP Negotiation을 시작하지 않기 때문이다.

LACP와 PAgP의 차이는 무엇인가?
- LACP는 IEEE 표준 Protocol이므로 여러 Vendor 장비에서 사용할 수 있고, PAgP는 Cisco 전용 Protocol이다.

`mode on`과 LACP의 차이는 무엇인가?
- `mode on`은 Negotiation 없이 EtherChannel을 강제로 구성하고, LACP는 상대방과 LACP 메시지를 교환하여 EtherChannel을 구성한다.

EtherChannel Member Port의 설정은 왜 같아야 하는가?
- 여러 물리적인 Port가 하나의 논리적인 Interface로 동작하기 때문에 Access/Trunk Mode, VLAN, Speed, Duplex 등의 설정이 일치해야 정상적으로 하나의 EtherChannel로 구성할 수 있다.

EtherChannel을 구성하면 STP는 각각의 Member Port를 따로 계산하는가?
- 아니다. STP는 EtherChannel에 포함된 Member Port를 각각의 Link로 보지 않고 Port-Channel 전체를 하나의 논리적인 Link로 처리한다.

1Gbps Link 4개를 EtherChannel로 구성하면 하나의 TCP Session이 4Gbps를 사용할 수 있는가?
- 일반적으로 그렇지 않다. EtherChannel은 Hash 값을 기준으로 Flow를 특정 Member Port에 할당하기 때문에 하나의 Session은 보통 하나의 Member Port를 통해 전달된다.

EtherChannel에서 Member Port 하나가 Down되면 Port-Channel도 Down되는가?
- 남아 있는 정상 Member Port가 있다면 Port-Channel은 계속 Up 상태를 유지하고 나머지 Link를 통해 Traffic을 전달할 수 있다.

`show etherchannel summary`에서 `P`는 무엇을 의미하는가?
- 해당 물리적인 Interface가 Port-Channel에 정상적으로 Bundled되어 동작하고 있다는 의미이다.

Layer 2 EtherChannel과 Layer 3 EtherChannel의 차이는 무엇인가?
- Layer 2 EtherChannel은 Switchport를 묶어 VLAN Traffic을 전달하고, Layer 3 EtherChannel은 Routed Port를 묶어 Port-Channel Interface에 IP Address를 설정하여 Routing에 사용한다.

LACP를 사용할 때 양쪽 Switch를 모두 Active Mode로 설정해도 되는가?
- 가능하다. Active와 Active는 양쪽 모두 LACP Negotiation을 시작하기 때문에 정상적으로 EtherChannel을 구성할 수 있다.

실무에서 Static EtherChannel보다 LACP를 사용하는 이유는 무엇인가?
- LACP는 상대방 장비와 Negotiation을 통해 EtherChannel 상태를 확인할 수 있기 때문에 설정 오류나 Link 상태를 확인하기 쉽고, 다른 Vendor 장비와도 구성할 수 있다.
