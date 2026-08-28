# Route Redistribution

## 개념

### Route Redistribution

Route Redistribution은 서로 다른 Routing Protocol 사이에서 Route 정보를 전달하는 기능이다.

OSPF를 사용하는 환경의 Route를 EIGRP 환경으로 Redistribution하기 위해서는 두 Routing Protocol이 연결되는 Router에서 OSPF와 EIGRP를 모두 실행해야 한다. 해당 Router에서 Redistribution을 설정하면 OSPF Route를 EIGRP Domain으로 전달할 수 있다.  
- 다른 Protocol에서 전달받은 Route는 일반적으로 External Route로 처리된다.
- 필요한 Route만 전달하도록 Prefix-List와 Route-Map을 사용하는 것이 안전하다.

### Redistribution 방향

Redistribution은 Route를 전달받는 Routing Protocol 아래에서 설정한다.

예를 들어 EIGRP AS `100`의 Route를 OSPF Process `1`에 전달하려면 다음과 같이 설정한다.
```
R2(config)# router ospf 1
R2(config-router)# redistribute eigrp 100 subnets
```

### Seed Metric

Routing Protocol마다 Metric을 계산하는 방식이 다르기 때문에 기존 Metric을 그대로 사용할 수 없다.

Seed Metric은 다른 Routing Protocol에서 전달받은 Route를 자신의 Routing Protocol에서 사용할 수 있도록 설정하는 Metric이다.
- OSPF는 Cost를 사용한다.
- EIGRP는 Bandwidth, Delay, Reliability, Load, MTU 값을 사용한다.
- RIP은 Hop Count를 사용한다.

#### EIGRP Seed Metric

다른 Dynamic Routing Protocol의 Route를 EIGRP로 Redistribution할 때는 다음 다섯 값을 직접 지정하거나 `default-metric`을 설정해야 한다. `default-metric`을 설정하면 다섯 값을 따로 지정하지 않아도 된다.  
```
R2(config)# router eigrp 100
R2(config-router)# redistribute ospf 1 metric 100000 100 255 1 1500
```

각 값의 의미는 다음과 같다.
- Bandwidth: `100000Kbps`
- Delay: `100` (10 Microsecond 단위)
- Reliability: `255`
- Load: `1`
- MTU: `1500`

#### OSPF Seed Metric

다른 Dynamic Routing Protocol의 Route를 OSPF로 Redistribution하면 OSPF는 해당 Route에 OSPF Cost를 설정한다.
- 이때 설정되는 OSPF Cost를 Seed Metric이라고 한다.

Metric을 따로 설정하지 않으면 기본 Seed Metric은 `20`이며, 기본 Route Type은 `E2`이다.
```
R2(config)# router ospf 1
R2(config-router)# redistribute eigrp 100 subnets
```

위와 같이 설정하면 해당 EIGRP Route는 Seed Metric `20`과 `E2` Type을 가진 OSPF External Route로 OSPF Domain에 광고된다.

필요한 경우 Seed Metric과 Route Type을 직접 변경할 수 있다.
```
R2(config)# router ospf 1
R2(config-router)# redistribute eigrp 100 subnets metric 30 metric-type 1
```
- `subnets`: Subnet Route을 OSPF로 Redistribution한다.  
- `metric 30`: Redistribution되는 Route의 OSPF Seed Metric을 `30`으로 설정한다.
- `metric-type 1`: Redistribution되는 Route를 OSPF `E1` Route로 설정한다.

#### E1

E1은 Seed Metric에 ASBR까지의 OSPF 내부 Cost를 더하여 최종 Metric을 계산한다.

예를 들어 Seed Metric이 `20`이고 ASBR까지의 OSPF Cost가 `10`이라면 최종 Metric은 `30`이 된다.

```
Seed Metric 20 + ASBR까지의 Cost 10 = 최종 Metric 30
```
Routing Table에서는 `O E1`으로 표시된다.

#### E2

E2는 ASBR까지의 OSPF 내부 Cost를 더하지 않고 Seed Metric만 사용한다.

예를 들어 Seed Metric이 `20`이라면 ASBR에서 멀어져도 기본적으로 Metric `20`을 유지한다.

```
Seed Metric 20 = 최종 Metric 20
```
E2는 OSPF Redistribution의 기본 Route Type이며 Routing Table에서는 `O E2`로 표시된다.
