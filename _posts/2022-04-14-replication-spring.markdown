---
title: "Database Replication (2) - Spring Boot Datasource Replication"
excerpt:

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Spring Boot
  - Replication

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# 프로퍼티 설정

Spring boot 에서 기본적으로 적용되는 Hikari CP 를 이용하여 Datasource 설정.

```yml
spring:
  datasource:
    master:
      hikari:
        jdbc-url: jdbc:mariadb://localhost:3306/cmx_cloud_v2_region_db
        driver-class-name: org.mariadb.jdbc.Driver
        read-only: false
        username: root
        password: qwerty1!
    slave:
      hikari:
        jdbc-url: jdbc:mariadb://localhost:3306/cmx_cloud_v2_region_db
        driver-class-name: org.mariadb.jdbc.Driver
        read-only: true
        username: root
        password: qwerty1!
```

이렇게 2개 이상의 Datasource 를 설정하면 스프링 부트의 AutoConfiguration 기능을 제공받지 못하기 때문에 설정 코드를 직접 작성해야 한다.

<br/>

# Datasource Config

마스터 데이터소스 설정

```java
@Configuration
public class MasterDataSourceConfig {

  @Primary
  @Bean(name = "masterDataSource")
  @ConfigurationProperties(prefix = "spring.datasource.master.hikari")
  public DataSource masterDataSource() {
    return DataSourceBuilder.create()
        .type(HikariDataSource.class)
        .build();
  }
}
```

슬레이브 데이터소스 설정

```java
@Configuration
public class SlaveDataSourceConfig {

  @Bean(name = "slaveDataSource")
  @ConfigurationProperties(prefix = "spring.datasource.slave.hikari")
  public DataSource slaveDataSource() {
    return DataSourceBuilder.create()
        .type(HikariDataSource.class)
        .build();
  }
}
```

데이터소스 타입은 Hikari CP 를 사용하였으므로 HikariDataSource 를 사용함.

<br/>

# 라우팅 설정

타입 세이프하도록 Enum 으로 Master / Slave 를 정의.

```java
public enum DataSourceType {
  Master, Slave
}
```

트랜잭션의 성격, read 인지 write 인지를 판단하여 위 `DataSourceType` 을 return 하는 객체 생성. `AbstractRoutingDataSource` 를 상속받아 `determineCurrentLookupKey()` 메서드를 override 하여 구현.

```java
public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {

  @Override
  protected Object determineCurrentLookupKey() {
    DataSourceType dataSourceType =
        TransactionSynchronizationManager.isCurrentTransactionReadOnly() ? DataSourceType.Slave : DataSourceType.Master;
    return dataSourceType;
  }
}
```

마지막으로 직접 생성한 DataSource 에 대한 정보를 생성하고, 실제로 트랜잭션이 시작되면 datasource 클래스에 정보를 전달해주는 `routingDataSource` 메서드를 작성.

```java
@Configuration
public class RoutingDataSourceConfig {

  @Bean(name = "routingDataSource")
  public DataSource routingDataSource(
      @Qualifier("masterDataSource") final DataSource masterDataSource,
      @Qualifier("slaveDataSource") final DataSource slaveDataSource
  ) {
    ReplicationRoutingDataSource routingDataSource = new ReplicationRoutingDataSource();
    Map<Object, Object> dataSourceMap = new HashMap<>();

    dataSourceMap.put(DataSourceType.Master, masterDataSource);
    dataSourceMap.put(DataSourceType.Slave, slaveDataSource);

    routingDataSource.setTargetDataSources(dataSourceMap);
    routingDataSource.setDefaultTargetDataSource(masterDataSource);

    return routingDataSource;
  }

  @Bean(name = "dataSource")
  public DataSource dataSource(@Qualifier("routingDataSource") DataSource routingDataSource) {
    return new LazyConnectionDataSourceProxy(routingDataSource);
  }
}
```

아래에 `dataSource` 메서드에서는 `LazyConnectionDataSourceProxy` 객체를 return 하도록 되어 있음. 연결을 lazy 하게 하지 않으면 스프링의 특성상 트랜잭션이 생성되면 Datasource connection 을 가져오기 때문에 항상 Master Data 를 가져오게 되어 replication 을 한 의미가 없게 됨. 따라서 `routingDataSource`를 `LazyConnectionDataSourceProxy` 로 한 번 감싸주어 실제 쿼리가 발생한 시점에 Datasource Connection 을 가져오도록 함.

<br/>

# 갈무리

이제 본격적으로 트랜잭션을 관리해야 하는 이유가 생겼음. 잘 배워서 잘 정리해보자.

출처:

[Spring boot :: Datasource Replication 구현](https://wave1994.tistory.com/177)
