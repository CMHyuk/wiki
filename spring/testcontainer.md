### TestContainer
TestContianer는 통합 테스트를 수행할 때 도커 컨테이너를 사용하는 오픈 소스 라이브러리이다. 이를 통해 테스트의 신뢰성을 높일 수 있고, 통합 테스트를 쉽고 빠르게 작성하고 실행할 수 있다.

### 활용

**JDBC URL 방식 - application.yml에서 사용**
```yml
spring:
  datasource:
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
    url: jdbc:tc:mysql:8.0:///test_container_test
    username: root
    password: password
  jpa:
    hibernate:
      format_sql: true
      show_sql: true
```

**NoSql - 코드에서 사용**
* application.yml에서 설정할 수 없고 따로 코드로 설정을 해야한다.
```java
@Slf4j
@Testcontainers
public class SpringElasticSearchTestContainer {

    static final DockerImageName myImage = DockerImageName.parse("elasticsearch:7.9.3").asCompatibleSubstituteFor("docker.elastic.co/elasticsearch/elasticsearch");
    static final ElasticsearchContainer ELASTICSEARCH_CONTAINER = new ElasticsearchContainer(myImage)
            .withEnv("ES_JAVA_OPTS", "-Xms512m -Xmx512m")
            .withEnv("discovery.type", "single-node");
    static final DockerImageName AUTHORIZATION_IMAGE = DockerImageName.parse("scr.softcamp.co.kr/secaas/ojt-minhyeok-authorization:latest");
    static final DockerImageName REDIS_IMAGE = DockerImageName.parse("redis");

    public static final GenericContainer<?> AUTHORIZATION_CONTAINER;
    public static final GenericContainer<?> REDIS_CONTAINER;

    static {
        ELASTICSEARCH_CONTAINER.start();

        // Redis
        REDIS_CONTAINER = new GenericContainer<>(REDIS_IMAGE)
                .withExposedPorts(6379);
        REDIS_CONTAINER.setWaitStrategy(new WaitAllStrategy());
        REDIS_CONTAINER.start();

        // Authorization
        AUTHORIZATION_CONTAINER = new GenericContainer<>(AUTHORIZATION_IMAGE)
                .withEnv("spring.profiles.active", "dev")
                .withEnv("ELASTIC_HOST", ELASTICSEARCH_CONTAINER.getHost())
                .withEnv("ELASTIC_PORT", ELASTICSEARCH_CONTAINER.getFirstMappedPort().toString())

                .withEnv("REDIS_HOST", REDIS_CONTAINER.getHost())
                .withEnv("REDIS_PORT", REDIS_CONTAINER.getFirstMappedPort().toString())
                .withEnv("REDIS_PASSWORD", "1234")

                .withExposedPorts(9000);

        AUTHORIZATION_CONTAINER.addExposedPorts(9000);

        AUTHORIZATION_CONTAINER.setWaitStrategy(new WaitAllStrategy());
        AUTHORIZATION_CONTAINER.start();

        Slf4jLogConsumer logConsumer = new Slf4jLogConsumer(log);
        ELASTICSEARCH_CONTAINER.followOutput(logConsumer);
        AUTHORIZATION_CONTAINER.followOutput(logConsumer);
        REDIS_CONTAINER.followOutput(logConsumer);
    }

    @DynamicPropertySource
    public static void overrideProperties(DynamicPropertyRegistry registry) {
        registry.add("elasticsearch.port", ELASTICSEARCH_CONTAINER::getFirstMappedPort);
        registry.add("elasticsearch.host", ELASTICSEARCH_CONTAINER::getHost);

        registry.add("spring.redis.host", REDIS_CONTAINER::getHost);
        registry.add("spring.redis.port", REDIS_CONTAINER::getFirstMappedPort);

        log.info("authorization host : {}", AUTHORIZATION_CONTAINER.getHost());
        log.info("authorization port : {}", AUTHORIZATION_CONTAINER.getFirstMappedPort().toString());
    }
}
```
