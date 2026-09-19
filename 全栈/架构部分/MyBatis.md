# **主题**：MyBatis 全面详解与实战指南，包括核心概述、环境搭建（Spring Boot 整合）、核心开发方式（注解式、XML 映射、Mapper 代理）、核心语法详解（参数传递、#{ }与${ }的区别、动态 SQL、结果集映射）

# MyBatis 全面详解与实战指南

## 一、MyBatis 核心概述

### 1.1 什么是 MyBatis

MyBatis 是一款开源的 Java 持久层框架，专注于解决 JDBC 编程的繁琐问题（如手动加载驱动、创建连接、处理结果集等），通过简化数据库操作，实现 Java 对象与数据库表的映射关系。

其核心优势：

- 无需手动编写 JDBC 代码，通过 XML 或注解配置 SQL
- 支持动态 SQL 编写，适配复杂查询场景
- 提供灵活的结果集映射，解决字段名与实体类属性名不一致问题
- 支持 Mapper 代理开发，无需手动实现 DAO 层接口
- 集成数据库连接池，优化连接管理
- 轻量级框架，配置简单，易于扩展

### 1.2 核心概念

- **Mapper 接口**：定义数据库操作方法的接口，通过 `@Mapper` 注解或 XML 配置生成代理对象，无需手动实现。
- **SQL 映射**：将 SQL 语句与 Mapper 接口方法关联，支持注解式（如 `@Select`）和 XML 式配置。
- **SqlSessionFactory**：MyBatis 核心工厂类，通过加载核心配置文件创建，用于生成 SqlSession。
- **SqlSession**：数据库操作会话对象，提供增删改查方法，是 MyBatis 与数据库交互的核心 API。
- **结果集映射**：将数据库查询结果转换为 Java 实体类对象，支持 `resultMap` 自定义映射规则。

## 二、MyBatis 环境搭建（Spring Boot 整合）

### 2.1 搭建步骤

#### 1. 创建 Maven 工程

选择 Spring Boot 初始化器，创建基础 Spring Boot 工程。

#### 2. 引入核心依赖

在 `pom.xml` 中添加 MyBatis 与 MySQL 驱动依赖（Spring Boot 整合版）：

```XML
<!-- MyBatis 整合 Spring Boot  starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.3.1</version> <!-- 适配 Spring Boot 版本 -->
</dependency>

<!-- MySQL 驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Spring Boot 测试依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### 3. 配置数据库连接

在 `application.properties` 中配置数据库连接信息及 MyBatis 基础配置：

```Properties
# 数据库连接配置
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver <!-- 注意：MySQL 8.0+ 驱动类名变更 -->
spring.datasource.username=root
spring.datasource.password=root

# MyBatis 配置
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl <!-- 打印 SQL 日志 -->
mybatis.type-aliases-package=com.example.pojo <!-- 实体类别名扫描包 -->
mybatis.mapper-locations=classpath:mapper/*.xml <!-- SQL 映射文件路径 -->
```

#### 4. 定义实体类（POJO）

创建与数据库表对应的实体类，示例：

```Java
package com.example.pojo;

import lombok.Data; // 推荐使用 Lombok 简化 getter/setter

@Data
public class User {
    private Integer id;
    private String username; // 若数据库字段为 username1，需通过 resultMap 映射
    private Integer age;
    private String email;
}
```

### 2.2 数据库连接池配置

MyBatis 本身不提供连接池，需依赖第三方实现，常用连接池对比：

| 连接池   | 特点                         | 配置方式                          |
| -------- | ---------------------------- | --------------------------------- |
| HikariCP | 性能最优（Spring Boot 默认） | 无需额外依赖，直接使用数据源配置  |
| Druid    | 功能强大（监控、防SQL注入）  | 需引入 Druid 依赖，配置 type 属性 |

#### Druid 连接池配置示例

1. 引入 Druid 依赖：

```XML
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.16</version>
</dependency>
```

1. 修改 `application.properties` 配置：

```Properties
# 指定连接池类型
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

# Druid 额外配置（可选）
spring.datasource.druid.initial-size=5 <!-- 初始连接数 -->
spring.datasource.druid.max-active=20 <!-- 最大活跃连接数 -->
spring.datasource.druid.min-idle=5 <!-- 最小空闲连接数 -->
spring.datasource.druid.max-wait=60000 <!-- 最大等待时间（毫秒） -->
```

## 三、MyBatis 核心开发方式

### 3.1 注解式开发（简单 SQL 场景）

通过 `@Select`、`@Insert`、`@Update`、`@Delete` 等注解直接在 Mapper 接口中编写 SQL，无需 XML 文件。

#### 步骤：

1. 定义 Mapper 接口（添加 `@Mapper` 注解）：

```Java
package com.example.mapper;

import com.example.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import java.util.List;

@Mapper // 自动生成代理对象，注入 IOC 容器
public interface UserMapper {

    // 根据 ID 查询用户
    @Select("SELECT * FROM user WHERE id = #{id}")
    User selectById(Integer id);

    // 多条件查询（需用 @Param 指明参数名）
    @Select("SELECT * FROM user WHERE username LIKE CONCAT('%',#{name},'%') AND age > #{age}")
    List<User> selectByCondition(@Param("name") String username, @Param("age") Integer age);

    // 新增用户（返回影响行数）
    @Insert("INSERT INTO user(username, age, email) VALUES(#{username}, #{age}, #{email})")
    int insert(User user);

    // 更新用户
    @Update("UPDATE user SET username = #{username}, age = #{age} WHERE id = #{id}")
    int update(User user);

    // 删除用户
    @Delete("DELETE FROM user WHERE id = #{id}")
    int delete(Integer id);
}
```

1. 单元测试（使用 `@SpringBootTest`）：

```Java
package com.example;

import com.example.mapper.UserMapper;
import com.example.pojo.User;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.util.List;

@SpringBootTest // Spring Boot 测试注解，加载 Spring 上下文
public class UserMapperTest {

    @Autowired // 注入 Mapper 代理对象
    private UserMapper userMapper;

    @Test
    public void testSelectById() {
        User user = userMapper.selectById(1);
        System.out.println("查询结果：" + user);
    }

    @Test
    public void testSelectByCondition() {
        List<User> users = userMapper.selectByCondition("张三", 18);
        users.forEach(System.out::println);
    }
}
```

### 3.2 XML 映射开发（复杂 SQL 场景）

对于动态 SQL、多表关联等复杂场景，推荐使用 XML 配置 SQL，便于维护和扩展。

#### 核心规范：

1. XML 文件名需与 Mapper 接口名一致，且放在 `classpath:mapper/` 目录下。
2. XML 中 `<mapper>` 标签的 `namespace` 属性需与 Mapper 接口全路径一致。
3. XML 中 SQL 标签（`<select>`、`<insert>` 等）的 `id` 属性需与 Mapper 接口方法名一致。
4. 参数类型（`parameterType`）和返回类型（`resultType`/`resultMap`）需与接口方法一致。

#### 开发步骤：

1. 定义 Mapper 接口：

```Java
package com.example.mapper;

import com.example.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import java.util.List;
import java.util.Map;

@Mapper
public interface UserXmlMapper {
    // 根据 ID 查询用户
    User selectById(Integer id);

    // 多条件动态查询
    List<User> selectByDynamicCondition(Map<String, Object> params);

    // 新增用户并返回主键
    int insertUser(User user);

    // 批量删除用户
    int batchDelete(List<Integer> ids);
}
```

1. 创建 XML 映射文件（`src/main/resources/mapper/UserXmlMapper.xml`）：

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace 必须与 Mapper 接口全路径一致 -->
<mapper namespace="com.example.mapper.UserXmlMapper">

    <!-- 自定义结果集映射（解决字段名与属性名不一致问题） -->
    <resultMap id="UserResultMap" type="com.example.pojo.User">
        <id column="id" property="id" /> <!-- 主键映射 -->
        <result column="username1" property="username" /> <!-- 数据库字段 username1 → 实体属性 username -->
        <result column="age" property="age" />
        <result column="email" property="email" />
    </resultMap>

    <!-- 根据 ID 查询（使用 resultMap 映射） -->
    <select id="selectById" parameterType="java.lang.Integer" resultMap="UserResultMap">
        SELECT id, username1, age, email FROM user WHERE id = #{id}
    </select>

    <!-- 多条件动态查询（where + if 标签） -->
    <select id="selectByDynamicCondition" parameterType="java.util.Map" resultMap="UserResultMap">
        SELECT * FROM user
        <where> <!-- 自动处理 AND/OR 关键字，避免语法错误 -->
            <if test="username != null and username != ''">
                AND username1 LIKE CONCAT('%', #{username}, '%')
            </if>
            <if test="age != null">
                AND age > #{age}
            </if>
            <if test="email != null and email != ''">
                AND email = #{email}
            </if>
        </where>
    </select>

    <!-- 新增用户并返回主键（useGeneratedKeys + keyProperty） -->
    <insert id="insertUser" parameterType="com.example.pojo.User" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO user(username1, age, email) VALUES(#{username}, #{age}, #{email})
    </insert>

    <!-- 批量删除（foreach 标签） -->
    <delete id="batchDelete" parameterType="java.util.List">
        DELETE FROM user WHERE id IN
        <foreach collection="list" item="id" open="(" separator="," close=")">
            #{id}
        </foreach>
    </delete>
</mapper>
```

1. 单元测试：

```Java
@Test
public void testSelectByDynamicCondition() {
    Map<String, Object> params = new HashMap<>();
    params.put("username", "张三");
    params.put("age", 18);
    List<User> users = userXmlMapper.selectByDynamicCondition(params);
    users.forEach(System.out::println);
}

@Test
public void testInsertUser() {
    User user = new User();
    user.setUsername("李四");
    user.setAge(20);
    user.setEmail("lisi@example.com");
    int rows = userXmlMapper.insertUser(user);
    System.out.println("影响行数：" + rows);
    System.out.println("新增用户 ID：" + user.getId()); // 自动获取主键
}
```

### 3.3 Mapper 代理开发（推荐）

MyBatis 提供 Mapper 代理机制，通过动态代理生成 Mapper 接口的实现类，无需手动编写 DAO 实现类，核心优势是简化代码、统一 SQL 配置与接口定义。

#### 代理开发核心要求：

1. Mapper 接口与 XML 映射文件同名且同包（资源目录下需用 `/` 分隔包名，如 `mapper/com/example/UserMapper.xml`）。
2. XML 中 `namespace` 与 Mapper 接口全路径一致。
3. SQL 标签 `id` 与 Mapper 接口方法名一致。
4. 参数类型和返回类型与接口方法一致。

#### 代理开发流程：

1. 加载核心配置文件（Spring Boot 环境下自动加载，无需手动处理）。
2. 获取 SqlSessionFactory（Spring Boot 自动配置）。
3. 通过 SqlSession 获取 Mapper 代理对象：

```Java
// 非 Spring Boot 环境手动获取（了解即可）
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession sqlSession = sqlSessionFactory.openSession();

// 获取 Mapper 代理对象
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

// 执行方法（自动关联 XML 中的 SQL）
List<User> users = userMapper.selectAll();

// 释放资源
sqlSession.close();
```

## 四、MyBatis 核心语法详解

### 4.1 参数传递

MyBatis 支持多种参数传递方式，适配不同场景：

| 参数传递方式  | 适用场景                         | 示例                                                         |
| ------------- | -------------------------------- | ------------------------------------------------------------ |
| 单个参数      | 方法只有一个参数                 | `User selectById(Integer id);` → `#{id}`                     |
| `@Param` 注解 | 多参数传递，明确参数名           | `List<User> selectByCondition(@Param("name") String username, @Param("age") Integer age);` → `#{name}` |
| Map 集合      | 多参数传递，参数数量不固定       | `List<User> selectByMap(Map<String, Object> params);` → `#{username}` |
| 实体类        | 多参数传递，参数与实体属性对应   | `int insert(User user);` → `#{username}`                     |
| 数组/List     | 批量操作（如批量删除、批量查询） | `int batchDelete(List<Integer> ids);` → 配合 `<foreach>` 标签 |

### 4.2 `#{}` 与 `${}` 的区别

| 特性     | `#{}`                                  | `${}`                                |
| -------- | -------------------------------------- | ------------------------------------ |
| 原理     | 预编译 SQL，替换为 `?` 占位符          | 字符串拼接，直接替换参数值           |
| SQL 注入 | 安全（预编译机制）                     | 不安全（直接拼接，可能注入恶意 SQL） |
| 类型转换 | 自动进行类型转换                       | 需手动处理类型转换                   |
| 适用场景 | 大部分查询、新增、修改场景（参数传递） | 表名、列名动态切换（如动态表查询）   |

#### 示例：

```SQL
-- #{id} → 预编译为 SELECT * FROM user WHERE id = ?
SELECT * FROM user WHERE id = #{id}

-- ${tableName} → 拼接为 SELECT * FROM t_user（若 tableName = "t_user"）
SELECT * FROM ${tableName}
```

### 4.3 动态 SQL

MyBatis 提供动态 SQL 标签，用于构建灵活的 SQL 语句，适配多条件查询、动态排序等场景：

#### 常用动态 SQL 标签：

| 标签                              | 功能                                 | 示例                                                         |
| --------------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| `<if>`                            | 条件判断，满足条件则拼接 SQL         | `<if test="username != null">AND username = #{username}</if>` |
| `<where>`                         | 自动处理 AND/OR 关键字，避免语法错误 | 包裹多个 `<if>` 标签                                         |
| `<set>`                           | 用于 UPDATE 语句，自动处理逗号       | `<set><if test="username != null">username = #{username},</if></set>` |
| `<foreach>`                       | 循环遍历集合/数组，用于批量操作      | 批量删除、批量插入                                           |
| `<choose>`/`<when>`/`<otherwise>` | 多条件分支判断（类似 if-else）       | `<choose><when test="age > 30">AND age > 30</when><otherwise>AND age <=30</otherwise></choose>` |

#### 批量插入示例：

```XML
<insert id="batchInsert" parameterType="java.util.List">
    INSERT INTO user(username1, age, email) VALUES
    <foreach collection="list" item="user" separator=",">
        (#{user.username}, #{user.age}, #{user.email})
    </foreach>
</insert>
```

### 4.4 结果集映射

当数据库字段名与实体类属性名不一致时，需通过以下方式解决：

#### 1. 别名法（简单场景）

在 SQL 中使用 `AS` 给字段起别名，与实体属性名一致：

```SQL
SELECT id, username1 AS username, age FROM user WHERE id = #{id}
```

#### 2. `resultMap` 法（复杂场景）

通过 `<resultMap>` 标签自定义映射规则，支持主键、关联查询等：

```XML
<resultMap id="UserResultMap" type="com.example.pojo.User">
    <id column="id" property="id" /> <!-- 主键映射（必须指定） -->
    <result column="username1" property="username" /> <!-- 普通字段映射 -->
    <result column="user_age" property="age" /> <!-- 下划线转驼峰（也可开启自动转换） -->
</resultMap>

<!-- 开启下划线自动转驼峰（application.properties） -->
mybatis.configuration.map-underscore-to-camel-case=true
```

### 4.5 增删查改（CRUD）核心操作

#### 1. 查询（Select）

- 单条查询：`resultType` 指定返回实体类或基本类型。
- 多条查询：`resultType` 指定集合泛型类型（如 `com.example.pojo.User`）。
- 分页查询：需配合分页插件（如 PageHelper），或手动编写分页 SQL。

#### 2. 新增（Insert）

- 返回影响行数：默认返回 int 类型（影响的记录数）。
- 返回主键：添加 `useGeneratedKeys="true"` 和 `keyProperty="主键属性名"`。

#### 3. 修改（Update）

- 返回影响行数：int 类型，代表修改成功的记录数。
- 动态修改：使用 `<set>` 标签配合 `<if>` 实现部分字段修改。

#### 4. 删除（Delete）

- 单条删除：根据 ID 或条件删除。
- 批量删除：使用 `<foreach>` 标签遍历 ID 集合。

## 五、MyBatis 核心配置文件（mybatis-config.xml）

MyBatis 核心配置文件用于全局配置，Spring Boot 环境下可通过 `application.properties` 简化配置，原生环境需手动编写 `mybatis-config.xml`：

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 1. 环境配置（可配置多环境） -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" /> <!-- 事务类型：JDBC/MANAGED -->
            <dataSource type="POOLED"> <!-- 数据源类型：POOLED（连接池）/UNPOOLED/JNDI -->
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/test?serverTimezone=Asia/Shanghai" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>

    <!-- 2. 别名配置（简化类全路径编写） -->
    <typeAliases>
        <!-- 单个类别名 -->
        <typeAlias type="com.example.pojo.User" alias="User" />
        <!-- 包扫描（别名默认为类名，不区分大小写） -->
        <package name="com.example.pojo" />
    </typeAliases>

    <!-- 3. 映射器配置（关联 XML 映射文件） -->
    <mappers>
        <!-- 单个 XML 文件 -->
        <mapper resource="mapper/UserMapper.xml" />
        <!-- 包扫描（Mapper 接口与 XML 同包） -->
        <package name="com.example.mapper" />
    </mappers>

</configuration>
```

## 六、MyBatis 高级特性

### 6.1 分页查询

MyBatis 本身不支持物理分页，需借助分页插件（如 PageHelper），步骤如下：

1. 引入依赖：

```XML
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.6</version>
</dependency>
```

1. 配置分页参数（`application.properties`）：

```Properties
pagehelper.helper-dialect=mysql <!-- 数据库方言 -->
pagehelper.reasonable=true <!-- 合理化分页（避免页码越界） -->
pagehelper.support-methods-arguments=true <!-- 支持通过方法参数传递分页参数 -->
```

1. 使用分页：

```Java
@Test
public void testPageQuery() {
    // 开启分页（第 1 页，每页 10 条）
    PageHelper.startPage(1, 10);
    // 执行查询（自动分页）
    List<User> users = userMapper.selectAll();
    // 封装分页结果
    PageInfo<User> pageInfo = new PageInfo<>(users);
    System.out.println("总记录数：" + pageInfo.getTotal());
    System.out.println("总页数：" + pageInfo.getPages());
    System.out.println("当前页数据：" + users);
}
```

### 6.2 关联查询（一对一/一对多）

MyBatis 支持多表关联查询，通过 `resultMap` 中的 `<association>`（一对一）和 `<collection>`（一对多）标签实现。

#### 示例：用户与订单（一对多）

1. 实体类：

```Java
@Data
public class User {
    private Integer id;
    private String username;
    private List<Order> orders; // 一对多：一个用户多个订单
}

@Data
public class Order {
    private Integer id;
    private String orderNo;
    private Integer userId;
}
```

1. XML 映射文件：

```XML
<resultMap id="UserWithOrderResultMap" type="com.example.pojo.User">
    <id column="user_id" property="id" />
    <result column="username" property="username" />
    <!-- 一对多关联：collection 标签 -->
    <collection property="orders" ofType="com.example.pojo.Order">
        <id column="order_id" property="id" />
        <result column="order_no" property="orderNo" />
        <result column="user_id" property="userId" />
    </collection>
</resultMap>

<select id="selectUserWithOrders" resultMap="UserWithOrderResultMap">
    SELECT u.id AS user_id, u.username, o.id AS order_id, o.order_no
    FROM user u
    LEFT JOIN `order` o ON u.id = o.user_id
    WHERE u.id = #{id}
</select>
```

### 6.3 缓存机制

MyBatis 提供两级缓存，用于优化查询性能：

#### 1. 一级缓存（SqlSession 级别，默认开启）

- 缓存范围：同一个 SqlSession 内。
- 原理：查询结果缓存到 SqlSession 中，同一 SqlSession 内重复查询同一 SQL 会直接从缓存获取。
- 失效场景：执行增删改操作、关闭 SqlSession、手动清除缓存（`sqlSession.clearCache()`）。

#### 2. 二级缓存（Mapper 级别，需手动开启）

- 缓存范围：同一个 Mapper 接口（所有 SqlSession 共享）。
- 开启方式：
    - 在 `mybatis-config.xml` 中开启全局缓存：
    - ```XML
        <settings>
            <setting name="cacheEnabled" value="true" /> <!-- 默认 true -->
        </settings>
        ```

    - 在 XML 映射文件中添加 `<cache>` 标签：
    - ```XML
        <mapper namespace="com.example.mapper.UserMapper">
            <cache /> <!-- 开启二级缓存 -->
            <!-- 其他 SQL 标签 -->
        </mapper>
        ```
- 注意：实体类需实现 `Serializable` 接口（缓存序列化存储）。

## 七、常见问题与注意事项

1. **字段名与属性名不一致**：使用 `resultMap` 映射或开启下划线自动转驼峰（`map-underscore-to-camel-case=true`）。
2. **SQL 注入风险**：优先使用 `#{}`，避免使用 `${}`；若必须使用 `${}`，需手动过滤参数。
3. **批量操作效率**：使用 `<foreach>` 标签时，注意批量插入的 SQL 长度限制（可拆分批次）。
4. **事务管理**：Spring Boot 环境下需添加 `@Transactional` 注解管理事务（MyBatis 本身事务默认手动提交，需 `sqlSession.commit()`）。
5. **XML 文件路径问题**：确保 XML 映射文件路径与 `mybatis.mapper-locations` 配置一致，资源目录下包名用 `/` 分隔（如 `mapper/com/example/`）。

## 八、总结

MyBatis 是 Java 持久层开发的主流框架，其核心优势在于灵活的 SQL 配置、简化的 JDBC 操作和良好的扩展性。通过注解式或 XML 式开发，可适配从简单查询到复杂动态 SQL 的各类场景。

在实际开发中，推荐结合 Spring Boot 整合 MyBatis，利用自动配置简化环境搭建，同时采用 Mapper 代理开发模式，配合分页插件、缓存机制等高级特性，提升开发效率和系统性能。

掌握 MyBatis 的核心语法（动态 SQL、参数传递、结果集映射）和最佳实践（如避免 SQL 注入、优化关联查询），是 Java 后端开发的必备技能。