---
layout: post
title: Spring Properties 
featured-img: springProperties.png
categories: ['spring']
author: oscar
---

# Spring Properties 설정

## 1. 스프링 프레임워크 Properties

1.스프링 프로퍼티 파일을 이용한 값 설정<br>
  설정정보를 XML로 분리해두면 빈 클래스나 의존관계 정보를 소스코드 수정 없이도 간단히 조작할 수 있다.<br>

  때로는 XML에서 다시 일부 설정정보를 별도의 파일로 분리해두면 유용할 때가 있다. 서버환경에 종속적인 정보가 있다면 이를 애플리케이션의 구성정보에서 분리하기 위해서다. (로컬, 개발, 운영에 대한 DB정보가 다를때)<br>

  프로퍼티 파일에서 설정 값을 분리하면 얻을 수 있는 장점은 @Value를 효과적으로 사용할 수 있다는 점이다.<br>

  @Value는 소스코드 안에 포함되는 애노테이션이어서 값을 수정하면 매번 새로 컴파일해야한다. 하지만 @Value에서 프로퍼티 파일의 내용을 참조하게 해주면 소스코드의 수정 없이 @Value를 통해 프로퍼티에 주입되는 값을 변경할 수 있다.<br>

### Spring Property 설정 방법 

#### 1. context:property-placeholder 태그 사용

```
<context:property-placeholder location="classpath:프로퍼티파일명"/>
```
<br>

* 빈팩토리 후처리기로써 빈 설정 메타정보 모두 준비됐을 때 메타정보 자체를 조작 : 빈 설정이 끝난 이후 ${} 표현을 찾아 해당하는 프로퍼티로 치환해준다는 의미인 듯<br>
* ${} 사용 : "${프로퍼티key}" 와 같이 사용<br>
* @Value annotation 에서도 사용이 가능 : @Value("${프로퍼티key}")
<br><br>

#### 2. bean으로 등록

```
<util:properties id="빈아이디" location="classpath:프로퍼티파일명" />
```
<br>

* Spring 3.0 이상부터 지원<br>
* 프로퍼티 파일 내용을 Properties 타입의 빈으로 생성<br>
* spEL(Spring Expression Language) 사용 : #{빈아이디['프로퍼티Key']} 와 같이 사용<br>
* @Value annotation 에서도 사용이 가능 : @Value("빈아이디['프로퍼티Key']")
<br><br>


common.properties 파일생성 (resources/properties/common.properties)<br>

```
ideatec.edu.test=IDEATEC 
ideatec.edu.jndi.name=java:comp/env/jdbc/edudb
```
<br><br>

property 설정(Bean 주입, properties-context.xml 설정)<br>

```
<context:property-placeholder location="classpath:properties/common.properties" />
혹은
<util:properties id="properties" location="classpath:properties/common.properties" />
```

#### 스프링 설정 파일에서 사용

1.context:property-placeholder 태그 사용한 property 설정시 
```
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
	<property name="jndiName" value="${ideatec.edu.jndi.name}"></property>	
</bean>
```
<br><br>

2.util:properties 와 같이 bean 으로 property 설정시 
```
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
	<property name="jndiName" value="#{properties['ideatec.edu.jndi.name']}"></property>	
</bean>
```
<br><br>

#### 소스에서의 사용

Controller에서 클래스 상단에<br>
PropertySource 어노테이션을 설정하고, 읽어 올 properties를 지정한다. @Value 어노테이션으로 데이터를 가져온다.<br>
또는 org.springframework.core.env.Environment를 import 해서 getProperty()로 데이터를 가져온다.
```
import org.springframework.core.env.Environment;

@RestController
@RequestMapping(value = "/test")
@PropertySource("classpath:properties/common.properties")
public class TestController {

	@Autowired
	private Environment env;	

	@Value("${ideatec.edu.test}")
	private String test;

	@RequestMapping(value = "/hellow3", method = RequestMethod.GET)
	public ResponseEntity<?> hellow3() throws Exception {

		String envData = env.getProperty("ideatec.edu.test");
		
		Map<String, String> out = new HashMap<>();
		out.put("result1", test);
		out.put("result2", envData);
		
		return new ResponseEntity<Map<String, String>>(out, HttpStatus.OK);
	}
}
```
<br><br>


## 2. 스프링 부트 Properties

common.properties 파일생성 (resources/properties/common.properties)<br>

```
ideatec.edu.test=IDEATEC 
ideatec.edu.jndi.name=java:comp/env/jdbc/edudb
```
<br><br>

#### 소스에서의 사용

Controller에서 클래스 상단에 PropertySource 어노테이션을 설정하고, 읽어 올 properties를 지정한다.
```
@RestController
@RequestMapping(value = "/test")
@PropertySource("classpath:properties/common.properties")
public class TestController {

	@Value("${ideatec.edu.test}")
	private String test;

	@RequestMapping(value = "/hellow3", method = RequestMethod.GET)
	public ResponseEntity<?> hellow3() throws Exception {
						
		System.out.println(test);
		
		Map<String, String> out = new HashMap<>();
		out.put("result", test);
		
		return new ResponseEntity<Map<String, String>>(out, HttpStatus.OK);
	}
}
```
<br><br>

