# 서버의 방화벽 설정/동작

서버에 설정된 방화벽으로 인해 연결이 안될 때가 있음.

일반적으로 화이트리스트 기반 정책 관리

## 리눅스 서버의 방화벽 확인 및 관리

iptables, firewalld, UFW 모두 netfilter에 대한 프론트엔드 역할

### iptables

서버로 유입되는 구간(INPUT), 서버에서 나가는 구간(OUTPUT), 서버를 통과하는 구간(FORWARD)

iptables에서 개별 정책의 방향성에 따라 구분한 그룹 : 체인(Chain)
체인을 역할별로 구분한 그룹 : 테이블(Table)
즉, 개별 정책의 그룹이 체인이 되고 체인 그룹이 테이블이 됨.

방화벽 역할을 위해 iptables에서 사용되는 것
- Filter 테이블 : iptables에서 패킷을 허용하거나 차단하는 역할을 선언
- INPUT, OUPTUT, FORWARD 체인  : 호스트 기준으로 IN,OUT,FORWARD 때 사용되는 정책의 그룹, 패킷에 방향성에 따라 각 체인에 정의된 정책이 적용
- Match : 제어하려는 패킷의 상태 혹은 정보값의 정의. 정책에 대한 조건
- Target : 조건과 일치하는 패킷을 허용할지 차단할지에 대한 패킷 처리 방식

### 방화벽 정책 확인
-L, or -S

### 리눅스 방화벽 정책 관리

상단의 정책이 하단의 정책보다 먼저 적용되는 Top Down 방식

대부분 방화벽 룰은 도착지 포트를 기반으로 지정(출발지 포트는 변동이 심하기 때문)

### 리눅스 방화벽 로그 확인

-L 뒤에 -v 옵션을 쓰면 통과하는 패킷과 바이트 수를 확인할 수 있음

MSS clamping : Maximum Segment Size를 비정상적으로 줄어들 수가 있는데, 이걸 방지하는 방법

### iptables 더 알아보기

#### 테이블

filter, nat, mangle, raw, security

- filter : 패킷 차단 허용 목적
- nat : NAT 기능을 위한. Source NAT - POSTROUTING chain / Destination NAT - PREROUTING chain
- mangle : 패킷 헤더의 TOS, TTL 값을 변경
- raw : 연결 추적 시스템에서 처리하면 안 되는 패킷을 표시하는 용도
- security : 필수 접근 제어(Mandatory Access Control, MAC) 네트워크 규칙에 사용

#### 체인

특정 패킷에 대해 적용할 정책을 정의
input, output, forward, prerouing, postrouting 등

#### 타깃

패킷이 iptables에 정의한 정책과 같을 때 취하는 행동

ACCEPT, REJECT, DROP, LOG, SNAT, DNAT 등

#### 실행 옵션

패킷 테스트를 비롯해 새로운 체인 추가 등 다양

#### 정책 옵션

정책에 매치되는 조건 설정

## 윈도우

윈도 AD를 통해 일괄적으로 AD에 연동된 사용자 PC에 방화벽 설정을 내려줄 수도 있음.

손실된 패킷 로그에 기록 : 방화벽에 차단된 패킷 로그에 대한 설정
