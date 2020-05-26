---
title: Redis服务器集群实战
date: 2020-05-13 13:49:03
categories: [Redis]
tags: [Redis]
toc: true
---

# Cluster 集群

## 原理

**sentinel** 哨兵模式基本可以满足一般生产的需求，具备高可用性。但是当数据量过大到一台服务器存放不下的情况时，**主从模式**或 **sentinel** 模式就不能满足需求了，这个时候需要对存储的数据进行分片，将数据存储到多个**Redis**实例中。**Cluster** 模式的出现就是为了解决单机 **Redis** 容量有限的问题，将 **Redis** 的数据根据一定的规则分配到多台机器。

**Cluster** 可以说是 **sentinel** 和 **主从模式** 的结合体，通过 **Cluster** 可以实现主从和 master 重选功能，所以如果配置两个副本三个分片的话，就需要六个 **Redis** 实例。因为 **Redis** 的数据是根据一定规则分配到 **Cluster** 的不同机器的，当数据量过大时，可以新增机器进行扩容。

**Redis-Cluster** 采用无中心结构，它的特点如下：

- 所有的 **redis** 节点彼此互联(PING-PONG机制)，内部使用二进制协议优化传输速度和带宽。
- 节点的 `fail` 是通过集群中超过半数的节点检测失效时才生效。
- 所有的节点都是一主一从（也可以是一主多从），其中从不提供服务，仅作为备用
- 客户端与 **redis** 节点直连，不需要中间代理层。客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可。

# 实战

使用集群，只需要将 **Redis** 配置文件中的 `cluster-enable` 配置打开即可。每个集群中至少需要三个主数据库才能正常运行。

## 环境准备

**Redis** 集群需要三个服务器，每个服务器有需要要一个 **salve** 服务器，所以最少要准备六个 **Redis** 服务器：

```bash
两台机器，分别开启三个 redis 服务（端口）
192.168.15.68		端口：7001,7002,7003
192.168.15.66		端口：7001,7002,7003
```

- 修改配置文件

192.168.15.68：

```bash
$ mkdir /usr/local/redis-5.0.8/cluster
$ cp /usr/local/redis-5.0.8/redis.conf /usr/local/redis-5.0.8/cluster/redis_7001.conf
$ cp /usr/local/redis-5.0.8/redis.conf /usr/local/redis-5.0.8/cluster/redis_7002.conf
$ cp /usr/local/redis-5.0.8/redis.conf /usr/local/redis-5.0.8/cluster/redis_7003.conf
```

```bash
$ vim /usr/local/redis-5.0.8/cluster/redis_7001.conf

bind 192.168.17.66 192.168.15.68
port 7001
daemonize yes
pidfile /var/run/redis_7001.pid
logfile "/var/log/redis/redis_7001.log"
dbfilename dump_7001.rdb
dir /var/lib/redis/
appendonly yes
appendfilename "appendonly_7001.aof"
cluster-enabled yes
cluster-config-file nodes_7001.conf
cluster-node-timeout 15000
masterauth 123456
requirepass 123456

......
```

其它 **5 **个配置与 `redis_7001.conf` 一致，此处省略。

- 开放端口，启动 **redis** 服务：

```bash
$ firewall-cmd --zone=public --add-port=7001/tcp --permanent
success
$ firewall-cmd --zone=public --add-port=17001/tcp --permanent
success
$ firewall-cmd --zone=public --add-port=7002/tcp --permanent
success
$ firewall-cmd --zone=public --add-port=17002/tcp --permanent
success
$ firewall-cmd --zone=public --add-port=7003/tcp --permanent
success
$ firewall-cmd --zone=public --add-port=17003/tcp --permanent
success
$ firewall-cmd --reload
success
$ firewall-cmd --list-ports
80/tcp 6379/tcp 7001/tcp 7002/tcp 7003/tcp 17001/tcp 17002/tcp 17003/tcp 7004/tcp 17004/tcp
$ redis-server /usr/local/redis-5.0.8/cluster/redis_7001.conf
$ redis-server /usr/local/redis-5.0.8/cluster/redis_7002.conf
$ redis-server /usr/local/redis-5.0.8/cluster/redis_7003.conf
```

另一台服务器进行同样的配置。**注意**，这里在启动一个端口时，同时启动了 `+10000` 端口（请注意，重新映射时，如果未设置总线端口，通常将使用 `10000` 的固定偏移量。）。

## 创建集群

所有环境准备完毕后，开始创建集群：

```bash
$ redis-cli -a 123456 --cluster create 192.168.15.68:7001 192.168.15.68:7002 192.168.15.68:7003 192.168.17.66:7001 192.168.17.66:7002 192.168.17.66:7003 --cluster-replicas 1	
# *注意*：`redis 5` 以后不再需要使用 `ruby` 组建进行创建集群了，直接使用 `redis-cli` 创建集群即可。
```

集群创建成功：

![redis集群创建.png](/images/redis集群创建.png)

从图中可以看到，集群生成了三个 **master** 和 对应的三个 **slave**，且一个 **salve** 对应一个 **master**。



集群自动生成了 `nodes.conf` 文件：

```bash
$ ls /var/lib/redis/
appendonly_7001.aof  appendonly_7004.aof  dump_7002.rdb  nodes_7001.conf  nodes_7004.conf
appendonly_7002.aof  appendonly.aof       dump_7003.rdb  nodes_7002.conf
appendonly_7003.aof  dump_7001.rdb        dump.rdb       nodes_7003.conf
```

## 集群操作

- 登陆集群

  ```bash
  $ redis-cli -c -h 192.168.17.66 -p 7001 -a 123456 # -c，使用集群方式登录
  Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
  192.168.17.66:7001> 
  ```

- 查看集群信息

  ```bash
  192.168.17.66:7001> CLUSTER INFO                   #集群状态
  cluster_state:ok
  cluster_slots_assigned:16384
  cluster_slots_ok:16384
  cluster_slots_pfail:0
  cluster_slots_fail:0
  cluster_known_nodes:7
  cluster_size:3
  cluster_current_epoch:10
  cluster_my_epoch:9
  cluster_stats_messages_ping_sent:3049
  cluster_stats_messages_pong_sent:3230
  cluster_stats_messages_update_sent:1
  cluster_stats_messages_sent:6280
  cluster_stats_messages_ping_received:3230
  cluster_stats_messages_pong_received:3045
  cluster_stats_messages_update_received:4
  cluster_stats_messages_received:6279
  ```

- 列出节点信息

  ```bash
  192.168.17.66:7001> CLUSTER NODES                  #列出节点信息
  c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 192.168.15.68:7003@17003 master - 0 1589519932000 9 connected 5461-10922
  d26dce64338aabb3bf83b5844e4f0a5868ab5764 192.168.17.66:7002@17002 slave cd61db40efd55708c1074dd4e751a2959d460f79 0 1589519935512 10 connected
  4349662174b6481bac831c4e062356b5433fa08f 192.168.15.68:7002@17002 master - 0 1589519932621 2 connected 10923-16383
  c74f7cece02aca84266278cdd589cc6d7103a4fd 192.168.17.66:7003@17003 slave 4349662174b6481bac831c4e062356b5433fa08f 0 1589519932000 2 connected
  cd61db40efd55708c1074dd4e751a2959d460f79 192.168.15.68:7001@17001 master - 0 1589519934628 10 connected 0-5460
  e7db7b26b32dfd746e67d84e165ecdfd76ea0d1c 192.168.15.68:7004@17004 master - 0 1589519933555 0 connected
  9fc6342460f3a4297260ad2a949056c899351720 192.168.17.66:7001@17001 myself,slave c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 0 1589519934000 4 connected
  ```

  可以看到三个 **master** 以及各自对应的 **slave**。

- 写入数据

  ```bash
  192.168.17.66:7001> set key111 111
  -> Redirected to slot [13680] located at 192.168.15.68:7002
  OK
  192.168.15.68:7002> set key222 222
  -> Redirected to slot [2320] located at 192.168.15.68:7001
  OK
  192.168.15.68:7001> set key333 333
  -> Redirected to slot [7472] located at 192.168.15.68:7003
  OK
  192.168.15.68:7003> set key444 444
  -> Redirected to slot [12752] located at 192.168.15.68:7002
  OK
  192.168.15.68:7002> set key555 555
  -> Redirected to slot [9712] located at 192.168.15.68:7003
  OK
  ```

  可以看出 **Redis Cluster** 集群是去中心化的，每个 **master** 节点都是平等的，连接哪个节点都可以获取和设置数据。

- 增加节点

  + 添加节点配置文件，方法同上面一样

  + 在集群中添加节点：

    ```bash
    192.168.15.68:7002> CLUSTER MEET 192.168.15.68 7005
    OK
    192.168.17.66:7001> CLUSTER NODES
    192.168.17.66:7001> CLUSTER NODES                  #列出节点信息
    c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 192.168.15.68:7003@17003 master - 0 1589519932000 9 connected 5461-10922
    d26dce64338aabb3bf83b5844e4f0a5868ab5764 192.168.17.66:7002@17002 slave cd61db40efd55708c1074dd4e751a2959d460f79 0 1589519935512 10 connected
    4349662174b6481bac831c4e062356b5433fa08f 192.168.15.68:7002@17002 master - 0 1589519932621 2 connected 10923-16383
    c74f7cece02aca84266278cdd589cc6d7103a4fd 192.168.17.66:7003@17003 slave 4349662174b6481bac831c4e062356b5433fa08f 0 1589519932000 2 connected
    cd61db40efd55708c1074dd4e751a2959d460f79 192.168.15.68:7001@17001 master - 0 1589519934628 10 connected 0-5460
    c7db7b234rf2dfd746e6784e165ecdfd76ea00db 192.168.15.68:7005@17005 master - 0 1589519933666 0 connected
    ed34fcece02aca842662hg65d589cc6d7103a4t5 192.168.17.66:7005@17005 slave c7db7b234rf2dfd746e6784e165ecdfd76ea00db 0 1589519933666 2 connected
    e7db7b26b32dfd746e67d84e165ecdfd76ea0d1c 192.168.15.68:7004@17004 master - 0 1589519933555 0 connected
    9fc6342460f3a4297260ad2a949056c899351720 192.168.17.66:7001@17001 myself,slave c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 0 1589519934000 4 connected
    ```

- 更换节点身份

  ```bash
  redis-cli -c -h 192.168.15.68 -p 7001 -a 123456 cluster replicate e7db7b26b32dfd746e67d84e165ecdfd76ea0d1c
  ```

- 删除节点

  ```bash
  192.168.15.68:7001> CLUSTER FORGET 9fc6342460f3a4297260ad2a949056c899351720
  (error) ERR I tried hard but I can't forget myself...     #无法删除登录节点
  
  192.168.15.68:7001> CLUSTER FORGET c1c22b5427ab32e0e4154a5b00c0d45a34fc9575
  (error) ERR Can't forget my master!                 	  #不能删除自己的master节点
  
  192.168.15.68:7001> CLUSTER FORGET c7db7b234rf2dfd746e6784e165ecdfd76ea00db
  OK              										  #可以删除其它的master节点
  ```

- 保存配置

  ```bash
  192.168.15.68:7001> CLUSTER SAVECONFIG                 #将节点配置信息保存到硬盘
  OK
  ```

- 模拟 **master** 节点挂掉：

  192.168.15.68

  ```bash
  $ ps -ef | grep redis-server
  root      5528     1  0 5月14 ?       00:00:56 redis-server 192.168.15.68:7003 [cluster]
  root      6618     1  0 5月14 ?       00:00:54 redis-server 192.168.15.68:7004 [cluster]
  root      8586     1  0 5月14 ?       00:00:49 redis-server 127.0.0.1:6379
  root      8654     1  0 5月14 ?       00:00:52 redis-server 192.168.15.68:7001 [cluster]
  root      8696     1  0 5月14 ?       00:00:53 redis-server 192.168.15.68:7002 [cluster]
  
  $ kill -9 8654
  ```

  ```bash
  192.168.15.68:7003> cluster nodes		# Kill 前
  4349662174b6481bac831c4e062356b5433fa08f 192.168.15.68:7002@17002 master - 0 1589521299646 2 connected 10923-16383
  c74f7cece02aca84266278cdd589cc6d7103a4fd 192.168.17.66:7003@17003 slave 4349662174b6481bac831c4e062356b5433fa08f 0 1589521298640 6 connected
  e7db7b26b32dfd746e67d84e165ecdfd76ea0d1c 192.168.15.68:7004@17004 master - 0 1589521297628 0 connected
  d26dce64338aabb3bf83b5844e4f0a5868ab5764 192.168.17.66:7002@17002 slave cd61db40efd55708c1074dd4e751a2959d460f79 0 1589521294000 10 connected
  cd61db40efd55708c1074dd4e751a2959d460f79 192.168.15.68:7001@17001 master - 1589521298236 1589521293594 10 disconnected 0-5460
  9fc6342460f3a4297260ad2a949056c899351720 192.168.17.66:7001@17001 slave c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 0 1589521300688 9 connected
  c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 192.168.15.68:7003@17003 myself,master - 0 1589521298000 9 connected 5461-10922
  
  192.168.15.68:7003> cluster nodes		# Kill 后
  4349662174b6481bac831c4e062356b5433fa08f 192.168.15.68:7002@17002 master - 0 1589521325829 2 connected 10923-16383
  c74f7cece02aca84266278cdd589cc6d7103a4fd 192.168.17.66:7003@17003 slave 4349662174b6481bac831c4e062356b5433fa08f 0 1589521326841 6 connected
  e7db7b26b32dfd746e67d84e165ecdfd76ea0d1c 192.168.15.68:7004@17004 master - 0 1589521326000 0 connected
  d26dce64338aabb3bf83b5844e4f0a5868ab5764 192.168.17.66:7002@17002 master - 0 1589521325000 11 connected 0-5460
  cd61db40efd55708c1074dd4e751a2959d460f79 192.168.15.68:7001@17001 master,fail - 1589521298236 1589521293594 10 disconnected
  9fc6342460f3a4297260ad2a949056c899351720 192.168.17.66:7001@17001 slave c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 0 1589521325000 9 connected
  c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 192.168.15.68:7003@17003 myself,master - 0 1589521324000 9 connected 5461-10922
  ```

  对应 `7001` 的一行可以看到，**master fail**，状态为 **disconnected**；而对应 `7002` 的一行，**slave** 已经变成 **master**。

- 重启 **7001** 节点：

  ```bash
  192.168.15.68:7003> cluster nodes
  4349662174b6481bac831c4e062356b5433fa08f 192.168.15.68:7002@17002 master - 0 1589521575000 2 connected 10923-16383
  c74f7cece02aca84266278cdd589cc6d7103a4fd 192.168.17.66:7003@17003 slave 4349662174b6481bac831c4e062356b5433fa08f 0 1589521579571 6 connected
  e7db7b26b32dfd746e67d84e165ecdfd76ea0d1c 192.168.15.68:7004@17004 master - 0 1589521578441 0 connected
  d26dce64338aabb3bf83b5844e4f0a5868ab5764 192.168.17.66:7002@17002 master - 0 1589521577447 11 connected 0-5460
  cd61db40efd55708c1074dd4e751a2959d460f79 192.168.15.68:7001@17001 slave d26dce64338aabb3bf83b5844e4f0a5868ab5764 0 1589521575513 11 connected
  9fc6342460f3a4297260ad2a949056c899351720 192.168.17.66:7001@17001 slave c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 0 1589521579000 9 connected
  c1c22b5427ab32e0e4154a5b00c0d45a34fc9575 192.168.15.68:7003@17003 myself,master - 0 1589521577000 9 connected 5461-10922
  ```

  这里看到 `7001` 已经变为 **salve**节点了，并且是 `7003` 的节点。即 **master** 节点如果挂掉，它的 **slave** 节点变为新 **master** 节点继续对外提供服务，而原来的 **master** 节点如果重启，则变为新 **master** 节点的 **slave** 节点。



更多参考：

[Redis集群常用命令](https://www.cnblogs.com/gossip/p/5993922.html)



本文参考：[Redis集群详解](https://blog.csdn.net/miss1181248983/article/details/90056960#commentBox)