---
title: Redis常用配置信息
date: 2020-05-13 14:37:57
categories: [Redis]
tags: [Redis]
toc: true
---

# Redis 常用配置信息

```bash
# ./redis-server /path/to/redis.conf	#启动redis服务，

# 指定 redis 只接收来自于这些地址的请求，如果不进行设置，那么将处理所有请求

# 允许多个IP只需要在后面空格追加IP即可，例如：
# bind 127.0.0.1 192.168.15.75
#也可以使用 0.0.0.0 来接受所有 IP 的访问
bind 127.0.0.1 

# 保护模式是一层安全保护，以避免任何 Internet 上打开的 Redis 实例访问和利用。开启该参数后，redis只会本地进行访问，拒绝外部访问。
# 如下情况需要启用保护模式时：
# 1) 服务器未使用 bind 命令明确绑定到一组地址。
# 2) 没有配置密码。
#
protected-mode yes

# 默认端口 is 6379
port 6379

# requirepass 配置可以让用户使用 AUTH 命令来认证密码，才能使用其他命令。
# requirepass foobared

# 此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的 /proc/sys/net/core/somaxconn 值，默认是 511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。
# 对于负载很大的服务程序来说大大的不够。一般会将 somaxconn值 修改为2048或者更大。
tcp-backlog 511

# 指定用于监听传入连接的 Unix 套接字的路径。没有默认值，因此在未指定 Redis 的情况下，Redis 不会在 Unix 套接字上侦听。
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# 客户端闲置 N 秒后服务端会断开连接 (0 不关闭)
timeout 0

# 定时向 client 发送 tcp_ack 包来探测 client 是否存活的，单位（秒）。
tcp-keepalive 300

# 默认情况下，Redis 不会作为守护进程（守护进程）运行。 如果需要，请使用“是”。请注意，Redis 守护进程将在 /var/run/redis.pid 中写入一个 pid 文件。
daemonize no

# 当服务器在非守护进程下运行时，如果没有 pid 文件，则不会创建在配置中指定。守护服务器时，即使未指定，也会使用pid文件，默认为“ /var/run/redis.pid”。
pidfile /var/run/redis_6379.pid

#指定了服务端日志的级别。级别包括：debug（很多信息，方便开发、测试），verbose（许多有用的信息，但是没有debug级别信息多），notice（适当的日志级别，适合生产环境），warn（只有非常重要的信息）
loglevel notice

# 指定了记录日志的文件。空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null
logfile "/var/log/redis_6379.log"

# 数据库的数量，默认使用的数据库是0。可以通过”SELECT 【数据库序号】“命令选择一个数据库，序号从0开始
databases 16

################################ SNAPSHOTTING 磁盘快照 ################################
# RDB 核心规则配置 save <指定时间间隔> <执行指定次数更新操作>，满足条件就将内存中的数据同步到硬盘中。
# 官方出厂配置默认是:
# 1) 900 秒内有   1   个更改
# 2) 300 秒内有   10  个更改
# 3)  60 秒内有 10000 个更改
# 则将内存中的数据快照写入磁盘。若不想用RDB方案，可以把 save "" 的注释打开，下面三个注释
#   save ""
save 900 1
save 300 10
save 60 10000

# 但是，如果您设置了对 Redis 服务器和持久性的适当监视，则可能希望禁用此功能，以便即使磁盘，权限等出现问题，Redis 仍将继续照常工作（如果设置了监听，可以考虑关掉此设置）。
stop-writes-on-bgsave-error yes

# 配置存储至本地数据库时是否压缩数据，默认为 yes。Redis 采用 LZF 压缩方式，但占用了一点 CPU 的时间。若关闭该选项，但会导致数据库文件变的巨大。建议开启。
rdbcompression yes

# 从RDB版本5开始，CRC64 校验和位于文件末尾。这使得该格式更能抵抗损坏，但是在保存和加载 RDB文件时会
# 付出一定的性能损失（大约10％），因此可以禁用该格式以实现最佳性能。
# 在禁用校验和的情况下创建的RDB文件的校验和为零，这将指示加载代码跳过该校验。
rdbchecksum yes

# 数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录
# dir ./
 dir /home/www/redis/data/
 
# 指定本地数据库文件名
dbfilename dump.rdb
```

