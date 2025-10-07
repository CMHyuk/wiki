### Slave Node 구동을 위한 conf 작성하기

```
// redis_6381.conf라는 이름의 Redis 설정 파일을 생성
touch redis_6381.conf 

// Redis 인스턴스(포트 6381에서 실행됨)는 마스터 Redis(포트 6380)의 데이터를 비동기로 복제
replicaof 127.0.0.1 6380

// Replica 서버가 마스터 서버와 연결 상태를 확인하기 위해 PING 메시지를 전송하는 주기 설정
repl-ping-replica-period 10

// Replica 서버가 마스터 서버로부터 응답(PING, 데이터 등)을 받지 못했을 때, 지정된 시간(60초) 안에 응답이 없으면 연결이 끊긴 것으로 간주
repl-timeout 60

// 지정된 conf 파일의 내용을 읽어 설정을 적용하고 서버를 시작
redis-server {conf 파일명}

// role(master, slave) 확인 가능
info replication
```