### 常用的node包

- [superagent](https://visionmedia.github.io/superagent/) 
> SuperAgent 是一个轻量的Ajax API，服务器端（Node.js）客户端（浏览器端）均可使用,SuperAgent具有学习曲线低、使用简单、可读性好的特点,可作为客户端请求代理模块使用，当你想处理get,post,put,delete,head请求时，可以考虑使用SuperAgent。
- [cheerio](https://github.com/cheeriojs/cheerio/wiki/Chinese-README)
> 为服务器特别定制的，快速、灵活、实施的jQuery核心实现. 可以用到一些爬虫类的需求

- [concurrently](npmjs.com/package/concurrently)
> 并行的执行命令

```shell
    # 并行的执行dev:build（监测ts文件变化并打包） dev:start（nodemon执行js文件）
    "dev:build": "tsc -w",
    "dev:start": "nodemon node ./build/crowller.js",
    "dev": "concurrently npm:dev:*" 
```
- [cookie-session](https://www.npmjs.com/package/cookie-session)


### 常见的一些小问题

- express使用body-parse等中间件处理后，新的参数ts检测不到

```javascript
import { Request } from 'express';
  
 // 手动写个接口来进行扩展
interface RequestWithBody extends Request {
  body: {
    [key: string]: string | undefined;
  };
} 
router.post('/getData', (req: RequestWithBody, res: Response) => {
  // 这样req.body 上面就可以检测出新的password等参数了  
  const { password } = req.body;  
}

```

- 自定义express的声明文件

```javascript
    // 这样req上就可以req.name = 'xx' 来进行赋值
    declare namespace Express {
        interface Request {
            name: string;
        }
    }
```