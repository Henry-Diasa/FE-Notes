#### 前言

我们以`查找指定目录下的最大文件`为例，感受从

回调函数 -> Promise -> Generator -> Async

异步处理方式的改变。

#### API介绍

为了实现这个功能，我们需要用到几个 Nodejs 的 API，所以我们来简单介绍一下。

**fs.readdir**

readdir 方法用于读取目录，返回一个包含文件和目录的数组。

**fs.stat**

stat 方法的参数是一个文件或目录，它产生一个对象，该对象包含了该文件或目录的具体信息。此外，该对象还有一个 isFile() 方法可以判断正在处理的到底是一个文件，还是一个目录。

#### 思路分析

我们基本的实现思路就是：

1. 用 `fs.readdir` 获取指定目录的内容信息
2. 循环遍历内容信息，使用 `fs.stat` 获取该文件或者目录的具体信息
3. 将具体信息储存起来
4. 当全部储存起来后，筛选其中的是文件的信息
5. 遍历比较，找出最大文件
6. 获取并返回最大文件

然后我们直接上代码吧。

#### 回调函数

```javascript
var fs = require('fs');
var path = require('path');

function findLargest(dir, cb) {
    // 读取目录下的所有文件
    fs.readdir(dir, function(er, files) {
        if (er) return cb(er);

        var counter = files.length;
        var errored = false;
        var stats = [];

        files.forEach(function(file, index) {
            // 读取文件信息
            fs.stat(path.join(dir, file), function(er, stat) {

                if (errored) return;

                if (er) {
                    errored = true;
                    return cb(er);
                }

                stats[index] = stat;

                // 事先算好有多少个文件，读完 1 个文件信息，计数减 1，当为 0 时，说明读取完毕，此时执行最终的比较操作
                if (--counter == 0) {

                    var largest = stats
                        .filter(function(stat) { return stat.isFile() })
                        .reduce(function(prev, next) {
                            if (prev.size > next.size) return prev
                            return next
                        })

                    cb(null, files[stats.indexOf(largest)])
                }
            })
        })
    })
}
```

使用方式为：

```javascript
// 查找当前目录最大的文件
findLargest('./', function(er, filename) {
    if (er) return console.error(er)
    console.log('largest file was:', filename)
});
```

#### Promise

```javascript
var fs = require('fs');
var path = require('path');

var readDir = function(dir) {
    return new Promise(function(resolve, reject) {
        fs.readdir(dir, function(err, files) {
            if (err) reject(err);
            resolve(files)
        })
    })
}

var stat = function(path) {
    return new Promise(function(resolve, reject) {
        fs.stat(path, function(err, stat) {
            if (err) reject(err)
            resolve(stat)
        })
    })
}

function findLargest(dir) {
    return readDir(dir)
        .then(function(files) {
            let promises = files.map(file => stat(path.join(dir, file)))
            return Promise.all(promises).then(function(stats) {
                return { stats, files }
            })
        })
        .then(data => {

            let largest = data.stats
                .filter(function(stat) { return stat.isFile() })
                .reduce((prev, next) => {
                    if (prev.size > next.size) return prev
                    return next
                })

            return data.files[data.stats.indexOf(largest)]
        })

}
```

使用方式为：

```javascript
findLargest('./')
.then(function(filename) {
    console.log('largest file was:', filename);
})
.catch(function() {
    console.log(error);
});
```

#### Generator

```javascript
var fs = require('fs');
var path = require('path');

var co = require('co')

var readDir = function(dir) {
    return new Promise(function(resolve, reject) {
        fs.readdir(dir, function(err, files) {
            if (err) reject(err);
            resolve(files)
        })
    })
}

var stat = function(path) {
    return new Promise(function(resolve, reject) {
        fs.stat(path, function(err, stat) {
            if (err) reject(err)
            resolve(stat)
        })
    })
}

function* findLargest(dir) {
    var files = yield readDir(dir);
    var stats = yield files.map(function(file) {
        return stat(path.join(dir, file))
    })

    let largest = stats
        .filter(function(stat) { return stat.isFile() })
        .reduce((prev, next) => {
            if (prev.size > next.size) return prev
            return next
        })

    return files[stats.indexOf(largest)]

}
```

使用方式为：

```javascript
co(findLargest, './')
.then(function(filename) {
    console.log('largest file was:', filename);
})
.catch(function() {
    console.log(error);
});
```

#### Async

```javascript
var fs = require('fs');
var path = require('path');

var readDir = function(dir) {
    return new Promise(function(resolve, reject) {
        fs.readdir(dir, function(err, files) {
            if (err) reject(err);
            resolve(files)
        })
    })
}

var stat = function(path) {
    return new Promise(function(resolve, reject) {
        fs.stat(path, function(err, stat) {
            if (err) reject(err)
            resolve(stat)
        })
    })
}

async function findLargest(dir) {
    var files = await readDir(dir);

    let promises = files.map(file => stat(path.join(dir, file)))
    var stats = await Promise.all(promises)

    let largest = stats
        .filter(function(stat) { return stat.isFile() })
        .reduce((prev, next) => {
            if (prev.size > next.size) return prev
            return next
        })

    return files[stats.indexOf(largest)]

}
```

使用方式为：

```javascript
findLargest('./')
.then(function(filename) {
    console.log('largest file was:', filename);
})
.catch(function() {
    console.log(error);
});
```

