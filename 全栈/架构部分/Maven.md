# 主题：Maven 全面知识点总结，包括核心概念、核心配置文件、生命周期与命令、分模块开发（聚合与继承）、变量与属性、多环境配置等方面的内容

# Maven 全面知识点总结

## 一、Maven 核心概念

Maven 是一款开源的 Java 项目管理工具，核心作用是 **自动化构建项目** 与 **统一管理 Jar 包依赖**，解决传统项目中手动导入 Jar 包易冲突、版本混乱的问题，同时标准化项目构建流程（编译、测试、打包、部署等）。

## 二、核心配置文件：pom.xml

`pom.xml`（Project Object Model，项目对象模型）是 Maven 项目的核心配置文件，存放项目基本信息、依赖配置、构建规则等，所有 Maven 操作均基于此文件执行。

### 2.1 项目基本信息（必填）

通过 Maven 坐标唯一标识一个项目/依赖，是 Jar 包在仓库中的“地址”：

```XML
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- POM 模型版本（固定为 4.0.0） -->
    <modelVersion>4.0.0</modelVersion>

    <!-- Maven 坐标（唯一标识项目） -->
    <groupId>com.company</groupId>    <!-- 项目隶属组织名称（例：公司域名反写） -->
    <artifactId>demo-project</artifactId>  <!-- 项目名称（Maven 仓库中 Jar 包的文件名前缀） -->
    <version>1.0.0-RELEASE</version>  <!-- 项目版本号 -->
    <packaging>jar</packaging>        <!-- 打包类型（jar/war/pom，默认 jar） -->

    <!-- 项目描述信息（可选） -->
    <name>Demo Project</name>
    <description>A sample Maven project</description>
</project>
```

#### 版本号说明：

- `RELEASE`：正式版本（稳定版，用于生产环境）
- `SNAPSHOT`：快照版本（开发版，实时更新，例如 `1.0.0-SNAPSHOT`）
- `Alpha`/`Beta`：测试版（未稳定，用于内部测试）

### 2.2 依赖配置（核心功能）

通过 `<dependencies>` 和 `<dependency>` 标签声明项目所需 Jar 包，Maven 会自动从仓库下载并导入项目，无需手动复制 Jar 包。

#### 基础依赖配置示例：

```XML
<dependencies>
    <!-- 导入 JUnit 单元测试依赖 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>  <!-- 依赖作用域（见 2.3 节） -->
    </dependency>

    <!-- 导入 Spring 核心依赖 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.20</version>
    </dependency>
</dependencies>
```

### 2.3 依赖作用域（scope）

控制依赖在 Maven 生命周期的生效范围，避免不必要的依赖传递，常用值如下：

| 作用域     | 说明                                                         | 传递性 |
| ---------- | ------------------------------------------------------------ | ------ |
| `compile`  | 默认值，编译、测试、运行阶段均生效（项目核心依赖，如 Spring-core） | 是     |
| `test`     | 仅测试阶段生效（如 JUnit），不会打包到最终产物（Jar/War）中  | 否     |
| `provided` | 编译、测试阶段生效，运行阶段由容器提供（如 Servlet-api，Tomcat 已包含） | 否     |
| `runtime`  | 测试、运行阶段生效，编译阶段不生效（如数据库驱动包 MySQL-connector） | 是     |
| `system`   | 本地 Jar 包依赖（需配合 `systemPath` 指定本地路径，不推荐使用） | 否     |

### 2.4 依赖传递与冲突解决

#### 2.4.1 依赖传递性

当 A 依赖 B，B 依赖 C 时，Maven 会自动将 C 传递给 A（无需手动导入 C），传递性受 **依赖作用域** 限制（如 `test` 作用域的依赖不传递）。

#### 2.4.2 依赖冲突场景与规则

当项目中引入多个版本的同一依赖时，Maven 按以下规则优先级解决冲突：

1. **路径就近原则**：直接依赖（项目自身声明的依赖）优先级高于传递依赖（间接引入的依赖）；
2. **声明顺序原则**：同一层级的依赖（均为直接依赖或均为传递依赖），后声明的依赖版本覆盖先声明的。

#### 2.4.3 手动解决冲突

##### （1）依赖排除（不使用传递依赖）

当传递依赖引发冲突时，通过 `<exclusions>` 排除指定传递依赖（“别人的我不想用”）：

```XML
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.20</version>
    <!-- 排除 spring-context 传递的 spring-jcl 依赖 -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jcl</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

##### （2）依赖屏蔽（禁止自身依赖被传递）

通过 `<optional>true</optional>` 声明依赖为“可选依赖”，仅当前项目可用，不传递给依赖当前项目的其他项目（“我的不给别人用”）：

```XML
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.83</version>
    <optional>true</optional>  <!-- 禁止 fastjson 传递给其他依赖当前项目的模块 -->
</dependency>
```

## 三、Maven 生命周期与命令

Maven 定义了标准化的 **构建生命周期**，每个生命周期包含一系列有序的“阶段（Phase）”，执行后续阶段时会自动执行所有前置阶段。

### 3.1 三大核心生命周期

| 生命周期            | 核心阶段（按执行顺序）                                       | 作用说明                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| **Clean**           | `pre-clean` → `clean` → `post-clean`                         | 清理上一次构建的产物（默认删除 `target` 文件夹） |
| **Default**（核心） | `validate` → `compile` → `test` → `package` → `install` → `deploy` | 项目构建的核心流程（编译、测试、打包、部署）     |
| **Site**            | `pre-site` → `site` → `post-site` → `site-deploy`            | 生成项目文档站点（如 API 文档、依赖说明）        |

### 3.2 常用命令（IDE 中可通过右侧 Maven 面板直接执行）

| 命令                | 对应阶段          | 作用                                                         |
| ------------------- | ----------------- | ------------------------------------------------------------ |
| `mvn clean`         | Clean 生命周期    | 清理 `target` 文件夹（删除编译后的 class、打包产物等）       |
| `mvn compile`       | Default → compile | 编译 `src/main/java` 下的源代码，编译结果输出到 `target/classes` |
| `mvn test`          | Default → test    | 执行 `src/test/java` 下的单元测试（依赖 `compile` 阶段）     |
| `mvn package`       | Default → package | 将编译后的文件打包（Jar/War/Pom），产物输出到 `target` 文件夹 |
| `mvn install`       | Default → install | 将打包产物安装到 **本地仓库**（默认路径：`C:\Users\用户名.m2\repository`），供其他本地项目依赖 |
| `mvn deploy`        | Default → deploy  | 将打包产物部署到 **远程仓库/私服**（需配置仓库地址，用于团队共享） |
| `mvn clean package` | 组合命令          | 先清理（clean），再打包（package）（常用）                   |

### 3.3 单元测试（JUnit）

Maven 原生支持 JUnit 单元测试，核心依赖为 `junit:junit`，测试代码放在 `src/test/java` 目录下：

1. 测试类命名规范：`XxxTest.java`（如 `UserServiceTest.java`）
2. 测试方法命名规范：`testXxx()`（如 `testAddUser()`），且方法无参数、无返回值
3. 执行 `mvn test` 时，Maven 会自动运行所有测试方法，通过断言判断用例是否通过：

```Java
import org.junit.Test;
import static org.junit.Assert.*;

public class CalculatorTest {
    @Test
    public void testAdd() {
        int result = 1 + 2;
        assertEquals(3, result); // 断言结果为 3，失败则测试不通过
    }
}
```

## 四、分模块开发与聚合/继承

当项目规模较大时，需按功能拆分模块（如 `user-module`、`order-module`），通过 **聚合** 和 **继承** 实现模块统一管理。

### 4.1 分模块开发流程

1. 拆分模块：按功能拆分多个子模块（每个子模块是独立的 Maven 项目，有自己的 `pom.xml`）；
2. 模块依赖：子模块间通过 Maven 坐标依赖（如 `order-module` 依赖 `user-module`）；
3. 安装依赖：被依赖的模块（如 `user-module`）需先执行 `mvn install` 安装到本地仓库，依赖它的模块才能正常引入。

### 4.2 聚合（多模块统一打包）

**作用**：将多个子模块聚合到一个“父工程”，执行父工程的打包命令时，自动按依赖顺序打包所有子模块（解决单个模块变动需手动打包所有依赖模块的问题）。

#### 聚合工程配置要点：

1. 父工程的 `packaging` 必须为 `pom`（仅作为聚合容器，无实际代码）；
2. 通过 `<modules>` 标签指定子模块路径（相对路径）。

#### 示例（父工程 pom.xml）：

```XML
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.company</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0.0-RELEASE</version>
    <packaging>pom</packaging> <!-- 聚合工程必须为 pom 打包类型 -->

    <!-- 聚合的子模块（相对路径） -->
    <modules>
        <module>user-module</module> <!-- 子模块 1 -->
        <module>order-module</module> <!-- 子模块 2 -->
    </modules>
</project>
```

### 4.3 继承（统一管理依赖版本）

**作用**：子模块继承父工程的配置（如依赖版本、插件配置），实现版本统一管理（无需在每个子模块重复声明版本号，修改时仅需改父工程）。

#### 继承配置要点：

1. 父工程的 `packaging` 必须为 `pom`；
2. 子模块通过 `<parent>` 标签声明继承父工程，指定父工程坐标和相对路径（`relativePath`，默认 `../pom.xml`）。

#### 示例：

##### （1）父工程 pom.xml（统一管理依赖版本）

```XML
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.company</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0.0-RELEASE</version>
    <packaging>pom</packaging>

    <!-- 依赖管理（子模块可直接继承，无需写版本号） -->
    <dependencyManagement>
        <dependencies>
            <!-- 统一声明 JUnit 版本 -->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
                <scope>test</scope>
            </dependency>
            <!-- 统一声明 Spring 版本 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>5.3.20</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

##### （2）子模块 pom.xml（继承父工程）

```XML
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.company</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0.0-RELEASE</version>
        <relativePath>../pom.xml</relativePath> <!-- 父工程 pom.xml 相对路径 -->
    </parent>

    <artifactId>user-module</artifactId> <!-- 子模块自身的 artifactId -->

    <!-- 继承父工程的依赖，无需写 version 和 scope（父工程已声明） -->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 4.4 聚合与继承的区别

| 特性     | 聚合（Aggregation）                  | 继承（Inheritance）                 |
| -------- | ------------------------------------ | ----------------------------------- |
| 核心目的 | 统一打包多个子模块                   | 统一管理依赖版本、配置              |
| 父子关系 | 父工程是“容器”，子模块是独立项目     | 子模块依赖父工程的配置              |
| 打包类型 | 父工程必须为 `pom` 类型              | 父工程必须为 `pom` 类型             |
| 依赖方向 | 父工程依赖子模块（通过 `<modules>`） | 子模块依赖父工程（通过 `<parent>`） |

**最佳实践**：一个父工程同时实现“聚合”和“继承”（既统一打包，又统一管理版本）。

## 五、Maven 变量与属性

通过变量可简化 `pom.xml` 配置（如统一版本号、路径），支持多种属性类型，使用语法为 `${属性名}`。

### 5.1 变量定义（自定义属性）

通过 `<properties>` 标签定义自定义变量，常用于统一管理版本号：

```XML
<properties>
    <spring.version>5.3.20</spring.version> <!-- Spring 版本变量 -->
    <junit.version>4.12</junit.version>     <!-- JUnit 版本变量 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> <!-- 编码格式变量 -->
</properties>

<!-- 使用变量 -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version> <!-- 引用 Spring 版本变量 -->
    </dependency>
</dependencies>
```

### 5.2 其他属性类型

| 属性类型      | 示例                                       | 说明                                                      |
| ------------- | ------------------------------------------ | --------------------------------------------------------- |
| 内置属性      | `${project.groupId}`、`${project.version}` | Maven 内置，引用项目自身坐标信息                          |
| Settings 属性 | `${settings.localRepository}`              | 引用 `settings.xml` 中的配置（如本地仓库路径）            |
| Java 系统属性 | `${java.version}`、`${user.home}`          | 引用 JVM 系统属性（可通过 `System.getProperties()` 获取） |
| 环境变量属性  | `${env.JAVA_HOME}`                         | 引用操作系统环境变量（前缀 `env.`）                       |

### 5.3 资源过滤（变量替换）

Maven 默认不解析资源文件（`src/main/resources` 下的文件）中的 `${变量名}`，需通过 `<build>` 标签开启资源过滤，实现配置文件中变量的动态替换。

#### 配置示例：

1. `pom.xml` 中定义变量并开启资源过滤：

```XML
<properties>
    <db.url>jdbc:mysql://localhost:3306/test</db.url>
    <db.username>root</db.username>
</properties>

<build>
    <resources>
        <resource>
            <!-- 资源文件目录 -->
            <directory>${project.basedir}/src/main/resources</directory>
            <filtering>true</filtering> <!-- 开启变量替换（允许解析 ${}） -->
        </resource>
    </resources>
</build>
```

1. 资源文件 `src/main/resources/jdbc.properties` 中使用变量：

```Properties
jdbc.url=${db.url}
jdbc.username=${db.username}
```

1. 构建后，变量会被替换为实际值（输出到 `target/classes/jdbc.properties`）：

```Properties
jdbc.url=jdbc:mysql://localhost:3306/test
jdbc.username=root
```

## 六、多环境配置

实际开发中需区分 **开发环境（dev）**、**测试环境（test）**、**生产环境（prod）**（如数据库地址、端口不同），可通过 Maven  Profiles 实现多环境动态切换。

### 6.1 配置步骤

1. 在 `pom.xml` 中定义多环境 Profiles：

```XML
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id> <!-- 环境唯一标识 -->
        <activation>
            <activeByDefault>true</activeByDefault> <!-- 默认激活开发环境 -->
        </activation>
        <properties>
            <db.url>jdbc:mysql://localhost:3306/dev_db</db.url>
            <server.port>8080</server.port>
        </properties>
    </profile>

    <!-- 生产环境 -->
    <profile>
        <id>prod</id>
        <properties>
            <db.url>jdbc:mysql://192.168.1.100:3306/prod_db</db.url>
            <server.port>80</server.port>
        </properties>
    </profile>
</profiles>

<!-- 开启资源过滤（让配置文件中的变量生效） -->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

1. 资源文件 `src/main/resources/application.properties` 中引用变量：

```Properties
spring.datasource.url=${db.url}
server.port=${server.port}
```

1. 切换环境的方式：
    1. 命令行：`mvn clean package -Pprod`（激活生产环境，`-P` 后跟 Profile ID）
    2. IDE 中：Maven 面板 → Profiles → 勾选对应环境 ID（如 `prod`）

## 七、Maven 仓库

Maven 仓库是存放 Jar 包的“仓库”，按访问范围分为三类：

### 7.1 仓库分类

| 仓库类型 | 说明                                                         | 路径/配置方式                                                |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 本地仓库 | 本地计算机的缓存仓库，下载的 Jar 包优先存于此（避免重复下载） | 默认路径：`C:\Users\用户名.m2\repository`，可通过 `settings.xml` 中的 `<localRepository>` 修改 |
| 中央仓库 | Maven 官方远程仓库（https://repo.maven.apache.org/maven2），包含绝大多数开源 Jar 包 | 无需手动配置，Maven 默认自动访问                             |
| 私服仓库 | 公司/团队内部的私有远程仓库（如 Nexus），存放内部 Jar 包、第三方付费 Jar 包 | 需在 `settings.xml` 中配置仓库地址、账号密码（见 7.2 节）    |

### 7.2 私服配置（Nexus 为例）

私服的核心优势：

- 内部 Jar 包共享（如公司自研框架）；
- 加速依赖下载（缓存中央仓库 Jar 包，避免全员重复访问外网）；
- 管理第三方付费 Jar 包（如 Oracle 驱动）。

#### 配置步骤：

1. 编辑 Maven 全局配置文件 `settings.xml`（路径：`Maven安装目录/conf/settings.xml` 或 `用户目录/.m2/settings.xml`）；
2. 配置私服仓库地址和认证信息：

```XML
<!-- 配置私服仓库 -->
<mirrors>
    <mirror>
        <id>company-nexus</id> <!-- 镜像 ID，需与下方 server 标签的 id 一致 -->
        <mirrorOf>central</mirrorOf> <!-- 镜像中央仓库（所有访问中央仓库的请求都转发到私服） -->
        <url>http://192.168.1.200:8081/repository/maven-public/</url> <!-- 私服仓库地址 -->
    </mirror>
</mirrors>

<!-- 配置私服账号密码（避免每次访问输入） -->
<servers>
    <server>
        <id>company-nexus</id> <!-- 与 mirror 标签的 id 一致 -->
        <username>nexus-user</username> <!-- 私服登录账号 -->
        <password>nexus-password</password> <!-- 私服登录密码 -->
    </server>
</servers>
```

## 八、常见问题与注意事项

1. **依赖下载失败**：
    1. 检查网络是否正常（中央仓库需外网）；
    2. 若使用私服，检查私服地址、账号密码是否正确；
    3. 删除本地仓库中对应依赖的文件夹（缓存损坏），重新执行 `mvn clean install`。
2. **版本冲突**：
    1. 执行 `mvn dependency:tree` 命令查看依赖树，定位冲突的依赖来源；
    2. 通过依赖排除（`<exclusions>`）或声明顺序解决冲突。
3. **资源文件中文乱码**：
    1. 在 `pom.xml` 中配置编码变量：`<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>`；
    2. 确保 IDE 项目编码与 Maven 编码一致（均为 UTF-8）。
4. **子模块无法继承父工程依赖**：
    1. 父工程依赖需放在 `<dependencyManagement>` 标签中（仅声明版本，不实际引入）；
    2. 子模块需在 `<dependencies>` 中显式声明要继承的依赖（无需写版本号）。