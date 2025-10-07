### Dead Letter
* RabbitMQ에서 메시지가 처리되지 못한 메시지
  * Dead Letter 메시지가 발생하는 경우
  * 메시지가 큐에서 만료(TTL)된 경우 
  * 메시지가 큐의 최대 길이를 초과한 경우 
  * 메시지가 basicNack 또는 basicReject로 거부되었고, requeue=false로 설정된 경우

### DLX
* RabbitMQ에서 메시지가 처리되지 못하고 "죽은 메시지(Dead Letter)"가 되었을 때 메시지를 다른 큐(Dead Letter Queue, DLQ)로 전달하기 위한 교환기

### DLQ
* DLQ (Dead Letter Queue)는 DLX를 통해 전달된 "죽은 메시지"를 저장하는 큐

```java
@Configuration
public class RabbitMqConfig {

   @Bean
   public DirectExchange deadLetterExchange() {
       return new DirectExchange("dlx.exchange");
   }

   // Dead Letter Queue (DLQ)
   @Bean
   public Queue deadLetterQueue() {
       return new Queue("dlq", true); // durable = true
   }

   // DLQ Binding to DLX
   @Bean
   public Binding deadLetterBinding() {
       return BindingBuilder.bind(deadLetterQueue())
            .to(deadLetterExchange())
            .with("dlq"); // Routing key
   }
   
   @Bean
   public Queue mainQueue() {
       return new Queue("main.queue", true, false, false,
         Map.of(
         "x-dead-letter-exchange", "dlx.exchange", // 연결될 DLX 설정
         "x-dead-letter-routing-key", "dlq"        // DLQ 라우팅 키 설정
         ));
   }
   
   @Bean
   public DirectExchange mainExchange() {
       return new DirectExchange("main.exchange");
   }

   @Bean
   public Binding mainQueueBinding() {
       return BindingBuilder.bind(mainQueue())
            .to(mainExchange())
            .with("main.routing.key");
   }
}
```

```java
@Service
@RequiredArgsConstructor
public class MessageProducer {

   private final RabbitTemplate rabbitTemplate;

   public void sendMessage(String message) {
        rabbitTemplate.convertAndSend("main.exchange", "main.routing.key", message);
        System.out.println("Sent message: " + message);
   }
}
```

```java
@Service
public class MainQueueConsumer {

   @RabbitListener(queues = "main.queue")
   public void consume(String message) {
       System.out.println("Received message from Main Queue: " + message);
       throw new RuntimeException("Simulated processing error");
   }
   
}
```

```java
@Service
public class DeadLetterQueueConsumer {

   @RabbitListener(queues = "dlq")
   public void consume(String message) {
       System.out.println("Received message from DLQ: " + message);
   }
}
```
   
**실행 과정**
1. Producer는 메시지를 main.exchange의 main.queue로 전송 
2. Main Queue Consumer는 메시지를 처리하다가 실패(RuntimeException)하고 메시지를 basicNack 처리 
3. 메시지는 x-dead-letter-exchange와 x-dead-letter-routing-key 설정에 따라 DLX로 전달되어 DLQ(dlq)에 저장 
4. DLQ Consumer는 DLQ에서 메시지를 수신하여 처리