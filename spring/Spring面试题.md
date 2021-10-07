## ApplicationContext 和 BeanFactory 区别
### 概念：
####BeanFactory：
>BeanFactory是spring中比较原始，比较古老的Factory。因为比较古老，所以BeanFactory无法支持spring插件，例如：AOP、Web应用等功能。

####ApplicationContext： 
>ApplicationContext是BeanFactory的子类，因为古老的BeanFactory无法满足不断更新的spring的需求，于是ApplicationContext就基本上代替了BeanFactory的工作，以一种更面向框架的工作方式以及对上下文进行分层和实现继承，并在这个基础上对功能进行扩展：

- <1>MessageSource, 提供国际化的消息访问
- <2>资源访问（如URL和文件）
- <3>事件传递
- <4>Bean的自动装配
- <5>各种不同应用层的Context实现

### 区别：
- <1>如果使用ApplicationContext，如果配置的bean是singleton，那么不管你有没有或想不想用它，它都会被实例化。好处是可以预先加载，坏处是浪费内存。
- <2>BeanFactory，当使用BeanFactory实例化对象时，配置的bean不会马上被实例化，而是等到你使用该bean的时候（getBean）才会被实例化。好处是节约内存，坏处是速度比较慢。多用于移动设备的开发。
- <3>没有特殊要求的情况下，应该使用ApplicationContext完成。因为BeanFactory能完成的事情，ApplicationContext都能完成，并且提供了更多接近现在开发的功能。


## Spring Bean生命周期
Spring中⼀个Bean的创建⼤概分为以下⼏个步骤：
1. 推断构造⽅法
2. 实例化
3. 填充属性，也就是依赖注⼊
4. 处理Aware回调
5. 初始化前，处理@PostConstruct注解
6. 初始化，处理InitializingBean接⼝
7. 初始化后，进⾏AOP

- 实例化bean
- Spring根据bean定义填充所有属性（依赖注入）
- 如果bean实现了BeanNameAware接口，Spring传递bean的ID到setBeanName方法。
- 如果bean实现了BeanFactoryAware接口，Spring传递beanFactory给setBeanFactory方法。
- 如果有与bean相关联的BeanPostProcessor，Spring会在postProcessBeforeInitialzation()方法内调用他们。
- 如果bean实现了处理InitializingBean，调用它的afterPropertySet方法，
- 如果bean声明了初始化方法，调用此初始化方法。
- 如果有与bean相关联的BeanPostProcessor，这些bean的postProcessAfterInitialzation()方法将被调用。

该方法在bean初始化方法前被调用，Spring AOP的底层处理也是通过实现BeanPostProcessor来执行代理逻辑的

### spring 如何处理线程并发问题
>在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，
因为Spring对一些Bean(如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等)中
非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。

>ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。
同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。
而ThreadLocal采用了“空间换时间”的方式。

>ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。
因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。
ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。
