
### 节点类型
- 持久化目录节点 
- 持久化顺序编号目录节点
- 临时节点
- 临时顺序编号目录节点

        在分布式系统中，顺序号可以被用于为所有事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序        
        临时节点适合动态上下线


### Leader选举
启动的时候，leader挂了的时候
leader选举的时候不能对外提供服务。

先投自己，然后将投票的数据发送给其他的server进行PK，发送的数据[myid(自己的),myid(投票的),zxid(自己的)]
Pk的原则：
- zxid最新的优先
- zxid都一样，myid大的优先

**过半原则，投票过半则是leader。 假设3台服务器，1和2都投给了2，此时2是Leader。**

此时服务器3上线，自己投自己，然后将投票数据发给1和2，1和2会把投票的结果发给3，不会再进行PK，3变成follow。
1. 如果3的zxid比leader的大，则3回退zxid.
2. 如果3的zxid比leader的小，leader和3进行数据同步，将日志发送给3进行同步。


### Leader处理写请求
1. 持久化日志zxid（自增）
2. 发送日志到其他服务器follow
3. Follow回ACK（过半原则）
4. 提交（更改内存数据）发送commit给follow
5. 响应客户端

2PC（1.预提交 2.ACK 3.提交）

超过一半的follow挂了，leader卸任，变成普通者，进行领导者选举，不能对外提供服务。


leader像follow发送commit命令
把commit命令放在queue里，异步发送。
zk并没有保证强一致性，保证最终一致性。

zoo.cfg配置文件里没有配置
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
代表单机模式，配置了，说明是集群模式

### ZAB协议
ZAB协议包括两种基本的模式：崩溃恢复和消息广播，当整个zk集群刚刚启动或者Leader服务器宕机、重启或者网络故障不存在过半的服务器与Leader服务器保持正常通信时，所有服务器进入崩溃恢复模式，首先选举产生新的Leader服务器，然后集群中Follower服务器开始与新的Leader服务器进行数据同步，当集群中超过半数机器与该Leader服务器完成同步数据之后，退出恢复模式进入消息广播模式，Leader服务器开始接收客户端的事务请求生成事务提案（超过半数同意）来进行事务请求处理


### 数据节点
Znode将节点分为持久节点和临时节点。
- 持久节点
持久节点一旦创建，只要不主动移除，他将永远存在于zk的系统中。
- 临时节点
临时节点与创建它的客户端会话相关联，一旦会话结束，客户端创建的临时节点将自动删除。
  
值得注意的是，只有持有节点可以拥有子节点，临时节点不能拥有子节点

###源码分析
https://segmentfault.com/a/1190000022726256?utm_source=tag-newest

https://mp.weixin.qq.com/s/ByfASCD2V-JcBtVCXTXPew
https://www.bilibili.com/video/BV1qK4y1V7LD?p=7&spm_id_from=pageDriver


