---
layout: post
title: CentOS7을 통해 Apache , tomcat 연동하는 2가지 방법
featured-img: main.png
categories: ['linux']
author: hbshin
---


## Mod_jk 방식

- apache , tomcat 연동을 하기 위한 모듈 
- AJP 프로토콜을 이용해 요청 중 톰캣이 처리할 요청을 AJP포트를 통해 tomcat에 전달하고 응답받는 역할을 수행 

### AJP 란?

- JP는 웹서버(Apache) 뒤에 있는 어플리케이션 서버로부터 웹서버로 들어오늘 요청을 위임할 수 있는 바이너리 프로토콜이다.


### 설치

기본적으로 apache , tomcat 필요
```
# yum install -y httpd
```
- 먼저 (http://tomcat.apache.org) tomcat 웹사이트에서 tomcat connectors 메뉴에서 최신버전의 connector 링크 주소 복사를 통해 가져와서 설치하고 압축파일을 풉니다. 

```
# wget tomcat-connectors의 최신버전링크

# tar xvfz tomcat-connectors-버전-src
```

```
# cd ./tomcat-connectors-버전-src/native/
# ./configure --whth-apps=/bin/apxs
# make
# make install
```

- 위에 작업을 모두 마쳤으면 설치를 확인합니다.
```
/SW/web/httpd-2.4.48/modules/mod_jk.so
```

![Modjk](../image/hbshin/20210825/Modjk.PNG)

## centOS7 - apache + tomcat connector

```
# /usr/local/lib/apache-tomcat-8.5.70/conf/server.xml 
```

- tomcat 설정파일 server.xml파일 안에 주석처리 되어있는 AJP포트 설정을 주석해제



![server](../image/hbshin/20210825/server.PNG)


- 주석을 지우고 tomcat을 재기동하면 8009포트를 netstat상에서 확인가능.

```
vi 나 nano를 통해 apache(httpd) /conf/httpd.conf 설정 파일에 LoadModule jk_modules/mod_jk.so 추가
```


![loadmodule](../image/hbshin/20210825/loadmodule.PNG)


### apache와 연동할 tomcat 을 설정하는 파일 생성



```
vi 나 nano를 통해 httpd/conf/workers.properties 접속 후 내용추가
```



![workers](../image/hbshin/20210825/workers.PNG)


- 모두 완료되었으면 tomcat 과 apache 재기동 하고 웹브라우저에 :8080 포트없이 본인 IP만으로 접속확인


![result](../image/hbshin/20210825/result.PNG)



## Mod_Proxy_ajp 방식

- 이전 mod_jk 방식과 비슷한 방법으로 설정합니다.

```
apache(httpd) /conf/httpd.conf
```

- 저는 httpd.conf 파일에서 Mod_jk는 주석처리하고 

```
#LoadModule proxy_module modules/mod_proxy.so
#LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
```
- proxy방식을 이용하기 위해서 이 두가지를 주석을 풀었습니다.
- 그 다음 하단부에 IP로 들어온 요청에 대해서 AJP를 통해 tomcat으로 보내도록 아래와 같이 설정했습니다.

![프록시V](../image/hbshin/20210825/프록시V.PNG)

- 이제 결과를 확인했을때 아래와 같이뜨면 성공입니다.

![result](../image/hbshin/20210825/result.PNG)

