---
layout: post
title:  "SpringBoot JPA Multiple Datasource 환경에서 Transaction 처리"
author: hyun
categories: [springboot, jpa]
---
<!-- image: {경로} -->
<!-- rating: {0~5} -->

이번에 업무를 진행하다 `Multiple Datasource` 를 사용하게 되었는데,  
둘 다 JPA를 활용하여 ORM 으로 사용해보려고 하던 중 `Dirty Checking` 이  
의도대로 동작하지 않는 문제가 있었다.


``` java
@Transactional
public void updateTriggerHookingLogStatus(long triggerHookingIdx, String status) {
    repository.findById(triggerHookingIdx)
              .get()
              .update(status);
}

...
@Entity
public class TriggerHookingLog {
  ...

  public void update(String status) {
    this.status = status;
  }
}
```
위 코드는 하나의 JPA를 auto-configuration 으로 사용할 때는 문제 없었는데,  
이번에 `Multiple Datasource`로 사용하면서 의도대로 동작하지 않는 이슈가 있었다.  


먼저 아래 Configuration을 살펴보자.  
``` java
/// mysql jpa configuration
@Configuration
@EnableJpaRepositories(
    entityManagerFactoryRef = "mysqlEntityManager",
    transactionManagerRef = "mysqlTransactionManager",
    basePackages = {"com.uneedcomms.triggerprocessingservice.domain.mysql"}
)
@RequiredArgsConstructor
public class MysqlConfiguration {
    private final MysqlProperties mysqlProperties;

    @Bean
    @Primary
    public DataSource mysqlDataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setUrl(mysqlProperties.getUrl());
        dataSource.setUsername(mysqlProperties.getUsername());
        dataSource.setPassword(mysqlProperties.getPassword());
        dataSource.setDriverClassName(mysqlProperties.getDriverClassName());
        return dataSource;
    }

    @Bean(name = "mysqlEntityManager")
    @Primary
    public LocalContainerEntityManagerFactoryBean mysqlEntityManager(EntityManagerFactoryBuilder builder) {
        Map<String, String> properties = new HashMap<>();
        properties.put(JpaConstant.NAMING_STRATEGY, JpaConstant.DEFAULT_NAMING);
        properties.put(JpaConstant.DIALECT, JpaConstant.MYSQL_DIALECT);
        return builder
            .dataSource(mysqlDataSource())
            .packages("com.uneedcomms.triggerprocessingservice.domain.mysql")
            .properties(properties)
            .persistenceUnit("mysql")
            .build();
    }

    @Bean(name = "mysqlTransactionManager")
    @Primary
    PlatformTransactionManager mysqlTransactionManager(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(mysqlEntityManager(builder).getObject());
    }


/// postgresql jpa configuration
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    entityManagerFactoryRef = "postgresqlEntityManager",
    transactionManagerRef = "postgresqlTransactionManager",
    basePackages = {"com.uneedcomms.triggerprocessingservice.domain.postgresql"}
)
@RequiredArgsConstructor
public class PostgresqlConfiguration {
    private final PostgresqlProperties postgresqlProperties;

    @Bean
    public DataSource postgresqlDataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setUrl(postgresqlProperties.getUrl());
        dataSource.setUsername(postgresqlProperties.getUsername());
        dataSource.setPassword(postgresqlProperties.getPassword());
        dataSource.setDriverClassName(postgresqlProperties.getDriverClassName());
        return dataSource;
    }

    @Bean(name = "postgresqlEntityManager")
    public LocalContainerEntityManagerFactoryBean postgresqlEntityManager(EntityManagerFactoryBuilder builder) {
        Map<String, Object> properties = new HashMap<>();
        properties.put(JpaConstant.NAMING_STRATEGY, JpaConstant.DEFAULT_NAMING);
        properties.put(JpaConstant.DIALECT, JpaConstant.POSTGRESQL_DIALECT);

        return builder
            .dataSource(postgresqlDataSource())
            .packages("com.uneedcomms.triggerprocessingservice.domain.postgresql")
            .properties(properties)
            .persistenceUnit("postgres")
            .build();
    }

    @Bean(name = "postgresqlTransactionManager")
    PlatformTransactionManager postgresqlTransactionManager(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(postgresqlEntityManager(builder).getObject());
    }
}
```
JPA를 `Multiple Datasource`로 사용하기 위해서는  
수동으로 configuration을 해줘야 하는 불편함이 있다.  

위와 같이 설정하면, 여러개의 Datasource를 JPA로 사용할 수 있게 된다.  
즉, 각각의 `entityManager`, `transactionManager` 를 만들어서 사용하게 된다.  

이렇게 여러개의 Datasource를 참조하는 상황에서는  
반드시 명시적으로 `entitiyManager`, `transactionManager` 를 사용해야 한다.  
``` java
// ex: entityManager 명시적 사용
@PersistenceContext(unitName = "postgres") // unitName 으로 구분

// ex: transactionManager 명시적 사용
@Transactional("postgresqlTransactionManager") // beanName 으로 구분
```

Multiple Datasource 환경에서 `@Transactional` 을 특별한 명시없이  
일반적으로 사용하게 되면 `@Primary`로 설정한 TransactionManager를 참조하게 된다.  

> postgresql의 트랜잭션을 처리해야 하는데,  
> mysql의 트랜잭션을 가져오게 되어서 정상적인 동작이 되지 않는 것이다!

어떻게 보면 당연한 결과인 것인데, 평소 `auto-configuration`의 편리함에 의존하다 보니   
놓친 부분이 있는 것 같다. (~~그래도 의존하고 싶다.. 설정 귀찮아.. 더 잘 만들어줘~~)


