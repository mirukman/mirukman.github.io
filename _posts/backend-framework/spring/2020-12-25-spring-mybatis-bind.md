---
title: Spring과 MyBatis 연동해서 사용하기(설정,예제코드)
author: mirukman
date: 2020-12-25 13:00:00 +0800
categories: [BACKEND, SPRING]
tags: [spring,mybatis,스프링,마이바티스,jdbc,mariadb]
---

SpringBoot의 경우 springboot-starter를 사용하면 귀찮은 설정작업등이 필요없이 자주 쓰이는 의존성 라이브러리들을 자동으로 불러와준다.

그러나 Spring에서는 의존성 추가, 설정 등을 직접 해야한다. Spring에서 DB를 연동하고 MyBatis를 사용하여 SQL 매핑하는 작업에 대해 정리한 글이다.

사용된 DB는 MariaDB이다.

또한 스프링 설정을 하는 방법에는 크게 XML을 이용한 방법과 Java 클래스를 이용하는 방법 두 가지로 나뉜다. 최근에는 Java를 사용해서 직접 설정을 하는 방식이 크게 증가하는 추세이므로 이 방식을 사용한다.

<br>

## 1. 의존성 추가 ##
----

~~~ xml

<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>5.2.11.RELEASE</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <version>5.2.9.RELEASE</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl -->
<dependency>
	<groupId>org.apache.logging.log4j</groupId>
	<artifactId>log4j-slf4j-impl</artifactId>
	<version>2.12.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.12</version>
  <scope>provided</scope>
</dependency>


<!--아래부터 DB 관련 의존성-->
<!-- https://mvnrepository.com/artifact/org.mariadb.jdbc/mariadb-java-client -->
<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
    <version>2.6.2</version>
</dependency>

<!-- https://mvnrepository.com/artifact/com.zaxxer/HikariCP -->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>3.4.2</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.3</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.3</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>


<!-- https://mvnrepository.com/artifact/org.springframework/spring-tx -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
~~~

<br>

**1) log4j-slf4j-impl**

로깅을 위한 라이브러리

<br>

**2) mariadb-java-client**

MariaDB 연결을 위한 드라이버

<br>

**3) HikariCP**

Connection Pool 설정을 위한 라이브러리. 스프링부트 2.0에도 포함될만큼 유명한 라이브러리이다.

<br>

**4) mybatis, mybatis-spring**

MyBatis와 스프링 연동을 위해서 두 라이브러리 모두 추가해주어야한다.

<br>

**5) spring-jdbc/spring-tx**

mybatis-spring이 직접 jdbc를 사용할 수 있도록 spring-jdbc를 추가하고 트랜잭션을 처리할 수 있도록 spring-tx 추가


<br>

## 2. 설정 ##
----

~~~ java
//Bean 생성을 위한 Configuration
@Configuration
@ComponentScan(basePackages = { "com.mirukman.web" })
@Slf4j
@MapperScan(basePackages = {"com.mirukman.web.mapper"})
public class RootConfig {
    
    //jdbc, HikariCP 설정
    @Bean
    public DataSource dataSource() {
        HikariConfig hikariConfig = new HikariConfig();
        hikariConfig.setDriverClassName("org.mariadb.jdbc.Driver");
        hikariConfig.setJdbcUrl("jdbc:mariadb://localhost:3306/mysql");
        hikariConfig.setUsername("root");
        hikariConfig.setPassword("root");
    
        HikariDataSource dataSource = new HikariDataSource(hikariConfig);
        
		return dataSource;
    }
    
    //MyBatis 설정
    @Bean
    public SqlSessionFactory SqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource());
        return (SqlSessionFactory) sqlSessionFactory.getObject();
    }
   
}
~~~

<br>

//log4j2.xml
~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n"/>
        </Console>
    </Appenders>

    <Loggers>
        <root level="info" additivity="false">
            <AppenderRef ref="console"/>
        </root>
    </Loggers>
</Configuration>
~~~

테스트를 위해 최소한의 설정만 적용했다.

<br>

이제 설정은 끝났다.

<br>

## 3. 테스트 ##
----

간단한 Mapper 클래스를 작성해서 MyBatis가 정상적으로 작동하는지 테스트를 해볼 차례다.

~~~ java
public interface TestMapper {
    
    @Select("SELECT host from user where User='root'")
    public String getRootHost();   
}
~~~

RootConfig 클래스의 jdbc url 설정을 확인해보면 db를 'mysql'로 설정했다. 참고로 'mysql'은 따로 만들지 않아도 원래부터 존재하는 기본 db이다. 거기서 이름이 'root'인 사용자의 host 정보를 조회하기 위한 쿼리를 매핑시켰다.

~~~ java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { RootConfig.class })
@Slf4j
public class MapperTest {
    
    @Setter(onMethod_ = {@Autowired})
    private TestMapper testMapper;

    @Test
    public void mapperTest() {
        log.info(testMapper.getClass().getName());
        log.info(testMapper.getRootHost());
    }
}
~~~

설정을 제대로 했다면 테스트를 실행시켰을 때 아래와 같이 문제없이 출력될 것이다.

~~~ console
...
[INFO ] 2020-12-25 13:52:31.748 [main] HikariDataSource - HikariPool-1 - Starting...
[INFO ] 2020-12-25 13:52:31.802 [main] HikariDataSource - HikariPool-1 - Start completed.
[INFO ] 2020-12-25 13:52:31.888 [main] mapperTest - com.sun.proxy.$Proxy40
[INFO ] 2020-12-25 13:52:31.929 [main] mapperTest - localhost
[INFO ] 2020-12-25 13:52:31.938 [SpringContextShutdownHook] HikariDataSource - HikariPool-1 - Shutdown initiated...
...
~~~

<br>

## 4. SQL 쿼리 로그를 위한 추가 적용 ##
----

MyBatis와 같은 SQL 매핑을 사용하면 실행된 SQL을 제대로 확인하기 쉽지 않다. SQL 쿼리 확인을 위한 용도로 log4jdbc-log4j2를 추가적으로 적용할 수 있다.

먼저 아래와 같이 log4jdbc 의존성을 추가해준다.

~~~ xml
<!-- https://mvnrepository.com/artifact/org.bgee.log4jdbc-log4j2/log4jdbc-log4j2-jdbc4 -->
<dependency>
    <groupId>org.bgee.log4jdbc-log4j2</groupId>
    <artifactId>log4jdbc-log4j2-jdbc4</artifactId>
    <version>1.16</version>
</dependency>
~~~

<br>

log4jdbc가 SQL 실행문을 추적하고 로깅할 수 있도록 하려면 초기에 설정했던 jdbc 드라이버 설정과 jdbc url 설정을 수정해주면 된다.

~~~ java
//hikariConfig.setDriverClassName("org.mariadb.jdbc.Driver");
//hikariConfig.setJdbcUrl("jdbc:mariadb://localhost:3306/mysql");

hikariConfig.setDriverClassName("net.sf.log4jdbc.sql.jdbcapi.DriverSpy");
hikariConfig.setJdbcUrl("jdbc:log4jdbc:mariadb://localhost:3306/mysql");
~~~

주석 처리된 부분이 변경전이고 주석 처리된 후가 변경후다.

마지막으로 /src/main/resources 하부에 log4jdbc.log4j2.properties 파일을 만들고 아래의 설정 한 줄을 추가한다.

~~~ txt
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
~~~

아까 작성했던 테스트를 다시 실행시키면 SQL로그가 자세히 출력되는걸 확인할 수 있다.
