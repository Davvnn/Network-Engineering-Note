# Device Access / Authentication

## 개념

### Console

Console은 장비의 Console Port에 직접 연결하여 관리하는 방식이다.

Network 연결이 없어도 장비에 접속할 수 있으므로 초기 설정이나 장애가 발생했을 때 사용한다.

### VTY

VTY(Virtual Teletype)는 SSH나 Telnet을 이용하여 Network를 통해 장비에 원격 접속할 때 사용하는 가상의 Line이다.
```
R1(config)# line vty 0 4
```
- `0 4`: 동시에 사용할 수 있는 VTY Line `5개`를 설정한다.
- 장비에 따라 지원하는 VTY Line의 개수가 다를 수 있다.

## SSH

SSH(Secure Shell)는 Network 장비에 안전하게 원격 접속하기 위한 Protocol이다.

사용자 계정, Password 및 원격 접속 중 주고받는 내용을 암호화하므로 중간에서 Packet을 확인해도 내용을 쉽게 확인할 수 없다.

SSH는 TCP Port `22`를 사용한다.

### SSH Key

Cisco 장비에서 SSH를 사용하려면 장비 확인에 사용할 RSA Key Pair를 생성해야 한다.
```
R1(config)# crypto key generate rsa modulus 2048
```
장비에 SSH로 로그인하려면 Username과 Password를 사용하거나, PC의 Private Key와 장비에 등록된 Public Key를 사용하여 인증한다.

### Telnet

Telnet은 Network 장비에 원격 접속하기 위한 Protocol이다.

Telnet은 TCP Port `23`를 사용한다.

Username, Password 및 원격 접속 중 주고받는 내용을 암호화하지 않고 Plaintext로 전송한다.

공격자가 중간에서 Packet을 수집하면 로그인 정보와 입력한 명령어를 그대로 확인할 수 있으며, 탈취한 계정으로 장비에 접속하여 설정을 변경할 수 있다.

따라서 실제 운영 환경에서는 Telnet 대신 SSH를 사용하며, Telnet은 주로 Lab 환경에서 사용한다.

### AAA

AAA는 장비에 접속하는 사용자를 확인하고, 사용할 수 있는 명령어를 제한하며, 사용 기록을 저장하는 기능이다.

#### Authentication

Authentication은 장비에 접속하려는 사용자가 누구인지 확인한다.

#### Authorization

Authorization은 인증된 사용자가 어떤 기능과 명령어를 사용할 수 있는지 결정한다.

#### Accounting

Accounting은 사용자가 장비에 접속한 시간과 실행한 명령어 등의 기록을 AAA Server에 전송하는 기능이다.
