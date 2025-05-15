# 11장 이중화 기술


### 이중화 기술 개요

이중화를 하지 않으면 SPoF(Single Point of Failure) 문제점이 있다.

**SPoF**

- 단일 장애점은 시스템 구성 요소 중 동작하지 않으면 전체 시스템이 중단되는 요소
- 예) 이더넷 네트워크 시스템에서 네트워크 허브 장치의 전원

인프라의 본질적인 목표는 가용성 높은 서비스에 필요한 인프라를 안정적으로 제공하는 것이므로, 단일 장애점이 전체 서비스에 영향을 주지 않도록 해야함.

**이중화의 목적**

- 서비스의 end to end에 속하는 모든 인프라에 이중화 구성을 고려해야함
<img width="459" alt="Screenshot 2025-05-15 at 7 18 59 PM" src="https://github.com/user-attachments/assets/dc82777f-5d90-4777-a00b-0aa852700b20" />



- 이중화된 구성요소 중 하나는 운영, 하나는 대기상태에 있다가
- 장애 발생 시 대기  상태가 운영으로 전환될지 여부에 따라 active-active, active-standby 형태로 구성된다.
- 특정 지점에 문제가 생겨도 서비스를 보장하므로 fault tolerance(FT)가 보장
    - 스위치의 STP는 단일 경로를 통해 트래픽을 통과하고 장애가 발생한 경우에만 차단된 경로를 활성화해 사용하므로 active-standby 형태
    - active-active의 경우 처리량이 늘기 때문에, 서비스를 이 기준으로 수용하다보면 장애가 발생 시 서비스가 정상적으로 운영되지 않을 수 있다.
- 데이터센터는 재난대비용으로 DR 센터를 별도로 두어서 이중화

---

### 1. LACP

벤더 별로 장비 간 대역폭을 늘리는 과정에서 호환성 문제가 발생하여 Link Layer가 표준화된 것이 LACP(Link Aggregation Control Protocol).

대역폭 확장을 통해 다음 2가지를 제공하고자 Link aggregation을 사용

- 링크 사용률 향상
- 향상된 장애 회복

LACP은 두 개의 물리 인터페이스로 구성된 논리 인터페이스를 이용해 모든 물리 인터페이스를 액티브 상태로 사용한다.

(물리 포트 여러개를 묶어서 하나의 인터페이스로 사용하되, 내부적으로는 두 포트 모두 트래픽을 받아서 처리하는 형태)

- 스위치 - 스위치, 혹은 스위치 - 서버 간의 네트워크 대역폭이 물리 인터페이스 수만큼 확장된다.
- 다만 active-active 구조에서 논리 인터페이스 대역폭을 서비스에 필요한 전체 트래픽 기준으로 산정하면 안된다.
    - 1.5G를 수용해야할 때 1G,1G 2개를 active로 쓴다고 했을 때, 하나가 장애가 발생하면 1G가 정상적으로 서비스를 수용하지 못하기 때문
- **또한 LACP으로 구성하는 물리 인터페이스 속도가 동일해야한다. (1G와 10G로 LACP 을 구성할 수 없음)**

**LACPDU**

LACP을 통해 장비 간 논리 인터페이스를 구성하기 위해 LACPDU 프레임을 사용

- 출발지 주소, 목적지 주소, 타입, 서브타입, 버전 등을 포함한 프레임
- 멀티캐스트 이용, 목적지는 01.80.c2:00:00:02 ~ 01.80.c2:00:00:10
- LCAP을 연결하려면 LACPDU 를 주고받는 장비가 한 장비여야하는데,
- LCAP을 통한 두 개 이상의 물리 인터페이스가 서로 다른 장비에 연결되어있으면 링크 이중화구성이 되지 않는다.

<img width="431" alt="Screenshot 2025-05-15 at 7 19 30 PM" src="https://github.com/user-attachments/assets/1cd1ed62-3f77-4be2-abf8-681765fb5fa6" />

- 또한 두 장비 모두 LACP 설정을 해야 LACPDU를 주고받을 수 있다.

<br>

LACP을 설정할 때는 액티브 모드와 패시브 모드 2가지가 있음.

- 액티브 : LACPDU를 먼저 송신하고, 상대방이 LACP으로 구성된 경우 LACP을 구성
- 패시브 : LACPDU를 수신받으면 응답하여 LACP을 구성

LACP을 구성한 모든 장비가 LACPDU를 보내는 것은 아니지만, 단뱡향으로라도 받으면 LACP 연결이 가능하다.

<img width="468" alt="Screenshot 2025-05-15 at 7 20 05 PM" src="https://github.com/user-attachments/assets/f2ab3979-790b-421d-9b89-3f794e8fb18f" />


**LACP와 PXE**

- LACP 처럼 서버의 인터페이스를 하나의 논리 포트로 묶는 bonding과 teaming 기술은 서버 운영체제 레벨에서 설정
- 하지만 PXE(Pre-boot eXecution Environment)는 서버가 운영체제를 설치하기 이전이므로 논리 인터페이스를 설정할 수 없다.
    - PXE는 네트워크 레벨에서 OS를 부팅할 수 있는 표준 프로토콜로, 네트워크 카드(NIC)가 PXE 코드를 실행하고 DHCP 서버에 정보를 요청하여 OS를 설치할 수 있음.
    - <img width="344" alt="Screenshot 2025-05-15 at 7 30 20 PM" src="https://github.com/user-attachments/assets/a838bd01-a3ff-4c52-8214-d22e44f991ab" />
    - LACPDU를 수신할 수 없어 정상적으로 활성화되지 않는 셈
- PXE로 운영체제를 설치할 때는 일반 인터페이스로 설치한다음, LACP 설정 후 스위치 포트 설정을 다시 해야하는 문제가 있음.


⇒ 이를 해결하기 위해 일정 시간동안 LACPDU를 수신하지 못하면, 1개 인터페이스만 활성화하고 수신받으면 두 개 모두 활성화하는 옵션을 제공한다.

- 서버에 운영체제가 설치되기 전이면 LACPDU를 수신하지 못하지만, 인터페이스 하나만 활성화해서 PXE boot 를 실행
- 그리고 운영체제가 설치되고 나면 LACPDU를 주고받고, LACP로 구성 변경

** Cisco에서는 lacp suspend-individual 

---

### 2. 서버 네트워크 이중화 설정(Linux)

- 리눅스는 bond, binding을 사용
    - 리눅스7부터는 teaming 기술도 추가됨
    
    [8.3. 네트워크 티밍과 본딩 비교 | Red Hat Product Documentation](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/networking_guide/sec-comparison_of_network_teaming_to_bonding#sec-Comparison_of_Network_Teaming_to_Bonding)
    
- 서버 인터페이스를 이중화하면 네트워크 장비도 마찬가지로 논리 인터페이스가 생성된다.

서버 인터페이스 이중화 구성은 네트워크와 달리 여러개의 동작모드가 있다.
<img width="457" alt="Screenshot 2025-05-15 at 7 20 31 PM" src="https://github.com/user-attachments/assets/3189a7cd-1fde-45f1-b240-3fe3acc6d688" />



**리눅스 본딩모드**

0~4가 있으나 보통 액티브 스탠바이 모드 1과 액티브 액티브 모드 4를 사용하고 나머지는 잘 사용되지 않는다.

- 모드 1 : active-standby
    - 액티브가 죽으면 스탠바이 인터페이스가 자동으로 활성화
    - 원래의 액티브가 살아나면 다시 활성화되거나, 수동으로 넘기기 전까지 스탠바이가 활성화 상태를 유지
- 모드 4: LACP
    - 표준 프로토콜인 LACP을 사용하여 active-active 사용

CentOS에서 이러한 본드 설정을 하기 위해서는 

- 네트워크 설정파일이 있는 디렉터리 (/etc/sysconfig/network-scripts)에
- bond 인터페이스 파일인 ifcfg-bond0을 생성하여
    - bond로 묶일 인터페이스에 추가속성을 설정하고
- 물리 인터페이스 파일(ifcfg-eth0, ifcfg-eth1)에도 속성을 추가하여
- bonding 설정파일인 bondig.conf에서  본딩모드와 옵션을 설정하면 된다.
<img width="464" alt="Screenshot 2025-05-15 at 7 20 41 PM" src="https://github.com/user-attachments/assets/33cfdf91-a7b2-456a-bd64-080550851fd5" />


→ bond 인터페이스를 설정하거나 수정 한 뒤에는 네트워크를 재시작해주자.

```jsx
systemctl restart network
// 본드 인터페이스가 잘 적용되었는지 확인
cat /proc/net/bonding/bond0
```

---

### 3. MC-LAG

- LACP을 구성할 때는 LACPDU 프로토콜을 주고받는 장비, 즉 mac 주소가 1:1이어야한다.
- 그래서 서버에서 본딩같은 이중화 구성을 할 때 각 네트워크 카드 별 물리 mac 주소를 별도로 사용하지 않고
- 두 개의 물리 mac 주소 중 하나를 primary로 사용한다.

- 서버는 인터페이스를 2개 이상 구성해도, 상단 스위치가 하나로 구성되어있으면 SPoF이기 때문에
- 서버 인터페이스를 서로 다른 스위치로 연결하는데, 이는 두 스위치 간 mac주소가 달라서 LACP을 사용할 수 없다.
- 그래서 **본딩 모드를 active-standby로 구성해야한다.**
- (액티브-액티브를 사용할 수 없는 것인가?)
    - 이 구성만으로는 액티브-액티브를 사용할 수 없음
    - LACP은 하나의 논리 스위치로 인식되어서 직접 LACPDU를 주고받을 수 있어야 aggregate해서 처리할 수 있는데
    - 스위치가 여러 개가 되면 스위치끼리 주고받을 수 없기 때문

⇒ 그래서 MC-LAG(Multi chassis link aggregate group) 기술로 서로 다른 스위치 간의 단말 mac 주소를 사용해 액티브-액티브 모드를 사용할 수 있다.
- 가상 mac 주소를 만들어 논리 인터페이스로 LACP을 구성
<img width="459" alt="Screenshot 2025-05-15 at 7 25 18 PM" src="https://github.com/user-attachments/assets/ea852bfd-6b7c-45ed-8729-d2a76f591075" />




**MC-LAG 동작방식**

벤더마다 세부적인 내용은 조금씩 다르지만, 기본적인 동작방식을 알아보자.

MC-LAG은 아래와 같은 장비로 구성되어있다.

- Peer 장비
- MC-LAG 도메인
    - 두 peer 장비를 하나의 논리 장비로 구성하기 위한 영역 ID
    - 상대방 자장비가 peer을 맺으려는 장비인지 해당 ID값을 통해 판단
- Peer-Link
    - 두 peer 장비 간의 데이터 트래픽을 전송하는 인터링크
<img width="466" alt="Screenshot 2025-05-15 at 7 25 30 PM" src="https://github.com/user-attachments/assets/d876e7e8-4edf-406b-8a5d-4b568896ac90" />

즉, Peer 장비 2개는 peer link를 통해 연결되어있고, 각 장비들은 하나의 MC-LAG 도메인으로 묶이게 된다.

<br>

- 피어링크는 다양한 네트워크가 통신할 수 있는 경로이므로 보통 Trunk로 구성
- MC-LAG은 피어 장비 간 제어 패킷을 주고받아야하는데,
- 피어 간 데이터 트래픽 전송을 위한 인터링크(피어링크)를 사용할지
    - 각 피어의 VLAN 인터페이스 ip를 설정하여 통신
- 피어 간 제어 패킷을 위해 피어끼리 통신 가능한 경로를 구성할것인지 선택할 수 있다.
    - 인터페이스를 L3 인터페이스로 구성해 ip 통신
<img width="468" alt="Screenshot 2025-05-15 at 7 25 48 PM" src="https://github.com/user-attachments/assets/19c43774-e7e8-4514-aa41-e9ec981b4ae6" />



MC-LAG 제어 패킷을 주고받아서 협상이 정상적으로 완료되면

- 두 대의 장비는 하나의 도메인으로 묶이고
- 인터페이스 이중화 구성에 사용될 가상 mac 주소를 피어 장비간에 동일하게 생성하게 된다.

⇒ 이 구성까지 마치면 LACP 사용이 가능해짐.

두 장비 간에 LACP을 구성할 때는 각 장비의 mac 주소가 출발지 주소가 되지만, 

MC-LAG의 경우 가상 mac 주소를 사용해서 LACPDU를 전송한다.

<br>

**MC-LAG을 이용한 디자인**

MC-LAG을 사용하면 서로 다른 스위치로 서버를 액티브-액티브로 구성해서,

루프나 STP(스패닝 트리 프로토콜)에 의해 block 되지 않는 네트워크 구조를 만들 수 있다.

1. 스위치를 물리적으로 이중화하여 액티브-액티브
2. 스위치 간 MC-LAG을 사용하면 STP 에 의해 차단되는 포트 없이 모든 포트 사용(루프 구조가 사라짐)
3. 
4. 상 하단을 모두 MC-LAG으로 구성해서 스위치 4대를 1:1 구조로 구성

---

### 4. 게이트웨이 이중화

- 특정 호스트가 동일한 서브넷에 있는 내부 네트워크와 통신할 때는 ARP를 브로드캐스트하여 직접 통신(L2 통신)
- 목적지가 외부 네트워크인 경우, 게이트웨이를 통해야함 (L3 통신)
    - 호스트에 게이트웨이 설정이 잘못되거나 안되어있으면 외부 네트워크와는 통신되지 않는 셈

그래서 게이트웨이 장비에 장애가 발생하면 외부 네트워크와 통신할 수 없어 이중화가 필요한데, 실제 물리적으로 외부 네트워크와 통신할 수 있는 경로가 이중화되어있어도 하단 호스트들은 하나의 게이트웨이만 보기 때문에 통신할 수 없는 상태가 된다.
<img width="449" alt="Screenshot 2025-05-15 at 7 37 22 PM" src="https://github.com/user-attachments/assets/9f11085b-2a74-4559-a51e-afaa532e4e0d" />
(게이트웨이 간 ip 주소가 다른 상태로 이중화)

<br>

두 대의 장비가 하나의 ip 주소를 가리키는 FHRP(Frist Hop Redundancy Protocol)을 사용하면 LACP처럼 게이트웨이를 이중화할 수 있다.

- 실제 ip 외에 추가로 가상 ip 주소와 mac 주소를 동일하게 갖게됨
- 가상 ip는 그룹 내 우선순위가 높은 장비를 active로 유지하고, ARP에 응답
- active에 문제가 생기면 standby 장비가 active로 변경되어서 통신할 수 있음.

**FHRP 기술**

외부 네트워크와 통신하기 위한 게이트웨이 장비를 2대 이상으로 구성하는 프로토콜

- 동일한 그룹으로 인식하기 위해 같은 그룹ID를 설정
- 게이트웨이 주소로 사용될 가상 IP도 각 장비에 설정
    - 가상 IP에 대한 mac 주소도 동일하게 갖게됨
- 하단 호스트가 가상IP로 ARP 요청을 보내면, 브로드캐스트로 모든 장비가 받아서 액티브 장비가 ARP에 응답한다.
<img width="457" alt="Screenshot 2025-05-15 at 7 37 44 PM" src="https://github.com/user-attachments/assets/b191df7d-5ff0-4459-b78f-7dac6b067a38" />



장애가 발생하면 스탠바이 장비가 액티브 장비가 비정상임을 체크하고, 액티브 역할을 수행한다.

- 그리고 가상 ip와 mac 주소 테이블을 갱신해두기 때문에
- 하단 호스트들은 아무 설정변경 없이 통신할 수 있다.

**VRRP**

FHRP 기술의 표준 프로토콜로, 게이트웨이 이중화 기술로서 모든 벤더 장비가VRRP 기능을 지원

- VRRP의 동작방식
    - 동일한 VRID 값을 설정한 장비가 하나의 VRRP 그룹이 됨
    - VRRP의 마스터를 선출하기 위해 장비 간 hello 패킷을 주고받고,
    - 패킷의 우선순위를 비교해서 액티브가 될 마스터 장비를 선정
    - 1초마다 전달되는 hello 패킷을 3회 이상 수신하지 못하면 비정상으로 간주

- 마스터 스위치 장비에 장애가 발생하면
    - 다른 스위치가 마스터 역할을 가져가면서, 가상IP와 가상mac 주소를 스위치 B에서 광고하여 mac 테이블은 갱신
    - 하지만 주소 자체가 변경된 것은 아니므로 ARP 테이블은 변경되지 않음

- 설정 예제(시스코)
    - VRRP 설정을 위해 해당 장비의 real IP 주소를 설정하고,
    - 1~255 사이의 VRRP ID 값을 설정한다. 같은 네트워크 간에는 중복되지 않도록 설정해야한다.
    - VRRP 그룹 장비의 하단 호스트는 가상 IP 주소를 게이트웨이 주소로 설정
    - VRRP 액티브/스탠바이 선출을 위해 우선순위 값을 선정
        - 값이 클수록 우선순위가 높다.
        - preempt는 스탠바이 장비가 액티브보다 우선순위가 높아지면, 액티브를 자동으로 가져오는 기능
            - 이 설정이 있으면서 액티브 장비에 계속 문제가 생기면 VRRP 주인이 계속 바뀌는 flapping 현상이 발생할 수 있음
            - 그래서 우선순위가 바뀌어도 일정 시간 기다렸다가 owner을 가져오는 옵션을 고려해야한다.
    - VRID는 장비 간 멀티캐스트 주소(244.0.0.18)로 전달
        - 가상 IP도 mac 주소가 필요하기 때문에 동일 네트워크 내에서 VRID가 중복되면 안된다.
    
    ---
    

**올 액티브 게이트웨이 이중화**

액티브-스탠바이 구조에서는 모두 액티브 장비를 통해서 가야하므로 불필요하게 트래픽을 우회하게 될 수 있다.

- MC-LAG 기술은 가상IP와 mac 주소를 액티브, 스탠바이 모두 사용 가능하게끔 기능을 제공함
- 액티브-액티브로 구성하는 셈
<img width="454" alt="Screenshot 2025-05-15 at 7 37 57 PM" src="https://github.com/user-attachments/assets/9f1e2ace-ff32-4596-b768-1e294c36263a" />



- 애니캐스트 게이트웨이
    - 각 위치에 같은 주소를 가지는 게이트웨이 여러개가 동작할 수 있음
    - 애니캐스트만 사용하는 게이트웨이
    - 같은 IP를 가진 게이트웨이가 존재하지만, 가장 가까운 위치에 있는 게이트웨이에서 서비스를 제공한다.
    - aws global accelerator
    

** 참고

동일한 가상화 서버에 있어도, 서로 다른 네트워크를 갖고 있다면 가상화 서버 외부에 있는 게이트웨이를 거쳐야만한다.

- 비슷하게 동일한 스위치에 구성된 서버 간 통신도, 네트워크가 다르면 외부 게이트웨이를 거쳐야한다.
- 하지만 최근에는 GW 역할을 가상화 서버의 가상 스위치에서 수행하거나
- 각 ToR 스위치에서 수용할 수 있도록 기능을 제공하기도 함
<img width="459" alt="Screenshot 2025-05-15 at 7 38 05 PM" src="https://github.com/user-attachments/assets/1c1d9f36-2cf5-468c-a6c1-144dde6e74fa" />


