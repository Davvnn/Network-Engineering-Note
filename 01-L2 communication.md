# L2 통신 / MAC Address Table

## 개념

Layer 2 통신은 같은 네트워크(LAN) 안에 있는 장비들이 MAC Address를 기반으로 통신하는 방식이다. Layer 2에서는 데이터를 Frame 단위로 전달한다.

스위치는 전달받은 Frame의 Ethernet Header에서 Source MAC Address를 확인하고, 해당 MAC Address와 Frame이 들어온 인터페이스를 MAC Address Table에 매핑한다. 이후 Destination MAC Address를 확인하고, MAC Address Table에 해당 주소가 있으면 매핑된 인터페이스로 Frame을 Forwarding한다.

---

## 동작 원리

1. 단말이 Frame을 전송한다.  
2. 스위치가 인터페이스를 통해 Frame을 수신한다.  
3. 스위치는 Ethernet Header의 Source MAC Address와 Destination MAC Address를 확인한다.  
4. Source MAC Address는 Frame이 들어온 인터페이스와 함께 MAC Address Table에 매핑하여 학습한다.  
5. Destination MAC Address는 자신의 MAC Address Table에서 Lookup한다.  
6. Destination MAC Address가 있으면 매핑된 인터페이스로 Frame을 Forwarding한다.  
7. 만약 Destination MAC Address가 없으면 Frame이 들어온 인터페이스를 제외하고, 같은 VLAN에 속한 모든 인터페이스로 Frame을 Flooding한다.  
8. 목적지 단말이 응답하면 스위치는 응답 Frame의 Source MAC Address도 같은 방식으로 학습한다.  
9. 이후에는 학습된 정보를 이용하여 목적지 단말이 연결된 인터페이스로 Frame을 Forwarding한다.

---

## 예시 및 구성도
PC1
- IP Address: 192.168.1.10
- MAC Address: AAAA.AAAA.AAAA

PC2
- IP Address: 192.168.1.20
- MAC Address: BBBB.BBBB.BBBB

![](images/git_example.png)

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
