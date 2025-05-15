# 11장 이중화 기술
## 11.1 이중화 기술 개요
### 11.1.1. SPoF
- 단일 장애점: Single Point of Failure

### 11.1.2. 이중화의 목적
- Active-Active vs Active-Standby
- Fault Tolerance(FT): 장애 허용, 결함 감내

## 11.2. LACP
- 초기에는 각자 벤더 독자적인 방법으로 대역폭을 늘림
- LACP(Link Aggregation Control Protocol) : 상호 호환ㅏ능한 연결 계층 표준화
- 목표:
	- 링크 사용률 향상(Improved Utilization of Available Link)
	- 향상된 장애 회복(Improved Resilence)

### 11.2.1. LACP 동작방식
- LACPDU(LACP Data Unit) 이라는 프레임 사용
- 멀티케스트를 이용: `01:80:c2:00:00:02` - `01:80:c2:00:00:10` 사용
- 같은 장비에서 사용해야한다.
	- 서로 다른 장비에서 LACP 를 사용하기 위해서는 MC-LAG 사용
- 연결된 모든 장비에서 LACP를 사용가능해야한다.
- LACP 모드:
	- Active: LACPDU를 먼저 송신하고 상대방이 LACP를 구성된 경우, LACP를 구성
	- Passive: LACPDU를 송신하지 않지만, LACPDU를 수신받으면 응답해  LACP를 구성

### 11.2.2. LACP와 PXE
- 서버의 인터페이스를 하나의 논리 포트로 묶는 Bonding/ Teaming
- PXE: Pre-boot eXecution Environment)
- LACP는 OS 설치 이후에 사용할수 있으므로, PXE로 세팅하고 사용한다.

## 11.3. 서버의 이중화 설정 (Windows, Linux)
- 윈도우: Team/Teaming
- 리눅스: Bond/Bonding

- 서버의 인터페이스 이중화 구성: 하나의 논리 인터페이스를 사용하더라도 액티브 액티브 이외로 사용 가능:
	- Round Robin
	- Active - Standby
	- Balance-xor
	- Broadcast
	- LACP
- 네트워크 인터페이스의 이중화 구성: 두개 이상의 물리 인터페이스를 하나의 논리 인터페이스로 사용하여 액티브 액티브로 사용

### 11.3.1 리눅스 본딩모드
- 액티브 스텐바이로는 Mode1, Active-Active로는 Mode4(LACP) 를 사용한다.

#### 11.3.1.1. Mode 1: Active-Standby
#### 11.3.1.2. Mode 4: LACP


### 11.3.2 윈도 티밍모드
- Skip

### 11.3.3. 리눅스 본드 설정 및 확인
#### 11.3.3.1 CentOS에서 본드 설정 및 확인

```
/etc/sysconfig/network-scripts

# ifcfg-bond0
DEVICE=bond0
BOOTPROTO=none
onBOOT=yes
BOOTPROTO=static
IPADDR=10.10.10.11
NETMASK=255.255.255.0
GATEWAY=10.10.10.1
```

```
/etc/modprobe.d

# bonding.conf
alias bond0 bonding
opotions bond0 mode=4 miimon=100
```

```
modprobe bonding
systemctl restart network
```

#### 11.3.3.2 우분투에서 본드 설정 및 확인
```
# apt-get install ifeslave
```

```
# /etc/network/interfaces
auto eth0
iface eth0 inet manual
  bond-master bond0

auto bond0
iface bond0 inet static
  address 192.168.1.10
  gateway 192.168.1.1
  netmask 255.255.255.0
  bond-mode 4
  bond-miimon 100
  bond-slaves none
```


## 11.4 MC-LAG
- 서로 다른 스위치 간의 실제 MAC주소 대신 가상 MAC 주소를 만들어 논리 인터페이스로 구성

### 11.4.1 MC-LAG 동작 방식
- Peer 장비: MC-LAG 구성하는 장비
- MC-LAG 도메인: 두 Peer 장비를 하나의 논리 장비로 구성하기 위한 영역 ID
- Peer-Link: MC-LAG을 구성하는 두 Peer 장비 간의 데이터 트래픽을 전송하는 인터링크

설정 방법
- Peer에 동일한 Domain ID 설정
- Peer 간 데이터 트래픽 전송을 위한 피어 링크 설정
- Peer 간 제어 패킷 정송을 위해 Peer 끼리 통신 가능한 IP 설정


## 11.5. 게이트웨이 이중화
### 11.5.1. 게이트웨이 이중화
- 외부와의 통신 지점
- FHRP(First Hop Redudancy Protocol)

### 11.5.2. FHRP
- 게이트웨이 장비를 두대 이상으로 설정할수 있는 프로토콜
- 표준 프로토콜: VRRP(Virtual Router Redundancy Protocol)
	- HSRP(Hot Standby Router Protocol): 시스코사의 프로토콜

### 11.5.3. 올 액티브 게이트웨이 이중화

### 11.5.4. 애니캐스트 게이트웨이
