# BGP 기초

## 개념

### BGP

BGP(Border Gateway Protocol)는 서로 다른 Autonomous System 사이에서 Network 경로를 교환하기 위해 사용하는 Path Vector Routing Protocol이다.

OSPF와 EIGRP는 하나의 AS 내부에서 최적 경로를 계산하는 IGP이고, BGP는 AS 사이에서 경로를 교환하고 정책을 기준으로 Best Path를 선택하는 EGP이다.

BGP는 인터넷 규모의 많은 Route를 처리해야 하므로 빠른 수렴보다 안정성, 확장성 및 정책 제어에 중점을 둔다.

BGP는 다음과 같은 특징이 있다.
- BGP Neighbor를 자동으로 발견하지 않고 관리자가 직접 설정한다.
- TCP `179` Port를 사용하여 신뢰성 있게 Route를 교환한다.
- 여러 Path Attribute를 순서대로 비교하여 Best Path를 선택한다.
- 처음에는 전체 Route를 교환하고, 이후에는 변경된 Route만 `Partial Update`한다. 
- OSPF와 EIGRP의 `network` 명령은 Interface에서 Protocol을 활성화하고, BGP의 `network` 명령은 Routing Table의 특정 Route를 BGP에 등록한다.  
