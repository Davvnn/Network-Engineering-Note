# PBR

## 개념

### PBR

PBR(Policy-Based Routing)은 일반적인 Routing Table의 경로가 아니라 관리자가 설정한 정책에 따라 Packet의 경로를 결정하는 기능이다.

일반적인 Routing은 Destination IP Address를 기준으로 경로를 선택하지만, PBR은 Source IP Address, Destination IP Address, Protocol 및 Port Number 등을 기준으로 경로를 선택할 수 있다.

예를 들어 일반 사용자의 Traffic은 ISP1로 전달하고 Guest Network의 Traffic은 ISP2로 전달할 수 있다.

### PBR 구성 요소

PBR은 ACL과 Route-Map을 사용하여 구성한다.
- ACL: PBR을 적용할 Packet을 선택한다.
- `match`: Packet이 ACL의 조건과 일치하는지 확인한다.
- `set`: 조건에 일치한 Packet을 전달할 Next-Hop이나 Interface를 지정한다.
- `ip policy route-map`: Route-Map을 Packet이 들어오는 Interface에 적용한다.

Standard ACL은 Source IP Address를 기준으로 Packet을 선택한다.

Extended ACL은 Source IP Address, Destination IP Address, Protocol 및 Port Number를 기준으로 Packet을 선택할 수 있다.

### PBR에서 ACL의 `deny`

PBR의 `match`에 사용하는 ACL은 Packet을 차단하는 용도가 아니라 PBR을 적용할 Packet을 선택하는 용도이다.
- ACL `permit`: PBR 대상으로 선택한다.
- ACL `deny`: PBR 대상으로 선택하지 않고 다음 Route-Map Sequence를 확인한다.

ACL을 Interface에 직접 적용하면 Packet을 허용하거나 차단하지만, PBR의 `match`에 사용하면 Packet을 선택하는 용도로 사용한다.

### PBR에서 Route-Map의 `deny`

PBR에서 Route-Map의 `deny`에 일치한 Packet은 차단되지 않는다.

PBR이 적용되지 않으면 해당 Packet은 Destination IP Address를 기준으로 일반 Routing Table을 사용하여 전달한다.  

Route-Map에서 따로 지정하지 않은 Packet은 Implicit Deny에 의해 PBR이 적용되지 않고 기존 Routing Table의 경로로 전달된다.
```
R1(config)# access-list 10 permit 172.16.20.0 0.0.0.255

R1(config)# route-map PBR permit 100
R1(config-route-map)# match ip address 10 
R1(config-route-map)# set ip next-hop 198.51.100.1
```
- 172.16.20.0/24에서 들어온 Packet은 Sequence 100과 일치하므로 Next-Hop 198.51.100.1을 사용한다.
- 반면 172.16.10.0/24에서 들어온 Packet은 어느 Sequence에도 일치하지 않으므로 Implicit Deny에 의해 PBR이 적용되지 않고 일반 Routing Table을 사용한다.

### PBR 적용 방향

PBR은 Packet이 Router로 들어오는 Inbound Interface에 적용한다.

ACL과 Route-Map만 생성하면 PBR 정책이 동작하지 않으며, Packet이 들어오는 Interface에 Route-Map을 적용해야 한다.
```
R1(config)# access-list 10 permit 172.16.20.0 0.0.0.255

R1(config)# route-map PBR permit 100
R1(config-route-map)# match ip address 10
R1(config-route-map)# set ip next-hop 198.51.100.1
```
```
R1(config)# interface gi0/0
R1(config-if)# ip policy route-map PBR
```
- `Gi0/0`으로 들어오는 Packet에 `PBR` Route-Map을 적용한다.

PBR 조건에 일치한 Packet도 지정한 Next-Hop이 Local Routing Table에서 도달 가능한 상태여야 PBR 경로로 전달된다.

하지만 PBR에서 지정한 Next-Hop에 도달할 수 없으면 일반 Routing Table의 경로를 사용한다.

### Local PBR

Interface로 들어오는 Packet이 아니라 Router 자신이 생성한 Packet에 PBR을 적용하려면 Local PBR을 사용한다.

```
R1(config)# ip access-list extended LOCAL-TRAFFIC
R1(config-ext-nacl)# permit icmp host 1.1.1.1 host 172.16.100.10

R1(config)# route-map LOCAL-PBR permit 10
R1(config-route-map)# match ip address LOCAL-TRAFFIC
R1(config-route-map)# set ip next-hop 10.0.12.2

R1(config)# ip local policy route-map LOCAL-PBR
```
- ACL: R1이 직접 생성한 Ping을 선택한다.
- Route-Map: 조건에 일치한 Packet의 Next-Hop을 `10.0.12.2`로 지정한다.
- `ip local policy`: Router가 직접 생성한 Packet에 Route-Map을 적용한다.

---

