redis整个笔记需要重新构建   

# Redis   

https://redis.io/commands/set/ 

## 介绍

> redis 默认16个数据库,从 0 - 15 号,默认使用0号  

> 使用命令 `select 1` 来切换到1号数据库  

> 单线程 + 多路IO复用    

## 基本操作  


可执行文件 | 作用 |
---------|----------|
 `redis-server` | 启动Redis |
 `redis-cli` | Redis命令行客户端 |
 `redis-benchmark` | Redis基准测试工具 |
 `redis-check-aof` | Redis AOF 持久化文件检测和修复工具 |
 `redis-check-dump`| Redis RDB 持久化文件检测和修复工具 |
 `redis-sentinel`| 启动redis哨兵 |  


> `redis-server` 加上要修改的配置,可以进行临时配置,而不用修改配置文件       
> `redis-server --configKey1 --configKey2 --configKey3`   
> 如: `redis-server --port 6308 --logfile ./files --daemonize yes`


> `redis-cli shutdown` 停止Redis服务  
> 除了可以通过shutdown命令关闭Redis服务以外，还可以通过kill进程号的方式关闭掉Redis，但是不要粗暴地使用kill-9强制杀死Redis服务，  
> 不但不会做持久化操作，还会造成缓冲区等资源不能被优雅关闭，极端情况会造成AOF和复制丢失数据的情况
> `redis-cli shutdown nosave|save` 选择是否持久化数据   

## 为什么这么快? 
- 纯内存访问，Redis将所有数据放在内存中，内存的响应时长大约为100纳秒，这是Redis达到每秒万级别访问的重要基础。
- 非阻塞I/O，Redis使用 `epoll` 作为I/O多路复用技术的实现，再加上Redis自身的事件处理模型将epoll中的连接、读写、关闭都转换为事件，不在网络I/O上浪费过多的时间  
- 单线程避免了线程切换和`竞态`产生的消耗   

> `竞态`就是在多线程的编程中，你在同一段代码里输入了相同的条件，但是会输出不确定的结果的情况    
> 当两个线程竞争同一资源时，如果对资源的访问顺序敏感(会导致不可预测的结果)，就称存在竞态条件

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis多路复用.png)

## 键 key

- `keys *` 查看当前库所有key
- `exists key` 判断key是否存在
- `del key` 删除
- `unlink key` 非阻塞删除  
- `expire key 10` 过期  -1 永不过期, -2 已过期失效 

## 值 五+三 种类型  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis5种数据结构.png)

### String字符串  
> String 类型是二进制安全的。意味着 Redis 的 string 可以包含任何数据。比如 jpg 图片或者序列化的对象。  
> String 类型是 Redis 最基本的数据类型，一个 Redis 中字符串 value 最多可以是 512M。   
> String 的数据结构为简单动态字符串,内部结构类似 `ArrayList`   
> 扩容机制,字符串长度小于1M 时,扩容策略时翻倍当前空间,超过1M时每次扩容1M空间   

- 总体示例: `set key value ex(秒) px(毫秒) (nx|xx)`
  - nx,键不存在才能赋值
  - xx,键必须存在才能赋值
  - ex|px 都是过期时间,单位不同  
- `set <key><value>`：添加键值对
- `get <key>`：查询对应键值
- `append <key><value>`：将给定的 <value> 追加到原值的末尾
- `strlen <key>`：获得值的长度
- ⭐`setnx <key><value>`：只有在 key 不存在时，设置 key 的值
- `incr <key>`：将 key 中储存的数字值增 1，只能对数字值操作，如果为空，新增值为 1（**具有原子性**）
- `decr <key>`：将 key 中储存的数字值减 1，只能对数字值操作，如果为空，新增值为 -1
- `incrby/decrby <key><步长>`：将 key 中储存的数字值增减。自定义步长
- `mset <key1><value1><key2><value2> `：同时设置一个或多个 key-value 对
- `mget <key1><key2><key3>...`：同时获取一个或多个 value
- `msetnx <key1><value1><key2><value2>...` ：同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在
- `getrange <key><起始位置><结束位置>`：获得值的范围 获取值的截断值  
- `setrange <key><起始位置><value>`：用 <value> 覆写 <key> 所储存的字符串值
- ⭐`setex <key><过期时间><value>`：设置键值的同时，设置过期时间，单位秒。
- `getset <key><value>`：以新换旧，设置了新值同时获得旧值。  

> 原子性,redis 操作不会被打断具有原子性,Redis 单命令的原子性 得益于Redis 的单线程   

### Hash哈希

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redishash.png)

> Redis hash 是一个键值对集合。
> Redis hash 是一个 String 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。 
> 类似java中的 Map, Json 格式 (参考)   
> 数据结构: field-value 长度较短(`hash-max-ziplist-value` 默认 64字节)且个数较少(`hash-max-ziplist-entries` 默认 512个)时,使用ziplist 压缩列表 ,否则使用hashtable 哈希表    
> 注意hashtable 会占用更多内存, 所以要尽量保证

- `hset <key> <field> <value>`：给 <key> 集合中的 <field> 键赋值 <value>
- `hget <key1> <field>`：从 <key1> 集合 <field> 取出 value
- `hmset <key1> <field1> <value1> <field2> <value2>...`： 批量设置 hash 的值
- `hexists <key1> <field>`：查看哈希表 key 中，给定域 field 是否存在
- `hkeys <key>`：列出该 hash 集合的所有 field
- `hvals <key>`：列出该 hash 集合的所有 value
- `hincrby <key> <field> <increment>`：为哈希表 key 中的域 field 的值加上增量 1 -1
- `hsetnx <key> <field> <value>`：将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redishash.png)


### List列表  
> 单键多值 
> 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。   
> 它的底层实际是个双向链表，**对两端的操作性能很高**，通过**索引下标的操作中间的节点性能会较差**。  

- `lpush/rpush <key><value1><value2><value3> ....`： 从左边/右边插入一个或多个值。
- lpush 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redislistexp.png)

- rpush

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redislistexprpush.png)

- `lpop/rpop <key>`：从左边/右边吐出一个值。消耗值,**调用完该值即会移除,取完所有值,键就消失**。
- `rpoplpush <key1><key2>`：从 <key1> 列表右边吐出一个值，插到 <key2> 列表左边。
- `lrange <key><start><stop>`：按照索引下标获得元素（从左到右）
- `lindex <key><index>`：按照索引下标获得元素（从左到右）
- `llen <key>`：获得列表长度
- `linsert <key> before/after <value><newvalue>`：在 <value> 的前面/后面插入 <newvalue> 插入值
- `lrem <key><n><value>`：从左边删除 n 个 value (n>0 从左到右) (n<0 从右到左) (n=0 全部删除)
- `lset<key><index><value>`：将列表 key 下标为 index 的值替换成 value
- `blpop/brpop key timeout` 等待指定时间返回,如果timeout是0会一直阻塞  


> List 的数据结构为快速链表 quickList。  
> 首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是 ziplist，也即是压缩列表。   
> 当数据量比较多的时候(元素个数超过512个)才会改成 quicklist(多个ziplist 使用双向指针结合起来)。  


- 使用场景 
  - 消息队列 `lpush+brpop` 
  - 栈 `lpush+lpop`
  - 队列 `lpush+rpop` 


### Set集合  
> Set 对外提供的功能与 List 类似列表的功能，特殊之处在于 Set 是可以自动排重的,无序的   
> 当需要存储一个列表数据，又不希望出现重复数据时，Set 是一个很好的选择，并且 Set 提供了判断某个成员是否在一个 Set 集合内的重要接口，这个也是 List 所不能提供的  
> Set 数据结构是 dict 字典,字典使用哈希表实现的   

- `sadd <key><value1><value2> .....` ：将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
- `smembers <key>`：取出该集合的所有值。
- `sismember <key><value>`：判断集合 <key> 是否为含有该 <value> 值，有返回 1，没有返回 0
- `scard<key>`：返回该集合的元素个数。
- `srem <key><value1><value2> ....`：删除集合中的某个元素
- `spop <key>`：随机从该集合中吐出一个值
- `srandmember <key><n>`：随机从该集合中取出 n 个值，不会从集合中删除
- `smove <source> <destination> <value>`：把集合中一个值从一个集合移动到另一个集合
- `sinter <key1> <key2>`：返回两个集合的交集元素
- `sunion <key1> <key2>`：返回两个集合的并集元素
- `sdiff <key1> <key2>`：返回两个集合的差集元素（key1 中的，不包含 key2 中的）




### Zset  

> Redis 有序集合 zset 与普通集合 set 非常相似，是一个没有**重复元素**的字符串集合。   
> 不同之处是有序集合的每个成员都关联了一个评分（score）,这个评分（score）被用来按照从**最低分到最高分的方式排序集合中的成员**。集合的成员是唯一的，但是评分可以是重复的。    
> 因为元素是有序的，所以可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。   
> 访问有序集合的中间元素也是非常快的，因此能够使用有序集合作为一个没有重复成员的智能列表。   

> `zadd topn 200 java 400 c++ 400 mysql 500 php` 向数据库中添加 四条数据    
> 数据会自动根据score 有小到大排序   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/rediszset.png)

- `zadd <key><score1><value1><score2><value2>…`：将一个或多个 member 元素及其 score 值加入到有序集 key 当中
- `zrange <key><start><stop> [WITHSCORES]` ：返回有序集 key 中，下标在 <start><stop> 之间的元素当带 WITHSCORES(大小写忽略)，可以让分数一起和值返回到结果集
- `zrangebyscore key min max [withscores] [limit offset count]`：返回有序集 key 中，所有 score 值介于 min 和 max 之间（包括等于 min 或 max ）的成员。有序集成员按 score 值递增（从小到大）次序排列。
- `zrevrangebyscore key max min [withscores] [limit offset count]` ：同上，改为从大到小排列
- `zincrby <key><increment><value>`：为元素的 score 加上增量
- `zrem <key><value>`：删除该集合下，指定值的元素
- `zcount <key><min><max>`：统计该集合，分数区间内的元素个数
- `zrank <key><value>`：返回该值在集合中的排名，从 0 开始。


> SortedSet（zset）是 Redis 提供的一个非常特别的数据结构，一方面它等价于 Java 的数据结构 `Map<String, Double>`，可以给每一个元素 value 赋予一个权重 score，另一方面它又类似于 TreeSet，内部的元素会按照权重 score 进行排序，可以得到每个元素的名次，还可以通过 score 的范围来获取元素的列表。    
> zset 底层使用了两个数据结构    
> hash，hash 的作用就是关联元素 value 和权重 score，保障元素 value 的唯一性，可以通过元素 value 找到相应的 score 值    
> 跳跃表，跳跃表的目的在于给元素 value 排序，根据 score 的范围获取元素列表   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/跳跃表示例.png)  
 
### Bitmaps  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis-bitmap.png)

> 合理使用操作位能有效提高内存使用率      
> Redis 提供的 Bitmaps 这个数据类型 可以实现对位的操作    
> Bitmaps 本身不是一种数据类型,实际上就是字符串(key-value)   
> Bitmaps 单独提供了一套命令,来操作数据  
> 可以把Bitmaps 看作是一个以位为单位的数组,数组的每个单元只能存储 0 和 1 ,数组的下标在Bitmaps 中叫做偏移量   

- `setbit <key> <offset> <value>` 设置Bitmaps 中某个偏移量的值 (0或1)  
- `getbit <key> <offset>` 取出Bitmaps 中某个偏移量的值 (0或1)    
- `bitcount <key> [start end]` 统计字符串从start字节到end字节比特值为1 的数量  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/bitmap示例.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/bitmaps对比set.png)


### HyperLogLog  

> 基数,简单理解为自动去重数据集    
> 比如,数据集{1,2,3,4,5,4,3},那么这个数据集的基数集为 {1,2,3,4,5}   

> Redis HyperLogLog 是用来做基数统计的算法,HyperLogLog 的优点是,在输入元素的数量或者体积非常大时,计算基数所需的空间很小

- `pfadd key element...` 
- `pfcount key` 返回基数集长度
- `pfmerge targetkey sourcekey ...` 合并key

### Geospatial  
> Geospatinal 地理信息类型,元素的2维坐标,经纬度,redis基于该类型,提供了经纬度设置,查询,范围查询,距离查询,经纬度hash等操作 

- `geoadd <key> <longitude> <latitude> <memnber> ` 添加地理位置 经度纬度名称

```
geoadd china:city 121.47 31.23 shanghai
geoadd china:city 106.50 29.53 chongqing 114.05 22.52 shenzhen 116.38 39.90 beijing
```

- `geodist key member1 member2 [m|km|ft|mi]` 获取两个位置之间的**直线距离** 米|千米|英尺|英里    

```
geodist china:city beijing shanghai km 
```

- `georadius key longitude latitude radius [m|km|ft|mi]` 以给定的经纬度为中心,找出某一半径内的元素   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redisgeoradius.png)

## 配置文件详解   

### 单位设置方式  

```conf
# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```

### 文件包含 配置分离

```conf
# include /path/to/local.conf
# include /path/to/other.conf
```

### 网络配置  

```conf
# 打开的话就是限定本地连接
#bind 127.0.0.1
# 保护模式
protected-mode yes
# 端口
port 6777

# TCP listen() backlog.
# 设置tcp 队列总和  理解为连接数  
tcp-backlog 511

# Close the connection after a client is idle for N seconds (0 to disable)
# 超时时间 
timeout 0

# 心跳时间 单位 秒 
tcp-keepalive 300

# By default Redis does not run as a daemon. Use 'yes' if you need it.
daemonize yes

# Specify the server verbosity level. 日志级别  
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice

# 默认数据库数量
databases 16

# 密码 
requirepass 123123

```

## 发布订阅  

> Redis 发布订阅（ pub/sub ）是一种**消息通信模式**：发送者（ pub ）发送消息，订阅者（ sub ）接收消息。  
> Redis 客户端可以订阅任意数量的频道。  

- `subscribe channel1` 订阅频道  
- `publish channel1 hello` 给 channel1 发消息 hello 返回值是订阅者的数量  



## Jedis  



## Redis 事务 
> Redis 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。   
> Redis 事务的主要作用就是**串联多个命令**防止别的命令插队。

### Multi、Exec、Discard

> 从输入 Multi 命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入 Exec 后，Redis 会将之前的命令队列中的命令依次执行。  

> 组队的过程中可以通过 Discard 来放弃组队

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis组队.png)

> 分为两种情况,若组队中有错误,那么所有命令都不会执行,若执行阶段有错误,那么只有错误的命令不会生效执行    

> 关注 jedis 和 redisTemplate 中的使用,以及事务的操作

### 悲观锁  
> 悲观锁(Pessimistic Lock), 顾名思义,就是很悲观,每次去拿数据的时候都认为别人会修改,所以每次在拿数据的时候都会上锁,   
> 这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制,比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。

### 乐观锁 

> 乐观锁（Optimistic Lock），即每次去拿数据的时候都认为其他线程不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间有没有其他线程去更新这个数据，可以使用版本号等机制。  
> 乐观锁适用于多读的应用类型，这样可以提高吞吐量。   
> Redis 就是利用这种 check-and-set 机制实现事务的   

### Watch、unwatch 
> 在执行 multi 之前，先执行 `watch key1 [key2]`，可以监视一个（或多个 ）key 。如果在事务执行之前这个 key 被其他命令所改动，那么事务将被打断。   
> 取消 WATCH 命令对所有 key 的监视。如果在执行 WATCH 命令之后，EXEC 命令或 DISCARD 命令先被执行，那么就不需要再执行 UNWATCH 

### 事务三特性(redis)  

- 单独的隔离操作
  - 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
- 没有隔离级别的概念
  - 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行。
- 不保证原子性
  - 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 。



## 持久化 

### RDB (Redis DataBase)

> 在指定的**时间间隔**内将内存中的数据集快照写入磁盘， 即 Snapshot 快照，恢复时是将快照文件直接读到内存里    
> Redis 会单独创建一个子进程（fork）来进行持久化。    
> 先将数据写入到一个临时文件中，待持久化过程完成后，再将这个临时文件内容覆盖到 dump.rdb。    
> 默认文件为 `dump.rdb`  默认路径为 `./`
> 整个过程中，主进程是不进行任何 IO 操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那 RDB 方式要比 AOF 方式更加的高效。
> RDB 的缺点是最后一次持久化后的数据可能丢失  

Fork
> 作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程   
> 在 Linux 程序中，fork() 会产生一个和父进程完全相同的子进程，但子进程在此后多会 exec 系统调用，出于效率考虑，Linux 中引入了 写时复制技术   
> 一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程   

```conf
# 自动持久化 策略
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
save 900 1
save 300 10
save 60 10000

# 在 redis.conf 中配置文件名称，默认为 dump.rdb。
dbfilename dump.rdb

# rdb 文件的保存路径可以修改。默认为 Redis 启动时命令行所在的目录下。
dir ./

# 即当 redis 无法写入磁盘，关闭 redis 的写入操作。
stop-writes-on-bgsave-error yes

# 持久化的文件是否进行压缩存储。
rdbcompression yes

#完整性的检查，即数据是否完整性、准确性。
rdbchecksum yes
```  

- 优点
  - 适合大规模的数据恢复；
  - 对数据完整性和一致性要求不高更适合使用；
  - 节省磁盘空间；
  - 恢复速度快。
- 缺点
  - Fork 的时候，内存中的数据被克隆了一份，大致 2 倍的膨胀性需要考虑；  
  - 虽然 Redis 在 fork 时使用了写时拷贝技术，但是如果数据庞大时还是比较消耗性能；
  - 在备份周期在一定间隔时间做一次备份，所以如果 Redis 意外 down 掉的话，就会丢失最后一次快照后的所有修改  


### AOF (Append Only File) 

> 以日志的形式来**记录每个写操作(增量保存)**, 将Redis 执行过的所有指令记录下来(读除外),只许追加文件不能改写文件   
> redis 启动时会读取该文件重新构建数据  

- AOF 默认不开启
- 若AOF 和 RDB 同时开启,系统默认去取AOF来恢复数据

```conf
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly yes

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"

# If unsure, use "everysec".

# 同步频率设置  
# appendfsync always
appendfsync everysec
# appendfsync no



# 重写 压缩
# AOF 文件持续增长过大,会Fork 一条进程,来将文件重写,原理:将rdb快照以二进制形式附加在AOF头部
no-appendfsync-on-rewrite no

# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.


#重写触发条件 :当大于 64Mb 100% 时 即 128Mb 触发 

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

```

- 优点
  - 备份机制更稳健，丢失数据概率更低；
  - 可读的日志文本，通过操作 AOF 稳健，可以处理误操作。
- 缺点
  - 比起 RDB 占用更多的磁盘空间；
  - 恢复备份速度要慢；
  - 每次读写都同步的话，有一定的性能压力；
  - 存在个别 Bug，造成不能恢复。
- 选择
- 官方推荐两个都启用。
- 如果对数据不敏感，可以选单独用 RDB。
- 不建议单独用 AOF，因为可能会出现 Bug。
- 如果只是做纯内存缓存，可以都不用。


## 主从复制  

> 主机数据更新后,根据配置策略,自动同步到备份机的 `master/slave` 机制, `Master` 以写为主, `Slave` 以读为主   
> 每个从节点只能有一个主节点，而主节点可以同时具有多个从节点。复制的数据流是单向的，只能由主节点复制到从节点    

- 读写分离性能扩展
- 容灾快速恢复  

- 开启主从复制
  - 在配置文件中加入`slaveof{masterHost}{masterPort}`随Redis启动生效
  - 在redis-server启动命令后加入`--slaveof{masterHost}{masterPort}`生效 从服务器重启后失效
  - 直接使用命令：`slaveof{masterHost}{masterPort}`生效 从服务器重启后失效

> `slaveof` 本身是异步命令，执行slaveof命令时，节点只保存主节点信息后返回，后续复制流程在节点内部异步执行    
> 使用`info replication`命令查看复制相关状态  

- 断开复制
  - 节点执行 `slaveof no one`断开与主节点的复制关系
  - 从节点晋升为主节点   
  - 节点断开复制后并不会抛弃原有数据，只是无法再获取主节点上的数据变化
  - 执行 `slaveof{newMasterIp}{newMasterPort}` 命令可以切换主机   

- 切主流程
  - 断开与旧主节点复制关系
  - 与新主节点建立复制关系
  - 删除当前节点所有数据
  - 从新主节点复制数据  

> 默认情况下，从节点使用`slave-read-only=yes`配置为只读模式,所以从节点默认是只读  



### 复制流程  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis主从复制流程.png)



### 数据同步

> Redis在2.8及以上版本使用 `psync` 命令完成主从数据同步，同步过程分为：全量复制和部分复制

> 全量复制简单说就是发送主机 RDB 文件   
> **主节点执行bgsave保存RDB文件到本地**,主节点发送RDB文件给从节点，从节点把接收的 RDB 文件保存在本地并直接作为从节点的数据文件    
> 对于数据量较大的主节点，比如生成的RDB文件超过6GB以上时要格外小心。传输文件这一步操作非常耗时  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redi全量复制流程.png)


> 从节点保存了自身已复制的偏移量和主节点的运行ID。因此会把它们当作psync参数发送给主节点，要求进行部分复制操作   
> 主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis部分复制过程.png)


## 哨兵模式   

> 主从复制模式的问题:    
> 一旦主节点出现故障,需要手动将一个从节点晋升为主节点,同时还需要修改应用方的地址,以及命令其他节点复制新主节点,非常繁琐     


### 创建  

- 创建 `sentinel.conf` 写入 `sentinel monitor mymaster 127.0.0.1 6379 1`  
  - mymaster 为监控对象名称,1 是至少有一个哨兵同意迁移  
- `./redis-sentinel sentinel.conf` 命令启动

> 命令解读 `sentinel monitor <master-name> <ip> <port> <quorum>`   
> Sentinel节点要监控的是一个名字叫做`<master-name>`,ip地址和端口为`<ip><port>`的主节点。`<quorum>`代表要判定主节点最终不可达所需要的票数  


> `<quorum>`参数用于故障发现和判定，例如将quorum配置为2，代表至少有2个Sentinel节点认为主节点不可达,那么这个不可达的判定才是客观的。  
> 对于`<quorum>`设置的越小，那么达到下线的条件越宽松，反之越严格。一般建议将其设置为Sentinel节点的一半加1   

### 选举规则
- 根据优先级别，slave-priority/replica-priority，优先选择优先级靠前的。
- 根据偏移量，优先选择偏移量大的。偏移量:即和`Master`的数据差别
- 根据 runid，优先选择最小的服务


## 集群  

> Redis Cluster(无中心化集群)是Redis的分布式解决方案，在3.0版本正式推出，有效地解决了Redis分布式方面的需求。   
> 当遇到单机内存、并发、流量等瓶颈时，可以采用Cluster架构方案达到负载均衡的目的   

### 搭建

```conf
# 以redis6379.conf为例
include /opt/etc/redis.conf
pidfile /var/run/redis_6379.pid # 更改
port 6379 # 更改
dbfilename dump6379.rdb # 更改
cluster-enabled yes # 打开集群模式
cluster-config-file nodes-6379.conf # 设置节点配置文件名称，需要更改
cluster-node-timeout 15000 # 设置节点失联事件，超过该时间（ms），集群自动进行主从切换
```

- 节点准备好 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis集群.png)


```conf
# 进入redis安装目录
/redis-stable/src/

# 执行
./redis-cli --cluster create --cluster-replicas 1 192.168.56.104:6379 192.168.56.104:6380 192.168.56.104:6381 192.168.56.104:6389 192.168.56.104:6390 192.168.56.104:6391
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis集群过程.png)

- 通过命令登录集群查看 `./redis-cli -c -p 6379` 集群信息

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis集群显式.png)

> 可以看到分配的集群有三组 `master-slove` 节点

- redis cluster 如何分配这六个节点?
  - 一个集群至少要有三个主节点。
  - 选项` --cluster-replicas 1`，表示希望为集群中的每个主节点创建一个从节点。
  - 分配原则尽量保证每个主数据库运行在不同的 IP 地址，每个从库和主库不在一个 IP 地址上。

> Redis集群把所有的数据映射到16384个槽中。每个key会映射为一个固定的槽，只有当节点分配了槽，才能响应和这些槽关联的键命令  
> 在插入key时 会计算key 属于那个槽,然后将其防毒对应节点中  
> 集群中每个节点负责处理一部分插槽     
> 如:节点A 处理 0-5460,节点B 处理 5461-10922 ,节点 C 处理10923-16383  


> 如果所有某-段插槽的主从节点都宕掉, redis 服务是否还能继续?   
> 如果某一段插槽的主从都挂掉,而redis.conf 中`cluster-require-full-coverage`为yes , 那么整个集群都挂掉    
> 如果某一段插槽的主从都挂掉 ,而`cluster-require-full-coverage`为no,那么该插槽的数据全部无法使用无法存储     



## 应用问题 


### 缓存穿透  

> key 对应的数据在数据源并不存在，每次针对此 key 的请求从缓存获取不到，请求都会压到数据源，从而可能压垮数据源  


#### 如何解决

- 对空值缓存

> 如果一个查询返回的数据为空（不管是数据是否不存在），仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟。

- 设置可访问的名单（白名单）：

> 使用 bitmaps 类型定义一个可以访问的名单，名单 id 作为 bitmaps 的偏移量，每次访问和 bitmap 里面的 id 进行比较，如果访问 id 不在 bitmaps 里面，进行拦截，则不允许访问。

- 采用布隆过滤器

> 布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量（位图）和一系列随机映射函数（哈希函数）。   
> 布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。  
> 将所有可能存在的数据哈希到一个足够大的 bitmaps 中，一个一定不存在的数据会被这个 bitmaps 拦截掉，从而避免了对底层存储系统的查询压力。

- 进行实时监控

> 当发现 Redis 的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务


### 缓存击穿

> 某一个热点 key，在缓存过期的一瞬间，同时有大量的请求打进来，由于此时缓存过期了，所以请求最终都会走到数据库，造成瞬时数据库请求量大、压力骤增，甚至可能打垮数据库  

- 加互斥锁  

> 在并发的多个请求中，只有第一个请求线程能拿到锁并执行数据库查询操作，其他的线程拿不到锁就阻塞等着，等到第一个线程将数据写入缓存后，直接走缓存  

- 热点数据不过期  

> 直接将缓存设置为不过期，然后由定时任务去异步加载数据，更新缓存   


### 缓存雪崩  


> 大量的热点 key 设置了相同的过期时间，导在缓存在同一时刻全部失效，造成瞬时数据库请求量大、压力骤增，引起雪崩，甚至导致数据库被打挂  

- 构建多级缓存架构

> nginx 缓存 + redis 缓存 + 其他缓存（ehcache等）

- 过期时间打散 

> 给缓存的过期时间时加上一个随机值时间，使得每个 key 的过期时间分布开来，不会集中在同一时刻失效。

- 热点数据不过期。

> 该方式和缓存击穿一样，也是要着重考虑刷新的时间间隔和数据异常如何处理的情况。  

- 加互斥锁 

> 该方式和缓存击穿一样，按 key 维度加锁，对于同一个 key，只允许一个线程去计算，其他线程原地阻塞等待第一个线程的计算结果，然后直接走缓存即可 

- 设置过期标志,更新缓存

> 设置提前量,临近过期时触发通知更新缓存   

### 分布式锁  

- 使用命令 `setnx` 上锁,通过del释放锁  
- 可以给锁设置过期时间,防止异常无法释放锁`set user 10 nx ex 12` 给 user,10 上锁, 12s 过期  
- 使用uuid 防止误删操作,释放锁前判断值是否相同 
- 分布式锁:使用lua脚本实现整个获取锁,操作,释放锁的过程,保证原子性  

为了保证分布式锁可用,必须保证锁同时满足如下条件   
- 互斥,任意时刻只有一个客户端能持有锁
- 不会发生死锁,持有锁的客户端没有及时解锁,也能保证后续其他客户端能加锁
- 加锁和解锁必须是同一个客户端
- 加锁和解锁操作必须是一个原子操作









# 参考
> - [尚硅谷视频](https://www.bilibili.com/video/BV1Rv41177Af)  
> - [尚硅谷视频Redis7](https://www.bilibili.com/video/BV13R4y1v7sP)  
> - [Redis开发与运维]





