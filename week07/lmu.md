## 7장 통신을 도와주는 네트워크 주요 기술
### 7.1. NAT/PAT
- NAT: Network Address Translation
- NAPT (Network Address Port Translation, RFC2663)
- PAT (Port Address Translation)

#### 7.1.1. NAT/PAT의 용도와 필요성
- IPv4 주소 고갈문제
- 보안 강화
- IP 주소 체계가 같은 두 개의 네트워크 간 통신을 가능하게 해준다.
- 불필요한 설정 변경을 줄일수 있다.

#### 7.1.2. NAT 동작 방식

#### 7.1.3 PAT 동작 방식

#### 7.1.4. SNAT와 DNAT
- SNAT(Source NAT) : 출발지 주소를 변경하는 NAT
- DNAT(Destination NAT) : 도착지 주소를 변경하는 NAT
- ref. https://blog.naver.com/alice_k106/221305928714
- DNAT: PREROUTING
- SNAT: POSTROUTING

#### 7.1.5. 동적 NAT와 정적 NAT

### 7.2. DNS
### 7.2.1 DNS 소개
### 7.2.2. DNS 구조와 명명 규칙
- e.g. www.naver.com
- Third-Level Domain: www
- Second-Level Domain: naver
- Top-Level Domain (TLD): com
- root: . 은 보통 생략

#### 7.2.2.1. 루트 도메인
a.root-server.net ~ m.root-servers.net

#### 7.2.2.2. Top-Level Domain (TLD)
- Generic (gTLD)
	- com, edu, gov, int, mil, net, org
- country-code (ccTLD)
	- kr
- sponsored(sTLD)
	- 특정 목적을 위한 도메인
	- aero, asia, edu, museum 등
- infrastructure
	- .arpa: 인터넷 안정성을 유지하기 위함
- generic-restricted(grTLD)
	- 특정 기준을 충족하는 사람이나 단체가 사용할수 있는 도메인
	- .blz, .name, .pro
- test(tTLD)
	- 테스트 목적, .test

### 7.2.3. DNS 동작 방식

- hosts 파일
- DNS local cache
- DNS 서버 쿼리
- 재귀적 쿼리(Recursive Query) vs 반복적 쿼리 (Iterative Query)

### 7.2.4. 마스터와 슬레이브
- 엑티브-스탠바이
- 엑티브-엑티브

### 7.2.5. DNS 주요 레코드
- A: 도메인 주소를 IPv4 로 매핑
- AAAA: 도메인 주소를 IPv6 로 매핑
- CNAME: 도메인 주소에 대한 별칭
	- <img width="715" alt="Pasted image 20250410024044" src="https://github.com/user-attachments/assets/a2171c58-c8d5-452e-84ba-6ee20b7ba689" />

- SOA: 본 영역에 대한 권한
- NS: 본 영역에 대한 네임 서버
- MX: 도메인에 대한 메일 서버 정보
- PTR: IP 주소를 도메인에 매핑 (역방향)
- TXT: 도메인에 대한 일반 텍스트

### 7.2.6. DNS에서 알아두면 좋은 내용
#### 7.2.6.1. 도메인 위임(DNS Delegation)

- CDN이나 GSLB를 사용하는게 제일 일반적

#### 7.2.6.2. TTL
- 윈도우: 1시간
- 리눅스: 3시간

#### 7.2.6.3. 화이트 도메인
- 대량으로 이메일을 발송할 경우 스팸메일로 차단된다.
- KISA에 화이트 도메인을 등록해야한다

#### 7.2.6.4 한글 도메인
- 퓨니코드로 드록한 이후 이걸 DNS 에 도메인을 생성해야한다
- 퓨니코드: 클라이언트 단에서 퓨니코드로 변환되어서 전송된다. (xn--으로 시작하는 문자열)
- RFC 3492 에 정의되어 있다.

### 7.2.7 DNS 설정 (windows)
### 7.2.8 DNS 설정 (Linux)
- bind 패키지 사용

### 7.2.9 호스트 파일 설정

### 7.3. GSLB
### 7.3.1. GSLB 동작 방식

### 7.3.2. GSLB 구성방식
- 도메인 자체를 GSLB로 사용
- 도메인 내의 특정 레코드만 GSLB를 사용

### 7.3.3. GSLB 분산 방식
- 서비스 제공의 가능 여부를 체크해 트래픽 분산
- 지리적으로 멀리 떨어진 다른 데이터 센터에 트래픽 분산
- 지역적으로 가까운 서비스에 접속해 더 빠른 서비스 제공이 가능하도록 분산


## 7.4. DHCP
### 7.4.1 DHCP 프로토콜
- BOOTP(Bootstrap Protocol)을 기반으로 한다.

### 7.4.2. DHCP 동작 방식
1. DHCP Discover: DHCP 서버를 찾기 위한 브로드캐스트
2. DHCP Offer : DHCP 서버가 DHCP서버 정보를 클라이언트로 전송
3. DHCP Request :  DHCP 서버로부터 제안 받은 주소를 브로드캐스트로 전송
4. DHCP Acknowledgement: DHCP 클라이언트로부터 IP 주소를 사용하겠다는 요청을 받으면 DHCP Request를 잘 받았다는 신호로 응답 전송

- 위 과정중에 왜 3번 과정 DHCP Request 는 Dest MAC 이 Broadcast 인가를 고민...
	- <img width="796" alt="Pasted image 20250410025815" src="https://github.com/user-attachments/assets/a4abbe5b-805b-4c0c-8740-f2a0b94c6de0" />

	- https://blog.naver.com/jga0674/221004234317
	- DHCP 서버가 여러개일수 있는데, 어떤게 선택되었는지를 전파하기 위해서 브로드캐스트여야한다
	- 


### 7.4.4. DHCP 릴레이
