## Spring IoC
### IoC（Inversion of Control，控制倒转）。这是spring的核心，贯穿始终。
所谓IoC，对于spring框架来说，就是由spring来负责控制对象的生命周期和对象间的关系。

>IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的。
比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，
有了 spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。
在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。
A需要依赖 Connection才能正常运行，而这个Connection是由spring注入到A中的，依赖注入的名字就这么来的。

那么DI是如何实现的呢？ Java 1.3之后一个重要特征是反射（reflection），
**它允许程序在运行的时候动态的生成对象、执行对象的方法、改变对象的属性， spring就是通过反射来实现注入的。**

### 实现IOC的步骤
- 定义用来描述bean的配置的Java类
- 解析bean的配置，將bean的配置信息转换为上面的BeanDefinition对象保存在内存中，spring中采用HashMap进行对象存储，其中会用到一些xml解析技术.
- 遍历存放BeanDefinition的HashMap对象，逐条取出BeanDefinition对象，获取bean的配置信息，利用Java的反射机制实例化对象，將实例化后的对象保存在另外一个Map中即可。

### 依赖注入的方式
1、Set注入 2、构造器注入 3、接口注入

## AOP （Aspect Orient Programming）,
直译过来就是 面向切面编程。AOP 是一种编程思想，是面向对象编程（OOP）的一种补充。
面向对象编程将程序抽象成各个层次的对象，而面向切面编程是将程序抽象成各个切面。

AOP就是希望将这些分散在各个业务逻辑代码中的相同代码，
通过横向切割的方式抽取到一个独立的模块中，让业务逻辑类依然保存最初的单纯。

比如安全，事务，全局异常处理等。

### Spring AOP中的动态代理
JDK动态代理和CGLIB动态代理。
>JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。
> JDK动态代理的核心是InvocationHandler接口和Proxy类。

>如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。
> CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，
> **注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。**

### AOP的应用场景
>Authentication 权限 ，Caching 缓存 ，Context passing 内容传递 ，Error handling 错误处理 ，
> Lazy loading 懒加载 ，Debugging 调试 ，logging, tracing, 
> profiling and monitoring 记录跟踪　优化　校准，Performance optimization 性能优化 ，
> Persistence 持久化 ，Resource pooling 资源池 ，Synchronization 同步，Transactions 事务。