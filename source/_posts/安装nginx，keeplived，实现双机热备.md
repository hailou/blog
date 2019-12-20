title: 安装nginx，keeplived，实现双机热备
date: 2019-11-22 11:09:04
categories: 环境部署
tags: 
	- keeplived
	- nginx
---
### 安装nginx 
详见《linux安装nginx 配置https》
### 安装keepalived
```
yum install -y keepalived
```

<!-- more -->

###### 修改配置文件
```
vi /etc/keepalived/keepalived.conf
```
```
! Configuration File for keepalived
global_defs {
	## keepalived 自带的邮件提醒需要开启 sendmail 服务。 建议用独立的监控或第三方 SMTP
	router_id hailoupc ## 标识本节点的字条串，通常为 hostname
} 
## keepalived 会定时执行脚本并对脚本执行的结果进行分析，动态调整 vrrp_instance 的优先级。如果脚本执行结果为 0，并且 weight 配置的值大于 0，则优先级相应的增加。如果脚本执行结果非 0，并且 weight配置的值小于 0，则优先级相应的减少。其他情况，维持原本配置的优先级，即配置文件中 priority 对应的值。
vrrp_script chk_nginx {
	script "/etc/keepalived/nginx_check.sh" ## 检测 nginx 状态的脚本路径
	interval 2 ## 检测时间间隔
	weight -20 ## 如果条件成立，权重-20
}
## 定义虚拟路由， VI_1 为虚拟路由的标示符，自己定义名称
vrrp_instance VI_1 {
	state MASTER ## 主节点为 MASTER， 对应的备份节点为 BACKUP
	interface enp0s3 ## 绑定虚拟 IP 的网络接口，与本机 IP 地址所在的网络接口相同， 我的是 eth0
	virtual_router_id 33 ## 虚拟路由的 ID 号， 两个节点设置必须一样， 可选 IP 最后一段使用, 相同的 VRID 为一个组，他将决定多播的 MAC 地址
	mcast_src_ip 192.168.199.185 ## 本机 IP 地址
	priority 100 ## 节点优先级， 值范围 0-254， MASTER 要比 BACKUP 高
	nopreempt ## 优先级高的设置 nopreempt 解决异常恢复后再次抢占的问题
	advert_int 1 ## 组播信息发送间隔，两个节点设置必须一样， 默认 1s
	## 设置验证信息，两个节点必须一致
	authentication {
		auth_type PASS
		auth_pass 1111 ## 真实生产，按需求对应该过来
	}
	## 将 track_script 块加入 instance 配置块
	track_script {
		chk_nginx ## 执行 Nginx 监控的服务
	} #
	# 虚拟 IP 池, 两个节点设置必须一样
	virtual_ipaddress {
		192.168.199.130 ## 虚拟 ip，可以定义多个
	}
}
```
###### 注意: 主存机上面不同的地方改四个：1，state（MASTER/BACKUP） 2,mcast_src_ip(主存机的ip) 3 priority （主机大于从机）4，router_id （hailoupc自行设置就行，可以两台机子不一样）virtual_ipaddress这个ip一定要是能访问到的虚拟ip，是网段内的一个有效ip，可以不存在但并必须有效
######  编写 Nginx 状态检测脚本
```
vi /etc/keepalived/nginx_check.sh
```
```
#!/bin/bash
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then
/usr/sbin/nginx
sleep 2
if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
	killall keepalived
fi
fi
```
###### 注意/usr/sbin/nginx这个路径要和nginx对应上
###### 保存后，给脚本赋执行权限：
```
chmod +x /etc/keepalived/nginx_check.sh
```
###### 启动 Keepalived
```
service keepalived start
```