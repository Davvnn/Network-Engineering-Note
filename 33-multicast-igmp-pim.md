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
- `224.0.0.0/24`: - `224.0.0.0/24`: 같은 Local Network 내의 Protocol Message에 사용하며, 해당 Multicast Traffic은 다른 Network로 전달되지 않는다.
- `232.0.0.0/8`: SSM(Source-Specific Multicast)에 사용하며, Receiver가 Traffic을 받을 Source를 직접 지정한다.
- `239.0.0.0/8`: 사내 Network처럼 제한된 범위에서 사용하는 Multicast Address이다. 예를 들어 사내 방송을 `239.1.1.1`로 전송하고, 해당 Traffic이 회사 외부로 전달되지 않도록 범위를 제한할 때 사용한다.

### IGMP

IGMP(Internet Group Management Protocol)는 Receiver가 같은 Network에 있는 Multicast Router에게 특정 Multicast Group의 Traffic을 받고 싶다고 알릴 때 사용하는 Protocol이다.

Receiver는 IGMP Membership Report를 전송하여 특정 Multicast Group의 Traffic을 수신하겠다고 Router에 알린다.

Router는 IGMP Query를 주기적으로 전송하여 Group에 가입한 Receiver가 여전히 존재하는지 확인한다.

Multicast Router는 같은 Network에 있는 Receiver들에게 IGMP Query를 주기적으로 전송하여, 해당 Multicast Group의 Traffic을 받는 Receiver가 계속 존재하는지 확인한다.

IGMP는 IP Protocol Number `2`를 사용한다.

### IGMP Version

#### IGMPv1

IGMPv1은 Multicast Group 가입 기능을 제공하지만 Group에서 즉시 탈퇴하는 Leave Message가 없다.

어떤 Receiver도 응답하지 않으면 Multicast Router는 Timer가 만료된 후 해당 Network의 Multicast Group 가입 정보를 삭제한다.

#### IGMPv2

IGMPv2는 Leave Group Message를 지원한다.

Receiver가 Group에서 탈퇴하면 Router가 Group-Specific Query를 전송하여 다른 Receiver가 남아 있는지 확인한다.

Receiver가 Multicast Group에서 탈퇴하면 Multicast Router는 해당 Group Address로 Group-Specific Query를 전송한다. 

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
