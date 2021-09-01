### 加载时性能

> 压缩文件大小、使用CDN加速，检查加载性能的指标：白屏时间和首屏时间
>
> 白屏时间：输入网址到页面开始显示内容的时间
>
> 首屏时间：输入网址，到首屏页面渲染完毕的时间

**白屏时间**

> 放到head标签中获取

```js
Date.now() - performance.timing.navigationStart
```

**首屏时间**

> Window.onload中执行

```
Date.now() - performance.timing.navigationStart
```

### 运行时性能

#### DNS预解析

```
<meta http-equiv="x-dns-prefetch-control" content="on"> // 当前页面
<link rel="dns-prefetch" href="http://www.baidu.com">
```

#### HTTP2

**二进制传输**

> 数据分割为帧和流

**多路复用**

> 多个请求共用一个TCP连接

**头部压缩**

**服务端推送**

> 服务端主动推送数据

#### 减少HTTP请求数量

> 

#### 压缩合并文件

> 较少体积和请求数量
>
> 可以使用webpack来进行资源的压缩和公共代码的提取（css、js）

#### 采用svg图片或字体图标

>

#### 按需加载

> import动态加载  runtime合并helper函数

#### 服务端渲染

>有利于seo

#### 使用Defer加载JS

>

#### 静态资源使用CDN

>cdn预热  cdn更新

#### 图片优化

**雪碧图**

**懒加载和预加载**

**图片格式的优化、压缩、webp**

#### 减少重排和重绘

>

#### 长列表优化

>

#### 滚动事件

> 防抖和节流

#### 事件委托

>

#### 合理利用缓存

>

#### 内存处理

**内存的生命周期**

- 内存分配：声明变量，函数，对象的时候，js会自动分配内存
- 内存使用：调用的时候，使用的时候
- 内存回收

**垃圾回收机制**

- 引用计数垃圾回收 （a对象对b对象有访问权限，那么称a引用b对象）  循环引用
- 标记清除

**内存泄漏**

- 全局变量
- 定时器和dom引用
- 闭包

```js
// sizeof
const seen = new WeakSet()

function sizeOfObject(object) {
	if(!object) {
		return 0
	}
	let bytes = 0
	const props = Object.keys(object)
	for(let i =0;i<props.length;i++) {
		const key = props[i]
		bytes+=calculator(key)
		
		if(typeof object[key] == 'object' && object[key]!==null) {
			if(seen.has(object[key])) {
				continue
			}
			seen.add(object[key])
		}
		bytes +=calculator(object[key])
	}
	return bytes
}

function calculator(object) {
	const type = typeof object
	switch(type) {
		case 'string':
			return object.length * 2
			break;
	  case 'boolean':
    	return 4
    	break
    case 'number':
     return 8
     break;
    case 'object':
    	if(Array.isArray(object)) {
    			return object.map(calculator).reduce((res, current) => res + current, 0)
    	}else{
    		return sizeOfObject(object)
    	}
    	break
    default: 
      return 0		
	}
}
```















