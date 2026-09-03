# Network Management / Monitoring

## 개념

Network Management와 Monitoring은 Network 장비의 상태, 장애, 성능 및 Traffic 사용량을 확인하고 관리하는 기능이다.
- NTP: 장비들의 시간을 동기화한다.
- SNMP: 장비의 CPU, Memory 및 Interface 상태를 수집한다.
- Syslog: 장비에서 발생한 Event와 오류 Message를 기록한다.
- NetFlow/IPFIX: 어떤 장비가 어디와 얼마나 통신했는지 수집한다.

### NTP

NTP(Network Time Protocol)는 Network 장비의 시간을 NTP Server와 동기화하는 Protocol이다.

장비들의 시간이 일치해야 Syslog, SNMP Trap 및 장애 발생 시간을 정확하게 비교할 수 있다.

#### Stratum

Stratum은 NTP Server가 기준 시간과 얼마나 가까운지 나타내는 값이다.
- Stratum `0`: GPS나 원자시계와 같은 기준 시간 장비이다.
- Stratum `1`: Stratum 0과 직접 연된결 NTP Server이다.
- Stratum `2`: Stratum 1 NTP Server로부터 시간을 직접 동기화하는 NTP Server이다.
- Stratum `16`: 시간을 동기화하지 못한 상태이다.

Stratum 값이 낮을수록 기준 시간에 가까운 Server이다.

### SNMP

SNMP(Simple Network Management Protocol)는 NMS가 Network 장비의 상태와 성능 정보를 수집하는 Protocol이다.
```
NMS Manager ↔ SNMP Agent → MIB/OID
```
- NMS Manager: Network 장비의 정보를 수집하여 관리자가 확인할 수 있도록 Web 화면에 표시하는 Monitoring Server이다.
- SNMP Agent: Router나 Switch에서 장비 정보를 NMS에 제공하는 기능이다.
- MIB: SNMP로 확인할 수 있는 장비 정보들을 정리한 데이터 구조이다.
- OID: MIB에서 확인하려는 장비 정보를 찾을 때 사용하는 고유 번호이다.


#### SNMP Trap과 Inform

Trap과 Inform은 장애나 Interface Down 같은 Event가 발생하면 SNMP Agent가 NMS Manager로 알림을 보내는 방식이다.  

Trap은 NMS의 응답을 기다리지 않고, Message가 전달되지 않아도 다시 확인하지 않는다.

Inform은 NMS가 Message를 받았다는 응답을 보내며, 응답이 없으면 장비가 다시 전송할 수 있다.

#### SNMP Version

SNMPv1은 오래된 Version으로 인증과 보안 기능이 약하다.

SNMPv2c는 Community String을 사용하며 성능이 개선되었지만 인증 정보와 수집 내용이 암호화되지 않는다.

SNMPv3는 사용자 인증과 암호화를 지원하므로 운영 환경에서 사용하는 것이 좋다.

관리자가 SNMPv3 사용자를 생성할 때 사용할 보안 수준을 설정한다  
- `noAuthNoPriv`: 인증과 암호화를 사용하지 않는다.
- `authNoPriv`: 사용자 인증만 사용한다.
- `authPriv`: 사용자 인증과 암호화를 모두 사용한다.

### Syslog

Syslog는 Router, Switch 및 Firewall에서 발생한 Event와 오류 Message를 기록하고 Syslog Server로 전달하는 기능이다.
```
Sep 2 10:30:15: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
```
Syslog Message에는 발생 시간, Facility, Severity, Message 이름 및 내용이 포함된다.

#### Syslog Severity

```
0 Emergencies: 장비를 사용할 수 없는 심각한 상태
1 Alerts: 즉시 조치가 필요한 상태
2 Critical: 중요한 기능에 문제가 발생한 상태
3 Errors: 오류가 발생한 상태
4 Warnings: 장애가 발생할 가능성이 있는 경고 상태
5 Notifications: 정상적이지만 확인할 필요가 있는 상태
6 Informational: 일반적인 동작 정보
7 Debugging: 장애 분석을 위한 상세 정보
```
Severity Number가 낮을수록 더 심각한 Message이다.


### NetFlow

NetFlow는 Cisco가 개발한 Traffic Monitoring 기술로, Network를 통과하는 Traffic 정보를 Flow 단위로 수집한다.
```
Source/Destination IP Address
Source/Destination Port
Protocol
Interface
Packet/Byte 수
통신 시작 및 종료 시간
```
- 출발지와 목적지 등이 같은 여러 Packet을 하나의 Flow라고 한다.  

NetFlow는 일반적으로 Packet의 전체 내용을 저장하지 않고, 누가 어디로 얼마나 통신했는지에 대한 정보만 수집한다.

#### Flexible NetFlow 구성 요소

Flow Record: 어떤 Traffic 정보를 수집할지 지정한다. 

Flow Monitor: Flow Record에 따라 Traffic 정보를 수집하고 Cache에 저장한다. 

Flow Exporter: 수집한 정보를 어느 Flow Collector로 전송할지 지정한다. 

Flow Collector: 장비가 전송한 Flow 정보를 수신하고 분석하는 Server이다.

```
192.168.10.100 → 198.51.100.10:443
                  ↓
Gi0/0의 Flow Monitor가 Traffic 확인
                  ↓
Flow Record가 IP Address, Port 및 전송량 수집
                  ↓
Flow Exporter가 10.0.0.50로 정보 전송
                  ↓
Flow Collector가 통신 내역을 분석하고 표시
```

### IPFIX

IPFIX(IP Flow Information Export)는 NetFlow Version 9를 기반으로 만들어진 IETF 표준 Flow Export Protocol이다.

IPFIX는 NetFlow와 같은 목적으로 사용하며, 여러 제조사의 장비에서 사용할 수 있는 표준 Protocol이다.


---


