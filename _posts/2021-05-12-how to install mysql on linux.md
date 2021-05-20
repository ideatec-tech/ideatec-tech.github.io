---
layout: post
title: How to install mysql on linux
featured-img: mysqlOnLinux.jpg
categories: ['linux','mysql']
author: oscar
---

# linux(centos7) 환경에서 mysql8.x 설치하기

## 1. 다른 버전 있는지 확인 및 불필요 버전 삭제

아래 명령어를 실행 시켜서 mysql 확인 후 기존 버전이 있다면 삭제를 해줍니다.
```
# yum list installed | grep mysql
```
그리고 centos7에는 기본적으로 mariadb가 설치 되어 있어서 충돌이 일어날 수 있으므로 삭제해 줍니다.

```
# yum list installed | grep mariadb
```

## 2. MySQL rpm 패키지 링크 가져오기

Centos7의 yum에 MYSQL 관려 repository 구축을 위한 rpm 패키지를 다운받습니다.

먼저 [MYSQL DOWNLOAD](https://dev.mysql.com/downloads/) 페이지를 열고 운영체제 및 버전을 선택합니다.

![mysql1](../image/oscar/2021-05-12_mysql/1.png)
<br>
![mysql2](../image/oscar/2021-05-12_mysql/2.png)
설치 서버의 운영체제 버전에 맞게 Download 누르면 되는데 저는 Centos7이기 때문에 두번째를 선택합니다.
<br>
![mysql3](../image/oscar/2021-05-12_mysql/3.png)
하단의 No thanks,just start my download. 부분을 우클릭 해서 링크 주소를 복사해서 rpm 패키지 설치를 진행하면 됩니다.
<br><br>

## 3. yum localinstall 명령어로 MySQL rpm 패키지 설치

localinstall 명령어로 rpm 파일 이용해서 repository를 구성합니다.
```
# yum localinstall https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```
rpm 파일이 정상적으로 설치되었을 경우 'yum repolist' 명령어로 yum repository 목록에 mysql 관련 항목이 함께 출력될 것입니다.
![mysql4](../image/oscar/2021-05-12_mysql/4.png)
<br>
mysql 서버 설치 위해 필요한 repository 구성이 완료 되었으니 mysql 서버를 설치합니다.

<br><br>

## 4. mysql-community-server 설치

```
# yum install mysql-community-server
```
yum install 명령어로 mysql-community-server를 설치합니다.

혹시 충돌이 일어날 경우에 다음 명령어로 yum 저장소를 정리 후 다시 설치를 진행해줍니다.

```
# yum clean all
# yum update
```
<br><br>

## 5. mysql 기본 설정 및 실행

설치가 완료 되었으면 mysql을 기동시킵니다.
```
# systemctl start mysqld
```

접속하기 전에 전에 미리 임시비밀번호를 확인합니다.
```
# nano /var/log/mysqld.loc
```
![mysql5](../image/oscar/2021-05-12_mysql/5.png)
<br>

접속해봅니다.
```
# mysql -u root -p
password : 
```

그런데 비밀번호 규칙이 엄격해져서 비밀번호 바꾸기 위해서 보안 규칙을 내려야 합니다. 그렇지 않으면 다음과 같은 에러 발생합니다.
![mysql6](../image/oscar/2021-05-12_mysql/6.png)

다음 명령어를 통해서 정책을 수정해줘야 한다.
```
# nano /etc/my.cnf
```
하단에 다음 내용을 추가해 줍니다.
```
#캐릭터셋
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

#비밀번호 정책
validate_password.policy=LOW
```

수정을 했으면 mysql을 재기동 시켜 줍니다.
```
# systemctl restart mysqld
```

접속 후 다음과 같이 비밀번호를 변경해 줍니다.
```
# mysql -u root -p
password : 

alter user 'root'@'localhost' identified by '변경할 비밀번호';
flush privileges;
```







