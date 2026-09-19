# JDBC 详解

# JDBC 详解（Java 数据库连接）

JDBC（Java Database Connectivity）是 Java 语言中用于连接和操作关系型数据库的**标准接口**，它定义了一套统一的 API，屏蔽了不同数据库（如 MySQL、Oracle、SQL Server）的底层实现差异，使开发者能通过统一的代码操作各种数据库。

## 一、JDBC 核心原理

JDBC 的核心是「接口 + 数据库厂商实现」的设计模式：

- Java 官方提供 `java.sql` 和 `javax.sql` 包下的核心接口（如 `Connection`、`Statement` 等）；
- 数据库厂商提供对应接口的实现类（封装在 **JDBC 驱动 Jar 包** 中，如 MySQL 的 `mysql-connector-java`）；
- 开发者通过调用标准接口，间接使用厂商实现类完成数据库连接和操作。

## 二、JDBC 操作完整流程（MySQL 示例）

### 1. 前置准备：引入数据库驱动 Jar 包

JDBC 接口本身不包含数据库连接逻辑，必须引入对应数据库的驱动 Jar 包，才能实现协议层面的通信：

- **MySQL 驱动**：Maven 依赖（推荐）
    - ```XML
        <!-- MySQL 8.0+ 驱动（兼容 5.x，包名变更为 com.mysql.cj.jdbc.Driver） -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.33</version>
        </dependency>
        ```
- **手动导入**：下载 Jar 包后，添加到项目的 `lib` 目录并配置依赖（适用于非 Maven 项目）。

### 2. 注册驱动（加载驱动类）

目的是将数据库驱动类加载到 JVM 中，让 `DriverManager` 能识别并管理该驱动。

- **传统方式**（MySQL 5.x 及以下）：
    - ```Java
        // 显式加载驱动类，底层调用 DriverManager.registerDriver(new Driver())
        Class.forName("com.mysql.jdbc.Driver");
        ```
- **MySQL 8.0+ 方式**（包名变更，且可省略注册步骤）：
    - ```Java
        Class.forName("com.mysql.cj.jdbc.Driver");
        // 注：MySQL 8.0+ 驱动支持 SPI 自动注册，可省略 Class.forName()，但建议显式声明以提高可读性
        ```

### 3. 获取数据库连接（Connection）

通过 `DriverManager` 的静态方法获取 `Connection` 对象，该对象代表 Java 与数据库的**物理连接**。

```Java
// 连接 URL 格式：jdbc:数据库类型://主机地址:端口号/数据库名?参数1&参数2
String url = "jdbc:mysql://localhost:3306/test?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true";
String username = "root"; // 数据库用户名
String password = "123456"; // 数据库密码

// 获取连接（JDBC 4.0+ 可自动加载驱动，无需手动注册）
Connection conn = DriverManager.getConnection(url, username, password);
```

- **URL 核心参数说明**：
    - `useSSL=false`：禁用 SSL 连接（开发环境常用，生产环境建议开启）；
    - `serverTimezone=UTC`：指定时区（MySQL 8.0+ 必传，否则报错）；
    - `allowPublicKeyRetrieval=true`：允许获取服务器公钥（解决密码加密相关报错）。

### 4. 定义 SQL 语句

根据业务需求编写 SQL（DQL 查询、DML 增删改、DDL 建表等）：

```Java
// DQL 查询示例
String sql = "select id, username from user where age > ?"; 
// DML 插入示例
// String sql = "insert into user(username, age) values(?, ?)";
```

### 5. 创建执行 SQL 的对象

有两种核心对象用于执行 SQL：`Statement` 和 `PreparedStatement`（推荐后者）。

#### （1）Statement（基础执行对象）

直接执行静态 SQL，无预编译机制，**存在 SQL 注入风险**，仅适用于简单场景。

```Java
Statement st = conn.createStatement();
```

#### （2）PreparedStatement（预编译执行对象）

- 继承自 `Statement`，支持 SQL 预编译和参数绑定，**彻底防止 SQL 注入**；
- 用 `?` 作为占位符，通过 `setXxx(index, value)` 方法赋值（index 从 1 开始）。

```Java
// 传入带占位符的 SQL，创建预编译对象（预编译操作在数据库端执行）
PreparedStatement pst = conn.prepareStatement(sql);

// 为占位符赋值（第一个 ? 赋值为 18，第二个 ? 赋值为 "zhangsan"，根据 SQL 调整）
pst.setInt(1, 18); // 对应 age > ?
// pst.setString(2, "zhangsan"); // 若有多个占位符，依次赋值
```

### 6. 执行 SQL 语句

根据 SQL 类型选择不同的执行方法：

| 执行方法             | 适用场景                                        | 返回值说明                                                   |
| -------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| `executeQuery(sql)`  | DQL 语句（select）                              | 返回 `ResultSet` 对象（封装查询结果集）                      |
| `executeUpdate(sql)` | DML（insert/update/delete）、DDL（create/drop） | DML 返回影响的行数；DDL 返回 0                               |
| `execute(sql)`       | 通用（支持任意 SQL）                            | 返回 boolean：true 表示有结果集（如 select），false 表示无结果集 |

#### 示例 1：执行 DQL 查询（返回结果集）

```Java
// 执行查询，获取结果集
ResultSet rs = pst.executeQuery();
```

#### 示例 2：执行 DML 插入（返回影响行数）

```Java
String sql = "insert into user(username, age) values(?, ?)";
PreparedStatement pst = conn.prepareStatement(sql);
pst.setString(1, "lisi");
pst.setInt(2, 20);

int affectedRows = pst.executeUpdate(); // 返回插入成功的行数
System.out.println("插入成功：" + affectedRows + " 条数据");
```

### 7. 处理执行结果

#### （1）处理 ResultSet 结果集（DQL 查询后）

`ResultSet` 是查询结果的封装对象，本质是一个**游标（指针）**，初始指向结果集第一行之前，需通过 `rs.next()` 移动游标：

- `rs.next()`：游标向下移动一行，返回 `true` 表示当前行有数据，`false` 表示已到结果集末尾；
- 通过 `rs.getXxx(columnName)` 或 `rs.getXxx(columnIndex)` 获取字段值（推荐用列名，避免列顺序变更导致错误）。

```Java
// 遍历结果集
while (rs.next()) {
    // 根据列名获取值（推荐）
    int id = rs.getInt("id");
    String username = rs.getString("username");
    
    // 根据列索引获取值（不推荐，列顺序变更会报错）
    // int id = rs.getInt(1);
    // String username = rs.getString(2);
    
    System.out.println("id: " + id + ", username: " + username);
}
```

#### （2）处理影响行数（DML/DDL 执行后）

通过 `executeUpdate()` 的返回值判断操作结果：

```Java
if (affectedRows > 0) {
    System.out.println("操作成功！");
} else {
    System.out.println("操作失败（如无匹配数据更新）！");
}
```

### 8. 释放资源（关键步骤）

JDBC 资源（`ResultSet`、`Statement/PreparedStatement`、`Connection`）属于稀缺资源，必须手动释放，否则会导致数据库连接泄漏。

**推荐使用 try-with-resources 自动关闭资源**（Java 7+ 特性，无需手动写 finally）：

```Java
// 资源声明在 try() 中，自动关闭（关闭顺序：ResultSet → Statement → Connection）
try (Connection conn = DriverManager.getConnection(url, username, password);
     PreparedStatement pst = conn.prepareStatement(sql);
     ResultSet rs = pst.executeQuery()) {
     
    // 为占位符赋值
    pst.setInt(1, 18);
    
    // 遍历结果集
    while (rs.next()) {
        System.out.println(rs.getString("username"));
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

**传统手动关闭方式**（Java 7 之前）：

```Java
ResultSet rs = null;
PreparedStatement pst = null;
Connection conn = null;

try {
    // 1. 注册驱动、获取连接、创建对象、执行 SQL、处理结果...
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    // 关闭顺序：先开后关（ResultSet → Statement → Connection）
    if (rs != null) rs.close();
    if (pst != null) pst.close();
    if (conn != null) conn.close();
}
```

## 三、JDBC 核心 API 详解

### 1. DriverManager（驱动管理类）

- 核心作用：管理数据库驱动、获取数据库连接；
- 关键方法：
    - `registerDriver(Driver driver)`：手动注册驱动（不推荐，`Class.forName()` 底层已调用）；
    - `getConnection(String url, String user, String password)`：获取数据库连接；
    - `setLoginTimeout(int seconds)`：设置连接超时时间（默认 0，无限制）。

### 2. Connection（数据库连接对象）

- 代表 Java 与数据库的物理连接，是所有操作的基础；
- 核心功能：
    - **创建执行对象**：`createStatement()`、`prepareStatement(String sql)`；
    - **事务管理**（默认自动提交事务）：
        - ```Java
            conn.setAutoCommit(false); // 关闭自动提交，开启手动事务
            conn.commit(); // 提交事务（操作成功后执行）
            conn.rollback(); // 回滚事务（操作失败时执行，如 catch 异常中）
            conn.setSavepoint(); // 设置事务保存点（部分回滚用）
            ```
    - **关闭连接**：`close()`（必须调用，释放资源）。

### 3. Statement（SQL 执行对象）

- 执行静态 SQL 语句（无预编译）；
- 关键方法：
    - `executeQuery(String sql)`：执行 DQL，返回 `ResultSet`；
    - `executeUpdate(String sql)`：执行 DML/DDL，返回影响行数；
    - 缺点：SQL 注入风险，例如：
        - ```Java
            // 恶意输入：username = " ' or '1'='1 "
            String sql = "select * from user where username = '" + username + "'";
            // 拼接后 SQL：select * from user where username = '' or '1'='1' → 恒成立，查询所有数据
            ```

### 4. PreparedStatement（预编译执行对象）

- 解决 SQL 注入的核心：预编译时将 SQL 结构与参数分离，参数自动转义（如单引号 `'` 转译为 `'`）；
- 关键方法（除继承 Statement 的方法外）：
    - `setXxx(int parameterIndex, Xxx value)`：为占位符赋值（`Xxx` 为数据类型，如 `setInt`、`setString`、`setDate`）；
    - `getGeneratedKeys()`：获取插入操作生成的自增主键（如 `id` 自增时）。

### 5. ResultSet（结果集对象）

- 封装 DQL 查询结果，通过游标遍历；
- 关键方法：
    - 游标操作：`next()`（向下移动）、`previous()`（向上移动）、`beforeFirst()`（回到初始位置）；
    - 取值操作：`getInt(String columnName)`、`getString(int columnIndex)`、`getDate(String columnLabel)` 等；
    - 关闭资源：`close()`（必须关闭，否则占用内存）。

## 四、JDBC 常见问题与优化

### 1. SQL 注入问题

- 原因：使用 `Statement` 拼接 SQL 时，用户输入的恶意字符（如 `'`、`or`）被解析为 SQL 逻辑；
- 解决方案：强制使用 `PreparedStatement`，所有参数通过占位符 `?` 传入。

### 2. 数据库连接泄漏

- 原因：`Connection`、`Statement`、`ResultSet` 未正确关闭（如异常时未执行 `close()`）；
- 解决方案：
    - 使用 try-with-resources 自动关闭资源；
    - 手动关闭时，将 `close()` 放在 `finally` 块中。

### 3. 性能优化

- **连接池**：使用 Druid、HikariCP 等连接池管理 `Connection`，避免频繁创建/关闭连接（连接池复用连接，提升性能）；
- **预编译缓存**：`PreparedStatement` 的预编译结果会被数据库缓存，重复执行相同 SQL 时无需重新编译；
- **结果集映射**：使用 ORM 框架（如 MyBatis、JPA）替代手动遍历 `ResultSet`，简化代码。

### 4. 版本兼容问题

- MySQL 5.x 驱动包名：`com.mysql.jdbc.Driver`，URL 无需指定时区；
- MySQL 8.0+ 驱动包名：`com.mysql.cj.jdbc.Driver`，URL 必须指定 `serverTimezone` 参数。

## 五、总结

JDBC 是 Java 操作关系型数据库的基础，核心流程为「加载驱动 → 获取连接 → 创建执行对象 → 执行 SQL → 处理结果 → 释放资源」。实际开发中，推荐使用 `PreparedStatement` 防止 SQL 注入，结合连接池提升性能，必要时使用 ORM 框架简化开发。掌握 JDBC 原理和 API，是理解更高层数据访问框架（如 MyBatis）的关键。