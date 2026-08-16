# ARP / Gratuitous ARP

## 개념

ARP(Address Resolution Protocol)는 같은 네트워크에서 통신할 때 IPv4 Address에 매핑된 MAC Address를 확인하는 프로토콜이다.

단말은 목적지와 통신하기 전에 자신의 ARP Table에서 목적지 IP Address에 해당하는 MAC Address를 확인한다. ARP 정보가 없으면 ARP Request를 같은 네트워크에 Broadcast로 전송하고, 해당 IP Address를 사용하는 단말은 자신의 MAC Address를 ARP Reply에 포함하여 Unicast로 전송한다.

목적지가 다른 네트워크에 있다면 단말은 Default Gateway로 ARP Request를 보낸다. Default Gateway는 자신의 MAC Address를 ARP Reply에 포함하여 단말로 Unicast 전송한다.

Gratuitous ARP는 단말이 ARP Request를 받지 않은 상태에서 자신의 IP Address와 MAC Address 정보를 같은 네트워크에 광고하는 기능이다.

---

## 동작 원리

### ARP

1\. 단말은 목적지 IP Address가 같은 네트워크에 있는지 확인한다.

2\. 같은 네트워크에 있다면 목적지 IP Address에 매핑된 MAC Address를 ARP Table에서 확인한다. 다른 네트워크에 있다면 Default Gateway의 IP Address에 매핑된 MAC Address를 확인한다.

3\. ARP Table에 필요한 MAC Address 정보가 없으면 해당 IP Address를 Target IP Address로 지정한 ARP Request를 Broadcast로 전송한다.

4\. 스위치는 ARP Request를 수신한 인터페이스를 제외하고, 같은 VLAN에 속한 모든 인터페이스로 Flooding한다.

5\. Target IP Address를 사용하는 장비는 자신의 MAC Address를 포함한 ARP Reply를 요청한 단말에게 Unicast로 전송한다.

6\. 요청한 단말은 전달받은 IP Address와 MAC Address 정보를 ARP Table에 매핑한다.

7\. 이후 단말은 응답받은 MAC Address를 Destination MAC Address로 지정하여 Ethernet Frame을 생성하고 전송한다.

### Gratuitous ARP

1\. 단말은 자신의 IP Address와 MAC Address가 포함된 Gratuitous ARP를 Broadcast로 전송한다.

2\. 같은 네트워크에 있는 장비들은 Gratuitous ARP를 확인하고 ARP Table을 갱신할 수 있다.

3\. 이중화 환경에서는 Active 장비가 변경되었을 때 새로운 Active 장비의 MAC Address를 네트워크에 알리는 용도로도 사용한다.

---

## 예시 및 구성도

### 목저기가 같은 네트워크인 경우 

PC1
- IP Address: 192.168.1.10/24
- MAC Address: AAAA.AAAA.AAAA

PC2
- IP Address: 192.168.1.20/24
- MAC Address: BBBB.BBBB.BBBB

![](images/03-arp-same-network.png)
![](images/03-arp-format.png)

1\. PC1은 PC2가 자기와 같은 네트워크인것을 확인한다.

2\. PC1은 자신의 ARP Table에서 PC2의 IP Address인 `192.168.1.20`에 매핑된 MAC Address를 확인한다.

3\. ARP 정보가 없으면 `192.168.1.20`을 Target IP Address로 지정한 ARP Request를 Broadcast로 전송한다.
- Destination MAC Address에는 Broadcast 주소인 `FFFF.FFFF.FFFF`를 지정한다.

4\. SW1은 PC1의 ARP Request를 수신한 인터페이스를 제외하고, 같은 VLAN에 속한 모든 인터페이스로 Flooding한다.

5. PC2는 Ethernet Frame을 Decapsulation하고, Payload에 들어 있는 ARP 메시지를 확인한다.
- Target IP Address가 자신의 IP Address와 일치하는지 확인한다.

6\. PC2는 자신의 MAC Address인 `BBBB.BBBB.BBBB`를 포함한 ARP Reply를 PC1에게 Unicast로 전송한다.

7\. PC1은 전달받은 IP Address와 MAC Address 정보를 ARP Table에 매핑한다.
- `192.168.1.20` → `BBBB.BBBB.BBBB`

8\. PC1은 PC2의 MAC Address를 Destination MAC Address로 지정하여 Ethernet Frame을 생성하고 전송한다.

### PC1과 PC2가 다른 네트워크인 경우

PC1
- IP Address: 192.168.1.10/24
- MAC Address: AAAA.AAAA.AAAA
- Default Gateway: 192.168.1.1

R1
- Gi0/1 IP Address: 192.168.1.1/24
- Gi0/1 MAC Address: CCCC.CCCC.CCCC

R2
- Gi0/1 IP Address: 192.168.2.1/24
- Gi0/1 MAC Address: DDDD.DDDD.DDDD

PC2
- IP Address: 192.168.2.10/24
- MAC Address: BBBB.BBBB.BBBB
- Default Gateway: 192.168.2.1

![](images/03-arp-different-network.png)

1\. PC1은 PC2의 IP Address가 자신의 네트워크에 속하지 않는 것을 확인한다.

2\. PC1은 Default Gateway인 `192.168.1.1`에 매핑된 MAC Address를 ARP Table에서 확인한다.

3\. ARP 정보가 없으면 `192.168.1.1`을 Target IP Address로 지정한 ARP Request를 Broadcast로 전송한다.

4\. SW1은 ARP Request를 수신한 인터페이스를 제외하고, 같은 VLAN에 속한 모든 인터페이스로 Flooding한다.

5\. R1은 Ethernet Frame의 Payload에 들어 있는 ARP 메시지를 확인한다.
- Target IP Address가 자신의 `Gi0/1` IP Address와 일치하는지 확인한다.

6\. R1은 `Gi0/1`의 MAC Address인 `CCCC.CCCC.CCCC`를 포함한 ARP Reply를 PC1에게 Unicast로 전송한다.

7\. PC1은 전달받은 IP Address와 MAC Address 정보를 ARP Table에 매핑한다.
- `192.168.1.1` → `CCCC.CCCC.CCCC`

8\. PC1은 R1 `Gi0/1`의 MAC Address를 Destination MAC Address로 지정하여 Ethernet Frame을 생성하고 전송한다.

9\. R1은 Ethernet Frame을 Decapsulation하고, Destination IP Address가 `192.168.2.10`인 것을 확인한다.

10\. R1은 `192.168.2.0/24`가 `Gi0/2`에 Directly Connected되어 있는 것을 확인한다.

11\. R1은 ARP Table에서 PC2의 IP Address인 `192.168.2.10`에 매핑된 MAC Address를 확인한다.

12\. ARP 정보가 없으면 R1은 `192.168.2.10`을 Target IP Address로 지정한 ARP Request를 `Gi0/2`를 통해 Broadcast로 전송한다.

13\. SW2는 ARP Request를 수신한 인터페이스를 제외하고, 같은 VLAN에 속한 모든 인터페이스로 Flooding한다.

14\. PC2는 Ethernet Frame의 Payload에 들어 있는 ARP 메시지를 확인한다.
- Target IP Address가 자신의 IP Address와 일치하는지 확인한다.

15\. PC2는 자신의 MAC Address인 `BBBB.BBBB.BBBB`를 포함한 ARP Reply를 R1에게 Unicast로 전송한다.

16\. R1은 전달받은 IP Address와 MAC Address 정보를 ARP Table에 매핑한다.
- `192.168.2.10` → `BBBB.BBBB.BBBB`

17\. R1은 PC2의 MAC Address를 Destination MAC Address로 지정하여 새로운 Ethernet Frame을 생성하고 PC2에게 전송한다.

18\. PC2는 R1이 전달한 Ethernet Frame을 수신하고 Decapsulation하여 PC1이 전송한 Packet의 Data를 확인한다.


---

## 명령어

내용 작성

---

## Troubleshooting

내용 작성

---

## 실무 질문

Q1. 스위치는 목적지 MAC Address를 모르면 어떻게 동작하는가?
- L2 Header에서 Destination MAC을 확인한다.

