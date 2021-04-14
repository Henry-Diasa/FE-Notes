#### 定义链表

> 链表是由一组节点组成的集合。每个节点都使用一个对象的引用指向它的后继。指向另一 个节点的引用叫做链
>
> 数组元素靠它们的位置进行引用，链表元素则是靠相互之间的关系进行引用
>
> 要标识出链表的起始节点却有点麻烦，许多链表的实现都在链表最前面有一个特殊节 点，叫做头节点

#### 设计一个基于对象的链表

```js
// Node类
function Node(element) {
  this.element = element;
  this.next = null;
}

// LinkedList
function LList() {
  this.head = new Node('head'); // 虚拟头结点
  this.find = find; // 查找元素
  this.insert = insert; // 插入元素
  this.findPrevious = findPrevious; // 查找前一个节点
  this.remove = remove; // 删除元素
  this.display = display; // 输出元素链表节点的值
}

function find(item) {
  let currNode = this.head;
  while(currNode.element!=item) {
    // element值不相等一直找下去
    currNode = currNode.next;
  }
  return currNode;
}

function insert(newElement, item) {
  let newNode = new Node(newElement);
  let current = this.find(item);
  newNode.next = current.next;
  current.next = newNode;
}

function display() {
  let currNode = this.head;
  while(currNode.next!==null) {
    console.log(currNode.next.element);
    currNode = currNode.next;
  }
}

function findPrevious(item) {
  let currNode = this.head;
  while(currNode.next!==null && currNode.next.element!==item) {
    currNode = currNode.next;
  }
  return currNode;
}

function remove(item) {
  let prevNode = findPrevious(item);
  if(prevNode.next!==null) {
    prevNode.next = prevNode.next.next;
  }
}
```

#### 双向链表

> 尽管从链表的头节点遍历到尾节点很简单，但反过来，从后向前遍历则没那么简单。通过 给 Node 对象增加一个属性，该属性存储指向前驱节点的链接，这样就容易多了

```js
// Node类
function Node(element) {
  this.element = element;
  this.next = null;
  this.previous = null; // 前置节点
}

function insert(newElement, item) {
  let newNode = new Node(newElement);
  let current = this.find(item);
  newNode.next = current.next;
  newNode.previous = current;
  current.next = newNode;
  newNode.next.previous = newNode;
}

function remove(item) {
  let currNode = this.find(item);
  if(currNode.next!==null) {
    currNode.previous.next = currNode.next;
    currNode.next.previous = currNode.previous;
    currNode.next = null;
    currNode.previous = null;
  }
}

function findLast() {
  let currNode = this.head;
  while (!(currNode.next == null)) {
    currNode = currNode.next;
  }
  return currNode;
}

function dispReverse() {
  let currNode = this.head;
  currNode = this.findLast();
  while (!(currNode.previous == null)) {
    console.log(currNode.element);
    currNode = currNode.previous; 
  }
}

```

#### 循环链表

> 循环链表和单向链表相似，节点类型都是一样的。唯一的区别是，在创建循环链表时，让其头节点的 next 属性指向它本身

```js
function LList() {
  this.head = new Node("head"); 
  this.head.next = this.head; 
  this.find = find;
  this.insert = insert; 
  this.display = display; 
  this.findPrevious = findPrevious; 
  this.remove = remove;
}
```

只需要修改一处，就将单向链表变成了循环链表。但是其他一些方法需要修改才能工作正 常。比如，display() 就需要修改，原来的方式在循环链表里会陷入死循环。while 循环的 循环条件需要修改，需要检查头节点，当循环到头节点时退出循环。

```js
function display() {
	let currNode = this.head;
	while (!(currNode.next == null) &&
!(currNode.next.element == "head")) {
    console.log(currNode.next.element);
		currNode = currNode.next;
} }
```

