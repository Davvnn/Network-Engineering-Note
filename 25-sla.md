# IP SLA / Object Tracking

## 개념

### IP SLA

IP SLA(IP Service Level Agreement)는 Cisco 장비가 지정한 목적지로 Test Packet을 주기적으로 전송하여 Network의 연결 상태와 성능을 측정하는 Cisco 전용 기능이다.

단순히 Interface의 Up/Down 상태만 확인하는 것이 아니라 실제 목적지까지 Packet이 정상적으로 전달되는지 확인할 수 있다.

```
R1(config)# ip sla 1
R1(config-ip-sla)# icmp-echo 8.8.8.8 source-interface gi0/0
R1(config-ip-sla-echo)# frequency 5
R1(config-ip-sla-echo)# timeout 1000
R1(config-ip-sla-echo)# exit
```
- `ip sla` 1: IP SLA Operation Number 1을 생성한다.
- `icmp-echo 8.8.8.8`: 8.8.8.8로 ICMP Echo Packet을 전송한다.
- `source-interface gi0/0`: Test Packet을 Gi0/0에서 전송한다.
- `frequency 5`: 5초마다 Test Packet을 전송한다.
- `timeout 1000`: 1000ms 동안 응답이 없으면 실패로 판단한다.

IP SLA로 다음 정보를 확인할 수 있다.
- 목적지 도달 가능 여부
- RTT(Round Trip Time)
- Delay
- Jitter
- Packet Loss
- DNS 및 TCP Service 응답 시간

### IP SLA Operation 활성화

생성한 IP SLA Operation은 `ip sla schedule` 명령어를 사용하여 활성화해야 한다.
```
R1(config)# ip sla schedule 1 life forever start-time now
```
- `ip sla schedule 1`: IP SLA Operation `1`의 실행 일정을 설정한다.
- `life forever`: IP SLA Operation을 계속 실행한다.
- `start-time now`: IP SLA Operation을 즉시 시작한다.

`ip sla schedule`을 설정하지 않으면 IP SLA Operation은 생성만 되고 Test Packet을 전송하지 않는다.


### Object Tracking

Object Tracking은 Interface, Route 또는 IP SLA의 상태를 `Up`과 `Down`으로 관리하는 기능이다.
- `Up`: 연결된 경로나 장비를 정상적으로 사용할 수 있다는 신호를 전달한다.
- `Down`: 연결된 경로나 장비에 장애가 발생했다는 신호를 전달한다.

Tracking 결과를 Static Route, HSRP, VRRP 및 PBR 등의 기능과 연결하여 Network 상태에 따라 동작을 자동으로 변경할 수 있다.
```
R1(config)# track 1 ip sla 1 reachability
R1(config-track)# delay down 10 up 20
```
- 첫 번째 `1`: Object Tracking을 구분하는 Track Number이다.
- 두 번째 `1`: 상태를 확인할 IP SLA Operation Number이다.
- `reachability`: IP SLA 목적지까지 통신 가능한지 추적한다.
- `delay down 10`: 장애가 `10초` 동안 지속되면 Track을 Down으로 변경한다.
- `delay up 20`: 통신이 복구된 후 `20초` 동안 정상 상태가 유지되면 Track을 Up으로 변경한다.

`track`을 설정하지 않으면 IP SLA는 Test Packet을 전송하고 측정 결과만 저장하며, 해당 결과를 Static Route나 FHRP에서 사용할 수 없다.

또한 Object Tracking을 생성하더라도 Static Route나 FHRP에 Track을 연결하지 않으면 상태만 확인할 수 있으며 실제 Network 동작은 변경되지 않는다.

### Interface Tracking

Interface Tracking은 자신의 Interface 상태만 확인한다.
```
R1(config)# track 1 interface gi0/0 line-protocol
```
Interface가 물리적으로 연결되어 있어도 ISP 내부나 목적지까지의 경로에 장애가 발생할 수 있다. 이 경우 Interface는 `Up`이지만 실제 외부 통신은 불가능할 수 있다.

### IP SLA Tracking

IP SLA Tracking은 Test Packet을 목적지까지 전송하여 실제 통신 가능 여부를 확인한다.

IP SLA는 목적에 따라 여러 Operation을 사용할 수 있다.

#### ICMP Echo

ICMP Echo Request와 Reply를 사용하여 목적지 도달 가능 여부와 RTT를 측정한다.

일반적인 경로 이중화에서 가장 많이 사용한다.
```
R1(config)# ip sla 1
R1(config-ip-sla)# icmp-echo 8.8.8.8 source-interface gi0/0
R1(config-ip-sla-echo)# frequency 5
R1(config-ip-sla-echo)# timeout 1000
R1(config-ip-sla-echo)# threshold 500
R1(config-ip-sla-echo)# exit

R1(config)# ip sla schedule 1 life forever start-time now
```
`Gi0/0`을 통해 `8.8.8.8`로 `5초`마다 ICMP Echo Packet을 전송한다. 응답 시간이 `500ms`를 초과하면 느린 응답으로 기록하고, `1초` 동안 응답이 없으면 실패로 판단한다.

#### UDP Jitter

UDP Packet을 사용하여 Delay, Jitter 및 Packet Loss를 측정한다.

Voice나 Video Traffic의 품질을 확인할 때 사용할 수 있다.
```
R1(config)# ip sla 2
R1(config-ip-sla)# udp-jitter 192.0.2.10 5000 num-packets 10 interval 20
R1(config-ip-sla-jitter)# frequency 30
R1(config-ip-sla-jitter)# timeout 5000
R1(config-ip-sla-jitter)# exit

R1(config)# ip sla schedule 2 life forever start-time now
```
`30`초마다 `192.0.2.10`의 UDP Port `5000`으로 UDP Packet `10개`를 `20ms` 간격으로 전송하여 응답 시간, Jitter 및 Packet Loss를 측정한다. `5초` 안에 응답이 없으면 실패로 판단한다.

#### TCP Connect

지정한 TCP Port로 연결을 시도하여 Server의 Service가 정상적으로 동작하는지 확인한다.
```
R1(config)# ip sla 3
R1(config-ip-sla)# tcp-connect 192.0.2.20 443 control disable
R1(config-ip-sla-tcp)# frequency 30
R1(config-ip-sla-tcp)# timeout 3000
R1(config-ip-sla-tcp)# exit

R1(config)# ip sla schedule 3 life forever start-time now```
```
- `control disable`: 지정한 TCP Port에 바로 접속하여 Service가 정상인지 확인한다.

`30초`마다 Web Server `192.0.2.20`의 TCP Port `443`으로 연결을 시도하여 HTTPS Service가 정상적으로 동작하는지 확인한다. 만약 `3초` 동안 연결되지 않으면 실패로 판단한다.   


#### DNS

DNS Query를 전송하여 DNS Server의 응답 여부와 응답 시간을 측정한다.
```
R1(config)# ip sla 4
R1(config-ip-sla)# dns www.google.com name-server 8.8.8.8
R1(config-ip-sla-dns)# frequency 30
R1(config-ip-sla-dns)# timeout 3000
R1(config-ip-sla-dns)# threshold 1000
R1(config-ip-sla-dns)# exit
```
`30초`마다 DNS Server `8.8.8.8`에 `www.google.com`의 `IP Address`를 물어본다. `3`초 안에 응답이 없으면 실패로 판단하고, 응답에 `1초`가 넘게 걸리는지도 확인한다.

### Delay Up과 Delay Down

Object Tracking의 상태가 너무 빠르게 변경되는 것을 방지하기 위해 Delay를 설정할 수 있다.
```
delay down 10 up 20
```
- `delay down 10`: 장애가 감지되어도 `10초` 동안 기다린 후 Track을 Down으로 변경한다.
- `delay up 20`: 통신이 복구된 후 20초 동안 정상 상태가 유지되면 Track을 Up으로 변경한다.

---

## 동작 원리

### IP SLA와 Object Tracking 동작 과정

1\. R1은 IP SLA에 설정된 목적지로 Test Packet을 주기적으로 전송한다.

2\. 목적지에서 정상적으로 응답하면 IP SLA Operation은 성공 상태가 된다.

3\. Object Tracking은 IP SLA의 결과를 확인하고 `Up` 상태를 유지한다.

4\. 만약 일정 시간 동안 목적지로부터 Test Packet에 대한 응답을 받지 못하면 IP SLA Operation이 실패한다.

5\. Object Tracking은 `delay down` 시간이 지난 후 `Down` 상태로 변경된다.

6\. Track과 연결된 Primary Static Route가 Routing Table에서 제거된다.

7\. AD 값이 높은 Floating Static Route가 Routing Table에 등록되어 Backup 경로로 Traffic을 전달한다.

8\. Primary 경로가 복구되면 IP SLA가 다시 응답을 받는다.

9\. `delay up` 시간이 지난 후 Track이 `Up`으로 변경되고 Primary Route가 다시 등록된다.

---

