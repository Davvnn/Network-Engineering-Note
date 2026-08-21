# STP Toolkit

## 개념

STP Toolkit은 Layer 2 Loop를 방지하며, 링크 장애 발생 시 Network가 빠르게 수렴할 수 있게 해주는 기능이다.

### PortFast

PortFast는 해당 Port를 Listening과 Learning 상태를 거치지 않고 즉시 Forwarding 상태로 전환해주는 기능이다.

PortFast는 PC, Server, Printer, AP와 같은 단말이 연결된 Edge Port에 사용한다. PortFast를 설정해도 STP가 비활성화되는 것은 아니며, BPDU는 계속 수신한다.

만약 Switch가 연결된 Port에 PortFast를 잘못 설정하면 Layer 2 Loop가 발생할 수 있으므로, 해당 Port가 어디에 연결되어 있는지 주의해야 한다.  

### BPDU Guard

BPDU Guard는 연결된 Port에서 BPDU가 수신되면 해당 Port를 `Err-Disabled` 상태로 전환하여 해당 Port를 차단하여 혹시 모를 Layer 2 Loop를 방지하는 기술이다. 

일반적으로 PortFast가 설정된 단말 Port에 함께 사용한다. 사용자가 임의로 Switch를 연결하여 STP Topology를 변경하거나 Layer 2 Loop를 발생시키는 것을 방지한다.

### BPDU Filter

BPDU Filter는 Port에서 BPDU를 송수신하지 않도록 하는 기능이다.

BPDU를 전달할 필요가 없거나 BPDU가 외부 장비로 전달되는 것을 막아야 하는 특정 환경에서 사용한다. BPDU Filter 기능은 일반적인 상황에서는 잘 사용하지 않는 기술이다.

인터페이스에 직접 BPDU Filter를 설정하면 해당 Port는 BPDU를 송수신하지 않기 때문에 STP가 정상적으로 Loop를 감지할 수 없어 Loop가 발생할 수 있다.

### Root Guard

Root Guard는 Port에서 Superior BPDU가 수신되면 해당 Port가 Root Port로 변경되지 않도록 제한하는 기능이다.  

Root Guard가 설정된 Port에서 Superior BPDU가 수신되면 해당 Port는 `Root-Inconsistent` 상태로 전환되면서 해당 인터페이스로 들어오는 트래픽을 차단한다. 

Root Guard는 새로 추가되는 Access Switch나 외부 Network의 Switch가 Root Bridge로 선출되는 것을 방지하기 위해 백본 Switch의 하위 방향 Port에 주로 설정한다.

### Loop Guard

Loop Guard는 Alternate Port에 BPDU 수신이 중단되었을 때 해당 Port가 Designated Port로 변경되어 Forwarding 상태로 전환되는 것을 방지하는 기능이다.

연결된 장비나 링크의 문제로 인해 Alternate Port에서 BPDU를 받지 못하면 해당 Port는 연결된 장비의 링크가 죽었다고 판단하고 자신의 Alternate Port를 Forwarding 상태로 만든다. 하지만 연결된 장비는 BPDU만 보내지 못하고 있고 실질적인 Data Frame은 계속 보내고 있는 상황이기 때문에 Loop를 구성하는 Port들이 모두 Forwarding 상태가 되어 Layer 2 Loop가 발생할 수 있다.

Root Guard와 Loop Guard는 목적이 다르기 때문에 같은 Port에 동시에 설정할 수 없다.
- Root Guard는 BPDU를 받으면 동작하는 기능
- Loop Guard는 BPDU를 받지 않으면 동작하는 기능

### UplinkFast

UplinkFast는 Access Switch의 Root Port에 직접적인 Link 장애가 발생했을 때 Backup Link를 빠르게 활성화하는 Cisco 전용 기능이다.

기존 Root Port가 Down 상태가 되면 Blocking 상태였던 Backup Port를 새로운 Root Port로 선택하고 즉시 Forwarding 상태로 전환한다.

UplinkFast를 활성화하면 해당 Switch가 Root Bridge로 선출되지 않도록 Bridge Priority가 `49152`로 변경된다.

UplinkFast는 Traditional STP에서 사용하던 기능이며, RSTP에서는 기본적으로 UplinkFast의 기능이 탑재되어 있어 해당 설정 없이 장애 발생 시 Block Port를 빠르게 Forwarding 상태로 전환할 수 있다.

### BackboneFast

BackboneFast는 Switch에 직접 연결되지 않은 간접적인 Link 장애가 발생했을 때 STP의 수렴 속도를 높이는 Cisco 전용 기능이다.

Switch가 Block Port를 통해 Neighbor Switch로부터 Inferior BPDU를 수신하면 해당 Switch는 자신의 Root Port 방향으로 Root Link Query를 보내 Root Bridge까지 가는 경로가 살아 있는지 확인한다. Root Bridge는 RLQ Response를 보내 해당 경로가 정상적으로 살아 있다는 것을 알려준다.

Root Bridge로 가는 경로가 확인되면 해당 Switch는 Max Age Timer를 기다리지 않고 Port를 Listening / Learning 상태로 전환해 복구 시간을 단축한다.  

BackboneFast는 STP Domain에 속한 모든 Switch에 설정해야 한다. RSTP에는 BackboneFast와 유사한 빠른 수렴 기능이 기본적으로 포함되어 있다.


