title: minio利用docker安装集群环境
date: 2020-07-14 11:09:04
categories: 环境部署
tags: 
	- docker
	- minio
---
两台虚拟机

192.168.181.132（data,data1）

192.168.181.133（data,data1）

执行以下命令：

node1

```
docker run -d  --net=host --restart=always --name minio-node1 -e "MINIO_ACCESS_KEY=minio"   -e"MINIO_SECRET_KEY=minio123"   -v /data:/data -v /data1:/data1   minio/minio server http://192.168.181.132:9000/data/ http://192.168.181.132:9000/data1/ http://192.168.181.133:9000/data/ http://192.168.181.133:9000/data1/
```

node2

```
docker run -d  --net=host --restart=always --name minio-node2 -e "MINIO_ACCESS_KEY=minio"   -e"MINIO_SECRET_KEY=minio123"   -v /data:/data -v /data1:/data1   minio/minio server http://192.168.181.132:9000/data/ http://192.168.181.132:9000/data1/ http://192.168.181.133:9000/data/ http://192.168.181.133:9000/data1/
```

注意看这两个脚本不同之处只有一个地方：--name启的别名不一样，其它一模一样。这里我用--net=host可以创建但是改成-p 9000:9000就不通了。

这样访问：http://192.168.181.132:9000/给里面上传文件 ，另一个节点自动同步。