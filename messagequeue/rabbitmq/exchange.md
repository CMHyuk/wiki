### Exchange

**Direct Exchange** 
* 메시지를 특정 라우팅 키와 정확히 일치하는 큐로 전달
* 메시지의 라우팅 키와 큐가 바인딩된 라우팅 키가 완전히 일치해야 메시지를 전달 
  * 특정 사용자 또는 서비스에 대한 메시지 전송

```java
@Configuration
public class DirectExchangeConfig {

    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("direct-exchange");
    }

    @Bean
    public Queue directQueue() {
        return new Queue("direct-queue");
    }

    @Bean
    public Binding directBinding(Queue directQueue, DirectExchange directExchange) {
        return BindingBuilder.bind(directQueue).to(directExchange).with("routing-key");
    }
}

@Autowired
private RabbitTemplate rabbitTemplate;

public void sendMessage(String message) {
  rabbitTemplate.convertAndSend("direct-exchange", "routing-key", message);
}
```

**Fanout Exchange**
* 라우팅 키를 무시하고 연결된 모든 큐로 메시지를 브로드캐스트
* 메시지가 Exchange에 도착하면 연결된 모든 큐로 복사 후 전송 
* 라우팅 키는 사용되지 않음 
* 1:N 브로드캐스팅에 적합

```java
@Configuration
public class FanoutExchangeConfig {

    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanout-exchange");
    }

    @Bean
    public Queue fanoutQueue1() {
        return new Queue("fanout-queue-1");
    }

    @Bean
    public Queue fanoutQueue2() {
        return new Queue("fanout-queue-2");
    }

    @Bean
    public Binding fanoutBinding1(Queue fanoutQueue1, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    @Bean
    public Binding fanoutBinding2(Queue fanoutQueue2, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}

@Autowired
private RabbitTemplate rabbitTemplate;

public void sendMessage(String message) {
  rabbitTemplate.convertAndSend("fanout-exchange", "", message); // 라우팅 키 무시
}

```

**Topic Exchange**
* 라우팅 키의 패턴에 따라 메시지를 큐로 전달 
* 라우팅 키는 점(.)으로 구분된 문자열 
* 큐의 바인딩 키에 와일드카드(*, #) 사용 가능
  * (Asterisk): 한 단어와 일치
  * (Hash): 0개 이상의 단어와 일치
* 복잡한 라우팅 조건 설정 가능

```java
@Configuration
public class TopicExchangeConfig {

    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange("topic-exchange");
    }

    @Bean
    public Queue topicQueue1() {
        return new Queue("topic-queue-1");
    }

    @Bean
    public Queue topicQueue2() {
        return new Queue("topic-queue-2");
    }

    @Bean
    public Binding topicBinding1(Queue topicQueue1, TopicExchange topicExchange) {
        return BindingBuilder.bind(topicQueue1).to(topicExchange).with("service.*");
    }

    @Bean
    public Binding topicBinding2(Queue topicQueue2, TopicExchange topicExchange) {
        return BindingBuilder.bind(topicQueue2).to(topicExchange).with("region.#");
    }
}

@Autowired
private RabbitTemplate rabbitTemplate;

public void sendMessage(String message) {
  rabbitTemplate.convertAndSend("topic-exchange", "service.log", message); // 특정 패턴
}
```

**Headers Exchange**
* 메시지의 헤더 속성에 따라 큐로 전달 
* 라우팅 키를 사용하지 않고, 메시지 헤더에 포함된 속성값 기반으로 라우팅 
  * 헤더 속성은 key:value 형식 
  * 바인딩 시 x-match 옵션 사용
  * all: 모든 헤더 조건을 만족해야 전달 
  * any: 하나라도 조건을 만족하면 전달

```java
@Configuration
public class HeadersExchangeConfig {

    @Bean
    public HeadersExchange headersExchange() {
        return new HeadersExchange("headers-exchange");
    }

    @Bean
    public Queue headersQueue() {
        return new Queue("headers-queue");
    }

    @Bean
    public Binding headersBinding(Queue headersQueue, HeadersExchange headersExchange) {
        return BindingBuilder.bind(headersQueue)
                             .to(headersExchange)
                             .where("header-key").matches("header-value");
    }
}

@Autowired
private RabbitTemplate rabbitTemplate;

public void sendMessage(String message) {
  MessageProperties messageProperties = new MessageProperties();
  messageProperties.setHeader("header-key", "header-value");
  Message amqpMessage = new Message(message.getBytes(), messageProperties);
  rabbitTemplate.send("headers-exchange", "", amqpMessage);
}
```

### 정리

| Exchange Type | 라우팅 기준     | 사용 목적              | 예시                 |
|---------------|-----------------|-----------------------|----------------------|
| Direct        | 라우팅 키       | 정확한 라우팅          | 특정 로그 처리        |
| Fanout        | 없음            | 메시지 브로드캐스팅     | 전체 알림             |
| Topic         | 라우팅 키 패턴  | 주제 기반 라우팅       | 특정 지역/서비스 메시징 |
| Headers       | 헤더 속성       | 복잡한 조건 기반 라우팅 | 메타데이터 기반 라우팅 |

