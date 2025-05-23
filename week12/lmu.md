
# 12장 로드 밸런서
## 12.1 부하 분산이란?
- SLB (Server Load Balancer)
- FWLB(Firewall Load Balancing)

## 12.2 부하 분산 방법
- VIP, RIP

## 12.3 헬스 체크
### 12.3.1. 헬스 체크 방식
#### 12.3.1.1. ICMP
#### 12.3.1.2. TCP Service Port
#### 12.3.1.3. TCP Service Port: Half Open
- 헬스 체크로 인한 부하를 줄이거나 정상적인 종료 방식보다 빨리 헬스 체크 세션을 끊기 위해 정상적인 3Way handshake 나 4way handshake 가 아닌 TCP Half Open 방식을 사용하기도 한다.
- 초기의 3way handshake 와 동일하게 sync 를 보내고 syn, ack 을 받지만 이후 ack 대신  rst를 보내 세션을 끊난다.

#### 12.3.1.4. HTTP Status Code

#### 12.3.1.5. 콘텐츠 확인 (문자열 확인)
- 비정상적인 상태더라도 잘못해서 정상으로 판단할수 있으니, Status Code 도 활용하거나, 비일반적인 문자열을 사용한다.


### 12.3.2.  헬스 체크 주기와 타이머
- 주기(Interval): 로드 밸런서에서 서버로 헬스 체크 패킷을 보내는 주기
- 응답 시간(Response): 로드 밸런서에서 서버로 헬스 체크 패킷을 보내고 응답을 기다리는 시간 해당 시간까지 응답이 오지 않으면 실패로 간주
- 시도 횟수(Retries): 로드 밸런서에서 헬스 체크 실패 시 최대 시도 횟수. 최대 시도 횟수 이전에 성공시 시도 횟수는 초기화됨
- Timeout: 로드 밸런서에서 헬스 체크 실패시 최대 대기시간. 헬스 체크 패킷을 서버로 전송한 후 이 시간 내에 성공하지 못하면 해당 서버는 다움
- 서비스 다운 시의 주기(Dead Interval): 서비스의 기본적인 헬스 체크 주기가 아닌, 서비스 다운 시의 헬스 체크 주기. 서비스가 죽은 상태에서 헬스 체크 주기를 별도로 더 늘릴 때 사용


## 12.4. 부하 분산 알고리즘
- Round Robin: 현재 구성된 장비에 부하를 순차적으로 분산함. 총 누적 세션 수는 동일하지만 활성화된 세션 수는 달라질 수 있음
- Least Connection: 현재 구성된 장비 중 가장 활성화된 세션 수가 적은 장비로 부하를 분산함
- Weighted Round Robin: 각 장비에 가중치를 두어 가중치가 높은 장비에 부하를 더 많이 분산함. 처리 용량이 다른 서버에 부하를 분산하기 위한 분산 알고리즘
- Weighted Least Connection
- Hash: 해시 알고리즘을 이용한 부하 분산
	- 세션을 유지해야하는 서비스에 적합한 분산 방식
	- Unbalanced 할수 있음

## 12.5. 로드 밸런서 구성 방식
- One-Arm
- Inline

## 12.6. 로드 밸런서 동작 모드
### 12.6.1. Transparent, Bridge
- OSI 2계층 스위치 처럼 동작하는 구성

### 12.6.2. Routed
- 로드밸런서가 라우팅 역할을 하는 구조

### 12.6.3. DSR모드
- 사용자의 요청이 로드 밸런서를 통해서 서버로 유입된 후 로드밸런서를 통하지 않고 서버가 사용자에게 직접 응답하는 모드
- 로드 밸런서의 부하가 감소한다.


## 12.7 로드 밸런서 유의사항
### 12.7.1. 원암 구성의 동일 네트워크 사용시


## 12.8 HAProxy를 사용한 로드밸런서 설정
