#### 数据结构

> 数据结构研究的是数据如何在计算机中进行组织和存储，使得我们可以高效的获取数据或者修改数据。在内存世界的**增删改查**

#### 栈 Stack

> 栈也是一种线性结构
>
> 相比数组，栈对应的操作是数组的子集
>
> 只能从一端添加元素，也只能从一端取出元素
>
> 这一端称为栈顶
>
> 栈是一种后进先出的数据结构 （Last In First Out）

#### 栈的应用

- 无处不在的Undo操作（撤销）- 编辑器
- 程序调用的系统栈 - 操作系统
- 括号匹配 - 编译器

#### 栈的实现

```javascript
class ArrayStack {
  constructor() {
    this.data = [];
  },
  push(e) {
    this.data.push(e);
  },
  pop() {
		this.data.pop();
  },
  peek() {
    return this.data[this.data.length - 1];
  },
  size() {
    return this.data.length;
  },
  isEmpty() {
    return this.data.length === 0;
  },
  toString() {
    let result =  '';
    for(let i = 0; i < this.data.length; i++) {

      let sign = i === this.data.length-1 ? '' : ',';

      result += this.data[i] + sign 
    }
    return result;
  }
  
}
```

[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

#### 队列 Queue

> 队列也是一种线性结构
>
> 相比数组，队列对应的操作是数组的子集
>
> 只能有一端（队尾）添加元素，只能从一端（队首）取出元素
>
> 队列是一种先进先出的数据结构（First In First Out FIFO）

#### 队列是实现

```javascript
class ArrayQueue {
	constructor() {
		this.data = [];
	},
	size() {
    return this.data.length;
  },
  isEmpty() {
    return this.data.length === 0;
  },
  enqueue(e) {
    this.data.push(e);
  },
  dequeue() {
    return this.data.shift();
  },
  getFront() {
    return this.data[0];
  }
}
```

#### 循环队列

```javascript
function loopQueue(len){
  // 需要多一个空间 来进行判断队列满
  let _arr = new Array(len+1);
  // 队头指针
  let front = 0;
  // 队尾指针
  let rear = 0;
	// 队列大小
  let size = 0;
  this.enQueue = function(item){
    if(this.isFull()){
      console.log('队列已满!');
      return false;
    }    
    _arr[rear] = item;
    rear = (rear+1)%_arr.length;
    size++;
  }

  this.deQueue = function(){
    if(this.isEmpty()){
      console.log('队列为空!');
      return false;
    }
    let item = _arr[front];
    _arr[front] = null;
    front = (front+1)%_arr.length;
    size--;
    return item;
  }

  this.isEmpty = function(){
    return front === rear;
  }

  this.isFull = function(){
    // 预留一个空间
    return (rear+1) % _arr.length === front;
  }
}
```

[用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

[用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

[ 最小栈](https://leetcode-cn.com/problems/min-stack/)

[设计一个支持增量操作的栈](https://leetcode-cn.com/problems/design-a-stack-with-increment-operation/)

[用栈操作构建数组](https://leetcode-cn.com/problems/build-an-array-with-stack-operations/)

[设计循环队列](https://leetcode-cn.com/problems/design-circular-queue/)

