---
layout: post
title: message in Spring
featured-img: Spring.png
categories: ["spring", "SpringFramework", "SpringBoot", "properties", "message"]
author: jay
---

# Spring에서 message 사용하기
<br>
<hr style="border:1px solid gray">
<br>
<br>
Message를 활용하면 유저에게 보여줘야하는 정보를 Locale에 따라 출력하거나 관리하기에 용이하다.
<br>
<br>

![message]](../image/jay/20210704/xml.PNG)
<br>
우선 MessageSource를 사용하기 위해 xml 에 설정을 등록한다.
<br>
MessageSource는 메세지를 다국화할 수 있는 인터페이스이다. 
<br>
class에는 StaticMessageSource, ResourceBundleMessageSource, ReloadableResourceBundelMessageSource 등을 쓸 수 있다.
<br>
여기서 사용한 ReloadableResourceBundelMessageSource는 주기적으로 reloading이 가능하기 때문에 Application을 다시 빌드하지 않아도 반영이 가능하다.
<br>
basename을 classpath:/message/message 처럼 지정하면 classpath의 message 디렉토리 밑에 message로 시작하는 proerties 들을 스캔한다.
<br>
<br>
또 MessageSourceAccessor는 메시지에 쉽게 액세스 할 수있는 클래스로 오버로드 된 다양한 getMessage 메소드를 사용할 수 있다.
<br>
이 MessageSourceAccessor를 커스텀 자바 클래스와 매핑하면 간단하게 MessageUtil을 만들 수 있다.
<br>
<br>
그리고 localeResolver의 defaultLocale 프로퍼티로 기본 locale을 한국으로 설정했다.
<br>
<br>

![message]](../image/jay/20210704/util.PNG)
<br>
MessageUtil에서는 위에서 설명한 것 처럼 getMessage를 사용했다.
<br>
xml 에서는 이 클래스와 MessageSourceAccessor을 매핑시켜 bean으로 주입했다.
<br>
<br>
이제 메세지로 출력될 파일을 만들건데 네이밍 규칙이 존재한다.
<br>
<br>
<hr style="border:1px solid gray">
 OOO.properties : Default Message로 시스템의 언어 및 지역에 맞는 Property 파일이 존재하지 않을 경우 사용
<br>
 OOO_en.properties : 시스템 언어 코드가 영어일 때 사용
<br>
 OOO_ko.properties : 시스템 언어 코드가 한글일 때 사용
<br>
 OOO_en_UK.properties : 시스템 언어 코드가 영어일때 영국(국가코드)을 위한 메시지 
<br>
<hr style="border:1px solid gray">
<br>
<br>
위의 규칙에서 OOO는 xml에서 설정한 basename의 마지막(여기서는 message)이 들어가야 한다.
<br>
<br>

![message]](../image/jay/20210704/msg.PNG)
<br>
src/main/resource/message에 message_ko_KR.properties 파일을 만들고 메세지를 입력했다.
<br>
<br>

![message]](../image/jay/20210704/cnt.PNG)
<br>
그 후 Controller에서 MessageUtil.getMessage 메소드에 key를 입력하여 메세지를 가져온 후 반환했다.
<br>
<br>

![message]](../image/jay/20210704/res.PNG)
<br>
결과가 잘 출력되는것을 확인할 수 있다.
<br>
<br>









