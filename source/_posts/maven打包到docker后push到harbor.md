---
title: maven打包到docker后push到harbor
date: 2020-07-09 13:10:17
categories: 环境部署
tags: 
	- docker
discription: maven打包到docker后push到harbor
copyright: ture
---

公司项目需要，开始玩docker，开始折腾harbor，各种百度各种问题，满满的搞了一整天，终于搞定了maven打包到docker，然后push到harbor，现记录一下自己研究的成果。现在环境为远程私有仓库在阿里云，本地安装docker DeskTop，流利是先打包到本地docekr容器再推送到harbor。

#  Ubuntu 安装 Docker

```
apt-get remove docker docker-engine docker.io containerd runc
```

```
apt install docker.io
```

## 阿里云加速器（推荐）

[点击链接获取](https://promotion.aliyun.com/ntms/act/qwbk.html?userCode=hgqku7c5)

# Docker Compose 安装

Compose 支持 Linux、macOS、Windows 10 三大平台。在 Linux 上的也安装十分简单，从 [官方 GitHub Release](https://github.com/docker/compose/releases) 处直接下载编译好的二进制文件即可。

```
curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 验证

```
docker-compose version

# 输出如下
docker-compose version 1.24.0, build 0aa59064
docker-py version: 3.7.2
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.1.0j  20 Nov 2018
```

# 安装harbor

```ruby
# 下载
wget https://github.com/goharbor/harbor/releases/download/v1.10.1/harbor-offline-installer-v1.10.1.tgz
# 解压
tar  -xvf harbor-offline-installer-v1.10.1.tgz
```

## 配置

### 打开文件 harbor.yml

```php
# 修改hostname，也可使用ip：132.232.22.xxx
hostname: harbor.example.com

# 修改端口号，默认为80，可以默认
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

# 使用https修改，不需要则注释即可
#https:
  # https port for harbor, default is 443
  #port: 443
  # The path of cert and key files for nginx
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path

# 修改admin的密码
harbor_admin_password: Harbor12345
```

注意：修改三个地方：1，hostname为harbor的仓库IP.。2，注释掉https开头的这一堆，因为我打算用账户密码访问。3，修改harbor_admin_password

### 安装

```bash
# 检查
./prepare
# 安装
./install
```

## 使用

登录浏览器(**因为我用的是http访问，所以用火狐，如果用谷歌，可能会登录不上**)，使用上面配置的域名或者ip+port，进入harbor内创建一个项目test

# 安装Docker DeskTop

## 打开客户端，修改docker Engine 

{
  "registry-mirrors": [],
  "insecure-registries": ["47.91.220.3111"],
  "debug": true,
  "experimental": true
}

其中：insecure-registries为harbor的地址，开启http请求，experimental为true，能从公网下载镜像。

![](http://blog.osyun.net/20200709141834.png)

##  勾选如下

![](http://blog.osyun.net/20200709141921.png)

## 测试登录

在cmd中输入

docker login 47.91.220.3111 输入用户名和密码（harbor的用户名和密码）

# maven配置

    <server>
       <!--maven的pom中可以根据这个id找到这个server节点的配置-->  
       <id>docker-harbor-registry</id>
       <!--远程harbor的用户名-->  
       <username>admin</username>
       <!--远程harbor的密码-->  
       <password>Harbor12345</password>
     </server>
```
<!--项目名,需要和Harbor中的项目名称保持一致 -->
<docker.image.prefix>mypro</docker.image.prefix>
<!-- docker私服地址,Harbor配置完默认地址就是服务器IP地址,默认不带端口号 -->
<docker.registry>47.91.220.3111</docker.registry>
<!--项目名,需要和Harbor中的项目名称保持一致 -->
<docker.registry.name>test</docker.registry.name>
```

```
<build>
    <finalName>app</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <!-- Docker maven plugin -->
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>1.2.2</version>
            <configuration>
                <serverId>docker-harbor-registry</serverId>
                <registryUrl>http://${docker.registry}</registryUrl>
                <!--必须配置dockerHost标签（除非配置系统环境变量DOCKER_HOST）-->
                <dockerHost>http://localhost:2375</dockerHost>
                <!--Building image 47.91.220.3111 /demo1-->
                <imageName>${docker.registry}/${docker.registry.name}/${project.build.finalName}:${project.version}</imageName>

                <dockerDirectory>src/main/docker</dockerDirectory>
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
        </plugin>
        <!-- Docker maven plugin -->
    </plugins>
</build>
```

Dockerfile内容（src/main/docker下）

```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD *.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar"]
```

# 部署在其它服务器

## docker开启http

如果不开启docker login时默认是https

```
vim /etc/docker/daemon.json

  "insecure-registries": ["47.91.220.3111"]

systemctl daemon-reload

systemctl restart docker
```



记录几个常用：

docker pull 47.91.220.3111/test/app:0.0.1

docker run -p 8989:8989 -name app -d  47.91.220.3111/test/app:0.0.1

docker logs -f -t --since="2017-05-31" --tail=10 edu_web_1

--since : 此参数指定了输出日志开始日期，即只输出指定日期之后的日志。

-f : 查看实时日志

-t : 查看日志产生的日期

-tail=10 : 查看最后的10条日志。

edu_web_1 : 容器名称