### 메시지 처리 방식

**basicAck**

* 메시지가 성공적으로 처리되었음을 알림, 메시지는 큐에서 삭제

```java
public void basicAck(long deliveryTag, boolean multiple);
```

- **`deliveryTag`** : RabbitMQ가 메시지마다 제공하는 고유 식별자, 어떤 메시지에 대해 작업을 수행할지 식별
- **`false`**: 이 값은 "현재 메시지만 확인"하도록 지정 즉, 이 메시지의 `deliveryTag`에 해당하는 하나의 메시지만 확인, 다른 메시지들은 영향을 받지 않음
- **`true`**: 이 값은 "현재 메시지와 그보다 작은 `deliveryTag` 값을 가진 모든 메시지를 확인"하도록 지정 즉, 현재 메시지와 함께, 그 이전에 받은 모든 메시지를 한 번에 ack 처리

**basicNack**

* 여러 메시지에 대해 거부하거나 다시 큐에 넣는 데 사용

```java
public void basicNack(long deliveryTag, boolean multiple, boolean requeue);
```

**basicReject**

* 하나의 메시지에 대해 거부하거나 다시 큐에 넣는 데 사용

```java
public void basicReject(long deliveryTag, boolean requeue);
```

**정리**

| 메서드           | 대상 메시지 수   | 재처리 옵션 (requeue)       | 주요 용도                 |
|---------------|------------|------------------------|-----------------------|
| `basicAck`    | 1개 또는 여러 개 | 없음                     | 메시지 처리 완료 후 확인        |
| `basicNack`   | 1개 또는 여러 개 | 지원 (`true` 또는 `false`) | 여러 메시지 거부 및 재처리 여부 결정 |
| `basicReject` | 1개         | 지원 (`true` 또는 `false`) | 단일 메시지 거부 및 재처리 여부 결정 |

### Mandatory 플래그

* 메시지가 큐에 도달하지 못했을 때 처리할 수 있는 기능
    * `Basic.Return`: Mandatory 플래그가 true이고 라우팅되지 않은 메시지가 발생했을 때, Broker가 Producer에게 반환하는 이벤트

```yaml
spring:
  rabbitmq:
    publisher-returns: true
```

```java

@Bean
RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
    rabbitTemplate.setMandatory(true); // Basic.Return 활성화

    rabbitTemplate.setReturnsCallback(returned -> {
        // Basic.Return 이벤트 처리
        log.warn("Message returned: replyCode={}, replyText={}, exchange={}, routingKey={}",
                returned.getReplyCode(),
                returned.getReplyText(),
                returned.getExchange(),
                returned.getRoutingKey());
    });
    return rabbitTemplate;
}
```

### Publisher Confirms

* 메시지를 꺼내서 처리할 때만 ACK를 보내는게 아니라, 정상적으로 보냈는지 확인할 ACK도 받고 싶을 때 Publisher Confirm을 사용
* 동작 방식
    * Publisher가 메시지를 RabbitMQ 서버에 전송
    * RabbitMQ 서버는 메시지를 수신하고 해당 메시지를 큐에 저장
    * 서버는 저장이 완료되면 ACK(확인 응답)를 Publisher에게 전송
    * 만약 저장에 실패하면 NACK(부정 응답)을 전송

```java

@Bean
public ConnectionFactory rabbitConnectionFactory() {
    CachingConnectionFactory connectionFactory = new CachingConnectionFactory("localhost", 5672);
    connectionFactory.setPublisherConfirms(true);
    return connectionFactory;
}

@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory rabbitConnectionFactory) {
    RabbitTemplate template = new RabbitTemplate(rabbitConnectionFactory);
    template.setConfirmCallback((correlationData, ack, cause) -> {
        if (ack) {
            System.out.println("ACK");
        } else {
            System.out.println("NACK: " + cause);
        }
    });
    return template;
}
```

* 장점
    * 메시지 손실 방지: 메시지가 RabbitMQ 서버에 성공적으로 도달했는지 확인 가능
* 단점
    * 메시지 확인을 위해 추가 통신이 필요하기 때문에 오버헤드 발생
    * 대량 메시지 처리 시 성능 저하

### Transactional Messaging

* 메시지 송신을 하나의 트랜잭션 단위로 처리하는 방식으로, 메시지의 송신 성공 여부에 따라 커밋(commit) 또는 롤백(rollback)을 수행
* 원자성 보장
    * 메시지를 브로커로 완전히 보내거나, 아예 보내지 않음
    * 메시지 전송 중 에러가 발생하면 롤백되어 브로커에 전송되지 않음
* 성능 저하
    * 메시지를 트랜잭션 단위로 처리하기 때문에 성능이 떨어질 수 있음
    * 특히 고성능이 필요한 시스템에서는 트랜잭션 메시징보다 Publisher Confirms를 사용하는 경우가 많음

```java

@Bean
public RabbitTransactionManager rabbitTransactionManager(ConnectionFactory connectionFactory) {
    return new RabbitTransactionManager(connectionFactory);
}

@Bean
RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
    rabbitTemplate.setChannelTransacted(true); // 트랜잭션 활성화
    return rabbitTemplate;
}

// 트랜잭션 내에서 메시지 송신
@Transactional
public void sendMessage(String exchange, String routingKey, String message) {
    rabbitTemplate.convertAndSend(exchange, routingKey, message);
}
```

|           | **Transactional Messaging** | **Publisher Confirms**          | **Mandatory 플래그**           |
|-----------|-----------------------------|---------------------------------|-----------------------------|
| **목적**    | 메시지 전송의 원자성 보장              | 메시지가 Broker(Exchange)에 도달했는지 확인 | 메시지가 Queue에 라우팅되었는지 확인      |
| **에러 처리** | 실패 시 롤백 및 재처리               | ConfirmCallback으로 브로커 도달 실패 감지  | ReturnsCallback으로 라우팅 실패 감지 |
| **성능**    | 상대적으로 느림                    | 성능 최적화 가능                       | 성능 영향 없음                    |
| **사용 상황** | 메시지 손실이 절대 허용되지 않는 경우       | 메시지의 전송 성공 여부만 확인               | 메시지의 라우팅 성공 여부만 확인          |

