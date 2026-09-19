# MySQL全面学习文档

# 一、MySQL服务基础操作

## 1.1 服务启动与停止

- 启动命令：`net start mysql80`（mysql80为服务名，需根据实际配置调整）
- 停止命令：`net stop mysql80`

## 1.2 客户端连接

基础连接命令：`mysql [-h 127.0.0.1] [-P 3306] -u root -p`

- `-h`：指定数据库主机地址，默认127.0.0.1（本地）
- `-P`：指定端口号，默认3306（注意大写）
- `-u`：指定用户名，如root
- `-p`：提示输入密码，避免直接在命令行明文写密码

## 1.3 数据类型参考

详细数据类型讲解视频：[MySQL数据类型详解](https://www.bilibili.com/video/BV1Kr4y1i7ru?t=594.0&p=8)

# 二、MySQL核心语言（DDL/DML/DQL/DCL）

## 2.1 数据定义语言（DDL）

用于定义数据库、表结构，核心操作包括创建、查询、修改、删除

### 2.1.1 数据库操作

- 查询所有数据库：`show databases;`
- 查询当前使用的数据库：`select database();`
- 创建数据库：`create database [if not exists] db_name [charset utf8mb4];`（if not exists避免重复创建错误，utf8mb4支持emoji）
- 删除数据库：`drop database db_name;`（谨慎操作，数据不可恢复）
- 使用数据库：`use db_name;`（操作表前必须指定数据库）

### 2.1.2 表结构操作

- 查询当前数据库所有表：`show tables;`
- 查询表结构：`desc table_name;`
- 查询指定表的建表语句：`show create table table_name;`（查看完整表定义，包括引擎、字符集）
- 创建表：`create table table_name(字段1 字段1类型 约束, 字段2 字段2类型 约束, ...);`
- 修改表-添加字段：`alter table table_name add 字段名 字段类型 约束;`
- 修改表-修改数据类型：`alter table table_name modify 字段名 字段类型;`
- 修改表-修改字段名：`alter table table_name change 旧字段名 新字段名 字段类型;`
- 修改表-删除字段：`alter table table_name drop 字段名;`
- 修改表-修改表名：`alter table table_name rename to 新表名;`
- 删除表：`drop table table_name;`

## 2.2 数据操作语言（DML）

用于对表中数据进行增删改操作

### 2.2.1 插入数据

- 指定字段添加：`insert into table_name(字段1, 字段2, ...) values(值1, 值2, ...);`
- 全部字段添加：`insert into table_name values(值1, 值2, ...);`（需按字段顺序传入所有值）
- 批量添加：`insert into table_name values(值1,值2...),(值1,值2...),(值1,值2...);`（效率高于多次单条插入）
- 注意：字符串和日期类型值需用单引号包裹，如`'2024-05-20'`、`'张三'`

### 2.2.2 修改数据

```
update table_name set 字段1=值1, 字段2=值2... where 条件;
```

必须加where条件，否则会修改表中所有数据！

### 2.2.3 删除数据

```
delete from table_name where 删除条件;
```

where条件不可省略，否则删除全表数据；delete删除的记录可通过事务回滚恢复，truncate（DDL）不可恢复

## 2.3 数据查询语言（DQL）

核心语法：`select 字段列表 from 表名 where 查询条件 group by 分组字段 having 分组条件 order by 排序字段 limit 分页参数`

执行顺序：1.where → 2.group by → 3.having → 4.select → 5.order by → 6.limit

### 2.3.1 基本查询

- 查询多个字段：`select 字段1, 字段2... from table_name;`
- 查询所有字段：`select * from table_name;`（开发中尽量避免用*，明确字段更高效）
- 设置别名：`select 字段1 as 别名1, 字段2 别名2... from table_name;`（as可省略，别名含空格需用双引号）
- 去除重复记录：`select distinct 字段1, 字段2... from table_name;`（多字段时需同时重复才会去重）

### 2.3.2 条件查询（where）

基础语法：`select 字段列表 from table_name where 条件1 and/or 条件2...`

常用条件符号：

| 符号/关键字       | 含义                   | 示例                  |
| :---------------- | :--------------------- | :-------------------- |
| `<>` 或 `!=`      | 不等于                 | age <> 18             |
| `is NULL`         | 为空（不能用=NULL）    | name is NULL          |
| `is not NULL`     | 不为空                 | name is not NULL      |
| `between A and B` | 在A到B范围内（含边界） | age between 18 and 30 |
| `in(A,B,C)`       | 在指定集合中           | gender in('男','女')  |
| `LIKE`            | 模糊匹配               | name LIKE '%张%'      |

#### 模糊匹配详解

MySQL支持通配符、内置函数、正则表达式三种模糊匹配方式：

1. **通配符匹配**：%（任意数量字符，包括0个）、_（单个字符）        匹配含"网"字：`SELECT * FROM app_info WHERE appName LIKE '%网%';`
2. 匹配以"网"结尾：`SELECT * FROM app_info WHERE appName LIKE '%网';`
3. 匹配以"网"开头：`SELECT * FROM app_info WHERE appName LIKE '网%';`
4. 匹配3字符且以"网"结尾：`SELECT * FROM app_info WHERE appName LIKE '__网';`
5. **内置函数匹配**：返回子串位置，>0表示存在        LOCATE：`SELECT * FROM app_info WHERE LOCATE('网', appName) > 0;`
6. POSITION：`SELECT * FROM app_info WHERE POSITION('网' IN appName) > 0;`
7. INSTR：`SELECT * FROM app_info WHERE INSTR(appName, '网') > 0;`
8. **正则表达式匹配（REGEXP）**：支持复杂模式        包含指定字符：`SELECT * FROM app_info WHERE appName REGEXP '网';`
9. 多条件匹配：`SELECT * FROM app_info WHERE appName REGEXP '中国|互联网|大学';`
10. 数字开头：`SELECT * FROM app_info WHERE appName REGEXP '^[0-9]';`

模糊匹配注意：%开头会导致索引失效，影响性能；数据含通配符需用ESCAPE转义；默认不区分大小写

### 2.3.3 聚合查询

使用聚合函数对数据进行统计，常用函数：count（计数）、sum（求和）、avg（平均值）、max（最大值）、min（最小值）

- 统计总记录数：`select count(*) from user;`（count(*)包含NULL，count(字段)不包含NULL）
- 按条件统计：`select count(age) from user where age > 18;`（统计年龄大于18的记录数）

### 2.3.4 分组查询（group by）

将数据按指定字段分组，结合聚合函数使用

- 基础用法：`select 分组字段, count(*) from user group by 分组字段;`
- 示例：按年龄分组统计人数：`select age, count(*) from user group by age;`
- 分组筛选（having）：`select age, count(*) from user group by age having count(*) > 5;`（筛选分组后人数大于5的组）
- 区别：where筛选原始数据（分组前），having筛选分组结果（分组后）

### 2.3.5 排序查询（order by）

按指定字段排序，默认升序（asc），可指定降序（desc）

- 单字段升序：`select * from user order by age asc;`（asc可省略）
- 单字段降序：`select * from user order by age desc;`
- 多字段排序：`select * from user order by age desc, id asc;`（先按年龄降序，年龄相同按id升序）

### 2.3.6 分页查询（limit）

用于实现分页功能，语法：`limit 起始索引, 查询记录数`

- 起始索引计算：`起始索引 = (查询页码 - 1) * 每页显示记录数`
- 示例：第2页，每页显示10条：`select * from user limit 10, 10;`（索引从0开始，第1页为limit 0,10）

## 2.4 权限控制语句（DCL）

用于管理数据库用户和权限分配

### 2.4.1 用户管理

- 查询用户：`use mysql; select * from user;`（用户信息存储在mysql库的user表中）
- 创建用户：`create user '用户名'@'主机名' identified by '密码';`（主机名%表示任意主机可连接）
- 修改用户密码：`alter user '用户名'@'主机名' identified with mysql_native_password by '新密码';`
- 删除用户：`drop user '用户名'@'主机名';`

### 2.4.2 权限控制

常用权限：all privileges（所有权限）、select（查询）、insert（插入）、update（修改）、delete（删除）、create（创建）、alter（修改结构）、drop（删除）

- 查询权限：`show grants for '用户名'@'主机名';`
- 授予权限：`grant 权限 on 数据库.表 to '用户名'@'主机名';`（*.*表示所有数据库所有表）
- 示例：授予查询test库所有表权限：`grant select on test.* to 'testuser'@'%';`
- 撤销权限：`revoke 权限 on 数据库.表 from '用户名'@'主机名';`

# 三、MySQL函数

## 3.1 字符串函数

| 函数名                   | 功能                     | 示例                                         |
| :----------------------- | :----------------------- | :------------------------------------------- |
| concat(str1,str2,...)    | 字符串拼接               | concat('张','三') → '张三'                   |
| lower(str)               | 转为小写                 | lower('MySQL') → 'mysql'                     |
| upper(str)               | 转为大写                 | upper('mysql') → 'MYSQL'                     |
| lpad(str,n,pad)          | 左填充至n位，不足用pad补 | lpad('123',5,'0') → '00123'                  |
| rpad(str,n,pad)          | 右填充至n位，不足用pad补 | rpad('123',5,'0') → '12300'                  |
| trim(str)                | 去除首尾空格             | trim('  abc  ') → 'abc'                      |
| substring(str,start,len) | 截取字符串，start从1开始 | substring('MySQL',2,3) → 'SQL'               |
| length(str)              | 获取字符串字节长度       | length('张三') → 6（utf8mb4下一个中文3字节） |

## 3.2 数值函数

| 函数名       | 功能                  | 示例                   |
| :----------- | :-------------------- | :--------------------- |
| ceil(num)    | 向上取整              | ceil(1.2) → 2          |
| floor(num)   | 向下取整              | floor(1.8) → 1         |
| mod(a,b)     | 取余数（a%b）         | mod(5,2) → 1           |
| abs(num)     | 绝对值                | abs(-3) → 3            |
| rand()       | 返回0~1随机数         | rand() → 0.654321      |
| round(num,n) | 四舍五入，保留n位小数 | round(3.1415,2) → 3.14 |

## 3.3 日期函数

| 函数名                         | 功能                                | 示例                                               |
| :----------------------------- | :---------------------------------- | :------------------------------------------------- |
| datediff(date1,date2)          | date1 - date2的天数                 | datediff('2024-05-20','2024-05-10') → 10           |
| now()                          | 当前日期时间（YYYY-MM-DD HH:MM:SS） | now() → 2024-05-20 15:30:00                        |
| date_add(date,interval n type) | 日期加n个指定单位                   | date_add(now(),interval 7 day) → 7天后             |
| date_sub(date,interval n type) | 日期减n个指定单位                   | date_sub(now(),interval 1 month) → 1个月前         |
| date_format(date,format)       | 日期格式化                          | date_format(now(),'%Y年%m月%d日') → 2024年05月20日 |
| curdate()                      | 当前日期（YYYY-MM-DD）              | curdate() → 2024-05-20                             |
| curtime()                      | 当前时间（HH:MM:SS）                | curtime() → 15:30:00                               |
| year(date)                     | 提取年份                            | year('2024-05-20') → 2024                          |
| month(date)                    | 提取月份                            | month('2024-05-20') → 5                            |

## 3.4 流程函数

| 函数名            | 功能                          | 示例                                                         |
| :---------------- | :---------------------------- | :----------------------------------------------------------- |
| if(condition,t,f) | condition为真返回t，否则返回f | if(age>18,'成年','未成年')                                   |
| ifnull(v1,v2)     | v1为NULL返回v2，否则返回v1    | ifnull(salary,0) → 薪资为NULL时返回0                         |
| case多条件判断1   | 按条件匹配返回对应值          | case when score>=90 then 'A' when score>=80 then 'B' else 'C' end |
| case多条件判断2   | 按值匹配返回对应值            | case gender when '1' then '男' when '2' then '女' else '未知' end |

# 四、约束与外键

## 4.1 六大约束

| 约束类型 | 关键字      | 功能                                              |
| :------- | :---------- | :------------------------------------------------ |
| 非空约束 | not null    | 字段值不可为NULL                                  |
| 唯一约束 | unique      | 字段值唯一，可存多个NULL                          |
| 主键约束 | primary key | 非空+唯一，表中唯一标识（自动增长auto_increment） |
| 默认约束 | default     | 字段未赋值时使用默认值                            |
| 检查约束 | check       | 限制字段值范围（如check(age>0)）                  |
| 外键约束 | foreign key | 关联两张表，保证数据一致性                        |

## 4.2 外键详解

外键用于关联主表（被参照表）和从表（参照表），从表外键值需在主表主键值范围内或为NULL

### 4.2.1 外键操作语法

- 创建表时添加外键：`create table 从表名(字段1 类型, 外键字段 类型, constraint 外键名 foreign key (外键字段) references 主表名(主表主键));`
- 已有表添加外键：`alter table 从表名 add constraint 外键名 foreign key (外键字段) references 主表名(主表主键);`
- 删除外键：`alter table 从表名 drop foreign key 外键名;`

### 4.2.2 外键行为（主表数据变更时从表处理方式）

| 行为类型             | 功能                                                       |
| :------------------- | :--------------------------------------------------------- |
| no action / restrict | 主表数据有对应外键时，禁止删除/修改（默认行为）            |
| cascade              | 主表数据删除/修改时，从表对应外键数据同步删除/修改         |
| set null             | 主表数据删除时，从表对应外键数据设为NULL（需外键允许NULL） |
| set default          | 主表数据变更时，从表外键设为默认值（需提前设置默认值）     |

示例：添加级联删除外键：`alter table order add constraint fk_order_user foreign key (user_id) references user(id) on delete cascade;`

# 五、多表查询

## 5.1 基础多表查询

核心：通过外键关联多张表，语法：`select * from 表1, 表2 where 表1.外键 = 表2.主键;`

示例：查询订单及对应用户信息：`select o.id, o.order_no, u.name from order o, user u where o.user_id = u.id;`

## 5.2 连接查询

### 5.2.1 内连接（inner join）

查询两张表的交集部分（匹配上的记录）

- 隐式内连接：`select * from 表1, 表2 where 关联条件;`（即基础多表查询）
- 显式内连接：`select * from 表1 inner join 表2 on 关联条件;`（推荐，逻辑更清晰）

### 5.2.2 外连接

- 左外连接（left join）：查询左表所有记录 + 右表匹配记录，右表无匹配则为NULL        语法：`select * from 左表 left join 右表 on 关联条件;`
- 右外连接（right join）：查询右表所有记录 + 左表匹配记录，左表无匹配则为NULL        语法：`select * from 左表 right join 右表 on 关联条件;`

### 5.2.3 自连接

同一张表当作两张表查询，需给表取别名

示例：查询员工及直属领导信息：`select e.name 员工, l.name 领导 from emp e left join emp l on e.leader_id = l.id;`

### 5.2.4 联合查询（union）

合并多张表的查询结果，要求字段数和类型一致，会自动去重（union all不去重）

语法：`select 字段列表 from 表1 union select 字段列表 from 表2;`

### 5.2.5 子查询

一个查询语句作为另一个查询的条件或字段，按返回结果分为四类：

- 标量子查询：返回单个值（用于=、>、<等条件）        示例：查询薪资最高的员工：`select * from emp where salary = (select max(salary) from emp);`
- 列子查询：返回单列多行（用于in、not in等条件）        示例：查询部门编号为1、2的员工：`select * from emp where dept_id in (select id from dept where id in (1,2));`
- 行子查询：返回单行多列（用于多字段匹配）        示例：查询与张三薪资和部门相同的员工：`select * from emp where (salary, dept_id) = (select salary, dept_id from emp where name='张三');`
- 表子查询：返回多行多列（用于from子句，当作临时表）        示例：查询各部门薪资前2的员工：`select * from (select *, rank() over(partition by dept_id order by salary desc) rk from emp) t where rk <=2;`

# 六、事务

## 6.1 事务的ACID特性

- **原子性（Atomicity）**：事务中所有操作要么全成功，要么全失败（回滚）
- **一致性（Consistency）**：事务执行前后数据库数据完整性不变（如转账前后总金额不变）
- **隔离性（Isolation）**：多个事务并发执行时，相互不干扰
- **持久性（Durability）**：事务提交后，数据永久保存到磁盘

## 6.2 事务操作语法

- 查看事务提交方式：`select @@autocommit;`（1：自动提交，0：手动提交）
- 设置手动提交：`set @@autocommit = 0;`
- 开启事务：`begin;` 或 `start transaction;`
- 提交事务：`commit;`（事务成功时执行，数据写入磁盘）
- 回滚事务：`rollback;`（事务失败时执行，恢复到事务前状态）

## 6.3 并发事务问题与隔离级别

### 6.3.1 并发事务三大问题

- 脏读：一个事务读取到另一个事务未提交的修改数据
- 不可重复读：同一事务内，多次读取同一数据结果不一致（另一事务提交了修改）
- 幻读：同一事务内，多次查询同一条件，结果集行数不一致（另一事务提交了插入/删除）

### 6.3.2 事务隔离级别（从低到高）

| 隔离级别 | 关键字           | 解决问题             | MySQL默认 |
| :------- | :--------------- | :------------------- | :-------- |
| 读未提交 | read uncommitted | 无                   | 否        |
| 读已提交 | read committed   | 脏读                 | 否        |
| 可重复读 | repeatable read  | 脏读、不可重复读     | 是        |
| 串行化   | serializable     | 所有问题（并发最低） | 否        |

操作语法：

- 查看隔离级别：`select @@transaction_isolation;`
- 设置隔离级别：`set [session | global] transaction isolation level 隔离级别;`

# 七、存储引擎

## 7.1 常用存储引擎

MySQL支持多种存储引擎，通过`engine = 引擎名`指定，默认InnoDB

| 存储引擎 | 特点                                                    | 适用场景                         |
| :------- | :------------------------------------------------------ | :------------------------------- |
| InnoDB   | 支持事务、行级锁、外键，数据文件.ibd，崩溃恢复能力强    | 绝大多数业务场景（电商、金融等） |
| MyISAM   | 不支持事务和外键，表级锁，查询速度快，数据文件.myd+.myi | 只读或读写少的场景（日志、报表） |
| Memory   | 数据存内存，速度极快，重启数据丢失                      | 临时数据存储（缓存）             |

InnoDB重要参数：`innodb_file_per_table=1`（1：每张表一个ibd文件；0：所有表共享表空间）

# 八、索引

## 8.1 索引类型

| 索引类型 | 关键字      | 功能                                    |
| :------- | :---------- | :-------------------------------------- |
| 主键索引 | primary key | 基于主键创建，唯一且非空                |
| 唯一索引 | unique      | 字段值唯一，可存NULL                    |
| 常规索引 | index       | 加速普通查询，无唯一性约束              |
| 全文索引 | fulltext    | 用于长文本关键词检索（InnoDB 5.6+支持） |

## 8.2 InnoDB索引结构

- **聚集索引**：数据与索引存储在一起，叶子节点存完整行数据，表中唯一，默认是主键索引；无主键则用第一个唯一索引；无唯一索引则自动生成rowid作为聚集索引。
- **二级索引**：数据与索引分离，叶子节点存对应主键值，需通过主键值回表查询完整数据（回表：先查二级索引得主键，再查聚集索引得数据）。

## 8.3 索引操作语法

- 创建索引：`create [unique|fulltext] index 索引名 on 表名(字段1, 字段2...);`（多字段为联合索引）
- 查看索引：`show index from 表名;`