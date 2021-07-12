---
layout: post
title: Spring Message 
featured-img: springMessage.jpg
categories: ['spring']
author: oscar
---

# Spring Message 설정

## 1. 스프링 프레임워크 Message

1.Spring MessagSource<br>
  국제화(i18n)을 제공하는 인터페이스이다. 메세지 설정 파일을 모아놓고 각 국가마다 국가마다 로컬라이징을 함으로서 쉽게 각 국가에 맞춘 메세지를 제공할 수 있다.

### message.properties 파일 설정

message.properties 파일 안에 값들은 key와 value 형태로 정의합니다.<br>
프로퍼티 값에는 MessageFormat 클래스가 해석할 수 있는 메시지 문자열을 지정해야 합니다.
<br>

```
welcome.message=\uD658\uC601\uD569\uB2C8\uB2E4.
```
<br>

### message-context.xml 파일 설정

다음엔 message-context.xml 파일을 만들어  basenames라는 프로퍼티 밑에 파일의 위치를 지정해줍니다.<br>
이 때 뒤에 .properties 는 생략합니다.
<br><br>
스프링이 외부에서 메시지를 가져오기 위해서는 MessageSource라는 인터페이스를 사용합니다.<br>
스프링 프레임워크에서는 다양한 MessageSource의 구현체들을 제공하는데 대표적인 예가 ResourceBundleMessageSource 와 ReloadableResourceBundleMessageSource 이다.<br>

ReloadableResourceBundleMessageSource는 일정 주기로 업데이트를 확인하는 기능을 합니다.
<br>

### contorller 설정

메시지에 접근하기 위해서 MessageSource를 주입받고 MessageSource의 .getMessage() 메서드를 이용해 메시지를 가지고 옵니다.

```
@Autowired
private MessageSource messageSource;

@RequestMapping(value = "/hellow2", method = RequestMethod.GET)
public ResponseEntity<?> hellow2() throws Exception {
		
	log.debug(messageSource.getMessage("welcome.message", new String[] {"이데아텍"}, Locale.KOREAN));
	
	Map<String, String> out = new HashMap<>();
	out.put("result", "Hellow World 1");
	
	return new ResponseEntity<Map<String, String>>(out, HttpStatus.OK);
}
```
<br>

### MessageUtils.java 설정

이제 설정한 Bean으로부터 Message Property를 편하게 사용하기 위한 유틸 객체를 작성하겠습니다.
<br>

```
package util;

import java.util.Locale;

import org.springframework.context.support.MessageSourceAccessor;

public class MessageUtils {
    private static MessageSourceAccessor msAcc = null;
    
    public void setMessageSourceAccessor(MessageSourceAccessor msAcc) {
        MessageUtils.msAcc = msAcc;
    }
     
    public static String getMessage(String code) {
        return msAcc.getMessage(code, Locale.getDefault());
    }
     
    public static String getMessage(String code, Object[] objs) {
        return msAcc.getMessage(code, objs, Locale.getDefault());
    }
}
```
<br>

### controller 설정

```
	@RequestMapping(value = "/hellow2", method = RequestMethod.GET)
	public ResponseEntity<?> hellow2() throws Exception {
				
		log.debug(MessageUtils.getMessage("welcome.message"));
		
		Map<String, String> out = new HashMap<>();
		out.put("result", "Hellow World 1");
		
		return new ResponseEntity<Map<String, String>>(out, HttpStatus.OK);
	}
```
<br><br>

## 2. 스프링 부트 Message

### MessageConfig 파일 생성

config라는 이름의 패키지를 만든 후 MessageConfig 설정 파일을 만들어 위와 같이 선언했습니다.
디테일한 설정은 스프링과 같습니다. ReloadableResourceBundleMessageSource 객체 선언 후, basename에 프로퍼티 파일의 위치,
업데이트 확인 주기 설정, 인코딩 설정 등을 설정해준 후 반환 해주는 역할을 합니다.
<br>

```
package spring.boot.tomcat.config;

import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ReloadableResourceBundleMessageSource;

@Configuration
public class MessageConfig {
	@Bean
	public MessageSource messageSource() {
		ReloadableResourceBundleMessageSource source = new ReloadableResourceBundleMessageSource();
		
		source.setBasename("classpath:message/message");
		source.setCacheSeconds(60);
		source.setDefaultEncoding("UTF-8");
		source.setUseCodeAsDefaultMessage(true);
		
		return source;
	}
}
```
<br>

### controller 설정

spring framework 설정과 같다.

```
@Autowired
private MessageSource messageSource;
	
private static final Logger log = LoggerFactory.getLogger(TestController.class);

@RequestMapping(value = "/hellow2", method = RequestMethod.GET)
public ResponseEntity<?> hellow2() throws Exception {
		
	log.debug(messageSource.getMessage("welcome.message", new String[] {"이데아텍"}, Locale.KOREAN));
				
	Map<String, String> out = new HashMap<>();
	out.put("result", "Hellow World 2");
	
	return new ResponseEntity<Map<String, String>>(out, HttpStatus.OK);
}
```




