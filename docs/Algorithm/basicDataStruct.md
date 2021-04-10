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

#### 栈的例题分析

**判断字符串括号是否合法**

> 【题目】字符串中只有字符'('和')'。合法字符串需要括号可以配对。比如：
>
> 输入："()"
>
> 输出：true
>
> 解释：()，()()，(())是合法的。)(，()(，(()是非法的。
>
> 请你实现一个函数，来判断给定的字符串是否合法

首先，分析题目的时候，要特别注意以下 4 点，归纳为“四步分析法”。

- 模拟：模拟题目的运行。

- 规律：尝试总结出题目的一般规律和特点。

- 匹配：找到符合这些特点的数据结构与算法。

- 边界：考虑特殊情况。

通过模拟可以得出如下规律

- 每个左括号'('或者右括号')'都完成配对，才是合法的
- 配对可以通过**消除法**来消掉合法的括号，如果最后没有任何字符了，那么就是合法字符串
- 奇数长度的字符串总是非法的。
- 同时可以看出每次消除的时候都是把最近的匹配的一项消除掉。也就是满足栈的特点

最后还需要考虑一些边界情况

- 字符串为空
- 字符串只有 1 个或者奇数个
- 字符串是"(((())))"嵌套很多层的是否可以处理

最后可以写出如下代码

```java
boolean isValid(String s) {

  // 当字符串本来就是空的时候，我们可以快速返回true

  if (s == null || s.length() == 0) {

    return true;

  }

  // 当字符串长度为奇数的时候，不可能是一个有效的合法字符串

  if (s.length() % 2 == 1) {

    return false;

  }

  // 消除法的主要核心逻辑: 

  Stack<Character> t = new Stack<Character>();

  for (int i = 0; i < s.length(); i++) {

    // 取出字符

    char c = s.charAt(i);

    if (c == '(') {

      // 如果是'('，那么压栈

      t.push(c);

    } else if (c == ')') {

      // 如果是')'，那么就尝试弹栈

      if (t.empty()) {

        // 如果弹栈失败，那么返回false

        return false;

      }

      t.pop();

    }



  return t.empty();

}

```

**复杂度分析**：每个字符只入栈一次，出栈一次，所以时间复杂度为 O(N)，而空间复杂度为 O(N)，因为最差情况下可能会把整个字符串都入栈。

**1. 深度扩展**

如果仔细观察，你会发现，栈中存放的元素是一样的。全部都是左括号'('，除此之外，再也没有别的元素，优化方法如下。

**栈中元素都相同时，实际上没有必要使用栈，只需要记录栈中元素个数**

```java
public boolean isValid(String s) {

  // 当字符串本来就是空的时候，我们可以快速返回true

  if (s == null || s.length() == 0) {

    return true;

  }

  // 当字符串长度为奇数的时候，不可能是一个有效的合法字符串

  if (s.length() % 2 == 1) {

    return false;

  }

  // 消除法的主要核心逻辑:

  int leftBraceNumber = 0;

  for (int i = 0; i < s.length(); i++) {

    // 取出字符

    char c = s.charAt(i);

    if (c == '(') {

      // 如果是'('，那么压栈

      leftBraceNumber++;

    } else if (c == ')') {

      // 如果是')'，那么就尝试弹栈

      if (leftBraceNumber <= 0) {

        // 如果弹栈失败，那么返回false

        return false;

      }

      --leftBraceNumber;

    }

  }

  return leftBraceNumber == 0;

}

```

**复杂度分析**：每个字符只入栈一次，出栈一次，所以时间复杂度为 O(N)，而空间复杂度为 O(1)，因为我们已经只用一个变量来记录栈中的内容。

【**小结**】经过前面的分析，现在我们可以将题目的特点做一下小结：

![](https://s0.lgstatic.com/i/image6/M00/0B/77/Cgp9HWA4ny2ASkpXAABGeRYQOyU298.png)



#### 队列 Queue

> 队列也是一种线性结构
>
> 相比数组，队列对应的操作是数组的子集
>
> 只能有一端（队尾）添加元素，只能从一端（队首）取出元素
>
> 队列是一种先进先出的数据结构（First In First Out FIFO）

#### 队列实现

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

#### leetcode的一些题目

[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

[用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

[用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

[ 最小栈](https://leetcode-cn.com/problems/min-stack/)

[设计一个支持增量操作的栈](https://leetcode-cn.com/problems/design-a-stack-with-increment-operation/)

[用栈操作构建数组](https://leetcode-cn.com/problems/build-an-array-with-stack-operations/)

[设计循环队列](https://leetcode-cn.com/problems/design-circular-queue/)

