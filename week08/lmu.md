# 8장 서버 네트워크 기본
## 8.1. 서버의 네트워크 설정 및 확인

#### 8.1.1. 리눅스 서버 네트워크
- centos: `/etc/sysconfig/network-scripts`
- ubuntu: `/etc/network/interfaces`

## 8.2. 서버의 라우팅 테이블
### 8.2.1. 서버의 라우팅 테이블
- Destination : 목적지
- Genmask : 서브넷
- Gateway : 게이트웨이
- Iface : 인터페이스
- Metric : 우선순위
- 

### 8.2.2. 리눅스 서버의 라우팅 확인 및 관리
- 참고자료: https://pr0gr4m.github.io/linux/kernel/ipv4_routing/

```
ip route
```

```
route add { -host | -net } Target[/prefix] [gw Gw] [metric M] [[dev] If]
```

```
route del { -host | -net } Target[/prefix] [gw Gw] [metric M] [[dev] If]
```


#### 8.2.1.1 CentOS 의 영구적 라우팅 설정
```
/etc/sysconfig/network-scripts/route-장치명
```

#### 8.2.2.2. Ubuntu의 영구적 라우팅 설정
`
`/etc/network/interfaces`

```
up route add [ -net | -host] <host/net>/<mask> gw <host/IP> dev <Interface>
```

## 8.3. 네트워크 확인을 위한 명령어
### 8.3.1. ping (Packet InterNet Grouper)
- 살짝 명령어 먼저 만들고 나중에 끼워 맞춘거 같은데...

### 8.3.3. traceroute(리눅스)

### 8.3.4. tcptraceroute

### 8.3.5. netstat (network statistics)
- ss, ip route, ip -s link, ip maddr

### 8.3.6. ss(socket statistics)

### 8.3.7 nslookup(name server lookup)

### 8.3.8. telnet(tele network)

### 8.3.9. ipconfig

### 8.3.10. tcpdump
