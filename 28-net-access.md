# Network Access Security

## 개념

### 802.1X

802.1X는 Switch Port에 연결된 사용자나 장비를 인증한 후 Network 접근을 허용하는 인증 방식이다.

인증에 성공하기 전에는 일반 Traffic을 차단하고 인증에 필요한 EAPOL Traffic만 허용한다.
- `EAPOL Traffic`은 PC와 Switch가 802.1x 인증 정보를 주고 받을때 사용하는 Traffic이다.

Supplicant
- Supplicant는 Network에 접속하기 위해 인증을 요청하는 Client이다.

Authenticator
- Authenticator는 Client가 연결된 Switch 또는 Wireless Access Point이다.
- Client의 인증 정보를 직접 확인하지 않고 RADIUS Server로 전달한다.

Authentication Server
- Authentication Server는 사용자의 계정이나 인증서를 확인하고 인증 결과를 전달한다.
- 일반적으로 RADIUS Protocol을 사용하는 Cisco ISE, Windows NPS 및 FreeRADIUS 등이 있다.
- 802.1X 인증에는 TACACS+가 아니라 RADIUS를 사용한다.
```
RADIUS Authentication: UDP Port 1812
RADIUS Accounting: UDP Port 1813
```

### EAP와 EAPOL

EAP(Extensible Authentication Protocol)는 802.1X 인증 과정에서 사용자나 장비의 인증 정보를 주고받을 때 사용하는 Protocol이다.

Client와 Switch 사이에서는 EAPOL(EAP over LAN)을 사용한다.

Client가 Switch Port에 연결되면 EAP 인증 정보를 EAPOL에 담아 Switch로 전달한다. Switch는 받은 EAP 인증 정보를 RADIUS Message에 담아 RADIUS Server로 전달한다  

### EAP-TLS

EAP-TLS는 Client와 Authentication Server가 Certificate를 사용하여 서로를 인증하는 방식이다.

Password만 사용하는 방식보다 안전하지만 Client와 Server에 Certificate를 배포하고 관리해야 한다.

기업의 업무용 PC 인증에 많이 사용한다.

### PEAP

PEAP는 Server Certificate로 암호화 Tunnel을 만든 후 Username과 Password를 확인하는 방식이다.

EAP-TLS보다 구성이 간단하지만 사용자 계정과 Password를 관리해야 한다.

### MAB

MAB(MAC Authentication Bypass)는 802.1X 인증을 사용할 수 없는 장비를 MAC Address로 확인하여 Network 접근을 허용하는 방식이다.

관리자는 MAB 인증을 허용할 단말의 MAC Address를 RADIUS Server에 미리 등록해야 한다.
- Printer
- IP Phone
- Camera
- IoT 장비

일부 Printer, IP Phone 및 Camera는 802.1X 인증 기능이 없어서 Certificate를 Switch에 보낼 수 없다.

MAC Address는 위조할 수 있으므로 MAB는 802.1X보다 보안 수준이 낮다.

### NAC

NAC(Network Access Control)는 Network에 접속하는 사용자와 장비를 확인하고, 보안 정책에 따라 접근 권한을 결정한다. 

802.1X는 인증을 Client를 인증하고, NAC는 인증 결과를 이용하여 실제 Network 접근 정책을 적용한다.

NAC는 먼저 802.1X 또는 MAB를 통해 사용자와 장비를 인증하고, 장비의 보안 상태를 확인한 후 결과에 따라 VLAN이나 ACL을 적용한다.  
- Cisco에서는 ISE(Identity Services Engine)를 NAC Server로 사용할 수 있다.

### Port Security

Port Security는 Switch Port에서 허용할 MAC Address와 개수를 제한하는 기능이다.

허용하지 않은 MAC Address가 연결되면 Packet을 차단하거나 Interface를 Error-Disabled 상태로 변경할 수 있다.
```
SW1(config)# interface gi1/0/10
SW1(config-if)# switchport mode access
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 1
SW1(config-if)# switchport port-security mac-address aaaa.bbbb.cccc
SW1(config-if)# switchport port-security violation shutdown
```

### Static Secure MAC Address

관리자가 허용할 MAC Address를 직접 설정하는 방식이다.
```
SW1(config-if)# switchport port-security mac-address aaaa.bbbb.cccc
```

#### Dynamic Secure MAC Address

Switch가 연결된 장비의 MAC Address를 자동으로 학습한다.

설정된 Maximum 개수까지 자동으로 학습하며, 허용 개수를 초과한 다른 MAC Address가 연결되면 Violation이 발생한다.

Dynamic Secure MAC Address는 Running Configuration에 저장되지 않으므로 Switch를 Reload하면 사라진다.

### Sticky Secure MAC Address

Switch가 MAC Address를 자동으로 학습하고 Running Configuration에 추가하는 방식이다.
```
SW1(config-if)# switchport port-security mac-address sticky
```
Switch가 Reload된 후에도 사용하려면 설정을 Startup Configuration에 저장해야 한다.

### Port Security Violation Mode

Protect
- 허용되지 않은 MAC Address의 Packet을 차단하지만 Log를 남기지 않는다.
- Interface는 계속 Up 상태를 유지한다.

Restrict
- 허용되지 않은 MAC Address의 Packet을 차단하고 Syslog, SNMP Trap 및 Violation Counter를 남긴다.
- Interface는 계속 Up 상태를 유지한다.

Shutdown
- Violation이 발생하면 Interface를 Error-Disabled 상태로 변경한다.
- Port Security의 기본 Violation Mode이다.

### 802.1X와 Port Security의 차이

802.1X
- 사용자 계정이나 PC에 미리 설치한 Certificate를 이용하여 사용자와 장비를 인증한다.
- Switch는 Client의 계정이나 Certificate 인증 정보를 RADIUS Server로 전달하고, RADIUS Server가 Network 접근 허용 여부를 결정한다.  
- 인증 결과에 따라 사용자나 장비별로 VLAN과 ACL을 다르게 적용할 수 있다.
- 많은 사용자와 장비의 접근 권한을 중앙에서 관리할 수 있어 기업 환경에 적합하다. 

Port Security
- Switch가 연결된 장비의 MAC Address를 확인한다.
- Switch 자체에서 MAC Address를 확인하므로 별도의 인증 Server가 필요하지 않다.
- 프린터나 고정 PC처럼 각 Port에 연결되는 장비가 정해져 있는 환경이나 소규모 환경에서 사용하기 적합하다.
