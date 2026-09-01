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

내부 장비가 먼저 통신을 시작해야 Router에 Translation 정보가 생성되며, 외부에서 먼저 내부 Server에 접속하도록 하려면 Public IP Address와 Port를 내부 Server에 미리 연결하는 Static PAT를 설정해야 한다.  
```
203.0.113.6:443 → 192.168.20.10:443
```
- Static PAT 설정이 없으면 외부 사용자가 Public IP Address `203.0.113.6`의 TCP Port `443`으로 접속했을 때 Router는 해당 Traffic을 어떤 내부 Server로 전달해야 하는지 모른다. 따라서 `203.0.113.6:443 → 192.168.20.10:443`과 같이 Static PAT를 설정하여 외부에서 들어온 Traffic을 내부 Web Server로 전달할 수 있도록 한다.

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

### Dynamic PAT with Pool

여러 내부 장비가 여러 개의 Public IP Address를 Port Number로 나누어 사용하는 방식이다.

```
Public IP Pool: 203.0.113.3 ~ 203.0.113.5
```

서로 다른 내부 장비가 같은 Public IP Address를 사용하더라도 변환된 Port Number가 다르기 때문에 각각의 통신을 구분할 수 있다.

```
192.168.10.100:51000 → 203.0.113.3:30001
192.168.10.101:52000 → 203.0.113.3:30002
192.168.10.102:53000 → 203.0.113.4:30003
192.168.10.103:54000 → 203.0.113.4:30004
192.168.10.104:55000 → 203.0.113.5:30005
192.168.10.105:56000 → 203.0.113.5:30006
```
Dynamic NAT와 Dynamic PAT의 차이는 `overload` 사용 여부이다.
- `overload` 없음: 하나의 내부 IP Address가 Public IP Address 하나를 사용한다.
- `overload` 있음: 여러 내부 IP Address가 Public IP Address를 공유하며, Port Number로 각각의 통신을 구분한다.

### Two-Way NAT

A 회사와 B 회사가 Router를 통해 연결되어 있지만, 두 회사 모두 `10.16.0.0/24`를 사용하면 IP Address가 서로 겹치는 문제가 발생한다.

이를 해결하기 위해 상대방의 장비를 실제 IP Address가 아닌 다른 IP Address로 보이게 한다.
- A Client 실제 IP: `10.16.0.10`
- A 회사에서 보이는 B Server: `10.16.6.100`
- B Server 실제 IP: `10.16.0.100`
- B 회사에서 보이는 A Client: `10.16.7.10`

A Client는 B Server의 실제 IP Address를 모르고 `10.16.6.100`으로 접속한다.
```
Source IP: 10.16.0.10
Destination IP: 10.16.6.100
```

Router는 Source와 Destination IP Address를 모두 변환한다.
```
Source IP: 10.16.7.10
Destination IP: 10.16.0.100
```

B Server는 요청을 자신의 실제 IP Address인 `10.16.0.100`으로 받으며, A Client를 `10.16.7.10`으로 인식한다.

응답 Traffic은 Router에서 반대로 변환된다.
```
Source IP: 10.16.0.100
Destination IP: 10.16.7.10
```
```
Source IP: 10.16.6.100
Destination IP: 10.16.0.10
```
즉, 두 회사는 실제로 같은 IP Address 대역을 사용하지만 상대방의 장비를 다른 대역으로 보이게 하여 Address 충돌 없이 통신할 수 있다.

### NAT ACL

PAT와 Dynamic NAT에서 ACL은 NAT를 적용할 내부 Source IP Address를 선택한다.
```
R1(config)# access-list 10 permit 192.168.10.0 0.0.0.255
```
- ACL의 `permit`: 해당 Source IP Address에 NAT를 적용한다.
- ACL의 `deny`: Traffic을 차단하는 것이 아니라 NAT 적용 대상에서 제외한다.

---

## 동작 원리

### PAT 동작 과정

Client `192.168.10.100`이 외부 Web Server `198.51.100.10:443`으로 접속하는 경우이다.

1\. Client는 다음 Packet을 Default Gateway인 R1으로 전송한다.
```
Source: 192.168.10.100:51000
Destination: 198.51.100.10:443
```

2\. R1은 Packet을 `ip nat inside`가 설정된 Interface에서 수신한다.

3\. R1은 NAT ACL을 확인하여 Source IP Address가 NAT 적용 대상인지 확인한다.

4\. PAT 적용 대상이면 Source IP Address와 Source Port를 변환한다.
```
192.168.10.100:51000 → 203.0.113.2:30001
```

5\. R1은 변환 정보를 NAT Translation Table에 저장하고 Packet을 외부로 전달한다.

6\. 외부 Web Server는 `203.0.113.2:30001`로 응답한다.

7\. R1은 NAT Translation Table을 확인하여 Destination 정보를 원래 Client 정보로 변환한다.
```
203.0.113.2:30001 → 192.168.10.100:51000
```

8\. 변환된 응답 Packet을 내부 Client에게 전달한다.

### Static PAT 동작 과정

1\. 외부 사용자가 `203.0.113.6:443`으로 접속한다.

2\. R1은 Static PAT 설정을 확인한다.

3\. Destination 정보를 내부 Web Server 주소로 변환한다.
```
203.0.113.6:443 → 192.168.20.10:443
```

4\. 내부 Web Server의 응답은 R1에서 다시 Public IP Address로 변환된다.

5\. 외부 사용자는 내부 Server의 실제 IP Address를 알 수 없으며, 자신이 접속한 `203.0.113.6:443`에서 응답이 돌아온 것으로 인식한다.

### Static Outside PAT 동작 과정

1\. 내부 사용자는 외부 Server의 변환된 주소인 `10.0.6.100:6783`으로 접속한다.

2\. R1은 Static Outside PAT 설정을 확인한다.

3\. R1은 Destination 정보를 실제 외부 Server 주소로 변환한다.
```
10.0.6.100:6783 → 198.51.100.10:80
```

4\. 외부 Server는 R1으로 응답한다.

5\. R1은 응답 Packet의 Source 정보를 내부에서 사용하는 주소로 변환한다.
```
198.51.100.10:80 → 10.0.6.100:6783
```

6\. 내부 사용자에게는 `10.0.6.100:6783`에서 응답한 것으로 보인다.

### Two-Way NAT 동작 과정

1\. A 회사 Client는 B 회사 Server의 변환된 주소인 `10.16.6.100`으로 Packet을 전송한다.

2\. Packet은 NAT가 설정된 R1으로 전달된다.

3\. R1은 Source IP Address를 `10.16.0.10`에서 `10.16.7.10`으로 변환한다.

4\. R1은 Destination IP Address를 `10.16.6.100`에서 실제 B 회사 Server 주소인 `10.16.0.100`으로 변환한다.

5\. R1은 변환된 Packet을 R2로 전달하고, R2는 B 회사 Server로 Routing한다.

6\. B 회사 Server는 A Client를 `10.16.7.10`으로 인식하고 응답한다.

7\. 응답 Packet은 R2를 거쳐 R1으로 전달된다.

8\. R1은 응답 Packet의 Source와 Destination IP Address를 반대로 변환하여 A 회사 Client에게 전달한다.

---
