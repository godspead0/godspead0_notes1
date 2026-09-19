# MyBatis-Plus 核心使用

# MyBatis-Plus 核心使用笔记

MyBatis-Plus（简称 MP）是 MyBatis 的增强工具，**无侵入式**扩展 MyBatis 功能，旨在简化开发流程、提高开发效率。核心支持 CRUD 接口封装、Lambda 条件查询、查询投影、字段/表名映射、乐观锁等实用功能，无需手动编写大量 XML 或 SQL 语句。

## 一、核心特性与基础依赖

### 1. 核心优势

- 无侵入：完全基于 MyBatis 扩展，不改变原有 MyBatis 用法
- 内置 CRUD：BaseMapper 接口封装常用操作，直接继承即可使用
- 支持 Lambda：避免硬编码字段名，减少拼写错误
- 功能丰富：条件查询、查询投影、乐观锁、字段映射等一站式支持

### 2. 基础准备

- 实体类需与数据库表对应（支持注解映射）
- Mapper 接口继承 `BaseMapper<T>`（T 为实体类）
- 配置 MP 核心依赖（Maven/Gradle）及拦截器（如乐观锁）

## 二、内置 CRUD 操作（BaseMapper 核心方法）

Mapper 接口继承 `BaseMapper<T>` 后，直接调用以下内置方法，无需手动编写 SQL：

| 方法名                                                       | 功能描述                               | 示例调用                                                     |
| ------------------------------------------------------------ | -------------------------------------- | ------------------------------------------------------------ |
| `int insert(T entity)`                                       | 新增一条记录                           | `userMapper.insert(new User("张三", 20))`                    |
| `int updateById(T entity)`                                   | 根据 ID 更新记录（非 null 字段才更新） | `userMapper.updateById(new User(1L, "李四", 22))`            |
| `T selectById(Serializable id)`                              | 根据 ID 查询单条记录                   | `User user = userMapper.selectById(1L)`                      |
| `int deleteById(Serializable id)`                            | 根据 ID 删除记录                       | `userMapper.deleteById(1L)`                                  |
| `List<T> selectList(Wrapper<T> queryWrapper)`                | 条件查询列表                           | 见「条件查询」章节                                           |
| `IPage<T> selectPage(IPage<T> page, Wrapper<T> queryWrapper)` | 分页查询                               | `IPage<User> page = userMapper.selectPage(new Page<>(1,10), qw)` |

## 三、条件查询（Wrapper 用法）

MP 提供 `QueryWrapper`（普通格式）和 `LambdaQueryWrapper`（Lambda 格式），用于构造查询条件，避免硬编码字段名。

### 1. 三种使用格式

#### （1）普通 QueryWrapper（硬编码字段名）

```Java
// 构造条件：id = 1 且 username 包含 "张"
QueryWrapper<User> qw = new QueryWrapper<>();
qw.eq("id", 1L) // 等于
   .like("username", "张"); // 模糊查询

List<User> userList = userMapper.selectList(qw);
```

#### （2）QueryWrapper + Lambda（避免硬编码）

```Java
QueryWrapper<User> qw = new QueryWrapper<>();
// 用 lambda() 方法切换为 Lambda 模式，直接引用实体类方法
qw.lambda()
   .eq(User::getId, 1L)
   .like(User::getUsername, "张");

List<User> userList = userMapper.selectList(qw);
```

#### （3）LambdaQueryWrapper（推荐，直接 Lambda 格式）

```Java
// 直接创建 Lambda 格式 Wrapper，无需调用 lambda() 方法
LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();
lqw.eq(User::getId, 1L)
   .like(User::getUsername, "张");

List<User> userList = userMapper.selectList(lqw);
```

### 2. 空值处理（避免条件失效）

使用 `lt`/`gt`/`eq` 等方法的**三参数重载**，第一个参数为「条件是否成立」，实现空值过滤：

```Java
Integer minAge = 18;
LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();
// 当 minAge 不为 null 时，才添加 "age > minAge" 条件
lqw.gt(minAge != null, User::getAge, minAge);
```

### 3. 常用条件方法

| 方法名       | 功能描述          | 示例（Lambda 格式）                      |
| ------------ | ----------------- | ---------------------------------------- |
| `eq`         | 等于              | `lqw.eq(User::getId, 1L)`                |
| `ne`         | 不等于            | `lqw.ne(User::getUsername, "admin")`     |
| `like`       | 模糊查询（%值%）  | `lqw.like(User::getUsername, "张")`      |
| `likeLeft`   | 左模糊（%值）     | `lqw.likeLeft(User::getUsername, "张")`  |
| `likeRight`  | 右模糊（值%）     | `lqw.likeRight(User::getUsername, "张")` |
| `gt`/`lt`    | 大于/小于         | `lqw.gt(User::getAge, 18)`               |
| `ge`/`le`    | 大于等于/小于等于 | `lqw.ge(User::getAge, 18)`               |
| `in`         | 包含              | `lqw.in(User::getId, 1L, 2L, 3L)`        |
| `isNull`     | 字段为空          | `lqw.isNull(User::getEmail)`             |
| `orderByAsc` | 升序排序          | `lqw.orderByAsc(User::getAge)`           |

## 四、查询投影（字段筛选）

控制查询返回的字段（避免查询所有字段），支持指定字段、别名。

### 1. Lambda 格式（推荐）

使用 `select` 方法传入实体类方法引用，指定需要查询的字段：

```Java
LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();
// 只查询 id 和 username 字段
lqw.select(User::getId, User::getUsername);
List<User> userList = userMapper.selectList(lqw);
```

### 2. 普通格式（硬编码字段名）

直接传入字段名字符串，支持用 `as` 指定别名：

```Java
QueryWrapper<User> qw = new QueryWrapper<>();
// 查询 id（别名 uid）和 username（别名 uname）
qw.select("id as uid", "username as uname");
List<Map<String, Object>> userMapList = userMapper.selectMaps(qw);
```

## 五、字段/表名映射（解决不一致问题）

当实体类与数据库表、字段名不一致时，使用 MP 注解进行映射。

### 1. 表名映射（类名 ≠ 表名）

用 `@TableName` 注解指定数据库表名：

```Java
// 实体类名 User → 数据库表名 t_user
@TableName("t_user")
public class User {
    // ... 字段
}
```

### 2. 字段名映射（变量名 ≠ 字段名）

用 `@TableField(value = "数据库字段名")` 映射字段，支持别名：

```Java
public class User {
    // 实体变量名 userId → 数据库字段名 user_id
    @TableField(value = "user_id")
    private Long userId;
    
    // 字段别名（查询时用别名，数据库实际字段名仍为 username）
    @TableField(value = "username", column = "uname")
    private String username;
}
```

### 3. 忽略数据库不存在的字段

实体类中存在数据库表没有的字段时，用 `@TableField(exist = false)` 忽略：

```Java
public class User {
    private Long id;
    private String username;
    
    // 该字段仅用于内存计算，数据库中不存在
    @TableField(exist = false)
    private String tempValue;
}
```

### 4. 忽略查询时的保密字段

敏感字段（如密码）不需要查询返回时，用 `@TableField(select = false)` 忽略：

```Java
public class User {
    private Long id;
    private String username;
    
    // 查询时默认不返回 password 字段
    @TableField(select = false)
    private String password;
}
```

## 六、乐观锁（解决多线程并发问题）

乐观锁基于「版本号」机制，避免并发更新时的数据覆盖，MP 提供注解 + 拦截器一键实现。

### 1. 实现步骤

#### （1）数据库表添加版本字段

在表中新增 `version` 字段（int 类型，默认值 1）：

```SQL
ALTER TABLE t_user ADD COLUMN version INT DEFAULT 1 COMMENT '乐观锁版本号';
```

#### （2）实体类添加版本字段并加注解

用 `@Version` 注解标记版本字段：

```Java
public class User {
    private Long id;
    private String username;
    
    // 乐观锁版本字段
    @Version
    private Integer version;
}
```

#### （3）配置乐观锁拦截器

在 MP 配置类中注册乐观锁拦截器（Spring Boot 示例）：

```Java
@Configuration
public class MyBatisPlusConfig {
    // 注册乐观锁拦截器
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

### 2. 工作原理

1. 线程 1 查询数据时，获取当前版本号 `version = 1`；
2. 线程 1 执行更新操作，MP 自动拼接 SQL：`UPDATE t_user SET ..., version = 2 WHERE id = ? AND version = 1`；
3. 若线程 2 同时更新该数据，其查询的版本号仍为 1，但此时数据库中 version 已被线程 1 改为 2，线程 2 的更新 SQL 因 `version = 1` 不成立而执行失败；
4. 线程 2 需重新查询最新数据（获取 version = 2），再次尝试更新。

### 3. 调用方式

直接调用 `updateById` 方法即可，MP 自动处理版本号：

```Java
// 1. 查询最新数据（获取当前 version = 1）
User user = userMapper.selectById(1L);
// 2. 修改字段
user.setUsername("李四");
// 3. 自动触发乐观锁机制，版本号自增
userMapper.updateById(user);
```