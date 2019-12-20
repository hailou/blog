title: linux安装nginx 配置https
categories: 环境部署
tags:
  - nginx
date: 2019-12-13 11:40:57
---
###### 安装命令：(一定按照这样操作，不然后面很麻烦)
```
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel

//下载tar包
wget http://nginx.org/download/nginx-1.17.0.tar.gz

##进入nginx目录
cd nginx-1.17.0
## 配置
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module

# make
make
make install
```

<!-- more -->

###### 安装完成后，测试是否安装成功
```
cd 到刚才配置的安装目录/usr/local/nginx/
./sbin/nginx -t
```
###### 这样就是成功的，如果按照我上面的来的话，基本是没有问题的
```
[root@MiWiFi-R3P-srv nginx]# ./sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```
###### 启动nginx
```
cd /usr/local/nginx/sbin
./nginx //启动nginx
```
###### 下面是配置nginx 的https conf
配置ssl证书之前，先准备SSL证书，至于获取的途径很多（阿里云的服务，第三方服务购买）

准备好证书后，找到nginx的安装目录，我的安装位置为：/usr/local/nginx 打开nginx下面的conf 配置里面的nginx.conf ,新手可以把这个配置文件拉到本地配置

开始配置文件的修改在修改配置文件之前，最好做一个备份，防止修改错误，也能及时回退错误
```
server {
        listen       80;
        server_name  需要访问的域名;

        rewrite ^(.*) https://$server_name$1 permanent; #这句是代表 把http的域名请求转成https

        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass   http://需要访问的域名; #因为这里还是80端口，所以保持http就可以了

        }
    }
```
这里除了HTTPS server这行之外，其他的 # 删除，启动https模块
```
 # HTTPS server
    #
    server {
        listen       443 ssl;
        server_name  需要访问的域名，这里也不用加https;

        ssl_certificate      /usr/local/nginx/ssl/ssl.pem;  #这里是ssl key文件存放的绝对路径，根据自己的文件名称和路径来写
        ssl_certificate_key  /usr/local/nginx/ssl/ssl.key;  #这里是ssl key文件存放的绝对路径，根据自己的文件名称和路径来写

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass   http://需要访问的域名:8080/;
        }
    }
```
接下来就是要让配置文件生效：进去nginx的sbin文件夹，我的sbin文件夹在：/usr/local/nginx/sbin执行以下语句：检验配置文件是否有错误
```
./nginx -t

#成功就是这样（安装我的流程走下来，一定是成功的）
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```
[转自](https://www.jianshu.com/p/1ad30370b604)