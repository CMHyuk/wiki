### Network

- 컴퓨터나 기타 기기들이 리소스를 공유하거나 데이터를 주고 받기 위해 유선 혹은 무선으로 연결된 통신 체계
- 애플리케이션 목적에 맞는 통신 방법 제공
- 신뢰할 수 있는 데이터 전송 방법 제공
- 네트워크 간의 최적의 통신 경로 결정
- 목적지로 데이터 전송
- 노드 사이의 데이터 전송

통신 기능이 제대로 동작하기 위해서는 참여자들 사이에서 약속된 통신 방법이 있어야 함

### **네트워크 프로토콜**

네트워크 통신을 하기 위해서 통신에 참여하는 주체들이 따라야 하는 형식, 절차, 구약

위 모든 기능을 단 하나의 프로토콜을 구현할 수 없음 = 모든 기능을 하나의 클래스에서 구현하는 것 → 모듈화

- OSI model (7 layer)

  : 범용적인 네트워크 구조

- TCP/IP stack (4 layer)

  : 인터넷에 특화된 네트워크 구조

### OSI 7 layer

- **application layer L7**
    - 애플리케이션 목적에 맞는 통신 방법 제공
    - HTTP, DNS, FTP, SMTP
- **presentation layer L6**
    - 애플리케이션 간의 통신에서 메시지 포맷 관리
    - 인코딩 ↔ 디코딩
    - 암호화 ↔ 복호화
    - 압축 ↔ 압축 풀기
- **session layer L5**
    - 애플리케이션 간의 통신에서 세션을 관리
    - RPC (remote procedure call)
- **transport layer L4**
    - 애플리케이션 간의 통신 담당
    - 목적지 애플리케이션으로 데이터 전송
    - 안정적이고 신뢰할 수 있는 데이터 전송 보장 (TCP)
    - 필수 기능만 제공 (UDP)
- **network layer L3**
    - 호스트 간의 통신 담당 (IP)
    - 목적지 호스트로 데이터 전송
    - 네트워크 간의 최적의 경로 결정
- **data link layer L2**
    - 직접 연결된 노드 간의 통신 담당
    - MAC 주소 기반 통신 (ARP)
- **physical layer L1**
    - 매개체를 통해 bits 단위로 데이터 전송

### OSI 7 계층을 나눈이유는?

- 통신이 일어나는 과정이 단계별로 파악 용이

- 7단계 중 특정한 곳에 이상이 생기면 다른 단계의 장비 및 소프트웨어를 건들이지 않고도 이상이 생긴 단계만 고칠 수 있기 때문

### **전송 과정**
encapsulation / decapsulation 과정을 거침

application layer부터 메시지를 보내기 위해 역할에 맞게 처리 후 헤더를 붙이고 밑 계층으로 보냄 data link layer에선 헤더와 함께 트레일러를 붙임 (에러 체크 용도)

A → B로 메시지를 보내면 각각 레이어 별로 처리를 해준 뒤에 포장이 돼서 라우터로 보내서 network layer까지 올리고 IP 주소를 확인 후 데이터를 다시 포장후 physical layer로 내리고 B로 전송 B에서 다시 applicaiton layer까지 올려서 포장지를 뜯음

### TCP/IP stack

**application layer →** applicaiton ~ session layer

**transport layer**

**internet laye**r → network layer

**link layer →** data link + physical