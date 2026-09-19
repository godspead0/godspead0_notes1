# **主题**：Spring Boot 核心知识点全面解析，包括核心优势、配置文件详解（类型、优先级、加载路径、获取方式、数据校验与单位配置、自定义配置文件）、多环境开发（命名规范、激活方式、配置优先级）、启动机制（核心组件、自动配置原理、嵌入式服务器替换）以及第三方技术整合（JUnit 测试、MyBatis 整合）

# Spring Boot 核心知识点全解析（含补充扩展）

## 一、Spring Boot 核心优势

Spring Boot 是 Spring 生态的快速开发脚手架，核心优势包括：

- **自动配置**：基于 classpath 依赖自动配置 Spring 环境，无需手动编写大量 XML 配置
- **起步依赖**：通过 Maven/Gradle 坐标整合常用技术栈（如 `spring-boot-starter-web` 包含 Spring MVC + 嵌入式 Tomcat）
- **嵌入式服务器**：内置 Tomcat、Jetty、Undertow 等服务器，无需额外部署
- **简化开发**：自动管理依赖版本、简化配置、内置监控（Actuator）等
- **独立运行**：打包为可执行 JAR 包，直接通过 `java -jar` 运行，无需依赖外部容器

## 二、配置文件详解

### 2.1 配置文件类型与优先级

Spring Boot 支持三种配置文件格式，优先级从高到低：

1. **`.properties`**：键值对格式（`key=value`），优先级最高
2. **`.yml`**：yaml 语法，层次清晰（推荐使用）
3. **`.yaml`**：与 `.yml` 语法一致，仅后缀不同

注意：命令行参数（`java -jar app.jar --server.port=8081`）优先级高于所有配置文件，可覆盖配置。

### 2.2 YAML 语法规则

- 不允许使用 `Tab` 缩进，仅支持空格（建议 2 个空格）
- 键值对用 `:` 分隔，**值前面必须加空格**（如 `name: 张三`）
- 层次结构通过缩进体现，相同缩进为同一层级
- 数组用 `-` 表示（如：`list: [1,2,3]` 或换行写法）
- 字符串默认无需引号，若包含特殊字符（如空格、`:`）需用双引号 `""`
- 数字注意进制问题：以 `0` 开头的数字会被解析为八进制（如 `0127` 实际为十进制 87），密码等场景需用引号包裹为字符串（`password: "0127"`）

示例：

```YAML
server:
  port: 8080
  servlet:
    context-path: /demo
user:
  name: 张三
  age: 20
  hobbies:
    - 篮球
    - 编程
  address:
    province: 广东
    city: 深圳
  password: "0127"  # 避免八进制解析
```

### 2.3 配置文件加载路径（四级优先级）

Spring Boot 按以下路径顺序加载配置文件，后加载的会覆盖先加载的：

1. **file:./config/**（项目根目录下的 config 文件夹）→ 优先级最高
2. **file:./**（项目根目录）
3. **classpath:./config/**（resources/config 文件夹）
4. **classpath:./**（resources 根目录）→ 优先级最低

说明：`file:` 指应用运行目录（如打包后 `target` 目录同级），`classpath:` 指 `resources` 目录。

### 2.4 配置数据获取方式

#### 方式 1：`@Value` 注解（单个属性）

直接绑定配置文件中的单个属性，支持 SpEL 表达式：

```Java
@Component
public class UserConfig {
    // 绑定 yml 中的 user.name
    @Value("${user.name}")
    private String userName;
    
    // 绑定数组（需指定分隔符）
    @Value("${user.hobbies:篮球,读书}") // 冒号后为默认值
    private String[] hobbies;
    
    // 绑定嵌套属性
    @Value("${user.address.city}")
    private String city;
}
```

注意：`@Value` 不支持松散绑定，属性名必须与配置文件完全一致（区分大小写、连字符等）。

#### 方式 2：`Environment` 接口（批量属性）

注入 `Environment` 对象，通过 `getProperty()` 方法获取任意配置，无需提前绑定：

```Java
@Component
public class ConfigUtil {
    @Autowired
    private Environment env;
    
    public String getServerPort() {
        // 获取 server.port，默认值 8080
        return env.getProperty("server.port", "8080");
    }
    
    public Integer getUserId() {
        // 指定返回类型
        return env.getProperty("user.id", Integer.class, 1001);
    }
}
```

#### 方式 3：`@ConfigurationProperties`（批量绑定）

适用于复杂对象绑定，支持松散绑定、数据校验、类型转换：

1. 定义配置类并绑定前缀：

```Java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;
import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;

// 绑定前缀为 user 的配置，开启数据校验
@ConfigurationProperties(prefix = "user")
@Validated // 启用数据校验
@Component // 注册为 Spring Bean（或用 @EnableConfigurationProperties 替代）
public class UserProperties {
    @NotBlank(message = "用户名不能为空")
    private String name;
    
    @Max(value = 150, message = "年龄不能超过 150")
    private Integer age;
    
    private Address address; // 嵌套对象
    private List<String> hobbies;
    
    // getter/setter 必须存在（否则无法绑定）
}

// 嵌套对象
class Address {
    private String province;
    private String city;
    // getter/setter
}
```

1. 启用配置绑定（二选一）：
    1. 配置类加 `@Component` 注解（直接注册 Bean）
    2. 主启动类加 `@EnableConfigurationProperties(UserProperties.class)`（显式启用，无需 `@Component`）

松散绑定规则：配置文件中的 `user.user-name`、`user.userName`、`user.USER_NAME` 均可绑定到 `userName` 属性。

### 2.5 数据校验与单位配置

#### 数据校验

需引入依赖（Spring Boot 2.3+ 需手动引入）：

```XML
<!-- 校验 API -->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
</dependency>
<!-- 校验实现（Hibernate Validator） -->
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
```

常用校验注解：

- `@NotBlank`：字符串非空且不为空白
- `@NotNull`：对象不为 null
- `@Min(value)`/`@Max(value)`：数字范围
- `@Pattern(regexp)`：正则表达式匹配
- `@Email`：邮箱格式校验

#### 单位配置

- 时间单位：`@DurationUnit` + `Duration` 类型
- 存储单位：`@DataSizeUnit` + `DataSize` 类型

示例：

```Java
import java.time.Duration;
import java.time.temporal.ChronoUnit;
import org.springframework.boot.convert.DataSizeUnit;
import org.springframework.boot.convert.DurationUnit;
import org.springframework.util.unit.DataSize;
import org.springframework.util.unit.DataUnit;

@ConfigurationProperties(prefix = "app")
public class AppProperties {
    // 配置文件中写 app.timeout=5 → 解析为 5 分钟
    @DurationUnit(ChronoUnit.MINUTES)
    private Duration timeout;
    
    // 配置文件中写 app.file-size=10 → 解析为 10 MB
    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize fileSize;
    
    // getter/setter
}
```

### 2.6 自定义配置文件

默认加载 `application.*` 配置文件，若需加载自定义文件（如 `config-dev.yml`），可通过以下方式：

#### 方式 1：通过注解指定

```Java
@PropertySource(value = "classpath:config-dev.properties", encoding = "UTF-8")
@Component
public class CustomConfig {
    @Value("${custom.key}")
    private String customKey;
}
```

注意：`@PropertySource` 默认不支持 YAML 文件，需自定义 `PropertySourceFactory`。

#### 方式 2：命令行指定

```Bash
java -jar app.jar --spring.config.location=classpath:config-dev.yml
```

#### 方式 3：主启动类指定

```Java
public class Application {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        // 加载多个自定义配置文件
        application.setDefaultProperties(
            Collections.loadProperties(new FileInputStream("classpath:config-dev.yml"))
        );
        application.run(args);
    }
}
```

## 三、多环境开发

### 3.1 多环境配置文件命名规范

主配置文件：`application.yml`（公共配置）  

子环境配置文件：`application-{profile}.yml`（环境专属配置），如：

- `application-dev.yml`（开发环境）
- `application-test.yml`（测试环境）
- `application-prod.yml`（生产环境）

### 3.2 激活环境的 3 种方式

#### 方式 1：主配置文件中指定

```YAML
# application.yml（公共配置）
spring:
  profiles:
    active: dev # 激活开发环境
    # include: dev-mvc,dev-db # 额外加载多个环境（Spring Boot 2.4 前）
    group: # Spring Boot 2.4+ 推荐使用 group 替代 include，支持分组管理
      dev: dev-mvc,dev-db # 激活 dev 时，同时加载 dev-mvc 和 dev-db
```

#### 方式 2：命令行指定

```Bash
# 激活生产环境
java -jar app.jar --spring.profiles.active=prod
```

#### 方式 3：Maven  Profiles 整合（推荐）

通过 Maven 控制环境，实现打包时自动切换环境：

1. Maven `pom.xml` 中配置 profiles：

```XML
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault> <!-- 默认激活开发环境 -->
        </activation>
    </profile>
    <!-- 生产环境 -->
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
    </profile>
</profiles>
```

1. 主配置文件中引用 Maven 变量：

```YAML
# application.yml
spring:
  profiles:
    active: @spring.profiles.active@ # @变量名@ 引用 Maven 中定义的属性
```

1. 打包时指定环境：

```Bash
# 打包生产环境（激活 prod  profile）
mvn package -Pprod
```

优先级：Maven Profiles > 命令行参数 > 配置文件指定（因 Maven 打包时已替换配置文件中的变量）

### 3.3 环境配置优先级

- 公共配置（`application.yml`）先加载
- 子环境配置（`application-{profile}.yml`）后加载，覆盖公共配置
- 多个子环境（如 `dev` + `dev-mvc`）按配置顺序加载，后面的覆盖前面的

## 四、Spring Boot 启动机制

### 4.1 核心组件

#### 1. 启动类（引导类）

必须包含 `@SpringBootApplication` 注解，是 Spring Boot 应用的入口：

```Java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        // 启动 Spring 容器，返回 ApplicationContext
        SpringApplication.run(Application.class, args);
    }
}
```

#### 2. `@SpringBootApplication` 注解拆解

该注解是复合注解，包含以下核心注解：

- `@SpringBootConfiguration`：标记当前类为配置类（等价于 `@Configuration`）
- `@EnableAutoConfiguration`：开启自动配置（核心），扫描 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件中的自动配置类
- `@ComponentScan`：扫描当前包及其子包下的 `@Component`、`@Service`、`@Controller` 等注解，注册为 Spring Bean

自定义扫描范围：`@SpringBootApplication(scanBasePackages = "com.example")`

### 4.2 自动配置原理

1. Spring Boot 启动时，`@EnableAutoConfiguration` 注解触发自动配置
2. 加载 classpath 下所有 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件中的自动配置类（如 `DataSourceAutoConfiguration`、`WebMvcAutoConfiguration`）
3. 自动配置类通过 `@Conditional` 系列注解判断是否需要生效（如 `@ConditionalOnClass` 检查类路径是否存在指定类，`@ConditionalOnMissingBean` 检查容器中是否缺少指定 Bean）
4. 生效的自动配置类会向 Spring 容器中注册相关 Bean，完成环境配置

示例：`DataSourceAutoConfiguration` 会在类路径存在 `DataSource` 类且容器中无 `DataSource` Bean 时，自动配置数据源。

### 4.3 嵌入式服务器替换

默认嵌入式服务器为 Tomcat，若需替换为 Jetty 或 Undertow：

#### 替换为 Jetty

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 排除 Tomcat 依赖 -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 引入 Jetty 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

#### 替换为 Undertow

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

## 五、第三方技术整合

### 5.1 JUnit 测试

Spring Boot 提供 `spring-boot-starter-test` 依赖，整合 JUnit、Mockito 等测试工具：

#### 1. 引入依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### 2. 测试类编写

```Java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import javax.transaction.Transactional;

// 启动 Spring 容器，classes 指定启动类（路径不同时必须指定）
@SpringBootTest(classes = Application.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Transactional // 测试后自动回滚数据库操作（避免污染数据）
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    void testQueryUser() {
        // 测试逻辑
        User user = userService.getUserById(1L);
        assert user != null;
    }
}
```

#### 测试类常用配置

- `webEnvironment`：指定 Web 环境（`RANDOM_PORT` 随机端口，`DEFINED_PORT` 使用配置端口，`NONE` 无 Web 环境）
- `properties`：临时添加配置（`@SpringBootTest(properties = {"user.name=测试用户"})`）
- `args`：临时添加命令行参数（`@SpringBootTest(args = "--server.port=8082")`），优先级高于 `properties`
- `@Import`：临时导入 Bean（`@Import({TestBean.class})`）
- `@AutoConfigureMockMvc`：启用虚拟 MVC 测试

### 5.2 MyBatis 整合

#### 1. 引入依赖

```XML
<!-- MyBatis 起步依赖 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.3.2</version> <!-- 与 Spring Boot 版本兼容 -->
</dependency>
<!-- 数据库驱动（以 MySQL 为例） -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

#### 2. 配置数据源与 MyBatis

```YAML
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test_db?useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis:
  mapper-locations: classpath:mapper/*.xml # Mapper XML 文件路径
  type-aliases-package: com.example.entity # 实体类别名包
  configuration:
    map-underscore-to-camel-case: true # 下划线转驼峰
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 打印 SQL 日志
```

#### 3. 编写 Mapper 接口

```Java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import com.example.entity.User;
import java.util.List;

@Mapper // 标记为 MyBatis Mapper 接口（或主启动类加 @MapperScan("com.example.mapper")）
public interface UserMapper {
    List<User> selectAll();
    User selectById(@Param("id") Long id);
    int insert(User user);
}
```

#### 4. 编写 Mapper XML（`resources/mapper/UserMapper.xml`）

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    <select id="selectAll" resultType="User">
        select * from user
    </select>
    <select id="selectById" resultType="User">
        select * from user where id = #{id}
    </select>
</mapper>
```

### 5.3 其他常用技术整合

| 技术            | 起步依赖                         | 核心配置要点                     |
| --------------- | -------------------------------- | -------------------------------- |
| Spring MVC      | `spring-boot-starter-web`        | 端口、上下文路径、静态资源路径   |
| Spring Data JPA | `spring-boot-starter-data-jpa`   | 数据源、JPA 方言、显示 SQL       |
| Redis           | `spring-boot-starter-data-redis` | 主机、端口、密码、序列化方式     |
| RabbitMQ        | `spring-boot-starter-amqp`       | 主机、端口、虚拟主机、用户名密码 |
| Spring Cache    | `spring-boot-starter-cache`      | 缓存类型（Redis、Caffeine 等）   |

## 六、热部署配置

热部署指修改代码后无需重启应用即可生效，提高开发效率。

### 6.1 引入依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> <!-- 防止依赖传递 -->
    <scope>runtime</scope>
</dependency>
```

### 6.2 激活热部署

- IDEA：修改代码后按 `Ctrl+F9`（重新编译），或开启自动编译：
    - `File → Settings → Build, Execution, Deployment → Compiler` → 勾选 `Build project automatically`
    - 按 `Ctrl+Alt+Shift+/` → 选择 `Registry` → 勾选 `compiler.automake.allow.when.app.running`
- Eclipse：修改代码后自动编译，热部署生效

### 6.3 自定义热部署范围

```YAML
spring:
  devtools:
    restart:
      enabled: true # 启用热部署（默认 true）
      exclude: WEB-INF/**,*.xml # 排除不需要热部署的文件/目录
      additional-paths: src/main/java # 额外监控的目录（默认监控 classpath）
```

### 6.4 关闭热部署

#### 方式 1：配置文件（可能不生效，因系统变量优先级更高）

```YAML
spring:
  devtools:
    restart:
      enabled: false
```

#### 方式 2：系统变量（推荐）

在启动类 `main` 方法中设置系统变量：

```Java
public class Application {
    public static void main(String[] args) {
        // 关闭热部署（系统变量优先级最高）
        System.setProperty("spring.devtools.restart.enabled", "false");
        SpringApplication.run(Application.class, args);
    }
}
```

## 七、日志配置

Spring Boot 默认使用 SLF4J + Logback 作为日志框架，无需额外引入依赖（`spring-boot-starter` 已包含）。

### 7.1 基础配置（application.yml）

```YAML
logging:
  level: # 日志级别（TRACE < DEBUG < INFO < WARN < ERROR < FATAL）
    root: INFO # 根日志级别
    com.example.mapper: DEBUG # Mapper 包日志级别（打印 SQL）
    com.example.service: WARN # Service 包日志级别
  file:
    name: logs/app.log # 日志文件路径（相对路径或绝对路径）
    max-size: 10MB # 单个日志文件最大大小
    max-history: 7 # 日志文件保留天数
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n" # 控制台输出格式
    file: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n" # 文件输出格式
```

### 7.2 自定义 Logback 配置

若需更复杂的日志配置（如多环境日志、滚动策略），可在 `resources` 目录下创建 `logback-spring.xml` 文件（Spring 扩展的 Logback 配置文件，支持 Spring 表达式）：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 日志输出格式 -->
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n" />
    <!-- 日志存储路径 -->
    <property name="LOG_PATH" value="logs/app.log" />
    
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <!-- 文件输出（滚动策略） -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <!-- 多环境日志级别 -->
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="FILE" />
        </root>
    </springProfile>
    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="FILE" />
        </root>
    </springProfile>
</configuration>
```

## 八、项目打包与运行

### 8.1 打包为可执行 JAR

Spring Boot 项目默认打包为可执行 JAR（包含所有依赖），通过 `spring-boot-maven-plugin` 插件实现：

```XML
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <!-- 可选：指定主启动类 -->
            <configuration>
                <mainClass>com.example.Application</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

打包命令：

```Bash
mvn clean package -Dmaven.test.skip=true # 跳过测试打包
```

### 8.2 运行 JAR 包

```Bash
# 基本运行
java -jar app.jar

# 指定端口运行
java -jar app.jar --server.port=8081

# 激活生产环境运行
java -jar app.jar --spring.profiles.active=prod

# 后台运行（Linux/Mac）
nohup java -jar app.jar > app.log 2>&1 &
```

### 8.3 禁用外部命令行参数

若需禁止通过命令行修改配置（如端口、环境），可在启动类中屏蔽 `args` 参数：

```Java
public class Application {
    public static void main(String[] args) {
        // 屏蔽外部参数，使用固定配置
        SpringApplication.run(Application.class);
    }
}
```

## 九、Web 测试（虚拟请求）

通过 `MockMvc` 模拟 HTTP 请求，无需启动真实服务器即可测试 Controller 接口。

### 9.1 测试类编写

```Java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc // 启用 MockMvc
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void testHello() throws Exception {
        // 模拟 GET 请求：/hello?name=张三
        mockMvc.perform(get("/hello")
                .param("name", "张三") // 请求参数
                .header("token", "test-token")) // 请求头
                .andExpect(status().isOk()) // 预期响应状态码 200
                .andExpect(content().string("Hello, 张三")) // 预期响应体
                .andExpect(header().exists("Content-Type")); // 预期响应头存在
    }

    // 测试 JSON 响应
    @Test
    void testQueryUser() throws Exception {
        mockMvc.perform(get("/user/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1)) // 匹配 JSON 字段
                .andExpect(jsonPath("$.name").value("张三"));
    }
}
```

### 9.2 常用匹配器

- `status().isOk()`：响应状态码 200
- `content().string("xxx")`：响应体为指定字符串
- `jsonPath("$.field")`：匹配 JSON 响应体字段（需引入 `json-path` 依赖）
- `header().string("name", "value")`：响应头匹配
- `cookie().value("name", "value")`：Cookie 匹配

## 十、测试数据随机生成

使用 Spring Boot 内置的 `RandomValuePropertySource` 生成随机测试数据，避免硬编码：

### 10.1 配置文件中使用

```YAML
test:
  user:
    id: ${random.int} # 随机整数
    age: ${random.int(18,30)} # 18-30 之间的随机整数
    name: ${random.value} # 随机字符串（32 位）
    email: ${random.uuid}@example.com # 随机 UUID 作为邮箱前缀
    score: ${random.double(0,100)} # 0-100 之间的随机小数
```

### 10.2 代码中使用

```Java
@Component
public class TestDataConfig {
    @Value("${test.user.id}")
    private Integer userId;
    
    @Value("${test.user.name}")
    private String userName;
    
    // getter/setter
}
```

## 十一、常见问题与注意事项

1. **配置文件编码问题**：建议统一使用 UTF-8 编码，避免中文乱码
2. **端口占用**：通过 `server.port=0` 让 Spring Boot 自动分配可用端口
3. **依赖冲突**：使用 `mvn dependency:tree` 分析依赖树，排除冲突依赖
4. **自动配置失效**：检查是否缺少依赖、是否存在自定义 Bean 覆盖自动配置、是否禁用了自动配置（`@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)`）
5. **日志不打印**：检查日志级别是否过高、日志配置路径是否正确、是否有其他日志框架冲突（如 Log4j2）
6. **热部署不生效**：检查依赖是否正确引入、IDE 自动编译是否开启、是否排除了修改的文件

## 十二、扩展推荐

- **Spring Boot Actuator**：监控应用健康状态、指标、日志等（依赖 `spring-boot-starter-actuator`）
- **Spring Boot Admin**：可视化监控平台，整合 Actuator 数据
- **Spring Cloud**：微服务架构，基于 Spring Boot 实现服务注册、配置中心、网关等
- **Lombok**：简化 Java 代码（`@Data`、`@NoArgsConstructor` 等注解）
- **MapStruct**：对象映射工具，编译期生成映射代码，性能优于 BeanUtils