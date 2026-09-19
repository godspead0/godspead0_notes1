# 

# Webpack 基础使用指南

Webpack 是一款前端模块化打包工具，能够将多个模块（JS、CSS、图片等）整合为静态资源，优化项目部署性能。以下是完整的 Webpack 基础使用流程、配置说明及补充要点。

## 一、前置准备

### 1. 环境要求

- 安装 Node.js（自带 npm 包管理器），推荐版本 14+
- 验证安装：终端执行 `node -v` 和 `npm -v`，显示版本号即成功

### 2. 初始化项目（可选但推荐）

若项目未初始化，先创建项目文件夹并执行初始化命令，生成 `package.json`（项目依赖配置文件）：

```Bash
# 1. 创建项目文件夹并进入
mkdir webpack-demo && cd webpack-demo

# 2. 初始化 npm（默认配置直接回车，或加 -y 快速生成）
npm init -y
```

## 二、安装 Webpack 相关依赖

需安装核心包 `webpack`（打包核心）和 `webpack-cli`（命令行工具，用于终端执行打包命令），推荐**本地安装**（避免全局版本冲突）：

### 1. 开发依赖安装（常用）

```Bash
# 安装最新稳定版（--save-dev 缩写为 -D，标识为开发环境依赖）
npm install webpack webpack-cli --save-dev
```

### 2. 依赖说明

- `webpack`：打包核心逻辑，处理模块依赖、资源转换
- `webpack-cli`：提供终端命令（如 `webpack` 打包命令）
- 安装后 `package.json` 会新增 `devDependencies` 字段，记录开发依赖

## 三、准备项目代码结构

### 1. 默认目录结构（Webpack 约定）

Webpack 有默认的入口/出口约定，无需额外配置即可打包：

```Plain
webpack-demo/
├── node_modules/       # 依赖包文件夹（安装后自动生成）
├── package.json        # 项目配置文件
├── src/                # 源码文件夹（需手动创建）
│   └── index.js        # 默认入口文件（必须存在）
└── dist/               # 打包输出文件夹（打包后自动生成）
    └── main.js         # 默认出口文件（打包后自动生成）
```

### 2. 示例代码

在 `src/index.js` 中编写测试代码（可引入其他模块）：

```JavaScript
// src/index.js
import { sayHello } from './utils'; // 引入自定义模块（需创建 src/utils.js）

sayHello('Webpack');

// src/utils.js（自定义模块）
export const sayHello = (name) => {
  console.log(`Hello, ${name}!`);
};
```

## 四、配置打包命令（package.json）

Webpack 不推荐全局安装，需在 `package.json` 的 `scripts` 字段中配置自定义命令，通过 `npm run 命令名` 执行打包：

### 1. 配置 scripts

打开 `package.json`，添加 `build` 命令（自定义名称，如 `bundle` 也可）：

```JSON
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack" // 新增打包命令，默认使用生产环境打包
  },
  "devDependencies": {
    "webpack": "^5.90.0",
    "webpack-cli": "^5.1.4"
  }
}
```

### 2. 命令说明

- `npm run build`：执行打包（生产环境，代码会压缩优化）
- 若需开发环境打包（不压缩、保留注释），可添加开发模式命令：
    - ```JSON
        "scripts": {
          "build": "webpack --mode production", // 生产模式（默认）
          "dev": "webpack --mode development"  // 开发模式（未压缩）
        }
        ```

## 五、执行打包操作

终端进入项目根目录，执行配置好的命令：

```Bash
# 生产环境打包（推荐部署用）
npm run build

# 或开发环境打包（调试用）
npm run dev
```

### 打包成功标志

- 终端显示打包进度、文件大小等信息
- 项目根目录生成 `dist` 文件夹，内含 `main.js`（默认出口文件）
- 打开 `dist/main.js` 可查看打包后的整合代码（生产模式会压缩）

## 六、自定义入口/出口配置（核心）

Webpack 支持通过配置文件修改默认的入口（entry）和出口（output），步骤如下：

### 1. 创建配置文件

项目根目录新建 `webpack.config.js`（Webpack 默认识别该文件名）：

```JavaScript
// webpack.config.js
const path = require('path'); // Node.js 内置模块，处理文件路径

module.exports = {
  // 1. 入口配置：指定打包的起始文件（可单入口/多入口）
  entry: './src/index.js', // 默认值，可修改（如 './src/main.js'）
  
  // 多入口示例（适用于多页面应用）
  // entry: {
  //   page1: './src/page1.js',
  //   page2: './src/page2.js'
  // },

  // 2. 出口配置：指定打包后的文件路径和名称
  output: {
    // 出口文件夹路径（必须是绝对路径，通过 path.resolve 生成）
    path: path.resolve(__dirname, 'dist'), // 默认值，可修改（如 'build' 文件夹）
    
    // 出口文件名（单入口时直接写文件名）
    filename: 'main.js', // 默认值，可修改（如 'bundle.js'）
    
    // 多入口时，用 [name] 占位符匹配入口名称（生成 page1.js、page2.js）
    // filename: '[name].js',
    
    // 可选：打包前清空 dist 文件夹（避免旧文件残留）
    clean: true
  }
};
```

### 2. 配置说明

- `path.resolve(__dirname, 'dist')`：`__dirname` 是 Node.js 全局变量，代表当前文件（webpack.config.js）所在目录，结合 `dist` 生成绝对路径
- 多入口配置后，执行打包会在 `dist` 中生成多个出口文件（如 `page1.js`、`page2.js`）
- `clean: true`：Webpack 5+ 新增配置，打包前自动清空 `output.path` 指定的文件夹

## 七、补充要点

### 1. 模式（mode）的作用

| 模式        | 特点                                               | 用途         |
| ----------- | -------------------------------------------------- | ------------ |
| development | 未压缩代码、保留注释、支持调试、构建速度快         | 开发调试     |
| production  | 代码压缩、Tree-Shaking（移除未使用代码）、优化性能 | 项目部署     |
| none        | 无任何优化，默认模式（不推荐）                     | 特殊场景测试 |

### 2. 常见错误及解决

- 「Module not found」：入口文件路径错误，或依赖未安装（执行 `npm install`）
- 「webpack 不是内部命令」：未安装 `webpack-cli`，或未通过 `npm run` 执行命令
- 「路径错误」：`output.path` 必须是绝对路径，需用 `path.resolve` 生成

### 3. 扩展：处理非 JS 资源

Webpack 默认仅支持 JS 和 JSON 文件，若需打包 CSS、图片、字体等，需安装对应的 loader（如 `css-loader`、`style-loader`、`file-loader`），后续可扩展配置。

### 4. 开发工具

- `webpack-dev-server`：开发环境热更新服务器（修改代码后自动刷新页面），需额外安装并配置 `devServer` 字段。

## 总结

Webpack 基础使用流程：

1. 初始化项目（`npm init -y`）
2. 安装依赖（`npm install webpack webpack-cli -D`）
3. 编写源码（默认 `src/index.js`）
4. 配置打包命令（`package.json` 的 `scripts`）
5. （可选）自定义入口/出口（`webpack.config.js`）
6. 执行打包（`npm run build`）