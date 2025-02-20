## 1장 네트워크 시작하기
### 1.1 네트워크 구성도 살펴보기
- 3계층 디자인
- 2계층 디자인

---
	- Spine-Leaf 아키텍쳐 
- [https://alluknow.tistory.com/40](https://alluknow.tistory.com/40 "https://alluknow.tistory.com/40")
- https://blog.naver.com/jkjk010jkjk/222239390955

### 1.2 프로토콜
- 물리적 측면: 데이터 전송 매체, 신호규약, 회선 규칙 등
- 논리적 측면: 장치들끼리 통신하기 위한 프로토콜 규격

- 비트기반: 효율성
- 텍스트기반: 확장성


---
- kubernetes 에서 mtu 관련 버그가 있어서 사이즈를 꽉채워서 보내면 버그가 있다.
	- https://github.com/kubernetes/kubernetes/issues/100212#issuecomment-800000542

- [https://www.fibermall.com/ko/blog/what-is-infiniband-network-and-difference-with-ethernet.htm](https://www.fibermall.com/ko/blog/what-is-infiniband-network-and-difference-with-ethernet.htm "https://www.fibermall.com/ko/blog/what-is-infiniband-network-and-difference-with-ethernet.htm")

### 1.3. OSI 7 계층과 TCP/IP
- 볼만한 글: https://www.fibermall.com/ko/blog/what-is-infiniband-network-and-difference-with-ethernet.htm

- [https://www.cisco.com/c/ko_kr/support/docs/routers/12000-series-routers/47321-ciscoef.html](https://www.cisco.com/c/ko_kr/support/docs/routers/12000-series-routers/47321-ciscoef.html "https://www.cisco.com/c/ko_kr/support/docs/routers/12000-series-routers/47321-ciscoef.html")
### 1.5. 인캡슐레이션과 디캡슐레이션
- 현재 계층에서 정의하는 정보
- 상위 프로토콜 지시자

- 이더넷에서 수용 가능한 최대 크기 1500
- MSS 1460
