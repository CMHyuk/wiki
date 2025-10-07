MVCC (multiversion concuerrency control)
- 동시 처리량을 높이기 위해 등장

|  | read-lock | write-lock |
| --- |-----------|------------|
| read-lock | o         | o          |
| write-lock | o         | x          |
- 데이터를 읽을 때 특정 시점 기준으로 가장 최근에 commit된 데이터를 읽음
    - 특정 시점은 격리 수준에 따라 달라짐
        - repeatable read이면 tx 시작 시간 기준으로 그전에 commit된 데이터를 읽음
        - read uncommitted는 보통 MVCC가 적용 x
- 데이터 변화 이력을 관리
    - 추가적인 저장 공간을 가짐 (단점이라면 단점)
- read와 write는 서로를 block하지 않음
- postgresql은 같은 데이터에 먼저 update한 tx가 commit 되면 나중 tx는 rollback 된다
- mysql은 직접 쿼리에 `for update - x lock` `for share - s lock`   를 작성해주어야 함 - Locking read
    - Locking read는 격리 수준에 상관없이 가장 최근의 commit된 데이터를 읽음