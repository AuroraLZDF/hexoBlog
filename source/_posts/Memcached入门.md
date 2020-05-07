---
title: Memcached入门
date: 2020-04-14 13:56:43
categories: [Memcached]
tags: [Memcached]
---

# 简介
Memcached是一个自由开源的，高性能，分布式内存对象缓存系统。

Memcached是以LiveJournal旗下Danga Interactive公司的Brad Fitzpatric为首开发的一款软件。现在已成为mixi、hatena、Facebook、Vox、LiveJournal等众多服务中提高Web应用扩展性的重要因素。

Memcached是一种基于内存的key-value存储，用来存储小块的任意数据（字符串、对象）。这些数据可以是数据库调用、API调用或者是页面渲染的结果。

Memcached简洁而强大。它的简洁设计便于快速开发，减轻开发难度，解决了大数据量缓存的很多问题。它的API兼容大部分流行的开发语言。

本质上，它是一个简洁的key-value存储系统。

一般的使用目的是，通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。

# 特征
memcached作为高速运行的分布式缓存服务器，具有以下的特点。
- 协议简单
- 基于 `libevent` 的事件处理
- 内置内存存储方式
- `memcached` 不互相通信的分布式

# 安装（略）
`membercached` 网上的安装方法很多了，这里不赘述。

# Memcached 连接
我们可以通过 `telnet` 命令并指定主机 `ip` 和端口来连接 `Memcached` 服务。
```bash
telnet HOST PORT    # HOST：ip PORT：端口号
```
## 实例
```bash
$ telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
```
这样，`membercached` 便链接成功了，提示信息显得比较简陋。

# Memcached 存储命令
## Memcached set 命令
Memcached `set` 命令用于将 **value(数据值)** 存储在指定的 **key(键)** 中。
如果 `set` 的 **key** 已经存在，该命令可以更新该 **key** 所对应的原来的数据，也就是实现**更新**的作用。

### 语法
`set` 命令的基本语法格式如下：
```bash
set key flags exptime bytes [noreply] 
value 
```
参数说明如下：
- `key`：键值 **key-value** 结构中的 **key**，用于查找缓存值。
- `flags`：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- `exptime`：在缓存中保存键值对的时间长度（以秒为单位，**0** 表示永远）
- `bytes`：在缓存中存储的字节数
- `noreply`（可选）： 该参数告知服务器不需要返回数据
- `value`：存储的值（**始终位于第二行**）（可直接理解为 **key-value** 结构中的 **value**）

### 实例
```bash
set runoob 0 900 9      # set key flags exptime bytes
memcached               # value
STORED                  # 输出信息

get runoob              # get key
VALUE runoob 0 9        # VALUE key flags bytes
memcached               # value

END                     # 结束标记
```
如果数据设置成功，则输出 **STORED**

输出信息说明：

- `STORED`：保存成功后输出。
- `ERROR`：在保存失败后输出。

## Memcached add 命令
Memcached `add` 命令用于将 **value(数据值)** 存储在指定的 **key(键)** 中。

如果 `add` 的 **key** 已经存在，则**不会更新数据**(过期的 key 会更新)，之前的值将仍然保持相同，并且您将获得响应 **NOT_STORED**。

### 语法
`add` 命令的基本语法格式如下：
```bash
add key flags exptime bytes [noreply]
value
```
参数说明如下：
- `key`：键值 **key-value** 结构中的 **key**，用于查找缓存值。
- `flags`：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- `exptime`：在缓存中保存键值对的时间长度（以秒为单位，**0** 表示永远）
- `bytes`：在缓存中存储的字节数
- `noreply`（可选）： 该参数告知服务器不需要返回数据
- `value`：存储的值（**始终位于第二行**）（可直接理解为 **key-value** 结构中的 **value**）

### 实例
```bash
add new_key 0 900 10        # add key flags exptime bytes
data_value                  # value
STORED                      # 输出信息
get new_key                 # get key
VALUE new_key 0 10          # VALUE key flags bytes
data_value                  # value
END                         # 结束标记
```

## Memcached replace 命令
Memcached `replace` 命令用于替换已存在的 **key(键)** 的 **value(数据值)**。

如果 **key** 不存在，则替换失败，并且您将获得响应 **NOT_STORED**。

### 语法
`replace` 命令的基本语法格式如下：
```bash
replace key flags exptime bytes [noreply]
value
```
参数说明如下：
- `key`：键值 **key-value** 结构中的 **key**，用于查找缓存值。
- `flags`：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- `exptime`：在缓存中保存键值对的时间长度（以秒为单位，**0** 表示永远）
- `bytes`：在缓存中存储的字节数
- `noreply`（可选）： 该参数告知服务器不需要返回数据
- `value`：存储的值（**始终位于第二行**）（可直接理解为 **key-value** 结构中的 **value**）

### 实例
以下实例中我们使用的键位 'mykey' 并存储对应的值 data_value。执行后我们替换相同的 key 的值为 'some_other_value'
```bash
add mykey 0 900 10              # add key flags exptime bytes
data_value                      # value
STORED                          # 输出信息
get mykey                       # get key
VALUE mykey 0 10                # VALUE key flags bytes
data_value                      # value
END                             # 结束标记

replace mykey 0 900 16          # replace key flags exptime bytes
some_other_value                # value
get mykey                       # get key
VALUE mykey 0 16                # VALUE key flags bytes
some_other_value                # value
END                             # 结束标记
```

## Memcached append 命令
Memcached `append` 命令用于向已存在 **key(键)** 的 **value(数据值)** 后面追加数据 。

### 语法
`append` 命令的基本语法格式如下：
```bash
append key flags exptime bytes [noreply]
value
```
参数说明如下：
- `key`：键值 **key-value** 结构中的 **key**，用于查找缓存值。
- `flags`：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- `exptime`：在缓存中保存键值对的时间长度（以秒为单位，**0** 表示永远）
- `bytes`：在缓存中存储的字节数
- `noreply`（可选）： 该参数告知服务器不需要返回数据
- `value`：存储的值（**始终位于第二行**）（可直接理解为 **key-value** 结构中的 **value**）

### 实例
- 首先我们在 Memcached 中存储一个键 runoob，其值为 memcached。
- 然后，我们使用 `get` 命令检索该值。
- 然后，我们使用 `append` 命令在键为 runoob 的值后面追加 "redis"。
- 最后，我们再使用 `get` 命令检索该值。

```bash
set runoob 0 900 9          # set key flags exptime bytes
memcached                   # value
STORED                      # 输出信息

get runoob                  # get key
VALUE runoob 0 9            # VALUE key flags bytes
memcached                   # value
END                         # 结束标记

append runoob 0 900 5       # append key flags exptime bytes
redis                       # value
STORED                      # 输出信息

get runoob                  # get key
VALUE runoob 0 14           #
memcachedredis              # value
END                         # 结束标记
```

## Memcached prepend 命令
Memcached `prepend` 命令用于向已存在 **key(键)** 的 **value(数据值)** 前面追加数据 。

### 语法
`prepend` 命令的基本语法格式如下：
```bash
prepend key flags exptime bytes [noreply]
value
```
参数说明如下：
- `key`：键值 **key-value** 结构中的 **key**，用于查找缓存值。
- `flags`：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- `exptime`：在缓存中保存键值对的时间长度（以秒为单位，**0** 表示永远）
- `bytes`：在缓存中存储的字节数
- `noreply`（可选）： 该参数告知服务器不需要返回数据
- `value`：存储的值（**始终位于第二行**）（可直接理解为 **key-value** 结构中的 **value**）

### 实例
- 首先我们在 Memcached 中存储一个键 runoob，其值为 memcached。
- 然后，我们使用 `get` 命令检索该值。
- 然后，我们使用 `prepend` 命令在键为 runoob 的值前面追加 "redis"。
- 最后，我们再使用 `get` 命令检索该值。

```bash
set runoob 0 900 9              # set key flags exptime bytes
memcached                       # value
STORED                          # 输出信息

get runoob                      # get key
VALUE runoob 0 9                # VALUE key flags bytes
memcached                       # value
END                             # 结束标记

prepend runoob 0 900 5          # prepend key flags exptime bytes
redis                           # value
STORED                          # 输出信息

get runoob                      # get key
VALUE runoob 0 14               # VALUE key flags bytes
redismemcached                  # value
END                             # 结束标记
```

## Memcached CAS 命令
Memcached `CAS`（Check-And-Set 或 Compare-And-Swap） 命令用于执行一个"检查并设置"的操作

它仅在当前客户端最后一次取值后，该 **key** 对应的值没有被其他客户端修改的情况下， 才能够将值写入。

检查是通过 `cas_token` 参数进行的， 这个参数是 `Memcach` 指定给已经存在的元素的一个唯一的 64 位值。

### 语法
`CAS` 命令的基本语法格式如下：
```bash
cas key flags exptime bytes unique_cas_token [noreply]
value
```
参数说明如下：
- `key`：键值 **key-value** 结构中的 **key**，用于查找缓存值。
- `flags`：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- `exptime`：在缓存中保存键值对的时间长度（以秒为单位，**0** 表示永远）
- `bytes`：在缓存中存储的字节数
- `unique_cas_token`: 通过 `gets` 命令获取的一个唯一的 64 位值。
- `noreply`（可选）： 该参数告知服务器不需要返回数据
- `value`：存储的值（**始终位于第二行**）（可直接理解为 **key-value** 结构中的 **value**）

### 实例
要在 Memcached 上使用 `CAS` 命令，你需要从 Memcached 服务商通过 `gets` 命令获取令牌（**token**）。

`gets` 命令的功能类似于基本的 `get` 命令。两个命令之间的差异在于，gets 返回的信息稍微多一些：64 位的整型值非常像名称/值对的 "版本" 标识符。

- 如果没有设置唯一令牌，则 `CAS` 命令执行错误。
- 如果键 `key` 不存在，执行失败。
- 添加键值对。
- 通过 `gets` 命令获取唯一令牌。
- 使用 `cas` 命令更新数据
- 使用 `get` 命令查看数据是否更新

```bash
cas tp 0 900 9
ERROR                   # 缺少 token

cas tp 0 900 9 2
memcached
NOT_FOUND               # 键 tp 不存在

set tp 0 900 9
memcached
STORED

gets tp
VALUE tp 0 9 1
memcached
END

cas tp 0 900 5 1
redis
STORED

get tp
VALUE tp 0 5
redis
END
```
输出信息说明：

- `STORED`：保存成功后输出。
- `ERROR`：保存出错或语法错误。
- `EXISTS`：在最后一次取值后另外一个用户也在更新该数据。
- `NOT_FOUND`：Memcached 服务上不存在该键值。

# Memcached 查找命令
## Memcached get 命令
Memcached `get` 命令获取存储在 **key(键)** 中的 **value(数据值)** ，如果 key 不存在，则返回空。

### 语法
`get` 命令的基本语法格式如下：
```bash
get key
```
多个 key 使用空格隔开，如下:
```bash
get key1 key2 key3
```

### 实例
在以下实例中，我们使用 runoob 作为 key，过期时间设置为 5 秒。
```bash
set runoob 0 5 9        # set key flags exptime bytes
memcached               # value
STORED                  # 输出信息
get runoob              # get key
VALUE runoob 0 9        # VALUE key flags bytes
memcached               # value
END                     # 结束标记
get runoob              # get key   # --这里超过 key 存储时间，返回空
END                     # 结束标记
```

## Memcached gets 命令
Memcached `gets` 命令获取带有 CAS 令牌存 的 **value(数据值)** ，如果 key 不存在，则返回空。

### 语法
`gets` 命令的基本语法格式如下：
```bash
gets key
```
多个 key 使用空格隔开，如下:
```bash
gets key1 key2 key3
```
### 实例
在以下实例中，我们使用 runoob 作为 key，过期时间设置为 900 秒。
```bash
set runoob 0 900 9
memcached
STORED
gets runoob
VALUE runoob 0 9 1
memcached
END
```
在 使用 `gets` 命令的输出结果中，在最后一列的数字 `1` 代表了 key 为 runoob 的 CAS 令牌。


## Memcached delete 命令
Memcached `delete` 命令用于删除已存在的 key(键)。

### 语法
`delete` 命令的基本语法格式如下：
```bash
delete key [noreply]
```
参数说明如下：

- `key`：键值 key-value 结构中的 key，用于查找缓存值。
- `noreply`（可选）： 该参数告知服务器不需要返回数据

### 实例
在以下实例中，我们使用 runoob 作为 key，过期时间设置为 900 秒。之后我们使用 `delete` 命令删除该 key。
```bash
set runoob 0 900 9
memcached
STORED
get runoob
VALUE runoob 0 9
memcached
END
delete runoob
DELETED
get runoob
END
delete runoob
NOT_FOUND
```

### 输出
输出信息说明：

- `DELETED`：删除成功。
- `ERROR`：语法错误或删除失败。
- `NOT_FOUND`：key 不存在。

## Memcached incr 与 decr 命令
Memcached `incr` 与 `decr` 命令用于对已存在的 key(键) 的数字值进行自增或自减操作。

`incr` 与 `decr` 命令操作的数据必须是**十进制的32位无符号整数**。

如果 key 不存在返回 **NOT_FOUND**，如果键的值不为数字，则返回 **CLIENT_ERROR**，其他错误返回 **ERROR**。

### incr 命令
#### 语法
`incr` 命令的基本语法格式如下：
```bash
incr key increment_value    # key 用于查找缓存值；increment_value 增加的数值
```

#### 实例
在以下实例中，我们使用 visitors 作为 key，初始值为 10，之后进行加 5 操作。
```bash
set visitors 0 900 2        
10
STORED
get visitors
VALUE visitors 0 2
10
END
incr visitors 5         # incr visitors 增加 5
15
get visitors
VALUE visitors 0 2
15
END
```

### decr 命令
`decr` 命令的基本语法格式如下：
```bash
decr key decrement_value        # key decrement_value 减少的数值 
```

#### 实例
```bash
set visitors 0 900 2
10
STORED
get visitors
VALUE visitors 0 2
10
END
decr visitors 5             # incr visitors 减少 5
5
get visitors
VALUE visitors 0 1
5
END
```

### 输出
输出信息说明：
- `NOT_FOUND`：key 不存在。
- `CLIENT_ERROR`：自增值不是对象。
- `ERROR`: 其他错误，如语法错误等。

# Memcached 统计命令
## Memcached stats 命令
Memcached `stats` 命令用于返回统计信息例如 PID(进程号)、版本号、连接数等。

### 语法：
`stats` 命令的基本语法格式如下：
```bash
stats
```

### 实例
在以下实例中，我们使用了 `stats` 命令来输出 Memcached 服务信息。
```bash
stats
STAT pid 1                          # pid：	memcache服务器进程ID
STAT uptime 2868                    # uptime：服务器已运行秒数
STAT time 1586919085                # time：服务器当前Unix时间戳
STAT version 1.6.3                  # version：memcache版本
STAT libevent 2.1.11-stable         # 
STAT pointer_size 64                # pointer_size：操作系统指针大小
STAT rusage_user 0.282316           # rusage_user：进程累计用户时间
STAT rusage_system 0.084094         # rusage_system：进程累计系统时间
STAT max_connections 1024           # max_connections: 最大连接数量
STAT curr_connections 2             # curr_connections：当前连接数量
STAT total_connections 3            # total_connections：Memcached运行以来连接总数
STAT rejected_connections 0         # rejected_connections: 拒绝连接数量
STAT connection_structures 3        # connection_structures：Memcached分配的连接结构数量
STAT response_obj_bytes 1168        #
STAT response_obj_total 1           #
STAT response_obj_free 0            #
STAT response_obj_oom 0             #
STAT read_buf_bytes 16384           #
STAT read_buf_bytes_free 0          #
STAT read_buf_oom 0                 #
STAT reserved_fds 20                #
STAT cmd_get 14                     # cmd_get：get命令请求次数
STAT cmd_set 4                      # cmd_set：set命令请求次数
STAT cmd_flush 0                    # cmd_flush：flush命令请求次数
STAT cmd_touch 0                    #
STAT cmd_meta 0                     #
STAT get_hits 7                     # get_hits：get命令命中次数
STAT get_misses 7                   # get_misses：get命令未命中次数
STAT get_expired 0                  #
STAT get_flushed 0                  #
STAT delete_misses 2                # delete_misses：delete命令未命中次数
STAT delete_hits 1                  # delete_hits：delete命令命中次数
STAT incr_misses 0                  # incr_misses：incr命令未命中次数
STAT incr_hits 1                    # incr_hits：incr命令命中次数
STAT decr_misses 0                  # decr_misses：decr命令未命中次数
STAT decr_hits 0                    # decr_hits：decr命令命中次数
STAT cas_misses 0                   # cas_misses：cas命令未命中次数
STAT cas_hits 0                     # cas_hits：cas命令命中次数
STAT cas_badval 0                   # cas_badval：使用擦拭次数
STAT touch_hits 0                   #
STAT touch_misses                   #
STAT auth_cmds 0                    # auth_cmds：认证命令处理的次数
STAT auth_errors 0                  # auth_errors：认证失败数目
STAT bytes_read 320                 # bytes_read：读取总字节数
STAT bytes_written 302              # bytes_written：发送总字节数
STAT limit_maxbytes 67108864        # limit_maxbytes：分配的内存总大小（字节）
STAT accepting_conns 1              # accepting_conns：服务器是否达到过最大连接（0/1）
STAT listen_disabled_num 0          # listen_disabled_num：失效的监听数
STAT time_in_listen_disabled_us 0   #
STAT threads 4                      # threads：当前线程数
STAT conn_yields 0                  # conn_yields：连接操作主动放弃数目
STAT hash_power_level 16            # 
STAT hash_bytes 524288              #
STAT hash_is_expanding 0            #
STAT slab_reassign_rescues 0        #
STAT slab_reassign_chunk_rescues 0  #
STAT slab_reassign_evictions_nomem 0 #
STAT slab_reassign_inline_reclaim 0 #
STAT slab_reassign_busy_items 0     #
STAT slab_reassign_busy_deletes 0   #
STAT slab_reassign_running 0        #
STAT slabs_moved 0                  #
STAT lru_crawler_running 0          #
STAT lru_crawler_starts 2550        #
STAT lru_maintainer_juggles 5696    #
STAT malloc_fails 0                 #
STAT log_worker_dropped 0           #
STAT log_worker_written 0           #
STAT log_watcher_skipped 0          #
STAT log_watcher_sent 0             #
STAT bytes 65                       # bytes：当前存储占用的字节数
STAT curr_items 1                   # curr_items：当前存储的数据总数
STAT total_items 4                  # total_items：启动以来存储的数据总数
STAT slab_global_page_pool 0        #
STAT expired_unfetched 1            #
STAT evicted_unfetched 0            #
STAT evicted_active 0               #
STAT evictions 0                    # evictions：LRU释放的对象数目
STAT reclaimed 2                    # reclaimed：已过期的数据条目来存储新数据的数目
STAT crawler_reclaimed 0            #
STAT crawler_items_checked 2        #
STAT lrutail_reflocked 8            #
STAT moves_to_cold 7                #
STAT moves_to_warm 4                #
STAT moves_within_lru 0             #
STAT direct_reclaims 0              #
STAT lru_bumps_dropped 0            #
END
```

## Memcached stats items 命令
Memcached `stats items` 命令用于显示各个 slab 中 item 的数目和存储时长(最后一次访问距离现在的秒数)。

### 语法
`stats items` 命令的基本语法格式如下：
```bash
stats items
```

### 实例
```bash
stats items
STAT items:1:number 1
STAT items:1:age 7
STAT items:1:evicted 0
STAT items:1:evicted_nonzero 0
STAT items:1:evicted_time 0
STAT items:1:outofmemory 0
STAT items:1:tailrepairs 0
STAT items:1:reclaimed 0
STAT items:1:expired_unfetched 0
STAT items:1:evicted_unfetched 0
END
```

## Memcached stats slabs 命令
Memcached `stats slabs` 命令用于显示各个slab的信息，包括chunk的大小、数目、使用情况等。

### 语法
`stats slabs` 命令的基本语法格式如下：
```bash
stats slabs
```

### 实例
```bash
stats slabs
STAT 1:chunk_size 96
STAT 1:chunks_per_page 10922
STAT 1:total_pages 1
STAT 1:total_chunks 10922
STAT 1:used_chunks 0
STAT 1:free_chunks 10922
STAT 1:free_chunks_end 0
STAT 1:get_hits 7
STAT 1:cmd_set 4
STAT 1:delete_hits 1
STAT 1:incr_hits 1
STAT 1:decr_hits 0
STAT 1:cas_hits 0
STAT 1:cas_badval 0
STAT 1:touch_hits 0
STAT active_slabs 1
STAT total_malloced 1048576
END
```

## Memcached stats sizes 命令
Memcached `stats sizes` 命令用于显示所有item的大小和个数。

该信息返回两列，第一列是 item 的大小，第二列是 item 的个数。

### 语法：
`stats sizes` 命令的基本语法格式如下：
```bash
stats sizes
```

### 实例
```bash
stats sizes
STAT 96 1
END
```

## Memcached flush_all 命令
Memcached `flush_all` 命令用于清理缓存中的所有 **key=>value(键=>值)** 对。

该命令提供了一个可选参数 time，用于在制定的时间后执行清理缓存操作。

### 语法
`flush_all` 命令的基本语法格式如下：
```bash
flush_all [time] [noreply]
```

### 实例
清理缓存:
```bash
set runoob 0 900 9
memcached
STORED
get runoob
VALUE runoob 0 9
memcached
END
flush_all               # 清理缓存
OK
get runoob
END
```

# Memcached 实例
## PHP 连接 Memcached 服务
### PHP Memcache 扩展安装
PHP Memcache 扩展包下载地址：[http://pecl.php.net/package/memcache](http://pecl.php.net/package/memcache)

```bash
$ wget http://pecl.php.net/get/memcache-2.2.7.tgz               
$ tar -zxvf memcache-2.2.7.tgz
$ cd memcache-2.2.7
$ /usr/local/php/bin/phpize
$ ./configure --with-php-config=/usr/local/php/bin/php-config
$ make && make install
```
**注意**：/usr/local/php/ 为php的安装路径，需要根据你安装的实际目录调整。

安装成功后会显示你的 `memcache.so` 扩展的位置:
```bash
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
```
然后在 `php.ini` 文件中，将 Memcache 添加到扩展项中即可：
```php
[Memcache]
extension_dir = "/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/"
extension = memcache.so
```
添加完成后，重启 php服务：
```bash
service php restart
```
输出 `phpinfo()` 信息，看到 `memcache` 扩展信息，即表示安装成功。

### PHP 连接 Memcached
```php
<?php
$memcache = new Memcache;             //创建一个memcache对象
$memcache->connect('localhost', 11211) or die ("Could not connect"); //连接Memcached服务器
$memcache->set('key', 'test');        //设置一个变量到内存中，名称是key 值是test
$get_value = $memcache->get('key');   //从内存中取出key的值
echo $get_value;
?>
```








