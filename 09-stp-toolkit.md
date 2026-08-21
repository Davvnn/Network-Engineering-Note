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

## 동작 원리

### PortFast와 BPDU Guard 동작 과정

1\. 단말이 연결된 Access Port에 PortFast와 BPDU Guard를 설정한다.

2\. 단말이 연결되면 해당 Port는 Listening과 Learning 상태를 기다리지 않고 즉시 Forwarding 상태로 전환된다.

3\. 단말들은 BPDU를 전송하지 않으므로 Port는 계속 Forwarding 상태로 동작한다.

4\. 만약 사용자가 해당 단말을 제거하고 Switch를 연결해서 BPDU가 수신되면 해당 Port에서 BPDU Guard가 동작한다.

5\. BPDU Guard는 해당 Port를 `Err-Disabled` 상태로 전환하여 Traffic을 차단한다.

6\. 관리자는 연결 상태와 Loop 원인을 확인한 후 Port를 수동으로 복구한다.

### BPDU Filter 동작 과정

1\. 인터페이스에 BPDU Filter를 직접 설정하면 해당 Port는 BPDU를 송수신하지 않는다.

2\. BPDU Filter 설정이 들어간 Switch와 연결된 Switch는 서로의 BPDU를 확인할 수 없기 때문에 STP Topology를 정상적으로 계산할 수 없다.

3\. 물리적인 이중화 Link가 구성되어 있으면 STP가 Loop를 감지하지 못하여 Layer 2 Loop가 발생할 수 있다.

### Root Guard 동작 과정

1\. 백본 스위치의 하위 방향 Port에 Root Guard를 설정한다.

2\. 하위 Switch가 현재 Root Bridge보다 낮은 Bridge ID를 가진 Superior BPDU를 전송한다.

3\. Root Guard가 Superior BPDU를 감지하고 해당 Port를 `Root-Inconsistent` 상태로 전환한다.

4\. 해당 Port는 BPDU를 확인하지만 일반 Frame은 전달하지 않는다.

5\. 하위 Switch가 Superior BPDU 전송을 중단하면 Port는 자동으로 정상 상태로 복구된다.

6\. 기존에 지정한 Root Bridge가 계속 STP Topology의 Root Bridge로 동작한다.

### Loop Guard 동작 과정

1\. Alternate Port는 이웃 Switch로부터 BPDU를 계속 수신한다.

2\. Alternate Port와 연결된 이웃 Switch에 문제가 생겨 일반 Frame은 전달되지만 BPDU는 전달되지 않는다.

3\. Loop Guard가 없다면 Max Age Timer가 지난 후 해당 Alternate Port는 Designated Port로 변경되어 Forwarding 상태로 전환될 수 있다.

4\. 하지만 해당 Port에 Loop Guard가 설정되어 있으므로 BPDU 수신이 중단되면 Port는 `Loop-Inconsistent` 상태로 전환된다.

5\. 해당 Port는 Forwarding 상태로 전환되지 않으므로 Layer 2 Loop가 발생하지 않는다.

6\. BPDU가 다시 수신되면 Port는 자동으로 정상 상태로 복구된다.

### UplinkFast 동작 과정

1\. Access Switch는 Root Bridge 방향의 Port를 Root Port로 사용한다.

2\. 해당 스위치에 이중화된 다른 Uplink는 Loop 방지를 위해 Blocking 상태로 대기한다.

3\. Access Switch의 Root Port에 직접적인 Link 장애가 발생하면, UplinkFast는 Blocking 상태의 Backup Port를 새로운 Root Port로 선택한다.

4\. Backup Port는 Listening과 Learning 상태를 기다리지 않고 즉시 Forwarding 상태로 전환된다.

5\. Access Switch의 Traffic은 새로운 Root Port를 통해 Root Bridge 방향으로 전달된다.

### BackboneFast 동작 과정

1\. 다른 Switch에서 Link 장애가 발생하지만 현재 Switch에 직접 연결된 Link의 장애가 아니므로 물리적으로 감지할 수 없다.

2\. 장애가 일어나 Root Bridge로 가는 경로를 잃어버린 Neighbor Switch는 자신을 Root Bridge로 표시한 Inferior BPDU를 전송한다.

3\. Inferior BPDU를 수신한 다른 Switch는 Network의 간접적인 Link 장애를 감지한다.

4\. 해당 Switch는 R`oot Link Query Request`를 전송하여 기존 Root Bridge로 가는 경로가 남아 있는지 확인한다.

5\. Root Bridge는 `Root Link Query Response`를 전송한다.

6\. Response를 수신한 Switch는 기존 Root Bridge가 정상적으로 동작하고 있음을 확인한다.

7\. Switch는 Max Age Timer를 기다리지 않고 Port를 Listening과 Learning 상태로 전환해 복구 시간을 단축한다.
