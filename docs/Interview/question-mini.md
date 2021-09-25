#### 图片导出

> 这是一种常见的引流方式，一般同时会在图片中附加一个小程序二维码。

**基本原理**

- 借助 `canvas` 元素，将需要导出的样式首先在 `canvas` 画布上绘制出来 （`api`基本和`h5`保持一致，但有轻微差异，使用时注意即可
- 借助微信提供的 `canvasToTempFilePath` 导出图片，最后再使用 `saveImageToPhotosAlbum` （需要授权）保存图片到本地

> - 小程序中无法绘制网络图片到`canvas`上，需要通过`downLoadFile` 先下载图片到本地临时文件才可以绘制
> - 通常需要绘制二维码到导出的图片上，有一种方式导出二维码时，需要携带的参数必须做编码，而且有具体的长度（`32`可见字符）限制，可以借助服务端生成 短链接 的方式来解决

#### 数据统计

**设计一个埋点sdk**

> 小程序的代码结构是，每一个 `Page` 中都有一个 `Page` 方法，接受一个包含生命周期函数，数据的 业务逻辑对象 包装这层数据，借助小程序的底层逻辑实现页面的业务逻辑。通过这个我们可以想到思路，对`Page`进行一次包装，篡改它的生命周期和点击事件，混入埋点代码，不干扰业务逻辑，只要做一些简单的配置即可埋点，简单的代码实现如下

```
// 代码仅供理解思路
 page = function(params) {
   let keys = params.keys()
   keys.forEach(v => {
       if (v === 'onLoad') {
         params[v] = function(options) {
           stat()   //曝光埋点代码
           params[v].call(this, options)
         }
       }
       else if (v.includes('click')) {
         params[v] = funciton(event) {
           let data = event.dataset.config
           stat(data)  // 点击埋点
           param[v].call(this)
         }
       }
   })
 }

```

> 这种思路不光适用于埋点，也可以用来作全局异常处理，请求的统一处理等场景。

**分析接口**

> 对于特殊的一些业务，我们可以采取 接口埋点，什么叫接口埋点呢？很多情况下，我们有的`api`并不是多处调用的，只会在某一个特定的页面调用，通过这个思路我们可以分析出，该接口被请求，则这个行为被触发了，则完全可以通过服务端日志得出埋点数据，但是这种方式局限性较大，而且属于分析结果得出过程，可能存在误差，但可以作为一种思路了解一下。

**微信自定义数据分析**

> 微信本身提供的数据分析能力，微信本身提供了常规分析和自定义分析两种数据分析方式，在小程序后台配置即可。借助小程序数据助手这款小程序可以很方便的查看

#### 工程化

**工程化做什么**

> 目前的前端开发过程，工程化是必不可少的一环，那小程序工程化都需要做些什么呢，先看下目前小程序开发当中存在哪些问题需要解决：

- 不支持 `css`预编译器,作为一种主流的 `css`解决方案，不论是 `less`,`sass`,`stylus` 都可以提升`css`效率
- 不支持引入npm包 （这一条，从微信公开课中听闻，微信准备支持）
- 不支持`ES7`等后续的`js`特性，好用的`async await`等特性都无法使用
- 不支持引入外部字体文件，只支持`base64`
- 没有 `eslint` 等代码检查工具

**方案选型**

> 对于目前常用的工程化方案，`webpack`，`rollup`，`parcel`等来看，都常用与单页应用的打包和处理，而小程序天生是 “多页应用” 并且存在一些特定的配置。根据要解决的问题来看，无非是文件的编译，修改，拷贝这些处理，对于这些需求，我们想到基于流的 `gulp`非常的适合处理，并且相对于`webpack`配置多页应用更加简单。所以小程序工程化方案推荐使用 `gulp`

**具体开发思路**

> 通过 `gulp` 的 `task` 实现：

- 实时编译 `less` 文件至相应目录
- 引入支持`async`，`await`的运行时文件
- 编译字体文件为`base64` 并生成相应`css`文件，方便使用
- 依赖分析哪些地方引用了`npm`包，将`npm`包打成一个文件，拷贝至相应目录
- 检查代码规范

#### 小程序的问题

- 小程序仍然使用 `WebView` 渲染，并非原生渲染。（部分原生）
- 服务端接口返回的头无法执行，比如：`Set-Cookie`。
- 依赖浏览器环境的 `JS`库不能使用。
- 不能使用 `npm`，但是可以自搭构建工具或者使用 `mpvue`。（未来官方有计划支持）
- 不能使用 `ES7`，可以自己用`babel+webpack`自搭或者使用 `mpvue`。
- 不支持使用自己的字体（未来官方计划支持）。
- 可以用 `base64` 的方式来使用 `iconfont`。
- 小程序不能发朋友圈（可以通过保存图片到本地，发图片到朋友前。二维码可以使用B接口）。
- 获取二维码/小程序接口的限制
- 程序推送只能使用“服务通知” 而且需要用户主动触发提交 `formId`，`formId` 只有7天有效期。（现在的做法是在每个页面都放入`form`并且隐藏以此获取更多的 `formId`。后端使用原则为：优先使用有效期最短的）
- 小程序大小限制 2M，分包总计不超过 8M
- 转发（分享）小程序不能拿到成功结果，原来可以。链接（小游戏造的孽）
- 拿到相同的unionId必须绑在同一个开放平台下。开放平台绑定限制：
  - `50`个移动应用
  - `10`个网站
  - `50`个同主体公众号
  - `5`个不同主体公众号
  - `50`个同主体小程序
  - `5`个不同主体小程序
- 公众号关联小程序
  - 所有公众号都可以关联小程序。
  - 一个公众号可关联10个同主体的小程序，3个不同主体的小程序。
  - 一个小程序可关联500个公众号。
  - 公众号一个月可新增关联小程序13次，小程序一个月可新增关联500次。
- 一个公众号关联的10个同主体小程序和3个非同主体小程序可以互相跳转
- 品牌搜索不支持金融、医疗
- 小程序授权需要用户主动点击
- 小程序不提供测试 `access_token`
- 安卓系统下，小程序授权获取用户信息之后，删除小程序再重新获取，并重新授权，得到旧签名，导致第一次授权失败
- 开发者工具上，授权获取用户信息之后，如果清缓存选择全部清除，则即使使用了`wx.checkSession`，并且在`session_key`有效期内，授权获取用户信息也会得到新的`session_key`

#### 授权获取用户信息流程

- `session_key` 有有效期，有效期并没有被告知开发者，只知道用户越频繁使用小程序，`session_key` 有效期越长
- 在调用 `wx.login` 时会直接更新 `session_key`，导致旧 `session_key` 失效
- 小程序内先调用 `wx.checkSession` 检查登录态，并保证没有过期的 `session_key` 不会被更新，再调用 `wx.login` 获取 `code`。接着用户授权小程序获取用户信息，小程序拿到加密后的用户数据，把加密数据和 `code` 传给后端服务。后端通过 `code` 拿到 `session_key` 并解密数据，将解密后的用户信息返回给小程序

**面试题：先授权获取用户信息再 login 会发生什么？**

- 用户授权时，开放平台使用旧的 `session_key` 对用户信息进行加密。调用 `wx.login` 重新登录，会刷新 `session_key`，这时后端服务从开放平台获取到新 `session_key`，但是无法对老 `session_key` 加密过的数据解密，用户信息获取失败
- 在用户信息授权之前先调用 `wx.checkSession` 呢？`wx.checkSession` 检查登录态，并且保证 wx.login 不会刷新 `session_key`，从而让后端服务正确解密数据。但是这里存在一个问题，如果小程序较长时间不用导致 `session_key` 过期，则 `wx.login` 必定会重新生成 `session_key`，从而再一次导致用户信息解密失败
