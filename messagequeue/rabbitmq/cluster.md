### RabbitMQ 클러스터
RabbitMQ 클러스터는 기본적으로 Exchange는 모든 노드에서 공유됩니다. 그러나 Queue의 경우, 기본적으로 모든 노드에 복제되지 않으며 (Queue Mirroring이 활성화되지 않은 경우), 하나의 노드에만 존재할 수 있다. 즉, 해당 노드에 저장된 메시지는 그 노드에서만 존재하며, 만약 노드가 실패하면 해당 메시지는 유실될 수 있다.

**Queue Mirroring**
- Queue Mirroring은 여러 노드에 동일한 Queue를 복제하여 HA (High Availability)를 제공한다.
- 하나의 Master Queue와 여러 Slave Queue로 구성되어, Master에서 처리되는 메시지가 Slave에도 복제되어 저장된다. 이로 인해 노드 실패 시 메시지 유실을 방지할 수 있다.
- 미러링은 정책을 이용해 설정할 수 있다. 
  - 클러스터 모든 노드의 Queue를 미러링하기 
    - `$ rabbitmqctl set_policy ha-all "^ha\\." '{"ha-mode":"all"}'`
  - 지정한 개수의 리플리카만 생성 
    - `$ rabbitmqctl set_policy ha-two "^two\\." '{"ha-mode":"exactly", "ha-params": 2, "ha-sync-mode": "automatic"}'`
  - 지정한 이름의 노드만 미러링 
    - `$ rabbitmqctl set_policy ha-nodes "^nodes\\." '{"ha-mode": "nodes", "ha-params": ["rabbit@nodeA", "rabbit@nodeB"]}'`

**Master-슬레이브 구조**
- Master Queue: 실제로 메시지를 소비(consume)하는 Queue로, 모든 Producer는 Master에 메시지를 보낸다.
- Slave Queue: Master의 메시지를 복제하여 저장하는 Queue다. Consumer는 Master Queue에서만 메시지를 소비하고, Slave Queue는 주로 장애 복구 및 가용성을 위한 용도로 사용된다.
- **Mirroring 과정** 
  - 메시지 발행
    - Producer는 Master Queue로 메시지를 전송한다. 이 메시지는 Master Queue에 저장되고, 동시에 Slave Queue에 복제된다.
  - 메시지 소비
    - Consumer는 Master Queue에서만 메시지를 소비하며, 메시지가 consume되고 나면 acknowledgment(ack)를 보낸다. 이때 Slave Queue에 저장된 메시지는 drop된다.
  - 장애 처리 및 가용성
    - 만약 Master Queue가 있는 노드가 실패하면, Slave Queue 중에서 가장 오래된 Slave가 Master로 승격된다. 이 승격된 노드는 새로 들어오는 메시지를 처리하고, 승격되는 동안 저장되지 않았던 메시지는 Slave Queue에 저장되며, 승격이 완료되면 Master Queue에 추가된다. 
    - 이 과정에서 메시지 유실을 최소화하며, 장애 복구가 가능하다.
- **미러링의 목적**
  - HA(High Availability)와 가용성을 높이기 위해 사용되지만, 메시지 처리량을 높이기 위한 것은 아니다. 
  - 미러링된 Slave는 메시지 소비에는 참여하지 않으며, 메시지 처리량을 향상시키지는 않는다. 오직 장애 발생 시 메시지 유실을 방지하고 서비스 중단 없이 클러스터를 운영하는 데 목적이 있다.
- **Synchronized 상태** 
  - 미러링된 Slave Queue는 Master Queue와 항상 동기화되어야 한다. 클러스터가 동작하는 동안 새로운 노드가 추가되면 Slave Queue는 새로 들어오는 메시지만 저장하고, 기존의 메시지는 비동기적으로 동기화된다. 
  - 이 상태를 Unsynchronized 상태라고 하며, 클러스터가 완전히 동기화되면 Synchronized 상태가 된다.
- **Message 유실과 재소비** 
  - Slave Queue에 메시지가 저장되어 있지만 Master Queue에서 메시지가 소비되면, 해당 메시지는 acknowledgment를 받으면 삭제된다. 
  - 만약 연결이 끊어져서 acknowledgment가 전달되지 않으면, 해당 메시지는 requeue되어 다시 큐에 들어가며, Consumer가 다시 처리할 수 있게 된다. 이로 인해 이미 처리된 메시지를 재소비하는 일이 발생할 수 있다.
- **마스터-슬레이브 관계에서의 유실 및 동기화** 
  - Master Queue가 실패하면, Slave Queue 중 하나가 승격되어 새로운 Master가 된다. 이때 메시지 유실은 없지만, 승격되기 전에 들어온 메시지는 Slave Queue에 저장되고 승격 후 Master에 전파된다. 
  - Unsynchronized 상태일 때, Slave Queue는 Master Queue와 일시적으로 동기화되지 않은 상태로 남을 수 있다. 이 상태는 시간이 지나면서 자동으로 동기화된다.
    
**정리**
- Queue Mirroring은 HA (High Availability)를 제공하여, 노드 실패 시 메시지 유실을 방지
- 메시지 처리량을 높이는 것이 아니라, 서비스의 가용성을 높이는 데 사용
- Master Queue가 실패하면 Slave Queue 중 하나가 승격되어 메시지 유실을 최소화

[클러스터 구성 설정법](https://jonnung.dev/rabbitmq/2019/08/08/rabbitmq-cluster/#gsc.tab=0)