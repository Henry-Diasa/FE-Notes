## 站在前端研发视角，分析开发脚手架的必要性

架构师视角：开发脚手架的目标是什么，能带来什么价值

开发脚手架的核心目标是： **提升前端研发效能**

### 开发过程的共有操作

- 创建项目

  创建项目一般都会有通用代码

- git操作

  创建仓库、处理代码冲突、同步远程分支、创建版本、发布后的tag等，耗时耗力

- 构建和发布上线

  本地构建、资源发布到静态资源服务器、同步CDN、域名绑定、区分正式和测试服务器，人为操作耗时多

解决上述问题，即可提效。突出脚手架的核心价值

### 脚手架的核心价值

将研发过程：

- 自动化

  从创建项目到git操作，到发布上线的自动化，减少人工的耗时

- 标准化

  交给脚手架完成共有操作，可以统一代码风格，规避git flow混乱：杂乱无章的commit。统一发布和回滚流程

- 数据化

  统计脚手架创建项目、git操作、构建和发布的时间，可以对研发过程进行优化。可以把研发过程保存下来

### 和自动化工具的区别

- 不满足需求。jenkins/travis通常需要在git hooks中触发，负责云构建，不能覆盖开发人员本地功能 

- 定制复杂。jenkins、travis定制过程需要开发插件，开发过程复杂，且使用java语言，对前端开发人员不够友好

## 从使用的角度理解什么是脚手架

### 脚手架简介

脚手架是操作系统的客户端。只不过它是通过命令执行：

```bash
vue create vue-test-app
```

上面命令由3部分组成：

- 主命令：`vue` 
- command：`create` 
- command 的 param：`vue-test-app`   

命令表示创建一个vue项目，项目名为`vue-test-app` 

```bash
vue create vue-test-app --force
```

创建项目并覆盖已有同名目录。`--force` 称为option。用来辅助脚手架确认在特定场景下，用户的选择。

```bash
vue create vue-test-app --fore -r https://registry.npm.taobao.org
```

这里`-r` 也是option，--registry的简写。`-r https://registry.npm.taobao.org` 后面的`https://registry.npm.taobao.org` 是`-r` 的param. `--force`简写是 `-f` 。 `--force` 也有param：`--force` `-f` 是 `--force true` 的缩写。

### 脚手架执行原理

`vue create vue-test-app`  命令干了些啥？

1. 终端在环境变量中找到vue指令，`which vue`  指令寻找（mac系统下）本机运行which vue的到结果：    `/usr/local/bin/vue` 

2. 进入node路径，图1，关注bin和lib文件夹(本机是进入`/usr/local/` )，进入bin路径后，ls -la得到vue信息：`lrwxr-xr-x  1 root wheel  39  9 19 15:15 vue -> ../lib/node_modules/@vue/cli/bin/vue.js` 得到vue是一个指向`/usr/local/lib/node_modules/@vue/cli/bin/vue.js` 的软连接

   ![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/usr_local.png)

   图1：bin文件夹中是软链接。lib中是软链接指向的全局文件（npm包）

3. 以上，实际执行的是`/usr/local/lib/node_modules/@vue/cli/bin/vue.js` 文件。解析command后调用相应方法(create)执行。

**执行过程图解：**

![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

- 终端中输入 `vue create vue-test-app`
- 终端解析出`vue` 命令
- 终端在环境中找到`vue` 命令（使用which vue指令）
- 终端根据`vue` 命令链接到实际文件vue.js
- 终端利用node 执行vue.js
- vue.js解析`command` /`opiton`
- vue.js执行`command` (对应的方法)
- 执行完毕，退出执行

### 开发脚手架的步骤

以`vue-cli` 为例

1. 创建npm项目`mkdir vue-cli && cd vue-cli && npm init -y` 项目中包含`bin/vue.js` 文件，`package.json` 配置：

   ```json
   "bin": {
     "vue": "bin/vue.js"
   }
   ```

   最后，把项目发布到npm。

2. 将`vue-cli` 安装到`node` 的`lib/node_modules` 中：`npm i -g @vue/cli`
3. 在`node` 的`bin` 目录下配置`vue` 软链到`lib/node_modules/@vue/cli/bin/vue.js` 。这一步，npm安装时自动完成。

### 需要解决的问题

1. 为什么全局安装`@vue/cli` 后会添加的命令为vue？

   `package.json` 中`bin` 配置了key`vue`，`npm` 全局安装时，会根据这个配置生成软链接。

2. 全局安装`@vue/cli` 时发生了什么？

   全局安装@vue/cli时

   -  下载npm包到全局的node_modules文件夹下

   - 按`package.json` 中`bin` 的配置，生成软链接

3. 为什么`vue` 指向一个`js` 文件，我们却可以通过`vue` 命令直接执行它？

   - `vue.js` 文件第一行添加`#! /usr/bin/env node` 告诉操作系统，在直接打开文件时，从环境变量中找寻`node` 命令，使用`node` 执行文件。`/usr/bin/env` 在终端中打印出来的是环境变量。
   - `npm` 按照`package.json` 中`bin` 的配置，生成了软链接`vue` 指向了`vue.js` 。所以输入`vue` 就可以执`vue,js` 。

**涉及到的bash命令**

- `chmod 777 filename` 可以将文件改为可执行文件

  `drwxr-xr-x`  最后一个`x` 表示这是一个可执行文件

- `echo $PATH` 打印环境变量

**手动创建软连接**

创建软连接指向js：在node bin目录下，运行命令 `ln -s js路径 name`。之后可以在任何位置，执行`name` 命令

## 脚手架原理进阶

1. 为什么说脚手架本质是操作系统的客户端？它和我们在PC上安装的应用/软件有什么区别？

   因为node是操作系统的客户端，不是编写的对应脚手架js文件是客户端。区别是没有gui，node是通过命令行方式，传入参数执行

2. 如何为node脚手架命令创建别名？

   相当于为现有的vue 这个命令在添加个别名：新建软连接指向他就好了。在`node/bin` 下执行：`ln -s /vue vue2` 即可使用`vue2` 指向`vue` 。`windows` 系统是通过新建快捷方式创建

3. 描述脚手架命令执行的全过程：

   ![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

## 脚手架的开发流程

### 开发流程

1. 创建`npm` 项目

2. 创建脚手架入口文件，最上方添加：

   ```javascript
   #!/usr/bin/env node
   ```

3. 配置`package.json` ，添加`bin` 属性

4. 编写脚手架代码

5. 将脚手架发布到`npm` 

### 安装使用

**作业1**

- 安装

  ```bash
  npm i -g jolly-test
  ```

- 使用

  ```bash
  $ jolly-test -V
  1.0.1
  $ jolly-test --version
  1.0.1
  $ jolly-test init test --option
  执行init流程 test --option
  
  ```

## 脚手架开发难点

- 分包：将复杂的系统拆分成若干个模块

- 命令注册，使用若干命令实现对应功能

  ```bash
  vue create
  vue add
  vue invoke
  ```

- 参数解析

  ```bash
  vue command [options] <params>
  ```

  主命令、command、带params的options等

- options有全称 `--version` 有简写`-v` `-h` 怎么实现

- 帮助文档

  - Global help (针对主命令)
    - Usage
    - Options
    - Commands

  ```bash
  jolly@JollydeMacBook-Pro % vue -h
  Usage: vue <command> [options]
  
  Options:
    -V, --version                              output the version number
    -h, --help                                 output usage information
  
  Commands:
    create [options] <app-name>                create a new project powered by vue-cli-service
    add [options] <plugin> [pluginOptions]     install a plugin and invoke its generator in an already created project
    invoke [options] <plugin> [pluginOptions]  invoke the generator of a plugin in an already created project
    inspect [options] [paths...]               inspect the webpack config in a project with vue-cli-service
    serve [options] [entry]                    serve a .js or .vue file in development mode with zero config
    build [options] [entry]                    build a .js or .vue file in production mode with zero config
    ui [options]                               start and open the vue-cli ui
    init [options] <template> <app-name>       generate a project from a remote template (legacy API, requires @vue/cli-init)
    config [options] [value]                   inspect and modify the config
    outdated [options]                         (experimental) check for outdated vue cli service / plugins
    upgrade [options] [plugin-name]            (experimental) upgrade vue cli service / plugins
    migrate [options] [plugin-name]            (experimental) run migrator for an already-installed cli plugin
    info                                       print debugging information about your environment
  
    Run vue <command> --help for detailed usage of given command.
  ```

还有

- 交互式命令行
- 日志打印
- 命令行文字颜色变化
- 网络通信：HTTP/WebSocket
- 文件处理

......

### 脚手架本地调试

**没有分包的本地调试**

假定在开发jolly-test脚手架，要本地调试`jolly-test`命令

- 在`package.json` 所在目录下`npm link` 

  - 此时会按照 `package.json` 中`bin` 配置在`/usr/local/bin` 中生成软链`jolly-test`（`bin` 配置的）
  - 先指向/usr/local/lib/node_modules/jolly-test，这也是一个软连接
  - /node_modules/jolly-test 指向项目的本地实际路径

- 在`package.json` 所在目录的上级`npm i -g 文件夹名` 也可以实现`npm link` 的效果。唯一的区别在于`node_modules` 中软链指向的地址写法：`npm link` `node_modules` 中软链指向的地址是绝对路径：

  ```bash
  lrwxr-xr-x   1 jolly  wheel    37  1 10 20:42 jolly-test -> /Users/jolly/Desktop/imooc/jolly-test
  ```

  而`npm i -g jolly-test` `node_modules` 中软链指向的地址是相对路径：

  ```bash
  lrwxr-xr-x  1 jolly wheel  48 1 10 16:24 jolly-test -> ../../../../Users/jolly/Desktop/imooc/jolly-test
  ```

**分包的本地调试**

假定开发`jolly-cli`，和`jolly-utils`位于`jolly-cli-dev`文件夹下。`jolly-cli`引入依赖`jolly-utils` 

- 首先在`jolly-utils`中`npm link` 放到全局`node_modules` 文件夹下， 然后在`jolly-cli` 运行指令`npm link jolly-utils`，并在 `package.json` 中手动配置依赖。

- `npm` `package` 的形式之一，就是本地的某个文件夹。所以`npm` 提过如下写法，支持本地包的引入（`jolly-cli`引入依赖`jolly-utils`）

```json
  "dependencies": {
    "@jolly-cli/utils": "file:../utils"
  },
```

	配置之后，需要 `npm install`（存在一个现象，自用mac上， `npm link` 也能更新依赖，包括安装非本地依赖。可能因为没有使用nvm管理的node。之后验证吧），但发布时需要具备像`lerna publish` 将本地连接解析成线上连接的功能对依赖进行处理。

**本地调试后发布上线**

- 使用`npm link` 的要先`npm unlink`
  - 没有本地分包调试的，可以直接发布了
  - 有本地分包的，如开发`jolly-cli` ，应先在 `jolly-cli` 中`npm unlink jolly-utils` , 这时`package.json` 中对应依赖会被移除。然后回到 `jolly-utils` 中 `npm unlink` 并先对 `jolly-utils` 进行发布。发布成功之后，在 `jolly-cli` 运行 `npm i -S jolly-utils` 安装依赖，再对 `jolly-cli` 进行发布。
- 通过 `"file:../utils"` 引入依赖的，要使用 `lerna publish` 进行发布。

### npm link & npm unlink

**npm link**

- `npm link` 将当前项目链接到 `node` 全局 `node_modules` 中作为一个库文件，并解析 bin 配置创建可执行文件
- `npm link your-lib` 将当前项目中 `node_modules` 下指定的库文件链接到 `node` 全局 `node_modules` 下的库文件

**npm unlink**

- `npm unlink` 将当前项目从 `node` 全局 `node_modules` 中移除
- `npm unlink your-lib` 将当前项目中的库文件依赖移除

## 脚手架命令注册和参数解析

**不使用第三方库进行解析**

使用 `node` `process` 模块。`process` 是 `node` 的进程事件，`process.argv`  属性会返回一个数组，其中包含当 `Node.js` 进程被启动时传入的命令行参数。第一个元素是 `process.execPath`（启动 `Node.js` 进程的可执行文件的绝对路径名）。 第二个元素是正被执行的 `JavaScript` 文件的路径。 其余的元素是任何额外的命令行参数。

下图是输入 `imooc-test init` 后的 `process.argv`

![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/process.argv.jpg)

## 使用lerna进行脚手架开发

lerna 是一个优化基于 git + npm 的多 package 项目的管理工具

### lerna解决的问题

- 大型项目、多 `packages` 的重复操作

  - 本地分包之间的link

  - 依赖安装

  - 单元测试

  - 代码提交

    所有 `package` 提交到一个仓库

  - 代码发布

    每一个 `package` 都要发布成独立的 `npm` 包，version 升级

- 大型项目、多 `packages` 的版本一致性

  - 发布时版本一致性

    `packages` 之间版本一致，只需保证一个版本下的 `packages` 可以协同工作即可，不必考虑向下兼容

  - 发布后相互依赖的版本升级

    一个包升级后，依赖这个包的其他包，也要进行升级

- leran 提升操作的标准化

  标准的发布流程：发布前的 bump version，检查代码是否提交了，生成仓库的 tag 标签等，最后进行 npm 发布

### lerna与架构

lerna 是一个架构优化的产物，它揭示了一个架构真理：项目复杂度提升后，就需要对项目进行架构优化。架构优化的主要目标往往都是以效能为核心。

标椎化带来的好处是减少出错，减少排查错误的成本。

### lerna 开发脚手架流程

紫色为常用命令

![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/lerna%E5%BC%80%E5%8F%91%E6%B5%81%E7%A8%8B.png)

### 创建 npm Organazition

`pakage.json` 中 `name` 以 `@` 开头，说明是个 `group` ，需要先注册一个 `group` ，否则后面没法发布和获取

**添加 Organization**

- 点击 add Organization

![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/add_organization.jpg)

- 创建 group

  ![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/create_group.png)

- 创建后，在packages中查看

  ![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/packages.png)

### 创建 lerna repo

```bash
npm i -g lerna
mkdir lerna-test && lerna-test
npm init -y
lerna init
npm i
```



### lerna 命令解释

- `lerna init`

  直接运行，没有参数。创建一个lerna仓库(lerna repo)，生成 `lerna.josn` 并且在 `package.json` 中写入开发依赖

  ```json
  "devDependencies": {
    "lerna": "^3.22.1"
  }
  ```

- `lerna create packageName [location]` 

  创建新的 `package` 并根据交互命令行输入生成 `package.json`；可以指定路径。

  ```powershell
  lerna create get-npm-info ./utils/get-npm-info
  ```

- `lerna add` 

  ```bash
  lerna add <package>[@version] [--dev] [--exact] [--peer] 
  ```

  向packages添加依赖，不传package，表示向所有package添加依赖。 **`<package>` 是路径，不是包名**

  ```bash
  lerna add axios utils/get-npm-info
  ```

  - 一次只能添加一个包，使用 `npm` 安装多包的写法，只会安装第一个
  - **`npm link` 后的包，在 `lerna add` 后，要重新 `npm link`，不然会报错。这是坑**

- `lerna link`

  将相互依赖的本地包进行链接

- `lerna exec`

  对每个package执行任意 `shell` 命令，上下文在package文件夹中

  ```bash
  lerna exec -- rm -rf ./node_modules
  ```

  ```bash
   lerna exec --scope @imooc-cli-dev/core（包名） -- ls -la
  ```

  通过 `--scope` 指定包名，对特定包执行 `shell` 脚本

- `lerna run`

  执行**存在**于每个包中的 npm script

  ```bash
  lerna run test 
  ```

  ```bash
  lerna run --scope @imooc-cil-dev/utils test 
  ```

  通过 `--scope` 指定包名，对特定包执行 `npm` 命令

- `lerna clean`

  删除所有packages的node_modules目录，但是不会修改package.json的依赖，且始终不会删除根目录下的 `node_modules` 中的文件夹

- `lerna boorstrap`

  安装 `lerna repo` 中所有包的依赖，包括包与包的相互依赖

- `lerna version`
  bump version。1.0.1 -> 1.0.2 / 1.1.0 ...

- `lerna changed`
  表示自上一个版本，有哪些package做了变更

- `lerna diff`
  检查做出的修改，前提是有提交仓库或者有`git commite`

- `lerna publish`

  发布 `npm` 包，登录部分，跟 `npm publish` 一致。 

### lerna项目发布问题说明

在发布之前，确保已经初始化了git仓库

1. `lerna version`
   *`leran version` bump 版本号后，会提交到远程仓库，并在远程上产生一个tag。`leran version` 成功之后，是不能`lerna publish` 发布成功的，会说没有修改*

2. `lerna publish`

   - 如果要发布，不要先使用`lerna version` 。提交完之后，直接使用`lerna publish` 第一步就会 bump 版本号。

   - **删除多余或者调试产生的tag的方法**，大胆调试发布流程

     - 先在远程上删除tag 
     - 在本地仓库根目录中，找到文件夹`./git/refs/tags` 中删除之前在远程上删除的tag
     - 接下来，就可以手动修改`lerna.josn`中的version，使用一个没有发不过的较低的版本号 `lerna publish` 了

   - 使用 group 发布的包，默认情况都是私有的，publish不了。需要在 `package.json` 中追加配置

     ```json
     "publishConfig": {
       "access": "public"
     }
     ```

   - `lerna publish` 成功之后的日志

     ![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/publishSucc.png)

## lerna 源码阅读

### 准备源码

准备工作

- 下载源码
- 安装依赖
- IDE打开

准备完成的标准

- 找到入口文档
- 能够本地调试

学习目标

- lerna 源码结构和执行流程
- `import-local` 

### 源码学到的知识点

- `require('.')` === `require('./')`

- `npm` `file:` 接相对路径的处理

  `commands/publish/index.js` `resolveLocalDependencyLinks()` 将本地连接解析成线上连接。

  ![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/resolveLocalDependencyLinks.jpg)

### yargs 用法

yargs 可以用来帮助构建交互式命令行工具，解析参数、生成优雅的用户界面。

**基本使用：**

```javascript
#!/usr/bin/env node
const yargs = require('yargs/yargs');
const { hideBin } = require('yargs/yargs'); // 用作参数解析

const arg = hideBin(process.argv); // 获得参数列表(Array)

yargs(arg)
		.argv;
```

**Api解读**

- `yargs(arg).strict()` 在输入的命令找不到时，报错。默认会输出 `--help` 的内容及默认报错

- `yargs(arg).usage(msg)` 展示命令的用法。可以理解成，输出msg

  - ```javascript
    yargs(arg).usage('Usage: $0 <command> [option]')
    ```

    `$0` 指向的是 `command` 。如本地开发一个 `yargs-test` 命令行工具，输入 `yargs-test ls`

    `argv:  { _: [ 'ls' ], cliVersion: '1.0.0', '$0': 'yargs-test' }`

- `yargs(arg).demandCommand(1, '最少需要一个命令')` 表示最少需要几个命令，不需要有 `.strict()` 的调用

- `yargs(arg).recommendCommands()` 直接调用，输入错误时，提示最近似的命令

- `yargs(arg).alias('h', 'help')` 为命令设置别名，为 `--help` 使用别名 `-h` 

- `yargs.wrap(yargs(arg).terminalWidth())` 设置脚手架输出的命令行宽度。`yargs(arg).terminalWidth()` 表示终端的宽度。整体表示占满整行

- `yargs(arg).epilogue('msg')` 在结尾输出信息

  - `dedent` npm 包去掉行缩进或者行首空格，不会去掉空行。使用方式：

    ```javascript
    dedent` 111
    111`
    dedent(`111
    111`)
    ```

-    `yargs(arg).options(obj)` 增加一个全局选项，所有 `command` 都能访问。添加一个 `--debug` option

  ```javascript
  yargs(arg).options({
  	debug: {
  		type: 'boolean',
  		describe: 'Bootstrap debug mode'
  	}
  })
  ```

  更多命令可以从参考 [`yargs github`](https://github.com/yargs/yargs/blob/HEAD/docs/api.md#optionskey-opt)

- `yargs(arg).option('opt', obj)` 和 `yargs(arg).options(obj)` 作用一样，只不过一次只能添加一个全局命令

- `yargs(arg).group(Array, str)` 为命令分组，在 `--help` 显示时，分组显示命令。参数是一个 `option` 数组，第二个参数是分组名

- `yargs(arg).command(cmd, desc, [builder], [handler])` 注册命令，`lerna --option` 中的 `--option`，这个方法很重要

  - `cmd` 是 `command` 的格式 
  - `desc` 是 `cmd` 的描述
  - `builder` 函数，在执行命令之前执行
  - `handler` 函数，`command` 的行为

  [`npm`](https://www.npmjs.com/package/yargs) 上的例子

  ```javascript
  #!/usr/bin/env node
  const yargs = require('yargs/yargs')
  const { hideBin } = require('yargs/helpers')
  
  yargs(hideBin(process.argv))
    .command('serve [port]', 'start the server', (yargs) => {
      yargs
        .positional('port', { 
          describe: 'port to bind on',
          default: 5000
        })
    }, (argv) => {
      if (argv.verbose) console.info(`start server on :${argv.port}`)
      serve(argv.port)
    })
    .option('verbose', {
      alias: 'v',
      type: 'boolean',
      description: 'Run with verbose logging'
    })
    .argv
  ```

    `.positional()` 为 `command` 定义参数，和 `.option()` 类似，但只在 `builder` 函数中的 `yargs` 实例上存在。

  - `lerna` 中  `command` 的对象传参

    ```javascript
    yargs(arg).commamd({
      command,
      alias,
      describe,
      builder,
      handler
    })
    ```

  - [更多](https://github.com/yargs/yargs/blob/HEAD/docs/api.md#commandcmd-desc-builder-handler)

- `yargs(arg).fail(err, msg)` 处理在命令执行失败时的操作，需要先调用 `.strict()`

- `yargs().parse(argv, obj)` 参数解析。`argv` 是 `process.argv.slice(2)`，**和直接传入个 `yargs()` 的`argv` 不同—`hideBin(process.argv)`**， 命令行输入的 `command` 后面的 `option` ，该方法会将 `argv` 和 `obj` 进行合并，可以向 `argv` 中注入参数

  - 这里有个地方要注意

    ```javascript
    let context = {
      otherVersion: pak.version
    }
    yargs().parse(argv, context)
    ```

    `context` 中的版本号 `key` 不要用 `version`，否则输入命令是，将始终执行 `--version`。总之，就是**不要使用跟命令重复的 `key`**

### lerna 命令的执行流程

执行 `lerna ls` ：

- 入口文件 `core/lerna/cli.js` 获取命令号 `option` `argv = process.argv.slice(2)` 
- 调用 `core/lerna/index.js` `mian(argv)` 方法
- 在 `mian(argv)` 方法，根据 `option` `ls` 执行对应的`.command() `：`commands/list/commond.js`
- 执行 `.command() ` 中的 `handler` ：`commands/list/index.js` `factory(argv)`
  - 知识点
    - 深拷贝，使用 `npm` 包 `clone-deep`
    - `this.constructor.name`
    - `runner` 
    - `Promise.then` 微任务执行栈
    - `Promise.resolve` 方法允许调用时不带参数，直接返回一个`resolved` 状态的 `Promise` 对象。

## import-local

用途：当全局 `node_modules` 和 当前项目 `node_modules` 当中同时存在一个脚手架命令时，优先使用当前项目的脚手架。在 `lerna` 项目下，执行命令 `lerna ls`，会输出 `info cli using local version of lerna` 

```bash
jolly@JollydeMacBook-Pro lerna % lerna ls
info cli using local version of lerna
lerna notice cli v3.22.1
@lerna/add
@lerna/bootstrap
@lerna/changed
@lerna/clean
@lerna/create
```

```javascript
const localFile = resolveCwd.silent(path.join(pkg.name, relativePath));
```

**上面这句话，是 `import-local` 中的核心**，作用是查看当前项目中有没有对应的 `npm` 包

`path.relative(localFile, filename)` 找到和本地文件 `localFile` 同名的全局的文件的路径。

**细节**

全局加载执行本地 `cli.js` 和执行后续 `lerna` 流程，都是在全局 `cli.js` 中  `require("npmlog").info("cli", "using local version of lerna");` 代码之前，但 `info cli using local version of lerna` 会先打印，是因为 `command` 方法的 `handler` 中，代码执行都是通过 `Promise.resolve().then()` 添加到微任务中的，所以会在宏任务 ``require("npmlog").info()` 执行之后执行。

**基础知识点**

- `__filename`：当前模块(js)的文件名。 这是当前的模块文件的绝对路径（符号链接会被解析）
- 运行 `node` 代码时，默认会注入 `module` `require` `__filename`  `__dirname` `exports` 五个变量

### 优秀的文件操作库

**`pkg-dir` **

找到 `npm` 或者 `node.js` 项目的根目录

**`findup` **

通过遍历父级目录查账文件或文件夹

**`locate-path` **

从多个文件中获取第一个存在的路径

**`path-exists`**

检查路径是否存在

**`resolve-cwd` **

从当前工作目录，像 `require.resolve` 一样解析模块路径 

**`resolve-from` **

从给定路径，像 `require.resolve` 一样解析模块路径 

**涉及到的 `node.js` 方法**

- `process.cwd()`

  返回 Node.js 进程的当前工作目录(working directory)。

- path.dirname()

- `path.resolve()` 相对路径转换成真实路径

- path.parse()

- `fs.accessSync(filepath)`

  判断文件是否存在（能访问到），不存在会抛出异常

- path.relative()

- path.join()

### `Node.js` 模块路径解析流程

- `Node.js` 项目模块路径解析是通过 `require.resolve`  方法来实现的

- `require.resolve`  就是通过 `Module._resolveFileName` 方法实现的

  - `Module._resolveFileName`  方法核心流程有 3 点： 

    - 判断是否为内置模块

      `NativeModule.canBeRequireByUsers()` 判断是否是可以被加载的内置模块

    - 通过 `Module._resolveLookupPaths`  方法生成 `node_modules` 可能存在的路径

      - 如果路径为 `/`（根目录），直接返回 `['/node_modules']` 
      - 否则，将路径字符串从后往前遍历，查询到 `/`  时，拆分路径，在后面加上 `node_modules`，并传入一个 `paths` 数组，直至查询不到 

    - 通过 `zModule._findPath`   查询模块的真实路径

  - `Module._findPath`  核心流程有 4 点：

    - 查询缓存（将 `request` 和 `paths` 通过 `\x00` 合并成 `cacheKey`）
    - 遍历 `paths`，将 `path` 与 `request` 组成文件路径 `basePath`
    - 如果 basePath 存在则调用 <code>fs.realPathSync</code> 获取文件真实路径
    - 将文件真实路径缓存到 <code>Module._pathCache</code>（key 就是前面生成的 cacheKey）

  - `fs.realPathSync` 核心流程有 3 点：

    - 查询缓存（缓存的 key 为 p，即 `Module._findPath` 中生成的文件路径）
    - 从左往右遍历路径字符串，查询到 `/`时，拆分路径，判断该路径是否为软链接，如果是软链接则查询真实链接，并生成新路_径 p，然后继续往后遍历，这里有 1 个细节需要特别注意：
    - 遍历过程中生成的子路径 base 会缓存在 knownHard 和 cache 中，避免重复查询
    - 遍历完成得到模块对应的真实路径，此时会将原始路径 original 作为 key，真实路径作为 value，保存到缓存中

- `require.resolve.paths`  等价于 `Module._resolveLookupPaths` ，该方法用于获取所有 `node_modules` 可能存在的路径

![](http://imooc-lego-homework.oss-cn-hangzhou.aliyuncs.com/docs/pages/jolly_chen/images/node_Module.png)

图中知识点列表：

- 通过 `require` 加载模块以后，会向 `Module._pathCache` 对象中添加该模块的路径，key：模块名称加 `\x00` 路径，value：实际路径

- `stat(path)` ：查看文件状态。-2 不存在， 0 文件，1 文件夹
- `realPathCache` 是一个 `Map` 对象
- `linux` `stat` 命令打印文件相关信息 `dev_t` 设备编号、`ino_t` 文件的唯一标识
- `fs.realPathSync` 在拿到软连接指向的真实路径后，还会在遍历，查看指向的路径是否还是一个软连接

### 正则表达式

`Modules._findPath()` 中的正则 `/(?:^|\/)\.?\$/` 匹配 ..、/..。包含知识点

- `?` 匹配0到1个
- `(?:)` 表示非匹配分组，作用是不保存括号中匹配的内容
- `/^/`  匹配的是个空字符串''，任意字符都能匹配成功，任意字符中都有空字符串
- `/[^]/` 表示非

## 简历和总结

**简历要有“简历简介”**

### 面试官询问细节的回答

** yargs 开发脚手架**

1. 阐述脚手架命令构成
   1. `command`：命令 vue
   2. `option` 及 `param`
   3. js 文件顶部
2. 阐述脚手架初始化流程
   1. `package.json` 中配置 `bin` 属性，`npm link` 本地安装，生成脚手架命令行
   2. `#! /usr/bin/env node`
   3. `yargs()`
   4. 介绍一下常用方法
3. 脚手架参数的解析
   1. `hideBin(process.argv) / Yargs.argv`
   2. `Yargs.parse(argv, options)`
4. 命令的注册 
   1. `Yargs.command(command, describe, builder, handler)`
   2. `Yargs.command({ command, describe, builder, handler })`

**lerna**

- Lerna 是基于 git+npm 的多`package` 项目管理工具
- 实现原理 
  - 通过 `import-local` 优先调用本地 lerna 命令
  - 通过 `Yargs` 生成脚手架，先注册全局属性，再注册命令，最后通过 `parse` 方法解析参数
  - `lerna` 命令注册时需要传入 `builder` 和 `handler` 两个方法，`builder` 方法用于注册命令专属的 `options`，`handler` 用来处理命令的业务逻辑
  - `lerna` 通过配置 `npm` 本地依赖的方式来进行本地开发，具体写法是在 `package.json` 的依赖中写入：`file:your-local-module-path` ，在 `lerna publish` 时会自动将该路径替换



**`Node.js` 模块路径解析流程**

见上