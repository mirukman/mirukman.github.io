---
title: 스프링 웹 MVC를 위한 기본적인 의존성 및 설정 초기 세팅
author: mirukman
date: 2020-12-27 17:00:00 +0800
categories: [BACK-END, SPRING]
tags: [spring,스프링,설정,config,의존성]
---

나중에 스프링 프로젝트 초기 생성시 혹은 필요한 설정이 있을 시 가져다 쓰기위해 작성.

추후 계속해서 추가될 예정이다.

<br>

## 1. 의존성 ##
---

~~~ xml
<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
<dependencies>

	<!--스프링 관련 의존성-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.11.RELEASE</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.2.9.RELEASE</version>
      <scope>test</scope>
    </dependency>

    <!--DB 관련 의존성-->
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
    
	<!--로깅-->
	<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl -->
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-slf4j-impl</artifactId>
      <version>2.12.1</version>
    </dependency>

	<!--롬복-->
    <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.12</version>
      <scope>provided</scope>
    </dependency>
	
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

	<!--jsp jstl-->
    <!-- https://mvnrepository.com/artifact/jstl/jstl -->
    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>
~~~

<br>

## 2. 설정 ##
---

Java 설정 사용

**1) WebConfig.java**

~~~ java
public class WebConfig  extends AbstractAnnotationConfigDispatcherServletInitializer {

	/*설정파일 및 DispatcherServlet 매핑*/
	@Override
	protected Class<?>[] getRootConfigClasses() {
        return new Class[] { RootConfig.class };
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
        return new Class[] { ServletConfig.class };
	}

	@Override
	protected String[] getServletMappings() {
        return new String[] { "/" };
	}
	
	//핸들러 매핑 실패 시 404에러 핸들러 동작 설정
	@Override
	protected void customizeRegistration(ServletRegistration.Dynamic registration) {
		registration.setInitParameter("throwExceptionIfNoHandlerFound", "true");
	}
    
}
~~~

**2) RootConfig.java**

~~~ java
@Configuration
@ComponentScan(basePackages = { "빈 scan할 패키지" })
@MapperScan(basePackages = {"MyBatis mapper 패키지"})
public class RootConfig {
    
	/*MyBatis 설정*/
    @Bean
    public DataSource dataSource() {
        HikariConfig hikariConfig = new HikariConfig();
        hikariConfig.setDriverClassName("org.mariadb.jdbc.Driver");
        hikariConfig.setJdbcUrl("jdbc:mariadb://localhost:3306/[db이름]");
        hikariConfig.setUsername("사용자");
        hikariConfig.setPassword("패스워드");
    
        HikariDataSource dataSource = new HikariDataSource(hikariConfig);
        
		return dataSource;
    }
    
    @Bean
    public SqlSessionFactory SqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource());
        return (SqlSessionFactory) sqlSessionFactory.getObject();
    }
   
}
~~~

<br>

**3) ServletConfig.java**

~~~ java
@EnableWebMvc
@ComponentScan(basePackages = {"컨트롤러 패키지", "예외 클래스 패키지"})
public class ServletConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setViewClass(JstlView.class);
        viewResolver.setPrefix("/WEB-INF/views/");
        viewResolver.setSuffix(".jsp");

        registry.viewResolver(viewResolver);
    }
    
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
    }
}
~~~
