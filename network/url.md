## URI와 웹 브라우저 요청 흐름
### **URI, URL, URN**

URI(Uniform Resource Identifier)

URI는 로케이터, 이름 또는 둘 다 추가로 분류될 수 있다

Uniform : 리소스 식별하는 통일된 방식

Resource : 자원, URI로 식별할 수 있는 모든 것(제한 없음)

Identitfier : 다른 항목 구분하는데 필요한 정보

URL은 리소스 위치

URN은 리소스 이름

**위치는 변할 수 있지만, 이름은 변하지 않는다**

### **웹 브라우저 요청 흐름**

1. DNS 조회
2. ip port 정보 찾아냄
3. 애플리케이션에서 http 요청 메시지 생성
4. Socket 라이브러리를 통해 TCP/IP로 구글과 연결 후 데이터를 OS에 전송
5. TCP/IP 패킷 생성 (http 메시지 - 전송 데이터, 출발지 목적지 ip port 정보 담겨있음)
6. 요청 패킷 전달
7. 도착하면 요청 패킷을 까서 버리고 http 메시지를 보고 해석
8. 응답 메시지 생성해서 tcp ip 패킷 씌우고 응답 패킷으로 전달 (응답 메시지엔 status code, content-type, length, html 데이터 등등)
9. 응답 패킷을 까서 웹 브라우저 html 렌더링