#### Webpack中的module

> webpack支持ESM，CommonJS, AMD, Aseets (image, font, video, audio, json)

#### Webpack 中的module如何表达依赖关系

- ESM import 语句
- CommonJS require语句
- AMD define require
- css/sass/less @import

#### chunk和bundle的区别

- chunk是webpack**打包过程**中module的集合。 webpack的打包是从一个入口模块开始，入口模块引用其他模块...。webpack通过引用关系逐个打包模块，这些module就形成了一个chunk。如果有多个入口模块，可能会产出多条打包路径。每条路径都会形成一个chunk
- Bundle 是我们最终输出的一个或多个打包好的文件
- 大多数情况下，一个chunk会生产一个bundle，但也有例外。但是如果加了sourcemap，一个entry， 一个chunk对应两个bundle

#### plugin 和 loader 分别是什么？ 怎么工作的？

**loader**

模块转换器，将非js模块转化为webpack能识别的js模块

本质上，webpack loader将所有类型的文件， 转换为应用程序的 **依赖图** 可以直接引用的模块

**plugin**

扩展插件，webpack运行的各个阶段，将会广播出对应的事件，插件去监听对应的事件

**compiler**

对象，包含了webpack环境的所有配置信息，包括options、loader、plugins。webpack启动的时候实例化，它正在全局是唯一的，可以理解为webpack的实例

**compliation**

包含了当前的模块资源，编译生成资源  webpack在开发模式下运行的时候 每当一个文件变化，就会创建一次新的compliation

#### webpack的打包过程

- 初始化参数： shell webpack.config.js
- 开始编译：初始化一个Compiler对象，加载所有的配置，开始执行编译
- 确定入口：根据entry中的配置，找出所有的入口文件
- 编译模块：从入口文件开始，调用所有的loader，再去递归的找依赖
- 完成模块编译：得到每个模块被编译后的最终内容和之间的依赖关系
- 输出资源：根据得到的依赖关系，组装成一个个包含多个module的chunk
- 输出完成：根据配置，确定要输出的文件名及文件路径



