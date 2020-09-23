# Redis 笔记总结

[TOC]



## 前言 

```
> 该文档为 Redis 的总结性文档 , 包含Redis 常用概念 , 常见用法 , 底层算法等项目常用知识点

```

### >>> 关注我

```
> 如果感觉不错记得给个Star哦

> Github : https://github.com/black-ant
> Blog : http://www.antblack.xyz/
> CSDN : https://blog.csdn.net/zzg19950824
```



## 一  Redis 基础



### 1 . 1 数据结构

```
> 字符串
> 字典 hash
> 列表 list
> 集合 set
> 有序结合 SortedSet

> HyperLogLog
> Geo
> BitMap
```

### 1 . 2 Redis 客户端

```
> Redisson
> Jedis
> lettuce
> 
```

### 1 . 3 Redis 实现分布式锁

```
需要考虑的方面
> 正确的获取锁 : 使用nx 参数
> 正确的释放锁 : Lua 脚本 ,比对锁持久的是不是自己
> 超时的自动释放锁 : expire 参数 ,过期机制实现超时释放
> 未获得锁的等待机制 : sleep 或者 订阅 pub/sub机制
> 锁超时的处理 : 考虑告警 + 后台线程自动续锁的超时时间
> Redis 分布式锁丢失问题
```

### 1 . 4 Redis 单线程的原因

```
>  Redis 是非阻塞 IO ，多路复用
	- Redis 内部使用文件事件处理器 file event handler，这个文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型。
	- 采用 IO 多路复用机制同时监听多个 Socket，根据 Socket 上的事件来选择对应的事件处理器进行处理
	
> 文件时间处理器
	- 多个 Socket
	- IO 多路复用程序
	- 文件事件分派器
	- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

> Redis 的线程模型

> 单线程提高 CPU 性能

```

### 1 .5 Redis 性能高的原因

```
> C 语言实现
> 纯内存操作
> 基于非阻塞的 IO 多路复用机制
> 单线程，避免了多线程的频繁上下文切换问题
	|- 4.0 开始添加异步线程 , 惰性删除和异步AOF
> 丰富的数据结构
```

### 1 . 6 Redis 持久化方式

```
> 【全量】RDB 持久化，是指在指定的时间间隔内将内存中的数据集快照写入磁盘
	|- fork 一个子线程 
	|- 将数据集写入临时文件
	|- 写入成功后 ,替换之前的文件 , 用二进制压缩存储
	Y- 灵活设置备份频率和周期。
	Y- 非常适合冷备份
	Y- 性能最大化
	Y- 恢复更快
	N- 不易保存数据完整性 , 数据集较大得时候会短时间阻塞
> 【增量】AOF持久化，以日志的形式记录服务器所处理的每一个写、删除操作
	|- 查询操作不记录
	Y- 带来更高的数据安全性，即数据持久性
	Y- 由于该机制对日志文件的写入操作采用的是 append 模式，因此在写入过程中即使出现宕机现象，也不会破坏日志文件中已经存在的内容
	Y- 如果 AOF 日志过大，Redis 可以自动启用 rewrite 机制。即使出现后台重写操作，也不会影响客户端的读写
	Y- AOF 包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作
	N- 对于数据集 , AOF 的要大于 RDB
	N- AOF 的数据安全性不靠谱
	N- AOF 的效率要低于 RDB
```

### 1 . 7 Redis 实现分布式限流

```

```

### 1 . 8 Redis Pipelinging

```
管道（pipelining）

一次请求/响应服务器能实现处理新的请求即使旧的请求还未被响应。这样就可以将多个命令发送到服务器，而不用等待回复，最后在一个步骤中读取该答复。
> Redis Pipelining 是 Redis Client 实现的功能
> 所以，Pipeline 的作用，是避免每发一个请求，就阻塞等待这个请求的结果。

> 未使用 Pipeline 时，那么整个执行的顺序是，req1->resp1->req2->resp2->req3->resp3 。
> 在使用 Pipeline 时，那么整个执行的顺序是，[req1,req2,req3] 一起发给 Redis Server ，而 Redis Server 收到请求后，一个一个请求进行执行，然后响应，不会进行什么特殊处理。而 Client 在收到 resp1,resp2,resp3 后，进行响应给业务上层。
```



### 1 . 9  Redis key 过期策略

```java
> Redis 提供三种过期策略 :// 该过期策略非互斥
	- 被动删除：当读/写一个已经过期的 key 时，会触发惰性删除策略，直接删除掉这个过期 key
        : 过期删除返回null ,对CPU友好 , 对内存不友好 , 可能出现内存泄漏
	- 主动删除：由于惰性删除策略无法保证冷数据被及时删掉，所以 Redis 会定期主动淘汰一批已过期的 key 
        : 服务器会定期对主机的资源和状况进行检查,redis主要有以下的检查工作
            - 更新服务器的各类统计信息，比如时间、内存占用、数据库占用情况等
            - 清理数据库中的过期键值对。
            - 对不合理的数据库进行大小调整
            - 关闭和清理连接失效的客户端
            - 尝试进行 AOF 或 RDB 持久化操作
            - 如果服务器是主节点的话，对附属节点进行定期同步
            - 如果处于集群模式的话，对集群进行定期同步和连接测试
	- 主动删除：当前已用内存超过 maxmemory 限定时，触发主动清理策略
	

> 主动删除的策略模式(淘汰策略)
- volatile-lru : 只对设置了过期时间的key进行LRU（默认值）
- volatile-ttl : 删除即将过期的
- volatile-random : 随机删除即将过期key
- allkeys-lru : 删除lru算法的key
- allkeys-random : 随机删除
- no-enviction 【默认策略】 : 永不过期
- volatile-lfu : 对有过期时间的key采用LFU淘汰算法
- allkeys-lfu : 对所有的key 进行 LFU淘汰算法
```

## 二  深入知识点

### 2 . 1 Hash 槽

```

```



## 三  Redis 深入算法

###  LFU

```

```

## 四  Redis 集群

```
• 1、Redis Sentinel
• 2、Redis Cluster
• 3、Twemproxy
• 4、Codis
• 5、客户端分片


```

### Redis 哨兵模式

```

```



## 五  其他知识点

### 5 . 1 基础知识点

```

```





### 主从同步

```java
主从复制的作用 : 
读写分离：适用于读多写少的应用，增加多个从机，提高读的速度，提高程序并发
数据容灾恢复：从机复制主机的数据，相当于数据备份，如果主机数据丢失，那么可以通过从机存储的数据进行恢复。
高并发、高可用集群实现的基础：在高并发的场景下，就算主机挂了，从机可以进行主从切换，从机自动成为主机对外提供服务。


「1」slave发送psync，由于是第一次复制，不知道master的runid，自然也不知道offset，所以发送psync ？ -1
「2」master收到请求，发送master的runid和offset给从节点。
「3」从节点slave保存master的信息
「4」主节点bgsave保存rdb文件
「5」主机点发送rdb文件
并且在「④」和「⑤」的这个过程中产生的数据，会写到复制缓冲区repl_back_buffer之中去。
「6」主节点发送上面两个步骤产生的buffer到从节点slave
「7」从节点清空原来的数据，如果它之前有数据，那么久会清空数据
「8」从节点slave把rdb文件的数据装载进自身。


// 全量复制的开销 :
「1」bgsave时间
「2」rdb文件网络传输时间
「3」从节点清空数据的
「4」从节点加载rdb的时间
「5」可能的aof重写时间，这是针对从节点，例如开启了aof之后，从节点添加buffer数据时候，可能需要aof重写
    
// 

```

<img src="C:\Users\10169\OneDrive\笔记文档\图片文件\缓存\redis01.png" style="zoom:200%;" />

<img src="C:\Users\10169\OneDrive\笔记文档\图片文件\缓存\redis02.png" style="zoom:200%;" />

### 主从部分复制

```
「1」假设发送网络抖动或者别的情况，暂时失去了连接
「2」这个时候，master还在继续往buffer里面写数据
「3」slave重新连接上了master
「4」slave向master发送自己的offset和runid
「5」master判断slave的offset是否在buffer的队列里面，如果是，那就返回continue给slave，否则需要进行全量复制（因为这说明已经错过了很多数据了）
「6」master发送从slave的offset开始到缓冲区队列结尾的数据给slave

```

<img src="C:\Users\10169\OneDrive\笔记文档\图片文件\缓存\redis03.png" style="zoom:150%;" />

## 六 . Spring Redis 使用

### 配置序列化方式

```java
@Configuration
public class RedisConfig {


    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);

        // 使用Jackson2JsonRedisSerialize替换默认序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 设置key和value的序列化规则
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.afterPropertiesSet();

        return redisTemplate;
    }
}


// example 2 
@SuppressWarnings("rawtypes")
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory){
	RedisTemplate redisTemplate = new RedisTemplate();
        RedisSerializer stringSerializer = new StringRedisSerializer();
        redisTemplate.setConnectionFactory(factory);
        redisTemplate.setKeySerializer(stringSerializer);
        redisTemplate.setValueSerializer(stringSerializer);
        redisTemplate.setHashKeySerializer(stringSerializer);
        redisTemplate.setHashValueSerializer(stringSerializer);
        return redisTemplate;
    }
```



## Redis HashMap 操作

```

```

## SpringBoot Redis 知识点



### 基本使用

```
> RedisTemplate 不可直接种注入 ,StringRedisTemplate 可以直接注入使用

> StringRedisTemplate / RedisTemplate 均可以自定义Bean使用
```





## 问题及解决方案

### 大量Key 需要同一时间过期

```
> 调大HZ 参数 : 一秒钟之内 , 后台任务期望被调用的次数 ,默认为10 , hz 调大将会提高 Redis 主动淘汰的频率，如果你的 Redis 存储中包含很多冷数据占用内存过大的话，可以考虑将这个值调大

> 
```