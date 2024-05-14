**Dirty read**

- commit 되지 않은 변화를 읽음

**Non-repeatable read = Fuzzy read**

- 같은 데이터의 값이 달라짐

**Phantom read**

- 없던 데이터가 생김

이런 이상 현상들이 모두 발생하지 않게 만들 수 있지만 제약사항이 많아져서 동시 처리 가능한 트랜잭션 수가 줄어들어 결국 DB의 처리량이 줄어듦

→ Isolation level을 통해 전체 처리량과 일관성 사이에서 어느 정도 거래할 수 있음

| Isolation level | Dirty read | Non-repeatable read | Phantom read |
| --- | --- | --- | --- |
| Read uncommitted | o | o | o |
| Read committed | x | o | o |
| Repeatable read | x | x | o |
| Serializable | x | x | x |

**Dirty write**

- commit 안된 데이터를 write
    - rollback 시 정상적은 recovery는 매우 중요하기 때문에 모든 isolation level에서 dirty write를 허용하면 안됨

**Lost update**

- 업데이트를 덮어씀

**SNAPSHOT ISOLATION**
- 동시성을 어떻게 컨트롤할지에 대한 내용
- tx 시작 전에 commit된 데이터만 보임
- First-committer win (같은 데이터에 대해 write complete가 발생하면 첫 번째만 반영)