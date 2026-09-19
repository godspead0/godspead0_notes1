# **主题**：HTML 核心知识点全面解读，包括基础概念、文本标签、媒体标签、路径知识、超链接标签、列表标签和表格标签等内容

# HTML 核心知识点完全指南

## 一、HTML 基础概念

### 1. 标签定义与分类

- **标签本质**：网页中用 `<>` 括起来的语法单元，是 HTML 的基本构建块
- **核心规则**：
    - 双标签（闭合标签）：成对出现，结束标签带 `/`，如 `<strong>内容</strong>`
    - 单标签（自闭合标签）：无需结束标签，如 `<hr>`、`<br>`、`<img>`
    - 标签名不区分大小写，但规范要求小写（如 `<HTML>` 不推荐，`<html>` 标准）

### 2. 基本文档结构

```HTML
<!DOCTYPE html> <!-- 声明文档类型为 HTML5，必须放在首行 -->
<html lang="zh-CN"> <!-- 根标签，lang指定页面语言（en/zh-CN等） -->
  <head> <!-- 头部：存放页面元信息，不直接显示 -->
    <meta charset="UTF-8"> <!-- 字符编码（核心，避免中文乱码） -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> <!-- 适配移动端 -->
    <title>网页标题</title> <!-- 浏览器标签栏标题 -->
    <style>/* CSS 样式代码 */</style> <!-- 内部样式表 -->
    <link rel="stylesheet" href="style.css"> <!-- 外部样式表引入 -->
  </head>
  <body> <!-- 主体：用户可见的所有内容（文本、图片、视频等） -->
    网页主体内容
  </body>
</html>
```

### 3. 标签关系

- **父子关系**：父标签包含子标签，如 `<body>` 是 `<p>` 的父标签，`<p>` 是 `<strong>` 的父标签
    - ```HTML
        <body> <!-- 父 -->
          <p>这是<strong>加粗文本</strong></p> <!-- p是strong的父，body是p的父 -->
        </body>
        ```
- **并列关系**：同级标签，如两个 `<p>` 标签、`<head>` 与 `<body>`
    - ```HTML
        <p>第一段</p>
        <p>第二段</p> <!-- 两个p是并列关系 -->
        ```

### 4. 注释语法

- 格式：`<!-- 注释内容 -->`（注释仅开发者可见，浏览器不渲染）
- 用途：解释代码功能、临时注释代码
- 示例：
    - ```HTML
        <!-- 导航栏区域 - 2025-11-21 -->
        <div class="nav">导航内容</div>
        ```

## 二、文本相关标签

### 1. 标题标签（h1-h6）

- 双标签，语义：表示页面层级标题，从 h1 到 h6 字体逐渐变小、权重降低
- 特性：独占一行，自带上下间距
- 规范：一个页面建议只写一个 h1（搜索引擎优化核心）
- 示例：
    - ```HTML
        <h1>一级标题（页面主标题）</h1>
        <h2>二级标题（章节标题）</h2>
        <h3>三级标题（小节标题）</h3>
        <!-- 依此类推到h6 -->
        ```

### 2. 段落标签（p）

- 双标签，语义：表示一个完整段落
- 特性：独占一行，段落之间自动添加间距（margin）
- 示例：
    - ```HTML
        <p>这是第一个段落。HTML 段落标签会自动处理换行和间距，无需手动添加多个空格或换行符。</p>
        <p>这是第二个段落。段落内的文字会自动换行，直到遇到段落结束标签。</p>
        ```

### 3. 换行与水平线

- **换行标签**：`<br>`（单标签），强制换行（类似文本中的 \n）
    - 区别于 `<p>`：`<br>` 仅换行，无额外间距；`<p>` 是段落分隔
- **水平线标签**：`<hr>`（单标签），在页面插入一条水平分隔线
- 示例：
    - ```HTML
        <p>第一行文字<br>强制换行后的文字</p>
        <hr> <!-- 分隔线 -->
        <p>分隔线下方的段落</p>
        ```

### 4. 文本格式化标签

| 标签       | 效果   | 语义特性               | 推荐度 |
| ---------- | ------ | ---------------------- | ------ |
| `<strong>` | 加粗   | 带语义强调（重要内容） | 推荐   |
| `<em>`     | 倾斜   | 带语义强调（强调语气） | 推荐   |
| `<ins>`    | 下划线 | 表示插入的内容         | 推荐   |
| `<del>`    | 删除线 | 表示删除的内容         | 推荐   |
| `<b>`      | 加粗   | 仅样式，无语义         | 不推荐 |
| `<i>`      | 倾斜   | 仅样式，无语义         | 不推荐 |
| `<u>`      | 下划线 | 仅样式，无语义         | 不推荐 |
| `<s>`      | 删除线 | 仅样式，无语义         | 不推荐 |

- 示例：
    - ```HTML
        <p>这是<strong>重要的加粗内容</strong>，这是<em>需要强调的倾斜内容</em></p>
        <p>原价<del>99元</del>，现价<ins>59元</ins></p>
        ```

## 三、媒体标签（图片、音频、视频）

### 1. 图片标签（img）

- 单标签，核心功能：在页面插入图片

- 核心属性：

    - | 属性     | 作用                                  | 必需性           |
        | -------- | ------------------------------------- | ---------------- |
        | `src`    | 图片路径（相对路径/绝对路径/在线URL） | 是               |
        | `alt`    | 图片加载失败时显示的替代文本          | 是（无障碍+SEO） |
        | `title`  | 鼠标悬停时显示的提示文本              | 否               |
        | `width`  | 图片宽度（数值，无单位，默认像素）    | 否               |
        | `height` | 图片高度（数值，无单位）              | 否               |

- 图片缩放规则：

    - 仅设置 `width` 或 `height`：图片按原始比例缩放
    - 同时设置 `width` 和 `height`：可能导致图片变形（需手动保持比例）
    - 推荐做法：只设置一个维度，或用 CSS `object-fit` 控制适配

- 示例：

    - ```HTML
        <!-- 相对路径（图片与HTML文件同目录） -->
        <img src="./logo.png" alt="网站Logo" title="点击返回首页" width="200">
        <!-- 在线路径 -->
        <img src="https://example.com/banner.jpg" alt="首页横幅" height="300">
        <!-- 子文件夹路径 -->
        <img src="./images/product.jpg" alt="产品图片">
        ```

### 2. 音频标签（audio）

- 双标签，支持格式：MP3、WAV、OGG

- 核心属性：

    - | 属性       | 作用                                     | 简写形式                          |
        | ---------- | ---------------------------------------- | --------------------------------- |
        | `src`      | 音频路径（相对/绝对/在线URL）            | -                                 |
        | `controls` | 显示音频控制栏（播放/暂停/音量）         | `controls`（属性名=属性值可简写） |
        | `loop`     | 循环播放                                 | `loop`                            |
        | `autoplay` | 页面加载完成后自动播放（多数浏览器禁用） | `autoplay`                        |
        | `muted`    | 静音播放                                 | `muted`                           |
        | `volume`   | 音量（0-1，默认1）                       | `volume="0.5"`                    |

- 示例：

    - ```HTML
        <audio src="./music.mp3" controls loop muted>
          您的浏览器不支持音频播放，请升级浏览器。 <!-- 降级提示 -->
        </audio>
        ```

### 3. 视频标签（video）

- 双标签，支持格式：MP4、WebM、OGG

- 核心属性（同 audio，新增尺寸控制）：

    - | 属性             | 作用                         | 备注       |
        | ---------------- | ---------------------------- | ---------- |
        | `width`/`height` | 视频尺寸（数值，无单位）     | 同图片规则 |
        | `poster`         | 视频加载前显示的封面图片路径 | 新增属性   |

- 示例：

    - ```HTML
        <video src="./video.mp4" controls autoplay poster="./cover.jpg" width="800">
          您的浏览器不支持视频播放，请升级浏览器。
        </video>
        ```

## 四、路径知识（核心重点）

### 1. 相对路径（推荐使用）

- 定义：以当前 HTML 文件所在位置为起点，查找目标文件的路径

- 常用语法：

    - | 语法        | 含义               | 示例                                       |
        | ----------- | ------------------ | ------------------------------------------ |
        | `./`        | 当前目录（可省略） | `./logo.png` 等同于 `logo.png`             |
        | `../`       | 上一级目录         | `../images/banner.jpg`（HTML在子文件夹中） |
        | `./文件夹/` | 下一级目录         | `./assets/audio/music.mp3`                 |

- 场景示例：

    - ```Plain
        项目文件夹结构：
        project/
        ├─ index.html（当前文件）
        ├─ logo.png
        ├─ assets/
        │  ├─ images/
        │  │  └─ banner.jpg
        │  └─ audio/
        │     └─ music.mp3
        └─ ../（上一级目录）
           └─ common/
              └─ icon.png
        ```

    - ```HTML
        <img src="logo.png" alt=""> <!-- 同目录 -->
        <img src="./assets/images/banner.jpg" alt=""> <!-- 下一级目录 -->
        <audio src="../common/icon.png" alt=""> <!-- 上一级目录 -->
        ```

### 2. 绝对路径（谨慎使用）

- 定义：从文件系统根目录或网络根地址出发的完整路径
- 两种形式：
    - 本地绝对路径：`C:\Users\XXX\project\logo.png`（Windows）、`/Users/XXX/project/logo.png`（Mac/Linux）
    - 网络绝对路径：`https://example.com/images/banner.jpg`
- 缺点：
    - 本地绝对路径：文件迁移、换设备后路径失效（盘符/目录结构可能不同）
    - 网络绝对路径：依赖外部服务器，链接失效则资源无法加载
- 适用场景：
    - 引用外部公共资源（如百度图片、CDN资源）
    - 友情链接（跳转到其他网站）

### 3. 路径选择原则

- 引用项目内资源（图片、音频、本地页面）：用相对路径
- 引用外部资源（其他网站、公共CDN）：用网络绝对路径
- 避免使用本地绝对路径（兼容性极差）

## 五、超链接标签（a）

### 1. 核心功能

- 双标签，作用：实现页面跳转、下载文件、锚点定位

- 核心属性：

    - | 属性       | 作用                                   | 必需性 |
        | ---------- | -------------------------------------- | ------ |
        | `href`     | 目标地址（URL/本地路径/锚点/#）        | 是     |
        | `target`   | 跳转方式（`_self`默认/`_blank`新窗口） | 否     |
        | `download` | 点击时下载目标文件（值为文件名，可选） | 否     |

### 2. 常用场景

#### （1）跳转到外部网站

```HTML
<a href="https://www.baidu.com" target="_blank">百度一下（新窗口打开）</a>
```

#### （2）跳转到本地页面

```HTML
<a href="./about.html">关于我们（同项目页面）</a>
<a href="../contact.html" target="_self">联系我们（当前窗口打开）</a>
```

#### （3）下载文件

```HTML
<a href="./资料.pdf" download="学习资料.pdf">下载PDF资料</a>
<!-- download属性可选，不写则用原文件名 -->
```

#### （4）锚点定位（页面内跳转）

- 步骤1：给目标位置设置锚点（id属性）
- 步骤2：链接指向锚点（`href="#锚点id"`）

```HTML
<!-- 锚点目标 -->
<h2 id="section1">第一章节</h2>
<p>章节内容...</p>
<h2 id="section2">第二章节</h2>
<p>章节内容...</p>

<!-- 锚点链接 -->
<a href="#section1">跳转到第一章节</a>
<a href="#section2">跳转到第二章节</a>
<a href="#">回到顶部（#表示空锚点）</a>
```

#### （5）预留接口（开发初期）

```HTML
<a href="#">待开发功能（点击无跳转）</a>
```

## 六、列表标签

### 1. 无序列表（ul + li）

- 语义：无固定顺序的列表（如导航菜单、商品列表）
- 结构：`<ul>` 是容器，仅能嵌套 `<li>`（列表项），`<li>` 可嵌套其他标签
- 默认样式：列表项前有圆点（可通过 CSS 修改为方块、无标记等）
- 示例：
    - ```HTML
        <ul>
          <li>首页</li>
          <li>产品中心
            <ul> <!-- 嵌套子列表 -->
              <li>手机</li>
              <li>电脑</li>
            </ul>
          </li>
          <li>关于我们</li>
        </ul>
        ```

### 2. 有序列表（ol + li）

- 语义：有固定顺序的列表（如步骤、排名）
- 结构：同无序列表，`<ol>` 容器 + `<li>` 列表项
- 核心属性：`type`（序号类型：1/ A/ a/ I/ i）、`start`（起始序号）
- 示例：
    - ```HTML
        <ol type="A" start="3"> <!-- 从C开始的大写字母序号 -->
          <li>注册账号</li>
          <li>完善资料</li>
          <li>提交订单</li>
        </ol>
        ```

### 3. 定义列表（dl + dt + dd）

- 语义：用于解释说明（如术语解释、网页底部信息）
- 结构：
    - `<dl>`：容器，仅能嵌套 `<dt>`（定义标题）和 `<dd>`（定义描述）
    - 一个 `<dt>` 可对应多个 `<dd>`
- 示例：
    - ```HTML
        <dl>
          <dt>HTML</dt>
          <dd>超文本标记语言，用于构建网页结构</dd>
          <dd>不包含样式和交互逻辑</dd>
          
          <dt>CSS</dt>
          <dd>层叠样式表，用于美化网页</dd>
        </dl>
        ```

## 七、表格标签

### 1. 基本结构

- 核心标签：

    - | 标签      | 作用                                     |
        | --------- | ---------------------------------------- |
        | `<table>` | 表格容器                                 |
        | `<tr>`    | 表格行（table row）                      |
        | `<th>`    | 表头单元格（table header），默认加粗居中 |
        | `<td>`    | 内容单元格（table data），默认居左       |

- 基本示例：

    - ```HTML
        <table border="1"> <!-- border="1"显示边框（仅调试用，推荐CSS控制） -->
          <tr> <!-- 第一行（表头） -->
            <th>姓名</th>
            <th>年龄</th>
            <th>性别</th>
          </tr>
          <tr> <!-- 第二行（内容） -->
            <td>张三</td>
            <td>25</td>
            <td>男</td>
          </tr>
        </table>
        ```

### 2. 表格结构标签（推荐使用）

- 作用：优化表格语义，让浏览器更好解析，便于CSS样式控制
- 标签：
    - `<thead>`：表头区域（仅包含表头行 `<tr>`+`<th>`）
    - `<tbody>`：表格主体（包含数据行 `<tr>`+`<td>`）
    - `<tfoot>`：表尾区域（通常用于汇总行）
- 示例：
    - ```HTML
        <table border="1">
          <thead>
            <tr>
              <th>商品</th>
              <th>单价</th>
              <th>数量</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>手机</td>
              <td>3999</td>
              <td>2</td>
            </tr>
            <tr>
              <td>耳机</td>
              <td>599</td>
              <td>1</td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td colspan="2">总计</td> <!-- 合并单元格 -->
              <td>8597元</td>
            </tr>
          </tfoot>
        </table>
        ```

### 3. 表格核心属性

| 属性             | 作用                                        | 单位        |
| ---------------- | ------------------------------------------- | ----------- |
| `border`         | 表格边框宽度（0为隐藏）                     | 像素        |
| `cellspacing`    | 单元格之间的间距（默认2像素）               | 像素        |
| `cellpadding`    | 单元格内容与边框的内边距（默认1像素）       | 像素        |
| `width`/`height` | 表格整体宽度/高度                           | 像素/百分比 |
| `align`          | 表格在页面中的对齐方式（left/center/right） | -           |

### 4. 合并单元格（重点）

- 核心属性：
    - `colspan`：横向合并（跨列），值为合并的单元格数量
    - `rowspan`：纵向合并（跨行），值为合并的单元格数量
- 合并规则：
    - 合并时保留**左上角**的单元格，删除其他被合并的单元格
    - 不能跨结构标签合并（如 `<thead>` 与 `<tbody>` 之间不能合并）
- 示例（横向+纵向合并）：
    - ```HTML
        <table border="1" cellspacing="0">
          <tr>
            <td colspan="2">横向合并2列</td> <!-- 合并第1、2列 -->
            <td>第3列</td>
          </tr>
          <tr>
            <td rowspan="2">纵向合并2行</td> <!-- 合并第1、2行 -->
            <td>第2列第2行</td>
            <td>第3列第2行</td>
          </tr>
          <tr>
            <td>第2列第3行</td>
            <td>第3列第3行</td>
          </tr>
        </table>
        ```

## 八、表单标签（用户交互核心）

### 1. 表单容器（form）

- 双标签，作用：包裹所有表单元素，统一管理数据提交

- 核心属性：

    - | 属性      | 作用                               | 可选值                                                       |
        | --------- | ---------------------------------- | ------------------------------------------------------------ |
        | `action`  | 表单数据提交的目标地址（后端接口） | 接口URL/本地文件路径                                         |
        | `method`  | 提交方式                           | `get`（默认）/`post`                                         |
        | `enctype` | 数据编码格式（上传文件必需）       | `application/x-www-form-urlencoded`（默认）/`multipart/form-data`（文件上传）/`text/plain` |

- `get` 与 `post` 区别：

    - | 特性         | `get` 提交                  | `post` 提交                   |
        | ------------ | --------------------------- | ----------------------------- |
        | 数据可见性   | 数据拼接在URL后，暴露给用户 | 数据在请求体中，不暴露        |
        | 数据大小限制 | 受URL长度限制（约2KB）      | 无限制（适合大文件/大量数据） |
        | 安全性       | 低（不适合密码、敏感信息）  | 高（推荐用于注册、登录等）    |
        | 缓存         | 可被浏览器缓存              | 不被缓存                      |

### 2. 输入框标签（input）

- 单标签，通过 `type` 属性实现不同输入功能，核心属性：
    - `name`：表单数据的键名（后端接收数据的标识，必需）
    - `value`：输入框默认值/选中值
    - `placeholder`：输入提示文本（虚字，不影响提交）
    - `required`：标记为必填项（未填写则阻止提交）
    - `disabled`：禁用输入框（不可编辑，数据不提交）
    - `readonly`：只读输入框（不可编辑，数据可提交）

#### 常用 type 类型：

| `type` 值  | 功能                             | 示例代码                                                     |
| ---------- | -------------------------------- | ------------------------------------------------------------ |
| `text`     | 单行文本框（默认）               | `<input type="text" name="username" placeholder="请输入用户名" required>` |
| `password` | 密码框（输入内容隐藏为*）        | `<input type="password" name="password" placeholder="请输入密码" required>` |
| `radio`    | 单选框（互斥，需相同name）       | `<input type="radio" name="gender" value="male" checked> 男` |
| `checkbox` | 复选框（可多选，需相同name）     | `<input type="checkbox" name="hobby" value="game"> 游戏`     |
| `submit`   | 提交按钮（触发表单提交）         | `<input type="submit" value="立即注册">`（value修改按钮文字） |
| `reset`    | 重置按钮（清空表单内容）         | `<input type="reset" value="重新填写">`                      |
| `file`     | 文件上传（multiple支持多选）     | `<input type="file" name="avatar" multiple accept="image/*">`（accept限制文件类型） |
| `email`    | 邮箱输入框（自动验证格式）       | `<input type="email" name="email" placeholder="请输入邮箱">` |
| `tel`      | 电话输入框（移动端调起数字键盘） | `<input type="tel" name="phone" placeholder="请输入手机号">` |
| `number`   | 数字输入框（仅允许输入数字）     | `<input type="number" name="age" min="18" max="60">`（min/max限制范围） |

### 3. 下拉菜单（select + option）

- 语义：用于大量选项的选择（节省页面空间）
- 结构：
    - `<select>`：下拉菜单容器，`name` 属性为提交键名
    - `<option>`：选项，`value` 为提交值，`selected` 表示默认选中
- 示例：
    - ```HTML
        <label for="city">所在城市：</label>
        <select name="city" id="city">
          <option value="">请选择</option>
          <option value="beijing">北京</option>
          <option value="shanghai" selected>上海</option> <!-- 默认选中 -->
          <option value="guangzhou">广州</option>
        </select>
        ```

### 4. 文本域（textarea）

- 双标签，作用：多行文本输入（如备注、留言）
- 核心属性：
    - `rows`：默认显示行数（控制高度）
    - `cols`：默认显示列数（控制宽度）
    - `resize: none`：CSS属性（禁止拖拽改变大小）
- 示例：
    - ```HTML
        <label for="remark">备注：</label>
        <textarea name="remark" id="remark" rows="5" cols="30" style="resize: none;">默认文本</textarea>
        ```

### 5. 标签关联（label）

- 双标签，作用：
    - 关联表单元素，点击文字可触发元素（扩大点击范围）
    - 提升无障碍访问（屏幕阅读器识别）
- 两种用法：
    - ```HTML
        <!-- 用法1：for属性关联元素id（推荐） -->
        <label for="username">用户名：</label>
        <input type="text" id="username" name="username">
        
        <!-- 用法2：直接包裹表单元素（无需id） -->
        <label>
          <input type="radio" name="gender" value="female"> 女
        </label>
        ```

### 6. 按钮标签（button）

- 双标签，功能与 input 按钮类似，但更灵活（可嵌套图片等）
- 核心属性：`type`（决定按钮功能）
- 示例：
    - ```HTML
        <!-- 提交按钮 -->
        <button type="submit">
          <img src="submit-icon.png" alt=""> 立即提交
        </button>
        
        <!-- 重置按钮 -->
        <button type="reset">重新填写</button>
        
        <!-- 普通按钮（需配合JS实现功能） -->
        <button type="button" onclick="alert('点击成功')">普通按钮</button>
        ```

### 7. 完整表单示例

```HTML
<form action="/register" method="post" enctype="multipart/form-data">
  <!-- 用户名 -->
  <label for="username">用户名：</label>
  <input type="text" id="username" name="username" placeholder="请输入3-16位字符" required><br>

  <!-- 密码 -->
  <label for="password">密码：</label>
  <input type="password" id="password" name="password" placeholder="请输入6-20位密码" required><br>

  <!-- 性别（单选） -->
  <label>性别：</label>
  <input type="radio" name="gender" value="male" checked> 男
  <input type="radio" name="gender" value="female"> 女<br>

  <!-- 爱好（多选） -->
  <label>爱好：</label>
  <input type="checkbox" name="hobby" value="game"> 游戏
  <input type="checkbox" name="hobby" value="reading"> 阅读
  <input type="checkbox" name="hobby" value="travel"> 旅行<br>

  <!-- 所在城市（下拉菜单） -->
  <label for="city">所在城市：</label>
  <select name="city" id="city">
    <option value="beijing">北京</option>
    <option value="shanghai" selected>上海</option>
  </select><br>

  <!-- 头像上传 -->
  <label for="avatar">头像：</label>
  <input type="file" name="avatar" id="avatar" accept="image/*"><br>

  <!-- 备注 -->
  <label for="remark">备注：</label>
  <textarea name="remark" id="remark" rows="3" cols="30" style="resize: none;"></textarea><br>

  <!-- 按钮 -->
  <button type="submit">立即注册</button>
  <button type="reset">重新填写</button>
</form>
```

## 九、布局标签

### 1. 无语义布局标签（常用）

| 标签     | 特性                                 | 用途                           | 俗称   |
| -------- | ------------------------------------ | ------------------------------ | ------ |
| `<div>`  | 块级元素，独占一行，宽度默认100%     | 大区域布局（头部、主体、底部） | 大盒子 |
| `<span>` | 行内元素，不独占一行，宽度由内容决定 | 行内内容包裹（局部文本样式）   | 小盒子 |

- 示例：
    - ```HTML
        <div class="header"> <!-- 头部布局 -->
          <span class="logo">网站Logo</span>
          <div class="nav">导航菜单</div>
        </div>
        <div class="main"> <!-- 主体布局 -->
          <p>主体内容<span class="highlight">重点文本</span></p>
        </div>
        <div class="footer"> <!-- 底部布局 -->
          版权信息
        </div>
        ```

### 2. HTML5 语义化布局标签（推荐）

- 作用：替代 `<div>`，增强代码可读性和SEO优化
- 常用标签：
    - `<header>`：页面头部（导航、Logo）
    - `<nav>`：导航栏
    - `<main>`：页面主体（唯一）
    - `<section>`：章节、区块
    - `<article>`：文章、博客内容
    - `<aside>`：侧边栏、附属信息
    - `<footer>`：页面底部（版权、联系方式）
- 示例：
    - ```HTML
        <header>
          <h1>网站标题</h1>
          <nav>导航菜单</nav>
        </header>
        <main>
          <section>
            <h2>新闻栏目</h2>
            <article>新闻内容1</article>
            <article>新闻内容2</article>
          </section>
          <aside>侧边栏广告</aside>
        </main>
        <footer>版权所有 © 2025</footer>
        ```

## 十、字符实体（特殊字符显示）

- 定义：HTML 中部分字符（如 `<`、`>`）是语法关键字，需通过字符实体显示

- 常用字符实体：

    - | 显示结果 | 字符实体 | 含义                       |
        | -------- | -------- | -------------------------- |
        | 空格     | ` `      | 非换行空格（多个有效）     |
        | `<`      | `<`      | 小于号（避免被解析为标签） |
        | `>`      | `>`      | 大于号                     |
        | `&`      | `&`      | 和号                       |
        | `"`      | `"`      | 双引号                     |
        | `'`      | `'`      | 单引号                     |
        | ©        | `©`      | 版权符号                   |
        | ®        | `®`      | 注册商标符号               |

- 示例：

    - ```HTML
        <p>1. 多个空格：&nbsp;&nbsp;&nbsp;这是三个空格后的文字</p>
        <p>2. 显示标签：&lt;p&gt; 是段落标签 &lt;/p&gt;</p>
        <p>3. 特殊符号：版权所有 &copy; 2025</p>
        ```

## 十一、核心规范与最佳实践

1. **语义优先**：优先使用语义化标签（如 `<h1>`-`<h6>`、`<article>`、`<table>`），而非仅用 `<div>`+CSS
2. **兼容性**：
    1. 避免使用过时标签（如 `<font>`、`<center>`）
    2. 音频/视频提供降级提示（不支持浏览器显示文字）
3. **无障碍**：
    1. 图片必须添加 `alt` 属性
    2. 表单使用 `label` 关联
    3. 页面语言设置 `lang` 属性
4. **SEO优化**：
    1. 合理使用 `<h1>` 标签（单页面一个）
    2. 语义化标签帮助搜索引擎解析内容
5. **代码规范**：
    1. 标签名小写，属性值加引号（推荐双引号）
    2. 缩进一致（2/4个空格）
    3. 注释清晰（关键区域添加说明）