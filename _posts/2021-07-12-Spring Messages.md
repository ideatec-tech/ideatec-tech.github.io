---
layout: post
title: spring message
featured-img: springLogo.jpg
categories: ["spring", "SpringFramework", "SpringBoot", "Message"]
author: hbshin
---

#spring message 

- 먼저 web.xml에 message-context.xml을 읽을수 있도록 classpath지정
```
<param-value>classpath:spring/message-context.xml</param-value>

```
- message-context-xml 파일 생성
```
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource"> 
		<property name="basenames"> 
			<list>
			<!-- 메세지 파일의 위치를 지정합니다.
			message_언어.properties 파일을 찾습니다.-->
			 	<value>classpath:/messages/message</value>
			</list>
	 	</property> 
	 </bean>
	 <!-- MessageSource를 사용하기 위한 Accessor 설정 -->
    <bean id="messageSourceAccessor" class="org.springframework.context.support.MessageSourceAccessor">
        <constructor-arg ref="messageSource"/>
    </bean>
     
    <!-- MessageSource를 사용하기위한 MessageUtils 매핑 -->
    <bean id="message" class="moim.common.util.MessageUtils">
        <property name="messageSourceAccessor" ref="messageSourceAccessor"/>
    </bean>
      
    <!-- Default Location 설정 -->
    <bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver">
        <property name="defaultLocale" value="ko"></property>
    </bean>

```
-  메세지 설정 파일을 셋업하기 위해서는 .properties 확장자가 붙은 프로퍼티 파일에 [파일이름]_[언어]_[국가].properties 형식으로 메세지 파일을 추가하고 입력해야한다.

```
msg.name=\uC548\uB155\uD558\uC138
msg.company=\uC774\uB370\uC544\uD14D

```








