# 1.IoC（控制反转） 与 DI（依赖注入）

## 1.基本概念

### 什么是控制反转?

```tex
传统JAVA程序中，对象之间的依赖关系有程序员在代码中主动new 来维护，对象的创建和组装控制权在应用程序本身
IoC将这种控制权从应用程序转移到外部容器（SPring IoC容器），容器负责创建对象，管理对象生命周期、维护对象之间的依赖关系
```

### IoC的好处

```tex
降低组件之间的耦合度
提高可测试性（可以方便地注入Mock对象）
使代码更专注业务逻辑，而非对象创建
便于管理单例，作用域，生命周期等
```

### IoC 与DI的关系

```tex
DI(Dependency Injection,依赖注入) 是IoC的具体实现方式
容器在创建Bean时，将它的依赖对象通过构造器、Setter或字段注入进去
```

## 2.Spring IoC 容器

| 容器类型           | 特点                                                         | 常用实现                                                     |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| BeanFactory        | 懒加载，第一次获取bean时才初始化                             | XmlBeanFactory(已过时)                                       |
| ApplicationContext | 继承BeanFactory，启动时即初始化所有单例Bean，提供更多企业级功能（AOP、事件、国际化等） | ClassPathXmlApplicationContext、AnnotationConfigApplicationContext |

开发中几乎只用ApplicationContext

## 3.Bean的配置方式（声明Bean的方式）

### 1.XML配置（传统）

```xml
<bean id ="userService" class ="com.example.UserService"/>
```

### 2.注解配置（Spring 2.5+，推荐）

```
@Component/@Service/@Repository/@Controller
开启组件扫描：<context:component-scan> 或 @ComponentScane
```

### 3.Java Config配置（Spring 3.0+，无XML）

```java
@Configuration
public class AppConfig{
    @Bean
    public UserSrevice userService(){
        return new UserService();
    }
}
```

## 4.依赖注入（DI）的方式

| 注入方式   | 实现方式                  | 优点                                       | 缺点                                       |
| ---------- | ------------------------- | ------------------------------------------ | ------------------------------------------ |
| 构造器注入 | 通过构造函数参数注入      | 对象不可变（final）,依赖强制，利于单元测试 | 参数多时代码冗长                           |
| Setter注入 | 通过Setter方法注入        | 可选依赖，可重新注入                       | 可能使对象状态不一致                       |
| 字段注入   | @Autowrited直接加在字段上 | 代码简洁                                   | 不可变被破坏，无法用final,单元测试需要反射 |

最佳实践：强制依赖用构造器注入，可选依赖用Setter注入；Spring官方推荐构造器注入。

## 5.Bean的作用域（Scope）

| 作用域             | 描述                       | 适用场景                 |
| ------------------ | -------------------------- | ------------------------ |
| singleton（默认）  | 一个容器中只有一个实例     | 无状态的服务、DAO        |
| prototype          | 每次获取都创建新实例       | 有状态的Bean（如购物车） |
| request（Web）     | 每个HTTP请求一个实例       | 请求级别的数据           |
| session（Web）     | 每个HTTP Session一个实例   | 用户会话级数据           |
| application（Web） | 每个ServletContext一个实例 | 全局共享数据             |

使用@Scope注解指定

## 6.Bean 的声明周期

```tex
1.实例化：通过构造器或工厂方法创建对象（内存分配）
2.属性填充：执行依赖注入（@Autowrited、@Resource等）
3.BeanNameAware:调用setBeanName()（若实现该接口）
4.BeanFactoryAware:调用setBeanFactory()（若实现）
5.ApplicationContextAware:调用setApplicationContext()（若实现）
6.前置处理：BeanPostProcessor.postProcessBeforeInitialization()
7.初始化：
	执行@PostConstruct标注的方法
	或实现InitializingBean.afterPropertiseSet()
	或执行XML中init-method指定的方法
8.后置处理：BeanPostProcesser.postProcessAfterInitialization(),此处可生成代理对象（如AOP）
9.使用：Bean处于就绪状态
10.销毁：
	执行@PreDestory标注的方法
	或实现DisposableBean.destory()
	或实现XML中destory-method指定的方法

常见面试题：Spring如何解决循环依赖？答案设计三级缓存，与Bean生命周期密切相关
```



## 7.循环依赖（Circular Dependency）与三级缓存

```
定义：
	A依赖B,B依赖A（或更复杂）
Sping 解决单例构造器注入的循环依赖？
	不能 构造器注入循环依赖会抛出BeanCurrentlyInCtreationException。
	能解决的是单例且属性注入（Setter/字段）的循环依赖
三级缓存：
	一级缓存singletonObject:已完成初始化（完整Bean）的缓存
	二级缓存earlySingetonObjects:早期曝光对象（半成品Bean，尚未属性填充）
	三级缓存 singletonFactories:存放对象工厂（ObjectFactory）,用于生成代理对象
流程简述:
	1.A创建时，将自己放入三级缓存（工厂）
	2.A填充属性时发现需要B，去获取B
	3.B创建并填充属性时发现需要A,此时从三级缓存拿到A的早期引用（半成品），B完成创建
	4.B放入一级缓存，A继续完成属性填充和初始化，最终完成
```

关键：早期暴露半成品对象，通过工厂提前拿到引用，避免死锁

## 8.面试题

```tex
1. IoC 和 DI 的区别？
	答：IoC 是一种设计思想，将控制权从代码转移到容器；DI 是实现 IoC 的具体方式，通过构造器、Setter 或字段将依赖注入到对象中。
2. Spring 如何管理 Bean 的生命周期？
	答：从实例化 → 属性填充 → Aware 接口 → 初始化前/后 → 使用 → 销毁。具体步骤见第六节。
3. @Component 和 @Bean 的区别？
	@Component 加在类上，通过类路径扫描自动注册；@Bean 加在配置类的方法上，用于声明第三方类或需要自定义初始化逻辑的 Bean。
	@Component 由 Spring 自动检测；@Bean 需要显式调用方法返回实例。

4. Spring 如何解决循环依赖？
	答：通过三级缓存，允许提前暴露半成品对象。但仅支持单例且属性注入（Setter/字段）的循环依赖，构造器注入无法解决。

5. BeanFactory 和 ApplicationContext 的区别？
	BeanFactory 是底层容器，懒加载；ApplicationContext 是更高级的容器，启动即加载单例 Bean，提供更多功能（事件、AOP、国际化等）。
	日常使用 ApplicationContext。

6.说几个 Spring IoC 中的设计模式。
	工厂模式：BeanFactory 返回 Bean 实例。
	单例模式：默认作用域 singleton。
	代理模式：AOP 使用 JDK 动态代理或 CGLIB 代理。
	模板方法模式：JdbcTemplate、RestTemplate 等。
	观察者模式：Spring 事件监听（ApplicationEvent）。
```

## 9.复习建议

```
动手配置：分别用 XML、注解、Java Config 搭建一个小项目，体验 Bean 的定义和注入。
熟记生命周期：画一个流程图，理解每个阶段的作用。
理解循环依赖：可以断点调试 Spring 源码中的 doGetBean 方法，观察三级缓存的变化。
对比记忆：@Autowired vs @Resource、@Component vs @Bean、BeanFactory vs ApplicationContext。
```

# 2.Spring 配置方式

## 1.第一阶段：XML配置（spring 1.x时代）

```xml
方式：所有Bean定义、依赖关系、AOP、事务等全部写在applicationContext.xml中。
示例:
	<bean id ="userDao" class ="com.example.UserDao"/>
 	<bean id ="userService" class ="com.example.UserService">
		<property name ="userDao" ref ="userDao"/>
	</bean>
优点：
	所有配置集中在一个文件中，便于理解和修改（对不了解注解的人友好）
	修改配置无需重新编译代码
缺点：
	文件庞大，维护困难
	缺乏类型安全（字符串配置，出错只有运行时发现）
	配置繁琐，尤其时复杂依赖注入
```

## 2.第二阶段:注解配置（spring 2.5+，注解 + XML混合）

```xml
spring 2.5引入注解，减少XML配置量
常用注解：
	声明Bean：@Component,@Service,@Repository,@Controller
	注入依赖：@Autowrited,@Resource,@Inject
	配置作用域：@Scope
	生命周期回调：@PostConstruct,@PreDestroy
启动注解（xml中需要开启组件扫描）：
	<context:component-scan base-paskeage ='com.example' />
优点：
	配置精简，接近代码，类型安全
	自动扫描，减少显示<bean>定义
缺点：
    仍需少量xml（如扫描包路径、数据源配置）
    注解散布在各Java类中，配置逻辑不够集中
```

## 3.第三阶段：Java config(Spring 3.0+,零XML)

```java
Spring 3.0引入 @Configuration 和 @Bean，完全用Java类替代XML
核心注解：
    @Configuration:声明配置类
    @Bean:在方法上声明Bean
    @ComponentScan:等价于XML的<context:component-scan>
    @PropertySource:加载.properties文件
    @Import:导入其他配置类
    @Profile:多环境配置

实例：
@Configuration
@ComponentScan("com.example")
@PropertySource("classpath:db.properties")
public class AppConfig{
    @Value("${jdbc.url}")
    private String url;
    @Bean
    public DataSource dataSource(){
        return new DrivemanagerDataSource(url);
	}
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource ds){
        return  new JdbcTemplate(ds);
    }
}

容器启动：	
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppliConfig.class);

优点：
    纯Javam,类型安全，重构友好
    配置集中管理，可利用IDE代码提示和调试
    完全消除XML，符合”配置即代码“理念
缺点：
    修改配置需重新编译（但微服务/部署流程中通常可接受）
```

## 4.演进趋势总结

| 阶段        | 配置载体    | 声明Bean             | 注入依赖                     | 特点                |
| ----------- | ----------- | -------------------- | ---------------------------- | ------------------- |
| XML         | .xml        | <bean>               | <property>/<constructor-arg> | 全量XML，运行期解析 |
| 注解        | 注解+XML    | @Component           | @Autowrited                  | 类上注解，XML扫包   |
| Java Config | Java类+注解 | @Configuration+@Bean | 方法参数自动注入             | 零XML，纯Java       |

最终归宿：SpringBoot进一步简化，通过@SpringBootApplication(组合了@Configuration、@ComponentScan、@EnableAutoConfiguration)和application.yml外部配置，实现了约定大于配置，减少大量显示配置

## 5.面试

```
1. 三种配置方式分别适用什么场景？
	XML：遗留系统维护，或需要动态修改配置而不想重启（极少）。
	注解 + XML：大多数传统 Spring 项目。
	Java Config：Spring Boot 新项目首选，纯 Java 易维护。
2. @Bean 和 @Component 的区别？
	@Component 加在类上，通过扫描自动注册；@Bean 加在 @Configuration 类的方法上，手工声明 Bean。
	@Bean 适合实例化第三方类、需要复杂初始化逻辑的 Bean；@Component 用于自己写的类。
3. 如何完全不用 XML 启动 Spring 容器？
	使用 AnnotationConfigApplicationContext 并传入 @Configuration 类。
4. 为什么 Spring Boot 不需要显式配置？
	因为 @SpringBootApplication 隐含了 @ComponentScan（扫描当前包及子包）和 @EnableAutoConfiguration（根据类路径依赖自动配置），再配合 application.properties 外部化配置。
```

## 6.复习建议

```
理解三个阶段各自的标志性技术和优缺点。
能写出一个简单的 Java Config 配置类。
明确 @Component、@Bean、@Configuration 的关系。
知道 Spring Boot 如何基于 Java Config 进一步简化
```

# 3.Spring事务管理

## 1.事务管理方式

| 方式       | 实现                                                    | 优点             | 缺点                     |
| ---------- | ------------------------------------------------------- | ---------------- | ------------------------ |
| 编程式事务 | TransactionTemplate 或直接获取PlatfromTranactionManager | 精确控制事务边界 | 侵入业务代码，重复代码多 |
| 声明式事务 | @Transaction注解或XML配置<tx:advice>                    | 非侵入，简洁     | 粒度较粗（方法级）       |

开发中几乎只用声明式事务（@Transactional）

## 2.声明式事务的核心：@Transactional

### 1.常见属性

| 属性          | 说明                   | 默认值                          |
| ------------- | ---------------------- | ------------------------------- |
| propagation   | 事务传播行为           | Propagation.REQUIRED            |
| isolation     | 事务隔离级别           | Isolation.DEFAULT（数据库默认） |
| timeout       | 超时时间（秒）         | -1（无限制）                    |
| readOnly      | 是否只读               | false                           |
| rollbackFor   | 指定哪些异常出发回滚   | RunTimeException 和 Error       |
| noRollbackFor | 指定哪些异常不处罚回滚 | 无                              |

### 2.使用位置

```
方法上：覆盖类级别的配置
类上：该类所有public方法都应用该事务配置
```

### 3.注意事项

```
只能作用于public 方法（Spring AOP代理限制）
默认仅对RuntimeException 和 Error回滚，checked异常（Exception子类非 RuntimeException）不回滚，可通过rollbackFor指定
```

## 3.事务传播行为（propagation,高频考点）

| 传播行为         | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| REQUIRED（默认） | 当前有事务则加入，无则新建                                   |
| REQUIRES_NEW     | 挂起当前事务，新建事务（独立提交/回滚）                      |
| SUPPORTS         | 有则加入，无则以非事务执行                                   |
| NOT_SUPPORTED    | 以非事务执行，挂起当前事务                                   |
| MANDATORY        | 必须在以有事务中运行，否则抛异常                             |
| NEVER            | 必须在非事务中运行，否则抛异常                               |
| NESTED           | 嵌套事务（保存点机制），外层回滚内层可回滚，内层回滚外层不一定回滚 |

面试常见场景

​	方法A调用方法B，两个都加@Transactional，默认传播行为下，它们属于同一事务（B，出现异常，A的操作也会回滚）

​	希望B独立回滚：B的传播行为设为REQUIRES_NEW

​	NESTED 与 REQUIRES_NEW的区别：REQUIRES_NEW是完全独立事务；NESTED  是嵌套事务，依赖于外层事务，外层提交内层才提交，内层回滚可单独回滚到保存点

## 4.事务隔离级别

| 隔离级别           | 脏读 | 不可重复读 | 幻读                          |
| :----------------- | :--- | :--------- | :---------------------------- |
| `READ_UNCOMMITTED` | 可能 | 可能       | 可能                          |
| `READ_COMMITTED`   | 不会 | 可能       | 可能                          |
| `REPEATABLE_READ`  | 不会 | 不会       | 可能（InnoDB 通过间隙锁避免） |
| `SERIALIZABLE`     | 不会 | 不会       | 不会                          |

`ISOLATION_DEFAULT`：使用底层数据库的默认隔离级别（MySQL 为 `REPEATABLE_READ`，Oracle/PostgreSQL 为 `READ_COMMITTED`）。

## 5.readOnly优化

将事务标记为只读，可以提示数据库和Spring 进行优化（如：FlushMode = MANUAL，跳过脏数据检查）

适用于查询方法，但非强制，不影响业务逻辑

## 6.事务失效场景（高频面试题）

| 场景                                              | 原因                                | 解决办法                                                     |
| :------------------------------------------------ | :---------------------------------- | :----------------------------------------------------------- |
| `@Transactional` 加在非 `public` 方法上           | Spring AOP 代理只拦截 `public` 方法 | 改为 `public`                                                |
| 同类中方法调用（`this.method()`）                 | 没有经过代理，直接调用目标对象方法  | 通过代理对象调用（如 `self` 注入）                           |
| 异常被 `catch` 后未抛出                           | 切面感知不到异常，认为事务正常      | 重新抛出或手动回滚（`TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()`） |
| 抛出的异常不是 `RuntimeException`（Checked 异常） | 默认不回滚 checked 异常             | 指定 `@Transactional(rollbackFor = Exception.class)`         |
| 数据库引擎不支持事务（如 MyISAM）                 | 引擎限制                            | 改用 InnoDB                                                  |
| 事务方法在异步线程中执行                          | 事务上下文不会传播到子线程          | 使用 `@Async` 结合事务管理器配置，或编程式事务               |
| 方法被 `final` 或 `static` 修饰                   | 代理无法重写                        | 去掉 `final`/`static`                                        |

## 7.事务管理器的配置（Spring Boot）

```yaml
# application.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
  jpa:
    show-sql: true
```

Spring Boot 自动配置 `DataSourceTransactionManager`（对于 JPA 是 `JpaTransactionManager`）。

手动配置多事务管理器

```java
@Configuration
public class TxConfig {
    @Bean
    @Primary
    public PlatformTransactionManager txManager(DataSource ds) {
        return new DataSourceTransactionManager(ds);
    }
}
```

使用时指定 `transactionManager` 属性：

```java
@Transactional(transactionManager = "txManager")
```

## 8.面试题

```
1. Spring 支持哪两种事务管理方式？
	答：编程式事务（TransactionTemplate）和声明式事务（@Transactional）。推荐声明式事务。
2. @Transactional 默认的回滚规则是什么？
	答：只对 RuntimeException 和 Error 回滚，对 checked 异常不回滚。
3. REQUIRES_NEW 和 NESTED 的区别？
	REQUIRES_NEW：挂起当前事务，新建一个独立事务，两者互不影响；内层回滚不影响外层，外层回滚不影响内层（已提交）。
	NESTED：基于保存点，内层事务是外层的一部分，外层提交内层才提交，内层回滚可回滚到保存点，外层回滚会导致内层也回滚。
4. 哪些情况下 @Transactional 会失效？
	答：非 public 方法、同类调用、异常被吞、checked 异常未指定、数据库引擎不支持、异步线程等。
5. 一个方法加了 @Transactional，调用另一个加了 @Transactional 的方法，它们是一个事务吗？
	答：默认传播行为 REQUIRED，是同一个事务。如果想让被调用方法独立事务，需设为 REQUIRES_NEW。
6. 只读事务 readOnly = true 有什么作用？
	答：优化性能，避免不必要的 flush，某些数据库会使用快照读或跳过锁，但并非强制只读，仍可能执行写操作（会产生异常或无效）。
```

## 9.复习建议

```
熟练背下传播行为和隔离级别的取值及含义，尤其 REQUIRED 和 REQUIRES_NEW 的区别。
牢记失效场景，这是面试常问的“坑”题。
理解 Spring 如何实现声明式事务：AOP 代理 + TransactionInterceptor，在目标方法前后开启、提交/回滚事务。
会配置事务管理器（Spring Boot 自动配置常见，但也要知道手动覆盖）。
```

