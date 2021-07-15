---
layout: post
title: Spring Message
featured-img: Spring.png
categories: ['spring', 'message']
author: harold
---

# Srping Message

### MessageSource 다국어 처리

```
1. 하나의 템플릿을 만들어 다국어 처리를 하여 유지보수를 용이하게 한다.
```



### Example

```
1. {baseName}_{언어코드}_{국가코드}.properties 파일 생성
	- msg_ko.KR.properties
	- greeting= 안녕하세요  ##내용 입력
```



### How to apply

```
1. xml bean 생성
	<bean id="messageSource" 		   class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
		<property name="basenames">
			<list>
				<value>classpath:/messages/msg*</value>
			</list>
		</property>
		<property name="defaultEncoding" value="UTF-8" />
		<property name="fallbackToSystemLocale" value="false" />
	</bean>


	<bean id="messageSourceAccessor" class="org.springframework.context.support.MessageSourceAccessor">
		<constructor-arg ref="messageSource" />
	</bean>

	<bean id="message" class=" ## messageUtil class path ## ">
		<property name="messageSourceAccessor" ref="messageSourceAccessor" />
	</bean>

	<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver">
		<property name="defaultLocale" value="ko" />
	</bean>
```

### MessageUtil

```
public class MessageUtil {
	
	private static MessageSourceAccessor msAcc = null;

	public void setMessageSourceAccessor(MessageSourceAccessor msAcc) {
		MessageUtil.msAcc = msAcc;
	}

	public static String getMessage(String code) {
		return msAcc.getMessage(code, Locale.getDefault());
	}

	public static String getMessage(String code, Object[] objs) {
		return msAcc.getMessage(code, objs, Locale.getDefault());
	}
}
```

### Test

```
@GetMapping(value = "/msg")
public void msg() {
	logger.info("Message Util 1.Key greeting: {}", MessageUtil.getMessage("greeting"));
}

```

### Console

```
11:54:14.150 [INFO ] - Message Util 1.Key greeting: 안녕하세요
```