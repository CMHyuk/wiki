# DBCP
#### 서버와 통신할 때 TCP 기반으로 동작
* open connection (3 way hand shake)
* close connection (4 way hand shake)
→ 매번 connection을 열고 닫는 시간적인 비용 발생, 서비스 성능에 좋지 않음

#### DBCP(database connection pool) 등장
미리 DB 커넥션을 만들어 놓고, 요청이 들어오면 새롭게 커넥션을 맺는 것이 아닌 커넥션 풀에서 가져와서 데이터 처리 후 커넥션 반납→ connection 재사용, 열고 닫는 시간 절약

## DBCP 설정 방법

### DB 설정
#### max_connections
- client와 맺을 수 있는 최대 connection 수 
  - 만약 서버 1에서 max_connections 수가 4, DBCP의 최대 connection 수가 4인 상태에서 새로운 서버 2를 도입한다면? 
    → 서버 2에서 맺고 싶어도 이미 다 맺어진 상태이긴 때문에 맺을 수 없음

#### wait_timeout
- connection이 inactive할 때 다시 요청이 오기까지 얼마의 시간을 기다린 뒤에 close 할 것인지를 결정
- 시간 내에 요청이 도착하면 0으로 초기화

#### 사용 이유?  
* 불필요한 리소스 점유를 막기 위해
  * 비정상적인 connection 종료
  * connection 다 쓰고 반환이 안됨
  * 네트워크 단절

### HikariCp 설정
#### minimumldle
- pool에서 유지하는 최소한의 idle(기다리고 있는 커넥션) connection 수
- idle connection 수가 minimumldle보다 작고, 전체 connection 수도 maximumPoolSize보다 작다면 신속하게 추가로 connection을 만듦
- 기본 값은 maximumPoolSize와 동일 (=pool size 고정)

#### maximumPoolSize
- pool이 가질 수 있는 최대 connection 수
- idle과 active(in-use) connection 합쳐서 최대 수
- 우선순위 minimumldle < maximumPoolSize

#### maxLifetime
- pool에서 connection의 최대 수명
- maxLifetime을 넘기면 idle일 경우 pool에서 바로 제거, active인 경우 pool로 반환된 후 제거
- pool로 반환 안되면 maxLiketime 동작 안 함 = 반환을 잘 시켜주는 것이 중요
- DB의 connection time limit보다 몇 초 짧게 설정해야함
  - 만약 DB의 wait_timeout이 60초, DBCP의 maxLifetime이 60초라면?
    - 59초에 요청받고 처리하려는데, wait_timeout이 발동되어 커넥션 끊음 → 요청 처리 x

#### connectionTimeout
- pool에서 connection을 받기 위한 대기 시간
  - 만약 connectionTimeout이 30초라면?
      - 무한정 커넥션을 기다릴 수 없기 때문에 30초 뒤에 연결 끊음