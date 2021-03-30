
## Spring Boot中的starters是什么？
Spring Boot 中的starter 只不过是把我们某一模块，比如web 开发时所需要的所有JAR 包打包好给我们而已。不过它的厉害之处在于，能自动把配置文件搞好，不用我们手动配置。所以说，Spring Boot 是简化配置。

## Spring Boot怎么定义一个starters？
实现一个 starter 有四个要素：
1. starter 命名 ;
2. 自动配置类，用来初始化相关的 bean ;
3. 指明自动配置类的配置文件 spring.factories ;
4. 自定义属性实体类，声明 starter 的应用配置属性 ;