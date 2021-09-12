#Spring Cloud Ribbon负载均衡策略详解
## 一，IRule接口
IRule接口定义了选择负载均衡策略的基本操作。通过调用choose()方法，就可以选择具体的负载均衡策略。

```java
// 选择目标服务节点
Server choose(Object var1);

// 设置负载均衡策略
void setLoadBalancer(ILoadBalancer var1);

// 获取负载均衡策略
ILoadBalancer getLoadBalancer();
```
## 二，ILoadBalancer接口
ILoadBalancer接口定义了ribbon负载均衡的常用操作，有以下几个方法：


## 四，AbstractLoadBalancerRule的实现类
AbstractLoadBalancerRule的实现类就是ribbon的具体负载均衡策略，首先来看默认的轮询策略。

### 1，轮询策略（RoundRobinRule）
轮询策略理解起来比较简单，就是拿到所有的server集合，然后根据id进行遍历。这里的id是ip+端口，Server实体类中定义的id属性如下：

this.id = host + ":" + port

这里还有一点需要注意，轮询策略有一个上限，当轮询了10个服务端节点还没有找到可用服务的话，轮询结束。

### 2，随机策略（RandomRule）
随机策略：使用jdk自带的随机数生成工具，生成一个随机数，然后去可用服务列表中拉取服务节点Server。
如果当前节点不可用，则进入下一轮随机策略，直到选到可用服务节点为止。

### 3，可用过滤策略（AvailabilityFilteringRule）
策略描述：过滤掉连接失败的服务节点，并且过滤掉高并发的服务节点，然后从健康的服务节点中，使用轮询策略选出一个节点返回。

### 4，响应时间权重策略（WeightedResponseTimeRule）

### 5，轮询失败重试策略（RetryRule）

### 6，并发量最小可用策略（BestAvailableRule）

### 7，ZoneAvoidanceRule
策略描述：复合判断server所在区域的性能和server的可用性，来选择server返回。
