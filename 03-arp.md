# ARP / Gratuitous ARP

## 개념

ARP(Address Resolution Protocol)는 같은 네트워크에서 통신할 때 IPv4 Address에 매핑된 MAC Address를 확인하는 프로토콜이다.

단말은 목적지와 통신하기 전에 자신의 ARP Table에서 목적지 IP Address에 해당하는 MAC Address를 확인한다. ARP 정보가 없으면 ARP Request를 같은 네트워크에 Broadcast로 전송하고, 해당 IP Address를 사용하는 단말은 자신의 MAC Address를 ARP Reply에 포함하여 Unicast로 전송한다.

목적지가 다른 네트워크에 있다면 단말은 Default Gateway로 ARP Request를 보낸다. Default Gateway는 자신의 MAC Address를 ARP Reply에 포함하여 단말로 Unicast 전송한다.

Gratuitous ARP는 단말이 ARP Request를 받지 않은 상태에서 자신의 IP Address와 MAC Address 정보를 같은 네트워크에 광고하는 기능이다.

---

## 동작 원리

내용 작성

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

