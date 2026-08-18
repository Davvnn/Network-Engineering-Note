# Subnetting / VLSM / CIDR

## 개념

### Subnet Mask

Subnet Mask는 IP Address에서 Network 영역과 Host 영역을 구분하는 값이다.
```
192.168.1.10/24
```
- Subnet Mask: `255.255.255.0`
- Network Address: `192.168.1.0`
- Usable Host: `192.168.1.1~192.168.1.254`
- Broadcast Address: `192.168.1.255`

Network Address와 Broadcast Address는 장비에 할당할 수 없다.

### Subnetting

Subnetting은 하나의 큰 Network를 여러 개의 작은 Subnet으로 나누는 방식이다. 회사에서는 부서별 네트워크, 사무망, 공장망 등을 구분하고 각 네트워크의 규모에 맞게 IP Address를 할당하기 위해 사용한다.

`192.168.1.0/24`를 `/26`으로 Subnetting한다.

- 가져온 Host Bit: `2 Bit`
- 남아 있는 Host Bit: `6 Bit`
- Subnet 수: `2^2 = 4개`
- Subnet당 전체 IP Address 수: `2^6 = 64개`
- Subnet당 사용 가능한 Host 수: `2^6 - 2 = 62개`

생성되는 Subnet:

- `192.168.1.0/26`
- `192.168.1.64/26`
- `192.168.1.128/26`
- `192.168.1.192/26`


### VLSM

VLSM(Variable Length Subnet Mask)은 하나의 Network를 필요한 Host 수에 따라 서로 다른 크기의 Subnet으로 나누는 방식이다.

예를 들어 Host가 100대인 부서에는 `/25`, 20대인 부서에는 `/27`, Point-to-Point Link에는 `/30`을 할당할 수 있다.

VLSM을 적용할 때는 큰 주소 공간을 먼저 확보하기 위해 Host 수가 가장 많은 Network부터 할당한다.

### CIDR

CIDR(Classless Inter-Domain Routing)은 Class A, B, C와 같은 고정된 Class 기준을 사용하지 않고 Prefix Length로 Network의 크기를 표현하는 방식이다.

```
192.168.1.0/24
```

CIDR은 필요한 크기에 맞게 IP Address를 할당하거나 여러 개의 연속된 Network를 하나의 Summary Route로 묶을 때 사용한다.

### 주요 Prefix Length

- `/24`: Subnet Mask `255.255.255.0`, 사용 가능한 Host `254`
- `/25`: Subnet Mask `255.255.255.128`, 사용 가능한 Host `126`
- `/26`: Subnet Mask `255.255.255.192`, 사용 가능한 Host `62`
- `/27`: Subnet Mask `255.255.255.224`, 사용 가능한 Host `30`
- `/28`: Subnet Mask `255.255.255.240`, 사용 가능한 Host `14`
- `/29`: Subnet Mask `255.255.255.248`, 사용 가능한 Host `6`
- `/30`: Subnet Mask `255.255.255.252`, 사용 가능한 Host `2`
- `/31`: Point-to-Point Link에서 사용
- `/32`: 하나의 특정 Host Address를 나타낼 때 사용

---

## 동작 원리

### Subnetting

`192.168.1.0/24` Network를 네 개의 동일한 Subnet으로 나눈다.

1\. 네 개의 Subnet이 필요하므로 Host 영역에서 `2 Bit`를 가져온다.

```
2^2 = 4 Subnets
```

2\. Prefix Length는 `/24`에서 `/26`으로 변경된다.

```
Subnet Mask: 255.255.255.192
```

3\. Host Bit는 `6개`가 남으므로 각 Subnet은 `64개`의 IP Address를 가진다.

```
2^6 = 64
```

4\. Network Address와 Broadcast Address를 제외하면 `62개`의 Host Address를 사용할 수 있다.

```
64 - 2 = 62
```

5\. Block Size는 `64`이며 다음과 같이 Subnet이 나뉜다.

- `192.168.1.0/26`
  - Usable Host: `192.168.1.1~192.168.1.62`
  - Broadcast Address: `192.168.1.63`

- `192.168.1.64/26`
  - Usable Host: `192.168.1.65~192.168.1.126`
  - Broadcast Address: `192.168.1.127`

- `192.168.1.128/26`
  - Usable Host: `192.168.1.129~192.168.1.190`
  - Broadcast Address: `192.168.1.191`

- `192.168.1.192/26`
  - Usable Host: `192.168.1.193~192.168.1.254`
  - Broadcast Address: `192.168.1.255`

### CIDR Route Summarization

다음 네 개의 연속된 Network를 하나의 Summary Route로 묶을 수 있다.

```
192.168.0.0/24
192.168.1.0/24
192.168.2.0/24
192.168.3.0/24
```

Summary Route:

```
192.168.0.0/22
```

`192.168.0.0/22`는 `192.168.0.0`부터 `192.168.3.255`까지의 주소 범위를 포함한다.

Route Summarization을 사용하면 Routing Table의 Route 수와 Routing Update의 양을 줄일 수 있다.

---

## 예시 및 구성도

`192.168.10.0/24` Network를 다음 요구 사항에 맞게 VLSM으로 나눈다.

- Sales: Host `100대`
- Development: Host `50대`
- Management: Host `20대`
- R1-R2 Link: Host `2대`

1\. Host 수가 가장 많은 Sales Network에 `/25`를 할당한다.

```
Network: 192.168.10.0/25
Usable Host: 192.168.10.1~192.168.10.126
Broadcast: 192.168.10.127
```

2\. Development Network에 `/26`을 할당한다.

```
Network: 192.168.10.128/26
Usable Host: 192.168.10.129~192.168.10.190
Broadcast: 192.168.10.191
```

3\. Management Network에 `/27`을 할당한다.

```
Network: 192.168.10.192/27
Usable Host: 192.168.10.193~192.168.10.222
Broadcast: 192.168.10.223
```

4\. R1-R2 Point-to-Point Link에 `/30`을 할당한다.

```
Network: 192.168.10.224/30
R1: 192.168.10.225
R2: 192.168.10.226
Broadcast: 192.168.10.227
```

5\. 필요한 Host 수에 따라 서로 다른 Prefix Length를 사용하므로 IP Address의 낭비를 줄일 수 있다.

---

## Troubleshooting

1\. 단말의 IP Address, Subnet Mask 및 Default Gateway 설정이 올바른지 확인한다.

2\. 단말의 IP Address가 해당 Subnet의 Usable Host 범위에 포함되는지 확인한다.

3\. Network Address나 Broadcast Address가 단말에 할당되지 않았는지 확인한다.

4\. 단말과 Default Gateway에 서로 다른 Subnet Mask가 설정되지 않았는지 확인한다.
- Subnet Mask가 잘못 설정되면 목적지를 같은 Network 또는 다른 Network로 잘못 판단할 수 있다.

5\. 여러 Subnet의 주소 범위가 서로 겹치지 않는지 확인한다.

6\. Default Gateway의 IP Address가 단말과 같은 Subnet에 포함되는지 확인한다.

7\. 다른 Subnet과 통신할 수 없다면 Routing Table에 목적지로 가는 Route와 출발지로 돌아오는 Return Path가 있는지 확인한다.

8\. Summary Route를 사용하는 경우 실제로 존재하지 않는 Network까지 Summary 범위에 포함되지 않았는지 확인한다.
- 잘못된 Summary Route는 Packet이 잘못된 경로로 전달되어 폐기되는 Routing Black Hole을 발생시킬 수 있다.

---

## 실무 질문

Subnetting은 무엇인가?
- 하나의 큰 Network를 여러 개의 작은 Subnet으로 나누는 방식이다.

Subnetting을 사용하는 이유는 무엇인가?
- 부서나 서비스별로 Network를 구분하고 IP Address를 효율적으로 사용하기 위해서이다.

Subnet Mask는 어떤 역할을 하는가?
- IP Address에서 Network 영역과 Host 영역을 구분한다.

Network Address와 Broadcast Address의 차이는 무엇인가?
- Network Address는 Subnet 자체를 나타내고, Broadcast Address는 해당 Subnet의 모든 장비로 전송할 때 사용한다.

`/26`에서 사용할 수 있는 Host 수는 몇 개인가?
- 전체 주소는 `64개`이며, Network Address와 Broadcast Address를 제외한 `62개`를 사용할 수 있다.

VLSM은 무엇인가?
- 필요한 Host 수에 따라 서로 다른 Prefix Length를 사용하여 Subnet을 나누는 방식이다.

VLSM을 적용할 때 큰 Network부터 할당하는 이유는 무엇인가?
- 큰 Network에 필요한 연속된 주소 공간을 먼저 확보하고 주소 범위의 중복을 방지하기 위해서이다.

CIDR은 무엇인가?
- Class 구분 없이 Prefix Length를 사용하여 Network의 크기를 표현하는 방식이다.

Route Summarization을 사용하는 이유는 무엇인가?
- 여러 개의 연속된 Network를 하나의 Route로 묶어 Routing Table을 단순하게 관리하기 위해서이다.

Subnet Mask가 잘못 설정되면 어떤 문제가 발생하는가?
- 단말이 목적지를 같은 Network 또는 다른 Network로 잘못 판단하여 통신하지 못할 수 있다.
