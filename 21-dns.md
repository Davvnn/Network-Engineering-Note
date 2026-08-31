# DNS

## 개념

### DNS

DNS(Domain Name System)는 Domain Name에 해당하는 IP Address 등의 정보를 조회하는 시스템이다.

사용자는 Server의 IP Address를 직접 입력하지 않고 Domain Name으로 접속할 수 있다.

예를 들어 `www.google.com`에 접속하면 DNS로 해당 `Domain Name`의 `IP Address`를 확인한 후, 해당 IP Address를 사용하여 Server와 통신한다.

### A Record

A Record는 Domain Name과 IPv4 Address를 Mapping하여 저장하는 DNS Record이다.

DNS Server는 여러 개의 A Record를 가지고 있을 수 있으며, 같은 Domain에 속한 이름이라도 각각 별도의 A Record로 저장한다.
```
www.google.com     A    192.0.2.10
mail.google.com    A    192.0.2.20
maps.google.com    A    192.0.2.30
www.youtube.com    A    192.0.2.40
```
- 각 줄이 하나의 A Record이므로 위 예시는 총 4개의 A Record이다.
- `www.google.com`, `mail.google.com`, `maps.google.com`은 모두 `google.com`에 속하지만 각각 별도의 A Record이다.

### DNS Port

일반적인 DNS는 UDP와 TCP Port `53`을 사용한다.
- UDP `53`: 일반적인 DNS Query와 Response에 사용한다.
- TCP `53`: UDP 응답이 잘려 다시 조회하거나, DNS Server 사이에서 전체 Zone 정보를 전달하는 Zone Transfer 등에 사용한다.

### Zone Transfer

Zone Transfer는 Primary DNS Server가 관리하는 Zone의 DNS Record를 Secondary DNS Server에 복사하여 같은 정보를 유지하는 과정이다.

예를 들어 Primary DNS Server에서 관리하는 `www.google.com`, `mail.google.com` 등의 DNS Record를 Secondary DNS Server에 복사한다.
- 전체 Zone 정보를 전달할 때는 TCP Port `53`을 사용한다.
- 두 DNS Server 자체의 IP Address는 다르지만, 복사한 A Record에 저장된 IP Address 정보는 동일하다.
- Secondary DNS Server는 Primary에 장애가 발생해도 DNS 조회를 처리할 수 있으며, 평상시에도 DNS 요청에 응답할 수 있다.

### Domain Name과 FQDN

Domain Name은 DNS에서 사용하는 이름이며, 점으로 구분된 계층 구조를 가진다.

`www.example.com`은 다음과 같이 구분할 수 있다.
- `com`: Top-Level Domain(TLD)이다.
- `example.com`: Domain Name이다.
- `www`: Domain 안에서 웹 서비스를 나타내기 위해 사용하는 Host 이름이다.

FQDN(Fully Qualified Domain Name)은 Host부터 상위 Domain까지 포함한 전체 이름이다.

### DNS Server의 역할

Client가 `www.google.com`의 IP Address를 조회하는 경우이다.

1\. Recursive DNS Server
- Client를 대신하여 IP Address를 찾아주는 Server이다.
- Client가 `www.google.com`의 IP Address를 요청하면, 다른 DNS Server들을 조회하여 결과를 알려준다.

2\. Root DNS Server
- TLD DNS Server의 위치를 알려주는 Server이다.
- `www.google.com`을 조회하면 `.com`을 담당하는 TLD DNS Server 정보를 알려준다.

3\. TLD DNS Server
- 조회한 Domain을 담당하는 Authoritative DNS Server가 어디인지 알려주는 Server이다.
- `.com` TLD DNS Server는 `google.com`을 담당하는 Authoritative DNS Server 정보를 알려준다.

4\. Authoritative DNS Server
- 해당 Domain의 DNS Record를 직접 가지고 있는 Server이다.
- `google.com`을 담당하는 Authoritative DNS Server는 자신의 Record를 확인하고 `www.google.com`의 IP Address를 응답한다.

이렇게 찾은 IP Address를 Recursive DNS Server가 Client에게 전달한다.
- Client가 Root와 TLD DNS Server에 직접 물어보는 것이 아니라 Recursive DNS Server가 대신 조회한다.
- Recursive DNS Server에 이전 조회 결과가 유효한 Cache로 저장되어 있다면 위 과정을 반복하지 않고 바로 응답할 수 있다.

### DNS 조회 예시

Client와 회사 DNS Server에 필요한 Cache가 없는 경우이다.

1\. PC → 회사 DNS: “`www.google.com`의 IP Address를 알려줘.”
- 내부 DNS를 사용하는 경우: 내부 DNS 주소인 10.0.0.20으로 DNS Query를 보낸다. 
- 외부 DNS를 사용하는 경우: 외부 DNS 주소인 8.8.8.8 등으로 DNS Query를 보낸다.

2\. 회사 DNS → Root DNS: `.com` DNS Server의 위치를 받는다.

3\. 회사 DNS → `.com` DNS: `google.com` 담당 DNS Server의 위치를 받는다.

4\. 회사 DNS → `google.com` 담당 DNS: `www.google.com`의 IP Address를 받는다.

5\. 회사 DNS → PC: 찾은 IP Address를 알려준다.

6\. PC → Google Web Server: 받은 IP Address로 접속한다.

### Recursive Query와 Iterative Query

Recursive Query는 요청받은 DNS Server가 필요한 정보를 대신 조회하여 최종 결과나 오류를 반환하도록 요청하는 방식이다.
- Client → Recursive DNS Server: 일반적으로 Recursive Query를 사용한다.

Iterative Query는 DNS Server가 답을 알고 있으면 바로 응답하고, 답을 모르면 다른 DNS Server에 물어보라고 알려주는 방식이다.   	
- Recursive DNS Server → Root / TLD DNS Server: 일반적으로 Iterative Query를 사용한다.

### DNS Record

`A`: 이름에 해당하는 IPv4 Address를 지정한다.

`AAAA`: 이름에 해당하는 IPv6 Address를 지정한다.

`CNAME`: 별칭이 가리키는 다른 Domain Name을 지정한다.

`MX`: 해당 Domain의 Mail을 수신할 Server를 지정한다.

`NS`: 해당 Zone을 담당하는 Authoritative DNS Server를 지정한다.

`PTR`: IP Address에 해당하는 이름을 지정하며 Reverse Lookup에 사용한다.

`SOA`: Zone의 관리 정보와 Serial Number 등을 저장한다.

`TXT`: Domain 소유권 확인이나 Mail 인증 등에 사용하는 Text 정보를 저장한다.

`SRV`: 특정 서비스를 제공하는 Server와 Port 등의 정보를 지정한다.

```
www.example.com     A        192.0.2.10
portal.example.com  CNAME    www.example.com
```
- `www.example.com`은 `192.0.2.10`으로 조회된다.
- `portal.example.com`은 `www.example.com`을 가리키는 별칭이다.
- CNAME은 IP Address가 아니라 다른 Domain Name을 지정한다.

