# 主题：Node.js 核心知识点全面解析，包括文件系统模块、路径模块、HTTP 模块、模块化（CommonJS 与 ES6 模块）、NPM 包管理以及 Express 框架，详细介绍各模块的核心作用、引入方式、核心 API 及关键注意点等内容

# Node.js 核心知识点全解析（含扩展）

## 一、文件系统模块（fs 模块）

### 1. 核心作用

用于 Node.js 环境中操作本地文件（读写、创建、删除等），是 Node.js 核心内置模块，无需额外安装。

### 2. 模块引入

```JavaScript
const fs = require('fs'); // CommonJS 语法
// 若支持 ES6 模块（package.json 中添加 "type": "module"）
// import fs from 'fs/promises'; // 推荐使用 promise 版本，支持 async/await
```

### 3. 核心 API（同步/异步/ Promise 版）

Node.js 的 fs 模块提供三种操作模式：**异步回调版**（默认，非阻塞）、**同步版**（阻塞，后缀 `Sync`）、**Promise 版**（fs/promises，支持 async/await，Node.js 10+ 支持）。

#### （1）写入文件

- 异步回调版（覆盖写入）：
    - ```JavaScript
        fs.writeFile(
          '文件路径', // 绝对路径或相对路径
          '写入内容', // 字符串/Buffer
          { encoding: 'utf-8', flag: 'w' }, // flag: w(覆盖)、a(追加)、wx(不存在则创建)
          (err) => {
            if (err) throw err; // 错误处理
            console.log('写入成功');
          }
        );
        ```
- 同步版：
    - ```JavaScript
        try {
          fs.writeFileSync('文件路径', '写入内容', 'utf-8');
          console.log('写入成功');
        } catch (err) {
          console.error(err);
        }
        ```
- Promise 版（推荐）：
    - ```JavaScript
        import fs from 'fs/promises';
        async function writeFile() {
          try {
            await fs.writeFile('文件路径', '写入内容');
            console.log('写入成功');
          } catch (err) {
            console.error(err);
          }
        }
        ```

#### （2）读取文件

- 异步回调版：
    - ```JavaScript
        fs.readFile(
          '文件路径', // 推荐用 __dirname + 相对路径避免路径问题
          'utf-8', // 编码格式，不写则返回 Buffer
          (err, data) => {
            if (err) throw err;
            console.log('读取结果：', data); // data 为字符串（指定编码）或 Buffer
          }
        );
        ```
- 同步版：
    - ```JavaScript
        try {
          const data = fs.readFileSync('文件路径', 'utf-8');
          console.log('读取结果：', data);
        } catch (err) {
          console.error(err);
        }
        ```

#### （3）追加文件

```JavaScript
// 异步
fs.appendFile('文件路径', '追加内容', (err) => {});
// 同步
fs.appendFileSync('文件路径', '追加内容');
```

#### （4）删除文件

```JavaScript
// 异步
fs.unlink('文件路径', (err) => {});
// 同步
fs.unlinkSync('文件路径');
// Promise 版
await fs.unlink('文件路径');
```

### 4. 路径问题解决方案

- 问题：终端执行目录与文件实际目录不一致时，相对路径会拼接错误。
- 解决方法：
    - 使用 **绝对路径**（如 `C:/user/file.txt`）；
    - 使用 `__dirname`（当前模块所在目录的绝对路径）+ 相对路径：
        1. ```JavaScript
            const path = require('path');
            const filePath = path.join(__dirname, 'test.txt'); // 动态拼接绝对路径
            ```

## 二、路径模块（path 模块）

### 1. 核心作用

处理文件路径（拼接、解析、获取文件名/扩展名等），解决不同操作系统（Windows/Linux）路径分隔符差异问题。

### 2. 模块引入

```JavaScript
const path = require('path'); // CommonJS
// ES6 模块
// import path from 'path';
```

### 3. 核心 API

#### （1）路径拼接：`path.join([...paths])`

自动处理分隔符（Windows 用 `\`，Linux 用 `/`），支持 `../`（向上一级）和 `./`（当前目录）：

```JavaScript
path.join('/a', '/b', '../', 'c'); // 结果：'/a/c'（../ 抵消上一级 /b）
path.join(__dirname, 'public', 'images'); // 拼接当前目录下的 public/images 绝对路径
```

#### （2）获取文件名：`path.basename(path[, ext])`

- `path`：文件路径；
- `ext`：可选，要去除的扩展名。

```JavaScript
path.basename('/a/b/c.txt'); // 结果：'c.txt'
path.basename('/a/b/c.txt', '.txt'); // 结果：'c'
path.basename('/a/b/c.html', '.txt'); // 结果：'c.html'（扩展名不匹配，不去除）
```

#### （3）获取扩展名：`path.extname(path)`

返回文件后缀（含 `.`），无扩展名则返回空字符串：

```JavaScript
path.extname('/a/b/c.txt'); // 结果：'.txt'
path.extname('/a/b/c'); // 结果：''
path.extname('/a/b/c.md.html'); // 结果：'.html'（只取最后一个扩展名）
```

#### （4）获取目录名：`path.dirname(path)`

返回文件所在目录的绝对路径：

```JavaScript
path.dirname('/a/b/c.txt'); // 结果：'/a/b'
```

#### （5）解析路径：`path.parse(path)`

将路径解析为对象（包含根目录、目录、文件名、扩展名）：

```JavaScript
path.parse('/a/b/c.txt');
// 结果：
{
  root: '/',
  dir: '/a/b',
  base: 'c.txt',
  name: 'c',
  ext: '.txt'
}
```

## 三、HTTP 模块（创建原生 Web 服务器）

### 1. 核心作用

Node.js 内置模块，用于创建 HTTP 服务器，处理客户端请求并返回响应（无需 Nginx/Apache 等第三方服务器）。

### 2. 模块引入

```JavaScript
const http = require('http'); // CommonJS
// ES6 模块
// import http from 'http';
```

### 3. 创建 Web 服务器三步曲

#### （1）创建服务器实例

```JavaScript
const server = http.createServer();
```

#### （2）监听请求事件（`request` 事件）

客户端发起请求时触发，回调函数接收两个参数：

- `req`：请求对象（包含请求地址、方法、头信息等）；
- `res`：响应对象（用于向客户端返回数据）。

```JavaScript
server.on('request', (req, res) => {
  // 1. 获取请求信息
  const url = req.url; // 请求路径（如 /api/user）
  const method = req.method; // 请求方法（GET/POST/PUT/DELETE）
  const headers = req.headers; // 请求头（如 User-Agent、Content-Type）

  // 2. 设置响应头（解决乱码、跨域等问题）
  res.setHeader('Content-Type', 'text/html; charset=utf-8'); // 响应 HTML，防止乱码
  res.setHeader('Access-Control-Allow-Origin', '*'); // 允许跨域

  // 3. 动态响应（根据请求路径返回不同内容）
  if (url === '/' && method === 'GET') {
    res.end('首页'); // 结束响应并返回内容
  } else if (url === '/api/user' && method === 'GET') {
    res.end(JSON.stringify({ name: '张三', age: 20 })); // 返回 JSON 数据
  } else {
    res.statusCode = 404; // 设置响应状态码（404 未找到）
    res.end('404 页面不存在');
  }
});
```

#### （3）启动服务器（`listen` 方法）

```JavaScript
const port = 3000; // 端口号（1-65535，推荐 3000/8080/8000）
const host = 'localhost'; // 主机名（localhost/127.0.0.1/0.0.0.0，0.0.0.0 允许外网访问）

server.listen(port, host, () => {
  console.log(`服务器启动成功：http://${host}:${port}`);
});
```

### 4. 关键注意点

- 响应必须调用 `res.end()` 结束，否则客户端会一直等待；
- 返回 JSON 数据时需用 `JSON.stringify()` 转为字符串；
- 中文乱码解决方案：设置响应头 `Content-Type: text/html; charset=utf-8`；
- 端口号被占用：更换端口（如 3001），或关闭占用端口的进程。

## 四、模块化（CommonJS 与 ES6 模块）

### 1. 模块化核心概念

将代码按功能拆分到不同文件，每个文件是一个独立模块，通过「导入/导出」实现复用，避免全局变量污染。

### 2. CommonJS 模块（Node.js 默认支持）

Node.js 原生支持的模块化规范，适用于 Node.js 环境（浏览器不支持，需打包工具转换）。

#### （1）导出模块（`module.exports` 与 `exports`）

- `module.exports`：模块的默认导出对象，require 导入的就是它；
- `exports`：是 `module.exports` 的引用（简写），本质指向同一个对象；
- 注意：若直接给 `exports` 赋值（如 `exports = { a: 1 }`），会断开与 `module.exports` 的关联，此时以 `module.exports` 为准。

示例（导出模块：`utils.js`）：

```JavaScript
// 方式 1：module.exports 导出对象
module.exports = {
  add: (a, b) => a + b,
  PI: 3.14
};

// 方式 2：exports 简写（推荐，简洁）
exports.sub = (a, b) => a - b;
exports.multiply = (a, b) => a * b;

// 错误示例：直接赋值 exports 会失效
exports = { divide: (a, b) => a / b }; // 外部导入不到 divide 方法
```

#### （2）导入模块（`require()`）

```JavaScript
// 导入自定义模块（必须加 ./ 或 ../，否则视为第三方模块）
const utils = require('./utils.js'); // 后缀 .js 可省略
console.log(utils.add(1, 2)); // 3
console.log(utils.sub(5, 3)); // 2

// 导入第三方模块（无需路径，Node.js 会从 node_modules 查找）
const express = require('express');

// 导入内置模块（无需路径，直接写模块名）
const fs = require('fs');
```

### 3. ES6 模块（现代规范）

ES6 标准定义的模块化规范，浏览器原生支持（需设置 `type="module"`），Node.js 需在 `package.json` 中添加 `"type": "module"` 启用。

#### （1）导出模块（`export` 与 `export default`）

- `export`：命名导出（可导出多个，导入时需用对应名称）；
- `export default`：默认导出（一个模块只能有一个，导入时可自定义名称）。

示例（导出模块：`utils.js`）：

```JavaScript
// 命名导出（多个）
export const add = (a, b) => a + b;
export const PI = 3.14;

// 默认导出（一个）
export default {
  sub: (a, b) => a - b,
  multiply: (a, b) => a * b
};
```

#### （2）导入模块（`import`）

```JavaScript
// 导入默认导出（自定义名称）
import utils from './utils.js';
console.log(utils.sub(5, 3)); // 2

// 导入命名导出（名称必须匹配，可解构）
import { add, PI } from './utils.js';
console.log(add(1, 2)); // 3
console.log(PI); // 3.14

// 导入所有命名导出（用 * 批量导入）
import * as allUtils from './utils.js';
console.log(allUtils.add(1, 2)); // 3
```

### 4. 两者区别

| 特性             | CommonJS               | ES6 模块                |
| ---------------- | ---------------------- | ----------------------- |
| 导出方式         | module.exports/exports | export/export default   |
| 导入方式         | require()              | import                  |
| 执行时机         | 运行时加载（动态）     | 编译时加载（静态）      |
| 作用域           | 模块级作用域           | 模块级作用域            |
| Node.js 启用方式 | 默认支持               | 需设置 "type": "module" |

## 五、NPM 包管理

### 1. NPM 核心概念

NPM（Node Package Manager）是 Node.js 自带的包管理工具，用于安装/卸载/管理第三方模块（如 express、mysql）。

### 2. 核心命令

#### （1）初始化项目（生成 `package.json`）

`package.json` 是项目配置文件，记录项目依赖、脚本、版本等信息，一个项目只需执行一次：

```Bash
npm init -y # -y 自动生成默认配置（跳过交互）
```

#### （2）安装依赖

- 生产依赖（项目运行时需要，如 express、mysql）：
    - ```Bash
        npm i 包名 -S # 等价于 npm install 包名 --save（默认，Node.js 5+ 可省略 -S）
        # 示例：安装 express 4.17.1 版本
        npm i express@4.17.1 -S
        ```
- 开发依赖（项目开发时需要，如 eslint、webpack）：
    - ```Bash
        npm i 包名 -D # 等价于 npm install 包名 --save-dev
        ```
- 全局安装（工具类包，如 nodemon、npm）：
    - ```Bash
        npm i 包名 -g # 全局安装，可在任意目录使用
        ```

#### （3）卸载依赖

```Bash
npm uninstall 包名 # 卸载生产依赖
npm uninstall 包名 -D # 卸载开发依赖
npm uninstall 包名 -g # 卸载全局依赖
```

#### （4）其他常用命令

```Bash
npm install # 安装 package.json 中记录的所有依赖（克隆项目后执行）
npm update 包名 # 更新指定依赖
npm list # 查看项目依赖树
npm run 脚本名 # 执行 package.json 中 scripts 配置的脚本（如 npm run dev）
```

### 3. `package.json` 关键字段

```JSON
{
  "name": "node-project", // 项目名称
  "version": "1.0.0", // 项目版本
  "type": "module", // 启用 ES6 模块（默认 commonjs）
  "dependencies": {}, // 生产依赖（npm i -S 安装的包）
  "devDependencies": {}, // 开发依赖（npm i -D 安装的包）
  "scripts": { // 自定义脚本
    "dev": "node server.js", // 执行 npm run dev 等价于 node server.js
    "start": "node server.js" // 执行 npm start 等价于 node server.js（可省略 run）
  }
}
```

## 六、Express 框架（Web 开发利器）

### 1. 核心作用

Express 是 Node.js 最流行的 Web 开发框架，基于原生 HTTP 模块封装，简化了路由、中间件、请求处理等操作，提高开发效率。

### 2. 安装与基础使用

#### （1）安装

```Bash
npm i express@4.17.1 -S # 安装指定稳定版本
```

#### （2）基础示例（创建 Web 服务器）

```JavaScript
// 1. 导入模块
const express = require('express');

// 2. 创建 Express 实例
const app = express();

// 3. 定义端口
const port = 3000;

// 4. 监听 GET 请求（路由）
app.get('/', (req, res) => {
  res.send('Hello Express!'); // 替代原生 res.end()，支持直接返回字符串、JSON、HTML
});

// 5. 启动服务器
app.listen(port, () => {
  console.log(`服务器运行在 http://localhost:${port}`);
});
```

### 3. 核心功能

#### （1）路由（请求方法 + 路径映射）

Express 支持所有 HTTP 请求方法（GET/POST/PUT/DELETE 等），通过 `app.方法名(路径, 回调函数)` 定义路由：

```JavaScript
// GET 请求：获取用户列表
app.get('/api/users', (req, res) => {
  // req.query：获取 URL 中 ? 后的查询参数（如 /api/users?page=1&size=10）
  const { page, size } = req.query;
  res.send({
    status: 200,
    message: '获取用户列表成功',
    data: { page, size, list: [] }
  });
});

// POST 请求：添加用户
app.post('/api/users', (req, res) => {
  // req.body：获取 POST 请求体数据（需配合中间件解析，见下文）
  const user = req.body;
  res.send({
    status: 201,
    message: '添加用户成功',
    data: user
  });
});

// 动态路由参数（路径中 : 后跟参数名）
app.get('/api/users/:id', (req, res) => {
  // req.params：获取动态路由参数（如 /api/users/123 → { id: '123' }）
  const { id } = req.params;
  res.send({ status: 200, data: { id, name: '张三' } });
});
```

#### （2）请求数据解析

Express 需通过中间件解析请求体数据：

- 解析 JSON 格式请求体：`express.json()`（Express 4.16+ 内置）；
- 解析表单格式请求体（`application/x-www-form-urlencoded`）：`express.urlencoded({ extended: true })`。

```JavaScript
// 注册中间件（必须在路由之前）
app.use(express.json()); // 解析 JSON 格式
app.use(express.urlencoded({ extended: true })); // 解析表单格式

// 此时 req.body 可获取请求体数据
app.post('/api/login', (req, res) => {
  const { username, password } = req.body;
  if (username === 'admin' && password === '123456') {
    res.send({ status: 200, message: '登录成功' });
  } else {
    res.send({ status: 401, message: '账号或密码错误' });
  }
});
```

#### （3）静态资源托管

通过 `express.static()` 托管静态资源（如 HTML、CSS、JS、图片等），无需手动编写路由：

```JavaScript
// 托管 public 文件夹下的静态资源（访问 http://localhost:3000/images/logo.png 即可）
app.use(express.static('public'));

// 挂载路径前缀（访问需加 /static，如 http://localhost:3000/static/images/logo.png）
app.use('/static', express.static('public'));
```

#### （4）路由模块化

当项目路由较多时，可按功能拆分到不同文件（路由模块化），提高可维护性：

1. 创建路由模块文件（`routes/user.js`）：
    1. ```JavaScript
        const express = require('express');
        const router = express.Router(); // 创建路由对象
        
        // 定义路由（路径无需加 /api，挂载时统一添加）
        router.get('/users', (req, res) => {
          res.send('用户列表');
        });
        
        router.post('/users', (req, res) => {
          res.send('添加用户');
        });
        
        // 导出路由对象
        module.exports = router;
        ```
2. 在主文件（`app.js`）中挂载路由：
    1. ```JavaScript
        const userRouter = require('./routes/user');
        // 挂载路由，统一添加 /api 前缀
        app.use('/api', userRouter);
        ```

## 七、Express 中间件

### 1. 中间件核心概念

中间件是 Express 的核心特性，本质是一个**函数**，在请求到达路由之前或响应发送之前执行，可用于：

- 请求拦截与处理（如解析请求体、身份验证）；
- 响应处理（如设置响应头、统一错误处理）；
- 日志记录、跨域处理等。

中间件函数格式：`(req, res, next) => {}`，其中：

- `req`：请求对象（与路由中的 req 是同一个对象，可共享数据）；
- `res`：响应对象（与路由中的 res 是同一个对象）；
- `next()`：调用下一个中间件或路由（必须调用，否则请求会被挂起）。

### 2. 中间件分类与使用

#### （1）应用级中间件（绑定到 app 实例）

- 全局应用级中间件：通过 `app.use()` 注册，对所有请求生效；
- 局部应用级中间件：仅对指定路由生效。

```JavaScript
// 1. 全局中间件（日志记录）
app.use((req, res, next) => {
  console.log(`[${new Date().toLocaleString()}] ${req.method} ${req.url}`);
  next(); // 必须调用，否则路由无法执行
});

// 2. 局部中间件（身份验证）
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization;
  if (token) {
    next(); // 验证通过，执行下一个中间件/路由
  } else {
    res.status(401).send({ status: 401, message: '未授权' });
  }
};

// 仅对 /api/user 路由生效
app.get('/api/user', authMiddleware, (req, res) => {
  res.send('用户信息');
});
```

#### （2）路由级中间件（绑定到 router 实例）

与应用级中间件用法一致，仅作用于当前路由模块：

```JavaScript
const router = express.Router();

// 路由级全局中间件（对当前路由模块的所有路由生效）
router.use((req, res, next) => {
  console.log('路由级中间件');
  next();
});

router.get('/users', (req, res) => {
  res.send('用户列表');
});
```

#### （3）错误级中间件（统一错误处理）

专门处理请求过程中抛出的错误，**必须有 4 个参数（err, req, res, next）**，且需在所有路由之后注册：

```JavaScript
// 错误级中间件（最后注册）
app.use((err, req, res, next) => {
  console.error('错误信息：', err.message);
  res.status(500).send({
    status: 500,
    message: '服务器内部错误',
    error: err.message
  });
});

// 测试错误抛出
app.get('/error', (req, res, next) => {
  // 手动抛出错误，会被错误级中间件捕获
  next(new Error('手动触发错误'));
});
```

#### （4）内置中间件

Express 4.16+ 内置了 3 个常用中间件：

- `express.static`：静态资源托管（见上文）；
- `express.json`：解析 JSON 格式请求体；
- `express.urlencoded`：解析表单格式请求体。

#### （5）第三方中间件

需通过 npm 安装，扩展 Express 功能：

- `cors`：解决跨域问题；
- `morgan`：日志记录；
- `multer`：文件上传。

示例：使用 `cors` 中间件解决跨域：

```Bash
npm i cors -S # 安装
const cors = require('cors');
app.use(cors()); // 全局注册，允许所有跨域请求
```

### 3. 中间件使用注意事项

1. 中间件必须在路由之前注册（否则路由先执行，中间件无效）；
2. 多个中间件按注册顺序执行；
3. 必须调用 `next()` 才能进入下一个中间件/路由（错误级中间件除外）；
4. `next()` 之后不要写额外代码（会被执行，但无意义）；
5. 多个中间件共享 `req` 和 `res` 对象（可通过 `req.自定义属性` 传递数据）。

## 八、数据库操作（MySQL）

### 1. 核心依赖

使用 `mysql` 模块操作 MySQL 数据库，支持连接池（推荐，提高性能）。

### 2. 安装

```Bash
npm i mysql -S
```

### 3. 连接池配置与使用

连接池避免频繁创建/关闭数据库连接，提高并发处理能力：

```JavaScript
// 1. 导入模块
const mysql = require('mysql');

// 2. 创建连接池
const pool = mysql.createPool({
  host: 'localhost', // 数据库主机（本地默认 localhost）
  user: 'root', // 数据库用户名（默认 root）
  password: '123456', // 数据库密码（自己设置的密码）
  database: 'test', // 要操作的数据库名
  port: 3306, // 数据库端口（默认 3306）
  connectionLimit: 10 // 连接池最大连接数（默认 10）
});

// 3. 通用数据库操作函数（封装 query 方法）
function executeSql(sql, params = []) {
  return new Promise((resolve, reject) => {
    pool.query(sql, params, (err, results) => {
      if (err) {
        reject(err); // 错误回调
        return;
      }
      resolve(results); // 成功回调
    });
  });
}

// 4. 示例：查询数据
async function getUserList() {
  const sql = 'SELECT * FROM users';
  try {
    const results = await executeSql(sql);
    console.log('用户列表：', results);
    return results;
  } catch (err) {
    console.error('查询失败：', err.message);
  }
}

// 5. 示例：插入数据（使用 ? 占位符，防止 SQL 注入）
async function addUser(name, age) {
  const sql = 'INSERT INTO users (name, age) VALUES (?, ?)';
  const params = [name, age];
  try {
    const results = await executeSql(sql, params);
    console.log('插入成功，新增 ID：', results.insertId);
    return results;
  } catch (err) {
    console.error('插入失败：', err.message);
  }
}

// 6. 示例：更新数据
async function updateUser(age, id) {
  const sql = 'UPDATE users SET age = ? WHERE id = ?';
  const params = [age, id];
  try {
    const results = await executeSql(sql, params);
    console.log('更新成功，影响行数：', results.affectedRows);
    return results;
  } catch (err) {
    console.error('更新失败：', err.message);
  }
}

// 7. 示例：删除数据
async function deleteUser(id) {
  const sql = 'DELETE FROM users WHERE id = ?';
  const params = [id];
  try {
    const results = await executeSql(sql, params);
    console.log('删除成功，影响行数：', results.affectedRows);
    return results;
  } catch (err) {
    console.error('删除失败：', err.message);
  }
}
```

### 4. 关键注意事项

- 使用 `?` 占位符传递参数，避免 SQL 注入攻击；
- 封装 Promise 版 `executeSql` 函数，支持 async/await，简化异步操作；
- 数据库连接信息（用户名、密码、数据库名）建议通过环境变量（如 `.env` 文件）配置，避免硬编码。

## 九、身份认证（Session 与 JWT）

### 1. 认证核心场景

用户登录后，服务器需要识别后续请求的身份（如访问个人中心、提交订单），常用认证方案：

- **Session 认证**：适用于服务器渲染（如 Express + EJS）；
- **JWT 认证**：适用于前后端分离（如 Vue/React + Node.js）。

### 2. Session 认证（服务器渲染）

#### （1）核心原理

- HTTP 是无状态协议，服务器通过 **Cookie** 存储 Session ID，客户端每次请求自动携带 Cookie，服务器通过 Session ID 查找用户信息。
- 流程：用户登录 → 服务器创建 Session（存储用户信息）→ 向客户端发送 Cookie（含 Session ID）→ 后续请求携带 Cookie → 服务器通过 Session ID 验证身份。

#### （2）使用步骤

1. 安装依赖：
    1. ```Bash
        npm i express-session -S
        ```
2. 配置 Session：
    1. ```JavaScript
        const express = require('express');
        const session = require('express-session');
        const app = express();
        
        // 配置 Session（必须在路由之前）
        app.use(session({
          secret: 'node-secret', // 密钥（用于加密 Session ID，任意字符串）
          resave: false, // 是否强制重新保存 Session（固定 false）
          saveUninitialized: true, // 是否保存未初始化的 Session（固定 true）
          cookie: {
            maxAge: 24 * 60 * 60 * 1000 // Session 过期时间（1 天，单位 ms）
          }
        }));
        ```
3. 登录与身份验证：
    1. ```JavaScript
        // 登录接口（设置 Session）
        app.post('/login', (req, res) => {
          const { username, password } = req.body;
          if (username === 'admin' && password === '123456') {
            // 存储用户信息到 Session（req.session 是 Session 对象）
            req.session.user = { id: 1, username: 'admin' };
            res.send({ status: 200, message: '登录成功' });
          } else {
            res.send({ status: 401, message: '账号或密码错误' });
          }
        });
        
        // 个人中心接口（验证 Session）
        app.get('/profile', (req, res) => {
          // 从 Session 中获取用户信息
          const user = req.session.user;
          if (user) {
            res.send({ status: 200, data: user });
          } else {
            res.send({ status: 401, message: '请先登录' });
          }
        });
        
        // 退出登录（销毁 Session）
        app.get('/logout', (req, res) => {
          req.session.destroy((err) => {
            if (err) throw err;
            res.send({ status: 200, message: '退出登录成功' });
          });
        });
        ```

#### （3）Session 特性

- 优点：安全性高（用户信息存储在服务器，客户端仅存 Session ID）；
- 缺点：服务器集群部署时需要共享 Session（如 Redis），不适用于前后端分离（跨域时 Cookie 可能被拦截）。

### 3. JWT 认证（前后端分离）

#### （1）核心原理

JWT（JSON Web Token）是一种基于 Token 的认证方案，用户信息加密后存储在客户端（如 localStorage），服务器无需存储 Token，仅通过密钥验证 Token 有效性。

- Token 格式：`Header.Payload.Signature`（三部分用 `.` 连接）；
    - Header：算法信息（如 HS256）；
    - Payload：用户信息（如用户名、ID，可被解码，不建议存敏感信息）；
    - Signature：Header + Payload + 密钥 加密后的结果（用于验证 Token 未被篡改）。
- 流程：用户登录 → 服务器生成 Token → 客户端存储 Token → 后续请求通过 Authorization 头携带 Token → 服务器验证 Token 有效性。

#### （2）使用步骤

1. 安装依赖：
    1. ```Bash
        npm i jsonwebtoken express-jwt -S
        ```

    2. `jsonwebtoken`：生成 JWT Token；
    3. `express-jwt`：解析 JWT Token 并验证（Express 中间件）。
2. 配置与使用：
    1. ```JavaScript
        const express = require('express');
        const jwt = require('jsonwebtoken');
        const expressJWT = require('express-jwt');
        const app = express();
        
        // 密钥（必须保密，生产环境建议通过环境变量配置）
        const secretKey = 'node-jwt-secret';
        
        // 解析 JSON 请求体（用于获取登录参数）
        app.use(express.json());
        
        // 注册 JWT 解析中间件（验证 Token）
        // unless：指定不需要认证的接口（如登录接口）
        app.use(
          expressJWT({ secret: secretKey, algorithms: ['HS256'] })
            .unless({ path: ['/api/login'] })
        );
        
        // 登录接口（生成 Token）
        app.post('/api/login', (req, res) => {
          const { username, password } = req.body;
          if (username === 'admin' && password === '123456') {
            // 生成 Token（expiresIn：过期时间，1h 表示 1 小时）
            const token = jwt.sign(
              { id: 1, username: 'admin' }, // Payload：用户信息
              secretKey, // 密钥
              { expiresIn: '1h' } // 过期时间
            );
            res.send({ status: 200, message: '登录成功', token });
          } else {
            res.send({ status: 401, message: '账号或密码错误' });
          }
        });
        
        // 个人中心接口（需要认证）
        app.get('/api/profile', (req, res) => {
          // 解析后的用户信息存储在 req.user 中
          const user = req.user;
          res.send({ status: 200, data: user });
        });
        
        // 错误级中间件（捕获 JWT 验证错误）
        app.use((err, req, res, next) => {
          if (err.name === 'UnauthorizedError') {
            return res.send({ status: 401, message: 'Token 无效或已过期' });
          }
          res.send({ status: 500, message: '服务器内部错误' });
        });
        
        // 启动服务器
        app.listen(3000, () => {
          console.log('服务器运行在 http://localhost:3000');
        });
        ```
3. 客户端请求示例（携带 Token）：
    1. ```HTTP
        GET /api/profile HTTP/1.1
        Host: localhost:3000
        Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...（Token 字符串）
        ```

#### （3）JWT 特性

- 优点：无状态（服务器无需存储 Token），支持分布式部署，适用于前后端分离；
- 缺点：Token 一旦生成无法撤回（只能等过期），Payload 可被解码（不存敏感信息）。

## 十、跨域问题解决（CORS）

### 1. 跨域核心概念

跨域是指浏览器基于同源策略（协议、域名、端口一致）限制，阻止不同源的客户端向服务器发送请求。例如：

- 前端地址：`http://localhost:8080`（Vue 项目）；
- 后端地址：`http://localhost:3000`（Node.js 项目）；
- 因端口不同，属于跨域，浏览器会拦截请求。

### 2. 解决方法（Express 中）

#### （1）手动设置响应头（简单跨域）

适用于简单请求（GET/POST/HEAD，无自定义请求头）：

```JavaScript
app.use((req, res, next) => {
  // 允许所有源跨域（生产环境建议指定具体域名，如 http://localhost:8080）
  res.setHeader('Access-Control-Allow-Origin', '*');
  // 允许的请求头
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  // 允许的请求方法
  res.setHeader('Access-Control-Allow-Methods', '*');
  // 允许携带 Cookie（需配合 Access-Control-Allow-Origin 指定具体域名）
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  next();
});
```

#### （2）使用 `cors` 中间件（推荐）

支持复杂跨域请求（如 PUT/DELETE、自定义请求头），配置简单：

1. 安装：`npm i cors -S`；
2. 使用：
    1. ```JavaScript
        const cors = require('cors');
        app.use(cors()); // 全局允许跨域（默认配置支持所有源、方法、头）
        
        // 自定义配置（生产环境推荐）
        app.use(cors({
          origin: 'http://localhost:8080', // 允许的前端域名
          methods: ['GET', 'POST', 'PUT', 'DELETE'], // 允许的方法
          allowedHeaders: ['Content-Type', 'Authorization'], // 允许的请求头
          credentials: true // 允许携带 Cookie
        }));
        ```

## 十一、常见问题与最佳实践

### 1. 路径问题

- 始终使用 `path.join(__dirname, '相对路径')` 拼接文件路径，避免相对路径拼接错误；
- 静态资源托管时，建议挂载路径前缀（如 `/static`），避免与路由冲突。

### 2. 错误处理

- 所有异步操作（文件读写、数据库操作、HTTP 请求）必须加错误处理（try/catch 或回调函数 err 参数）；
- 注册全局错误级中间件，统一处理所有错误，返回友好的错误信息。

### 3. 安全性

- 数据库操作使用 `?` 占位符，避免 SQL 注入；
- JWT 密钥、数据库密码等敏感信息，不要硬编码，通过环境变量（如 `.env` 文件 + `dotenv` 模块）配置；
- 生产环境禁用 `console.log`，使用日志模块（如 `winston`）记录日志；
- 跨域时限制允许的源（不要用 `*`），避免恶意网站访问接口。

### 4. 性能优化

- 使用数据库连接池，避免频繁创建/关闭数据库连接；
- 静态资源启用缓存（通过 `express.static` 配置 `maxAge`）；
- 大文件传输使用流（`fs.createReadStream`/`fs.createWriteStream`），避免占用过多内存。

### 5. 开发效率

- 使用 `nodemon` 工具（全局安装 `npm i nodemon -g`），修改代码后自动重启服务器（执行 `nodemon app.js`）；
- 按功能拆分模块（路由、中间件、数据库操作、工具函数），提高代码可维护性；
- 使用 ES6+ 语法（箭头函数、解构赋值、async/await）简化代码。