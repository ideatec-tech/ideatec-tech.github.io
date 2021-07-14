---
layout: post
title: Spring, Spring Boot message
featured-img: message11.PNG
categories: ["SpringFramework", "SpringBoot", "message"]
author: hbshin
---


## Spring Message

- message를 사용하기 위해서는 먼저 context.xml에 필요한 설정을 합니다.

```
		<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
			<property name="basenames">
				<!-- message폴더 안에 있는 label 을 지정 -->
				<list>
					<value>message/message</value>
				</list>
			</property>
		<property name="defaultEncoding" value="UTF-8" />
	</bean>
	<!-- MessageSource를 사용하기 위한 Accessor 설정 -->
    <bean id="messageSourceAccessor" class="org.springframework.context.support.MessageSourceAccessor">
        <constructor-arg ref="messageSource"/>
    </bean>
     
    <!-- MessageSource를 사용하기위한 MessageUtils 매핑 -->
    <bean id="message" class="ideatec.edu.spring.frwk.tomcat.util.MessageUtils">
        <property name="messageSourceAccessor" ref="messageSourceAccessor"/>
    </bean>
      
    <!-- Default Location 설정 -->
    <bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver">
        <property name="defaultLocale" value="ko"></property>
    </bean>

```
1. message폴더안에 있는 properties파일의 위치를 지정하는데 2가지 방식이 있다.
- ResourceBundleMessageSource 와 ReloadableResourceBundleMessageSource 이다.
- 여기서 주의할점은 ResourceBundleMessageSource는 resources파일을 직접읽기때문에 classpath를 따로 지정하지 않아도되고 ReloadableResourceBundleMessageSource 은 classpath를 지정해줘야한다.
2. MessageSource를 사용하기 위해서 Accessor 설정을하고
3. MessageSource를 간편하게 사용하기 위해 MessageUtils를 매핑한다.
4. Default 값은 한국어로 지정

- 다음으로는 메시지를 불러올 값을 key value형태로 properties파일에 작성한다.
- properties명은 message_ko_KR.properties로 지정
```
# message_ko_KR.properties
message.start=start
message.stop=stop
```

- Controller를 통해 메시지를 출력하기전 context에서 messageSource를 쉽게 사용하기 위해 매핑했던 MessageUtils파일을 작성
```
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

- Controller 작성

```
@RestController
@RequestMapping(value = "/test")
public class TestController {

	@RequestMapping(value = "/msg", method = RequestMethod.POST)
	public ResponseEntity<?> test() {

		
		String msg1 = MessageUtils.getMessage("message.start");
		String msg2 = MessageUtils.getMessage("message.stop");

		Map<String, String> result = new HashMap<>();
		result.put("msg1", msg1);
		result.put("msg2", msg2);

		return new ResponseEntity<>(result, HttpStatus.OK);
	}

}

```


- 결과 출력

![msgResult](../image/hbshin/20210714/msgResult.PNG)


## Spring boot Message

- spring boot 에서는 config파일을 만들어서 파일위치와 인코딩 설정 , MessageSource를 사용했다.

```
@Configuration
public class MsgConfig {

	@Bean
	public MessageSource messageSource() {
		
		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource(); 
		messageSource.setBasenames("message/message"); 
		messageSource.setDefaultEncoding("UTF-8");
		
		return messageSource;
	}
	
}

```

- properties파일명과 작성은 spring과 큰 차이는 없다. 

```
MyCompany=\uD68C\uC0AC\uC774\uB984 : {0}

```


- boot는 더 간단하게 @Component를 사용해서 빈클래스에서 빈을 직접 등록하고 MessageSource 주입받아 사용했다.

 ```

@Component
public class MsgRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(messageSource.getMessage("MyCompany", new String[]{"이데아텍"}, Locale.KOREA));
    }
}


 ```


 - 결과 출력

 ![result](../image/hbshin/20210714/result.PNG)
 