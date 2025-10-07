#### Partitioning
database table을 더 작은 table들로 나누는 것

#### 종류
#### vertical partitioning = 정규화
- column 기준으로 table을 나누는 방식
- 자주 사용되는 데이터, 사용되지 않는 데이터만 모아서 구성할 수도 있음

#### horizontal partitioning
- row를 기준으로 table을 나누는 방식
- hash-based horizontal partitioning
    - 가장 많이 사용될 패턴에 따라 partition key를 정하는 것이 중요
    - 데이터가 균등하게 분배될 수 있도록 hash function을 잘 정의하는 것도 중요
    - 한 번 나눠서 사용되면 이후에 추가하기 까다로움 → 설계를 잘해야함
    - 같은 스키마를 가지는 테이블에 나눠서 저장
- 만약 사용자, 채널, 구독 테이블이 있을 경우
    - 사용자 M 채널 N이면 최대 크기 M * N
    - 테이블의 크기가 커질수록 인덱스의 크기도 커짐
    - 테이블에 읽기/쓰기가 있을 때마다 인덱스에서 처리되는 시간도 조금씩 늘어남

#### Sharding
- horizontal partitioning처럼 동작
- 각 partition이 독립된 DB 서버에 저장 (horizontal partitioning은 같인 DB)
    - 부하를 분산시키는 목적
    - partition key = shard key
    - partition = shard

####  Replication
- master / primary / leader와 slave / secondary / replica로 나뉨
- 한 DB 서버에 문제가 생겨도 다른 서버가 있어서 대처 가능
  - HA를 보장 (high availability 고가용성)
  - read / write 쿼리 분산 → 서버 부하를 낮춤