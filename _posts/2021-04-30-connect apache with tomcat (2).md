---
layout: post
title: connect apache with tomcat(2)
featured-img: connectApacheTomcat.jpg
categories: ['linux', 'apache', 'tomcat']
author: oscar
---

# linux(centos7) 환경에서 Apache와 tomcat 연동하기(2)

## 1. mod_proxy_ajp 연동

mod_proxy_ajp을 사용하여 연동하는 것은 mod_jk보다 훨씬 간단하다.

아파치 홈/conf/httpd.conf 설정 파일을 설정합니다. nano 편집기로 httpd.conf파일을 열어서 'mod_proxy'와 'mod_proxy_ajp' 모듈을 로드하도록 변경합니다. (주석 풀어줍니다.)
![mod_proxy1](../image/oscar/2021-04-30_mod_proxy/1.png)

그리고 하단에 다음을 추가해줍니다.
![mod_proxy2](../image/oscar/2021-04-30_mod_proxy/2.png)


설정이 다 되었으면 아파치를 재기동 하고 결과를 확인입니다.
```
# systemctl restart httpd
# ps - ef | grep httpd
```

웹브라우저에서 '본인IP'로 접속해서 톰캣사이트가 나오면 성공입니다.
![mod_jk6](../image/oscar/2021-04-30_mod_jk/6.png)


