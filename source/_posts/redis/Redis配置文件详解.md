---
title: Redis配置文件详解
date: 2020-05-11 15:01:52
categories: [Redis]
tags: [Redis]
---

# 版本

```bash
$ redis-server -v
Redis server v=5.0.8 sha=00000000:0 malloc=jemalloc-5.1.0 bits=64 build=d21bcca96e4afc9c
```

# 配置文件

```bash
# Redis配置文件示例。
#
# 加载配置文件：
#
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:3333
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.

################################## 包含其他配置文件 ###################################

# 如果你想在所有 Redis 服务器上，在公共配置文件之后加载其他的少量配置，可以 使用 include 来加载特殊的配置。
#
# 注意，管员和 Redis 前哨不能通过命令 “CONFIG REWRITE” 重写 include 加载的配置文件。
# 因为 Redsi 始终使用最后一行作为配置的执行，所以为了不影响原有的配置信息，最好件开头使用 
# include 加载配置文件。
#
# 相反如果你想用 include 加载的配置文件覆盖配置，因该在最后一行使用 include。
#
# include /path/to/local.conf
# include /path/to/other.conf

################################## 模块 #####################################

# Load modules at startup. If the server is not able to load modules
# it will abort. It is possible to use multiple loadmodule directives.
#
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so

################################## 网络 #####################################

# 默认情况下，如果未指定“ bind”配置指令，则Redis侦听来自服务器上所有可用网络接口的连接。
# 使用以下功能可以只收听一个或多个所选接口
# “ bind”配置指令，后跟一个或多个IP地址。
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ 如果运行Redis的计算机直接暴露于 Internet，则绑定到所有接口都是很危险的，
# 并且会将实例暴露给 Internet上 的所有人。因此，默认情况下，我们取消注释以下 bind 指令，它将强制 Redis 仅侦听 IPv4 环回接口地址（这意味着 Redis 将只能接受本机访问）。
#
# 如果你确定你想将实例取监听所有的地址，只要放在下面一行即可。
bind 127.0.0.1

# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited. 
# 保护模式是一层安全保护，以避免Internet上打开的Redis实例访问和利用。
#
# 如下情况需要启用保护模式时：
#
# 1) 服务器未使用 bind 命令明确绑定到一组地址。
# 2) 没有配置密码。
#
# 服务器接受以下集中方式连接：
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
protected-mode yes

# Accept connections on the specified port, 默认端口 is 6379 (IANA #815344).
# If port 0 is specified 则Redis将不会在TCP套接字上侦听.
port 6379

# TCP listen() backlog.
#
# In high requests-per-second environments you need an high backlog in order
# to avoid slow clients connections issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect. 
# 此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。
# 对于负载很大的服务程序来说大大的不够。一般会将 somaxconn值 修改为2048或者更大。
tcp-backlog 511

# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
# 指定用于监听传入连接的 Unix 套接字的路径。没有默认值，因此在未指定 Redis 的情况下，Redis 不会在 Unix 套接字上侦听。
#
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# 客户端闲置N秒后关闭连接 (0 不关闭)
timeout 0

# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Take the connection alive from the point of view of network
#    equipment in the middle.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
# 定时向 client 发送 tcp_ack 包来探测 client 是否存活的，单位（秒）。
tcp-keepalive 300

################################# GENERAL #####################################

# By default Redis does not run as a daemon（守护进程）. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
# 默认情况下，Redis不会作为守护进程（守护进程）运行。 如果需要，请使用“是”。请注意，Redis 守护进程将在 /var/run/redis.pid 中写入一个 pid 文件。
daemonize no

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:	可以通过upstart和systemd管理Redis守护进程
#   supervised no      - no supervision interaction
#		没有监督互动
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#		通过将Redis置于SIGSTOP模式来启动信号
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#		signal systemd将READY = 1写入$ NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
#		检测upstart或systemd方法基于 UPSTART_JOB或NOTIFY_SOCKET环境变量
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised no

# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit. 
# 如果指定了 pid 文件，则 Redis 会在启动时将其写入指定位置，然后在退出时将其删除。
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
# 当服务器在非守护进程下运行时，如果没有pid文件，则不会创建在配置中指定。守护服务器时，即使未指定，也会使用pid文件，默认为“ /var/run/redis.pid”。
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid

# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice

# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""

# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# 是否打开记录syslog功能
# syslog-enabled no

# Specify the syslog identity. 	syslog的标识符
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.	日志的来源、设备
# syslog-facility local0

# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
# 默认 16 个库
databases 16

# By default Redis shows an ASCII art logo only when started to log to the
# standard output and if the standard output is a TTY. Basically this means
# that normally a logo is displayed only in interactive sessions.
#
# However it is possible to force the pre-4.0 behavior and always show a
# ASCII art logo in startup logs by setting the following option to yes.
always-show-logo yes

################################ SNAPSHOTTING 磁盘快照 ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
# 如果同时发生了给定的秒数和给定的针对数据库的写操作数，则将保存数据库。
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""
# 磁盘快照根据 redis key修改频率不同，设置不同的备份规则
save 900 1
save 300 10
save 60 10000

# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
# 默认情况下，如果启用RDB快照（至少一个保存点）。并且最新的后台保存失败，则Redis将停止接受写入。这将使用户（以一种困难的方式）意识到数据无法正确地持久存储在磁盘上，否则，可能没人会注意到并且会发生一些灾难。
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
# 如果后台保存过程再次开始工作，则 Redis 将自动允许再次写入。
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
# 但是，如果您设置了对 Redis 服务器和持久性的适当监视，则可能希望禁用此功能，以便即使磁盘，权限等出现问题，Redis 仍将继续照常工作（如果设置了监听，可以考虑关掉此设置）。
stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes

# 从RDB版本5开始，CRC64校验和位于文件末尾。这使得该格式更能抵抗损坏，但是在保存和加载 RDB文件时会
# 付出一定的性能损失（大约10％），因此可以禁用该格式以实现最佳性能。
# 在禁用校验和的情况下创建的RDB文件的校验和为零，这将指示加载代码跳过该校验。
rdbchecksum yes

# The filename where to dump the DB
dbfilename dump.rdb

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
# 数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录
# dir ./
 dir /home/www/redis/data/

################################# REPLICATION 主从复制#################################

# Master-Replica replication. Use replicaof to make a Redis instance a copy of
# another Redis server. A few things to understand ASAP about Redis replication.
#
#   +------------------+      +---------------+
#   |      Master      | ---> |    Replica    |
#   | (receive writes) |      |  (exact copy) |
#   +------------------+      +---------------+
#
# 1) Redis replication is asynchronous, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least
#    a given number of replicas.
# Redis复制是异步的，但如果没有链接至少一个拷贝副本，可以设置主服务器停止接受写入。
# 2) Redis replicas are able to perform a partial resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next
#    sections of this file) with a sensible value depending on your needs.
# 短时间内，如果 redis 副本丢失相对较少的复制链接，redis副本可以实现与主服务器重新同步丢是的数据。
# 这里可能需要根据需要将复制 backlog 大小配置为合理的值。
# 3) Replication is automatic and does not need user intervention. After a
#    network partition replicas automatically try to reconnect to masters
#    and resynchronize with them.
# 复制是自动的，不需要用户干预。网络分区副本会自动尝试重新连接到主服务器并与他们重新同步。
#
# 命令
# replicaof <masterip> <masterport>

# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the replica to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the replica request.
# 如果主服务器受密码保护（使用下面的“ requirepass”配置指令），则可以在开始复制同步过程之前
# 告诉副本服务器进行身份验证，否则主服务器将拒绝副本请求。
#
# masterauth <master-password>

# When a replica loses its connection with the master, or when the replication
# is still in progress, the replica can act in two different ways:
# 当副本失去与主数据库的连接时，或者当复制仍在进行中，副本可以以两种不同的方式起作用：
#
# 1) if replica-serve-stale-data is set to 'yes' (the default) the replica will
#    still reply to client requests, possibly with out of date data（过期数据）, or the
#    data set may just be empty if this is the first synchronization.
# 	 从库会继续响应客户端的请求
#
# 2) if replica-serve-stale-data is set to 'no' the replica will reply with
#    an error "SYNC with master in progress" to all the kind of commands
#    but to INFO, replicaOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG,
#    SUBSCRIBE, UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB,
#    COMMAND, POST, HOST: and LATENCY.
#	 上述命令之外的任何请求都会返回一个错误”SYNC with master in progress”
#
# 从服务器稳定输出，不报错（都是连接或数据为空）
replica-serve-stale-data yes

# You can configure a replica instance to accept writes or not. Writing against
# a replica instance may be useful to store some ephemeral data (because data
# written on a replica will be easily deleted after resync with the master) but
# may also cause problems if clients are writing to it because of a
# misconfiguration.
#
# Since Redis 2.6 by default replicas are read-only.
#
# Note: read only replicas are not designed to be exposed to untrusted clients
# on the internet. It's just a protection layer against misuse of the instance.
# Still a read only replica exports by default all the administrative commands
# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
# security of read only replicas using 'rename-command' to shadow all the
# administrative / dangerous commands.
# 从服务器只读
replica-read-only yes

# Replication SYNC strategy: disk or socket.
# 复制SYNC策略：磁盘或套接字。
#
# -------------------------------------------------------
# WARNING: DISKLESS REPLICATION IS EXPERIMENTAL CURRENTLY
# -------------------------------------------------------
#
# New replicas and reconnecting replicas that are not able to continue the replication
# process just receiving differences, need to do what is called a "full
# synchronization". An RDB file is transmitted from the master to the replicas.
# The transmission can happen in two different ways:
# 仅接收差异而无法继续复制过程的新副本和重新连接的副本需要执行所谓的“完全同步”。 
# RDB文件从主数据库传输到副本数据库。 传输可以通过两种不同的方式进行：
#
# 1) Disk-backed: The Redis master creates a new process that writes the RDB
#                 file on disk. Later the file is transferred by the parent
#                 process to the replicas incrementally.
# Redis主服务器创建一个新过程，将RDB文件写入磁盘。 后来，文件由父进程逐步传输到副本。
# 2) Diskless: The Redis master creates a new process that directly writes the
#              RDB file to replica sockets, without touching the disk at all.
# Redis主服务器创建一个新进程，该进程将RDB文件直接写入副本套接字，而完全不接触磁盘。
#
# With disk-backed replication, while the RDB file is generated, more replicas
# can be queued and served with the RDB file as soon as the current child producing
# the RDB file finishes its work. With diskless replication instead once
# the transfer starts, new replicas arriving will be queued and a new transfer
# will start when the current one terminates.
# 使用磁盘支持的复制，在生成RDB文件的同时，只要生成RDB文件的当前子级完成工作，
# 以将更多副本排入队列并与RDB文件一起使用。 如果使用无盘复制，则一旦传输开始，
# 副本将排队，并且当当前副本终止时将开始新的传输。
#
# When diskless replication is used, the master waits a configurable amount of
# time (in seconds) before starting the transfer in the hope that multiple replicas
# will arrive and the transfer can be parallelized.
# 使用无盘复制时，主服务器在开始传输之前会等待一段可配置的时间（以秒为单位），以希望多个副本可以到达并且传输可以并行化。
#
# With slow disks and fast (large bandwidth) networks, diskless replication
# works better. 使用慢速磁盘和快速（大带宽）网络，无盘复制效果更好
repl-diskless-sync no

# When diskless replication is enabled, it is possible to configure the delay
# the server waits in order to spawn the child that transfers the RDB via socket
# to the replicas.
# 启用无盘复制后，可以配置服务器等待的延迟，以便生成通过套接字将RDB传输到副本的子代。
#
# This is important since once the transfer starts, it is not possible to serve
# new replicas arriving, that will be queued for the next RDB transfer, so the server
# waits a delay in order to let more replicas arrive.
# 这一点很重要，因为一旦传输开始，就无法为到达下一个RDB传输的新副本提供服务，因此服务器会等待一段时间，以便让更多副本到达。
#
# The delay is specified in seconds, and by default is 5 seconds. To disable
# it entirely just set it to 0 seconds and the transfer will start ASAP（尽快）
repl-diskless-sync-delay 5

# Replicas send PINGs to server in a predefined interval. It's possible to change
# this interval with the repl_ping_replica_period option. The default value is 10
# seconds.
# 从服务器以预定义的时间间隔将 PING 发送到主服务器。 可以使用 repl_ping_replica_period 选项更改此间隔。 默认值为10秒。 
# repl-ping-replica-period 10

# The following option sets the replication timeout for:
#
# 1) Bulk transfer I/O during SYNC, from the point of view of replica.
# 2) Master timeout from the point of view of replicas (data, pings).
# 3) Replica timeout from t
#e point of view of masters (REPLCONF ACK pings).
#
# It is important to make sure that this value is greater than the value
# specified for repl-ping-replica-period otherwise a timeout will be detected
# every time there is low traffic between the master and the replica.
# 值要大于 repl-ping-replica-period 的值，不然每次都会检测为超时。
# repl-timeout 60

# Disable TCP_NODELAY on the replica socket after SYNC?
# 在同步后禁用副本套接字上的TCP_NODELAY？
#
# If you select "yes" Redis will use a smaller number of TCP packets and
# less bandwidth to send data to replicas. But this can add a delay for
# the data to appear on the replica side, up to 40 milliseconds with
# Linux kernels using a default configuration.（减少带宽，增加从服务器延迟）
#
# If you select "no" the delay for data to appear on the replica side will
# be reduced but more bandwidth will be used for replication.（增加带宽，减少从服务器延迟）
#
# By default we optimize for low latency, but in very high traffic conditions
# or when the master and replicas are many hops away, turning this to "yes" may
# be a good idea.（在流量非常高的情况下，或者在距离主服务器和副本很多跳的情况下，将其设置为“是”可能是个好主意。）
repl-disable-tcp-nodelay no

# Set the replication backlog size. The backlog is a buffer that accumulates
# replica data when replicas are disconnected for some time, so that when a replica
# wants to reconnect again, often a full resync is not needed, but a partial
# resync is enough, just passing the portion of data the replica missed while
# disconnected.（设置从服务器 backlog 大小，以便于从服务器断开连接一段时间，它可以缓冲部分数据。因此，当从服务器重新连接时，不需要完全同步，只同步缓冲区中的数据即可。）
#
# The bigger the replication backlog, the longer the time the replica can be
# disconnected and later be able to perform a partial resynchronization.
# repl-backlog-size 值越大，从服务器可以断开连接并在下次重新连接同步时间越长。
#
# The backlog is only allocated once there is at least a replica connected.
# 仅当至少有一个副本连接时，才分配 backlog。
# repl-backlog-size 1mb

# After a master has no longer connected replicas for some time, the backlog
# will be freed. The following option configures the amount of seconds that
# need to elapse, starting from the time the last replica disconnected, for
# the backlog buffer to be freed.
#
# Note that replicas never free the backlog for timeout, since they may be
# promoted to masters later, and should be able to correctly "partially
# resynchronize" with the replicas: hence they should always accumulate backlog.
#
# A value of 0 means to never release the backlog.
#	主服务器断开与从服务器连接到释放 backlog 缓冲区时间。0 意味着不释放缓冲区。（从服务器不会释放缓冲区）
# repl-backlog-ttl 3600

# The replica priority is an integer number published by Redis in the INFO output.
# It is used by Redis Sentinel in order to select a replica to promote into a
# master if the master is no longer working correctly.
# 副本优先级是 Redis 在 INFO 输出中发布的整数。如果主服务器不再正常工作，Redis Sentinel 会使用它来选择要升级为主服务器的副本。
#
# A replica with a low priority number is considered better for promotion, so
# for instance if there are three replicas with priority 10, 100, 25 Sentinel will
# pick the one with priority 10, that is the lowest.
#
# However a special priority of 0 marks the replica as not able to perform the
# role of master, so a replica with priority of 0 will never be selected by
# Redis Sentinel for promotion.（优先级不能设置为 0,设置为 0 不是被升级为主服务器）
#
# By default the priority is 100.
replica-priority 100

# It is possible for a master to stop accepting writes if there are less than
# N replicas connected, having a lag less or equal than M seconds.
# 如果连接的副本少于N个，并且延迟小于或等于M秒，则主服务器可能会停止接受写入。
#
# The N replicas need to be in "online" state.
#
# The lag in seconds, that must be <= the specified value, is calculated from
# the last ping received from the replica, that is usually sent every second.
# 延迟（以秒为单位）必须小于等于指定值，该延迟是根据从副本接收到的最后ping（通常每秒发送一次）计算得出的。
#
# This option does not GUARANTEE that N replicas will accept the write, but
# will limit the window of exposure for lost writes in case not enough replicas
# are available, to the specified number of seconds.
#
# For example to require at least 3 replicas with a lag <= 10 seconds use:
#
# min-replicas-to-write 3
# min-replicas-max-lag 10
#
# Setting one or the other to 0 disables the feature.
#
# By default min-replicas-to-write is set to 0 (feature disabled) and
# min-replicas-max-lag is set to 10.

# A Redis master is able to list the address and port of the attached
# replicas in different ways. For example the "INFO replication" section
# offers this information, which is used, among other tools, by
# Redis Sentinel in order to discover replica instances.
# Another place where this info is available is in the output of the
# "ROLE" command of a master.
# Redis主服务器能够以不同方式列出附加副本的地址和端口。 例如，“ INFO复制”部分提供了此信息，
# Redis Sentinel使用此信息以及其他工具来发现副本实例。 该信息可用的另一个位置是主服务器的“ ROLE”命令的输出。
#
# The listed IP and address normally reported by a replica is obtained
# in the following way:
#
#   IP: The address is auto detected by checking the peer address
#   of the socket used by the replica to connect with the master.
#
#   Port: The port is communicated by the replica during the replication
#   handshake, and is normally the port that the replica is using to
#   listen for connections.
#
# However when port forwarding or Network Address Translation (NAT) is
# used, the replica may be actually reachable via different IP and port
# pairs. The following two options can be used by a replica in order to
# report to its master a specific set of IP and port, so that both INFO
# and ROLE will report those values.
#
# There is no need to use both the options if you need to override just
# the port or the IP address.
#
# replica-announce-ip 5.5.5.5
# replica-announce-port 1234

################################## SECURITY 安全 ###################################

# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
# 要求客户端在处理任何其他命令之前发出AUTH <PASSWORD>。 
# 在您不信任其他人无法访问运行redis-server的主机的环境中，这可能很有用。

#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
# requirepass 配置可以让用户使用 AUTH 命令来认证密码，才能使用其他命令。这让 redis 可以使用在不受信任的网络中。为了保持向后的兼容性，可以注释该命令，因为大部分用户也不需要认证。使用 requirepass 的时候需要注意，因为 redis 太快了，每秒可以认证 15w 次密码，简单的密码很容易被攻破，所以最好使用一个更复杂的密码
#
# requirepass foobared

# Command renaming.
# 命令重命名
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
# 可以在共享环境中更改危险命令的名称。 例如，可以将CONFIG命令重命名为一些难以猜测的名称，
# 以便它仍可用于内部使用的工具，但不适用于一般客户。
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to replicas may cause problems.
# 请注意，更改 命令名称 记录到AOF文件或传输到副本可能会导致问题。

################################### CLIENTS 客户端 ####################################

# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
#
# 同一时间最大客户端连接数
# maxclients 10000

############################## MEMORY MANAGEMENT 内存管理 ################################

# Set a memory usage limit to the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
# 将内存使用限制设置为指定的字节数。当达到内存限制时，Redis将根据选择的搬迁策略，尝试删除 key。
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
# 如果Redis无法根据策略删除 key，或者如果策略设置为“ noeviction”，Redis将开始响应命令错误
# 这将使用更多的内存，例如SET，LPUSH等些命令，但仍可以继续回复诸如GET之类的只读命令。
#
# This option is usually useful when using Redis as an LRU or LFU cache, or to
# set a hard memory limit for an instance (using the 'noeviction' policy).
# 当将Redis用作LRU或LFU缓存时，设置实例的硬性内存限制，此选项通常很有用。
#
# WARNING: If you have replicas attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the replicas are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of replicas is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
# 如果您将副本附加到具有maxmemory的实例，减去提供副本所需的输出缓冲区的大小
# 从已用内存计数中，以便网络问题/重新同步不会触发退出键的循环，进而导致输出
# 副本缓冲区已满，有逐出的DEL键触发删除直到数据库完全清空为止。

#
# In short... if you have replicas attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for replica
# output buffers (but this is not needed if the policy is 'noeviction').
# 简而言之...如果您附加了副本，建议您设置一个较低的 maxmemory 的限制，
# 以便系统上有一些可用的RAM用于复制输出缓冲区（但是如果策略为“ noeviction”，则不需要这样做）。
#
# maxmemory <bytes>

# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
# 达到 maxmemory 时，Redis 将如何选择要删除的内容。 您可以选择以下五种行为：
#
# volatile-lru -> Evict using approximated LRU among the keys with an expire set.
# 使用具有过期时间的 keys 中的近似 LRU 进行驱逐（LRU：Least Recently Used，最近最少使用）。
# allkeys-lru -> Evict any key using approximated LRU.
# 驱逐任何最近最少使用的 key。
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
# 使用具有过期时间的 keys 中的近似 LFU 进行驱逐（LFU：least frequently used，最不经常使用）。
# allkeys-lfu -> Evict any key using approximated LFU.
# 驱逐任何最不经常使用的 key。
# volatile-random -> Remove a random key among the ones with an expire set.
# 随机去除带有过期时间的 key。
# allkeys-random -> Remove a random key, any key.
# 随机去除任何 key。
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# 驱逐最接近过期时间的 key。
# noeviction -> Don't evict anything, just return an error on write operations.
# 不驱逐任何东西，仅在写操作时返回错误
#
# LRU means Least Recently Used（最近最少使用）
# LFU means Least Frequently Used（最不经常使用）
#
# Both LRU, LFU and volatile-ttl are implemented using approximated
# randomized algorithms. （LRU，LFU和volatile-ttl均使用近似随机算法实现）
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
# 使用上述任何策略，当没有合适的键时驱逐时，Redis将在写入操作中返回错误。
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy noeviction

# LRU, LFU and minimal TTL algorithms are not precise algorithms but approximated
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. For default Redis will check five keys and pick the one that was
# used less recently, you can change the sample size using the following
# configuration directive.
# LRU，LFU和最小TTL算法不是精确算法，而是近似算法（以节省内存），因此您可以针对速度或准确性进行调整。 
# 对于默认情况，Redis将检查 5 个键并选择最近使用少的键，您可以使用以下配置指令更改样本大小。
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs more CPU. 3 is faster but not very accurate.
# 默认值为5会产生足够好的结果。 10非常接近真实的LRU，但是会花费更多的CPU。 3更快，但不是很准确。
#
# maxmemory-samples 5

# Starting from Redis 5, by default a replica will ignore its maxmemory setting
# (unless it is promoted to master after a failover or manually). It means
# that the eviction of keys will be just handled by the master, sending the
# DEL commands to the replica as keys evict in the master side.
# 从Redis 5开始，默认情况下，从服务器将忽略其 maxmemory 设置（除非在故障转移后或手动提升为主副本）。
# 这意味着 keys 删除将仅由主服务器处理，将 DEL 命令作为副本在主计算机侧逐出，将 DEL 命令发送到副本。
#
# This behavior ensures that masters and replicas stay consistent, and is usually
# what you want, however if your replica is writable, or you want the replica to have
# a different memory setting, and you are sure all the writes performed to the
# replica are idempotent, then you may change this default (but be sure to understand
# what you are doing).
# 此行为可确保主副本和副本始终保持一致，这通常是您想要的，但是，如果副本是可写的，
# 或者您希望副本具有不同的内存设置，并且您确定对副本执行的所有写操作都是幂等的， 
# 那么您可以更改此默认设置（但请务必了解您的操作）。
#
# Note that since the replica by default does not evict, it may end using more
# memory than the one set via maxmemory (there are certain buffers that may
# be larger on the replica, or data structures may sometimes take more memory and so
# forth). So make sure you monitor your replicas and make sure they have enough
# memory to never hit a real out-of-memory condition before the master hits
# the configured maxmemory setting.
# 请注意，由于默认情况下该副本不会退出，因此它可能会使用比通过maxmemory设置的内存更多的内存
#（某些副本上的某些缓冲区可能更大，或者数据结构有时会占用更多内存等等）。 因此，请确保您监视副本，
# 并确保副本具有足够的内存，以确保在主副本达到配置的最大内存设置之前，永不达到实际的内存不足状态。
#
# replica-ignore-maxmemory yes

############################# LAZY FREEING 懒惰释放 ####################################

# Redis has two primitives to delete keys. One is called DEL and is a blocking
# deletion of the object. It means that the server stops processing new commands
# in order to reclaim all the memory associated with an object in a synchronous
# way. If the key deleted is associated with a small object, the time needed
# in order to execute the DEL command is very small and comparable to most other
# O(1) or O(log_N) commands in Redis. However if the key is associated with an
# aggregated value containing millions of elements, the server can block for
# a long time (even seconds) in order to complete the operation.
# Redis有两个删除键的原语。 一种称为DEL，它是对象的阻塞删除。 这意味着服务器停止处理新命令，
# 以便以同步方式回收与对象关联的所有内存。 如果删除的键与一个小对象相关联，
# 则执行DEL命令所需的时间非常短，可与大多数其他对象相比 Redis 中的 O（1）或 O（log_N）命令。 
# 但是，如果键与包含数百万个元素的聚合值关联，则服务器可能会阻塞很长时间（甚至几秒钟）以完成操作。
#
# For the above reasons Redis also offers non blocking deletion primitives
# such as UNLINK (non blocking DEL) and the ASYNC option of FLUSHALL and
# FLUSHDB commands, in order to reclaim memory in background. Those commands
# are executed in constant time. Another thread will incrementally free the
# object in the background as fast as possible.
# 由于上述原因，Redis还提供了非阻塞删除原语，例如UNLINK（非阻塞DEL）以及FLUSHALL和FLUSHDB命令的
# ASYNC选项，以便在后台回收内存。 这些命令在固定时间内执行。 另一个线程将尽可能快地在后台逐渐释放对象。
#
# DEL, UNLINK and ASYNC option of FLUSHALL and FLUSHDB are user-controlled.
# It's up to the design of the application to understand when it is a good
# idea to use one or the other. However the Redis server sometimes has to
# delete keys or flush the whole database as a side effect of other operations.
# Specifically Redis deletes objects independently of a user call in the
# following scenarios:
# 用户可以控制FLUSHALL和FLUSHDB的DEL，UNLINK和ASYNC选项。 由应用程序的设计来决定何时使用
# 一个或另一个是一个好主意。 但是，Redis服务器有时必须删除键或刷新整个数据库，这是其他操作的副作用。 
# 特别是在以下情况下，Redis会独立于用户调用删除对象：
#
# 1) On eviction, because of the maxmemory and maxmemory policy configurations,
#    in order to make room for new data, without going over the specified
#    memory limit.
# 在逐出时，由于maxmemory和maxmemory策略配置，为了在不超过指定的内存限制的情况下为新数据腾出空间。
# 2) Because of expire: when a key with an associated time to live (see the
#    EXPIRE command) must be deleted from memory.
# 由于过期：必须从内存中删除具有相关生存时间的 key（请参阅 EXPIRE 命令）。
# 3) Because of a side effect of a command that stores data on a key that may
#    already exist. For example the RENAME command may delete the old key
#    content when it is replaced with another one. Similarly SUNIONSTORE
#    or SORT with STORE option may delete existing keys. The SET command
#    itself removes any old content of the specified key in order to replace
#    it with the specified string.
# 由于将数据存储在可能已经存在的键上的命令的副作用。 例如，当 RENAME 命令被另一旧密钥内容替换时，
# 它可能会删除它。 同样，SUNIONSTORE 或S ORT with STORE 选项可能会删除现有密钥。 
# SET 命令本身会删除指定键的所有旧内容，以便将其替换为指定的字符串。
# 4) During replication, when a replica performs a full resynchronization with
#    its master, the content of the whole database is removed in order to
#    load the RDB file just transferred.
# 在复制期间，当副本与其主副本执行完全重新同步时，将删除整个数据库的内容，以便加载刚传输的 RDB 文件。
#
# In all the above cases the default is to delete objects in a blocking way,
# like if DEL was called. However you can configure each case specifically
# in order to instead release memory in a non-blocking way like if UNLINK
# was called, using the following configuration directives:
# 在上述所有情况下，默认设置都是以阻塞方式删除对象，就像调用DEL一样。 但是，您可以专门配置每种情况，
# 以便使用以下配置指令以非阻塞方式释放内存，例如调用UNLINK的情况：

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

############################## APPEND ONLY MODE AOF日志模式 ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
# 默认情况下，Redis异步将数据集转储到磁盘上。 此模式在许多应用程序中已经足够好，
# 但是Redis进程问题或电源中断可能会导致几分钟的写入丢失（取决于配置的保存点）。
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
# 仅附加文件是一种替代的持久性模式，可提供更好的持久性。 例如，使用默认数据fsync策略（请参阅配置文件中的稍后内容），Redis可能在服务器断电等严重事件中丢失一秒钟的写入，如果Redis进程本身发生错误，则一次写入将丢失一次，但是 操作系统仍在正常运行。
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
# 可以同时启用AOF和RDB持久性，而不会出现问题。 如果在启动时启用了AOF，则Redis将加载AOF，
# 即该文件具有更好的持久性保证。
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"

# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
# fsync()调用告诉操作系统将数据实际写在磁盘上，而不是等待输出缓冲区中的更多数据。 有些操作系统确实会刷新磁盘上的数据，而另一些操作系统只会尝试尽快完成该操作。
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
#			不要fsync，只要让OS在需要时刷新数据即可。快。
# always: fsync after every write to the append only log. Slow, Safest.
#			每次写入 AOF 后 fsync。慢，安全
# everysec: fsync only one time every second. Compromise.
#			每秒仅同步一次fsync。 妥协。
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
# 默认值为“ everysec”，因为这通常是速度和数据安全性之间的正确折衷。 您可以了解是否可以将其放宽为“ no”，
# 以便操作系统在需要时刷新输出缓冲区，以获得更好的性能（但是如果您可以忍受某些数据丢失的想法，
# 请考虑使用默认的持久模式 （即快照），或者相反，请使用“总是”，该速度非常慢，但比 everysec 更安全。

#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
appendfsync everysec
# appendfsync no

# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
# 当AOF fsync 策略设置为 always 或 everysec，并且后台保存进程（后台保存或 AOF 日志后台重写）对磁盘执行大量I / O时，在某些Linux配置中，Redis fsync() 调用可能会在磁盘上阻塞太长时间。 请注意，目前尚无此修复程序，因为即使在其他线程中执行 fsync 也将阻止我们的同步 write（2）调用。
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
# 为了缓解此问题，可以使用以下选项来防止在 BGSAVE 或 BGREWRITEAOF 进行时在主进程中调用 fsync()。
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
# 这意味着当另一个孩子正在保存时，Redis 的持久性与“ appendfsync none”相同。 实际上，
# 这意味着在最坏的情况下（使用默认的 Linux 设置）可能会丢失多达 30 秒的日志。
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.
#  如果您有延迟问题，请将其设置为“是”。 否则将其保留为,从耐用性的角度来看，“ no”是最安全的选择。

no-appendfsync-on-rewrite no

# Automatic rewrite of the append only file.（自动重写 AOF。）
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
# 当AOF日志大小增加指定的百分比时，Redis可以隐式调用 BGREWRITEAOF 自动重写日志文件。
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
# 它是这样工作的：Redis会在最近一次重写后记住AOF文件的大小（如果自重新启动以来未发生任何重写，则使用启动时AOF的大小）。
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
# 将此基本大小与当前大小进行比较。 如果当前大小大于指定的百分比，则触发重写。 
# 另外，您需要指定要重写的AOF文件的最小大小，这对于避免重写AOF文件非常有用，
# 即使达到百分比增加，但它仍然很小。
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.   
# 指定零百分比以禁用自动AOF重写功能。

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
# AOF 文件可能在尾部是不完整的，当 Redis 启动的时候，AOF文件的数据被载入内存。重启可能发生在redis所在的主机操作系统宕机后，尤其在ext4文件系统没有加上data=ordered选项（redis宕机或者异常终止不会造成尾部不完整现象。）出现这种现象，可以选择让redis退出，或者导入尽可能多的数据。
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
# 发生这种情况时，Redis 可能会退出并显示错误，也可以在发现 AOF 文件最后被截断后，加载尽可能多的数据（当前为默认值）。以下选项控制此行为。
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
# 如果 aof-load-truncated 设置为 yes，则将加载截短的 AOF 文件，并且 Redis 服务器将开始
# 发出日志以将事件通知用户。否则，如果该选项设置为 no，则服务器将中止并显示错误，并拒绝启动。
# 如果将该选项设置为 no，则用户需要在重新启动服务器之前使用“ redis-check-aof”修复 AOF 文件。
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
# 请注意，如果在中间发现 AOF 文件已损坏，则服务器仍将退出并出现错误。 仅当 Redis 尝试从 AOF 
# 文件读取更多数据但找不到足够的字节时，此选项才适用。

aof-load-truncated yes

# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
# 重写AOF文件时，Redis可以使用AOF文件中的RDB前同步码来更快地进行重写和恢复。 启用此选项后，重写的AOF文件由两个不同的节组成：
#
#   [RDB file][AOF tail]
#
# When loading Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, and continues loading the AOF
# tail.
# 加载时，Redis会识别AOF文件以“ REDIS”字符串开头并加载带前缀的RDB文件，然后继续加载AOF尾部。
aof-use-rdb-preamble yes

################################ LUA SCRIPTING LUA脚本  ###############################

# Max execution time of a Lua script in milliseconds.（Lua脚本的最大执行时间（以毫秒为单位）。）
#
# If the maximum execution time is reached Redis will log that a script is
# still in execution after the maximum allowed time and will start to
# reply to queries with an error.
# 如果达到了最大执行时间，Redis将记录在允许的最大时间后脚本仍在执行中，并将开始以错误答复查询。
#
# When a long running script exceeds the maximum execution time only the
# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
# used to stop a script that did not yet called write commands. The second
# is the only way to shut down the server in the case a write command was
# already issued by the script but the user doesn't want to wait for the natural
# termination of the script.
# 如果长时间运行的脚本超过了最大执行时间，则只有“ SCRIPT KILL”和“ SHUTDOWN NOSAVE”命令可用。第一个可用于停止尚未调用写命令的脚本。 第二种是在脚本已发出写命令但用户不想等待脚本自然终止的情况下关闭服务器的唯一方法。
#
# Set it to 0 or a negative value for unlimited execution without warnings.
# 将其设置为0或负值可无警告地无限执行。
lua-time-limit 5000

################################ REDIS CLUSTER REDIS集群 ##############################

# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
# 普通Redis实例不能属于Redis集群；只有作为群集节点启动的节点可以。为了将 Redis 实例作为群集节点启动，请在不注释以下内容的情况下启用群集支持：
#
# cluster-enabled yes

# Every cluster node has a cluster configuration file. This file is not
# intended to be edited by hand. It is created and updated by Redis nodes.
# Every Redis Cluster node requires a different cluster configuration file.
# Make sure that instances running in the same system do not have
# overlapping cluster configuration file names.
# 每个群集节点都有一个群集配置文件。 该文件不适合手工编辑。 它由Redis节点创建和更新。 每个Redis群集节点都需要一个不同的群集配置文件。 确保在同一系统上运行的实例没有重叠的集群配置文件名。
#
# cluster-config-file nodes-6379.conf

# Cluster node timeout is the amount of milliseconds a node must be unreachable
# for it to be considered in failure state.
# Most other internal time limits are multiple of the node timeout.
# 群集节点超时是一个节点必须不可达的毫秒数，才能将其视为故障状态。 
# 其他大多数内部时间限制是节点超时的倍数。
#
# cluster-node-timeout 15000

# A replica of a failing master will avoid to start a failover if its data
# looks too old.
# 如果发生故障的主副本的数据看起来太旧，它将避免启动故障转移。
#
# There is no simple way for a replica to actually have an exact measure of
# its "data age", so the following two checks are performed:
# 没有一种简单的方法可以使副本实际上具有其“数据年龄”的准确度量，因此执行以下两项检查：
#
# 1) If there are multiple replicas able to failover, they exchange messages
#    in order to try to give an advantage to the replica with the best
#    replication offset (more data from the master processed).
#    Replicas will try to get their rank by offset, and apply to the start
#    of the failover a delay proportional to their rank.
# 如果存在多个能够进行故障转移的副本，则它们会交换消息，以便尝试利用具有最佳复制偏移量（处理了更多来自主数据库的数据）的副本来获得优势。 副本将尝试按偏移量获取其等级，并将与它们的等级成比例的延迟应用于故障转移。
#
# 2) Every single replica computes the time of the last interaction with
#    its master. This can be the last ping or command received (if the master
#    is still in the "connected" state), or the time that elapsed since the
#    disconnection with the master (if the replication link is currently down).
#    If the last interaction is too old, the replica will not try to failover
#    at all.
# 每个单个副本都会计算与其母版之间最后一次交互的时间。 这可以是最后收到的ping或命令（如果主服务器仍处于“已连接”状态），也可以是自从与主服务器断开连接以来经过的时间（如果复制链接当前已断开）。如果最后一次交互也是 旧版本，副本将完全不会尝试故障转移。
#
# The point "2" can be tuned by user. Specifically a replica will not perform
# the failover if, since the last interaction with the master, the time
# elapsed is greater than:
# 用户可以调整点“ 2”。 具体而言，如果自从上次与主服务器进行交互以来，经过的时间大于以下时间，则副本将不执行故障转移：
#
#   (node-timeout * replica-validity-factor) + repl-ping-replica-period
#
# So for example if node-timeout is 30 seconds, and the replica-validity-factor
# is 10, and assuming a default repl-ping-replica-period of 10 seconds, the
# replica will not try to failover if it was not able to talk with the master
# for longer than 310 seconds.
# 因此，例如，如果节点超时（node-timeout）为30秒，并且副本有效性因子（replica-validity-factor）为10，并且假设默认的 repl-ping-replica-period 值为10秒，即如果超过 310 秒副本将不会尝试进行故障转移。
#
# A large replica-validity-factor may allow replicas with too old data to failover
# a master, while a too small value may prevent the cluster from being able to
# elect a replica at all.
# 较大的 replica-validity-factor 可能会使数据过旧的副本无法对主副本进行故障转移，而值太小则可能会使群集根本无法选择副本。
#
# For maximum availability, it is possible to set the replica-validity-factor
# to a value of 0, which means, that replicas will always try to failover the
# master regardless of the last time they interacted with the master.
# (However they'll always try to apply a delay proportional to their
# offset rank).
# 为了获得最大可用性，可以将副本有效性因子设置为 0，这意味着，无论副本上次与主服务器进行交互时，副本将始终尝试对主服务器进行故障转移。（但是，他们将始终尝试应用与其偏移等级成正比的延迟）。
#
# Zero is the only value able to guarantee that when all the partitions heal
# the cluster will always be able to continue.
# 零是唯一能够保证当所有分区恢复正常后群集将始终能够继续运行的值。
#
# cluster-replica-validity-factor 10

# Cluster replicas are able to migrate to orphaned masters, that are masters
# that are left without working replicas. This improves the cluster ability
# to resist to failures as otherwise an orphaned master can't be failed over
# in case of failure if it has no working replicas.
# 群集副本能够迁移到孤立的主数据库，即那些没有工作副本的主数据库。这样可以提高群集抵御故障的能力，因为如果孤立的主节点没有可用的副本，则该主节点在发生故障的情况下也无法进行故障转移。
#
# Replicas migrate to orphaned masters only if there are still at least a
# given number of other working replicas for their old master. This number
# is the "migration barrier". A migration barrier of 1 means that a replica
# will migrate only if there is at least 1 other working replica for its master
# and so forth. It usually reflects the number of replicas you want for every
# master in your cluster.
# 仅当旧的主副本仍存在至少给定数量的其他工作副本时，从副本才会迁移到孤立的主副本。这个数字是“移民壁垒”。 迁移障碍为 1 表示仅当副本数据库的主副本上至少有 1 个其他工作副本时，副本副本才会迁移。它通常反映出集群中每个主数据库所需的副本数。
#
# Default is 1 (replicas migrate only if their masters remain with at least
# one replica). To disable migration just set it to a very large value.
# A value of 0 can be set but is useful only for debugging and dangerous
# in production.
# 默认值为 1（仅当其主副本保留至少一个从副本时，从副本才会迁移）。要禁用迁移，只需将其设置为非常大的值即可。可以将值设置为 0，但仅用于调试和生产危险。
#
# cluster-migration-barrier 1

# By default Redis Cluster nodes stop accepting queries if they detect there
# is at least an hash slot uncovered (no available node is serving it).
# This way if the cluster is partially down (for example a range of hash slots
# are no longer covered) all the cluster becomes, eventually, unavailable.
# It automatically returns available as soon as all the slots are covered again.
# 默认情况下，如果 Redis 集群节点检测到至少发现一个哈希槽（没有可用的节点正在为其提供服务），它们将停止接受查询。这样如果集群部分关闭（例如，不再覆盖哈希槽的范围），则所有集群最终将变得不可用。再次覆盖所有插槽后，它将自动返回可用状态。
#
# However sometimes you want the subset of the cluster which is working,
# to continue to accept queries for the part of the key space that is still
# covered. In order to do so, just set the cluster-require-full-coverage
# option to no.
#  但是有时您希望正在运行的群集子集继续接受对仍覆盖的部分键空间的查询。为此只需将 cluster-require-full-coverage 选项设置为 no。
#
# cluster-require-full-coverage yes

# This option, when set to yes, prevents replicas from trying to failover its
# master during master failures. However the master can still perform a
# manual failover, if forced to do so.
# 设置为 yes 时，此选项可防止副本在主服务器发生故障时尝试对其主服务器进行故障转移。但是，主服务器仍然可以执行手动故障转移（如果被迫执行）。
#
# This is useful in different scenarios, especially in the case of multiple
# data center operations, where we want one side to never be promoted if not
# in the case of a total DC failure.
#  这在不同的情况下很有用，特别是在多个数据中心操作的情况下，如果我们希望在完全 DC 故障的情况下不对一侧进行升级，那么这是不希望的。
#
# cluster-replica-no-failover no

# In order to setup your cluster make sure to read the documentation
# available at http://redis.io web site.

########################## CLUSTER DOCKER/NAT support 集群DOCKER / NAT支持  ########################

# In certain deployments, Redis Cluster nodes address discovery fails, because
# addresses are NAT-ted or because ports are forwarded (the typical case is
# Docker and other containers).
# 在某些部署中，Redis 集群节点的地址发现失败，这是因为地址经过 NAT 限制或端口被转发（典型情况是 Docker 和其他容器）。
#
# In order to make Redis Cluster working in such environments, a static
# configuration where each node knows its public address is needed. The
# following two options are used for this scope, and are:
#  为了使Redis 集群在这样的环境中工作，需要一个静态配置，其中每个节点都知道其公共地址。以下两个选项用于此范围，分别是：
#
# * cluster-announce-ip			# 群集公告IP
# * cluster-announce-port		# 群集公告端口
# * cluster-announce-bus-port	# 群集公告总线端口
#
# Each instruct the node about its address, client port, and cluster message
# bus port. The information is then published in the header of the bus packets
# so that other nodes will be able to correctly map the address of the node
# publishing the information.
# 每一个都向节点指示其地址，客户端端口和集群消息总线端口。然后将信息发布在总线数据包的标头中，以便其他节点将能够正确映射发布信息的节点的地址。
#
# If the above options are not used, the normal Redis Cluster auto-detection
# will be used instead. 如果不使用上述选项，则将使用常规的Redis群集自动检测。
#
# Note that when remapped, the bus port may not be at the fixed offset of
# clients port + 10000, so you can specify any port and bus-port depending
# on how they get remapped. If the bus-port is not set, a fixed offset of
# 10000 will be used as usually.
# 请注意，重新映射时，总线端口可能不在客户端端口 + 10000的固定偏移处，因此您可以根据重新映射的方式指定任何端口和总线端口。如果未设置总线端口，通常将使用 10000 的固定偏移量。
#
# Example:
#
# cluster-announce-ip 10.1.1.5
# cluster-announce-port 6379
# cluster-announce-bus-port 6380

################################## SLOW LOG 慢日志 ###################################

# The Redis Slow Log is a system to log queries that exceeded a specified
# execution time. The execution time does not include the I/O operations
# like talking with the client, sending the reply and so forth,
# but just the time needed to actually execute the command (this is the only
# stage of command execution where the thread is blocked and can not serve
# other requests in the meantime).
# Redis Slow Log 是一个用于记录超过指定执行时间的查询的系统。执行时间不包括与客户端交谈，发送回复等I / O操作，而是实际执行命令所需的时间（这是命令执行的唯一阶段，在该阶段线程被阻塞并且可以在此期间不满足其他要求）。
#
# You can configure the slow log with two parameters: one tells Redis
# what is the execution time, in microseconds, to exceed in order for the
# command to get logged, and the other parameter is the length of the
# slow log. When a new command is logged the oldest one is removed from the
# queue of logged commands.
# 您可以使用以下两个参数配置慢速日志：一个告诉 Redis 为了使命令被记录而超过执行时间（以微秒为单位），另一个参数是慢速日志的长度。记录新命令时，最旧的命令将从记录的命令队列中删除。

# The following time is expressed in microseconds, so 1000000 is equivalent
# to one second. Note that a negative number disables the slow log, while
# a value of zero forces the logging of every command.
# 以下时间以微秒表示，因此 1000000 等于一秒。请注意，负数将禁用慢速日志记录，而零值将强制记录每个命令。
slowlog-log-slower-than 10000

# There is no limit to this length. Just be aware that it will consume memory.
# You can reclaim memory used by the slow log with SLOWLOG RESET.
# 慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录会被删掉。这个长度没有限制。只要有足够的内存就行。你可以通过 SLOWLOG RESET 来释放内存。
slowlog-max-len 128

################################ LATENCY MONITOR 延迟监视器 ##############################

# The Redis latency monitoring subsystem samples different operations
# at runtime in order to collect data related to possible sources of
# latency of a Redis instance.
# Redis 延迟监视子系统会在运行时对不同的操作进行采样，以收集与 Redis 实例的潜在延迟源相关的数据。
#
# Via the LATENCY command this information is available to the user that can
# print graphs and obtain reports. 通过 LATENCY 命令，该信息可以打印图形并获取报告。
#
# The system only logs operations that were performed in a time equal or
# greater than the amount of milliseconds specified via the
# latency-monitor-threshold configuration directive. When its value is set
# to zero, the latency monitor is turned off.
# 系统仅记录在等于或大于通过 delay-monitor-threshold 配置指令指定的毫秒量的时间内执行的操作。当其值设置为零时，等待时间监视器将关闭。
#
# By default latency monitoring is disabled since it is mostly not needed
# if you don't have latency issues, and collecting data has a performance
# impact, that while very small, can be measured under big load. Latency
# monitoring can easily be enabled at runtime using the command
# "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.
# 默认情况下，延迟监视是禁用的，因为如果您没有延迟问题，则通常不需要监视，并且收集数据会对性能产生影响，尽管影响很小，但是可以在大负载下进行测量。 如果需要，可以在运行时使用命令“ CONFIG SET delay-monitor-threshold <milliseconds>”轻松启用延迟监视。
latency-monitor-threshold 0

############################# EVENT NOTIFICATION 活动通知 ##############################

# Redis can notify Pub/Sub clients about events happening in the key space.
# Redis 可以通知发布/订阅客户端有关 key 空间中发生的事件。
# This feature is documented at http://redis.io/topics/notifications
#
# For instance if keyspace events notification is enabled, and a client
# performs a DEL operation on key "foo" stored in the Database 0, two
# messages will be published via Pub/Sub:
# 例如，如果启用了键空间事件通知，并且客户端对存储在数据库0中的键“ foo”执行了DEL操作，则将通过Pub / Sub发布两条消息：
#
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# It is possible to select the events that Redis will notify among a set
# of classes. Every class is identified by a single character:
# 可以在一组类中选择Redis将通知的事件。 每个类都由一个字符标识：
#
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
#
#  The "notify-keyspace-events" takes as argument a string that is composed
#  of zero or multiple characters. The empty string means that notifications
#  are disabled.
# “notify-keyspace-events”将由零个或多个字符组成的字符串作为参数。 空字符串表示已禁用通知。
#
#  Example: to enable list and generic events, from the point of view of the
#           event name, use: 从事件名称的角度来看，要启用列表事件和通用事件，请使用：
#
#  notify-keyspace-events Elg
#
#  Example 2: to get the stream of the expired keys subscribing to channel
#             name __keyevent@0__:expired use: 获取订阅频道名称__keyevent @0__的过期 keys 流：过期使用：
#
#  notify-keyspace-events Ex
#
#  By default all notifications are disabled because most users don't need
#  this feature and the feature has some overhead. Note that if you don't
#  specify at least one of K or E, no events will be delivered.
# 默认情况下，所有通知都被禁用，因为大多数用户不需要此功能，并且该功能有一些开销。 请注意，如果您未指定K或E中的至少一个，则不会传递任何事件。
notify-keyspace-events ""

############################### ADVANCED CONFIG ###############################

# Hashes are encoded using a memory efficient data structure when they have a
# small number of entries, and the biggest entry does not exceed a given
# threshold. These thresholds can be configured using the following directives.
# 当哈希条目只有少量条目且最大条目没有超过给定阈值时，将使用内存高效的数据结构对其进行编码。 可以使用以下指令配置这些阈值。
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# Lists are also encoded in a special way to save a lot of space.
# The number of entries allowed per internal list node can be specified
# as a fixed maximum size or a maximum number of elements.
# 列表也以特殊方式编码，以节省大量空间。 每个内部列表节点允许的条目数可以指定为固定的最大大小或最大元素数。
# For a fixed maximum size, use -5 through -1, meaning:
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# Positive numbers mean store up to _exactly_ that number of elements
# per list node. 正数表示每个列表节点最多可以存储该数量的元素。
# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
# but if your use case is unique, adjust the settings as necessary.
# 性能最高的选项通常是-2（8 Kb大小）或-1（4 Kb大小），但是如果您的用例是唯一的，请根据需要调整设置。
list-max-ziplist-size -2

# Lists may also be compressed.
# Compress depth is the number of quicklist ziplist nodes from *each* side of
# the list to *exclude* from compression.  The head and tail of the list
# are always uncompressed for fast push/pop operations.  Settings are:
# 列表也可以被压缩。 压缩深度是指从列表的*每个*一侧到*从压缩中排除*的快速列表 ziplist 节点的数量。 列表的开头和结尾始终是未压缩的，以便快速进行推入/弹出操作。 设置为：
# 0: disable all list compression	禁用所有列表压缩
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#	深度 1 表示“在列表中的 1 个节点之后才开始压缩，
#    going from either the head or tail"	从头部或尾部
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
# 	[头部]，[尾部]将始终未压缩；内部节点将压缩。
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
#	2 这里的意思是：不要压缩头部或头部->下一个或尾部->上一个或尾部，但是压缩它们之间的所有节点。
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# etc.
list-compress-depth 0	# 压缩深度

# Sets have a special encoding in just one case: when a set is composed
# of just strings that happen to be integers in radix 10 in the range
# of 64 bit signed integers.
# 在仅一种情况下，集合具有特殊的编码：当集合仅由字符串组成，这些字符串恰好是基数为10的整数，范围为64位有符号整数。
# The following configuration setting sets the limit in the size of the
# set in order to use this special memory saving encoding.
# 以下配置设置设置了大小限制，以便使用此特殊的内存节省编码。
set-max-intset-entries 512

# Similarly to hashes and lists, sorted sets are also specially encoded in
# order to save a lot of space. This encoding is only used when the length and
# elements of a sorted set are below the following limits:
# 与哈希表和列表类似，对有序集也进行了特殊编码，以节省大量空间。仅当有序集的长度和元素低于以下限制时，才使用此编码：
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

# HyperLogLog sparse representation bytes limit. The limit includes the
# 16 bytes header. When an HyperLogLog using the sparse representation crosses
# this limit, it is converted into the dense representation.
# HyperLogLog 稀疏表示形式的字节数限制。限制包括16个字节的标头。当使用稀疏表示的 HyperLogLog 超过此限制时，它将转换为密集表示。
#
# A value greater than 16000 is totally useless, since at that point the
# dense representation is more memory efficient.
# 大于 16000 的值是完全没有用的，因为在那一点上，密集表示的存储效率更高。
#
# The suggested value is ~ 3000 in order to have the benefits of
# the space efficient encoding without slowing down too much PFADD,
# which is O(N) with the sparse encoding. The value can be raised to
# ~ 10000 when CPU is not a concern, but space is, and the data set is
# composed of many HyperLogLogs with cardinality in the 0 - 15000 range.
# 建议值约为 3000，以便在不降低过多 PFADD 的情况下获得节省空间编码的好处，而 PFADD 的稀疏编码为O（N）。 当不关心 CPU 但空间很大时，该值可以提高到 10000，并且数据集由基数在 0-15000 范围内的许多 HyperLogLog 组成。
hll-sparse-max-bytes 3000

# Streams macro node max size / items. The stream data structure is a radix
# tree of big nodes that encode multiple items inside. Using this configuration
# it is possible to configure how big a single node can be in bytes, and the
# maximum number of items it may contain before switching to a new node when
# appending new stream entries. If any of the following settings are set to
# zero, the limit is ignored, so for instance it is possible to set just a
# max entires limit by setting max-bytes to 0 and max-entries to the desired
# value.
# 流宏节点最大大小/项目。流数据结构是一个大节点的基数树，它对内部的多个项目进行编码。使用此配置，可以配置单个节点的大小（以字节为单位），以及在添加新的流条目时切换到新节点之前它可能包含的最大项目数。如果以下任何设置被设置为零，则该限制将被忽略，例如，可以通过将 max-bytes 设置为 0 并将 max-entries 设置为所需值来仅设置最大整体限制。
stream-node-max-bytes 4096
stream-node-max-entries 100

# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
# order to help rehashing the main Redis hash table (the one mapping top-level
# keys to values). The hash table implementation Redis uses (see dict.c)
# performs a lazy rehashing: the more operation you run into a hash table
# that is rehashing, the more rehashing "steps" are performed, so if the
# server is idle the rehashing is never complete and some more memory is used
# by the hash table.
# 活动重新哈希处理每 100 毫秒 CPU 时间使用 1 毫秒，以帮助重新哈希主 Redis 哈希表（将顶级键映射到值的一个哈希表）。Redis 使用的哈希表实现（请参见dict.c）执行一次懒惰的重新哈希处理：您在要进行哈希处理的哈希表中运行的操作越多，执行的哈希处理“步骤”就越多，因此，如果服务器空闲，则哈希处理将永远不会完成 散列表使用更多的内存。
#
# The default is to use this millisecond 10 times every second in order to
# actively rehash the main dictionaries, freeing memory when possible.
# 默认值是每秒使用 10 毫秒的毫秒数来主动重新哈希主字典，并在可能的情况下释放内存。
#
# If unsure:
# use "activerehashing no" if you have hard latency requirements and it is
# not a good thing in your environment that Redis can reply from time to time
# to queries with 2 milliseconds delay.
# 如果不确定：如果您有严格的延迟要求，请使用“ activerehashing no”，并且在您的环境中，Redis 可以不时地以 2 毫秒的延迟答复查询不是一件好事。
#
# use "activerehashing yes" if you don't have such hard requirements but
# want to free memory asap when possible.
# 如果您没有如此严格的要求，但希望在可能的情况下尽快释放内存，请使用“ activerehashing yes”。
activerehashing yes

# The client output buffer limits can be used to force disconnection of clients
# that are not reading data from the server fast enough for some reason (a
# common reason is that a Pub/Sub client can't consume messages as fast as the
# publisher can produce them).
# 客户端输出缓冲区限制可用于强制出于某些原因而无法以足够快的速度从服务器读取数据的客户端断开连接（常见原因是Pub / Sub客户端无法像发布者产生消息那样快地消耗消息）。
#
# The limit can be set differently for the three different classes of clients:
# 可以为三种不同类别的客户端设置不同的限制：
#
# normal -> normal clients including MONITOR clients	普通客户端，包括 MONITOR 客户端
# replica  -> replica clients	 副本客户端
# pubsub -> clients subscribed to at least one pubsub channel or pattern
#			客户端至少订阅了一个 pubsub 频道或模式
#
# The syntax of every client-output-buffer-limit directive is the following:
#  每个 client-output-buffer-limit 指令的语法如下：
#
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# A client is immediately disconnected once the hard limit is reached, or if
# the soft limit is reached and remains reached for the specified number of
# seconds (continuously).
#  一旦达到硬限制，或者达到软限制并在指定的秒数内（连续）保持达到此限制，客户端将立即断开连接。
# So for instance if the hard limit is 32 megabytes and the soft limit is
# 16 megabytes / 10 seconds, the client will get disconnected immediately
# if the size of the output buffers reach 32 megabytes, but will also get
# disconnected if the client reaches 16 megabytes and continuously overcomes
# the limit for 10 seconds.
# 因此，例如，如果硬限制为 32 兆字节，软限制为 16 兆字节/ 10秒，则如果输出缓冲区的大小达到 32 兆字节，客户端将立即断开连接，但如果客户端达到 16 兆字节，则也会断开连接。持续超过限制 10 秒钟。
#
# By default normal clients are not limited because they don't receive data
# without asking (in a push way), but just after a request, so only
# asynchronous clients may create a scenario where data is requested faster
# than it can read.
# 默认情况下，普通客户端不受限制，因为它们不会在不询问的情况下（以推送方式）接收数据，而是在请求之后才接收数据，因此，只有异步客户端才能创建这样的场景：请求数据的速度比读取数据的速度快。
#
# Instead there is a default limit for pubsub and replica clients, since
# subscribers and replicas receive data in a push fashion.
# 相反，由于订阅者和副本以推送方式接收数据，因此对 pubsub 和副本客户端没有默认限制。
#
# Both the hard or the soft limit can be disabled by setting them to zero.
#  硬限制或软限制都可以通过将其设置为零来禁用。
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# Client query buffers accumulate new commands. They are limited to a fixed
# amount by default in order to avoid that a protocol desynchronization (for
# instance due to a bug in the client) will lead to unbound memory usage in
# the query buffer. However you can configure it here if you have very special
# needs, such us huge multi/exec requests or alike.
# 客户端查询缓冲区会累积新命令。默认情况下，它们被限制为固定数量，以避免协议不同步（例如由于客户端错误）导致查询缓冲区中的未绑定内存使用。但是，如果您有非常特殊的需求，例如巨大的 multi / exec请求等，则可以在此处进行配置。
#
# client-query-buffer-limit 1gb

# In the Redis protocol, bulk requests, that are, elements representing single
# strings, are normally limited ot 512 mb. However you can change this limit
# here.
# 在Redis协议中，批量请求（即表示单个字符串的元素）通常限制为512 mb。 但是，您可以在此处更改此限制。
#
# proto-max-bulk-len 512mb

# Redis calls an internal function to perform many background tasks, like
# closing connections of clients in timeout, purging expired keys that are
# never requested, and so forth.
# Redis 调用一个内部函数来执行许多后台任务，例如超时关闭客户端连接，清除从未请求的过期 key 等。
#
# Not all tasks are performed with the same frequency, but Redis checks for
# tasks to perform according to the specified "hz" value.
# 并非所有任务都以相同的频率执行，但是Redis根据指定的“ hz”值检查要执行的任务。
#
# By default "hz" is set to 10. Raising the value will use more CPU when
# Redis is idle, but at the same time will make Redis more responsive when
# there are many keys expiring at the same time, and timeouts may be
# handled with more precision.
#   默认情况下，“ hz”设置为10。提高该值将在 Redis 空闲时使用更多的 CPU，但是同时当有多个键同时到期时，它将使 Redis 的响应速度更快，并且可以使超时处理更精确。
#
# The range is between 1 and 500, however a value over 100 is usually not
# a good idea. Most users should use the default of 10 and raise this up to
# 100 only in environments where very low latency is required.
#  范围在1到500之间，但是通常不建议超过100。 大多数用户应该使用默认值10，并且仅在要求非常低延迟的环境中才将其提高到100。
hz 10

# Normally it is useful to have an HZ value which is proportional to the
# number of clients connected. This is useful in order, for instance, to
# avoid too many clients are processed for each background task invocation
# in order to avoid latency spikes.
# 通常，具有与连接的客户端数量成比例的 HZ 值很有用。例如，这有助于避免每次后台任务调用处理过多的客户端，从而避免延迟尖峰。
#
# Since the default HZ value by default is conservatively set to 10, Redis
# offers, and enables by default, the ability to use an adaptive HZ value
# which will temporary raise when there are many connected clients.
# 由于默认的默认 HZ 值保守地设置为 10，因此 Redis 提供并默认启用了使用自适应 HZ 值的能力，当有许多连接的客户端时，该值会暂时升高。
#
# When dynamic HZ is enabled, the actual configured HZ will be used as
# as a baseline, but multiples of the configured HZ value will be actually
# used as needed once more clients are connected. In this way an idle
# instance will use very little CPU time while a busy instance will be
# more responsive.
# 启用动态 HZ 后，实际配置的 HZ 将用作基准，但是一旦连接了更多客户端，实际将使用配置的 HZ 值的倍数。 这样，空闲实例将占用很少的 CPU 时间，而忙碌的实例将具有更高的响应速度。
dynamic-hz yes

# When a child rewrites the AOF file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
# 当重写 AOF 文件时，如果启用了以下选项，则每生成 32MB的 数据，文件就会进行同步处理。为了将文件更多地提交到磁盘并避免大的延迟尖峰，这很有用。
aof-rewrite-incremental-fsync yes

# When redis saves RDB file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
# 当 redis 保存 RDB 文件时，如果启用以下选项，则每生成 32MB 数据将对文件进行 fsync 处理。 为了将文件更多地提交到磁盘并避免大的延迟尖峰，这很有用。
rdb-save-incremental-fsync yes

# Redis LFU eviction (see maxmemory setting) can be tuned. However it is a good
# idea to start with the default settings and only change them after investigating
# how to improve the performances and how the keys LFU change over time, which
# is possible to inspect via the OBJECT FREQ command.
# 可以调整 Redis LFU 逐出（请参阅maxmemory设置）。 但是，最好从默认设置开始，仅在研究了如何提高性能以及LFU 键随时间的变化后才进行更改，可以通过 OBJECT FREQ 命令进行检查。
#
# There are two tunable parameters in the Redis LFU implementation: the
# counter logarithm factor and the counter decay time. It is important to
# understand what the two parameters mean before changing them.
# Redis LFU 实现中有两个可调参数：计数器对数因子和计数器衰减时间。 在更改这两个参数之前，请务必先了解这两个参数的含义。
#
# The LFU counter is just 8 bits per key, it's maximum value is 255, so Redis
# uses a probabilistic increment with logarithmic behavior. Given the value
# of the old counter, when a key is accessed, the counter is incremented in
# this way:
# LFU 计数器每个 key 只有 8 位，最大值是 255，因此 Redis 使用具有对数行为的概率增量。 给定旧计数器的值，当访问一个键时，计数器以这种方式递增：
#
# 1. A random number R between 0 and 1 is extracted.
# 2. A probability P is calculated as 1/(old_value*lfu_log_factor+1).
# 3. The counter is incremented only if R < P.
#
# The default lfu-log-factor is 10. This is a table of how the frequency
# counter changes with a different number of accesses with different
# logarithmic factors:
# 默认的 lfu-log-factor 是 10。这是一个表格，该表格显示了频率计数器如何随着具有不同对数因子的不同访问次数而变化：
#
# +--------+------------+------------+------------+------------+------------+
# | factor | 100 hits   | 1000 hits  | 100K hits  | 1M hits    | 10M hits   |
# +--------+------------+------------+------------+------------+------------+
# | 0      | 104        | 255        | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 1      | 18         | 49         | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 10     | 10         | 18         | 142        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 100    | 8          | 11         | 49         | 143        | 255        |
# +--------+------------+------------+------------+------------+------------+
#
# NOTE: The above table was obtained by running the following commands:
#
#   redis-benchmark -n 1000000 incr foo
#   redis-cli object freq foo
#
# NOTE 2: The counter initial value is 5 in order to give new objects a chance
# to accumulate hits.
#
# The counter decay time is the time, in minutes, that must elapse in order
# for the key counter to be divided by two (or decremented if it has a value
# less <= 10).
#
# The default value for the lfu-decay-time is 1. A Special value of 0 means to
# decay the counter every time it happens to be scanned.
#
# lfu-log-factor 10
# lfu-decay-time 1

########################### ACTIVE DEFRAGMENTATION 主动碎片整理 #######################
#
# WARNING THIS FEATURE IS EXPERIMENTAL. However it was stress tested
# even in production and manually tested by multiple engineers for some
# time.
#
# What is active defragmentation?
# -------------------------------
#
# Active (online) defragmentation allows a Redis server to compact the
# spaces left between small allocations and deallocations of data in memory,
# thus allowing to reclaim back memory.
#
# Fragmentation is a natural process that happens with every allocator (but
# less so with Jemalloc, fortunately) and certain workloads. Normally a server
# restart is needed in order to lower the fragmentation, or at least to flush
# away all the data and create it again. However thanks to this feature
# implemented by Oran Agra for Redis 4.0 this process can happen at runtime
# in an "hot" way, while the server is running.
#
# Basically when the fragmentation is over a certain level (see the
# configuration options below) Redis will start to create new copies of the
# values in contiguous memory regions by exploiting certain specific Jemalloc
# features (in order to understand if an allocation is causing fragmentation
# and to allocate it in a better place), and at the same time, will release the
# old copies of the data. This process, repeated incrementally for all the keys
# will cause the fragmentation to drop back to normal values.
#
# Important things to understand:
#
# 1. This feature is disabled by default, and only works if you compiled Redis
#    to use the copy of Jemalloc we ship with the source code of Redis.
#    This is the default with Linux builds.
#
# 2. You never need to enable this feature if you don't have fragmentation
#    issues.
#
# 3. Once you experience fragmentation, you can enable this feature when
#    needed with the command "CONFIG SET activedefrag yes".
#
# The configuration parameters are able to fine tune the behavior of the
# defragmentation process. If you are not sure about what they mean it is
# a good idea to leave the defaults untouched.

# Enabled active defragmentation	启用主动碎片整理
# activedefrag yes

# Minimum amount of fragmentation waste to start active defrag
# 启动主动碎片整理所需的碎片碎片最少
# active-defrag-ignore-bytes 100mb

# Minimum percentage of fragmentation to start active defrag
# 启动主动碎片整理的最小碎片百分比
# active-defrag-threshold-lower 10

# Maximum percentage of fragmentation at which we use maximum effort
# 我们在最大程度地使用碎片的最大百分比
# active-defrag-threshold-upper 100

# Minimal effort for defrag in CPU percentage	减少CPU碎片整理的工作量
# active-defrag-cycle-min 5

# Maximal effort for defrag in CPU percentage	尽最大努力整理CPU百分比
# active-defrag-cycle-max 75

# Maximum number of set/hash/zset/list fields that will be processed from
# the main dictionary scan
# 主字典扫描将处理的set / hash / zset / list字段的最大数量
# active-defrag-max-scan-fields 1000

```

