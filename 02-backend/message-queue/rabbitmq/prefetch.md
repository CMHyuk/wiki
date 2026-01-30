### Prefetch
* Queue의 메세지를 Consumer의 메모리에 쌓아놓을 수 있는 최대 메세지의 양
  * 예를 들어 Prefetch가 250일 경우, RabbitMQ는 250개의 메세지까지 한번에 Listener의 메모리에 Push하고 Consumer는 메모리에서 하나씩 메세지를 꺼내서 처리

### 문제 상황
![img.png](../../assets/images/prefetch1.png)

* 위의 상황에서 전체 메세지 처리 속도를 높이기위해 Consumer를 하나더 등록해 사용하기로 가정
  * Consumer3는 쌓여있는 메세지를 나눠가져 처리할 수 있을까? -> 답은 'No'

### Prefetch 값을 설정법
![img.png](../../assets/images/prefetch2.png)

* 메시지를 처리하는 process time이 fast일 경우에는 middle ~ high의 prefetch를 사용
* process time이 slow일 경우에는 반드시 prefetch를 1로 설정해야 전체 app 성능 저하가 없다.
  * process time이 느릴 경우에는 많은 메시지를 메모리에 먼저 올려놓고 처리할 이유가 전혀 없기 때문
* process time의 fast/slow의 기준은 모니터링 툴을 통해 Consumer의 Average Ack Time을 분석해 결정
* 위와 같은 Prefetch 전략은 Single / Multiple 중 선택해 사용할 수 있다.

### Single Prefetch

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: [PREFETCH_COUNT]
```

### Multiple Prefetch
```java
@Configuration
public class RabbitmqSchemaConfig {

    @Bean
    public RabbitListenerContainerFactory<SimpleMessageListenerContainer> prefetchOneContainerFactory(
            SimpleRabbitListenerContainerFactoryConfigurer configurer,
            ConnectionFactory connectionFactory
    )
     {
        var factory = new SimpleRabbitListenerContainerFactory();
        configurer.configure(factory, connectionFactory);
        factory.setPrefetchCount(1);
        return factory;
    }
}
```

```java
@Service
public class DummyPrefetchConsumer {

    @RabbitListener(queues = "q.dummy", concurrency = "2", containerFactory = "prefetchOneContainerFactory")
    public void listenDummy(DummyMessage message){
    }
}
```