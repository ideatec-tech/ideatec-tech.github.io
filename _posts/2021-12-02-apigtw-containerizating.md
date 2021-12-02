---
layout: post
title: apigtw-containerizating
featured-img: docker-container.png
categories: ['apigtw']
author: Dan
---


## apigtw-containerizating (~ing)
<br>

### docker-compose up
<br>

```
version: "3"
services:
  db:
    image: mysql:8.0
    restart: always
    command: --lower_case_table_names=1
    container_name: apigtwdb
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=qwer1234
      - TZ=Asia/Seoul
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    volumes:
      - /root/data/apigtw\mysql:/var/lib/mysql
```
```
- #docker-compose up 명령어로 실행 
```

![1](../image/hbshin/20211202/1.PNG)