# 9장 보안
## 9.1. 보안의 개념과 정의
### 9.1.1. 정보 보안의 정의
- 기밀성, 무결성, 가용성
- w진정성, 책임성, 부인 방지

### 9.1.2. 네트워크의 정보 보안
- 정보의 이동 경로

### 9.1.3. 네트워크 보안의 주요 개념
- Trust 네트워크
- Untrust 네크워크

- 네트워크 보안 분야의 분류:
	- 인터넷 시큐어 게이트웨이 (Internet Secure Gateway):
		- Trust (or DMZ) Network 에서 Un trust network 로의 통신을 통제
		- Firewall, SWG(Secure Web Gateway), WebFilter, Application Control, Sandbox
	- 데이터 센터 시큐어 게이트웨이(Data Center Secure Gateway):
		- Untrust 네트워크에서 Trust 네트워크로의 통신을 통제
		- IPS, DCSG(DataCenter Secure Gateway), WAF(Web Application Firewall), Anti-DDoS
#### 9.1.3.1. 네트워크 보안 정책 수립에 따른 분류
- 화이트리스트
- 블랙 리스트

#### 9.1.3.2. 정탐, 오탐, 미탐(탐지 에러 타입)
- True Positive, True Negative, False Positive, False Negative

### 9.1.4. 네트워크 정보 보안의 발전 추세와 고려사항
- 네트워크나 서비스의 큰 변화는 보통 필요에 의한 변화로 보안이 고려되지 않는 경우가 많다.

## 9.2. 보안 솔루션의 종류
- DDoS 방어 장비 - 방화벽 - IPSS - WAF - Server

### 9.2.1. DDoS 방어 장비

### 9.2.2. 방화벽
- 3,4 계층 정보를 기반으로 정책을 세우고 Allow or Deny

### 9.2.3. IDS, IPS
- IDS (Intrusion Detection System)
- IPS (Intrusion Prevention System)

### 9.2.4. WAF
- WAF 의 형태:
	- 전용 네트워크 장비
	- 웹 서버의 플러그인
	- ADC 플러그인
	- 프록시 장비 플러그인

### 9.2.5. 샌드박스
- APT (Advanced Persistent Threat)
- ATA (Advanced Target Attack)
- 샌드박스는 APT의 공격을 방어하는 대표적인 장비로 악성 코드를 샌드박스 시스템 안에서 직접 실행시킨다.

### 9.2.6 NAC
- Network Access Control

### 9.2.7 IP 제어

### 9.2.8. 접근 통제
- Agent Based, Agentless
- Bastion Host
	- 서버 접근을 위한 보든 통신은 베스천 호스트를 통해서만 가능하다.

### 9.2.9 VPN

## 9.3. 방화벽
### 9.3.1. 방화벽의 정의
- 일반적으로 방화벽은 네트워크 3,4 계층에서 동작하며 세션을 잊니하는 상태 기반 엔진(Stateful Packet Inspection, SPI)로 동작한다.


### 9.3.2. 초기 방화벽
- Stateless, Packet Filter:
	- Source IP, Destination IP, Protocol No, Source Port, Destination Port

### 9.3.3. 현대적 방화벽의 등장(SPI 엔진)

### 9.3.4. 방화벽 동작 방식
- TCP Anti-Replay:
	- 시퀀스와 ACK 번호가 갑자기 변경되는 것을 인지해 세션 탈취 공격을 방지하는 방법

### 9.3.5 ALG
- Application Layer Gateway
- 최근 대부분의 어플리케이션이 방화벽이나 NAT 를 고려해 개발되고 있고, STUN(Session Traversal Utilities for NAT)와 같은 홀 펀칭(Hole Punching) 기술들도 많이 발전해 오래된 프로토콜을 제외하면 ALG 기능을 사용하지 않는 추세이다.

### 9.3.6. 방화벽의 한계
- 현대 방화벽:
	- NGFW(Next Generation Firewall)
	- UTM(Unified Threat Management Firewall)
- 요즘은 점점 어플리케이션이나 어플리케이션 프로토콜에서 취약점을 가지고 있기 때문에 방화벽만으로는 대응하기 어렵다

- NFV (Network Function Virtualization): 네트워크 컴포넌트를 가상화해 범용 하드웨어에 적용하려는 기술

### 9.4. IPS, IDS
### 9.4.1. IPS, IDS 의 정의
- Intrusion Detection System: 침입 탐지 시스템
 - Intrusion Prevention System: 침입 방지 시스템
### 9.4.2. IPS, IDS 의 동작 방식
#### 9.4.2.1. 패턴 매칭 방식
#### 9.4.2.2. 어노말리 공격 방어
- 프로파일 어노말리
- 프로토콜 어노말리

### 9.4.3. IPS, IDS의 한계와 극복 (NGIPS)
- IPS는 오탐이 많이 발생
- NGIPS(Next Generation IPS)

## 9.5. DDoS 방어 장비
### 9.5.1. DDoS 방어 장비의 정의
### 9.5.2 DDoS 방어 장비 동작 방식
- 프로파일링 기법
- 일반적인 보안 장비처럼 데이터베이스 기반으로 방어

### 9.5.3. DDoS 공격 타입
- 볼류메트릭 공격:
	- NTP 증폭, DNS증폭, DP Flood, TCP Flod
- 프로토콜 공격:
	- SYn 플러드, Ping of Death
- 애플리케이션 공격:
	- HTTP Flood, DNS Service Attack, Slowloris (Slowloris는 공격자가 공격자와 대상 간에 많은 HTTP 연결을 동시에 열고 유지함으로써 대상 서버를 압도하는 서비스 거부 공격 프로그램)

### 9.5.4. 볼류메트릭 공격
#### 9.5.4.1. 좀비 PC를 이용한 볼류메트릭 공격


## 9.6. VPN
- Virtual Private Network
### 9.6.1 VPN 동작 방식
- VPN의 구현 방식:
	- Host To Host 통신 보호
	- Network To Network 통신 보호
	- Host 가 Network 로 접근할 때 보호

