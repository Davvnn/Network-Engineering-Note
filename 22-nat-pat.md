# NAT / PAT

## 개념

### NAT

NAT(Network Address Translation)는 Router가 Packet의 IP Address를 다른 IP Address로 변경하는 기능이다.

주로 내부 Private IP Address를 Public IP Address로 변환하여 Internet과 통신할 수 있도록 사용한다.
- 내부 Interface에는 `ip nat inside`를 설정한다.
- 외부 Interface에는 `ip nat outside`를 설정한다.

### Source and Destination NAT

Source NAT는 Packet의 Source IP Address를 변환하는 방식이다.
- 내부 Client가 외부 Network로 나갈 때 Private IP Address를 Public IP Address로 변환하기 위해 사용한다.
```
192.168.10.100 → 203.0.113.2
```

Destination NAT는 Packet의 Destination IP Address를 변환하는 방식이다.
- 주로 외부 사용자가 Public IP Address로 내부 Server에 접속할 때 사용한다.
```
203.0.113.6 → 192.168.20.10
```

### NAT 용어

Inside Local: 내부 장비의 Private IP Address
- `192.168.10.100`
Inside Global: NAT 변환 후 외부에서 보이는 IP Address
- `203.0.113.2`
Outside Local: 내부에서 바라보는 외부 장비의 IP Address
- `8.8.8.8`
Outside Global: 외부 장비가 실제로 사용하는 IP Address
- `8.8.8.8`

### Static NAT

Static NAT는 하나의 Private IP Address와 하나의 Public IP Address를 `1:1`로 고정하여 매핑하는 방식이다.
```
192.168.20.10 ↔ 203.0.113.6
```
Static NAT는 내부 장비마다 Public IP Address가 하나씩 필요하기 때문에 Public IP Address 절약 효과가 적다.

### Dynamic NAT

Dynamic NAT는 내부 장비가 외부로 통신할 때 Public IP Address Pool에서 사용 가능한 주소를 임시로 할당하는 방식이다.

Public IP Address가 10개라면 동시에 최대 10개의 내부 IP Address에 NAT를 적용할 수 있다.

Pool의 Public IP Address가 모두 사용 중이면 새로운 NAT Translation을 생성할 수 없다.


### PAT

PAT(Port Address Translation)는 여러 내부 장비가 하나의 Public IP Address를 함께 사용할 수 있도록 IP Address와 Port Number를 함께 변환하는 방식이다.

Cisco에서는 PAT를 `NAT Overload`라고도 한다.
```
192.168.10.100:51000 → 203.0.113.2:30001
192.168.10.101:51000 → 203.0.113.2:30002
```

같은 Public IP Address를 사용하지만 변환된 Port Number가 다르기 때문에 각각의 통신을 구분할 수 있다.
- ICMP는 Port Number가 없으므로 ICMP Identifier 등을 이용하여 구분한다.
- 일반적인 회사와 가정에서는 PAT를 가장 많이 사용한다.

### Static PAT

Static PAT는 특정 Public IP Address와 Port로 들어오는 Traffic을 내부 Server의 특정 Port로 전달하는 방식이다.
```
203.0.113.6:443 → 192.168.20.10:443
```

### Static Outside PAT

Static Outside PAT는 내부 사용자에게 외부 Server를 다른 IP Address와 Port로 보이게 하는 방식이다.

예를 들어 실제 외부 Web Server가 `198.51.100.10:80`이지만 내부 사용자는 `10.0.6.100:6783`으로 접속할 수 있다.
```
내부에서 보이는 주소: 10.0.6.100:6783
실제 외부 Server: 198.51.100.10:80
```

내부 사용자가 `10.0.6.100:6783`으로 접속하면 Router가 Destination 정보를 실제 외부 Server인 `198.51.100.10:80`으로 변환한다.

응답 Packet이 돌아오면 Source 정보를 다시 `10.0.6.100:6783`으로 변환하여 내부 사용자에게 전달한다.
- 일반적인 Static PAT는 외부 사용자가 내부 Server에 접속할 때 사용한다.
- Static Outside PAT는 내부 사용자가 외부 Server를 다른 주소로 보이게 할 때 사용한다.

### Dynamic PAT using Outside Interface

Router의 외부 Interface에 설정된 IP Address를 여러 내부 장비가 함께 사용하는 방식이다.
```
R1 Gi0/1 IP Address: 203.0.113.2
```

모든 내부 사용자는 `203.0.113.2`를 공유하고 Port Number를 통해 각각의 통신을 구분한다.

가장 일반적으로 사용하는 PAT 방식이다.

