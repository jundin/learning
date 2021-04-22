
### 什么是循环依赖
循环依赖其实就是循环引用，也就是两个或则两个以上的bean互相持有对方，最终形成闭环。
比如A依赖于B，B依赖于C，C又依赖于A

**Spring循环依赖的情况有两种：**

- 构造器的循环依赖 （Spring 是无法解决的，只能抛出 BeanCurrentlyInCreationException 异常）
- field属性的循环依赖


### Spring怎么解决循环依赖
**Spring的单例对象的初始化主要分为三步：**

    createBeanInstance实例化
    populateBean填充属性
    initializeBean初始化

从上面讲述的单例bean初始化步骤我们可以知道，循环依赖主要发生在第一、第二步。

那么我们要解决循环引用也应该从初始化过程着手，对于单例来说，在Spring容器整个生命周期内，有且只有一个对象，
所以很容易想到这个对象应该存在Cache中，Spring为了解决单例的循环依赖问题，使用了三级缓存。

**首先我们看源码，三级缓存主要指：**
```java
/** Cache of singleton objects: bean name to bean instance. */
//单例bean的缓存 一级缓存
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name to ObjectFactory. */
//单例对象工厂缓存 三级缓存
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name to bean instance. */
//预加载单例bean缓存 二级缓存
//存放的 bean 不一定是完整的
private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
```

我们在创建bean的时候，首先想到的是从cache中获取这个单例的bean，这个缓存就是singletonObjects。主要调用方法就就是：
**DefaultSingletonBeanRegistry#getSingleton(String, boolean)**

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // Quick check for existing instance without full singleton lock
    // 从一级缓存缓存 singletonObjects 中加载 bean
    Object singletonObject = this.singletonObjects.get(beanName);
    // 缓存中的 bean 为空，且当前 bean 正在创建
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        // 从 二级缓存 earlySingletonObjects 中获取
        singletonObject = this.earlySingletonObjects.get(beanName);
        // earlySingletonObjects 中没有，且允许提前创建
        if (singletonObject == null && allowEarlyReference) {
            // 加锁
            synchronized (this.singletonObjects) {
                // Consistent creation of early reference within full singleton lock
                
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    if (singletonObject == null) {
                        // 从 三级缓存 singletonFactories 中获取对应的 ObjectFactory
                        ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            //从单例工厂中获取bean
                            singletonObject = singletonFactory.getObject();
                            // 添加到二级缓存
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            // 从三级缓存中删除
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }
    return singletonObject;
}
```
上面的代码需要解释两个参数：

    isSingletonCurrentlyInCreation()判断当前单例bean是否正在创建中，也就是没有初始化完成(比如A的构造器依赖了B对象所以得先去创建B对象， 或则在A的populateBean过程中依赖了B对象，得先去创建B对象，这时的A就是处于创建中的状态。)
    allowEarlyReference 是否允许从singletonFactories中通过getObject拿到对象

分析getSingleton()的整个过程

    首先，尝试从一级缓存singletonObjects中获取单例Bean如果获取不到，
    则从二级缓存earlySingletonObjects中获取单例Bean如果仍然获取不到，
    则从三级缓存singletonFactories中获取单例BeanFactory
    最后，如果从三级缓存中拿到了BeanFactory，则通过getObject()把Bean存入二级缓存中，并把该Bean的三级缓存删掉


### 三级缓存
看看下存储缓存的代码 在 AbstractAutowireCapableBeanFactory 的 doCreateBean() 方法中，有这么一段代码：
```java
    // Eagerly cache singletons to be able to resolve circular references
    // even when triggered by lifecycle interfaces like BeanFactoryAware.
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
            isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        if (logger.isTraceEnabled()) {
            logger.trace("Eagerly caching bean '" + beanName +
                    "' to allow for resolving potential circular references");
        }
        // 为了后期避免循环依赖，提前将创建的 bean 实例加入到三级缓存 singletonFactories 中
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
```

其核心逻辑是，当满足以下3个条件时，把bean加入三级缓存中：

    单例
    允许循环依赖
    当前单例Bean正在创建

#### DefaultSingletonBeanRegistry#addSingletonFactory
```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

从这段代码我们可以看出，singletonFactories 这个三级缓存才是解决 Spring Bean 循环依赖的关键

同时这段代码发生在 createBeanInstance(...) 方法之后，也就是说这个 bean 其实已经被创建出来了，但是它还没有完善（没有进行属性填充和初始化），
但是对于其他依赖它的对象而言已经足够了（已经有内存地址了，可以根据对象引用定位到堆中对象），能够被认出来了.

### 一级缓存

在类 DefaultSingletonBeanRegistry 中，可以发现这个 addSingleton(String beanName, Object singletonObject) 方法，代码如下：
```java
protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        this.singletonObjects.put(beanName, singletonObject);
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```
该方法是在 #doGetBean(...)方法中，一级缓存里面是完整的Bean


### 总结
    一级缓存里面是完整的Bean,是当一个Bean完全创建后才put
    三级缓存是不完整的BeanFactory,是当一个Bean在new之后就put(没有属性填充、初始化)
    二级缓存是对三级缓存的易用性处理，只不过是通过getObject()方法从三级缓存的BeanFactory中取出Bean

### 循环依赖解决
Spring 在创建 bean 的时候并不是等它完全完成，
而是**在创建过程中将创建中的 bean 的 ObjectFactory 提前曝光（即加入到 singletonFactories 三级缓存中）**
这样，一旦下一个 bean 创建的时候需要依赖 bean ，则从三级缓存中获取

**最后来描述之前那个循环依赖 Spring 解决的过程：**

    首先 A 完成初始化第一步并将自己提前曝光出来（通过三级缓存将自己提前曝光），
    在初始化的时候，发现自己依赖对象 B，此时就会去尝试 get(B)，这个时候发现 B 还没有被创建出来

    然后 B 就走创建流程，在 B 初始化的时候，同样发现自己依赖 C，C 也没有被创建出来

    这个时候 C 又开始初始化进程，但是在初始化的过程中发现自己依赖 A，于是尝试 get(A)，
    这个时候由于 A 已经添加至缓存中（三级缓存 singletonFactories ），通过 ObjectFactory 提前曝光，
    所以可以通过 ObjectFactory#getObject() 方法来拿到 A 对象，C 拿到 A 对象后顺利完成初始化，然后将自己添加到一级缓存中

    回到 B ，B 也可以拿到 C 对象，完成初始化，A 可以顺利拿到 B 完成初始化，到这里整个链路就已经完成了初始化过程了

