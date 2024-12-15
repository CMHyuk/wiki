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

| 메서드         | 대상 메시지 수            | 재처리 옵션 (requeue)  | 주요 용도                       |
|----------------|--------------------------|------------------------|--------------------------------|
| `basicAck`     | 1개 또는 여러 개         | 없음                   | 메시지 처리 완료 후 확인        |
| `basicNack`    | 1개 또는 여러 개         | 지원 (`true` 또는 `false`) | 여러 메시지 거부 및 재처리 여부 결정 |
| `basicReject`  | 1개                     | 지원 (`true` 또는 `false`) | 단일 메시지 거부 및 재처리 여부 결정 |


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