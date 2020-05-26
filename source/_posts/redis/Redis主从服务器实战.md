---
title: Redis主从服务器实战
date: 2020-05-13 13:48:51
categories: [Redis]
tags: [Redis]
toc: true
---

# 主从复制

## 原理

- 从服务器连接主服务器，发送 `SYNC` 命令；
- 主服务器接收到 `SYNC` 命名后，开始执行 `BGSAVE` 命令生成 `RDB` 文件并使用缓冲区记录此后执行的所有写命令；
- 主服务器 `BGSAVE` 执行完后，向所有从服务器发送快照文件，并在发送期间继续记录被执行的写命令；
- 从服务器收到快照文件后丢弃所有旧数据，载入收到的快照；
- 主服务器快照发送完毕后开始向从服务器发送缓冲区中的写命令；
- 从服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；（**从服务器初始化完成**）
- 主服务器每执行一个写命令就会向从服务器发送相同的写命令，从服务器接收并执行收到的写命令（**从服务器初始化完成后的操作**）

## 优点

- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离
- 为了分载 **Master** 的读操作压力，**Slave** 服务器可以为客户端提供只读操作的服务，写服务仍然必须由 **Master** 来完成
- **Slave** 同样可以接受其它 **Slaves** 的连接和同步请求，这样可以有效的分载 **Master** 的同步压力。
- **Master Server** 是以非阻塞的方式为 **Slaves** 提供服务。所以在 **Master-Slave** 同步期间，客户端仍然可以提交查询或修改请求。
- **Slave Server** 同样是以非阻塞的方式完成数据同步。在同步期间，如果有客户端提交查询请求，**Redis** 则返回同步之前的数据

## 缺点

- **Redis** 不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 **IP** 才能恢复。
- 主机宕机，宕机前有部分数据未能及时同步到从机，切换 **IP** 后还会引入数据不一致的问题，降低了系统的可用性。
- **Redis** 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。

# 主从复制实战

## 主服务器配置

master IP：192.168.15.68

```bash
# 指定 redis 只接收来自于这些地址的请求，如果不进行设置，那么将处理所有请求
# bind 127.0.0.1 192.168.17.66 192.168.15.76
bind 0.0.0.0		# 指定IP报错，这里直接改为 0.0.0.0

# 默认端口 is 6379
port 6379

# 保护模式是一层安全保护，以避免任何 Internet 上打开的 Redis 实例访问和利用。开启该参数后，redis只会本地进行访问，拒绝外部访问（这里因为允许外部IP访问，所以因该设置为 no）。
protected-mode no

# requirepass 配置可以让用户使用 AUTH 命令来认证密码，才能使用其他命令。
requirepass redis:123:456

# 客户端闲置 N 秒后服务端会断开连接 (0 不关闭)
timeout 0

# 默认情况下，Redis 不会作为守护进程（守护进程）运行。 如果需要，请使用“是”。请注意，Redis 守护进程将在 /var/run/redis.pid 中写入一个 pid 文件。
daemonize yes

# 当服务器在非守护进程下运行时，如果没有 pid 文件，则不会创建在配置中指定。守护服务器时，即使未指定，也会使用pid文件，默认为“ /var/run/redis.pid”。
pidfile /var/run/redis_6379.pid

#服务端日志的级别。
loglevel notice

# 指定了记录日志的文件。空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null
logfile "/var/log/redis_6379.log"

# 数据库的数量，默认使用的数据库是0。
databases 16

# RDB 核心规则配置 save <指定时间间隔> <执行指定次数更新操作>，满足条件就将内存中的数据同步到硬盘中。
save ""		# 禁用 rdb，由从服务器生成 rdb 文件
#save 900 1
#save 300 10
#save 60 10000

# 指定本地数据库文件名
dbfilename dump_6379.rdb


```

## slave服务器1配置

slave1 IP：192.168.17.66

```bash
# 指定 redis 只接收来自于这些地址的请求，如果不进行设置，那么将处理所有请求
bind 127.0.0.1

# 默认端口 is 6379
port 6379

# 保护模式是一层安全保护，以避免任何 Internet 上打开的 Redis 实例访问和利用。开启该参数后，redis只会本地进行访问，拒绝外部访问（这里因为允许外部IP访问，所以因该设置为 no）。
protected-mode yes

# 客户端闲置 N 秒后服务端会断开连接 (0 不关闭)
timeout 0

# 默认情况下，Redis 不会作为守护进程（守护进程）运行。 如果需要，请使用“是”。请注意，Redis 守护进程将在 /var/run/redis.pid 中写入一个 pid 文件。
daemonize yes

# 当服务器在非守护进程下运行时，如果没有 pid 文件，则不会创建在配置中指定。守护服务器时，即使未指定，也会使用pid文件，默认为“ /var/run/redis.pid”。
pidfile /var/run/redis_6379.pid

#服务端日志的级别。
loglevel notice

# 指定了记录日志的文件。空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null
logfile "/var/log/redis_6379.log"

# 数据库的数量，默认使用的数据库是0。
databases 16

# RDB 核心规则配置 save <指定时间间隔> <执行指定次数更新操作>，满足条件就将内存中的数据同步到硬盘中。
# save ""
save 900 1		# 主服务器减少I/O，禁用 save rdb文件，由 slave1 来 save！
save 300 10
save 60 10000

# 指定本地数据库文件名
dbfilename dump_6379.rdb

# 复制选项，slave复制对应的master。
replicaof 192.168.15.68 6379

# 如果主服务器受密码保护，那么slave要连上master，需要有master的密码才行。
masterauth redis:123:456

# 当从库同主机失去连接或者复制正在进行，从服务器稳定输出，不报错
replica-serve-stale-data yes

# slave 服务器只读
replica-read-only yes
```



## slave服务器2配置

slave2 IP：192.168.15.76

```bash
# 指定 redis 只接收来自于这些地址的请求，如果不进行设置，那么将处理所有请求
bind 127.0.0.1

# 默认端口 is 6379
port 6379

# 保护模式是一层安全保护，以避免任何 Internet 上打开的 Redis 实例访问和利用。开启该参数后，redis只会本地进行访问，拒绝外部访问（这里因为允许外部IP访问，所以因该设置为 no）。
protected-mode yes

# 客户端闲置 N 秒后服务端会断开连接 (0 不关闭)
timeout 0

# 默认情况下，Redis 不会作为守护进程（守护进程）运行。 如果需要，请使用“是”。请注意，Redis 守护进程将在 /var/run/redis.pid 中写入一个 pid 文件。
daemonize yes

# 当服务器在非守护进程下运行时，如果没有 pid 文件，则不会创建在配置中指定。守护服务器时，即使未指定，也会使用pid文件，默认为“ /var/run/redis.pid”。
pidfile /var/run/redis_6379.pid

#服务端日志的级别。
loglevel notice

# 指定了记录日志的文件。空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null
logfile "/var/log/redis_6379.log"

# 数据库的数量，默认使用的数据库是0。
databases 16

# RDB 核心规则配置 save <指定时间间隔> <执行指定次数更新操作>，满足条件就将内存中的数据同步到硬盘中。
save ""		# 禁用 rdb，由slave1服务器生成 rdb 文件即可
#save 900 1
#save 300 10
#save 60 10000

# 指定本地数据库文件名
dbfilename dump_6379.rdb

# 复制选项，slave复制对应的master。
replicaof 192.168.15.68 6379

# 如果主服务器受密码保护，那么slave要连上master，需要有master的密码才行。
masterauth redis:123:456

# 当从库同主机失去连接或者复制正在进行，从服务器稳定输出，不报错
replica-serve-stale-data yes

# slave 服务器只读
replica-read-only yes
```

## 实际操作遇到问题

按照上面配置好主从服务器并重启 redis 服务后，并没有实现同步数据，查看 slave 服务器 redis 日志报错：

1）错误提示1：

```bash
$ cat /var/log/redis/redis.log
14015:S 13 May 2020 17:08:24.047 * Connecting to MASTER 192.168.15.68:6379
14015:S 13 May 2020 17:08:24.048 * MASTER <-> REPLICA sync started
14015:S 13 May 2020 17:08:24.049 # Error condition on socket for SYNC: No route to host
```

查阅资料发现是 **master** 服务器没有开放端口访问。

```bash
$firewall-cmd --list-ports
80/tcp
```

开放 **master** `redis` 端口：

```bash
$ firewall-cmd --zone=public --add-port=6379/tcp --permanent
success
$ firewall-cmd --reload
success
$ firewall-cmd --list-ports
80/tcp 6379/tcp
```

2）错误提示2：

```bash
14015:S 13 May 2020 17:08:24.047 * Connecting to MASTER 192.168.15.68:6379
14015:S 13 May 2020 17:08:24.048 * MASTER <-> REPLICA sync started
14015:S 13 May 2020 17:08:24.049 # Error condition on socket for SYNC: Connection refused
```

查阅资料将 **master** 服务器 `bind` 端口改为开放所有：

```bash
# 指定 redis 只接收来自于这些地址的请求，如果不进行设置，那么将处理所有请求
bind 0.0.0.0
```

重启 **master** `redis` 服务，`slave` 服务器已经看到了成功信息：

```bash
14015:S 13 May 2020 17:08:24.049 # Error condition on socket for SYNC: Connection refused
14015:S 13 May 2020 17:08:25.050 * Connecting to MASTER 192.168.15.68:6379
14015:S 13 May 2020 17:08:25.050 * MASTER <-> REPLICA sync started
14015:S 13 May 2020 17:08:25.140 * Non blocking connect for SYNC fired the event.
14015:S 13 May 2020 17:08:25.141 * Master replied to PING, replication can continue...
14015:S 13 May 2020 17:08:25.144 * Partial resynchronization not possible (no cached master)
14015:S 13 May 2020 17:08:25.146 * Full resync from master: c658aa4725c9200d910e7704e58a44bf36a57e28:0
14015:S 13 May 2020 17:08:25.233 * MASTER <-> REPLICA sync: receiving 224 bytes from master
14015:S 13 May 2020 17:08:25.234 * MASTER <-> REPLICA sync: Flushing old data
14015:S 13 May 2020 17:08:25.234 * MASTER <-> REPLICA sync: Loading DB in memory
14015:S 13 May 2020 17:08:25.234 * MASTER <-> REPLICA sync: Finished with success
```

`slave` 验证数据是否同步：

```bash
$ redis-cli 
127.0.0.1:6379> keys *
1) "to-slave"
2) "hello"
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> get to-slave
"hello this is master"
127.0.0.1:6379> 
```

至此，`redis` 主从同步实践成功！