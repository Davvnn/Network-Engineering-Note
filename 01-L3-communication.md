# L3 통신 / Routing

## 개념

Layer 3 통신은 서로 다른 네트워크에 있는 장비들이 IP Address를 기반으로 통신하는 방식이다. Layer 3에서는 데이터를 Packet 단위로 전달한다.

단말은 Destination IP Address가 자신의 네트워크에 속하지 않으면 Packet을 Default Gateway로 전송한다. 라우터는 IP Header의 Destination IP Address를 확인하고 Routing Table에서 목적지 네트워크를 Lookup한다. 목적지 경로가 있으면 Next-Hop과 Exit Interface를 확인하고 Packet을 Forwarding한다.

---

## 동작 원리

1. 단말이 Payload에 IP Header를 붙여 Packet을 만든다. IP Header 안에는 Source IP Address와 Destination IP Address 등의 정보가 포함되어 있다.
2. 단말은 Packet을 Ethernet Frame으로 Encapsulation하여 Default Gateway인 라우터에게 전송한다.
3. 라우터는 Frame을 Decapsulation하여 Ethernet Header를 제거하고, IP Header의 Destination IP Address를 확인한다.
4. 라우터는 Destination IP Address를 Routing Table에서 Lookup한다.
5. 목적지와 일치하는 경로가 있으면 Next-Hop과 Exit Interface를 확인하고, Packet을 새로운 Ethernet Frame으로 Encapsulation한다. Source MAC Address에는 현재 라우터의 Exit Interface MAC Address를, Destination MAC Address에는 Next-Hop 장비의 MAC Address를 지정한다.
6. 라우터는 Exit Interface를 통해 Frame을 Forwarding한다.
7. Next-Hop 라우터는 Frame을 다시 Decapsulation하고 Destination IP Address를 확인한다. 목적지 네트워크가 자신에게 Directly Connected되어 있으면 목적지 단말의 MAC Address를 확인한 후, 새로운 Ethernet Frame으로 Encapsulation하여 목적지 단말로 Forwarding한다. 목적지 경로가 다른 라우터를 통해 연결되어 있으면 같은 과정을 반복한다.
8. 목적지 단말은 Frame을 수신하고 Decapsulation하여 Packet의 Data를 확인한다.

---

## 예시 및 구성도

![](images/)

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

