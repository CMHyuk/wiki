### 개발자, 네트워크 참조 모델에서 무엇을 알아야 할까

- 직무 별로 네트워크 참조 모델 지식을 구분하던 시절이 있었다.
    - 응용, 전송, 네트워크 계층: 개발자들이 알아야 하는 네트워크 지식
    - 네트워크, 데이터 링크, 물리 계층: 네트워크 엔지니어들이 알아야 하는 네트워크 지식
- 다만, 최근에는 이 구분이 모호해지고 있다.
- 왜? 인프라를 "코드"로 다루는 시대이기 때문 (IaC: Infrastructure as Code)
    - 개발자는 코드를 다룬다.
    - 코드로 인프라를 다룰 수 있다. -> 개발자도 인프라를 다룰 수 있다.

--- 

### 이더넷

**이더넷**

- 현대 LAN, 특히 유선 LAN 환경에서 가장 대중적으로 사용되는 기술
- 다양한 통신 매체의 규격, 송수신되는 프레임의 형태, 프레임의 형태, 프레임을 주고받는 방법 등이 정의된 기술
    - 물리 계층과 데이터 링크 계층이 밀접하게 연관된 이유

유선 LAN 환경은 대부분 이더넷을 기반으로 구성된다.

- 물리 계층에서는 사용되는 케이블? 이더넷 규격을 따름
- 데이터 링크 계층에서 주고받는 프레임? 이더넷 프레임의 형식을 따름

**국제 표준으로써의 이더넷**

- 이더넷은 IEEE 802.3이라는 이름으로 국제 표준이 됨
- IEEE 802.3 == 이더넷 관련 다양한 표준의 모음
- 이더넷 표준에 따라 지원되는 네트워크 장비, 통신 매체의 종류, 전송 속도 등이 달라짐
- "전송 속도BASE-추가 특성"이 표기 공식
    - ex) 1000BASE-SX, 5GBASE-T
    - 숫자만 표기되어 있으면 Mbps 속도
    - 숫자 뒤에 G가 붙는 경우 Gbps 속도
    - base는 베이스밴드의 약자로, 변조 타입을 의미
        - 변조 타입 - 비트 신호로 변환된 데이터를 통신 매체로 전송하는 방법 (개발과는 다소 거리가 있는 내용)

**추가 특성**

- 통신 매체의 특성을 명시
- 다양한 특성이 명시될 수 있음
    - 전송 가능한 최대 거리 - 예) 10BASE-2, 10BASE-5
    - 물리 계층 인코딩 방식 - 데이터가 비트 신호로 변환되는 방식
    - 레인 수 - 비트 신호를 옮길 수 있는 전송로 수
- 가장 중요한 추가 특성? -> **통신 매체의 종류**

**통신 매체의 종류**

| 구분 | 통신 매체의 종류             | 케이블 종류                                                   |
|----|-----------------------|----------------------------------------------------------|
| C  | 동축 케이블(Coaxial Cable) | 굵은 이더넷(ThickNet), 얇은 이더넷(ThinNet) 등                      |
| T  | 트위스티드 페어 케이블          | UTP(Unshielded Twisted Pair), STP(Shielded Twisted Pair) |
| S  | 단파장 광섬유 케이블           | Single-mode Fiber (Short wavelength)                     |
| L  | 장파장 광섬유 케이블           | Single-mode Fiber (Long wavelength)                      |

- 10BASE-T 케이블: 10Mbps 속도를 지원하는 트위스티드 페어 케이블 
- 1000BASE-SX 케이블: 1000Mbps 속도를 지원하는 단파장 광섬유 케이블 
- 1000BASE-LX 케이블: 1000Mbps 속도를 지원하는 장파장 광섬유 케이블

**이더넷은 지금도 발전 중**  
- 고속 이더넷 
  - 100Mbps가량의 속도를 지원하는 표준
- 기가비트 이더넷 
  - 1Gbps가량의 속도를 내는 이더넷 표준
- 10기가비트 이더넷 
  - 10Gbps가량의 속도를 내는 이더넷 표준

---

### 이더넷 프레임
- (데이터링크 계층) 이더넷 네트워크에서 주고받는 프레임 
  - 캡슐화를 거쳐 송신됨: 상위 계층 정보 + 헤더 + 트레일러 
  - 헤더 - 프리앰블, **수신지 MAC 주소**, **송신지 MAC 주소**, 타입/길이 
  - 페이로드 = 데이터 
  - 트레일러 - FCS
- 역캡슐화를 거쳐 수신됨
  - 헤더, 트레일러 제거 후 상위 계층으로 올려보냄

![img.png](../../image/ethernet-frame.png)

**프리앰블**
- 이더넷 프레임의 시작을 알리는 8바이트(64비트) 크기의 정보 
- 첫 7바이트는 10101010 값을 가지고, 마지막 바이트는 10101011 값츨 가짐 
- 송수신지 간의 동기화를 위해 사용되는 정보

**수신지 MAC 주소와 송신지 MAC 주소**
- 물리적 주소라고도 불림 
- 일반적으로 고유하고, 일반적으로 변경되지 않는 주소 
- MAC 주소는 네트워크 인터페이스마다 부여되는 6바이트(48비트) 길이의 주소 
  - LAN 내의 송수신지 특정 
  - 일반적으로 NIC 장치가 네트워크 인터페이스 역할을 담당 
  - 한 컴퓨터에 MAC 주소도 여러 개 있을 수 있음

**타입/길이**  
- 타입 혹은 길이 명시 
- 필드에 명시된 크기가 1500(16진수 05DC) 이하일 경우: 이 필드는 프레임의 크기(길이)
- 필드에 명시된 크기가 1536(16진수 0600) 이상일 경우: 이 필드는 타입
- 타입이란? 
  - 이더타입이라고도 함 
  - 어떤 정보를 캡슐화했는지를 나타내는 정보 
  - 대표적으로 상위 계층에서 사용된 프로토콜을 명시

**데이터**  
- 페이로드: 상위 계층에서 전달받거나 전달해야 할 내용 
- 최대 크기: 1500바이트 
- 최소 크기: 46바이트 
  - 46바이트보다 작다면 크기 맞추기 용 데이터인 패딩이 채워짐 보통 0으로 채워짐

---

### NIC(Network Interface Controller)

**NIC**  
- 호스트와 통신 매체를 연결하고, MAC 주소가 부여되는 네트워크 장비 
- 네트워크를 이용할 수 있게끔 하는 하드웨어

**케이블은 NIC에 연결되는 물리 계층의 유선 통신 매체**  
- 트워스티드 페어 케이블 
- 광섬유 케이블

**호스트를 네트워크(LAN)에 연결하는 장비**   
- 호스트와 유무선 통신 매체를 연결 
- 통신 매체 신호와 컴퓨터가 이해하는 정보 상호 변환 
  - 호스트가 네트워크를 통해 송수신하는 정보는 NIC를 거치게 됨 
  - 네트워크 인터페이스 역할을 수행

**NIC는 MAC 주소를 인식**  
- 자신과는 관련 없는 수신지 MAC 주소가 명시된 프레임 폐기 
- FCS 필드를 토대로 오류를 검출해 잘못된 프레임을 폐기

**NIC 속도와 성능의 관계**  
- NIC마다 지원되는 속도가 다르다는 점에 유의 
- NIC의 지원 속도는 10Mbps부터 100Gbps에 이르기까지 NIC마다 다름 
- 네트워크 속도에 영향을 끼침
- (내장된 NIC가 있어도) 높은 대역폭에서 많은 트래픽을 감당해야 하는 환경에선 고속 NIC 추가하기도 함

**케이블**  
- 트위스티드 페어 케이블 
  - 구리 선으로 전기 신호를 주고받는 통신 매체 
  - 생김새 = 커넥터 + 케이블 본체 
    - 커넥터: 주로 활용되는 커넥터는 RJ-45 
    - 케이블 본체: 구리 선이 두 가닥씩 꼬아진 형태
  - 구리 선은 노이즈에 민감 
    - 차폐 - 구리 선 주변을 감싸 노이즈를 감소시키는 방식 
    - 브레이브 실드 혹은 포일 실드
  - 실드에 따른 트위스티드 페어 케이블의 분류 
    - STP - 브레이브 실드로 감싼 케이블 
    - FTP - 포일 실드로 노이즈를 감소시킨 케이블 
    - UTP - 아무것도 감싸지 않은 구리 선만 있는 케이블
    - XX에는 케이블 외부를 감싸는 실드의 종류(하나 혹은 두 개)
    - Y에는 꼬인 구리 선 쌍을 감싸는 실드의 종류 
    - U: 실드 없음, S: 브레이브 실드, F: 포일 실드

**카테고리에 따른 트위스티드 페어 케이블의 분류**  
- 카테고리가 높을 수록…
  - 지원 가능한 대역폭이 높아짐
  - 송수신 할 수 있는 데이터의 양이 많아짐
  - 일반적으로 더 빠른 전송이 가능함

**광섬유 케이블**  
- 빛(광신호)을 이용해 정보를 주고받는 케이블
- 전기 신호를 이용하는 케이블에 비해 속도도 빠르고, 먼 거리까지 전송이 가능
- 노이즈로부터 간섭받는 영향도 적으므로 대륙 간 네트워크 연결에도 사용
- 광섬유 케이블 본체 내부는 머리카락과 같은 형태의 광섬유로 구성 
  - 광섬유는 빛을 운반하는 매체 
  - 광섬유 중심에는 코어 - 코어는 광섬유에서 실질적으로 빛이 흐르는 부분 
  - 코어를 둘러싸는 클래딩 - 빛이 코어 안에서만 흐르도록 빛을 가두는 역할

---

### 물리 계층과 데이터 링크 계층의 장비

**네트워크 장비**  
- 물리 계층의 대표 장비 - 허브 
- 데이터 링크 계층의 대표 장비 - 스위치

**물리 계층에는 주소 개념이 없다**
- 단지 호스트와 통신 매체 간의 연결과 통신 매체상의 송수신이 이루어 질 뿐 
- 물리 계층 장비는 송수신되는 정보에 대한 어떠한 조작(송수신 내용 변경)이나 판단도 않음

**데이터링크 계층에는 주소 개념이 있다**
- MAC 주소 
- 데이터 링크 계층 이상 장비들은 송수신지 특정 가능, 송수신 정보에 대한 조작 가능

**허브**
- 여러 대의 호스트를 연결하는 장치
- 리피터 허브 혹은 이더넷 허브 
- 포트 - 커넥터를 연결할 수 있는 연결 지점
- 허브의 특징 1: 받은 정보는 모든 포트로 내보냄 
  - 정보에 대한 어떠한 조작도 판단도 하지 않음(물리 계층 장비니까)
  - 전달받은 신호를 다른 모든 포트로 그대로 다시 내보냄 
  - 데이터 링크 계층에서 패킷의 MAC 주소를 확인하고 자신과 관련 없는 주소는 폐기
- 허브의 특징 2: 반이중 통신 
  - 반이중 모드 - 마치 1차선 도로처럼 송수신을 번갈아 가면서 하는 통신 방식 
  - 전이중 모드 - 송수신을 동시에 양방향으로 할 수 있는 통신 방식
- 허브의 특징이 야기하는 문제, 충돌 
  - 동시에 허브에 신호를 송신하면 충돌이 발생 
  - 허브에 호스트가 많이 연결되어 있을수록 충돌 발생 가능성이 높음
- 콜리전 도메인 - 충돌이 발생할 수 있는 영역 
  - 허브에 연결된 모든 호스트는 같은 콜리전 도메인에 속함
- 충돌 해결 방법
  - CSMA/CD (Carrier Sense Multiple Access With Collision Detection)
    - 반이중 이더넷 네트워크에서 충돌을 방지하는 대표적인 프로토콜
      - (반이중) 이더넷을 대표하는 송수신 방법
    - **캐리어 감지**   
      - 통신 매체의 현재 사용 가능 여부 검사: 메시지를 보내기 전 현재 전송 중인 것이 있는지를 먼저 확인  
    - **다중 접근**   
      - 복수의 호스트가 부득이 동시에 네트워크에 접근할 때: 충돌 발생
    - **충돌 검출**
      - 전송 중단, 충돌 발생을 알리는 잼 신호 보냄 
      - 임의의 시간 동안 기다린 뒤에 재전송
- 허브는 오늘날에 많이 쓰이진 않지만 스위치와 대비되는 특성이 있기 때문에 설명됨

**스위치**  
- 허브의 충돌 문제 
  - CSMA/CD로 어느 정도 완화할 수 있었지만... 보다 근본적인 해결 방법이 있다 
  - 전달받은 신호를 수신지 호스트가 연결된 포트로만 내보내고, 전이중 모드로 통신하면 된다.
- **이를 위한 장비가 바로 스위치**
  - 허브와는 달리 특정 MAC 주소를 가진 호스트에만 프레임 전달 가능 
  - 전이중 모드 통신 지원으로 CSMA/CD 프로토콜이 필요하지 않음
- 스위치의 주요 기능 
  - 스위치의 MAC 주소 학습 기능 
    - 전달받은 신호를 원하는 포트로만 내보냄 
    - 포트별로 콜리전 도메인이 나누어지기에 충돌 위험이 감소
  - 스위치의 VLAN 기능 
    - 논리적으로 LAN을 분리하는 가상의 LAN, VLAN 구성 가능

**스위치의 MAC 주소 학습 기능**  
- 특정 포트와 해당 포트에 연결된 호스트의 MAC 주소와의 관계를 기억
- 원하는 호스트에만 프레임을 전달
- MAC 주소 테이블
  - 스위치 포트와 연결된 호스트의 MAC 주소 간의 연관 관계를 나타내는 정보
  - MAC 주소 학습 : 프레임 내 '송신지 MAC 주소' 필드를 바탕으로 이루어짐
- 플러딩 : 허브처럼 모든 포트로 프레임 전송 
- MAC 주소 테이블에 주소가 있으면 연결된 모든 포트로 내보내지 않도록 필터링하고 연결된 포트로 프레임을 포워딩함
- 에이징 
  - 만약 MAC 주소 테이블에 등록된 포트에서 일정 시간 동안 프레임을 받지 못하면 해당 항목은 삭제

**스위치의 VLAN 기능**  
- VLAN(Virtual LAN) - 한 대의 스위치로 가상의 LAN을 만드는 방법 
  - 불필요한 트래픽(허브, 스위치의 플러딩)으로 인한 성능 저하 방지 
  - "한 대의 물리적 스위치라 해도", "마치 여러 대의 스위치가 있는 듯이", "호스트의 물리적 위치와 관계 없이" 
- VLAN은 사실상 다른 LAN: 같은 스위치에 연결된 호스트라 하더라도 서로 다른 네트워크로 간주, 브로드캐스트 도메인 달라짐

**포트 기반 VLAN**
- 스위치의 포트가 VLAN을 결정하는 방식 
- 특정 포트에 VLAN을 할당한 뒤, 해당 포트에 호스트를 연결하여 VLAN에 참여  
 
**MAC 기반 VLAN**
- 사전에 설정된 MAC 주소에 따라 VLAN이 결정 
- 송수신하는 프레임 속 MAC 주소가 호스트가 속할 VLAN을 결정하는 방식

**브리지**  
- 브리지는 데이터 링크 계층의 스위치와 유사한 장비 
- 다만 단일 장비로서의 브리지는 스위치에 비해 사용 빈도가 줄어드는 추세 
- 일반적으로 스위치의 기능이 더 다양하고 성능도 우수하기 때문