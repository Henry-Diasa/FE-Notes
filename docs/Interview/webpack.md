### 基本配置

- 拆分配置和merge （webpack-merge）
- 启动本地服务（dev-server）
- 处理ES6（babel-loader、preset）
- 处理样式 （less-loader、postcss-loader、autoprefix 注意顺序）
- 处理图片 （file-loader、url-loader包括base64处理）

### 高级配置

- 多入口

  - entry配置改为对象形式

    ```
    entry: {
      index: path.join(__dirname, 'index.js'),
      other: path.join(__dirname, 'other.js'),
    }
    ```

  - Output中filename

    ```
    output: {
      filename: [name].[contentHash:8].js // name对应多入口中的对象的key
    }
    ```

  - Plugins配置多个

    ```
    new HtmlWebpackPlugin({
       template: path.join(__dirname, 'index.html'),
       filename: 'index.html',
       chunks: ['index']
    }),
    new HtmlWebpackPlugin({
       template: path.join(__dirname, 'other.html'),
       filename: 'other.html',
       chunks: ['other']
    })
    
    ```

- 抽离css

  - miniCssExtractPlugin

  - 替换掉之前使用的style-loader （miniCssExtractPlugin.loader）

  - plugins配置

    ```
    new MiniCssExtractPlugin({
      filename: 'css/main.[contentHash:8].css'
    })
    ```

  - 压缩css

    ```
    optimization: {
      minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin()]
    }
    ```

- 抽离公共代码

  - splitChunks

    ```
    splitChunks: {
      chunks: 'all',
      /*
      	initial 入口chunk，对于异步导入的文件不处理
      	async 异步chunk， 只对异步导入的文件处理
      	all 全部chunk
      */
      cacheGroups: {
        vendor: {
          name: 'vendor', // chunk名称
          priority: 1, // 权限更高，优先抽离
          test: /node_modules/,
          minSize: 0, // 大小限制
          minChunks: 1 // 最少复用过几次
        },
        common: {
          name: 'common',
          priority: 0,
          minSize: 0,
          minChunks: 2
        }
      }
    }
    ```

- 懒加载

  ```
  // 定义一个chunk
  import('./dynamic-data.js').then(res => {
  	console.log(res.default.message)
  })
  ```

- 处理JSX和Vue

  - Babel-preset-react
  - Vue-loader

- Module chunk bundle的区别

  - module - 各个源码文件，webpack中一切皆模块
  - chunk - 多模块合并成的，如entry import() splitChunk
  - Bundle - 最终的输出文件

### webpack性能优化

**构建速度**

- 优化babel-loader

  ```
  {
  	test: '/\.js$/',
  	use: ['babel-loader?cacheDirectory'] // 开启缓存
  	include: path.resolve(__dirname, 'src')
  }
  ```

- IgnorePlugin避免引入无用模块

  > 直接不引入，代码中没有

  ```
  new Webpack.IgnorePlugin(/\.\/locale/, /moment/) // 忽略掉moment中的语言包
  // 引入的地方手动引入
  import moment from 'moment'
  import 'moment/locale/zh-cn'
  ```

- noParse 避免重复打包

  > 引入，但不打包

  ```
  module.exports = {
    module: {
      // react.min.js文件就没有采用模块化，忽略对其文件的递归解析处理 
      noParse: [/react\.min\.js$/]
    }
  }
  ```

- happyPack多进程打包

  - JS单线程，开启多进程打包
  - 提高构建速度

- ParallelUglifyPlugin

  - webpack内置Uglify工具压缩JS
  - JS单线程，开启多进程压缩更快
  - 和happyPack同理

- 自动刷新

  ```
  module.exports = {
    watch: true, // 开启监听之后，webpack-dev-server会自动开启刷新浏览器
    watchOptions: {
      ignored: /node_modules/,
      aggregateTimeout: 300, // 监听到变化发生后会等300ms再去执行动作，防止文件更新太快导致重新编译频率太高
      poll: 1000 // 判断文件是否发生变化是通过不停的去询问系统指定文件有没有变化实现的
    }
  }
  ```

- 热更新

  - 自动刷新：整个网页全部刷新，速度较慢

  - 自动刷新：整个网页全部刷新，状态会丢失

  - 热更新：新代码生效，网页不刷新，状态不丢失

  - 配置之后css文件自动开启。js文件需要配置

    ```
    if(module.hot) {
      module.hot.accept(['./main'], () => {
         console.log('ok')
      })
    }
    ```

- DllPlugin

  - 前端框架，体积大，构建慢
  - 较稳定，不常升级版本
  - 同一个版本只构建一次即可，不用每次都重新构建
  - DllPlugin - 打包出dll文件
  - DllReferencePlugin - 使用dll文件

- 优化构建速度（可用于生产环境）

  - 优化babel-loader
  - IgnorePlugin
  - noParse
  - happyPack
  - ParallelUglifyPlugin

**产出代码**

- 小图片base64编码

- bundle加hash

- 懒加载

- 提取公共代码

- IgnorePlugin

- 使用cdn加速 （publicPath）

- 使用production

  - 自动开启代码压缩
  - Vue React等会自动删掉调试代码（如开发环境的warning）
  - 启动Tree-shaking (ES6 Module才能让tree-shaking生效)
    - ES6 Module静态引入，编译时引入
    - Commonjs动态引入，执行时引入
    - 只有ES6 Module才能静态分析，实现tree-shaking

- Scope Hosting

  - 代码体积更小

  - 创建函数作用域更少

  - 代码可读性更好

    ```
    const ModuleConcatenationPlugin = require('webpack/lib/optimize/ModuleConcatenationPlugin')
    
    module.exports = {
      resolve: {
        // 针对npm中的第三方模块优先采用 jsnext:main 中指向的ES6模块化语法的文件
        mainFields: ['jsnext:main', 'browser', 'main']
      },
      plugins: [
        new ModuleConcatenationPlugin()
      ]
    }
    ```

### babel

- 什么是polyfill

  > 主要是提供一些 为了解决一些低版本浏览器 有些方法不支持的一些写法

- Core-js 和 regenerator

  > polyfill的集合

- babel-polyfill即两者的集合

- babel只解析语法不解析api，不处理模块化

  ```
  import '@babel/polyfill' // 需要借助webpack引入模块
  const sum = (a, b) => a + b // 解析为普通函数
  
  Promise.resolve(200).then(data => data) // 不解析
  
  [1,2,3].includes(2) // 不解析
  ```

- babel-polyfill按需引入

  - 文件较大

  - 只有一部分功能，无需全部引入

  - 配置按需引入

    ```
    // .babelrc
    {
      "presets": [
      	[
      	  "@babel/preset-env",
      	  {
      	  	"useBuiltIns": "usage",
      	  	"corejs": 3
      	  }
      	]
      ],
      "plugins": []
    
    }
    ```

- Babel-polyfill的问题

  - 会污染全局环境
  - 如果做一个独立的web系统，无碍
  - 如果做一个第三方lib，则会有问题

- babel-runtime

  - @babel/plugin-transform-runtime 和@babel/runtime

  - 不会污染全局环境，会重新命名（例如Promise可能会命名为_promise）
  - plugin-transform-runtime会将多个重复的helper函数转化为一个单独的模块导入

### 面试题

**前端为何要进行打包和构建**

- 体积更小（Tree-Shaking、压缩、合并）,加载更快
- 编译高级语言或语法（TS, ES6+, 模块化，scss）
- 兼容性和错误检查（polyfill、postcss、eslint）
- 统一、高效的开发环境
- 统一的构建流程和产出标准
- 集成公司构建规范

**loader和plugin的区别**

- loader模块转换器，如less-> css
- plugin扩展插件，如HtmlWebpackPlugin

**babel和webpack的区别**

- babel - JS新语法编译工具，不关心模块化
- webpack - 打包构建工具，是多个loader plugin的集合

**如何产出一个lib**

- Output.library

**为何Proxy不能被polyfill**

- 如class可以用function模拟
- 如Promise可以用callback来模拟
- 但Proxy的功能用Object.defineProperty无法模拟