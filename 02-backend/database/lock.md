#### write-lock (exclusive lock)

- read / write 할 때 사용
- 다른 tx가 같은 데이터를 read / write 하는 것을 허용 x

#### read-lock (shared lock)

- read 할 때 사용
- 다른 tx가 같은 데이터를 read 하는 것을 허용

#### lock 호환성

|  | read-lock | write-lock |
| --- | --- | --- |
| read-lock | o | x |
| write-lock | x | x |

#### 2PL protocol (two-phase-locking)

- tx에서 모든 locking operation은 최초의 unlock operation보다 먼저 수행되도록 하는 것
- Expanding phase
  - lock을 취득하기만 하고 반환하지는 않는 phase
- Shrinking phase
  - lock을 반환만 하고 취득하지는 않는 phase

## Lock 전략
#### 낙관적 Lock
* 충돌이 발생하지 않음을 가정하고 쓰는 전략
* 애플리케이션 Lock

#### 비관적 Lock
* 매번 충돌이 발생함을 가정하고 쓰는 전략
* 데이터베이스 트랜잭션 Lock

#### JPA의 낙관적 락
* `@Version`을 이용
  * 버전을 체크 후 데이터 수정 (버전이 다르면 수정 x)

#### JPA의 비관적 락
`@Lock(LockModeType.PESSIMISTIC_WIRTE)`