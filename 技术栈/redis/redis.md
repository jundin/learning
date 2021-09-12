1. 数据量太大，一个机器放不了
2. 数据的索引（B+ Tree），一个机器内存放不下
3. 访问量（读写混合），一个服务器承受不了

网站80%的情况都在读，每次都去查询数据库就十分的麻烦。

缓存 + MySQL + 垂直拆分（读写分离）

分库分表 + 水平拆分 + MySQL集群
大数据量高性能（redis 一秒写8万次，读取11万）

## redis （Remote Dictionary Server）
1. 内存存储、持久化
2. 效率高，可以用于高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计时器，计数器（浏览量）

### 在 Redis 中，常用的 5 种数据结构和应用场景如下：
- String：缓存、计数器、分布式锁等。
- List：链表、队列、微博关注人时间轴列表等。
- Hash：用户信息、Hash 表等。
- Set：去重、赞、踩、共同好友等。
- Zset：访问量排行榜、点击量排行榜等。

### Redis为什么单线程还这么快
- 完全基于内存操作
- 使用单线程模型来处理客户端的请求，避免了上下文的切换
- IO 多路复用机制
- 自身使用 C 语言编写，有很多优化机制，比如动态字符串 sds

### 听说 redis 6.0之后又使用了多线程，不会有线程安全的问题吗？
不会

其实redis还是使用单线程模型来处理客户端的请求，只是使用多线程来处理数据的读写和协议解析，执行命令还是使用单线程，所以是不会有线程安全的问题。

之所以加入了多线程因为 redis 的性能瓶颈在于网络IO而非CPU，使用多线程能提升IO读写的效率，从而整体提高redis的性能。

## RDB
在指定的时间间隔内将内存中的数据集快照到磁盘，也就是行话讲的snapshot快照，它恢复时是将快照文件直接独到内存里。

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写到一个临时文件中，待持久化过程都结束了，再用这个临时文件
替换上次持久化好的文件。整过过程中，主进程不进行任何IO操作。 这就确保了极高的性能。
如果需要大规模的数据恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式比AOF更加高效。
RDB的缺点是最后一次持久化的数据可能丢失。我们默认的是RDB，一般情况下不需要改这个配置。

RDB保存的文件dump.rdb

### 触发机制
1. save的规则满足的情况下，会自动触发rdb规则
2. 执行flushall命令，也会触发rbd规则
3. 退出redis，也会触发rdb
   备份会自动生成rdb文件。

### 恢复数据
只需要将rdb文件放在redis启动目录就可以，redis启动时会自动检查dump.rdb，恢复其中的数据。
```shell
127.0.0.1:6379> config get dir
1) "dir"
2) "/data"   //只要这个目录下存在dump.rdb文件，启动就会自动恢复其中的数据。
```
### 优点：
1. 适合大规模的数据恢复
2. 对数据的完整性要求不高

###缺点
1. 需要一定的时间间隔进程操作，如果redis意外宕机，最后一次数据丢失。
2. fork子进程的时候会占用一定的内存空间。

## AOF:
redis 每次执行一个命令时,都会把这个「命令原本的语句记录到一个.aod的文件当中,然后通过fsync策略,将命令执行后的数据持久化到磁盘中」(不包括读命令)。

### AOF的优点
1. 最多丢失1秒数据
   一般AOF会以每隔1秒，通过后台的一个线程去执行一次fsync操作。
2. 写入性能高
   AOF是将命令直接追加在文件末尾的。
3. 适合灾难性误删除恢复。
   AOF日志文件的命令通过非常可读的方式进行记录，这个非常「，如果某人不小心用 flushall 命令清空了所有数据，
   只要这个时候还没有执行 rewrite，那么就可以将日志文件中的 flushall 删除，进行恢复。

### AOF的缺点
1. 文件相对大
2. 写入性能消耗大
3. 数据恢复慢


### spring boot集成redis
spring boot2.X之后，原来使用的jedis被替换为lettuce

jedis采用的是直连，多个线程操作的话是不安全的，如果想要避免不安全，使用jedis pool连接池，像BIO模式。
lettuce：采用netty，实例可以在多个线程中共享，不存在线程不安全的情况！可以减少线程数量，像NIO模式。

找到spring boot autoconfig jar
META-INF
spring.factories
找到自动配置类
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration


源码分析
RedisAutoConfiguration.java
```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate") //可以自定义一个redisTemplate
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 没有过多的配置，redis需要序列化
        //两个泛型都是Object，我们后期需要强制转换成<String,Object>
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class) //String 最常使用的类型，所以单独列出                     
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

}
```
RedisProperties.java
```java
@ConfigurationProperties(prefix = "spring.redis")
public class RedisProperties {

	/**
	 * Database index used by the connection factory.
	 */
	private int database = 0;

	/**
	 * Connection URL. Overrides host, port, and password. User is ignored. Example:
	 * redis://user:password@example.com:6379
	 */
	private String url;

	/**
	 * Redis server host.
	 */
	private String host = "localhost";

```


