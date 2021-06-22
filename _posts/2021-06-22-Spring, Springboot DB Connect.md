---
layout : post
title : Spring DB 연결
featured-img : springboot.jpg
categories : ['spring']
author : soohan
---

### Spring 프레임 워크 DB연결

pom.xml에 DB연결을 위한 spring-jdbc, MySql연동을 위한 MySql Connector, Mybatis 연동을 위한 Mybatis, mybatis-spring을 dependency에 추가한다.

```
pom.xml
 <!-- MYSQL -->
    <dependency>
    	<groupId>mysql</groupId>
    	<artifactId>mysql-connector-java</artifactId>
    	<version>8.0.22</version>
    </dependency>
    
    <!-- MyBatis -->
    <dependency>
    	<groupId>org.mybatis</groupId>
    	<artifactId>mybatis</artifactId>
    	<version>3.5.6</version>
    </dependency>
    
    <dependency>
    	<groupId>org.mybatis</groupId>
    	<artifactId>mybatis-spring</artifactId>
    	<version>2.0.5</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring-version}</version>
    </dependency>


이후 root-context.xml에 DataSource와 SqlSessionFactoryBean 을 정의 해야한다.


```
root-context.xml
	
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="ideatec.edu.spring.frwk.tomcat.*"/>
		<property name="annotationClass" value="org.apache.ibatis.annotations.Mapper"></property>
	</bean>

	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
		<property name="url" value="jdbc:mysql://localhost:3306/spring_edu?serverTimezone=Asia/Seoul"/>
		<property name="username" value=아이디/>
		<property name="password" value=비밀번호/>	
	</bean>

    	<!-- 	mybatis SqlSessionFactoryBean  -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<property name="configLocation" value="classpath:mybatis/mybatis-config.xml"/>
		<property name="mapperLocations" value="classpath:mapper/*.xml"/>
	</bean>

DataSource는 JDBC의 커넥션을 처리하는 기능을 가지고 있어 DB연동 작업에 반드시 필요하다.
SqlSessionFactory 는 MyBatis를 이용하여 DB에 접근할때 필요한 SqlSessiondmf 생성하기위해 사용한다.
* sqlSessionFactory의  mapperLocations는 매퍼의 위치를 설정한다. confiLocation은  기타 추가 설정 파일에 대한 경로이다.

MapperScannerConfigurer는 컴포넌트 스캔을 통하여 Mapper를 찾는다.
basePackage는 Mapper를 찾는 베이스 페키지