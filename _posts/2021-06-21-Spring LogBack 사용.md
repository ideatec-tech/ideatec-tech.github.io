---
layout : post
title : Spring Logback 설정
featured-img : spring-logback.jpg
cetegories : ['spring']
author : soohan
---

## Spring 에서 LogBack 사용하기

* logback : logback는 log4j를 토대로 새롭게 만든 Logging 라이브러리, Slf4j를 통해 연관 라이브러리들이 다른 logging framework 를 쓰더라도 logback로 통합 할 수 있다.


### Spring framework 에서 logback 사용

```pom.xml
pom.xml   
 <dependency>
    	<groupId>ch.qos.logback</groupId>
    	<artifactId>logback-classic</artifactId>
    	<version>1.2.3</version>
    </dependency>
  
    <dependency>
    	<groupId>ch.qos.logback</groupId>
    	<artifactId>logback-core</artifactId>
    	<version>1.2.3</version>
    </dependency> 
```

pom.xml dependency에 logback 라이브러리를 추가해준다.


```logback.xml
logback.xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- 30초마다 설정 파일의 변경 확인, 파일 변경되면 다시 로딩 -->
<configuration scan="true" scanPeriod="30 seconds">
<!-- 	logback정상종료 -->
	<shutdownHook/>

<!-- 	console에 로그 -->
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<layout class="ch.qos.logback.classic.PatternLayout">
			<Pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%logger] - %replace(%msg){'[\r\n]+', ''} %n</Pattern>
		</layout>
	</appender>
</configuration>
```

그후 classpath에 logback.xml 파일을 추가한다. appender태그는 로그를 출력하는 방법에대한 설정과 출력 패턴에 대한 정의를 할 수 있다.
