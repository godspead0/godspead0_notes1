# Tomcat 基础与部署

很抱歉，你提供的B站视频URL（https://www.bilibili.com/video/BV1Qf4y1T7Hx?t=13.8&p=92）解析失败，无法获取视频内的详细内容，仅能基于你补充的Tomcat相关文本信息进行整理和补充，并转换为Markdown格式。

### Tomcat 核心基础信息

Tomcat 是一款轻量级 Java Web 服务器，同时也是 Servlet 容器，主要用于简化 Java Web 项目的开发流程，帮助开发者将本地开发的项目快速部署到服务器中运行。

### Tomcat 核心目录结构及功能

- **bin 目录**：存放 Tomcat 启动、关闭服务器的脚本文件（如 Windows 系统的 startup.bat、shutdown.bat，Linux 系统的 [startup.sh](startup.sh)、[shutdown.sh](shutdown.sh)）。
- **conf 目录**：存储 Tomcat 核心配置文件，包含服务器端口、连接数、虚拟主机等关键配置，核心文件包括 server.xml（服务器核心配置）、web.xml（Web 应用全局配置）、tomcat-users.xml（用户权限配置）等。
- **lib 目录**：存放 Tomcat 运行所需的依赖 Jar 包，涵盖 Servlet API、JSP API 等基础类库，支撑 Web 应用的正常运行。
- **webapps 目录**：Web 项目的部署目录，需部署的项目文件（或打包后的 WAR 包）放置于此。其中，WAR 包会被 Tomcat 自动解压为项目目录，无需手动解压操作。
- **logs 目录**：记录 Tomcat 运行过程中的日志信息，包括启动日志、错误日志、访问日志等，用于问题排查和运行状态监控。

### Tomcat 启动流程

1. 执行 bin 目录下的启动脚本，启动 Tomcat 服务器进程。
2. 服务器优先加载 conf 目录下的 web.xml 文件，初始化 Web 应用的全局配置规则。
3. 根据配置规则，扫描并加载 webapps 目录下的所有 Web 项目（包括解压后的项目目录和未解压的 WAR 包）。
4. 完成项目加载后，Tomcat 监听配置的端口（默认 8080 端口），等待客户端的访问请求。

### Tomcat 项目部署方式

1. 直接部署：将 Web 项目的文件夹直接复制到 webapps 目录下，重启 Tomcat 即可完成部署。
2. WAR 包部署：将 Java Web 项目打包为 WAR 格式的压缩包，放置到 webapps 目录下。Tomcat 会自动识别 WAR 包并完成解压，重启服务器后项目即可运行。
3. 配置部署：通过修改 conf/server.xml 文件，手动配置项目的部署路径（非默认方式，需谨慎操作）。

### Tomcat 常用配置参数对照表

| 配置文件              | 核心参数                         | 默认值                                                       | 参数说明                                                     |
| :-------------------- | :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| conf/server.xml       | port（Connector端口）            | 8080                                                         | Tomcat 接收 HTTP 请求的端口，若该端口被占用，需修改为未占用端口（如 8081、8090 等） |
| maxThreads            | 200                              | Tomcat 处理请求的最大线程数，决定服务器同时能响应的请求数量，高并发场景可适当增大（如 500、1000） |                                                              |
| minSpareThreads       | 10                               | 核心线程数，即服务器始终保持的活跃线程数，确保有足够线程应对突发请求 |                                                              |
| redirectPort          | 8443                             | 当请求使用 HTTP 协议时，若需强制跳转至 HTTPS 协议，会重定向到该端口 |                                                              |
| conf/server.xml       | connectionTimeout                | 20000（毫秒）                                                | 客户端与服务器建立连接后的超时时间，超过该时间无请求则断开连接，避免资源占用 |
| URIEncoding           | 无（默认随系统）                 | 统一资源标识符（URI）的编码格式，建议设置为 UTF-8，避免中文参数乱码问题 |                                                              |
| conf/web.xml          | session-timeout                  | 30（分钟）                                                   | Web 应用中 Session 的过期时间，超过该时间无操作，Session 会被销毁，可根据业务需求调整（如 15、60 分钟） |
| welcome-file-list     | index.html、index.htm、index.jsp | Web 项目的默认首页，访问项目根路径时，会按顺序查找该列表中的文件，找到则直接返回 |                                                              |
| conf/tomcat-users.xml | role（角色）、user（用户）       | 无默认有效用户                                               | 配置 Tomcat 管理页面（如 Manager App）的访问权限，需手动添加角色（如 manager-gui）和关联用户，设置用户名和密码 |

此外，若你需要对某个配置参数的修改步骤进行详细说明，或者补充其他配置项的信息，都可以告诉我。