---
title: Redis入门
date: 2020-04-16 09:34:42
categories: [Redis]
tags: [Redis]
---

# Redis简介
`Redis` 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 **字符串（strings）**， **散列（hashes）**， **列表（lists）**， **集合（sets）**， **有序集合（sorted sets）** 与范围查询， **bitmaps**， **hyperloglogs** 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 **复制（replication）**，**LUA脚本（Lua scripting）**， **LRU驱动事件（LRU eviction）**，**事务（transactions）** 和不同级别的 **磁盘持久化（persistence）**， 并通过 **Redis哨兵（Sentinel）***和**自动 分区**（Cluster）提供高可用性（high availability）。


`Redis` 与其他 key - value 缓存产品有以下三个特点：
- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。

# Redis 数据类型

`Redis` 支持五种数据类型：**string（字符串）**，**hash（哈希）**，**list（列表）**，**set（集合）**及zset(sorted set：**有序集合**)。

## String（字符串）
string 是 `Redis` 最基本的类型，你可以理解成与 `Memcached` 一模一样的类型，一个 key 对应一个 value。

string 类型是二进制安全的。意思是 `Redis` 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。

string 类型是 `Redis` 最基本的数据类型，string 类型的值最大能存储 **512MB**。

### 实例
```bash
127.0.0.1:6379> set ruoob "Hello"
OK
127.0.0.1:6379> get ruoob
"Hello"
```
**注意**：一个键最大能存储 512MB。

## Hash（哈希）
`Redis` hash 是一个键值(key=>value)对集合。

`Redis` hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

### 实例
```bash
127.0.0.1:6379> hmset runoob field1 "hello" field2 "world"
OK
127.0.0.1:6379> HGET runoob field1
"Hello"
127.0.0.1:6379> HGET runoob field2
"World"
```
实例中我们使用了 `Redis` `HMSET`, `HGET` 命令，`HMSET` 设置了两个 **field=>value** 对, `HGET` 获取对应 **field** 对应的 **value**。

每个 hash 可以存储 `2^32 - 1` 键值对（40多亿）。

## List（列表）
`Redis` 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。
### 实例
```bash
127.0.0.1:6379> del runoob                # del key 删除一个 key
127.0.0.1:6379> lpush runoob redis        # lpush 往列表中插入一个值
(integer) 1
127.0.0.1:6379> lpush runoob mongodb
(integer) 2
127.0.0.1:6379> lpush runoob rabitmq
(integer) 3
127.0.0.1:6379> lpush runoob git
(integer) 4
127.0.0.1:6379> lpush runoob svn
(integer) 5
127.0.0.1:6379> lrange runoob 0 10        # lrange 获取列表 0-10 范围内的元素
1) "svn"
2) "git"
3) "rabitmq"
4) "mongodb"
5) "redis"
127.0.0.1:6379>
```
列表最多可存储 `2^32 - 1` 元素 (4294967295, 每个列表可存储40多亿)。

## Set（集合）
`Redis` 的 Set 是 string 类型的**无序集合**。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

### sadd 命令
添加一个 string 元素到 key 对应的 set 集合中，成功返回 1，如果元素已经在集合中返回 0。

### smembers 命令
读取一个 key 对应的 set 集合。

### 实例
```bash
127.0.0.1:6379> del runoob
127.0.0.1:6379> sadd runoob redis
(integer) 1
127.0.0.1:6379> sadd runoob mongodb
(integer) 1
127.0.0.1:6379> sadd runoob rabitmq
(integer) 1
127.0.0.1:6379> sadd runoob rabitmq
(integer) 0
127.0.0.1:6379> smembers runoob
1) "redis"
2) "rabitmq"
3) "mongodb"
```
注意：以上实例中 `rabitmq` 添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略。

集合中最大的成员数为 `2^32 - 1`(4294967295, 每个集合可存储40多亿个成员)。

## zset(sorted set：有序集合)
`Redis` zset 和 set 一样也是string类型元素的集合,且**不允许重复的成员**。
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

### zadd 命令
添加元素到集合，元素在集合中存在则更新对应score
```bash
zadd key score member 
```

### 实例
```bash
127.0.0.1:6379> del runoob
127.0.0.1:6379> zadd runoob 0 redis
(integer) 1
127.0.0.1:6379> zadd runoob 0 mongodb
(integer) 1
127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 1
127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 0
127.0.0.1:6379> > ZRANGEBYSCORE runoob 0 1000
1) "mongodb"
2) "rabitmq"
3) "redis"
```

# Redis 命令
## Redis 登陆命令
### 语法
Redis 客户端的基本语法为：
```bash
$ redis-cli
```

### 实例
```bash
$ redis-cli
127.0.0.1:6379>
127.0.0.1:6379> PING
PONG
```
在以上实例中我们连接到本地的 redis 服务并执行 **PING** 命令，该命令用于检测 redis 服务是否启动。

## 在远程服务上执行命令
### 语法
```bash
$ redis-cli -h host -p port -a password
```

### 实例
```bash
$redis-cli -h 127.0.0.1 -p 6379 -a "password"
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379>
127.0.0.1:6379> PING
PONG
```

## Redis 键(key)
`Redis` 键命令用于管理 redis 的键。

### 语法
`Redis` 键命令的基本语法如下：
```bash
127.0.0.1:6379> COMMAND KEY_NAME
```
### 实例
```bash
127.0.0.1:6379> SET runoobkey redis
OK
127.0.0.1:6379> DEL runoobkey
(integer) 1
```
在以上实例中 `DEL` 是一个命令， `runoobkey` 是一个键。 如果键被删除成功，命令执行后输出 **(integer) 1**，否则将输出 **(integer) 0**

## Redis keys 命令参考
[Redis 命令参考——Key（键）](http://doc.redisfans.com/key/index.html)

## Redis 字符串(String)
[Redis 命令参考——字符串(String)](http://doc.redisfans.com/string/index.html)

## Redis 哈希(Hash)
[Redis 命令参考——哈希(Hash)](http://doc.redisfans.com/hash/index.html)

## Redis 列表(List)
[Redis 命令参考——列表(List)](http://doc.redisfans.com/list/index.html)

## Redis 集合(Set)
[Redis 命令参考——集合(Set)](http://doc.redisfans.com/set/index.html)

## Redis 有序集合(sorted set)
[Redis 命令参考——有序集合(sorted set)](http://doc.redisfans.com/sorted_set/index.html)

## Redis 发布订阅
[Redis 命令参考——Pub/Sub（发布/订阅）](http://doc.redisfans.com/pub_sub/index.html)

## Redis 事务
[Redis 命令参考——Transaction（事务）](http://doc.redisfans.com/transaction/index.html)

## Redis 脚本
[Redis 命令参考——Script（脚本）](http://doc.redisfans.com/script/index.html)

## Redis 连接
[Redis 命令参考——Connection（连接）](http://doc.redisfans.com/connection/index.html)

## Redis 服务器
[Redis 命令参考——Server（服务器）](http://doc.redisfans.com/server/index.html)

# Redis 高级教程
## Redis 数据备份与恢复
### 备份数据
Redis `SAVE` 命令用于创建当前数据库的备份。
```bash
127.0.0.1:6379> SAVE 
OK
```
该命令将在 redis 安装目录中创建 **dump.rdb** 文件。

### 恢复数据
如果需要恢复数据，只需将备份文件 (**dump.rdb**) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：

redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"
以上命令 CONFIG GET dir 输出的 redis 安装目录为 /usr/local/redis/bin。

### Bgsave
创建 redis 备份文件也可以使用命令 **BGSAVE**，该命令在后台执行。
```bash
127.0.0.1:6379> BGSAVE
Background saving started   
```

## Redis 安全
我们可以通过 redis 的配置文件设置密码参数，这样客户端连接到 redis 服务就需要密码验证，这样可以让你的 redis 服务更安全。

通过以下命令查看是否设置了密码验证：
```bash
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
```
默认情况下 requirepass 参数是空的，这就意味着你无需通过密码验证就可以连接到 redis 服务。

你可以通过以下命令来修改该参数：
```bash
127.0.0.1:6379> CONFIG set requirepass "123456"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "123456"
```
设置密码后，客户端连接 redis 服务就需要密码验证，否则无法执行命令。
```bash
$ redis-cli 
127.0.0.1:6379> get runoob
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> get runoob
"requirepass test"
```

## Redis 性能测试

Redis 性能测试是通过同时执行多个命令实现的。

### 语法
redis 性能测试的基本命令如下：
```bash
redis-benchmark [option] [option value]
```
**注意**：该命令是在 redis 的目录下执行的，而不是 redis 客户端的内部指令。

### 实例
以下实例同时执行 10000 个请求来检测性能：
```bash
$ redis-benchmark -n 10000 -q
PING_INLINE: 77519.38 requests per second
PING_BULK: 69444.45 requests per second
SET: 79365.08 requests per second
GET: 81967.21 requests per second
INCR: 78740.16 requests per second
LPUSH: 86956.52 requests per second
RPUSH: 81300.81 requests per second
LPOP: 84033.61 requests per second
RPOP: 81967.21 requests per second
SADD: 88495.58 requests per second
HSET: 90090.09 requests per second
SPOP: 92592.59 requests per second
LPUSH (needed to benchmark LRANGE): 91743.12 requests per second
LRANGE_100 (first 100 elements): 91743.12 requests per second
LRANGE_300 (first 300 elements): 84033.61 requests per second
LRANGE_500 (first 450 elements): 90090.09 requests per second
LRANGE_600 (first 600 elements): 87719.30 requests per second
MSET (10 keys): 90909.09 requests per second
```
redis 性能测试工具可选参数如下所示：

| 序号	| 选项	| 描述	| 默认值 |
| --- | --- | --- | ---|
| 1	 | -h | 指定服务器主机名  |	127.0.0.1 |
| 2	 | -p | 指定服务器端口  |	6379 |
| 3	 | -s | 指定服务器  | socket	|
| 4	 | -c | 指定并发连接数  |	50 |
| 5	 | -n | 指定请求数  |	10000 |
| 6	 | -d | 以字节的形式指定 SET/GET 值的数据大小  |	2 |
| 7	 | -k | 1=keep alive 0=reconnect  |	1 |
| 8	 | -r | SET/GET/INCR 使用随机 key, SADD 使用随机值  |	|
| 9	 | -P | 通过管道传输 <numreq> 请求  |	1 |
| 10 | -q | 强制退出 redis。仅显示 query/sec 值	  | |
| 11 | --csv | 以 CSV 格式输出  |	|
| 12 | -l | 生成循环，永久执行测试  | |
| 13 | -t | 仅运行以逗号分隔的测试命令列表。  |	 |
| 14 | -I | Idle 模式。仅打开 N 个 idle 连接并等待。  |	|

### 实例
以下实例我们使用了多个参数来测试 redis 性能：
```bash
$ redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 10000 -q
SET: 71942.45 requests per second
LPUSH: 73529.41 requests per second
```
以上实例中主机为 127.0.0.1，端口号为 6379，执行的命令为 set,lpush，请求数为 10000，通过 -q 参数让结果只显示每秒执行的请求数。

## Redis 客户端连接
Redis 通过监听一个 TCP 端口或者 Unix socket 的方式来接收来自客户端的连接，当一个连接建立后，Redis 内部会进行以下一些操作：

首先，客户端 socket 会被设置为非阻塞模式，因为 Redis 在网络事件处理上采用的是非阻塞多路复用模型。
然后为这个 socket 设置 TCP_NODELAY 属性，禁用 Nagle 算法
然后创建一个可读的文件事件用于监听这个客户端 socket 的数据发送

### 最大连接数
在 Redis2.4 中，最大连接数是被直接硬编码在代码里面的，而在2.6版本中这个值变成可配置的。

`maxclients` 的默认值是 **10000**，你也可以在 redis.conf 中对这个值进行修改。
```bash
config get maxclients

1) "maxclients"
2) "10000"
```
### 实例
以下实例我们在服务启动时设置最大连接数为 100000：
```bash
redis-server --maxclients 100000
```

## Redis 管道技术
Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。这意味着通常情况下一个请求会遵循以下步骤：

客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
服务端处理命令，并将结果返回给客户端。

**Redis 管道技术**可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

### 实例
查看 redis 管道，只需要启动 redis 实例并输入以下命令：
```bash
$(echo -en "PING\r\n SET runoobkey redis\r\nGET runoobkey\r\nINCR visitor\r\nINCR visitor\r\nINCR visitor\r\n"; sleep 10) | nc localhost 6379

+PONG
+OK
$5
redis
:1
:2
:3
```
以上实例中我们通过使用 PING 命令查看redis服务是否可用， 之后我们设置了 runoobkey 的值为 redis，然后我们获取 runoobkey 的值并使得 visitor 自增 3 次。

在返回的结果中我们可以看到这些命令一次性向 redis 服务提交，并最终一次性读取所有服务端的响应

### 管道技术的优势
管道技术最显著的优势是提高了 redis 服务的性能。



## Redis 分区

分区是分割数据到多个Redis实例的处理过程，因此每个实例只保存key的一个子集。

### 分区的优势

- 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
- 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

### 分区的不足

redis的一些特性在分区方面表现的不是很好：

- 涉及多个key的操作通常是不被支持的。举例来说，当两个set映射到不同的redis实例上时，你就不能对这两个set执行交集操作。
- 涉及多个key的redis事务不能使用。
- 当使用分区时，数据处理较为复杂，比如你需要处理多个rdb/aof文件，并且从多个实例和主机备份持久化文件。
- 增加或删除容量也比较复杂。redis集群大多数支持在运行时增加、删除节点的透明数据平衡的能力，但是类似于客户端分区、代理等其他系统则不支持这项特性。然而，一种叫做presharding的技术对此是有帮助的。

### 分区类型

Redis 有两种类型分区。 假设有4个Redis实例 R0，R1，R2，R3，和类似user:1，user:2这样的表示用户的多个key，对既定的key有多种不同方式来选择这个key存放在哪个实例中。也就是说，有不同的系统来映射某个key到某个Redis服务。

### 范围分区

最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例。

比如，ID从0到10000的用户会保存到实例R0，ID从10001到 20000的用户会保存到R1，以此类推。

这种方式是可行的，并且在实际中使用，不足就是要有一个区间范围到实例的映射表。这个表要被管理，同时还需要各 种对象的映射表，通常对Redis来说并非是好的方法。

### 哈希分区

另外一种分区方法是hash分区。这对任何key都适用，也无需是object_name:这种形式，像下面描述的一样简单：

- 用一个hash函数将key转换为一个数字，比如使用crc32 hash函数。对key foobar执行crc32(foobar)会输出类似93024922的整数。
- 对这个整数取模，将其转化为0-3之间的数字，就可以将这个整数映射到4个Redis实例中的一个了。93024922 % 4 = 2，就是说key foobar应该被存到R2实例中。注意：取模操作是取除的余数，通常在多种编程语言中用%操作符实现。

## PHP 使用 Redis

### 安装

开始在 PHP 中使用 Redis 前， 我们需要确保已经安装了 redis 服务及 PHP redis 驱动，且你的机器上能正常使用 PHP。 接下来让我们安装 PHP redis 驱动：下载地址为:[**https://github.com/phpredis/phpredis/releases**](https://github.com/phpredis/phpredis/releases)。

### PHP安装redis扩展

以下操作需要在下载的 phpredis 目录中完成：

```bash
$ wget https://github.com/phpredis/phpredis/archive/3.1.4.tar.gz
$ cd phpredis-3.1.4                      # 进入 phpredis 目录
$ /usr/local/php/bin/phpize              # php安装后的路径
$ ./configure --with-php-config=/usr/local/php/bin/php-config
$ make && make install
```

### 修改php.ini文件

```bash
vi /usr/local/php/lib/php.ini
```

增加如下内容:

```bash
extension_dir = "/usr/local/php/lib/php/extensions/no-debug-zts-20090626"

extension=redis.so
```

安装完成后重启php-fpm 或 apache。查看phpinfo信息，就能看到redis扩展。

![PHP 使用 Redis](https://www.runoob.com/wp-content/uploads/2014/11/14022020088882.jpg)

### 连接到 redis 服务

```php
<?php
    //连接本地的 Redis 服务
   $redis = new Redis();
   $redis->connect('127.0.0.1', 6379);
   echo "Connection to server successfully";
         //查看服务是否运行
   echo "Server is running: " . $redis->ping();
?>
```

执行脚本，输出结果为：

```bash
Connection to server sucessfully
Server is running: PONG
```

### Redis PHP String(字符串) 实例

```php
<?php
   //连接本地的 Redis 服务
   $redis = new Redis();
   $redis->connect('127.0.0.1', 6379);
   echo "Connection to server successfully";
   //设置 redis 字符串数据
   $redis->set("tutorial-name", "Redis tutorial");
   // 获取存储的数据并输出
   echo "Stored string in redis:: " . $redis->get("tutorial-name");
?>
```

执行脚本，输出结果为：

```bash
Connection to server sucessfully
Stored string in redis:: Redis tutorial
```

------

## Redis PHP List(列表) 实例

```php
<?php
   //连接本地的 Redis 服务
   $redis = new Redis();
   $redis->connect('127.0.0.1', 6379);
   echo "Connection to server successfully";
   //存储数据到列表中
   $redis->lpush("tutorial-list", "Redis");
   $redis->lpush("tutorial-list", "Mongodb");
   $redis->lpush("tutorial-list", "Mysql");
   // 获取存储的数据并输出
   $arList = $redis->lrange("tutorial-list", 0 ,5);
   echo "Stored string in redis";
   print_r($arList);
?>
```

执行脚本，输出结果为：

```bash
Connection to server sucessfully
Stored string in redis
Mysql
Mongodb
Redis
```

------

## Redis PHP Keys 实例

```php
<?php
   //连接本地的 Redis 服务
   $redis = new Redis();
   $redis->connect('127.0.0.1', 6379);
   echo "Connection to server successfully";
   // 获取数据并输出
   $arList = $redis->keys("*");
   echo "Stored keys in redis:: ";
   print_r($arList);
?>
```

执行脚本，输出结果为：

```bash
Connection to server sucessfully
Stored string in redis::
tutorial-name
tutorial-list
```















