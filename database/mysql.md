### mysql 中 varchar 与 char 的区别？
varchar 与 char 的区别:
- char 是一种固定长度的类型， 
- varchar 则是 一种可变长度的类型

### varchar(50)中50 的涵义
最多存放 50个字节

### int（20）中 20 的涵义:
int(M)中的 M indicates the maximum display width(最大显示宽度)for integer types.
The maximum legal display width is 255.

### MySQL 中 InnoDB 支持的四种事务隔离级别名称， 以及逐级之间的区 别？
1.Read Uncommitted （读取未提交内容）
>在该隔离级别， 所有事 务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，
因为它的性能也不比其他级别好多少。读取未提交的数据， 也被称 之为脏读（ Dirty Read ）。

2.Read Committed （读取提交内容）
>这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。
它满足了隔离的简单定义： 一个事务只能看见已经提交事务所做的改变。
这种隔离级别也支持所谓的 不可重复读（Nonrepeatable Read），
因为同一事务的其他实例在该 实例处理其间可能会有新的 commit ， 
所以同一 select 可能返回不同结果

3.Repeatable Read （可重读）
>这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。
不过理论上，这会导致另一个棘手的问题：幻读（Phantom Read)。简单的说，幻读指当用户读取某一范围的数据行时，
另一个事 务又在该范围内插入了新行，当用户再读取该范围的数据行时， 会发现 有新的 “ 幻影 ” 行。
InnoDB 存储引擎通过多版本并发控制 （MVCC ，Multiversion Concurrency Control 间隙锁）机制解决了该问题。
注：其实多版本只是解决不可重复读问题，而加上间隙锁（也就是所谓的并发控制）才解决了幻读问题。

4.Serializable （可串行化）
>这是最高的隔离级别，它通过强制事务 排序，使之不可能相互冲突， 从而解决幻读问题。
简言之， 它是在每个读的数据行上加上共享锁。 在这个级别，可能导致大量的超时现象和锁竞争。


### Mysql架构
1. 最上层
> 最上层是一些客户端和连接服务，包含本地的sock通信和大多数基于客户端/服务端工具实现的类似于tcp/ip的通信，
主要完成一些类似于连接处理、授权认证及相关的安全方案，在该层上引用了线程池的概念，为通过认证安全接入的客户端提供线程。
同样在该层上可以实现基于ssl的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。
2. 第二层：
> 第二层架构主要完成大多数的核心服务功能。如sql接口，并完成缓存的查询。sql的分析和优化 以及部分内置函数的执行。
所有跨存储引擎的功能也在这一层实现，如过程，函数等。在该层，服务器会解析查询并创建相应的内部解析树，
并对其完成相应的优化如确定查询表的顺序，是否利用索引等。最后生成相应的执行操作。
如select语句，服务器还会查询内部的缓存。如果缓存空间足够大，这样就解决大量读操作的环境中能够很好的提升系统的性能。
3. 存储引擎层：
> 存储引擎真正的负责MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信，
   不同的存储引擎具有的功能不同， 这样我们可以根据自己的实际需进行选取。
4. 数据存储层：
> 主要是将数据存储在运行于裸设备的文件系统之上，并完成于存储引擎的交互。 