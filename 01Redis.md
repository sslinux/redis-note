# 01 redis

The word Redis means(REmote DIctionary Server),远程字典服务器；

Key-Value存储 数据结构存储；

Runs entirely in memory。

    All data is kept in memory；
    Quick data access since it is maintained in memory。
    Data can be backed up to disk periodically.
    Single threaded server.(单线程服务)

Extensible via Lua scripts。
Able to replicate data between servers。
Clustering also available。


Redis is an open source (BSD licensed), in-memory `data structure store`, used as a database, cache and message broker. It supports data structures such as `strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes with radius queries and streams.` Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

Redis:

    KV cache and store
        in-memory
        持久化
        主从(借助于sentinel实现一定意义上的HA)
        Clustering(分布式)，完成数据集存储的分布式；

    数据结构服务器：

        string，list，Hash(关联数组)，Set，Sorted Set，Bitmaps，hyperloglogs。

    
```bash
Redis is an in-memory but persistent on disk database.
1 Million small Key --> String value pairs use ~ 100MB of memory.
Single threaded - but CPU should not be the bottleneck.
    Average Linux system can deliver even 500K requests per second. 常见每秒能承载50万的并发访问。
Limit is likely the available memory in your system.
    max. 232 keys.
```

* QPS ： Query Per Second，每秒查询次数；
* TPS ： Transaction Per Second，每秒处理多少个事务；


### Persistence：持久化；

* Snapshotting：快照方式；
    Data is asynchronously(异步传输) transferred from memory to disk。
* AOF (Append Only File)
    Each modifying operation is written to a file.
    Can recreate data store by replaying operations.
    Without interrupting service, will rebuild AOF as the shortest sequence of commands needed to rebuild the current database in memory.

### Replication：

* Redis supports master-slave replication；
* Master-slave replication can be chained。
* Be careful：
  - Slaves are writeable
  - Potential for data inconsistency。
* Fully compatible with Pub/Sub features。

### Differences to Memcached：

* Memcached is a "distributed memory object caching system"
* Redis persists data to disk eventually.
* Memcached is an LRU cache.
* Redis has different data types and more features.
* Memcached is multithreaded.
* Similar speed.

### Redis的优势：

* 丰富的(资料形态)操作： Hashs，Lists，Sets，Sorted Sets，HyperLogLog等；
* 内建replication及cluster。
* 就地更新(in-place update)操作，在内存中更新。
* 支持持久化(磁盘)
  - 避免雪崩效应；

### Memcached的优势：
* 多线程：
  - 善用多核CPU；
  - 更少的阻塞操作；
* 更少的内存开销；
* 更少的内存分配压力；
* 可能有更少的内存碎片；


### Redis 3.0

2015年4月1日正式推出：

* Redis Cluster
* 新的 "embedded string"
* LRU演算法的推进：
  - 预设随机取5个样本，插入并排序至一个pool，移除最佳者，如此反复，直到内存用量小于maxmemory的设定；
  - 样本5比先前的3多。
  - 从局部最优倾向全局最优；


### 存储系统分类：

* RDBMS： Oracle，DB2，PostgreSQL，MySQL，SQL Server, ...
* NoSQL:  Cassandra, Hbase,Memcached, MongoDB,Redis, ...
* NewSQL: Aerospike,FoundationDB,RethinkDB, ... 在设计上就支持分布式的关系型数据库；


* NoSQL的四种技术流派：
  - Key-value NoSQL： Memcached， Redis，...    KV存储；
  - Column Family NoSQL: Cassandra, Hbase, ... 列族存储；
  - Documentation NoSQL: MongoDB,...           文档存储；
  - Graph NoSQL: Neo4j,...                     图式存储；


Redis 属于 KV NoSQL；


## Redis的组件：

    redis.io 意大利人研发；

* redis-server
* redis-cli ： command line interface；
* redis-benchmark： Benchmarking utility，性能压力测试工具；
* redis-check-dump & redis-check-aof ： 
    Corrupted RDB/AOF file utilities。 检查工具，检查持久化后的文件完整性；


```bash
[root@56ad9677ea63 /]# yum info redis
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * epel: mirrors.aliyun.com
 * extras: mirror.jdcloud.com
 * updates: mirrors.shu.edu.cn
Installed Packages
Name        : redis
Arch        : x86_64
Version     : 3.2.12
Release     : 2.el7
Size        : 1.4 M
Repo        : installed
From repo   : epel
Summary     : A persistent key-value database
URL         : http://redis.io
License     : BSD
```

```bash
[root@56ad9677ea63 /]# rpm -ql redis
/etc/logrotate.d/redis
/etc/redis-sentinel.conf
/etc/redis.conf
/etc/systemd/system/redis-sentinel.service.d
/etc/systemd/system/redis-sentinel.service.d/limit.conf
/etc/systemd/system/redis.service.d
/etc/systemd/system/redis.service.d/limit.conf
/usr/bin/redis-benchmark
/usr/bin/redis-check-aof
/usr/bin/redis-check-rdb
/usr/bin/redis-cli
/usr/bin/redis-sentinel
/usr/bin/redis-server
/usr/lib/systemd/system/redis-sentinel.service
/usr/lib/systemd/system/redis.service
/usr/libexec/redis-shutdown
/usr/share/doc/redis-3.2.12
/usr/share/doc/redis-3.2.12/00-RELEASENOTES
/usr/share/doc/redis-3.2.12/BUGS
/usr/share/doc/redis-3.2.12/CONTRIBUTING
/usr/share/doc/redis-3.2.12/MANIFESTO
/usr/share/doc/redis-3.2.12/README.md
/usr/share/licenses/redis-3.2.12
/usr/share/licenses/redis-3.2.12/COPYING
/usr/share/man/man1/redis-benchmark.1.gz
/usr/share/man/man1/redis-check-aof.1.gz
/usr/share/man/man1/redis-check-rdb.1.gz
/usr/share/man/man1/redis-cli.1.gz
/usr/share/man/man1/redis-sentinel.1.gz
/usr/share/man/man1/redis-server.1.gz
/usr/share/man/man5/redis-sentinel.conf.5.gz
/usr/share/man/man5/redis.conf.5.gz
/var/lib/redis
/var/log/redis
/var/run/redis
```

Redis守护进程：
    监听端口： 6379/tcp

Keys: 

* Arbitrary ASCII strings

    Define same format convention and adhere to it

    Key length matters.

* Multiple name spaces are available.

        Separeate DBs indexed by an integer value.
            SELECT command
            Multiples DBs vs. Single DB + key prefixes

* Keys can expire automatically.

在redis中，与其叫库，不如叫名称空间，在同一个名称空间中，键名是不可重复的。


### 常见数据类型：

    Strings:
        SET VALUE [EX #] [NX|XX]
        GET
        INCR
        DECR
        EXIST
        
    Lists:
        RPUSH
        LPUSH
        RPOP
        LPOP
        LINDEX
        LSET

    Sets:
        SADD
        SINTER
        SUNION
        SPOP
        SISMEMBER

    Sorted-Sets:
        ZADD
        ZSCORE
        ZRANGE
        ZCARD
        ZRANK

    Hashes:
        HSET
        HSETNX
        HGET
        HKEYS
        HVALS
        HDEL

    Bimaps,HyperLogLog

认证实现方法：

    (1) redis.conf
        requirepass PASSWORD
    (2) redis-cli:
        AUTH PASSWORD


清空数据库：

    FLUSHDB： 清空当前库；
    FLUSHALL： 清空所有库；

事务：

    一组相关的操作是原子性的；

    通过MULTI,EXEC,WATCH等命令实现事务功能；将一个或多个命令归并为一个操作提请服务器按顺序执行的机制；不支持回滚操作；

    MULTI： 启动一个事务；
    EXEC： 执行事务；
        一次性将事务中的所有操作执行完成后返回给客户端；

    WATCH： 乐观锁，在EXEC命令执行之前，用于监视指定数量键；如果监视中的某任意键数据被修改，则服务器拒绝执行事务；


```bash
127.0.0.1:6379> WATCH ip
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET ip 192.168.1.1
QUEUED
127.0.0.1:6379> GET ip
QUEUED
127.0.0.1:6379> SET port 8080
QUEUED
127.0.0.1:6379> GET port
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) "192.168.1.1"
3) OK
4) "8080"

```

Connection相关的命令：

HELP @connection:

    AUTH
    PING
    ECHO
    QUIT
    SELECT 

Server相关的命令：

```bash
127.0.0.1:6379> HELP @SERVER

  BGREWRITEAOF -
  summary: Asynchronously rewrite the append-only file
  since: 1.0.0

  BGSAVE -
  summary: Asynchronously save the dataset to disk
  since: 1.0.0

  CLIENT GETNAME - # 获取当前客户端的连接名；
  summary: Get the current connection name
  since: 2.6.9

# 关闭client；
  CLIENT KILL [ip:port] [ID client-id] [TYPE normal|master|slave|pubsub] [ADDR ip:port] [SKIPME yes/no]
  summary: Kill the connection of a client
  since: 2.4.0

  CLIENT LIST -
  summary: Get the list of client connections
  since: 2.4.0

  CLIENT PAUSE timeout
  summary: Stop processing commands from clients for some time
  since: 2.9.50

  CLIENT REPLY ON|OFF|SKIP
  summary: Instruct the server whether to reply to commands
  since: 3.2

# 设置当前连接的名称；
  CLIENT SETNAME connection-name
  summary: Set the current connection name
  since: 2.6.9

  COMMAND -
  summary: Get array of Redis command details
  since: 2.8.13

  COMMAND COUNT -
  summary: Get total number of Redis commands
  since: 2.8.13

  COMMAND GETKEYS -
  summary: Extract keys given a full Redis command
  since: 2.8.13

  COMMAND INFO command-name [command-name ...]
  summary: Get array of specific Redis command details
  since: 2.8.13
# 获取指定参数的值；
  CONFIG GET parameter
  summary: Get the value of a configuration parameter
  since: 2.0.0
# 重置INFO中所统计的数据；
  CONFIG RESETSTAT -
  summary: Reset the stats returned by INFO
  since: 2.0.0
# 修改配置，会同步到配置文件中；
  CONFIG REWRITE -
  summary: Rewrite the configuration file with the in memory configuration
  since: 2.8.0
# 运行时修改配置，不会同步到配置文件中；
  CONFIG SET parameter value
  summary: Set a configuration parameter to the given value
  since: 2.0.0
# 显示当前选中的数据库中的所有keys的数量；
  DBSIZE -
  summary: Return the number of keys in the selected database
  since: 1.0.0

  DEBUG OBJECT key
  summary: Get debugging information about a key
  since: 1.0.0

  DEBUG SEGFAULT -
  summary: Make the server crash
  since: 1.0.0

  FLUSHALL -
  summary: Remove all keys from all databases
  since: 1.0.0

  FLUSHDB -
  summary: Remove all keys from the current database
  since: 1.0.0

  INFO [section]
  summary: Get information and statistics about the server
  since: 1.0.0
# 获取最近一次save执行的时间戳；
  LASTSAVE -
  summary: Get the UNIX time stamp of the last successful save to disk
  since: 1.0.0
# 实时监控所有的接受到的请求；
  MONITOR -
  summary: Listen for all requests received by the server in real time
  since: 1.0.0

  ROLE -
  summary: Return the role of the instance in the context of replication
  since: 2.8.12

  SAVE -
  summary: Synchronously save the dataset to disk
  since: 1.0.0
# 将所有数据同步到磁盘后，安全的退出；
  SHUTDOWN [NOSAVE|SAVE]
  summary: Synchronously save the dataset to disk and then shut down the server
  since: 1.0.0

  SLAVEOF host port
  summary: Make the server a slave of another instance, or promote it as master
  since: 1.0.0
# 显示慢查询日志；
  SLOWLOG subcommand [argument]
  summary: Manages the Redis slow queries log
  since: 2.2.12

  SYNC -
  summary: Internal command used for replication
  since: 1.0.0

  TIME -
  summary: Return the current server time
  since: 2.6.0

```

发布与订阅功能(publish/subscribe):

    频道： 消息队列；

    SUBSCRIBE： 订阅一个或多个队列；

```bash
127.0.0.1:6379> HELP SUBSCRIBE
# 订阅频道：
  SUBSCRIBE channel [channel ...]
  summary: Listen for messages published to the given channels
  since: 2.0.0
  group: pubsub

127.0.0.1:6379> HELP @PUBSUB
# 使用模式匹配的方式订阅频道： 模式订阅；
  PSUBSCRIBE pattern [pattern ...]
  summary: Listen for messages published to channels matching the given patterns
  since: 2.0.0

# 向某个频道发送消息：
  PUBLISH channel message
  summary: Post a message to a channel
  since: 2.0.0

  PUBSUB subcommand [argument [argument ...]]
  summary: Inspect the state of the Pub/Sub subsystem
  since: 2.8.0

  PUNSUBSCRIBE [pattern [pattern ...]]
  summary: Stop listening for messages posted to channels matching the given patterns
  since: 2.0.0

  SUBSCRIBE channel [channel ...]
  summary: Listen for messages published to the given channels
  since: 2.0.0

# 退订此前订阅的频道；
  UNSUBSCRIBE [channel [channel ...]]
  summary: Stop listening for messages posted to the given channels
  since: 2.0.0

```

Redis的持久化：

实现持久化的两种机制：

    RDB： snapshot，二进制格式数据文件；按事先定制的策略，周期性的将数据保存至磁盘；数据文件默认为dump.rdb;  配置文件中使用save配置同步周期；
        客户端也可显式使用SAVE或BGSAVE命令启动快照保存机制；
            SAVE： 同步，在主线程中保存快照，此时会阻塞所有客户端请求；
            BGSAVE： 异步，立即返回保存成功，然后在后台同步；不会阻塞请求；
        RDB保存的数据一定不完整；

    AOF： Append Only File，将每一个操作命令以附加的形式记录到文件中；容易导致文件过大；
        会将收到的每一个写命令，都会通过write函数追加到文件末尾；类似于MySQL的二进制日志；
        问题： 文件会变得越来越大；redis程序能自动合并重写该类型的持久化文件；

        记录每一次写操作至指定的文件尾部实现持久化，当redis重启时，可通过重新执行文件中的命令在内存重建数据库；

            BGREWAITEAOF： AOF文件重写；
                不会读取正在使用的AOF文件，而通过将内存中的数据以命令的形式保存到临时文件中，完成之后替换原来的AOF文件；

    RDB：

        SAVE 900 1
        SAVE 300 10
        SAVE 60 10000
        stop-writes-on-bgsave-error yes
        rdbcompression yes
        rdbchecksum yes
        dbfilename dump.rdb
        dir /var/lib/redis

        关闭RDB功能： save ""

    AOF:

        重写过程：
            (1)redis主进程通过fork创建子进程；
            (2)子进程根据redis内存中的数据创建数据库重建命令序列于临时文件中；
            (3)父进程继续接收client的请求，并会把这些请求中的写操作继续追加至原来的AOF文件；而外地，这些新的写请求还会被放置于一个缓冲队列中；
            (4)子进程重写完成，会通知父进程，父进程把缓冲中的命令写到临时文件中；
            (5) 子进程用临时文件替换老的AOF文件；

        相关参数：
            appendonly yes   # 启用；
            appendfilename "appendonly.aof"  # 文件名；

            # appendfsync always         # 
            appendfsync everysec         # 推荐使用；
            # appendfsync no

            no-appendfsync-on-rewrite yes # 

            auto-aof-rewrite-percentage 100  # 增长率达到100%时，重写；
            auto-aof-rewrite-min-size 64mb   # 避免频繁重写，要求最小限制；


    注意：持久本身不能取代备份，还应该制定备份策略，对redis数据库定期进行备份；

    RDB与AOF同时启用：
        (1)BGSAVE和BGREWRITEAOF不会同时执行；
        (2)在Redis服务器启动用户恢复数据时，会优先使用AOF；


### Redis的复制：

* 特点：

        一个maser可以由多个slave；
        支持链式复制；
        Master以非阻塞方式同步数据至slave；

* 过程：

    Slave：
        > SLAVEOF　MASTER_IP MASTER_PORT
        > slave-read-only yes
        
    坑： 从服务器bind指令后监听的IP地址，要么写0.0.0.0，要么写能与master通信的ip；

注意： 如果master使用requirepass开启了认证功能，从服务器要使用masterauth <PASSWORD>来连接master服务器使用此密码进行认证；

            
Sentinel:哨兵

    用于管理多个redis服务实现HA；
        监控master
        通知
        自动故障转移；

        流言协议，投票协议，

    程序： 
        
        redis-sentinel /path/to/file.conf
        redis-server /path/to/file.conf --sentinel 

        (1) 服务器自身初始化，运行redis-server中专用于sentinel功能的diamante；
        (2) 初始化sentinel状态，根据给定的配置文件，初始化监控的master服务器列表；
        (3) 创建连向master的连接；

    专用配置文件：

        /etc/redis-sentinel.conf 
            (1) entinel monitor <master-name> <ip> <redis-port> <quorum>
                sentinel monitor mymaster 127.0.0.1 6379 2

            (2) sentinel down-after-milliseconds <master-name> <milliseconds>
                sentinel down-after-milliseconds mymaster 30000
                # 30秒找不到master，则认为其不在线了；

            (3) sentinel parallel-syncs mymaster 1

            (4) sentinel failover-timeout <master-name> <milliseconds>
                sentinel failover-timeout mymaster 180000
                # 故障转移的超时时间；

    主管下线，客观下线：

        主管下线： 一个sentinel实例判断出某节点下线；

        客观下线： 多个sentinel节点协商后判断出某节点下线；

    专用命令：

        SENTINEL masters
        SENTINEL slaves <master name>
        SENTINEL get-master-addr-by-name <master name>
        SENTINEL reset
        SENTINEL failover <master name>

    sentinel节点也应该有多个(奇数个)


### Clustering：

    分布式数据库： 通过分片机制进行数据分布，Clustering内的每个节点仅持有数据库的一部分数据。

    redis的集群属于去中心化的集群；

    每个节点持有全局元数据(键值映射)，但仅持有一部分数据；


* Twemproxy(Twitter)
* Codis(豌豆荚)
* Redis Cluster(官方)
* Cerberus(芒果TV)


* Twemproxy(Twitter)

        代理分片机制；
        优点： 非常稳定，企业级方案；
        缺点：
            单点故障；
            需依赖第三方软件，如Keepalived；
            无法平滑地横向扩展；
            没有后台界面；
            代理分片机制引入更多的来回次数并提高延迟；
            单核模式，无法充分利用多核，除非多实例；
            Twitter官方内部不再继续使用Twemproxy

* Codis(豌豆荚)

        代理分片机制；
        2014年11月开源；
        基于Go以及C语言研发；
        优点：
            非常稳定，企业级方案；
            数据自动平衡；
            高性能；
            简单的测试显示较Twemproxy快一倍；
            善用多核CPU
            简单：
                没有paxos类的协调机制；
                没有主从复制；
            有后台界面

        缺点：
            代理分片机制引入更多的来回次数并提高延迟；PING - PONG操作；
            需要第三方软件支持协调机制；
                目前支持zookeeper 及 Etcd；
            不支持主从复制，需要另外实现；
            Codis采用了Proxy的方案，所以必然会带来单机性能的损失，
                经测试，在不开pipline的情况下，大概会损失40%左右的性能；

* Redis Cluster(官方)

        官方实现
        需要Redis 3.0 或更高版本；

        优点：
            无中心的P2P Gossip分散式模式；
            更少的来回次数并降低延迟；
            自动于多个Redis节点进行分片；
            不需要第三方软件支持协调机制；

        缺点：
            依赖于Redis 3.0 或更高版本；
            需要时间验证其稳定性；
            没有后台界面；
            需要智能客户端；
            Redis客户端必须支持Redis Cluster结构；
            较Codis有更多的维护升级成本；


* Cerberus(芒果TV)

        优点：
            数据自动平衡；
            本身实现了Redis的Smart Client
            支持读写分离

        缺点：
            依赖Redis 3.0或更高版本；
            代理分片机制引入更多的来回次数并增大延迟；
            需要时间验证其稳定性；
            没有后台界面；



