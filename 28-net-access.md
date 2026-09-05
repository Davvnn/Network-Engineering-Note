# Network Access Security

## 개념

### 802.1X

802.1X는 Switch Port에 연결된 사용자나 장비를 인증한 후 Network 접근을 허용하는 인증 방식이다.

인증에 성공하기 전에는 일반 Traffic을 차단하고 인증에 필요한 EAPOL Traffic만 허용한다.
- `EAPOL Traffic`은 PC와 Switch가 802.1x 인증 정보를 주고 받을때 사용하는 Traffic이다.


802.1X는 다음 세 가지 구성 요소를 사용한다.

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
다음과 같은 장비에 사용할 수 있다.

관리자는 MAB 인증을 허용할 단말의 MAC Address를 RADIUS Server에 미리 등록해야 한다.
- Printer
- IP Phone
- Camera
- IoT 장비

MAC Address는 위조할 수 있으므로 MAB는 802.1X보다 보안 수준이 낮다.

Switch는 Client의 MAC Address를 RADIUS Server에 전달하고, 등록된 장비인지 확인한다.
