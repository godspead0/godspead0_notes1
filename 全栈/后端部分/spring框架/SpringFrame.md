# Spring Framework 核心概念详解

## 1. IOC（控制反转）与 DI（依赖注入）

### 1.1 IOC 思想

```java
// 传统方式 - 紧耦合
public class UserService {
    private UserDao userDao = new UserDaoImpl(); // 直接创建依赖对象
}

// IOC 方式 - 松耦合
public class UserService {
    private UserDao userDao; // 依赖由容器注入
}
```

**IOC 核心思想：**
- 将对象的创建和管理权从程序内部转移到外部容器
- 降低组件间的耦合度
- 提高代码的可维护性和可测试性

### 1.2 DI（依赖注入）

```java
// 容器自动管理对象间的依赖关系
public class OrderService {
    private UserService userService;    // 依赖 UserService
    private ProductService productService; // 依赖 ProductService
    // 容器会自动注入这些依赖
}
```

## 2. Bean 配置与管理

### 2.1 XML 配置方式

```xml
<!-- 导入 Spring 坐标 -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.21</version>
    </dependency>
</dependencies>

<!-- applicationContext.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 基础 Bean 配置 -->
    <bean id="userDao" class="com.example.dao.UserDaoImpl"/>
    
    <!-- 依赖注入配置 -->
    <bean id="userService" class="com.example.service.UserServiceImpl">
        <property name="userDao" ref="userDao"/> <!-- setter 注入 -->
    </bean>
    
    <!-- 构造器注入 -->
    <bean id="orderService" class="com.example.service.OrderServiceImpl">
        <constructor-arg index="0" ref="userService"/>
        <constructor-arg index="1" ref="productService"/>
    </bean>
</beans>
```

### 2.2 Bean 作用范围

```xml
<bean id="userService" class="com.example.service.UserServiceImpl" 
      scope="singleton"/> <!-- 默认，单例 -->

<bean id="userSession" class="com.example.model.UserSession" 
      scope="prototype"/> <!-- 原型，每次获取新实例 -->

<bean id="userRequest" class="com.example.model.UserRequest" 
      scope="request"/> <!-- 请求范围 -->

<bean id="userSession" class="com.example.model.UserSession" 
      scope="session"/> <!-- 会话范围 -->
```

### 2.3 Bean 实例化方式

#### 2.3.1 无参构造（默认）

```xml
<bean id="userDao" class="com.example.dao.UserDaoImpl"/>
```

#### 2.3.2 静态工厂方法

```java
// 静态工厂类
public class StaticBeanFactory {
    public static UserDao createUserDao() {
        return new UserDaoImpl();
    }
}
```

```xml
<bean id="userDao" class="com.example.factory.StaticBeanFactory" 
      factory-method="createUserDao"/>
```

#### 2.3.3 实例工厂方法

```java
// 实例工厂类
public class InstanceBeanFactory {
    public UserDao createUserDao() {
        return new UserDaoImpl();
    }
}
```

```xml
<!-- 先配置工厂 Bean -->
<bean id="beanFactory" class="com.example.factory.InstanceBeanFactory"/>

<!-- 再配置目标 Bean -->
<bean id="userDao" factory-bean="beanFactory" 
      factory-method="createUserDao"/>
```

#### 2.3.4 FactoryBean 接口

```java
public class UserDaoFactoryBean implements FactoryBean<UserDao> {
    @Override
    public UserDao getObject() throws Exception {
        return new UserDaoImpl();
    }
    
    @Override
    public Class<?> getObjectType() {
        return UserDao.class;
    }
    
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

```xml
<bean id="userDao" class="com.example.factory.UserDaoFactoryBean"/>
```

### 2.4 Bean 生命周期

```xml
<bean id="userService" class="com.example.service.UserServiceImpl" 
      init-method="init" destroy-method="destroy"/>
```

```java
public class UserServiceImpl {
    public void init() {
        System.out.println("Bean 初始化完成");
    }
    
    public void destroy() {
        System.out.println("Bean 即将销毁");
    }
}
```

**完整生命周期：**
1. 对象创建
2. 依赖注入
3. 初始化方法执行
4. 业务使用
5. 销毁方法执行

## 3. 依赖注入方式

### 3.1 Setter 注入

```xml
<bean id="userService" class="com.example.service.UserServiceImpl">
    <!-- 引用类型注入 -->
    <property name="userDao" ref="userDao"/>
    
    <!-- 简单类型注入 -->
    <property name="maxRetryCount" value="3"/>
    <property name="enabled" value="true"/>
</bean>
```

```java
public class UserServiceImpl {
    private UserDao userDao;
    private int maxRetryCount;
    private boolean enabled;
    
    // Setter 方法
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    
    public void setMaxRetryCount(int maxRetryCount) {
        this.maxRetryCount = maxRetryCount;
    }
    
    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }
}
```

### 3.2 构造器注入

```xml
<bean id="orderService" class="com.example.service.OrderServiceImpl">
    <!-- 按索引注入 -->
    <constructor-arg index="0" ref="userService"/>
    <constructor-arg index="1" ref="productService"/>
    <constructor-arg index="2" value="100"/>
    
    <!-- 按名称注入 -->
    <constructor-arg name="timeout" value="5000"/>
</bean>
```

```java
public class OrderServiceImpl {
    private UserService userService;
    private ProductService productService;
    private int timeout;
    
    public OrderServiceImpl(UserService userService, 
                           ProductService productService, 
                           int timeout) {
        this.userService = userService;
        this.productService = productService;
        this.timeout = timeout;
    }
}
```

### 3.3 自动装配

```xml
<!-- 按类型自动装配 -->
<bean id="userService" class="com.example.service.UserServiceImpl" 
      autowire="byType"/>

<!-- 按名称自动装配 -->
<bean id="userService" class="com.example.service.UserServiceImpl" 
      autowire="byName"/>

<!-- 构造器自动装配 -->
<bean id="orderService" class="com.example.service.OrderServiceImpl" 
      autowire="constructor"/>
```

### 3.4 集合注入

```xml
<bean id="complexBean" class="com.example.ComplexBean">
    <!-- List 注入 -->
    <property name="stringList">
        <list>
            <value>value1</value>
            <value>value2</value>
            <value>value3</value>
        </list>
    </property>
    
    <!-- Set 注入 -->
    <property name="stringSet">
        <set>
            <value>value1</value>
            <value>value2</value>
        </set>
    </property>
    
    <!-- Map 注入 -->
    <property name="stringMap">
        <map>
            <entry key="key1" value="value1"/>
            <entry key="key2" value="value2"/>
        </map>
    </property>
    
    <!-- Properties 注入 -->
    <property name="properties">
        <props>
            <prop key="db.driver">com.mysql.jdbc.Driver</prop>
            <prop key="db.url">jdbc:mysql://localhost:3306/test</prop>
        </props>
    </property>
</bean>
```

## 4. 外部属性文件配置

### 4.1 properties 文件配置

```properties
# jdbc.properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_db
jdbc.username=root
jdbc.password=123456

app.name=Spring Application
app.version=1.0.0
```

### 4.2 XML 中加载 properties

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 加载 properties 文件 -->
    <context:property-placeholder 
        location="classpath:jdbc.properties,classpath:app.properties"
        ignore-unresolvable="true"
        system-properties-mode="NEVER"/> <!-- 关闭系统环境变量覆盖 -->

    <!-- 使用属性值 -->
    <bean id="dataSource" class="com.example.DataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

## 5. 容器创建与 Bean 获取

### 5.1 创建容器

```java
// 类路径加载配置文件
ApplicationContext context1 = 
    new ClassPathXmlApplicationContext("applicationContext.xml");

// 文件系统绝对路径加载
ApplicationContext context2 = 
    new FileSystemXmlApplicationContext("C:/config/applicationContext.xml");

// 加载多个配置文件
ApplicationContext context3 = 
    new ClassPathXmlApplicationContext(
        "applicationContext.xml", 
        "applicationContext-dao.xml",
        "applicationContext-service.xml"
    );
```

### 5.2 获取 Bean 的方式

```java
ApplicationContext context = 
    new ClassPathXmlApplicationContext("applicationContext.xml");

// 1. 使用 Bean 名称获取（需要强制类型转换）
UserDao userDao1 = (UserDao) context.getBean("userDao");

// 2. 使用 Bean 名称和类型获取（推荐）
UserDao userDao2 = context.getBean("userDao", UserDao.class);

// 3. 使用 Bean 类型获取（要求该类型只有一个 Bean）
UserDao userDao3 = context.getBean(UserDao.class);

// 4. 获取所有特定类型的 Bean
Map<String, UserDao> userDaos = context.getBeansOfType(UserDao.class);

// 5. 获取所有 Bean 名称
String[] beanNames = context.getBeanDefinitionNames();
```

## 6. 注解开发

### 6.1 组件注解

```java
// 配置类
@Configuration
@ComponentScan(basePackages = "com.example")
@PropertySource("classpath:app.properties")
@Import({JdbcConfig.class, CacheConfig.class})
public class AppConfig {
    // 配置类内容
}

// Dao 层
@Repository("userDao")
public class UserDaoImpl implements UserDao {
    // 实现
}

// Service 层
@Service("userService")
public class UserServiceImpl implements UserService {
    // 实现
}

// Controller 层
@Controller
@RequestMapping("/user")
public class UserController {
    // 控制器方法
}
```

### 6.2 依赖注入注解

```java
@Service
public class UserServiceImpl implements UserService {
    
    // 按类型自动注入
    @Autowired
    private UserDao userDao;
    
    // 按名称注入（解决多个同类型 Bean）
    @Autowired
    @Qualifier("primaryUserDao")
    private UserDao specificUserDao;
    
    // 简单类型注入
    @Value("${app.max.retry.count:3}") // 默认值 3
    private int maxRetryCount;
    
    // 构造器注入
    @Autowired
    public UserServiceImpl(@Qualifier("userDao") UserDao userDao) {
        this.userDao = userDao;
    }
}
```

### 6.3 第三方 Bean 配置

```java
@Configuration
public class JdbcConfig {
    
    // 第三方 Bean 配置
    @Bean
    public DataSource dataSource(
            @Value("${jdbc.driver}") String driver,
            @Value("${jdbc.url}") String url,
            @Value("${jdbc.username}") String username,
            @Value("${jdbc.password}") String password) {
        
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
    
    // 引用其他 Bean 的配置
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

### 6.4 注解配置容器

```java
// 传统 XML 配置方式
ApplicationContext xmlContext = 
    new ClassPathXmlApplicationContext("applicationContext.xml");

// 注解配置方式
ApplicationContext annotationContext = 
    new AnnotationConfigApplicationContext(AppConfig.class);

// 扫描包方式
AnnotationConfigApplicationContext scanContext = 
    new AnnotationConfigApplicationContext();
scanContext.scan("com.example");
scanContext.refresh();
```

## 7. AOP（面向切面编程）

### 7.1 AOP 核心概念

- **切入点（Pointcut）**：需要增强的方法位置
- **连接点（Joinpoint）**：程序执行过程中的具体位置
- **通知（Advice）**：增强的代码逻辑
- **切面（Aspect）**：通知和切入点的关联

### 7.2 AOP 配置

```xml
<!-- 启用 AOP 注解支持 -->
<aop:aspectj-autoproxy/>
```

```java
// 启用 AOP 注解（配置类中）
@EnableAspectJAutoProxy
@Configuration
public class AppConfig {
}
```

### 7.3 切面定义

```java
@Component
@Aspect
public class LoggingAspect {
    
    // 切入点表达式：匹配 com.example.service 包下所有类的所有方法
    @Pointcut("execution(* com.example.service.*.*(..))")
    private void serviceLayer() {}
    
    // 前置通知
    @Before("serviceLayer()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("方法执行前: " + joinPoint.getSignature().getName());
    }
    
    // 后置通知
    @After("serviceLayer()")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println("方法执行后: " + joinPoint.getSignature().getName());
    }
    
    // 环绕通知
    @Around("serviceLayer()")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("方法执行前环绕");
        
        try {
            Object result = joinPoint.proceed(); // 执行原始方法
            System.out.println("方法执行后环绕");
            return result;
        } catch (Exception e) {
            System.out.println("方法执行异常");
            throw e;
        }
    }
    
    // 返回后通知
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("方法返回: " + result);
    }
    
    // 异常后通知
    @AfterThrowing(pointcut = "serviceLayer()", throwing = "error")
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {
        System.out.println("方法异常: " + error.getMessage());
    }
}
```

### 7.4 切入点表达式详解

```java
@Aspect
public class PointcutExamples {
    
    // 匹配所有 public 方法
    @Pointcut("execution(public * *(..))")
    public void anyPublicOperation() {}
    
    // 匹配指定包下的所有方法
    @Pointcut("execution(* com.example.dao.*.*(..))")
    public void daoLayer() {}
    
    // 匹配指定包及子包下的所有方法
    @Pointcut("execution(* com.example.service..*.*(..))")
    public void serviceLayer() {}
    
    // 匹配特定类的方法
    @Pointcut("execution(* com.example.service.UserService.*(..))")
    public void userServiceMethods() {}
    
    // 匹配特定方法名
    @Pointcut("execution(* *..find*(..))")
    public void findMethods() {}
    
    // 匹配特定参数类型
    @Pointcut("execution(* *..save(*, java.lang.String))")
    public void saveMethods() {}
    
    // 组合切入点
    @Pointcut("daoLayer() && anyPublicOperation()")
    public void publicDaoMethods() {}
}
```

### 7.5 AOP 通知获取数据

```java
@Aspect
@Component
public class DataAspect {
    
    @Around("execution(* com.example.service.*.*(..))")
    public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {
        // 获取方法参数
        Object[] args = pjp.getArgs();
        
        // 获取方法签名
        String methodName = pjp.getSignature().getName();
        
        // 执行原始方法并获取返回值
        Object result = pjp.proceed();
        
        // 修改返回值（如果需要）
        return modifyResult(result);
    }
    
    @AfterReturning(
        pointcut = "execution(* com.example.service.*.*(..))",
        returning = "result"
    )
    public void afterReturningAdvice(JoinPoint jp, Object result) {
        // 处理返回值
        System.out.println("方法返回值: " + result);
    }
    
    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "ex"
    )
    public void afterThrowingAdvice(JoinPoint jp, Exception ex) {
        // 处理异常
        System.out.println("方法异常: " + ex.getMessage());
    }
}
```

## 8. Spring 事务管理

### 8.1 声明式事务配置

```java
// 配置事务管理器
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}

// 启用事务管理
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
}
```

### 8.2 事务使用

```java
@Service
@Transactional
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserDao userDao;
    
    @Autowired
    private OrderDao orderDao;
    
    // 类级别的事务配置会应用到所有方法
    @Override
    public void createUser(User user) {
        userDao.save(user);
    }
    
    // 方法级别的事务配置会覆盖类级别配置
    @Transactional(
        readOnly = false,
        propagation = Propagation.REQUIRED,
        isolation = Isolation.DEFAULT,
        timeout = 30,
        rollbackFor = {Exception.class},
        noRollbackFor = {RuntimeException.class}
    )
    @Override
    public void createUserWithOrder(User user, Order order) {
        userDao.save(user);
        orderDao.save(order);
        
        if (order.getAmount() > 1000) {
            throw new BusinessException("订单金额过大");
        }
    }
}
```

### 8.3 事务传播行为

```java
@Service
public class OrderServiceImpl implements OrderService {
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void placeOrder(Order order) {
        // 如果当前没有事务，就新建一个事务
        // 如果已经存在事务，就加入这个事务
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateOrder(Order order) {
        // 新建事务，如果当前存在事务，把当前事务挂起
    }
    
    @Transactional(propagation = Propagation.SUPPORTS)
    public Order findOrder(Long id) {
        // 支持当前事务，如果当前没有事务，就以非事务方式执行
        return orderDao.findById(id);
    }
    
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void logOrderAction(Order order) {
        // 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
    }
    
    @Transactional(propagation = Propagation.NEVER)
    public void validateOrder(Order order) {
        // 以非事务方式执行，如果当前存在事务，则抛出异常
    }
    
    @Transactional(propagation = Propagation.MANDATORY)
    public void processOrder(Order order) {
        // 必须在事务中执行，如果当前没有事务，则抛出异常
    }
    
    @Transactional(propagation = Propagation.NESTED)
    public void complexOrderOperation(Order order) {
        // 如果当前存在事务，则在嵌套事务内执行
        // 如果当前没有事务，则执行与 PROPAGATION_REQUIRED 类似的操作
    }
}
```

### 8.4 事务相关配置属性

| 属性          | 说明         | 可选值                                                       |
| ------------- | ------------ | ------------------------------------------------------------ |
| propagation   | 传播行为     | REQUIRED, SUPPORTS, MANDATORY, REQUIRES_NEW, NOT_SUPPORTED, NEVER, NESTED |
| isolation     | 隔离级别     | DEFAULT, READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE |
| timeout       | 超时时间     | 秒数，-1 表示不超时                                          |
| readOnly      | 是否只读     | true, false                                                  |
| rollbackFor   | 回滚异常类   | Exception.class 等                                           |
| noRollbackFor | 不回滚异常类 | RuntimeException.class 等                                    |

## 9. Spring 整合 MyBatis

### 9.1 配置整合

```java
@Configuration
@MapperScan("com.example.mapper") // 扫描 Mapper 接口
public class MyBatisConfig {
    
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        
        // 配置其他 MyBatis 设置
        org.apache.ibatis.session.Configuration configuration = 
            new org.apache.ibatis.session.Configuration();
        configuration.setMapUnderscoreToCamelCase(true);
        sessionFactory.setConfiguration(configuration);
        
        return sessionFactory.getObject();
    }
}
```

### 9.2 Mapper 使用

```java
@Repository
public interface UserMapper {
    
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);
    
    @Insert("INSERT INTO users(name, email) VALUES(#{name}, #{email})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insert(User user);
    
    @Update("UPDATE users SET name=#{name}, email=#{email} WHERE id=#{id}")
    void update(User user);
    
    @Delete("DELETE FROM users WHERE id=#{id}")
    void delete(Long id);
}
```

## 总结

Spring Framework 通过 IOC 容器管理对象的生命周期和依赖关系，通过 AOP 实现横切关注点的分离，通过声明式事务简化数据访问层的事务管理。这些特性共同构成了 Spring 框架的核心价值，使得企业级应用开发更加简单、高效和可维护。