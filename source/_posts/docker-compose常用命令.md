---
title: docker-compose常用命令
date: 2017-09-19 15:48:22
tags:
---

### 构建本地开发环境

``` bash
$ docker-compose up
$ docker-compose down
$ docker-compose build
```

#### 编排文件 `docker-compose.yml`

``` bash
version: "3"
services:
  xw-wostore:
    build: .
    volumes:
      - ./target/xw-wostore:/usr/local/tomcat/webapps/ROOT
    ports:
      - "80:8080"
```
