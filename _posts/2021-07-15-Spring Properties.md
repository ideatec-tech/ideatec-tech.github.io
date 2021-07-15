---
layout: post
title: Spring Properties
featured-img: Spring.png
categories: ['spring', 'preperties']
author: harold
---

# Spring Properties

### Properties 관리의 목적

```
1. properties 파일을 이용하여 중요 정보를 분리, 세분화 시켜 관리한다.
2. ENUM, 변수를 할당하여 사용해도 되지만, Properties파일을 만들어 유지보수를 용이하게 한다.
```



### Example 

```
### ideatec-tech path ###
ideatec.tech.github.io= https://ideatec-tech.github.io/
```



### How to apply

1. 컴파일시 VM Option으로 설정하여 사용.

   ```
   1.1 WAS Server VM Option 
   	-Dspring.profiles.active= " ## absolute properties file path ## "
   	-Dspring.profiles.active= 
   	" ## C:\workspace\projectSrc\resources\common.properties ## "
   
   1.2 @Autowired
   	Environment env;
   	
   	env.getProperty("ideatec.tech.github.io")
   ```

   

2. 빈 설정 xml 파일에 등록하여 사용.

   ```
   2.1 context tag
   	<context:property-placeholder location="classpath:properties/common.properties" />
   	
   2.2 @Service
   	public class ServiceClass {
           @Value("${ideatec.tech.github.io}")
   		private String techPath;
       }
   ```

   

3. Spring Boot 

   ```
   3.1 application.properties / yml
   
   	properties: server.port:8080
   	yml:	server:
   	    		port: 8080
   	
   	@Value("${server.port}")
   	private String port;
   ```

   