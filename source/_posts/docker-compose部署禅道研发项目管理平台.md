title: docker-compose部署禅道研发项目管理平台
categories: 环境部署
tags:
  - Git
date: 2020-06-03 19:32:25
---

### 镜像准备

> mkdir zentao

### 编写docker-compose.yml文件

> cd zentao
>
> vim docker-compose.yml

```
version: "3"

services:
  zentao:
    image: idoop/zentao:latest
    container_name: ZentaoApplication
    restart: always
    environment:
      ADMINER_USER: "admin"
      ADMINER_PASSWD: "admin123"
      BIND_ADDRESS: "false"
    ports:
      - 8091:80
    volumes:
      - ./etc/localtime:/etc/localtime:ro
      - ./etc/timezone:/etc/timezone
      - ./data/zbox/:/opt/zbox/
```

ADMINER_USER ：设置 Web 登录数据库管理员帐户

ADMINER_PASSWD ：设置 Web 登录数据库的管理员密码

BIND_ADDRESS：如果使用设置值 false，MySQL 服务器将不绑定地址

SMTP_HOST：设置smtp服务器的 IP 和主机。（如果无法发送邮件，将会有所帮助。）也可以 extra_host 在 docker-compose.yml 中使用，或在使用 docker run 命令时使用 --add-host

禅道管理员帐户为 admin，默认初始化密码为 123456。MySQL 的 root 帐户密码为 123456，在首次登录时更改密码。

### 启动服务

>  docker-compose up -d