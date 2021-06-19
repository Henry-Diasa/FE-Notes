## 前导

### 收获

- 如何做架构设计和技术方案设计
- 脚手架核心流程，通过 commander 完成脚手架框架搭建
- 如何让Node项目支持ES Module

### 主要内容

- 脚手架需求分析和架构设计
- 脚手架模块拆分策略和core模块技术方案
- 脚手架执行准备过程实现
- 脚手架命令注册实现（基于commander 库）

### 注意事项

- 本周前半部分偏架构设计，是架构师日常工作

- **架构师应该把整体和局部想清楚在开始做**

- 将代码实现细节抽象，通过系统论思想构建复杂系统：

  建立子系统，关注子系统的输入和输出是什么。然后由子系统构建较复杂的系统，再由较复杂的系统构建更复杂的系统。

## 脚手架整体架构设计

### 大厂怎么做项目

**设计阶段**

- 搞清楚业务或研发过程中的**痛点** -- 为什么有当前业务
- 由痛点形成需求
  - PD(产品) -> PRD文档（产品需求文档）
    - 原型图
    - 预期目标
  - PRD 评审
    - 原型图评审
- 技术方案设计阶段，产生技术方案文档。确定需求在技术上的实现，及确定技术方案实现成本
  - 技术选型
  - 技术架构 -> 架构设计
  - API定义
  - 技术调研
  - 评估技术风险
- 成本可接受，项目立项
  - kick-off（启动）
    - 确定项目成员：PD、PM(项目经理)、前端、后端、测试人员、设计等
- 项目排期（计划）
  - 时间点
  - WBS 文档（工作分解结构）

**实施阶段**

- 软件类项目，交互/视觉设计，输出设计稿
- 开发，输出代码
  - 前后端开发
  - 联调
- 测试，输出测试报告
  - 单元测试（开发人员）
  - 功能测试（测试人员）
  - 性能测试（测试人员）
- 交给产品或业务人员验收
  - 微调
- 上线

<img src="img/structure/开发流程.png" style="zoom:50%;" />

### 痛点

- 创建项目/组件时，重复代码问题
- 协同开发时，git操作不规范问题
- 发布上线耗时，且容易出错问题

### 需求分析

- 通用的研发脚手架
- 通用的项目/组件创建能力
  - 模块支持定制；定制后能快速生效
  - 模板支持快速接入，极低的接入成本
- 通用的项目/组件发布能力
  - 发布过程自动完成标准的git操作
  - 发布成功后自动删除开发分支并创建tag
  - 发布后自动完成云构建、OSS(静态资源服务器)上传、CDN上传、域名绑定
  - 发布过程中支持测试/正式两种模式

### 大厂git操作规范

**分支管理**

- master

  不会再次基础上开发，仅用作代码同步：上线时，将 dev/0.0.1 push到master上，进行 merge 然后打上 release/0.0.1 tag

- dev 开发

  - ~~dev/0.0.1~~
  - dev/0.0.2

- release 发布

  - release/0.0.1 删除 dev/0.0.1

**git操作流程**

<img src="img/structure/git流程.png" style="zoom:50%;" />

### `henry-cli` 架构图

- 脚手架的核心架构
  - 脚手架初始化
  - 完成整个执行流程
    - 命令的执行
    - 异常的监听
    - ......
- 为什么需要后台 API
  - 实现通用能力
  - 接入外部项目
- webSocket 服务
  - 云构建
  - 云发布
- 静态资源
  - 组件构建结果
- 数据体系
  - MySQL 组件相关信息
  - MongoDB 项目模板

<img src="img/structure/imooc-cli架构设计图.png" style="zoom:50%;" />

## 脚手架技术方案设计

### 脚手架拆包策略

参考 `lerna` 项目的拆包，根据模块的功能，将脚手架模块分为：

- 核心模块 -- core
- 命令 commands
  - 初始化
  - 发布
  - 清除缓存
- 模型层 models
  - Command 命令
  - Project 项目
  - Component 组件
  - Npm 模块
  - Git 仓库
- 支撑模块 utils
  - Git 操作
  - 云构建
  - 工具方法
  - API 请求
  - Git API

### core 模块技术方案

实现命令的执行流程

- 准备阶段

  <img src="img/structure/core-prepare.png" style="zoom:50%;" />

  - 检查root启动：避免权限问题。如果是root启动（mac root 用户登录），把权限降级到普通用户
  - 检查用户主目录：要往主目录写入缓存。设计**本地缓存体系**中的**本地文件**
  - 检查环境变量：本地缓存需要
  - 检查是否为最新版本：检查cli版本
  - 提示更新：更新cli

- 命令注册

- 命令执行

### 涉及技术点

**核心库**

- `import-local` 优先执行本地脚手架
- `commander` 实现命令注册

**用到的工具库**

- `npmlog` 打印日志
- `fs-extra` 文件操作。基于 `fs` 封装的
- `semver` 版本比对。检查当前版本是否为最新版本
- `colors` 控制终端文本颜色
- `user-home` 获取用户主目录
- `dotenv` 获取环境变量
- `root-check` root 账户检查和自动降级

## 脚手架执行准备过程实现

### `require()` 支持加载的资源类型

- `.js`

  必须使用 `module.exports`/`exports` 输出模块

- `.json`

  使用 `JSON.parse()` 方法对 `json` 文件进行解析，生成一个对象

- `.node`

  `.node` 文件是 `C++` 插件(`C++ AddOns`)，使用 `process.dlopen()` 打开

- `.any`

  当 `.js` 文件处理

  **使用 `require()` 加载一个内容为 `javascript` 代码的 `.txt` 文件，是可以执行成功的**

### `require()` 支持的路径

- 绝对路径
- 相对路径
- `node` 内置对象
- `node_modules` 中的包

### `npmlog`

- 只能调用 `log.addLevel()` 添加的方法，进行日志输出

  ```
  log.addLevel('warn', 4000, { fg: 'black', bg: 'yellow' }, 'WARN')
  ```

- `log.level`

  默认 level 为 info 级(2000)。低于这个级别的日志，不会被打印

  ```
  // default level
  log.level = 'info'
  // ……
  log.addLevel('verbose', 1000, { fg: 'blue', bg: 'black' }, 'verb')
  ```

  `log.verbose('test', 'msg')` 默认下，本调用的也不会打印

- `log.heading`

  在 log 日志之前，添加前缀

  - 通过 `log.headingStyle` 定义样式

### 检查root启动

- `process.geteuid()`

  - 返回 0， 代表超级管理员
  - 返回501，代表登录用户

- `root-check` 降级权限：

  ```
  var rootCheck = require('root-check');
  rootCheck();
  ```

  - `process.env.SUDO_GID` 可以配置的登录用户分组id

  - `process.env.SUDO_UID` 可以配置的用户标识id

  - `process.setgid(gid)` 使用户所在的分组id

  - `process.setuid(uid)` 设置用户标识id

  - `defaultUid()` 返回平台的 用户标识

    ```
    var DEFAULT_UIDS = {
      darwin: 501, // mac UNIX-like
      freebsd: 1000, // FreeBSD UNIX-like
      linux: 1000, 
      sunos: 100 // Solaris 系统
    }
    ```

### 检查用户主目录

- `path-exists` 检查文件是否存在
- `user-home` 获取用户主目录
- `os-homedir` 源码

### 检查入参

在 debug 模式下，使用 `log.verbose()` 打印日志。但`log.verbose()` 打印日志，正常状态下，是不能打印的。所以这里我们需要解析参数，判断是否是 debug 模式。

- `minimist` 库，解析参数。基本使用

  ```
  const minimist = require('minimist');
  const args = minimist(process.argv.slice(2));
  ```

  console：

  ```
  args:  { _: [], debug: true }
  ```

- 参数解析之后，要修改 `log.LOG_LEVEL`；

### 检查环境变量

`dotenv` 获取环境变量

- 在用户主目录下创建 `.env` 文件存储和读取环境变量

- 默认路径：`path.resolve(process.cwd(), '.env')` 当前文件夹下的

- 从 `.env` 环境中获得的值，放在了 `process.env` 中

  ```
  const dotenvPath = path.resolve('path', '.env'); // 绝对路径
  require('dotenv').config({ // 不传值表示使用默认路径读取
    path: dotenvPath
  })
  ```

- `.env` 文件中的配置：`name=value` 。**没有 `.env` 要手动创建**

  ```
  DB_HOST=localhost
  DB_USER=root
  DB_PASS=s1mpl3
  ```

### 检查当前是否为最新版本

- 获取最新版本号和模块名

- 使用 npm API， 获取所有版本号

  网址 `仓库地址/npm包名 `，私有仓库也可以这样获取版本号

  - `http://registry.npmjs.org/@henry-cli/core`
  - `http://registry.npm.taobao.org/@henry-cli/core`

- 提取所有版本号，比对哪些版本号是大于当前版本号

- 获取最新版本号，提示用户更新到该版本

- **有个坑**

  ```
  lerna create get-npm-info ./utils/get-npm-info
  ```

  创建完包之后，包实际在 `core` 下面

  ```
  lerna success create New package get-npm-info created at ./core\get-npm-info
  ```

- `url-join` 库，将多个 `url` 碎片，拼接生成完整 `url`。

  ```
  const urlJoin  = require('url-join');
  urlJoin('url', 'parturl');
  ```

## commander 库使用

基本代码

```
#! /usr/bin/env node
const commander = require('commander');
const pkg = require('./package.json');

// 获取 commander 单例
// const { program } = commander;

// 实例化一个 Command 实例
const program = new commander.Command();

program
    .version(pkg.version)
    .parse(process.argv);  // 跟 yargs 不一样的参数传入
```

### 常用方法

- `program.usage('<command> [options]')` 说明脚手架命令的用法，不用像 `yargs` 一样获取命令名字 `$0` ，这个方法自动带上脚手架命令

- `program.name()` 指定命令的名字，会打印在`usage` 方法输出的信息前

- `program.option()` 定义全局 option。参数是 `option` 名字、描述、默认值。`yargs.option('optName', obj)`

  ```
  program.option('-d --debug', '是否开启调试模式', true); // 直接指定了简写和全称
  program.option('-e, --env <envName>', '获取环境变量') // 配置带参数的option
  ```

- `program.parse(process.argv)` 命令行参数解析

- `program.arguments()` 设置命令参数

### 注册命令的两种方式

- `program.command()` 注册的是命令

  - 返回新的对象：命令对象

  - 参数说明：

    - 第一个参数可以配置命令名称及命令参数，参数支持**必选**（**尖括号表示**）、**可选**（**方括号表示**）及**变长**参数（**点号表示**，如果使用，只能是最后一个参数）

  - 基本用法

    ```
    const clone = program.command('clone');
    clone
    	.description('clone a repository')
    	.action(() => {
    		console.log('do clone');
      });
    ```

- `program.addCommand()` 注册的是子命令。子命令下，又要注册新的命令。所以这个方法相当于分组

  ```
  const service = new commander.Command('service'); // 子命令
  service
    .command('start [port]') // 返回命令对象，不再是 service
    .description('start service at some port')
    .action(port => {
      console.log('do service start', port);
    }); // 回调函数接收的参数：第一个到第n个是命令的参数，n+1位是option，最后是command命令本身
  program.addCommand(service);
  ```

- 两个命令帮助说明上，是有区别的

  - `clone -h`

    ```
    % comm-test clone -h
    Usage: comm-test clone [options] <source> [destination]
    
    clone a repository
    
    Options:
      -f, --force  是否强制克隆
      -h, --help   display help for command
    ```

  - `service -h`

    ```
    % comm-test service -h
    Usage: comm-test service [options] [command]
    
    Options:
      -h, --help      display help for command
    
    Commands:
      start [port]    start service at some port
      stop            stop service
      help [command]  display help for command
    ```

- 备注：`program.parse(process.argv)` 要放在添加命令逻辑之后

### `commander` 厉害的功能

**自动匹配所有输入的命令**

使用 `program.arguments()`

```
program
  .arguments('<cmd> [options]') // 强制输入一个命令
  .description('test command', {
    cmd: 'command to run',
    options: 'options for command'
  })
  .action((cmd, options) => {
    console.log(cmd, options); // 对任何命令做出行为
  });
```

**`program.command()` 的第二个参数：描述参数**

当`.command()`带有描述参数时，就意味着使用独立的可执行文件作为子命令。`Commander` 将会尝试在入口脚本（例如 `./examples/pm`）的目录中搜索 `program-command` 形式的可执行文件，例如 `pm-install`, `pm-search `。通过配置选项（第三个参数） `executableFile` 可以自定义名字。可执行文件：`pm.js` `program-command.js`

```
program
  .command('install [name]', 'install package') // 使命令 install 出现在 --help 信息中
  .alias('i')
```

输入 `comm-test i`

```
comm-test i s
/Users/jolly/Desktop/imooc/commander-test/node_modules/commander/index.js:925
        throw new Error(executableMissing);
        ^

Error: 'comm-test-install' does not exist
```

**`program.command()` 的第三个参数：配置选项**

多个脚手架之间的串行使用

```
program
  .command('install [name]', 'install package', {
    executableFile: 'npm',
    isDefault: true,
  	hidden: true
  })
  .alias('i')
```

- `executableFile` 使得 `comm-test i` === `npm`，后面输入的 `option`，如果 `npm` 有就执行 `npm option`: `comm-test i -v` === `npm -v`。如果`npm` 没有但当前脚手架有，就执行当前脚手架的 `option`。两者都没有，使用 `executableFile` 指定的命令报错，对于 `npm` 只是展示 `npm` 帮助信息，这是 `npm` 自己的逻辑决定的。
- `isDefaul: true` 表示输入的 `option` 没有指定是，默认执行 `executableFile` 指定的命令
- `hidden: true` 表示不再帮助信息中显示

### 高级定制

**自定义help信息**

```
program.helpInformation = function() {
  // return 'your help infomation\n'; // 也可以在这里输出，不是很推荐
  return ''; // 先去掉原来的help
}
program.on('--help', () => {
  console.log('your help infomation');
})
```

**实现debug模式**

```
program.on('option:debug', () => { // 这里，使用 --debug 没有效果
  if (program.opts().debug) { // 6.2.1 中可以用 program.debug 替换
    process.env.LOG_LEVEL = 'verbose';
  }
});
```

上述代码，会在命令执行前进行处理。

**对未知命令监听**

```
program.on('command:*', (obj) => { // 未知命令都会命中这里的监听
  console.error('未知的命令：' + obj[0]);
  const availableCommands = program.commands.map(cmd => cmd.name());
  console.log('可用命令：' + availableCommands.join(','));
});
```

**注意：** 实现该功能，要注释掉 `program.arguments()` ，以及不让串行脚手架默认执行

## Node 支持 ES Module 的解决方案

### webpack

- 配置 `target: 'node'` 时，才会引入 `path` 等 `node` 内置库

- 垫片

  JS垫片就是，在低级环境中用高级语法时(例如`async`)，在低级环境中手动实现该高级功能，模拟高级环境。形象的解释：垫片的作用就是，使四只脚不一样长的桌子变得平稳，为什么不平稳，因为每个脚的js语法版本不一样。

- `@babel/plugin-transform-runtime` 的作用是，提供“垫片”，例如 `regeneratorRuntime` 的“垫片” -- `regeneratorRuntime` 的实现。以下错误是因为缺少 `@babel/plugin-transform-runtime` 插件。

  ```
  webpack://Node_ES_Module/./core/index.js?:17
  _asyncToGenerator( /*#__PURE__*/regeneratorRuntime.mark(function _callee() {
                                  ^
  
  ReferenceError: regeneratorRuntime is not defined
  ```

  - `babel` 的配置

    还需要安装 `@babel/runtime-corejs3` 库

    ```
    use: {
      loader: 'babel-loader',
        options: {
          presets: [ '@babel/preset-env' ],
            plugins: [
              ['@babel/plugin-transform-runtime',
               {
                 corejs: 3,
                 regenerator: true,
                 useESModules: true,
                 helpers: true
               }]
            ]
        }
    },
    ```

### Node 原生支持

- 将要执行的 `js` 文件和其依赖的 `js` 文件后缀改为 `mjs`

- 使用以下命令执行

  ```
  node --experimental-modules index.mjs
  ```

- `node` 14 之后默认支持 ES Module，直接执行以下命令即可，不需要option `--experimental-modules`

  ```
  node index.mjs
  ```