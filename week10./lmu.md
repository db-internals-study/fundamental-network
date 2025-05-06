# 10장 서버의 방화벽 설정/동작
## 10.1 리눅스 서버의 방화벽 확인 및 관리
- iptables: 방화벽의 역할처럼 패킷을 차단, 허용하는 등의 필터링 기능을 직접 수행하는 것은 아니며 리눅스 커널에 내장된 netfilter 라는 리눅스 커널 모듈을 통해서 실제로 필터링이 이루어집니다.
- firewalld, ufw 모두 결국 netfilter 에 대한 프런트엔드 역할

### 10.1.1. iptables 이해하기
- Filter 테이블: iptables 에서 패킷을 허용하거나 차단하는 역할을 선언하는 영역
- INPUT, OUTPUT, FORWARD 체인: 호스트 기준으로 호스트로 들어오거나 (INPUT), 호스트에서 나가거나(OUTPUT), 호스트를 통과할(FORWARD) 때 사용되는 정책들의 그룹. 패킷의 방향성에 따라 각 체인에 정의된 정책이 적용됨
- Match: 제어하려는 패킷의 상태 또는 정보 값의 정의, 정책에 대한 조건
- Target: Match와 일치하는 패킷을 허용할지, 차단할지에 대한 패킷 처리 방식

### 10.1.2 리눅스 방화벽 활성화/비활성화
- CentOS 7 이후 버전부터 iptables이 기본적으로 포함되지 않고 firewalld가 활성화되어 있어 firewalld 서비스를 비활성화하고 iptables을 설치해야 iptables 을 사용할 수 있다.

### 10.1.3 리눅스 방화벽 정책 확인


```
iptables -L
```

```
ACCEPT all -- anywhere anywhere state RELATED, ESTABLISHED
```

```
ACCEPT icmp -- anywhere anywhere
```

```
ACCEPT tcp -- anywhere anywhere NEW tcp dpt:ssh
```

```
REJECT all -- anywhere reject-with icmp-host-prohibited
```

### 10.1.4 리눅스 방화벽 정책 관리

```
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

- 특정 서비스 포트에 대해 특정 IP만 허용
```
iptables -A INPUT -i eth0 -p tcp -s 172.16.10.10/32 --dport 22 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp --dport 22 -j DROP
```

### 10.1.5 리눅스 방화벽 로그 확인
```
tail -f /var/log/messages
```

- iptables 더 알아보기:
	- https://velog.io/@koo8624/Linux-A-Deep-dive-into-iptables-and-Netfilter
	1. 테이블: filter, nat, mangle, raw, security
		- filter 테이블: 패킷을 차단하거나 허용할 목적으로 사용, INPUT, OUTPUT 체인
		- nat 테이블: NAT기능을 위한 테이블. POSTROUTING, PREROUTING
		- mangle 테이블: 패킷 헤더의 TOS, TTL 값을 변경하는 역할
		- raw 테이블: 연결 추적시스템에서 처리하면 안되는 패킷을 표시하는 용도로 사용 PREROUTING, OUTPUT 체인이 있지만 널리 사용되지는 않음
		- security 테이블: 필수 접근 제어 (Mandatory Access Control, MAC) 네트워크 규칙에 사용. SELinux와 같은 리눅스 보안 모듈에 의해 구현
	2. 체인: 특정 패킷에 대해 적용할 정책을 정의한 것
	3. 타깃: 정의한 정책과 패킷이 같을 때 취하는 행동:
		- ACCEPT: 패킷을 정상적으로 처리
		- REJECT: 패킷을 폐기하면서 패킷이 차단되었다는 응답 메시지를 전송
		- DROP: 패킷을 그대로 폐기
		- LOG: 패킷을 syslog 에 기록
	4. iptables 실행 옵션
		- `-A(--append)` : 새로운 규칙 추가
		- `-D(--delete)` : 규칙 삭제
		- `-C(--check)` : 패킷 테스트
		- `-R(--replace)` : 새로운 규칙으로 교체
		- `-I(--insert)` : 새로운 규칙 삽입
		- `-L(--list)` : 규칙 출력
		- `-F(--flush)` : chain 에서 규칙 삭제
		- `-Z(--zero)` : 모든 chain의 패킷과 바이트 카운터 값을 0으로 만듬
		- `-N(--new)` : 새로운 chain을 추가
		- `-X(--delete-chain)` : chain 삭제
		- `-P(--policy)` : 기본 정책 변경
	5.  정책 옵션
		- `-s(--source)` : 출발지 IP 주소나 네트워크와 매치
		- `-d(--destination)` : 목적지 IP주소나 네트워크와 매치
		- `-p(--protocol)` : 특정 프로토콜과 매치
		- `-i(--in-interface)` : 입력 인터페이스
		- `-o(--out-interface)` : 출력 인터페이스
		- `--state` : 연결 상태와 매치
		- `--string` : 어플리케이션 계층 데이터 바이트 순서와 매치
		- `--comment` : 커널 메모리 내의 규칙과 연계되는 최대 256바이트 주석
		- `-y(--sync)` : SYN 패킷 불허
		- `-f(--fragment)` : 두번째 이후의 조각 규칙 명시
		- `-t(--table)` : 처리될 테이블
		- `-j(--jump)` : 규칙에 맞는 패킷 처리방법 명시
		- `-m(--match)` : 특정 모듈과의 매치

## 10.2 윈도 서버의 방화벽 확인 및 관리
