# Redis基础

## Redis入门

### Redis简介

解决了关系型数据库性能瓶颈和扩展瓶颈，性能瓶颈：磁盘IO性能低下，扩展瓶颈：数据关系复杂，扩展性差，不便于大规模集群。Nosql解决这种问题,降低磁盘IO次数，去除数据间关系。

Nosql特征：

- 可扩容，可伸缩
- 大数据量下高性能
- 灵活的数据模型
- [高可用]()



#### Redis使用场景：

为热点数据加速查询（主要场景），如热点商品、热点新闻、热点资讯、推广类等高访问量信息

任务队列，如秒杀、抢购、购票排队等 

即时信息查询，如各位排行榜、各类网站访问统计、公交到站信息、在线人数信息（聊天室、网站）、设备信号等时效性信息控制，如验证码控制、投票控制等 

分布式数据共享，如分布式集群架构中的 session 分离 

消息队列 

分布式锁



#### Redis特征

1. 数据间没有必然的关系
2. 内部采用单线程机制进行工作（Redis4之后也支持多线程）
3. 高性能
4. 多数据类型支持
   1. String
   2. List
   3. hash
   4. set
   5. sorted_set
5. 持久化支持，可以进行数据灾难恢复。



Redis安装目录

redis-server.exe	服务器启动

redis-cli.exe		   客户端

redis.windows.conf  redis核心配置文件

redis-benchmark.exe 性能测试工具

redis-check-aof.exe AOF文件修复工具

redis-check-dump.exe RDB文件检查工具（快照持久化文件）



### Redis 基本操作

添加数据	set key value

信息查询	get key

帮助查询	help 命令名称/@组     名



## 数据类型

Redis作用:

缓存使用

1.原始业务功能设计

- 秒杀
- 618活动
- 双11活动
- 排队购票

2.运营平台监控到的突发高频访问数据

- 突发时政要闻，被强势关注围观

3.高频、复杂的统计数据

- 在线人数
- 投票排行榜

附加功能

- 单服务器升级集群
- Session 管理
- Token 管理

当考虑Redis的数据类型的时候，只考虑value的数据类型，key永远是String类型



### String

存储的数据：单个数据，最简单的数据存储类型，也是最常用的数据存储类型

存储数据的格式：一个存储空间保存一个数据

存储内容：通常使用字符串，如果字符串以整数的形式展示，可以作为数字操作使用



一次添加多个数据

```
mset key1 value1 key2 value2
```

获取多个数据

```
mget key1 key2
```

获取数据字符个数（字符串长度）

```
strlen key
```

追加信息到原始信息后部（如果原始信息存在就追加，否则新建）

```
append key value
```

使用多个数据同时操作的指令，效率更高（但是执行时长长可能造成阻塞）,但是如果数据太长容易造成数据阻塞。



#### 场景：解决Mysql多张分表主键id重复问题

```
incr key
incrby key increment
incrbyfloat key increment
```

```
decr key 
decrby key increment
```

Redis处理string的时候会自动进行类型转换

redis用于控制数据库表主键id，为数据库表主键提供生成策略，保障数据库表的主键唯一性

redis所有的操作都是原子性的，采用单线程处理所有业务，命令是一个一个执行的，因此无需考虑并发带来的数据影响。所以可以用来控制数据库主键id，为数据库表主键提供生成策略，保障数据库表的主键唯一性。



#### 设置数据具有指定的生命周期

```
setex key seconds value
psetex key milliseconds value
```



#### 注意事项 Redis的返回值

(integer) 0 → false 失败

(integer) 1 → true 成功

(integer) 3 → 3 3个
(integer) 1 → 1 1个

以上几种非常类似，需要仔细确认

数据未获取到

（nil）等同于null



#### 业务场景：主页高频访问信息显示控制，例如新浪微博大V主页显示粉丝数与微博数量

解决方案：

在redis中为大V用户设定用户信息，以用户主键和属性值作为key，后台设定定时刷新策略即可 

eg: user:id:3506728370:fans → 12210947 

eg: user:id:3506728370:blogs → 6164 

eg: user:id:3506728370:focuss → 83

在redis中以json格式存储大V用户信息，定时刷新（也可以使用hash类型）

 eg: user:id:3506728370 → {"id":3506728370,"name":"春晚","fans":12210862,"blogs":6164, "focus":83}

Tips 3：
redis应用于各种结构型和非结构型高热度数据访问加速

key设置约定：

表名：主键名：主键值：字段名



### Hash

新的存储需求：对一系列存储的数据进行编组，方便管理，典型应用存储对象信息

需要的存储结构：一个存储空间保存多个键值对数据

hash类型：底层使用哈希表结构实现数据存储

Hash结构：key value(field1 value1, field2 value2 ...)

如果field数量较少，存储结构优化为类数组结构
如果field数量较多，存储结构使用HashMap结构

#### 基本操作

```
//添加修改数据
hset key field value
//查询数据
hget key field 
hgetall key
//删除数据
hdel key field1 [field2]
//获取哈希表中所有的字段名(field)或字段值(value)
hkeys key
hvals key
//设置指定字段的数值数据增加指定范围的值
hincrby key field increment 
hincrbyfloat key field increment

```



#### hash注意事项

hash类型下的value只能存储字符串，不允许存储其他数据类型，不存在嵌套现象。如果数据未获取到，对应的值为（nil）

hash类型十分贴近对象的数据存储形式，并且可以灵活添加删除对象属性。但hash设计初衷不是为了存储大量对象而设计的，切记不可滥用，更不可以将hash作为对象列表使用

hgetall 操作可以获取全部属性，如果内部field过多，遍历整体数据效率就很会低，有可能成为数据访问瓶颈



#### hash 类型应用场景

购物车redis设计：

以客户id作为key，每位客户创建一个hash存储结构存储对应的购物车信息
将商品编号作为field，购买数量作为value进行存储
添加商品：追加全新的field与value
浏览：遍历hash
更改数量：自增/自减，设置value值
删除商品：删除field
清空：删除key

每条购物车中的商品记录保存成两条field:

field1专用于保存购买数量

​	命名格式：商品id:nums 

​	保存数据：数值

field2专用于保存购物车中显示的信息，包含文字描述，图片地址，所属商家信息

​	命名格式：商品id:info 

​	保存数据：json

商品信息单独保存



秒杀redis设计：

以商家id作为key
将参与抢购的商品id作为field
将参与抢购的商品数量作为对应的value
抢购时使用降值的方式控制产品数量

#实际业务中还有超卖等实际问题，这里不做讨论



#### Hash Vs String

String使用json存储数据，更适合数据读取，因为所有数据需要一次性保存，输出

Hash存储数据适合数据修改



### List

数据存储需求：存储多个数据，并对数据进入存储空间的顺序进行区分

需要的存储结构：一个存储空间保存多个数据，且通过数据可以体现进入顺序

list类型：保存多个数据，底层使用双向链表存储结构实现



```
//添加数据
lpush key value1 [value2] ……
rpush key value1 [value2] ……

//获取数据 start和stop是索引
lrange key start stop
lindex key index
//返回长度
llen key

//获取并移除数据
lpop key
rpop key

//规定时间内获取并移除数据
blpop key1 [key2] timeout
brpop key1 [key2] timeout
//从列表中取出最后一个元素，并插入到另外一个列表的头部
brpoplpush source destination timeout
```



#### 业务场景

微信朋友圈点赞，要求按照点赞顺序显示点赞好友信息 如果取消点赞，移除对应好友信息

```
lrem key count value// count:移除几个（用于移除重复元素） value:移除的具体value
```



#### 注意事项

list中保存的数据都是string类型的

list具有索引的概念，但是操作数据时通常以队列的形式进行入队出队操作，或以栈的形式进行入栈出栈操作

获取全部数据操作结束索引设置为-1

list可以对数据进行分页操作，通常第一页的信息来自于list，第2页及更多的信息通过数据库的形式加载



应用场景2

twitter、新浪微博、腾讯微博中个人用户的关注列表需要按照用户的关注顺序进行展示，粉丝列表需要将最近关注的粉丝列在前面

依赖list的数据具有顺序的特征对信息进行管理
使用队列模型解决多路信息汇总合并的问题
使用栈模型解决最新消息的问题



### set

新的存储需求：存储大量的数据，在查询方面提供更高的效率
需要的存储结构：能够保存大量的数据，高效的内部存储机制，便于查询
set类型：与hash存储结构完全相同，仅存储键，不存储值（nil），并且值是不允许重复的



![image-20210826145624213](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210826145624213.png)

```
//添加数据
sadd key member1 [member2]
//获取全部数据
smembers key
//删除数据
srem key member1 [member2]
//获取集合数据总量
scard key
//判断集合中是否包含指定数据
sismember key member
```



#### 业务场景

每位用户首次使用今日头条时会设置3项爱好的内容，但是后期为了增加用户的活跃度、兴趣点，必须让用户对其他信息类别逐渐产生兴趣，增加客户留存度，如何实现？



系统分析出各个分类的最新或最热点信息条目并组织成set集合
随机挑选其中部分信息
配合用户关注信息分类中的热点信息组织成展示的全信息集合（用户喜欢的内容+不喜欢的内容组合推送）



```
//随机获取集合中指定数量的数据
srandmember key [count]
//随机获取集合中的某个数据并将该数据移出集合
spop key [count]
```



美团外卖为了提升成单量，必须帮助用户挖掘美食需求，如何推荐给用户最适合自己的美食？

```
//求两个集合的交、并、差集
sinter key1 [key2]
sunion key1 [key2]
sdiff key1 [key2]
//求两个集合的交、并、差集并存储到指定集合中
sinterstore destination key1 [key2] 
sunionstore destination key1 [key2] 
sdiffstore destination key1 [key2]
//将指定数据从原始集合中移动到目标集合中
smove source destination member
```



#### set 类型数据操作的注意事项

set 类型不允许数据重复，如果添加的数据在 set 中已经存在，将只保留一份

set 虽然与hash的存储结构相同，但是无法启用hash中存储值的空间



#### 应用场景

公司对旗下新的网站做推广，统计网站的PV（访问量）,UV（独立访客）,IP（独立IP）。 PV：网站被访问次数，可通过刷新页面提高访问量 

UV：网站被不同用户访问的次数，可通过cookie统计访问量，相同用户切换IP地址，UV不变 

IP：网站被不同IP地址访问的总次数，可通过IP地址统计访问量，相同IP不同用户访问，IP不变

利用set集合的数据去重特征，记录各种访问数据
建立string类型数据，利用incr统计日访问量（PV）
建立set模型，记录不同cookie数量（UV）
建立set模型，记录不同IP数量（IP）



Redis的set也可以用来制作黑白名单



### Sorted_set

新的存储需求：数据排序有利于数据的有效展示，需要提供一种可以根据自身特征进行排序的方式
需要的存储结构：新的存储模型，可以保存可排序的数据
sorted_set类型：在set的存储结构基础上添加可排序字段



#### 基本操作

```
//添加数据
zadd key score1 member1 [score2 member2]
//获取全部数据
zrange key start stop [WITHSCORES]
zrevrange key start stop [WITHSCORES]
//删除数据
zrem key member [member ...]
//按条件获取数据
zrangebyscore key min max [WITHSCORES] [LIMIT]
zrevrangebyscore key max min [WITHSCORES]
//条件删除数据
zremrangebyrank key start stop (按照排名)
zremrangebyscore key min max
//获取集合数据总量
zcard key
zcount key min max
//集合交、并操作
zinterstore destination numkeys key [key ...] 
zunionstore destination numkeys key [key ...]
```

(当用withscores时候，单数是member，双数是score)



#### 业务场景

票选广东十大杰出青年，各类综艺选秀海选投票

```
//获取数据对应的索引（排名）
zrank key member (默认由小到大)
zrevrank key member

//score值获取与修改
zscore key member 
zincrby key increment member
```



注意事项

score保存的数据也可以是一个双精度的double值，基于双精度浮点数的特征，可能会丢失精度，使用时候要慎重

sorted_set 底层存储还是基于set结构的，因此数据不能重复，如果重复添加相同的数据，score值将被反复覆盖，保留最后一次修改的结果



复杂权限排名

把权限记录在score里，如果有多个优先级权限，就记录在不同的位置上，但是要注意补0操作

如：xxyy xx代表国内国际订单，国际订单优先，yy代表订单数量，yy越小越优先

假设 a: 0013 b:1331 需要注意a需要在13之前加0来补充数据长度。



## 通用指令

### Key指令

```
//删除指定的key
del key
//获取key是否存在
exists key
//获取key的类型
type key

//为指定key设置有效期
expire key seconds
pexpire key milliseconds
expireat key timestamp	//用时间戳
pexpireat key milliseconds-timestamp
//获取key的有效时间
ttl key //不存在返回-2 存在，返回数值是-1， 设置了有效期返回市场
pttl key//用时间戳
//切换key从时效性转换为永久性
persist key

//查询key
keys pattern

```

keys * 查询所有 

keys it* 查询所有以it开头 

keys *heima 查询所有以heima结尾 

keys ??heima 查询所有前面两个字符任意，后面以heima结尾 

keys user:? 查询所有以user:开头，最后一个字符任意 

keys u[st]er:1 查询所有以u开头，以er:1结尾，中间包含一个字母，s或t

```
//为key改名
rename key newkey
renamenx key newkey //可以防止覆盖操作
//对所有key排序
sort
//其他key通用操作
help @generic
```



### 数据库通用指令

redis为每个服务提供有16个数据库，编号从0到15

```
//切换数据库
select index
//基本操作
quit
ping
echo message
//数据移动
move key db
//数据清除
dbsize
flushdb
flushall
```



## Jedis

Maven

```XML
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

连接jedis

```java
Jedis jedis = new Jedis("localhost", 6379);
jedis.set("name", "itheima");
jedis.get("name");
jedis.close();
```

Jedis 工具类

```Java
public static Jedis getJedis() {
    JedisPoolConfig jpc = new JedisPoolConfig();
    jpc.setMaxTotal(30);
    jpc.setMiaxIdle(10);
    String host = "127.0.0.1";
    int port = 6379;
    JedisPool jp = new JedisPool (jpc,host,port);
    return jp.getResource;
}
以后调用这个方法就可以用了
    Jedis jedis = new jedis.getJedis();
```



# Redis高级

## 文件持久化

利用永久性存储介质将数据进行保存，在特定的时间将保存的数据进行恢复的工作机制称为持久化。

### RDB

#### 普通执行

```
save
```

相关配置

dbfilename dump.rdb 说明：设置本地数据库文件名，默认值为 dump.rdb 经验：通常设置为dump-端口号.rdb

dir 说明：设置存储.rdb文件的路径 经验：通常设置成存储空间较大的目录中，目录名称data

rdbcompression yes 说明：设置存储至本地数据库时是否压缩数据，默认为 yes，采用 LZF 压缩 经验：通常默认为开启状态，如果设置为no，可以节省 CPU 运行时间，但会使存储的文件变大（巨大）

rdbchecksum yes 说明：设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程均进行 经验：通常默认为开启状态，如果设置为no，可以节约读写性过程约10%时间消耗，但是存储一定的数据损坏风险

注意：Save指令的执行会阻塞当前Redis服务器，直到当前RDB过程完成为止，有可能会造成长时间阻塞，线上环境不建议使用



#### 后台操作

```
bgsave
```

创建一个子进程，来进行save操作

![image-20210826212835077](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210826212835077.png)

配置：

stop-writes-on-bgsave-error yes 后台存储过程中如果出现错误现象，是否停止保存操作（通常默认为开启状态）



#### 自动执行save

```
save second changes
```

在 x秒内，有多少key发生变化，就保存

![image-20210826213301918](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210826213301918.png)

注意：

save配置要根据实际业务情况进行设置，频度过高或过低都会出现性能问题，结果可能是灾难性的 

save配置中对于second与changes设置通常具有互补对应关系，尽量不要设置成包含性关系 

save配置启动后执行的是bgsave操作



#### save vs bgsave

![image-20210826213451219](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210826213451219.png)



#### rdb特殊启动形式

全量复制

服务器运行过程中重启

```
debug reload
```

关闭服务器时指定保存数据 (默认情况下执行shutdown命令时，自动执行bgsave)

```
shutdown save
```



#### 优缺点

优点：

RDB是一个紧凑压缩的二进制文件，存储效率较高
RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景
RDB恢复数据的速度要比AOF快很多
应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。

缺点：

RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据
bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能
Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象

### AOF

rbd缺点：

存储数据量较大，效率较低 基于快照思想，每次读写都是全部数据，当数据量巨大时，效率非常低
大数据量下的IO性能较低
基于fork创建子进程，内存产生额外消耗
宕机带来的数据丢失风险



解决方案：

不写全数据，仅记录部分数据
降低区分数据是否改变的难度，改记录数据为记录操作过程
对所有操作均进行记录，排除丢失数据的风险



#### 配置：

```
是否开启AOF持久化功能，默认为不开启状态
appendonly yes|no
AOF写数据策略
appendfsync always|everysec|no
AOF持久化文件名，默认文件名未appendonly.aof，建议配置为appendonly-端口号.aof
appendfilename filename
AOF持久化文件保存路径，与RDB持久化文件保持一致即可
dir
```



#### AOF重写

随着命令不断写入AOF，文件会越来越大. AOF文件重写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程。简单说就是将对同一个数据的若干个条命令执行结果转化成最终结果数据对应的指令进行记录。

降低磁盘占用量，提高磁盘利用率
提高持久化效率，降低持久化写时间，提高IO性能
降低数据恢复用时，提高数据恢复效率



重写规则：

进程内已超时的数据不再写入文件

忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令 

如del key1、 hdel key2、srem key3、set key4 111、set key4 222等
对同一数据的多条写命令合并为一条命令 如lpush list1 a、lpush list1 b、 lpush list1 c 可以转化为：lpush list1 a b c。 

为防止数据量过大造成客户端缓冲区溢出，对list、set、hash、zset等类型，每条指令最多写入64个元素



```
手动重写
bgrewriteaof
```

![image-20210827105005967](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210827105005967.png)

```
自动重写
auto-aof-rewrite-min-size size //缓冲区增加size
auto-aof-rewrite-percentage percentage //缓冲区增加百分比
自动重写触发比对参数（ 运行指令info Persistence获取具体信息 ）
aof_current_size
aof_base_size
```



AOF工作原理

![image-20210827110156693](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210827110156693.png)



### AOF VS RDB

![image-20210827110306020](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210827110306020.png)





- 对数据非常敏感，建议使用默认的AOF持久化方案

  AOF持久化策略使用everysecond，每秒钟fsync一次。该策略redis仍可以保持很好的处理性能，当出现问题时，最多丢失0-1秒内的数据。

  注意：由于AOF文件存储体积较大，且恢复速度较慢

- 数据呈现阶段有效性，建议使用RDB持久化方案
  数据可以良好的做到阶段内无丢失（该阶段是开发者或运维人员手工维护的），且恢复速度较快，阶段点数据恢复通常采用RDB方案
  注意：利用RDB实现紧凑的数据持久化会使Redis降的很低，慎重总结：

- 综合比对
  RDB与AOF的选择实际上是在做一种权衡，每种都有利有弊
  如不能承受数分钟以内的数据丢失，对业务数据非常敏感，选用AOF
  如能承受数分钟以内的数据丢失，且追求大数据集的恢复速度，选用RDB
  灾难恢复选用RDB
  双保险策略，同时开启 RDB 和 AOF，重启后，Redis优先使用 AOF 来恢复数据，降低丢失数据的量



## 事务

redis事务就是一个命令执行的队列，将一系列预定义命令包装成一个整体（一个队列）。当执行时，一次性按照添加顺序依次执行，中间不会被打断或者干扰。

```
开启事务 设定事务的开启位置，此指令执行后，后续的所有指令均加入到事务中
multi
执行事务 设定事务的结束位置，同时执行事务。与multi成对出现，成对使用
exec
取消事务
discard
```



### 工作流程

![image-20210827111903399](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210827111903399.png)



### 注意事项

命令格式输入错误：整体事务中所有命令均不会执行。包括那些语法正确的命令

命令执行出现错误（语法没错）：能够正确运行的命令会执行，运行错误的命令不会被执行

手动进行事务回滚:

记录操作过程中被影响的数据之前的状态
	单数据：string
	多数据：hash、list、set、zset
设置指令恢复所有的被修改的项
	单数据：直接set（注意周边属性，例如时效）
	多数据：修改对应值或整体克隆复制



### 锁

多个客户端有可能同时操作同一组数据，并且该数据一旦被操作修改后，将不适用于继续操作

在操作之前锁定要操作的数据，一旦发生变化，终止当前操作(事务无法运行)

```
对 key 添加监视锁，在执行exec前如果key发生了变化，终止事务执行
watch key1 [key2……]
取消对所有 key 的监视
unwatch
```



分布式锁

```
setnx lock-key value
```

利用setnx命令的返回值特征，有值则返回设置失败，无值则返回设置成功
	对于返回设置成功的，拥有控制权，进行下一步的具体业务操作
	对于返回设置失败的，不具有控制权，排队或等待 操作完毕通过del操作释放锁

操作完毕通过del操作释放锁



### 死锁

某个用户操作时对应客户端宕机，且此时已经获取到锁

使用 expire 为锁key添加时间限定，到时不释放，放弃锁

```
expire lock-key second 
pexpire lock-key milliseconds
```

锁时间设定推荐：最大耗时*120%+平均网络延迟*110%



## Redis删除策略

已经过期的数据并没有直接被删除，需要等待Redis将他删除



### 数据删除策略

数据删除策略的目标:

在内存占用与CPU占用之间寻找一种平衡

1.定时删除

创建一个定时器，当key设置有过期时间，且过期时间到达时，由定时器任务立即执行对键的删除操作

优点：节约内存，到时就删除，快速释放掉不必要的内存占用

缺点：CPU压力很大，无论CPU此时负载量多高，均占用CPU，会影响redis服务器响应时间和指令吞吐量



2.惰性删除

数据到达过期时间，不做处理。等下次访问该数据时

​	如果未过期，返回数据

​	发现已过期，删除，返回不存在

优点：节约CPU性能，发现必须删除的时候才删除

缺点：内存压力很大，出现长期占用内存的数据



3.定期删除

利用过期数据占比的方式控制删除频度

每秒钟redis都会执行sever.hz次清理，会在所有的database执行activeExpireCycle() 对每个expires[]逐个进行检测，每次执行250ms/sever.hz(默认为10) 随机挑选W个key检测。

删除超时key，如果随机检测到的超时key超过百分之W * 25，就循环过程，如果小于等于W * 25，检查下一个databas.  W可以自己设置。如果执行到一般到期，下次继续执行。



### 逐出算法

Redis使用内存存储数据，在执行每一个命令前，会调用freeMemoryIfNeeded()检测内存是否充足，如果内存不够，就要删除一些数据获得空间。

```
最大可使用内存
memory

每次选取待删除数据的个数
maxmemory-samples

删除策略
maxmemory-policy
```

删除策略

检测易失数据（可能会过期的数据集server.db[i].expires ）
① volatile-lru：挑选最久未使用的数据淘汰
② volatile-lfu：挑选最少使用的数据淘汰
③ volatile-ttl：挑选将要过期的数据淘汰
④ volatile-random：任意选择数据淘汰
检测全库数据（所有数据集server.db[i].dict ）
⑤ allkeys-lru：挑选最久未使用的数据淘汰
⑥ allkeys-lfu：挑选最少使用的数据淘汰
⑦ allkeys-random：任意选择数据淘汰
放弃数据驱逐
⑧ no-enviction（驱逐）：禁止驱逐数据（redis4.0中默认策略），会引发错误OOM（Out Of Memory）



## 核心配置

### 服务器端设定

```
设置服务器以守护进程的方式运行
daemonize yes|no
绑定主机地址
bind 127.0.0.1
设置服务器端口号
port 6379
设置数据库数量
databases 16
```

日志配置

```
设置服务器以指定日志记录级别
loglevel debug|verbose|notice|warning
日志记录文件名
logfile 端口号.log
```

客户端配置

```
设置同一时间最大客户端连接数
maxclients 0
客户端闲置等待最大时长，达到最大值后关闭连接。如需关闭该功能，设置为 0
timeout 300
```

导入并加载指定配置文件信息，用于快速创建redis公共配置较多的redis实例配置文件，便于维护

```
include /path/server-端口号.conf
```



## 高级数据类型

### Bitmaps

用一个bit保存一个数据，适合做是/否

```
获取指定key对应偏移量上的bit值
getbit key offset
设置指定key对应偏移量上的bit值，value只能是1或0
setbit key offset value
对指定key按位进行交、并、非、异或操作，并将结果保存到destKey中
bitop op(and, or, not, xor) destKey key1 [key2...]
统计指定key中1的数量
bitcount key [start end]
```





### HyperLogLog

统计不重复数据的数量 使用了loglog算法

```
添加数据
pfadd key element [element ...]
统计数据
pfcount key [key ...]
合并数据
pfmerge destkey sourcekey [sourcekey...]
```

用于进行基数统计，不是集合，不保存数据，只记录数量而不是具体数据

核心是基数估算算法，最终数值存在一定误差

误差范围：基数估计的结果是一个带有 0.81% 标准错误的近似值

耗空间极小，每个hyperloglog key占用了12K的内存用于标记基数

pfadd命令不是一次性分配12K内存使用，会随着基数的增加内存逐渐增大

Pfmerge命令合并后占用的存储空间为12K，无论合并之前数据量多少



### GEO

地理位置信息

```
添加坐标点
geoadd key longitude latitude member [longitude latitude member ...]
获取坐标点
geopos key member [member ...]
计算坐标点距离
geodist key member1 member2 [unit]
根据坐标求范围内的数据
georadius key longitude latitude radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
格局点求范围内的数据
georadiusbymember key member radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
获取指定点对应的坐标hash值
geohash key member [member ...]
```



# Redis集群

多台计算机连接方案

## 主从复制

### 简介

主从复制即将master中的数据即时、有效的复制到slave中

特征：一个master可以拥有多个slave，一个slave只对应一个master

Master:

​	写数据

​	执行写操作时，将出现变化的数据自动同步到slave

​	读数据（可忽略）

slave:

​	读数据

​	写数据（禁止）



作用：

读写分离：master写、slave读，提高服务器的读写负载能力
负载均衡：基于主从结构，配合读写分离，由slave分担master负载，并根据需求的变化，改变slave的数量，通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量
故障恢复：当master出现问题时，由slave提供服务，实现快速的故障恢复
数据冗余：实现数据热备份，是持久化之外的一种数据冗余方式
高可用基石：基于主从复制，构建哨兵模式与集群，实现Redis的高可用方案



主从复制三个阶段

建立连接阶段（即准备阶段）

![image-20210827182258875](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210827182258875.png)

- 步骤1：设置master的地址和端口，保存master信息
  步骤2：建立socket连接
  步骤3：发送ping命令（定时器任务）
  步骤4：身份验证
  步骤5：发送slave端口信息
  至此，主从连接成功！

```
主从连接（slave连接master）
方式一：客户端发送命令
slaveof <masterip> <masterport>
方式二：启动服务器参数
redis-server -slaveof <masterip> <masterport>
方式三：服务器配置(在config里改)
slaveof <masterip> <masterport>
```

```
客户端发送命令 (从slave发出)
slaveof no one
slave断开连接后，不会删除已有数据，只是不再接受master发送的数据
```

授权访问

```
master客户端发送命令设置密码
requirepass <password>

master配置文件设置密码
config set requirepass <password>
config get requirepass

slave客户端发送命令设置密码
auth <password>

slave配置文件设置密码
masterauth <password>

slave启动服务器设置密码
redis-server –a <password>
```



数据同步阶段 + 命令传播阶段

在slave初次连接master后，复制master中的所有数据到slave

将slave的数据库状态更新成master当前的数据库状态

- 步骤1：请求同步数据 
- 步骤2：创建RDB同步数据 
- 步骤3：恢复RDB同步数据 
- 步骤4：请求部分同步数据 
- 步骤5：恢复部分同步数据



![image-20210827182243518](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210827182243518.png)



Master:

如果master数据量巨大，数据同步阶段应避开流量高峰期，避免造成master阻塞，影响业务正常执行

复制缓冲区大小设定不合理，会导致数据溢出。如进行全量复制周期太长，进行部分复制时发现数据已经存在丢失的情况，必须进行第二次全量复制，致使slave陷入死循环状态。

master单机内存占用主机内存的比例不应过大，建议使用50%-70%的内存，留下30%-50%的内存用于执行bgsave命令和创建复制缓冲区



slave:

为避免slave进行全量复制、部分复制时服务器响应阻塞或数据不同步，建议关闭此期间的对外服务

```
slave-serve-stale-data yes|no
```

数据同步阶段，master发送给slave信息可以理解master是slave的一个客户端，主动向slave发送命令

多个slave同时对master请求数据同步，master发送的RDB文件增多，会对带宽造成巨大冲击，如果master带宽不足，因此数据同步需要根据业务需求，适量错峰

slave过多时，建议调整拓扑结构，由一主多从结构变为树状结构，中间的节点既是master，也是slave。注意使用树状结构时，由于层级深度，导致深度越高的slave与最顶层master间数据同步延迟较大，数据一致性变差，应谨慎选择



复制缓冲区

是一个先进先出（FIFO）的队列，用于存储服务器执行过的命令，每次传播命令，master都会将传播的命令记录下来，并存储在复制缓冲区

组成

- 偏移量
- 字节值

通过offset区分不同的slave当前数据传播的差异

master记录已发送的信息对应的offset

slave记录已接收的信息对应的offset





### 工作流程

![image-20210827185023801](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210827185023801.png)



### 心跳机制

进入命令传播阶段候，master与slave间需要进行信息交换，使用心跳机制进行维护，实现双方连接保持在线

master心跳：

- 指令：PING
  周期：由repl-ping-slave-period决定，默认10秒
  作用：判断slave是否在线
  查询：INFO replication

slave心跳任务

- 指令：REPLCONF ACK {offset}
  周期：1秒
  作用1：汇报slave自己的复制偏移量，获取最新的数据变更指令
  作用2：判断master是否在线

当slave多数掉线，或延迟过高时，master为保障数据稳定性，将拒绝所有信息同步操作

slave数量少于2个，或者所有slave的延迟都大于等于8秒时，强制关闭master写功能，停止数据同步

```
min-slaves-to-write 2 
min-slaves-max-lag 8
```

```
获取延迟和数量
REPLCONF ACK
```



### 常见问题

频繁的全量复制

master关机的时候会把runid和offset保存到RDB文件中，master重启后加载RDB文件，恢复数据

作用：本机保存上次runid，重启后恢复该值，使所有slave认为还是之前的master



网络环境不佳，出现网络中断，slave不提供服务

原因：复制缓冲区过小，断网后slave的offset越界，触发反复全量复制

解决方案：修改复制缓冲区大小

最优复制缓冲区空间 = 2 * second（重连平均时长） * write_size_per_second（平均每秒产生命令数据总量）



频繁的网络中断

问题：master的CPU占用过高 或 slave频繁断开连接

原因：

slave每1秒发送REPLCONF ACK命令到master
当slave接到了慢查询时（keys * ，hgetall等），会大量占用CPU性能
master每1秒调用复制定时函数replicationCron()，比对slave发现长时间没有进行响应

结果：master各种资源（输出缓冲区、带宽、连接等）被严重占用

解决方案: 通过设置合理的超时时间，确认是否释放slave

```
repl-timeout
```



问题现象:slave与master连接断开

问题原因:

- master发送ping指令频度较低
- master设定超时时间较短
- ping指令在网络中存在丢包

解决方案

- 提高ping指令发送的频度

```
repl-ping-slave-period 
超时时间repl-time的时间至少是ping指令频度的5到10倍
```



问题: 数据不一致

多个slave获取相同数据不同步

原因: 

网络信息不同步，数据发送有延迟

解决方案:

优化主从间的网络环境，通常放置在同一个机房部署，如使用阿里云等云服务器时要注意此现象

监控主从节点延迟（通过offset）判断，如果slave延迟过大，暂时屏蔽程序对该slave的数据访问

```
slave-serve-stale-data yes|no
```

开启后仅响应info、slaveof等少数命令（慎用，除非对数据一致性要求很高）



## 哨兵模式

当出现故障时通过投票机制选择新的master并将所有slave连接到新的master



### 哨兵的作用

1.监控： 不断的检查master和slave是否正常运行。 master存活检测、master与slave运行情况检测

2.通知（提醒）：当被监控的服务器出现问题时，向其他（哨兵间，客户端）发送通知。

3.自动故障转移: 断开master与slave连接，选取一个slave作为master，将其他slave连接到新的master，并告知客户端新的服务器地址



### 配置哨兵

redis-sentinel sentinel-端口号.conf

![image-20210828103322172](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828103322172.png)



### 哨兵工作原理

1. 监控

   同步各个节点的状态信息

   ​	获取各个sentinel的状态（是否在线）

   ​	获取master的状态 (info指令)

   ​		runid
   ​		role：master

   ​		各个slave的详细信息

   ​	获取所有slave的状态

   ​		runid
   ​		role：slave
   ​		master_host、master_port
   ​		offset...

   ​		![image-20210828104108639](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828104108639.png)

2. 通知

   ![image-20210828104500132](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828104500132.png)

3. 故障转移

   ![image-20210828104919068](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828104919068.png)

   超过半数以上的哨兵认为服务器挂了，那就是挂了

   

   服务器列表中挑选备选master的原则：

   ​	在线的

   ​	响应快的

   ​	与原master断开时间短的

   ​	优先原则

   ​		优先级

   ​		offset(越低越好)

   ​		runid

   ​	发送指令

   ​		向新的master发送slaveof no one

   ​		向其他slave发送slaveof 新masterIP端口



## 集群

集群就是使用网络将若干台计算机联通起来，并提供统一的管理方式，使其对外呈现单机的服务效果



### 数据存储设计

通过算法设计，计算出key应该保存的位置

将所有的存储空间计划切割成16384份，每台主机保存一部分

​	每份代表的是一个存储空间，不是一个key的保存空间

将key按照计算出的结果放到对应的存储空间



有新机器的时候，把一些分割的空间（槽）转换到新的槽就可以了

当新的数据加入的时候，会给一个服务器，如果不是正确位置，服务器会转发到正确的服务器（服务器内有一张表记录数据存放的位置）



Cluster配置

```
添加节点
cluster-enabled yes|no
cluster配置文件名，该文件属于自动生成，仅用于快速查找文件并查询文件内容
cluster-config-file <filename>
节点服务响应超时时间，用于判定该节点是否下线或切换为从节点
cluster-node-timeout <milliseconds>
master连接的slave最小数量
cluster-migration-barrier <count>
查看集群节点信息
cluster nodes
进入一个从节点 redis，切换其主节点
cluster replicate <master-id>
发现一个新节点，新增主节点
cluster meet ip:port
忽略一个没有solt的节点
cluster forget <id>
手动故障转移
cluster failover
```

```
添加节点
redis-trib.rb add-node
删除节点
redis-trib.rb del-node
重新分片
redis-trib.rb reshard
```



解决方案

缓存预热

现象：服务器启动后迅速宕机

原因：请求数量较高，主从之间数据吞吐量较大，数据同步操作频度较高

解决方案：

前置准备工作：
1.日常例行统计数据访问记录，统计访问频度较高的热点数据
2.利用LRU数据删除策略，构建数据留存队列 例如：storm与kafka配合 

准备工作：
1.将统计结果中的数据分类，根据级别，redis优先加载级别较高的热点数据
2.利用分布式多服务器同时进行数据读取，提速数据加载过程
3.热点数据主从同时预热 

实施：
1.使用脚本程序固定触发数据预热过程
2.如果条件允许，使用了CDN（内容分发网络），效果会更好



总结：缓存预热就是系统启动前，提前将相关的缓存数据直接加载到缓存系统。避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！





缓存雪崩

数据库服务器崩溃

1.系统平稳运行过程中，忽然数据库连接量激增
2.应用服务器无法及时处理请求
3.大量408，500错误页面出现
4.客户反复刷新页面获取数据
5.数据库崩溃
6.应用服务器崩溃
7.重启应用服务器无效
8.Redis服务器崩溃
9.Redis集群崩溃
10.重启数据库后再次被瞬间流量放倒



问题本质：

1.在一个较短的时间内，缓存中较多的key集中过期
2.此周期内请求访问过期的数据，redis未命中，redis向数据库获取数据
3.数据库同时接收到大量的请求无法及时处理
4.Redis大量请求被积压，开始出现超时现象
5.数据库流量激增，数据库崩溃
6.重启后仍然面对缓存中无数据可用
7.Redis服务器资源被严重占用，Redis服务器崩溃
8.Redis集群呈现崩塌，集群瓦解
9.应用服务器无法及时得到数据响应请求，来自客户端的请求数量越来越多，应用服务器崩溃
10.应用服务器，redis，数据库全部重启，效果不理想



问题原因：

短时间范围内
大量key集中过期



解决方案：

1.更多的页面静态化处理
2.构建多级缓存架构 Nginx缓存+redis缓存+ehcache缓存
3.检测Mysql严重耗时业务进行优化 对数据库的瓶颈排查：例如超时查询、耗时较高事务等
4.灾难预警机制 监控redis服务器性能指标
	CPU占用、CPU使用率
	内存容量
	查询平均响应时间
	线程数
5.限流、降级 短时间范围内牺牲一些客户体验，限制一部分请求访问，降低应用服务器压力，待业务低速运转后再逐步放开访问



1.LRU与LFU切换
2.数据有效期策略调整
	根据业务数据有效期进行分类错峰，A类90分钟，B类80分钟，C类70分钟
	过期时间使用固定时间+随机值的形式，稀释集中到期的key的数量
3.超热数据使用永久key
4.定期维护（自动+人工） 对即将过期数据做访问量分析，确认是否延时，配合访问量统计，做热点数据的延时
5.加锁 慎用！

缓存雪崩就是瞬间过期数据量太大，导致对数据库服务器造成压力。如能够有效避免过期时间集中，可以有效解决雪崩现象的出现（约40%），配合其他策略一起使用，并监控服务器的运行数据，根据运行记录做快速调整。



缓存击穿

1.系统平稳运行过程中
2.数据库连接量瞬间激增
3.Redis服务器无大量key过期
4.Redis内存平稳，无波动
5.Redis服务器CPU正常
6.数据库崩溃

问题排查:

1.Redis中某个key过期，该key访问量巨大
2.多个数据请求从服务器直接压到Redis后，均未命中
3.Redis在短时间内发起了大量对数据库中同一数据的访问

分析:

单个key高热数据过期

解决方案：

1.预先设定 以电商为例，每个商家根据店铺等级，指定若干款主打商品，在购物节期间，加大此类信息key的过期时长 注意：购物节不仅仅指当天，以及后续若干天，访问峰值呈现逐渐降低的趋势
2.现场调整 监控访问量，对自然流量激增的数据延长过期时间或设置为永久性key
3.后台刷新数据 启动定时任务，高峰期来临之前，刷新数据有效期，确保不丢失
4.二级缓存 设置不同的失效时间，保障不会被同时淘汰就行
5.加锁 分布式锁，防止被击穿，但是要注意也是性能瓶颈，慎重！

缓存击穿就是单个高热数据过期的瞬间，数据访问量较大，未命中redis后，发起了大量对同一数据的数据库访问，导致对数据库服务器造成压力。应对策略应该在业务数据分析与预防方面进行，配合运行监控测试与即时调整策略，毕竟单个key的过期监控难度较高，配合雪崩处理策略即可。



缓存穿透

1.系统平稳运行过程中
2.应用服务器流量随时间增量较大
3.Redis服务器命中率随时间逐步降低
4.Redis内存平稳，内存无压力
5.Redis服务器CPU占用激增
6.数据库服务器压力激增
7.数据库崩溃

问题排查:

1.Redis中大面积出现未命中
2.出现非正常URL访问

问题分析：

获取的数据在数据库中也不存在，数据库查询未得到对应数据
Redis获取到null数据未进行持久化，直接返回
下次此类数据到达重复上述过程
出现黑客攻击服务器

解决方案：

1.缓存null 对查询结果为null的数据进行缓存（长期使用，定期清理），设定短时限，例如30-60秒，最高5分钟
2.白名单策略
	提前预热各种分类数据id对应的bitmaps，id作为bitmaps的offset，相当于设置了数据白名单。当加载正常数据时，放行，加载异常数据时直接拦截（效率偏低）
	使用布隆过滤器（有关布隆过滤器的命中问题对当前状况可以忽略）
3.实施监控 实时监控redis命中率（业务正常范围时，通常会有一个波动值）与null数据的占比
非活动时段波动：通常检测3-5倍，超过5倍纳入重点排查对象
活动时段波动：通常检测10-50倍，超过50倍纳入重点排查对象 根据倍数不同，启动不同的排查流程。然后使用黑名单进行防控（运营）
4.key加密 问题出现后，临时启动防灾业务key，对key进行业务层传输加密服务，设定校验程序，过来的key校验 例如每天随机分配60个加密串，挑选2到3个，混淆到页面数据id中，发现访问key不满足规则，驳回数据访问



缓存击穿访问了不存在的数据，跳过了合法数据的redis数据缓存阶段，每次访问数据库，导致对数据库服务器造成压力。通常此类数据的出现量是一个较低的值，当出现此类情况以毒攻毒，并及时报警。应对策略应该在临时预案防范方面多做文章。
无论是黑名单还是白名单，都是对整体系统的压力，警报解除后尽快移除。



性能指标监控

性能指标：Performance

![image-20210828131810053](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828131810053.png)内存指标：Memory

![image-20210828131814797](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828131814797.png)基本活动指标：Basic activity

![image-20210828131817677](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828131817677.png)持久性指标：Persistence

![image-20210828131822461](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828131822461.png)错误指标：Error

![image-20210828131824957](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828131824957.png)



监控：

benchmark 压力测试

![image-20210828132015839](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828132015839.png)

```
命令
redis-benchmark [-h ] [-p ] [-c ] [-n <requests]> [-k ]
redis-benchmark  50个连接，10000次请求对应的性能
redis-benchmark -c 100 -n 5000 100个连接，5000次请求对应的性能
```

monitor

```
打印服务器调试信息
monitor
```



showlong

```
showlong [operator](get,len,reset)
slowlog-log-slower-than 1000 #设置慢查询的时间下线，单位：微妙 
slowlog-max-len 100 #设置慢查询命令对应的日志显示长度，单位：命令数
```

