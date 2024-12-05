### 메시지 처리 방식

**basicAck**
* 메시지가 성공적으로 처리되었음을 알림, 메시지는 큐에서 삭제

```java
public void basicAck(long deliveryTag, boolean multiple);
```

- **`false`**: 이 값은 "현재 메시지만 확인"하도록 지정 즉, 이 메시지의 `deliveryTag`에 해당하는 하나의 메시지만 확인, 다른 메시지들은 영향을 받지 않음
- **`true`**: 이 값은 "현재 메시지와 그보다 작은 `deliveryTag` 값을 가진 모든 메시지를 확인"하도록 지정 즉, 현재 메시지와 함께, 그 이전에 받은 모든 메시지를 한 번에 ack 처리

**basicNack**
* 여러 메시지에 대해 거부하거나 다시 큐에 넣는 데 사용

```java
public void basicNack(long deliveryTag, boolean multiple, boolean requeue);
```

- **`deliveryTag`** : 메시지의 `deliveryTag`로, 어떤 메시지에 대해 작업을 수행할지 식별
- **`multiple`** : `true`로 설정하면 현재 `deliveryTag` 이후의 모든 메시지를 한 번에 처리, `false`이면 하나의 메시지에 대해서만 처리
- **`requeue`** : `true`로 설정하면 메시지를 큐에 다시 넣어 재시도를 하게 되며, `false`로 설정하면 큐에서 메시지를 제거하고 재시도하지 않음


**basicReject**
* 하나의 메시지에 대해 거부하거나 다시 큐에 넣는 데 사용

```java
public void basicReject(long deliveryTag, boolean requeue);
```

- **`deliveryTag`** : 메시지의 `deliveryTag`로, 어떤 메시지에 대해 작업을 수행할지 식별
- **`requeue`** : `true`로 설정하면 메시지를 큐에 다시 넣어 재처리를 시도하고, `false`로 설정하면 큐에서 메시지를 제거하고 재처리하지 않음