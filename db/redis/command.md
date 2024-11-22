### 사용하면 안되는 커맨드
**Redis는 Single Thread로 동작**

- keys* → scan으로 대체
    - scan은 재귀적으로 key들을 호출
- Hash나 Sorted Set 등 자료구조는 키 내부에 아이템은이 많아질수록 성능이 저하
    - 하나의 키에 최대 100만 개 이상은 저장하지 말기
- key를 del로 지우면 그 key를 지우는 동안 다른 동작 x → unlink 커맨드를 사용해 key를 백그라운드로 지우자

### 변경하면 장애를 막을 수 있는 기본 설정 값
- **STOP-WIRTES-ON-BGSAVE-ERROR = NO**
    - RDB 파일이 정상적으로 저장되지 않았을 때, redis로 들어오는 모든 write를 차단
- **MAXMEMORY-POLICY = ALLKEYS-LRU**
    - redis를 캐시로 사용할 때 Expire Time 설정 권장
    - 메모리가 가득 찼을 때 MAXMEMORY-POLICY 정책에 의해 키 관리
        - noeviction : 삭제 안함, 즉 새로운 데이터 입력을 못해 장애로 이어짐
        - volatile-lru : 가장 최근에 사용하지 않았던 key부터 삭제
            - expire 설정이 된 key만 삭제해 noeviction와 같은 문제가 발생
        - allkeys-lru : expire 설정상관 없이 모든 key가 대상
- **MaxMemory는 실제 메모리의 절반으로 설정** 
  - 데이터를 파일을 저장할 때 포크를 통해 자식 프로세스 생성
    - 백그라운드에서는 데이터를 파일로 저장하지만 원래의 프로세스는 계속해서 요청을 받을 때, copy on write를 통해 메모리를 복사해 사용하기 때문에 메모리가 2배로 증가
- **CONFIG SET activedfrag yes** 
  - user_memory : 논리적으로 Redis가 사용하는 메모리
  - used_memory_rss: OS가 Redis에 할당하기 위해 사용한 물리적 메모리 양 (이 값을 보는 게 중요)
    - 삭제되는 키가 많으면 fragmentation 증가
      - fragmentation : 실제 저장된 데이터가 적으면 rss 값은 큰 상황이 발생할 수 있는데 이 차이가 클 때 fragmentation가 크다.
      - 특정 시점에 key가 피크를 치고 다시 삭제되는 경우 
      - TTL로 인해 삭제가 과도하게 많을 때

