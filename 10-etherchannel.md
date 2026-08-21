# EtherChannel / LACP

## 개념

### EtherChannel

EtherChannel은 여러 개의 물리적인 Ethernet Link를 하나의 논리적인 Link로 묶어 대역폭을 증가시키면서 동시에 이중화를 제공하는 기술이다.

여러 개의 물리적인 Port를 하나의 `Port-Channel` Interface에 포함시킨다. Layer 2 EtherChannel의 경우 STP에서는 여러 개의 물리적인 Link가 아니라 하나의 논리적인 Link로 인식한다.

또한 하나의 Member Port에 장애가 발생해도 나머지 Port를 통해 Traffic을 전달할 수 있다.

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
