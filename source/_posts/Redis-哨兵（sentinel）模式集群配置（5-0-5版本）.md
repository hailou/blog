title: Redis 哨兵（sentinel）模式集群配置（5.0.5版本）
categories: 环境部署
tags:
  - redis
date: 2019-12-01 16:11:56
---

##### 系统环境：CentOS7
##### 服务器2台（1主1从）：192.168.31.2（master）192.168.31.3（slave）
##### redis版本：5.0.5
##### 安装：

* 进入到目录：cd /usr/local
* 下载redis：wget http://download.redis.io/releases/redis-5.0.5.tar.gz
* 下载完成后解压：tar zxvf redis-5.0.5.tar.gz
* 重命名为redis文件夹：mv redis-5.0.5 redis
* 进入到redis文件夹：cd redis
* 编译及安装：make && make install
* 特别说明：官方文档只给出了make（编译），没有给出make install（安装）

<!-- more -->

##### 配置：
* Master配置，把master的redis.conf拷贝了一份命名为redis_master.conf
* 内容如下：
```
### NETWORK 设置：
#bind 0.0.0.0
#保护模式关闭，这里使用密码访问
protected-mode no
#设置端口，建议测试时可以使用默认端口，我这里改掉了，建议生产环境均使用自定义端口
port 17000
#Client 端空闲断开连接的时间
timeout 30

### GENERAL 设置：
#后台模式运行
daemonize yes
#pid进程文件名
pidfile /var/run/redis_17000.pid
#日志文件的位置
logfile /usr/local/redis/logs/redis.log

### SNAPSHOTTING 设置：
#快照文件的路径
dir /usr/local/redis/datas

### APPEND ONLY MODE 设置：
#默认值是No，意思是不使用AOF增量持久化的方式，使用RDB全量持久化的方式。把No值改成Yes，使用AOF增量持久化的方式
appendonly yes
appendfsync always

###SECURITY 设置密码:，生产环境一定要使用复杂密码
requirepass 123456

#必需配置这个，不然当主节点死了后，从替代主。原来的主节点启动后变成从节点后，数据不能同步，因为从变主后，认证不通过，这个就需要把原来的从的密码给配置上。
masterauth 123456
```
* Slave配置，我把master的redis.conf拷贝了一份命名为redis_slave.conf
* 内容如下：
```
### NETWORK 设置：
# bind 127.0.0.1 #注释掉bind，任何ip均可访问
#设置端口
port 17000
#保护模式关闭，使用密码访问
protected-mode no
#Client 端空闲断开连接的时间
timeout 30

### GENERAL 设置：
#后台模式运行
daemonize yes
#pid进程文件名
pidfile /var/run/redis_17000.pid
#日志文件的位置
logfile /usr/local/redis/logs/redis.log

### SNAPSHOTTING 设置：
#快照文件的路径
dir /usr/local/redis/datas

### REPLICATION 设置：
#主服务器的Ip地址和Port端口号
replicaof 192.168.31.2 17000
#如果slave 无法与master 同步，设置成slave不可读，方便监控脚本发现问题。
replica-serve-stale-data no
master的密码
masterauth 123456

### APPEND ONLY MODE 设置：
#默认值是No，意思是不使用AOF增量持久化的方式，使用RDB全量持久化的方式。把No值改成Yes，使用AOF增量持久化的方式
appendonly yes
appendfsync always

###SECURITY 设置密码:，生产环境一定要使用复杂密码
requirepass 123456
```
* Sentinel（哨兵）配置，配置文件为sentinel.conf
* 内容如下：
```
#哨兵端口号
port 16000
#关闭保护模式
protected-mode no
#守护进程
daemonize yes
#哨兵程序的工作路径
dir /usr/local/redis/sentinel/
#哨兵程序的工作日志文件
logfile "sentinel.log"

#Sentinel去监视一个名为mymaster的主redis实例，这个主实例的IP地址为本机地址192.168.31.2，端口号为17000，而将这个主实例判断为失效至少需要1个 Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行
#这个一定要放在下面这句的上面，不然sentinel启动不了
sentinel monitor mymaster 192.168.31.2 17000 1
#master的访问密码
sentinel auth-pass mymaster 123456
#哨兵程序每5秒检测一次Master是否正常
sentinel down-after-milliseconds mymaster 5000

#指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长
sentinel parallel-syncs mymaster 2
#如果在该时间（ms）内未能完成failover操作，则认为该failover失败，生产环境需要根据数据量设置该值
sentinel failover-timeout mymaster 300000
```
##### 启动Redis服务
* 在各服务器上先建立配置文件中需要的文件夹
 
```
mkdir -p /usr/local/redis/datas
mkdir -p /usr/local/redis/logs
mkdir -p /usr/local/redis/sentinel
```
 
* 启动服务,先开启master服务，也就是192.168.31.2服务器,启动master:
 
```
cd /usr/local/redis/src
./redis-server redis_master.conf
```
* 使用redis-cli访问服务端，查看状态
```
./redis-cli -p 17000 -a 123456
127.0.0.1:17000> info replication
```
* 这里有具体的集群信息（略）
* 启动Sentinel（哨兵）进程   
```
./redis-sentinel sentinel.conf
```
* 说明：哨兵进程不一定与redis数量一致，也不一定要放在redis服务器上，sentinel的作用是监控所有服务及与其它哨兵通信，若sentinel单独放其它服务器上，则也需要安装redis，sentinel只是redis软件包中的一个服务，每台服务器上都放了一个sentinel进程，sentinel的配置都是一样的，将这个sentinel.conf拷贝几个放在slave的机器上。
* 查看下进程：
```
ps aux | grep redis
```
* set aaa a 测试下数据是否同步
##### 故障转移
* 模拟master宕机，这里直接把master进程杀掉 
```
kill -9 2730
```
* 看下从节点是否变主节点，再启动原来主节点，测试是否正确
##### 当sentinel设置2为down（客观下线）后，这些sentinel开始进行投票选举，这里特别注意，选举的不是redis_server服务，而是sentinel服务，意思就是先选举一个sentinel，然后由它决定将哪个slave提升为master。

* 参考了[这里](https://www.cnblogs.com/chensuqian/p/10538365.html)，但实际做的时候遇到了很多问题，解决的方案都体现在配置文件里。