## 进阶类型

### 接口
> 接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

```javascript
    // 必须包含一个name属性，且类型为string
    interface name {
        name: string
    }
    function logName(obj: name) {
        console.log(obj.name)
    }
    logName({name: 'xx', age: 12})
```

`可选属性`
> 接口里的属性不全都是必需的。 有些是只在某些条件下存在，或者根本不存在，带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个`?`符号。

```javascript
    // age属性是可选的
    interface name {
        name: string
        age?: number
    }
    function logName(obj: name) {
        console.log(obj.name)
    }
    logName({name: 'xx', age: 12})
    logName({name: 'xx'})
```

`只读属性`


