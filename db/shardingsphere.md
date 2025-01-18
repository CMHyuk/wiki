### ShardingSphere

* 분산 데이터베이스 및 분산 트랜잭션을 위한 오픈 소스 솔루션이다.
* Sharding은 큰 데이터를 여러 작은 데이터 조각으로 나누어 관리하는 기술로, 데이터베이스의 확장성을 높이고 성능을 향상시키는 데 중요한 역할을 한다.
* 애플리케이션 코드 변경 없이 데이터베이스를 확장할 수 있으며, 데이터베이스를 추가하거나 제거할 때 복잡한 작업 없이 유연하게 처리할 수 있다.
* ShardingSphere 분산된 테이블을 하나의 테이블처럼 사용할 수 있다.
    * 예를 들어, order 테이블이 두 개 이상의 데이터베이스에 걸쳐 분산되어 있을 때, 사용자는 단일 테이블 order에 대한 쿼리를 실행하듯이 자동으로 모든 데이터베이스와 테이블을 조회할 수 있다.
* ShardingSphere-JDBC, ShardingSphere-Proxy 두 가지 묘듈이 존재한다.

**ShardingSphere-JDBC**

* ShardingSphere-JDBC는 애플리케이션에 라이브러리로 포함되어 직접 사용된다.
* SQL 쿼리의 샤딩, 라우팅, 병렬 처리, 분산 트랜잭션 등의 기능을 제공한다.
* 애플리케이션에 직접 라이브러리를 포함시켜 사용하는 형태. sharding-jdbc-core 라이브러리를 포함하여, 데이터 소스, 샤딩 규칙, 트랜잭션 관리 등을 설정한다.
* 설정: application.properties나 application.yml에 직접 분산 설정을 넣어 사용한다.

**ShardingSphere-Proxy**

* ShardingSphere-Proxy는 독립된 서버로 실행된다. 애플리케이션은 ShardingSphere-Proxy와 연결하고, Proxy는 백엔드의 여러 데이터베이스로 쿼리를 라우팅한다.
* Proxy는 SQL 라우팅, 샤딩, 트랜잭션 관리 등을 서버 수준에서 처리합니다. 데이터베이스와의 직접 연결 없이 클라이언트는 Proxy와만 연결하면 된다.
* MySQL, PostgreSQL 등 여러 DBMS와 호환된다.
* 애플리케이션이 직접 데이터베이스와 연결되지 않고, ShardingSphere-Proxy와만 연결합니다. Proxy는 데이터베이스 샤딩, 트랜잭션, 분산 처리 등을 수행한다.

| 항목              | **ShardingSphere-JDBC** | **ShardingSphere-Proxy**         |
|-----------------|-------------------------|----------------------------------|
| **운영 방식**       | 애플리케이션 내 라이브러리 형태로 동작   | 독립적인 서버로 동작 (프록시 서버)             |
| **설정**          | 애플리케이션 코드에 설정 파일을 통해 설정 | 별도의 독립된 서버 설정이 필요                |
| **애플리케이션 통합**   | 애플리케이션에 직접 통합           | 애플리케이션은 Proxy에만 연결               |
| **샤딩 및 라우팅 처리** | 애플리케이션 코드에서 분산 처리       | Proxy가 중앙에서 모든 라우팅 및 샤딩을 처리      |
| **지원 DBMS**     | JDBC 호환 DBMS와 연결        | 여러 DBMS (MySQL, PostgreSQL 등) 지원 |
| **장점**          | 애플리케이션 내에서 간편한 통합       | 애플리케이션 코드 간소화, 중앙 집중식 관리         |
| **단점**          | 애플리케이션 코드가 복잡해질 수 있음    | 프록시 서버 관리 필요, 성능 부하 가능성          |

**Java API 버전을 사용한 설정 적용**

```java
private Collection<RuleConfiguration> getRuleConfigurations() {
    ShardingRuleConfiguration ruleConfiguration = new ShardingRuleConfiguration();

    // Sharding tables rules
    Collection<ShardingTableRuleConfiguration> tableRulesConfList = new ArrayList<>();
    ShardingStrategyConfiguration strategyConfiguration = new StandardShargindStrategyConfiguration("샤드키", "generalModShardingAlgorithm");
    ShardingTableRuleConfiguration tableRuleConfiguration = new ShardingTableRuleConfiguration("테이블", "ds${0...1}.goods");
    goodsTableRuleConfiguration.setDatabaseShardingStrategy(strategyConfiguration);
    tableRuleConfList.add(goodsTableRuleConfiguration);

    ruleConfiguration.setTables(tableRulesConfList);


    // Algorithm rules
    Properties generalModShardingAlgorithmProperties = new Properties();
    generalModShardingAlgorithmProperties.setProperty("sharding-count", "4");

    ShardingSphereAlgorithmConfiguration shardingSphereAlgorithmConfiguration = new ShardingSphereAlgorithmConfiguration("MOD", generalModShardingAlgorithmProperties);
    Map<String, ShardingSphereAlgorithmConfiguration> shardingSphereAlgorithmConfigurationMap = new HashMap<>();
    shardingSphereAlgorithmConfigurationMap.put("generalModShardingAlgorithm", shardingSphereAlgorithmConfiguration);


    ruleConfiguration.setShardingAlgorithm(shardingSphereAlgorithmConfigurationMap);

    Collection<RuleConfiguration> ruleConfigs = new ArrayList<>();
    ruleConfigs.add(ruleConfiguration);

    return ruleConfigs;
}
```

**YML Configuration 버전을 사용한 설정 적용**

```yml
shardingsphere:
  datasources:
    ds0: ...
    ds1: ...

  rules:
    tables:
      테이블명:
        actual-data-nodes: ds$->{0..1}.테이블명
        database-strategy:
          sharding-column: 샤드키
          sharding-algorithm-name: 샤드 모드명
    sharding-algorithms:
      모드명:
        type: ...
        props:
          sharding-count: 샤드 mod 값
    sharding:
      broadcast-tables: #샤딩하지 않고, 모든 데이터베이스에 저장할 경우
        - 브로드캐스트 테이블명
```