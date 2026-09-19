# JDK 概述

# JDK 是什么？

JDK（Java Development Kit，Java 开发工具包）是 **开发、编译、运行 Java 程序的核心工具集合**，是 Java 开发的基础环境——简单说，没有 JDK，就无法编写、编译 Java 代码，更无法运行 Java 程序。

可以把它拆解成 3 个核心部分，理解起来更清晰：

### 1. 核心：Java 运行时环境（JRE）

JRE（Java Runtime Environment）是运行 Java 程序的最小环境，包含：

- JVM（Java Virtual Machine，Java 虚拟机）：Java “一次编写、到处运行” 的核心——它会把 Java 编译后的字节码（.class 文件）翻译成当前操作系统能识别的机器码，让程序在 Windows、Linux、Mac 等不同系统上都能运行；
- 核心类库（如 `java.lang`、`java.util` 等）：Java 自带的 “工具包”，包含了字符串处理、集合、IO、网络通信等常用功能，避免开发者重复造轮子。

注意：JDK 已经内置了 JRE（JDK 9 及以上版本默认不单独安装 JRE，但运行时会自动提供相关环境），所以安装 JDK 后，无需额外装 JRE 就能运行 Java 程序。

### 2. 开发必备：编译与调试工具

JDK 提供了一系列命令行工具，支撑 Java 开发的完整流程，常用的有：

- `javac`：编译器——把你写的 `.java` 源代码文件，编译成 JVM 能识别的 `.class` 字节码文件；
- `java`：运行工具——启动 JVM，执行 `.class` 文件中的程序；
- `jar`：打包工具——把多个 `.class` 文件和资源文件（如配置文件）打包成 `.jar` 包（Java 程序的发布格式）；
- `javadoc`：文档生成工具——根据代码中的注释，自动生成标准化的 API 文档；
- 其他工具：如 `jdb`（调试工具）、`jps`（查看 Java 进程）等。

### 3. 底层支撑：Java 源代码与头文件

JDK 包含了 Java 核心类库的源代码（如 `java.lang.String` 的源码文件），以及 C/C++ 语言的头文件——方便开发者查看底层实现、调试问题，或进行跨语言开发（如通过 JNI 调用 C 代码）。

### 常见误区：JDK vs JRE vs JVM

很多人会混淆这三个概念，用一句话总结区别：

- **JVM**：负责 “运行” 字节码（核心执行引擎）；
- **JRE**：= JVM + 核心类库，是 “运行 Java 程序” 的最小依赖（普通用户只需要 JRE 就能运行别人开发好的 Java 程序，比如运行一个 Java 写的桌面应用）；
- **JDK**：= JRE + 开发/编译/调试工具，是 “开发 Java 程序” 的完整环境（程序员必须安装 JDK）。

### 补充说明

- 版本：JDK 有多个版本分支，目前主流是 JDK 8（长期支持版，LTS）、JDK 11、JDK 17（最新 LTS 版），不同版本会新增特性（如 JDK 9 引入模块系统），但核心功能兼容；
- 发行方：除了 Oracle 官方 JDK，还有 OpenJDK（开源免费，Oracle JDK 的基础）、AdoptOpenJDK（现已更名 Temurin）等免费分发版，功能基本一致，日常开发可优先选择免费版本。