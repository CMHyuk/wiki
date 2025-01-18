### ShardingSphere
* 분산 데이터베이스 및 분산 트랜잭션을 위한 오픈 소스 솔루션이다.
* Sharding은 큰 데이터를 여러 작은 데이터 조각으로 나누어 관리하는 기술로, 데이터베이스의 확장성을 높이고 성능을 향상시키는 데 중요한 역할을 한다.
* 애플리케이션 코드 변경 없이 데이터베이스를 확장할 수 있으며, 데이터베이스를 추가하거나 제거할 때 복잡한 작업 없이 유연하게 처리할 수 있다.
* ShardingSphere 분산된 테이블을 하나의 테이블처럼 사용할 수 있다.
  * 예를 들어, order 테이블이 두 개 이상의 데이터베이스에 걸쳐 분산되어 있을 때, 사용자는 단일 테이블 order에 대한 쿼리를 실행하듯이 자동으로 모든 데이터베이스와 테이블을 조회할 수 있다.

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