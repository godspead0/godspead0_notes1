# 搭建 Git 私服的 4 种主流方案

搭建 Git 私服的核心是 **在自己的服务器（本地/云服务器）上部署 Git 服务**，支持团队内部代码托管、版本控制，且数据完全私有。以下是 **4 种主流方案**，从「功能全」到「轻量极简」，适配不同需求（推荐优先选 Docker 版 Gitea 或 GitLab，新手友好）：

## 一、搭建前提

### 1. 服务器要求

- 系统：优先 **Linux（Ubuntu/CentOS）**（Git 服务在 Linux 上更稳定，推荐 Ubuntu 20.04/22.04）；也支持 Windows Server 或本地虚拟机（如 VirtualBox、VMware）。
- 配置：
    - 轻量方案（Gitea/纯 Git）：1 核 2G 足够（个人/小团队）；
    - 全功能方案（GitLab）：至少 2 核 4G（推荐 4 核 8G，否则运行卡顿）。
- 网络：服务器需开放 **22 端口（SSH，Git 核心通信）**、80/443 端口（HTTP/HTTPS，图形界面访问）；若为云服务器，需在安全组放行这些端口。

### 2. 基础依赖

- 所有方案都需安装 `Git` 和 `SSH`（Linux 系统默认自带，Windows 需手动安装）；
- 图形界面方案（GitLab/Gitea）需安装 Docker（推荐，避免环境冲突）或直接安装在系统中。

## 二、方案 1：Docker 部署 Gitea（推荐，轻量全功能）

Gitea 是 **轻量级 Git 私服工具**，功能对标 GitLab（支持用户管理、项目权限、分支保护、CI/CD），但资源占用极低（1 核 2G 流畅运行），且部署简单，适合个人/小团队。

### 步骤 1：安装 Docker（若未安装）

Docker 能快速部署 Gitea，避免依赖冲突，按系统执行命令：

#### Ubuntu：

```Bash
# 更新软件源
sudo apt update && sudo apt upgrade -y
# 安装 Docker 依赖
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
# 添加 Docker 官方密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# 添加 Docker 仓库
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 安装 Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io
# 启动 Docker 并设置开机自启
sudo systemctl start docker && sudo systemctl enable docker
# 给当前用户授权（避免每次输 sudo）
sudo usermod -aG docker $USER && newgrp docker
```

#### CentOS：

```Bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io
sudo systemctl start docker && sudo systemctl enable docker
sudo usermod -aG docker $USER && newgrp docker
```

#### 验证 Docker 安装：

```Bash
docker --version  # 显示版本即成功
```

### 步骤 2：Docker 启动 Gitea

用 `docker-compose` 管理（更方便配置持久化存储，避免数据丢失）：

1. 创建目录（存储 Gitea 数据和配置）：

```Bash
mkdir -p ~/gitea/{data,config}  # 数据和配置持久化到本地
cd ~/gitea
```

1. 创建 `docker-compose.yml` 文件：

```Bash
vim docker-compose.yml
```

粘贴以下内容（按需修改端口和路径）：

```YAML
version: "3"
services:
  gitea:
    image: gitea/gitea:latest  # 最新稳定版
    container_name: gitea
    restart: always  # 开机自启
    environment:
      - USER_UID=1000  # 与当前用户 UID 一致（避免权限问题）
      - USER_GID=1000
      - GITEA__database__TYPE=sqlite3  # 数据库用 SQLite（无需额外安装，适合小团队）
    volumes:
      - ./data:/data  # 数据持久化
      - ./config:/etc/gitea  # 配置持久化
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"  # Web 图形界面端口（外部访问用）
      - "2222:22"    # Git SSH 端口（若 22 端口被系统 SSH 占用，用 2222 替代）
```

1. 启动 Gitea：

```Bash
docker-compose up -d  # 后台运行
```

### 步骤 3：初始化 Gitea（Web 界面）

1. 访问 Web 界面：浏览器输入 `http://服务器IP:3000`（如 `http://192.168.1.100:3000`）。
2. 配置基础信息（按提示填写）：
    1. 数据库：默认 SQLite（无需修改）；
    2. 站点设置：
        - 站点标题：自定义（如「我的 Git 私服」）；
        - 服务器域名：填写服务器 IP 或域名（如 `192.168.1.100`）；
        - SSH 端口：若 Docker 映射的是 2222，这里填 `2222`（否则填 22）；
        - 基础 URL：`http://服务器IP:3000`；
    3. 管理员账号：创建第一个账号（自动成为管理员）。
3. 点击「安装 Gitea」，等待 1 分钟左右即可完成。

### 步骤 4：使用 Gitea 私服

1. 创建项目：登录后点击「+」→「新建仓库」，填写仓库名（如 `test-project`），选择「私有」，点击创建。
2. 客户端连接：
    1. 方式 1：SSH 连接（推荐，免密码）：
        - 本地电脑生成 SSH 密钥（已生成可跳过）：
            1. ```Bash
                ssh-keygen -t ed25519 -C "你的邮箱"  # 一路回车默认生成
                ```
        - 复制公钥（Windows：`C:\Users\用户名.ssh\id_ed25519.pub`；Linux/Mac：`~/.ssh/id_ed25519.pub`）；
        - 在 Gitea 网页 → 个人设置 → SSH 密钥 → 粘贴公钥，保存。
        - 克隆仓库：
            1. ```Bash
                git clone ssh://git@服务器IP:2222/用户名/test-project.git  # 端口对应 docker-compose 中的 2222
                ```
    2. 方式 2：HTTP 连接（需输入账号密码）：
        - ```Bash
            git clone http://服务器IP:3000/用户名/test-project.git
            ```

## 三、方案 2：Docker 部署 GitLab（企业级，功能全）

GitLab 是 **企业级 Git 私服工具**，支持用户分组、精细权限控制、CI/CD 流水线、代码审查、漏洞扫描等高级功能，适合中大型团队。但资源占用较高（至少 2 核 4G）。

### 步骤 1：安装 Docker（同方案 1）

### 步骤 2：Docker 启动 GitLab

1. 创建目录（存储 GitLab 数据，需预留足够空间）：

```Bash
mkdir -p ~/gitlab/{config,logs,data}
cd ~/gitlab
```

1. 创建 `docker-compose.yml`：

```YAML
version: "3"
services:
  gitlab:
    image: gitlab/gitlab-ce:latest  # 社区版（免费）
    container_name: gitlab
    restart: always
    environment:
      - GITLAB_OMNIBUS_CONFIG="external_url 'http://服务器IP'; gitlab_rails['gitlab_shell_ssh_port'] = 2222;"  # 外部访问 URL 和 SSH 端口
    volumes:
      - ./config:/etc/gitlab
      - ./logs:/var/log/gitlab
      - ./data:/var/opt/gitlab
    ports:
      - "80:80"    # Web 端口（默认 80，可修改）
      - "2222:22"  # SSH 端口（避免与系统 SSH 冲突）
    shm_size: "256m"  # 共享内存，避免卡顿
```

1. 启动 GitLab（首次启动较慢，需 5-10 分钟）：

```Bash
docker-compose up -d
```

### 步骤 3：初始化 GitLab

1. 访问 Web 界面：`http://服务器IP`（首次访问需设置管理员密码，默认用户名 `root`）。
2. 登录后创建项目、添加用户（路径：管理中心 → 用户 → 新建用户）、分配权限（如添加到项目组）。
3. 客户端连接：同 Gitea（SSH/HTTP 方式），克隆地址格式：
    1. ```Bash
        git clone ssh://git@服务器IP:2222/root/test-project.git
        ```

## 四、方案 3：纯 Git 裸仓库（极简，无图形界面）

若只需「代码托管」，不需要图形界面，可直接用 Git 自带的「裸仓库」搭建，步骤最简单，适合技术人员快速使用。

### 步骤 1：服务器配置（Linux 为例）

1. 安装 Git（若未安装）：

```Bash
sudo apt install git -y  # Ubuntu
# 或
sudo yum install git -y  # CentOS
```

1. 创建 Git 专用用户（避免用 root，更安全）：

```Bash
sudo useradd -m git  # 创建用户并生成家目录
sudo passwd git      # 设置密码（客户端连接时需用）
```

1. 禁止 Git 用户登录 Shell（仅允许 Git 操作）：

```Bash
sudo vim /etc/passwd
```

找到 `git:x:1001:1001::/home/git:/bin/bash`，修改为：

```Bash
git:x:1001:1001::/home/git:/usr/bin/git-shell
```

### 步骤 2：创建裸仓库

1. 切换到 git 用户：

```Bash
su - git
```

1. 创建仓库目录（如 `repo`），初始化裸仓库（裸仓库无工作区，仅存储版本数据）：

```Bash
mkdir -p ~/repo
cd ~/repo
git init --bare test-project.git  # 裸仓库后缀必须是 .git
```

### 步骤 3：客户端连接

1. 方式 1：密码登录（简单，适合临时使用）：

```Bash
# 克隆仓库
git clone git@服务器IP:/home/git/repo/test-project.git
# 提交代码（克隆后修改文件，执行以下命令）
git add .
git commit -m "first commit"
git push origin main
```

首次推送会提示输入 `git` 用户的密码。

1. 方式 2：SSH 免密登录（推荐，同 Gitea 步骤）：
    1. 本地生成 SSH 公钥，复制到服务器的 `git` 用户家目录：
        - ```Bash
            # 本地执行（Windows/Linux/Mac）
            ssh-copy-id -i ~/.ssh/id_ed25519.pub git@服务器IP
            ```
    2. 之后克隆、推送无需输入密码。

## 五、方案 4：Windows 搭建 Git 私服（适合无 Linux 服务器）

若只有 Windows 服务器（如 Windows 10/11 或 Windows Server），可通过「Git for Windows + OpenSSH」搭建：

### 步骤 1：安装依赖

1. 安装 Git for Windows：官网下载 https://git-scm.com/download/win，默认安装（勾选「Add Git to PATH」）。
2. 启用 Windows OpenSSH 服务器：
    1. 控制面板 → 程序 → 启用或关闭 Windows 功能 → 勾选「OpenSSH 服务器」→ 确定。
    2. 启动 SSH 服务：按 `Win+R` 输入 `services.msc`，找到「OpenSSH SSH Server」，设置为「自动启动」并启动。

### 步骤 2：创建裸仓库

1. 打开「Git Bash」（安装 Git 后自带），创建仓库目录：

```Bash
mkdir -p /d/git-repo  # D盘创建 repo 目录
cd /d/git-repo
git init --bare test-project.git
```

### 步骤 3：客户端连接

- 克隆仓库（本地或其他电脑）：

```Bash
git clone git@Windows服务器IP:/d/git-repo/test-project.git
```

- 注意：Windows 路径在 Git Bash 中需用 `/d/` 代替 `D:\`，且需确保 Windows 防火墙放行 22 端口。

## 六、关键配置与注意事项

### 1. 防火墙配置（避免端口不通）

- Linux（Ubuntu）开放端口：
    - ```Bash
        sudo ufw allow 22/tcp  # SSH
        sudo ufw allow 80/tcp  # HTTP
        sudo ufw allow 3000/tcp  # Gitea Web
        sudo ufw reload
        ```
- Linux（CentOS）开放端口：
    - ```Bash
        sudo firewall-cmd --permanent --add-port=22/tcp
        sudo firewall-cmd --permanent --add-port=80/tcp
        sudo firewall-cmd --permanent --add-port=3000/tcp
        sudo firewall-cmd --reload
        ```
- 云服务器（阿里云/腾讯云）：在「安全组」中放行上述端口。

### 2. 数据备份（重要！避免数据丢失）

- Gitea 备份（Docker 版）：
    - ```Bash
        docker exec -it gitea gitite dump -f /data/backup.zip  # 备份到容器内 /data 目录（对应本地 ~/gitea/data）
        ```
- GitLab 备份（Docker 版）：
    - ```Bash
        docker exec -it gitlab gitlab-rake gitlab:backup:create  # 备份文件存储在 ~/gitlab/data/backups
        ```
- 纯 Git 备份：直接复制裸仓库目录（`/home/git/repo`）到其他位置。

### 3. HTTPS 配置（提升安全性）

默认是 HTTP 访问，若需 HTTPS（避免密码明文传输），可通过「Let's Encrypt 免费证书 + Nginx 反向代理」实现：

1. 安装 Nginx：`sudo apt install nginx -y`；
2. 申请免费证书（用 Certbot）：`sudo apt install certbot python3-certbot-nginx -y`，然后执行 `sudo certbot --nginx -d 你的域名`；
3. 配置 Nginx 反向代理（指向 Gitea/GitLab 的 Web 端口）。

### 4. 常见问题排查

- 克隆失败：检查服务器 IP 是否可达、端口是否开放、SSH 密钥是否配置正确；
- Gitea/GitLab 启动慢：检查服务器配置是否满足要求（GitLab 至少 2 核 4G）；
- 权限报错：确保 Docker 挂载目录的权限正确（如 `~/gitea` 目录的所有者是当前用户）。

## 七、方案对比与选择

| 方案          | 优点                               | 缺点                        | 适用场景                  |
| ------------- | ---------------------------------- | --------------------------- | ------------------------- |
| Docker Gitea  | 轻量、功能全、部署简单、资源占用低 | 高级功能（如漏洞扫描）较少  | 个人/小团队（10 人以内）  |
| Docker GitLab | 企业级功能、CI/CD 强大、权限精细   | 资源占用高、启动慢          | 中大型团队（10 人以上）   |
| 纯 Git 裸仓库 | 极简、无依赖、部署快               | 无图形界面、无用户/权限管理 | 技术人员个人使用/临时协作 |
| Windows 搭建  | 无需 Linux 服务器                  | 稳定性较差、端口配置复杂    | 只有 Windows 环境的用户   |

**最终推荐**：个人/小团队优先选「Docker Gitea」，兼顾简单性和功能性；中大型团队选「Docker GitLab」；技术人员快速搭建选「纯 Git 裸仓库」。