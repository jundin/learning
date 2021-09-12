
## Spring Boot中的starters是什么？
Spring Boot 中的starter 只不过是把我们某一模块，比如web 开发时所需要的所有JAR 包打包好给我们而已。不过它的厉害之处在于，能自动把配置文件搞好，不用我们手动配置。所以说，Spring Boot 是简化配置。

## Spring Boot怎么定义一个starters？
实现一个 starter 有四个要素：
1. starter 命名 ;
2. 自动配置类，用来初始化相关的 bean ;
3. 指明自动配置类的配置文件 spring.factories ;
4. 自定义属性实体类，声明 starter 的应用配置属性 ;


## SpringBoot是如何启动Tomcat的
1. 首先，SpringBoot在启动时会先创建⼀个Spring容器
2. 在创建Spring容器过程中，会利⽤@ConditionalOnClass技术来判断当前classpath中是否存在
   Tomcat依赖，如果存在则会⽣成⼀个启动Tomcat的Bean
3. Spring容器创建完之后，就会获取启动Tomcat的Bean，并创建Tomcat对象，并绑定端⼝等，然后
   启动Tomcat