- [서버의 방화벽 설정 동작](#-------------)
	* [10.1 리눅스 서버의 방화벽 확인 및 관리](#101--------------------)
		+ [10.1.1 iptables 이해하기](#1011-iptables-----)
		+ [10.1.2 리눅스 방화벽 활성화/비활성화](#1012-----------------)
		+ [10.1.3 리눅스 방화벽 정책 확인](#1013--------------)
		+ [10.1.4 리눅스 방화벽 정책 관리](#1014--------------)
		+ [10.1.5 리눅스 방화벽 로그 확인](#1015--------------)
	* [10.2 윈도 서버의 방화벽 확인 및 관리](#102-------------------)

# 서버의 방화벽 설정 동작

![](files/Pasted%20image%2020250504160500.png)


- 운영체제도 멀쩡하고 네트워크 설정도 문제가 없는데 ping 응답이 날라오지 않는 경우도 있음


![](files/Pasted%20image%2020250504160506.png)


- 혹은 `netstat`이나 `ss` 명령어로 상태를 확인했더니 서버는 Listening 상태로 잘 동작
- 그러나 외부에서는 서버에 접속 불가능
- 네트워크 문제인가 싶어 ping을 날리면 ping 응답도 잘 옴
- 보안 장비 로그도 별 문제가 없고 서비스 요청이 정상 통과됨




- 이런 문제가 발생할 수 있는 건 서버에 설정된 방화벽 기능때문
- 서버에 운영체제 설치하면 기본적으로 운영체제의 자체 방화벽이 설치됨
- 서버 보안을 강화하기 위해 최소한의 서비스 포트만 열어둔 채 대부분의 서비스는 차단




- 방화벽은 필요한 IP 주소와 서비스 포트만 열어주고 나머지는 모두 차단하는 화이트 리스트 기반
- 필요하다면 주요 포트만 차단하는 블랙리스트 운용도 가능




![](files/Pasted%20image%2020250504160752.png)


- 서버 방화벽은 운용상의 불편함 때문에 끄고 쓰는 경우도 있지만 반드시 사용해야 하는 경우도 있다
- 데이터 센터 서버 접근 제한 및 이력 관리를 위해 사용하는 '서버 접근 통제 솔루션'이나 데이터베이스 서버의 접근 및 세부 쿼리에 대한 제어를 위해 사용하는 '데이터베이스 접근 통제 솔루션'을 사용하는 경우, 반드시 서버 방화벽에서 해당 솔루션의 IP 주소와 포트만 허용하고 나머지 주소는 차단해야 허용되지 않은 접근을 막을 수 있음


![](files/Pasted%20image%2020250504160906.png)


## 10.1 리눅스 서버의 방화벽 확인 및 관리
- 리눅스에서는 호스트 방화벽 기능을 위해 보통 `iptables`을 많이 사용
- CentOS 7 이상은 기본적으로 `firewalld`를 사용하도록 되어있고
- Ubuntu에서는 `UFW(Ubuntu FireWall`)을 사용


- 단, `iptables`에 익숙한 사용자가 많아서 여전히 `iptables`을 이용하는 경우가 많고
- UFW는 iptables의 프론트엔드 역할을 수행하므로 iptables를 알아야한다.


- iptables, firewalld와 netfilter의 관계
	- 실제 iptables는 방화벽의 역할처럼 패킷을 차단, 허용하는 등 필터링을 직접 수행하는 것은 아니다
	- 리눅스 커널에 내장된 netfilter라는 리눅스 커널 모듈을 통해 실제 필터링이 이루어짐
	- 즉, iptables은 netfilter를 이용할 수 있도록 해주는 프로그램
	- 결국 iptables나 firewalld, UFW 모두 netfilter에 대한 프론트엔드 역할


### 10.1.1 iptables 이해하기
- iptables을 이용해 서버에서 허용하거나 차단할 IP, 서비스 포트에 대한 정책 수립
- 이를 정책 그룹으로 관리


- 정책 그룹은 서버 기준의 트래픽 구간별로 만드는데
	- 트래픽 구간은 서버로 유입되는 구간(INPUT), 서버에서 나가는 구간(OUTPUT), 서버를 통과하는 구간(FORWARD) 등
- 이렇게 만들어진 방향성과 관련된 정책 그룹은 각 정책의 역할에 따라 다시 상위 역할 그룹에 속함


![](files/Pasted%20image%2020250504161415.png)


- 즉, 정책은 방향성에 따라 방향성과 관련된 정책 그룹으로 분류하고
- 분류된 그룹은 역할과 관련된 정책 그룹으로 묶임


- iptables에서 개별 정책의 방향성에 따라 구분한 그룹을 Chain이라고 하고
- 체인을 역할별로 구분한 그룹을 Table이라고 한다
- 즉, 개별 정책의 그룹이 체인이 되고 체인 그룹이 테이블이 된다.


![](files/Pasted%20image%2020250504161520.png)


- iptables에는 필터, NAT 테이블, Mangle 테이블, Raw 테이블, Security 테이블 5개 있다
- 패킷을 허용하거나 차단하는 데 사용하는 테이블이 필터 테이블
- 즉, 리눅스의 호스트 방화벽은 필터 테이블로 트래픽을 제어


- 필터 테이블에는 서버로 들어가는 트래픽, 나가는 트래픽, 통과하는 트래픽에 따라 INPUT 체인, OUTPUT 체인, FORWARD 체인 있다
- 각 체인에는 해당 체인이 적용될 방화벽 정책을 정의
- 각 정책에는 정책을 적용하려는 패킷과 상태 또는 정보 값과의 일치 여부 조건인 매치와 조건과 일치하는 패킷의 허용(Accept)이나 폐기(Drop)에 대한 패킷 처리 방식을 결정하는 타깃(Target)으로 구성된다.


![](files/Pasted%20image%2020250504161920.png)


- 즉, 정리하면 다음과 같다
	- Filter 테이블
		- iptables에서 패킷을 허용하거나 차단하는 역할을 선언하는 영역
	- INPUT, OUTPUT, FORWARD 체인
		- 호스트 기준으로 들어오거나 나가거나 통과할 때 사용되는 정책들의 그룹.
		- 패킷의 방향성에 따라 각 체인이 적용된 정책이 적용됨
	- Match
		- 제어하려는 패킷의 상태 또는 정보 값의 정의
		- 즉, 정책에 대한 조건
	- Target
		- Match와 일치하는 패킷을 허용할지, 차단할지에 때한 패킷 처리 방식


### 10.1.2 리눅스 방화벽 활성화/비활성화
- CentOS 7, IPv4 기준


- CentOS 7부터는 iptables이 기본적으로 없고 firewalld가 활성화되어 있어 firewalld를 비활성화하고 iptables를 설치해야 함


```
systemctl disable firewalld
systemctl stop firewalld
```
그리고 iptables 설치
```
yum install iptables-services
```
그리고 service 명령어나 systemctl 명령어로 iptables 서비스 활성화
```
service iptables start
systemctl start iptables.service
```
사실 CentOS 7에서는 service 명령어를 이용하면 리다이렉트됨
```
service iptables start
Redirecting to /bin/systemctl start iptables.service
```




### 10.1.3 리눅스 방화벽 정책 확인
![](files/Pasted%20image%2020250504162823.png)


- INPUT, FORWARD, OUTPUT 체인별로 구분된 정책을 확인할 수 있다
- 체인별로 기본값이 적용된 정책을 살펴볼 수 있는데, 외부 -> 서버에 사용되는 체인인 INPUT 살펴보자.


```
ACCEPT all -- anywhere anywhere state RELATED, ESTABLISHED
```


- `RELATED`, `ESTABLISHED` state인 모든 출발지를 허용
- 이미 세션이 맺어져 있거나 연계된 세션이 있는 경우 어떤 출발지나 목적지인 패킷이더라도 허용


- FTP 같이 컨트롤 프로토콜과 데이터 프로토콜이 달라서 방화벽에서 막아버릴 수 있음
- RELATE state를 이용해 이 두 가지 연결을 하나로 간주할 수 있음


```
ACCEPT icmp -- anywhere anywhere
```


- ICMP 허용


```
ACCEPT tcp -- anywhere anywhere state NEW tcp dtp:ssh
```


- 신규 세션인 NEW state 중 목적지 서비스 포트가 ssh인 경우만 허용
- 즉, 외부에서 서버로 SSH 접속을 허용


```
REJECT all -- anywhere anywhere reject-with icmp-host-prohibited
```


- 위 정책에 매칭되지 않은 패킷을 차단
- INPUT 체인 자체는 기본 정책이 ACCEPT로 선언되어 있지만
- 이 정책 떄문에 화이트리스트 기반 방화벽처럼 동작
- `REJECT`는 곧바로 폐기하는 `DROP`과 달리 ICMP 프로토콜을 이용해 패킷 차단 이유를 출발지에 전달
- 이때 `icmp-port-unreachable` 전달됨
	- `--reject-with` 옵션으로 ICMP 에러 유형 지정 가능
- iptables 기본 룰에서는 `icmp-host-prohibited` 메시지를 이용해 해당 패킷이 차단되었음을 알려줌


```
ACCEPT all -- anywhere anywhere
```


- 세 번째 정책
- 이것만 보면 모든 출발지의 모든 트래픽에 대해 허용하므로 Any Open 정책처럼 보인다
- 하지만 실제로 외부에서 들어오는 패킷은 해당 정책을 거치지 않고 최하단 DROP 정책에서 대부분 걸러짐
- `iptables`을 `-L` 옵션이 아니라 `-S` 또는 `--list-rule`로 다시 확인


```
[root@zigi ~]# iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```


```
-A INPUT -i lo -j ACCEPT
```
- 실제로는 모든 정책이 아니라 `lo`  루프백 인터페이스에 적용되는 정책임
- 즉, 루프백 인터페이스에 대한 모든 정책을 허용하는 것이니 일반 서비스 인터페이스 패킷에는 적용되지 않음


- 즉, iptables 정책을 선언할 때는 특정 인터페이스에 대해 적용하려는 게 아니라면 `-L`만으로도 현재의 정책을 확인할 수 있지만
- 실제 정책이 어떻게 정의되어 있는지 확인하려면 `-S`나 iptables 파일을 직접 확인해야 함


### 10.1.4 리눅스 방화벽 정책 관리
- iptables에 웹 서비스가 가능하도록 http 서비스 포트를 열어줘보자
```
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```
- 정책을 추가하려면 `-A`, `--append` 옵션을 사용
	- 옵션 뒤에는 어떤 체인에 적용할지 정의
- 출발지는 `--sport` 옵션을 사용


- 출발지나 목적지 IP 주소를 제어하기 위해 `-s`, `--source`, `-d`, `--destination` 옵션을 사용 가능
- 특정 IP 주소로 한정하거나 서브넷 마스크를 적용해 네트워크로도 적용 가능
- 별도 설정하지 않으면 anywhere 적용


- 패킷을 어떻게 처리할 것인지는 `-j` 옵션 사용


![](files/Pasted%20image%2020250504164713.png)


- 하지만 서버를 켜도 실제로는 아직 외부에서 접근할 수 없다.


![](files/Pasted%20image%2020250504164742.png)


- INPUT 체인을 보면 방금 추가한 정책이 맨 아래에 있다.
- 위에서부터 순서대로 확인하므로 되지 않던 것
	- 세부적인 정책 먼저 적용되는 게 아니라 상단 정책이 먼저 적용되는 탑다운 방식


- 웹 서비스가 차단되는 상태에서 웹 서비스를 허용하도록 변경하기 위해 방금 설정한 정책을 삭제하고 정책을 다시 추가
- iptables의 정책 삭제는 `-A` 대신 `-D` 또는 `--delete` 옵션을 사용


```
iptables -D INPUT -p tcp --dport 21 -j ACCEPT
```


- 삭제 후에는 `-L` 옵션으로 다시 확인
- 이때 정책이 너무 많으면 확인이 어려우므로 `-C`나 `--check` 옵션을 사용해 해당 정책이 있는지 확인 가능


```
$ iptables -C INPUT -p tcp --dport 21 -j ACCEPT
iptables: Bad rule (does a matching rule exist in that chain?)
```


- REJECT에 대한 정책을 삭제하고 등록할 수도 있지만 보안상 맨 마지막 룰은 항상 있어야하므로 REJECT 정책 위에 FTP 서비스를 허용하는 정책을 만들어보자
- 특정 위치에 정책을 추가하려면 정책의 줄 번호를 지정
- `-L` 옵션 뒤에 `--line-number` 옵션 추가하면 확인 가능


![](files/Pasted%20image%2020250504165230.png)


- 특정 위치에 정책 추가하려면 `-I`나 `--insert` 옵션 사용
- 해당 정책 줄 번호에 정책이 삽입되고 그 번호에 해당하는 정책부터 뒤의 모든 정책은 하나씩 뒤로 밀린다


```
iptables -I INPUT 5 -p tcp --dport 80 -j ACCEPT
```


- `-I`는 줄 번호가 필요하므로 정책이 추가될 체인 뒤에 줄 번호 입력


![](files/Pasted%20image%2020250504165406.png)


- 서버 보안을 강화하기 위해 서버 접근 통제와 같은 솔루션에서만 서버로 SSH 접속 가능하게 하고 나머지에 대해서는 SSH 접속 차단


```
iptables -A INPUT -i eth0 -p tcp -s 172.16.10.10/32 --dport 22 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp --dport 22 -j DROP
```


- IP Range에 대해서도 접근 제어를 걸 수 있다


```
iptables -A INPUT -p all -m iprange --src-range 192.168.0.0-192.168.255.255 -j DROP
```


- 서비스 포트도 범위 지정 가능하다


```
iptables -A INPUT -p tcp -m multiport --dports 3001:3010 -j DROP
```


- 혹은 여러 개를 나열하는 것도 가능하다


```
iptables -A INPUT -p tcp -m multiport --sports 4001,4003,4005 -j DROP
```


- 단, 출발지 포트는 랜덤이므로 출발지 포트를 지정하는 경우는 드물다.


- `-F` 옵션 사용하면 정책을 한꺼번에 삭제할 수 있다.


```
iptables -F
```


- `-P` 옵션은 각 체인의 기본 정책을 변경


```
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUITPUT DROP
```


- 단, 정책을 설정해도 서버를 재부팅하거나 서비스를 재시작하면 정책이 초기화된다
- 라우팅 테이블처럼 `iptables` 파일을 직접 설정해야 한다
- `/etc/sysconfig/iptables`


![](files/Pasted%20image%2020250504203714.png)


### 10.1.5 리눅스 방화벽 로그 확인
- 로그로 차단 혹은 허용된 내용을 확인할 수 있다
- `iptables`의 로그는 `/var/log/messages`에 남는다


```
tail -f /var/log/messages
```


- 메시지 파일에는 다른 로그도 포함되어 있다
- `iptables` 로그만 보려면 설정 필요
- `rsyslog.conf` 파일에 다음을 추가
	- `rsyslog`는 리눅스에서 사용하는 로그 수집 서버
```
kern.* /var/log/iptables.log
```


- `rsyslog` 서비스 재시작
```
systemctl restart rsyslog.service
```


- `iptables`에 로그를 남기도록 설정
- `warning` 수준의 로그를 남기기 위해 log-level을 4로 했고, 로그를 구분하는 식별자로 `## ZIGI-Log ##`를 설정


```
iptables -I INPUT -j LOG --log-level 4 --log-prefix '## ZIGI-Log ##'
```


- 특정 정책에 대한 로그만 남기고 싶다면 iptables에 정책을 추가할 때처럼 프로토콜이나 서비스 포트와 같은 옵션을 사용
- --log-prefix 없이도 로그를 남길 수 있지만 있으면 편하다.


![](files/Pasted%20image%2020250504203950.png)


- `iptables -L -v` 옵션 사용하면 통과하는 패킷과 바이트 수 확인 가능


- 이 외에도 IP 주소와 서비스 포트 외에도 다양한 인자 값을 이용해 복잡한 정책 수립 가능
- NAT 기능, 포트 포워딩, MSS 클램핑과 같은 고급 방화벽 기능도 있음
- 단, 서버 앞단에 방화벽이나 IPS, IDS 같은 보안 장비를 운용하고 있어 호스트 방화벽 자체를 내리는 경우도 많다
- 단, 내부망에서 악성코드가 전파되는 걸 막고 내부 보안 강화를 위해 단말에서도 복잡하지 않은 기본 방화벽 룰을 운영하는 것이 최근 추세
- iptables 관련 내용은 보안을 강화하고, 서비스 장애 발생 시 해결을 위해 꼭 알아둬야 하는 항목이다.


- iptables 더 알아보기
	- iptables은 Table, Chain, Target 3가지로 구성
	- Table
		- filter
			- 패킷 차단이나 허용
			- 들어오거나 나오는 트래픽 제어를 위해 INPUT, OUTPUT 체인 사용
			- 리눅스를 보안 장비로 사용하는 단말을 통과하는 트래픽을 제어할 수 있는 FORWARD 체인도 사용할 수 있음
		- nat
			- 출발지, 목적지 IP 변환
			- 출바지 주소 변경하는 Source NAT, 목적지 주소를 변경하는 Destination NAT 존재
			- Source NAT에는 POSTROUTING 체인 사용
			- Destination NAT에는 PREROUTING 체인 사용
			- 라우팅 작업에는 도착지 주소만 확인하므로 목적지 주소를 변경하는 Destination NAT는 라우팅 작업 이전에 수행되어 라우팅 작업을 거친 후 목적지로 전송
		- mangle
			- 패킷 헤더의 TOS, TTL 값 변경
			- 네트워크 보안 장비로 사용되는 게 아니라면 자주 사용되지 않음
		- raw
			- 연결 추적 시스템(Connection Tracking System)에서 처리하면 안 되는 패킷을 표시하는 용도
			- PREROUTING, OUTPUT 체인 존재
			- mangle처럼 자주 사용되지는 않음
		- security
			- 필수 접근 제어(Mandatory Access Control, MAC) 네트워크 규칙에 사용
			- 이는 SELinux와 같은 리눅스 보안 모듈에 의해 구현됨
			- filter 테이블 이후에 호출되며 filter 테이블처럼 INPUT, OUTPUT, FORWARD 체인 제공
	- Chain
		- 특정 패킷에 적용할 정책 정의
		- 패킷의 허용(Accept), 차단(Reject), 폐기(Drop)를 결정하는 정책 집합(A Set of Rules)
		- input chain, forward chain, output chain, prerouting chain, postrouting chain 등 다양한 chain 존재
		- input chain
			- 외부로부터 iptables가 동작하는 호스트로 패킷이 통과하는 체인
		- output chain
			- 호스트에서 나가는 패킷이 통과
		- forward chain
			- 목적지가 호스트가 아니라 패킷이 호스트를 통과할 때
			- 일반적인 서비스 목적의 호스트라면 통과 X
			- 라우터, 방화벽 등 네트워크에서 패킷 포워딩하는 기능을 제공하는 호스트라면 지정 가능
		- prerouting, postrouting
			- 라우팅 처리 전후
			- input chain의 경우 호스트로 들어와 라우팅 처리된 후 해당 체인이 동작
			- 라우팅 처리 전후로 패킷을 수정할 때 사용
	- Target
		- 정책이 일치할 때 취할 행동
		- ACCEPT
			- 패킷을 정상 처리
		- REJECT
			- 패킷을 폐기하고 패킷이 차단되었다는 응답 메시지 전송
		- DROP
			- 패킷을 그대로 폐기
		- LOG
			- 패킷을 syslog에 기록


- iptables 실행 옵션


| 옵션                | 설명                                            |
|---------------------|-----------------------------------------------|
| -A (--append)        | 새로운 규칙 추가                              |
| -D (--delete)        | 규칙 삭제                                     |
| -C (--check)         | 패킷 테스트                                   |
| -R (--replace)       | 새로운 규칙으로 교체                          |
| -I (--insert)        | 새로운 규칙 삽입                              |
| -L (--list)          | 규칙 출력                                     |
| -F (--flush)         | chain에서 규칙 삭제                           |
| -Z (--zero)          | 모든 chain의 패킷과 바이트 카운터 값을 0으로 만듦 |
| -N (--new)           | 새로운 chain을 추가                           |
| -X (--delete-chain)  | chain 삭제                                    |
| -P (--policy)        | 기본 정책 변경                                |


- 정책 옵션


| 옵션                  | 설명                              |
| ------------------- | ------------------------------- |
| -s (--source)        | 출발지 IP 주소나 네트워크와 매치             |
| -d (--destination)   | 목적지 IP 주소나 네트워크와 매치             |
| -p (--protocol)      | 특정 프로토콜과 매치                     |
| -i (--in-interface)  | 입력 인터페이스                        |
| -o (--out-interface) | 출력 인터페이스                        |
| --state             | 연결 상태와 매치                       |
| --string            | 애플리케이션 계층 데이터 바이트 순서와 매치        |
| --comment           | 커널 메모리 내의 규칙과 연계되는 최대 256바이트 주석 |
| -y (--syn)           | SYN 패킷 불허                       |
| -f (--fragment)      | 두 번째 이후의 조각 규칙 명시               |
| -t (--table)         | 처리될 테이블                         |
| -j (--jump)          | 규칙에 맞는 패킷 처리 방법 명시              |
| -m (--match)         | 특정 모듈과의 매치                      |


## 10.2 윈도 서버의 방화벽 확인 및 관리


