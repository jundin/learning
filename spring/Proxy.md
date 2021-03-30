# 代理模式
代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。

按照代理的创建时期，代理类可以分为两种。
- 静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。
- 动态代理：在程序运行时，运用反射机制动态创建而成。 

## 静态代理
Service.java
```java
public interface Service {
    public void update();
}
```

ServiceImpl.java
```java
public class ServiceImpl implements Service{
    @Override
    public void update() {
        System.out.println("-----This is an update operation-----");
    }
}
```

StaticProxy.java
```java
public class StaticProxy implements Service {

    private ServiceImpl service;

    public StaticProxy(ServiceImpl service) {
        this.service = service;
    }

    @Override
    public void update() {
        System.out.println("-----Before operation-----");
        service.update();
        System.out.println("-----After operation-----");
    }
}
```
TestProxy.java
```java
public class TestProxy {
    public static void main(String[] args) {
        StaticProxy staticProxy = new StaticProxy(new ServiceImpl());
        staticProxy.update();
    }
}
```

**观察代码可以发现每一个代理类只能为一个接口服务，这样一来程序开发中必然会产生过多的代理**，而且，所有的代理操作除了调用的方法不一样之外，其他的操作都一样，则此时肯定是重复代码。解决这一问题最好的做法是可以通过一个代理类完成全部的代理功能，那么此时就必须使用动态代理完成。

## 动态代理

### JDK动态代理
JDK动态代理中包含一个类和一个接口：

**InvocationHandler接口：**
```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```
参数说明：
- Object proxy：指被代理的对象。
- Method method：要调用的方法
- Object[] args：方法调用时所需要的参数

**Proxy类：**
Proxy类是专门完成代理的操作类，可以通过此类为一个或多个接口动态地生成实现类，此类提供了如下的操作方法：
```java
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h) 
```
参数说明：
- ClassLoader loader：类加载器
- Class<?>[] interfaces：得到全部的接口
- InvocationHandler h：得到InvocationHandler接口的子类实例

Ps:类加载器
在Proxy类中的newProxyInstance（）方法中需要一个ClassLoader类的实例，ClassLoader实际上对应的是类加载器，在Java中主要有一下三种类加载器;
- Booststrap ClassLoader：此加载器采用C++编写，一般开发中是看不到的；
- Extendsion ClassLoader：用来进行扩展类的加载，一般对应的是jre\lib\ext目录中的类;
- AppClassLoader：(默认)加载classpath指定的类，是最常使用的是一种加载器。

动态代理
动态代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手工编写它的源代码。动态代理类不仅简化了编程工作，而且提高了软件系统的可扩展性，
因为Java 反射机制可以生成任意类型的动态代理类。java.lang.reflect 包中的Proxy类和InvocationHandler 接口提供了生成动态代理类的能力。

动态代理示例:
Service.java
```java
public interface Service {
    public void update();
}
```

ServiceImpl.java
```java
public class ServiceImpl implements Service{
    @Override
    public void update() {
        System.out.println("-----This is an update operation-----");
    }
}
```
JDKProxy.java
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class JDKProxy implements InvocationHandler {

    private Object target;
    /**
     * 绑定委托对象并返回一个代理类 
     * @param target
     * @return
     */
    public Object bind(Object target) {
        this.target = target;
        //取得代理对象  
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), this); //要绑定接口(这是一个缺陷，cglib弥补了这一缺陷)  
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("-----Before operation-----");
        Object ret = method.invoke(target,args);
        System.out.println("-----After operation-----");
        return ret;
    }
}
```
TestProxy.java
```java
public class TestProxy {
    public static void main(String[] args) {
        JDKProxy jdkProxy = new JDKProxy();
        Service service = (Service) jdkProxy.bind(new ServiceImpl());
        service.update();
    }
}
```
但是，JDK的动态代理依靠接口实现，如果有些类并没有实现接口，则不能使用JDK代理，这就要使用cglib动态代理了。

### Cglib动态代理 
JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理，cglib是针对类来实现代理的，
他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。 

Service.java
```java
public class Service {
    public void update() {
        System.out.println("-----This is an update operation-----");
    }
}
```
CglibProxy.java
```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class CglibProxy implements MethodInterceptor {

    private Object target;

    public Object getInstance(Object target) {
        this.target = target;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(this.target.getClass());
        // 回调方法
        enhancer.setCallback(this);
        // 创建代理对象
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, 
                            MethodProxy methodProxy) throws Throwable {
        System.out.println("-----Before operation-----");
        Object ret = methodProxy.invokeSuper(o, objects);
        System.out.println("-----After operation-----");
        return ret;
    }
}
```
```java
public class TestProxy {
    public static void main(String[] args) {
        CglibProxy cglibProxy = new CglibProxy();
        Service service = (Service1) cglibProxy.getInstance(new Service());
        service.update();
    }
}
```