## 2장 네트워크 연결과 구성 요소
### 2.1. 네트워크 연결 구분
- LAN, MAN, WAN
	- 예전에는 LAN, MAN, WAN 마다 사용하는 기술이 달라 프로토콜이나 전송 기술에 따라 쉽게 구분할 수 있었다. 하지만 요즘은 이를 통한 구분은 무의미하고, 관리 범위 기준으로 구분한다.

### 2.2. 네트워크 회선
#### 2.2.1. 인터넷 회선
- 광랜: 기가 ~ 100Mbps
- FTTH(Fibber To The Home): 기가 ~ 100Mbps
- 동축 케이블 인터넷: 수백 ~ 수십 Mbps
- xDSL(ADSL, VDSL 등): 수십 ~ 수 Mbps

#### 2.2.2. 전용 회선
- 저속: 음성 전송 기술 기반
- 고속: 메트로 이더넷

#### 2.2.3. 인터넷 전용 회선

#### 2.2.4 VPN
##### 2.2.4.1. 통신사업자 VPN
##### 2.2.4.2. 가입자 VPN

- VPN 관련 글
	- https://tech.devsisters.com/posts/wireguard-vpn-1/
	- https://blog.naver.com/simula/223771392775?trackingCode=rss
  - <img width="822" alt="Pasted image 20250227002824" src="https://github.com/user-attachments/assets/b1d4364b-72b4-4950-9c78-0ca62931a8b4" />

- 개인적으로는 VPN클라이언트들은 하나같이 좀 답답한 측면이 있는거 같지만... 회사에서는 어쩔수 없나 싶다.
- https://kimalarm.tistory.com/69 이거 봤는데, AWS 는 신이다. 이런것도 지원해주다니... 
#### 2.2.5. DWDM
- Dense Wavelength Division Multiplex 
- https://www.fibermall.com/ko/blog/cwdm-dwdm-mux-demux-technology.htm

### 2.3. 네트워크 구성요소
### 2.3.1. 네트워크 인터페이스 카드 (NIC)
- 직렬화
- MAC 주소
- 흐름 제어


### 2.3.2. 케이블과 커넥터
#### 2.3.2.1. 이더넷 네트워크 표준

#### 2.3.2.2 케이블 커넥터 구조
#### 2.3.2.3. 케이블 - 트위스티드 페어 케이블

#### 2.3.2.4. 케이블 - 동축 케이블

### 2.3.3. 허브
### 2.3.4. 스위치
### 2.3.5. 라우터

### 2.3.6. 로드밸런서
- DSR
- Proxy
- http://redisgate.kr/redis/network/load_balancing.php
- https://delightwook.tistory.com/39
- https://sysdocu.tistory.com/1369

### 2.3.7. 보안 장비(방화벽/IPS)
### 2.3.8 기타 (모뎀/공유기)
