什么是非关系型数据库？



### 1.redis是单线程怎么实现高并发的  探探2020

### 2.redis string 和zset实现  探探2020

### 3.redis I/O多路=复用如何实现 探探2020

4.redis基本类型  字节

5.redis 的分布式锁

6.[《基于REDIS实现延时任务》](https://juejin.im/post/5caf45b96fb9a0688b573d6c)

7.redis数据结构的底层实现 腾讯

8.redis 如何实现高可用 腾讯

9. zset 延时队列怎么实现的 字节2020

10. redis 持久化  字节

11. Redis 的 ZSET 怎么实现的？ 尽量介绍的全一点，跳跃表加哈希表以及压缩链表  字节

    作者：oscarwin
    链接：https://www.nowcoder.com/discuss/336659
    来源：牛客网

       Redis 的 ZSET 做排行榜时，如果要实现分数相同时按时间顺序排序怎么实现？ 说了一个将 score 拆成高 32 位和低 32 位，高 32 位存分数，低 32 位存时间的方法。问还有没有其他方法，想不出了 

    ### redis数据类型有哪一些？

**Redis 支持的五种数据类型：string   hash   list    set   zset**

string  字符串  一个key 对应一个value, string 是二进制安全的，意思是 redis 的 string 可以包含任何数据**。比如jpg图片或者序列化的对象。**

string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

hash  哈希：一个键值对组合。特别适合与存储对象

list 列表 : 简单的字符串列表 按照插入顺序排序 可以插入到头部或者尾部  lpush runoob  redis

set  集合  ：string类无序集合，集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。元素不允许重复。集合中最大的成员数为  232  - 1(4294967295, 每个集合可存储40多亿个成员)。

sadd   

zset (sorted set) 有序集合

zadd 命令 添加元素

### redis各种数据类的使用场景有哪一些？

| 类型                 | 简介                                                   | 特性                                                         | 场景                                                         |
| :------------------- | :----------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| String(字符串)       | 二进制安全                                             | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M | ---                                                          |
| Hash(字典)           | 键值对集合,即编程语言中的Map类型                       | 适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) | 存储、读取、修改用户属性，秒杀订单                           |
| List(列表)           | 链表(双向链表)                                         | 增删快,提供了操作某一段元素的API                             | 1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列          |
| Set(集合)            | 哈希表实现,元素不重复                                  | 1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了**求交集、并集、差集**等操作 | 1、共同好友 2、**利用唯一性,统计访问网站的所有独立ip** 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐 |
| Sorted Set(有序集合) | 将Set中的元素增加一个权重参数score,元素按score有序排列 | 数据插入集合时,已经进行天然排序                              | 1、排行榜 2、带权重的消息队列                                |

[FK](javascript:;)  FK 429***967@qq.com

### 4.缓存雪崩

我们都知道 Redis 不可能把所有的数据都缓存起来（内存昂贵且有限），所以 Redis
需要对数据设置过期时间，并采用的是惰性删除+定期删除两种策略对过期键删除。
如果缓存数据设置的过期时间是相同的，并且 Redis 恰好将这部分数据全部删光了。
这就会导致在这段时间内，这些缓存同时失效，全部请求到数据库中。这就是缓存雪崩：
Redis 挂掉了，请求全部走数据库。 缓存雪崩如果发生了，很可能就把我们的数据库搞
垮，导致整个服务瘫痪。  

#### 如何解决缓存雪崩？  

在缓存过期时间加上随机值，这样就可以防止在同一时间出现大面积的过期。

#### redis挂了，请求全部落到数据库？

（1）事发前避免： 实现redis 的高可用（主从架构+Sentinel或者redis Cluster）,尽量避免redis挂掉。

 （2）事发中： redis如果挂掉，那么可以何止本地缓存（ehcache）+限流（hystrix）,尽量避免数据库被干掉，起码保证服务正常运行。

（3）事发后：redis 持久化，重启后自动从磁盘加载数据，快速恢复缓存数据。

### 5.什么是缓存穿透？ 怎么解决？

缓存穿透只查询一个不存在的数据，缓存不命中的时候从数据库查询，查不到数据则不写入缓存，这将导致查询这个不存在的数据的时候每次都去数据库查询。造成缓存穿透。

例如查询ID.数据库中的都是证书 ，每次查询用负数，查询不到，又不会写入缓存。

##### 缓存穿透怎么解决？

方法1：

由于请求的参数不合法，（每次都请求不存在的参数），于是我们可以使用布隆过滤
器(BloomFilter) 或者压缩 filter 提前拦截，不合法就不让这个请求到数据库层；  

方法2： 

最简单粗暴的方法如果一个查询返回的数据为空（不管是数据不存在，还是系统故
障），我们就把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。  

### 6.缓存与数据库双写一致性问题

对于读操作  是没有问题的。

流程是这样的如果我们的数据在缓存里边有，那么就直接取缓存的。如果缓存里没有
我们想要的数据，我们会先去查询数据库，然后将数据库查出来的数据写到缓存中。最后
将数据返回给请求。  

但是对于写。

例如库存是999，缓存是1000

1、从理论上说，只要我们设置了合理的键的过期时间，我们就能保证缓存和数据库的数据
最终是一致的。因为只要缓存数据过期了，就会被删除。随后读的时候，因为缓存里没
有，就可以查数据库的数据，然后将数据库查出来的数据写入到缓存中。除了设置过期时
间，我们还需要做更多的措施来尽量避免数据库与缓存处于不一致的情况发生。  

2.新增、更改、删除数据库操作时同步更新 redis，**可以使用事务机制来**保证数据的一致
性。  

### 7.redis的持久化方式有哪一些？

1、RDB（Redis Database）：指定的**时间间隔能对你的数据进行快照存储**；
2、AOF（Append Only File）：**每一个收到的写命令都通过 write 函数追加到文件中**。  

### 8.redis 分布式锁

#### redis怎么实现分布式锁？

redis 分布式锁其实就是在系统里面占一个“坑”，其他程序也要占“坑”的时候，占
用成功了就可以继续执行，失败了就只能放弃或稍后重试。
占坑一般使用 **setnx(set if not exists)**指令，只允许被一个程序占有，使用完调
用 del 释放锁。  

### 9.内存优化

#### redis 怎么做内存优化？

尽可能使用**散列表（hash）**，散列表（是说散列表里面存储的数少）使用的内存非常
小，所以你应该尽可能的将你的数据模型抽象到一个散列表里面。
比如你的 web 系统中有一个用户对象，不要为这个用户的名称，姓氏，邮箱，密码设
置单独的 key，而是应该把这个用户的所有信息存储到一张散列表里面。  

### 10.redis的淘汰策略

1.volatile-lru

 从已设置过期时间的数据集（server. db[i]. expires）中挑选最近最少使
用的数据淘汰。  

2.volatile-ttl

从已设置过期时间的数据集（server. db[i]. expires）中挑选将要过期的
数据淘汰。  

3.volatile-random

已设置过期时间的数据集（server. db[i]. expires）中挑选将要过期的
数据淘汰。

4.allkeys-lru

：从数据集（server. db[i]. dict）中挑选最近最少使用的数据淘汰  

5.allkeys-random

从数据集（server. db[i]. dict）中任意选择数据淘汰  

6.no-enviction

禁止驱逐数据  



### 11.redis的性能问题

1.**主服务器写内存快照**，会阻塞主线程的工作，当快照比较大时，对性能影响非常大的会简短性暂停服务。

所以主服务器最好不要写内存快照。

2.**redis主从复制性能问题，**为了主从复制速度和连接稳定性，主从库最好在同一个局域网内。



### 12.redis的发布订阅

https://www.runoob.com/redis/redis-pub-sub.html

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 客户端可以订阅任意数量的频道。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![](.\img\pubsub1.png)

![pubsub2](\img\pubsub2.png)

![订阅频道](.\img\订阅频道.png)

​       1.打开串口订阅redisChat

![发布消息](.\img\发布消息.png)

在redisChat发布消息

![显示接收到的消息](.\img\显示接收到的消息.png)

在suscribe redissChat 窗口显示发布的消息。

#### Redis 发布订阅命令



下表列出了 redis 发布订阅常用命令：

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [PSUBSCRIBE pattern [pattern ...\]](https://www.runoob.com/redis/pub-sub-psubscribe.html)  订阅一个或多个符合给定模式的频道。 |
| 2    | [PUBSUB subcommand [argument [argument ...\]]](https://www.runoob.com/redis/pub-sub-pubsub.html)  查看订阅与发布系统状态。 |
| 3    | [PUBLISH channel message](https://www.runoob.com/redis/pub-sub-publish.html)  将信息发送到指定的频道。 |
| 4    | [PUNSUBSCRIBE [pattern [pattern ...\]]](https://www.runoob.com/redis/pub-sub-punsubscribe.html)  退订所有给定模式的频道。 |
| 5    | [SUBSCRIBE channel [channel ...\]](https://www.runoob.com/redis/pub-sub-subscribe.html)  订阅给定的一个或多个频道的信息。 |
| 6    | [UNSUBSCRIBE [channel [channel ...\]]](https://www.runoob.com/redis/pub-sub-unsubscribe.html)  指退订给定的频道。 |

13.布隆过滤器

  url黑名单怎么做？ 布隆过滤器 bitmap

### 14.redis的事务

redis提供的不是严格的事务，redis 只保证串行执行命令，并且能保证全部执行，但是执行命令失败不会回滚。

而是继续执行下去。