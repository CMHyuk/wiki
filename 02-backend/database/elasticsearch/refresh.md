## Refresh
* 데이터가 인덱스에 추가된 후 검색 가능하게 만드는 작업 


기본적으로, Elasticsearch는 데이터가 인덱스에 추가될 때마다 자동으로 refresh를 수행하지 않고 일정 주기로 수행한다. 이 주기는 기본적으로 1초이지만 그러나 필요에 따라 수동으로 refresh를 트리거할 수 있다.

#### 왜 refresh가 필요한가?
Elasticsearch는 성능을 최적화하기 위해 데이터를 인덱스에 추가할 때 디스크에 즉시 기록하지 않고, 일정 시간 동안 메모리에 보관한다. 이 데이터는 일정 시간이 지나면 디스크에 기록되고, 이때 refresh가 일어나야 해당 데이터가 검색 가능해진다.

#### Refresh의 장단점
* 장점  
  * 검색 가능성 향상 - 문서가 빠르게 검색에 반영되어 실시간 성능이 향상된다.  
* 단점  
  * 성능 저하 - 너무 잦은 refresh는 색인 성능을 저하시킬 수 있습니다. 새로운 세그먼트를 지속적으로 생성하고 병합해야 하므로, 디스크 I/O가 증가한다.  
  * 리소스 소모: refresh 작업은 CPU와 메모리를 소모하므로, 빈번한 refresh는 시스템 자원을 더 많이 사용하게 된다.

## Refresh Policy
```
(1) 디폴트 옵션 (false)
(2) 즉시 새로고침 (true)
(3) 새로고침 될 때까지 기다림 (wait_for)
```

실시간을 어느정도 보장받기 원한다면, (2) 번 혹은 (3) 번을 선택한다.

### 스프링에서의 Refresh Policy

![img.png](../../../../assets/images/refresh.png)
![img.png](../../../../assets/images/policy.png)
![img.png](../../../../assets/images/setting.png)