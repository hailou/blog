---
title: 解决nginx做反向代理后，端口丢失问题
date: 2020-07-30 16:10:17
categories: 环境部署
tags: 
	- nginx
    
在部署微服务项目中，发现nginx做反向代理后，端口丢失，以为项目代码出错，最后发现是nginx配置的问题

```
upstream uaa_server {
   server 10.108.0.101:8000;
   server 10.108.0.102:8000;
}

server {
    listen 8000;
    server_name localhost;

    charset utf-8;
    
    location / {
        proxy_read_timeout 90;
        proxy_pass http://uaa_server;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
    }

}
```

