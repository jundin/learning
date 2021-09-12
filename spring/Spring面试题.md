## ApplicationContext 和 BeanFactory 区别
BeanFactory是Spring中⾮常核⼼的组件，表示Bean⼯⼚，可以⽣成Bean，维护Bean，⽽
ApplicationContext继承了BeanFactory，所以ApplicationContext拥有BeanFactory所有的特点，也
是⼀个Bean⼯⼚。

但是ApplicationContext除开继承了BeanFactory之外，还继承了诸如
EnvironmentCapable、MessageSource、ApplicationEventPublisher等接⼝，从⽽
ApplicationContext还有获取系统环境变量、国际化、事件发布等功能，这是BeanFactory所不具备的。

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
