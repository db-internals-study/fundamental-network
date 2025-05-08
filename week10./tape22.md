# 10장 서버의 방화벽 설정/동작

서버를 새로 구축하면 정상적으로 연결되었는지 확인하기 위해 ping을 보냈을 때, 서버 os와 네트워크 설정에 문제가 없음에도 정상적으로 응답을 받지 못하는 경우가 있다.

- 예를 들어 netstat나 ss 명령어로 확인 시 서비스는 listening 상태이지만 외부 → 웹 서버 접속이 불가능한 케이스
- 서버에서 ping 응답이 잘 오고, 보안 장비에서도 정상적으로 통과되는데 응답이 안오는 이유는
⇒ OS의 자체 방화벽이 일부 포트를 제외하고 차단해두는 화이트리스트 정책으로 관리하기 때문이다.

⇒ 서버나 데이터베이스 접근통제 솔루션의 경우 해당 솔루션의 ip 주소와 포트만 허용하고 나머지는 차단해야할 때 서버 방화벽이 활용된다.


<img width="769" alt="Screenshot 2025-05-08 at 8 21 46 PM" src="https://github.com/user-attachments/assets/e554e309-8401-4c6c-9479-9492c7e6fcad" />

---

### 리눅스 서버의 방화벽 확인 및 관리

호스트의 방화벽 기능을 위해 다음과 같은 기능을 사용한다.

- iptables / CentOS 7 이상은 firewalld
    - 리눅스 커널에 내장된 netfilter 커널 모듈을 통해 실제 필터링을 진행하는데,
    - netfilter 커널 모듈처럼 실제로 패킷을 차단/허용하는 필터링을 직접 수행하는 것은 아님.
    - netfilter에게 어떤 룰로 처리할지를 알려주는 역할이다.
```
⇒ 둘 다 netfilter을 사용해서 패킷 필터링을 수행하는 것은 동일하나, netfilter 규칙을 다루는 방식에 차이가 있다.

- iptables : INPUT, OUTPUT, FORWARD와 같은 체인을 직접 조작해서 규칙을 관리하는 command 툴
- firewalld : 백그라운드에서 데몬으로 실행되면서 방화벽을 내리지 않아도 룰을 관리할 수 있음.
```
- ubuntu는 UFW(Ubuntu firewall)

**iptables 이해하기**

- 서버에서 허용하거나 차단할 ip 나 포트 rules를 정책 그룹으로 관리한다.
    - 트래픽 구간 별(INPUT , OUTPUT, FORWARD)로 그룹을 관리한다.
    - 즉, 정책의 방향성에 따라 관련 그룹으로 분류하고, 다시 상위 그룹으로 묶으면서 관리
    - 정책의 방향성에 따라 구분한 그룹을 chain, 이를 역할별로 구분한 그룹을 table이라고 정의한다.

<img width="754" alt="Screenshot 2025-05-08 at 8 23 01 PM" src="https://github.com/user-attachments/assets/63596de5-acd5-4dbd-8709-73559560d385" />


iptables에는 5가지 테이블이 있다.

- **NAT**
    - 출발지와 목적지의 IP를 변환하는 NAT 기능을 위한 테이블로, 라우팅 전후로 존재하는 체인이 사용
    - source NAT : POSTROUTING 체인
    - destination NAT : PREROUTING 체인
- **Mangle**
    - 패킷 헤더의 TOS, TTL 값을 변경하는 역할
    - 네트워크 보안 장비로 사용될 때를 제외하고는 자주 사용되지 않음
- **Raw**
    - 연결 추적 시스템에서 처리하면 안되는 패킷을 표시하는 용도
- **Security**
    - 필수접근제어(MAC) 네트워크 규칙에 사용
    - filter 테이블 이후에 호출되며, INPUT/OUTPUT/FORWARD 체인을 제공
- **Filter**
    - 패킷을 허용하거나 차단하는데 사용하고,
    - 리눅스의 호스트 방화벽은 이 필터테이블을 사용하여 트래픽을 제어한다.
    - 트래픽 방향성에 따라 INPUT, OUTPUT, FORWARD 체인이 있으며 각 체인 별로 방화벽 정책을 정의할 수 있다.
    - **호스트로 들어오거나(INPUT), 나가거나(OUTPUT), 호스트를 통과하거나(FORWARD)**.
        - 주로 라우터나 방화벽같은 호스트라면 forward 체인의 역할을 지정할 수 있다.
        - prerouting/postrouting 체인 → input 체인 순서

<img width="762" alt="Screenshot 2025-05-08 at 8 23 52 PM" src="https://github.com/user-attachments/assets/7de1c550-9eae-467b-b95a-8342ea20ff23" />

filter 테이블의 요소들은 다음과 같다.

- INPUT, OUTPUT, FORWARD 체인
    - 패킷의 방향성에 따라 체인에 정의된 정책이 적용
- Match
    - 제어하려는 패킷의 상태 또는 정보 값을 정의
- Target
    - 매치 조건과 일치하는 패킷을 허용할지 드랍할지  결정하는 패킷 처리방식


### iptables

CentOS 7 기준으로는 iptables를 다음 명령어로 시작을 할 수 있고

```
systemctl disable firewalld
systemctl stop firewalld
systemctl start iptables.service
```

iptables의 기본 정책을 —list 옵션으로 확인해보면 다음과 같다.

`ACCEPT icmp — anywhere anywhere` 
  - ping과 같은 서비스를 사용할 수 있음
    - IP 프로토콜의 한 구성요소로, 통신상태를 점검하고 문제를 알려주는 네트워크용 헬스체크
    - ping : echo request, echo reply 패킷 등이 예시
    - 패킷 구조도 간단하고 포트 번호도 없는 네트워크 진단용
 
`ACCEPT tcp — anywhere anywhere state NEW tcp dpt:ssh`
- 목적지 서비스포트가 ssh인 경우에만 허용하는 정책 (외부 → 서버 접속은 ssh만 허용)


`REJECT all —anywhere anywhere reject-with icmp-host-prohibited`
  - 정책에 매치되지 않는 패킷들을 차단하는데
  - reject는 drop 과 달리 ICMP 프로토콜을 이용해서 패킷 차단 이유를 출발지에 전달함.
  - icmp-port-unreachable이 전달되는데, reject-with 옵션으로 ICMP 에러유형을 지정해서 패킷이 차단된 이유를 알려줄 수 있다.

    
`accept all —anywhere anywhere`
    - 정책이 적용되는 인터페이스는 루프백 인터페이스이기 때문에,
    - 일반 서비스 인터페이스 패킷에는 적용되는 정책은 아님.
    - 출발지에 대해 모든 트래픽을 허용하는 것처럼 보이지만 -S로 명령어를 확인해보면 확인이 가능하다.

---

### 리눅스 방화벽 정책 관리

iptables 정책을 추가하거나 삭제하는 방법을 통해 기본적인 방화벽 동작을 서버에 적용해볼 수 있다.

```
 iptables -A INPUT -p tcp —dport 80 -j ACCEPT
```

- -A, —append 로 정책을 추가하고
- 어떤 체인을 적용할 것인지 정의
- -p 프로토콜을 tcp로 지정하고
- 목적지 포트를 제어하기 위해 80 포트를 정의
- ip 주소는 별도로 설정하지 않으면 anywhere
- 그리고 -j로 정책에 일치하는 패킷을 어떻게 처리할것인지 나타내는 target을 지정한다.

⇒ 기본적으로는 새로 입력한 정책은 가장 아래에 있기 때문에, 앞에 있는 정책들이 먼저 적용되므로 어느 위치에 설정했는지를 확인한 다음 테스트해야한다.

- 기존 설정은 삭제처리하거나 (iptables -D INPUT -p tcp —dport 21 -j ACCEPT)
- 혹은 순서를 변경해야하는 경우 (iptables -I INPUT 5 -p tcp —dport 80 -j ACCEPT) 명령어를 통해 관리

그리고 서버접근 통제솔루션의 경우 특정 서비스 포트에 대해 특정 ip만 허용해주어야하기 때문에, 아래처럼 특정 ip만 허용하고 나머지는 드롭하게끔 차단할 수 도 있다.

```
- iptables -A INPUT -i eth0 -p tcp -s 172.16.10.10/32 —dport 22 -j ACCPET
- iptables -A INPUT -i eth0 -p tcp  —dport 22 -j DROP
```

하지만 이런 설정들은 iptables에 정책을 설정하면, 서버를 재부팅하거나 iptables를 재시작했을 때 정책이 초기화되므로 `/etc/sysconfig/iptables` 정책 파일에 직접 설정해야 영구적으로 적용할 수 있다.

**방화벽 로그 확인하기**

- tail -f  /var/log/messages
- iptables 로그만 보려면 rsyslog.conf에 iptables.log를 별도로 추가하고, iptables에 로그를 남길 수 있도록 설정
    - iptables -I INPUT -j LOG —log-level 4 —log-prefix ‘## ZIGI-Log##’

---
