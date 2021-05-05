#### 二分搜索树

> 每一个根节点的左子树都比根节点的值小，右子树都比根节点的值大

**实现二分搜索树**

```js
function Node(data, left, right) {
	this.data = data;
	this.left = left;
	this.right = right;
}

function BST() {
	this.root = null;
	this.insert = insert;
  this.preOrder = preOrder; // 前序
  this.inOrder = inOrder; // 中序
  this.postOrder = postOrder; // 后序
}
// 非递归实现
function insert(data) {
	let n = new Node(data, null, null);
	if(this.root == null) {
		this.root = n;
	}else{
    let current = this.root;
    let parent;
    while(true) {
      parent = current;
      if(data < current.data) {
        current = current.left;
        if(current == null) {
          parent.left = n;
          break;
        }
      }else{
        current = current.right;
        if(current == null) {
          parent.right = n;
          break;
        }
      }
    }
  }
}

// 递归实现
function insert(node, data) {
  if(data == node.data) {
    return;
  }else if(data < node.data && node.left == null) {
    node.left = new Node(data, null, null);
    return;
  }else if(data > node.data && node.right == null) {
    node.right = new Node(data, null, null);
    return;
  }
  
  if(data < node.data) {
    insert(node.left, data);
  }else{
    insert(node.right, data);
  }
}

function preOrder(node) {
  if(node!== null) {
    console.log(node.data);
    preOrder(node.left);
    preOrder(node.right);
  }
}
// 前序 非递归方式
function preOrderNR() {
  const arr = [this.root];
  while(arr.length > 0) {
    const cur = arr.pop();
    console.log(cur.data);
    // 先放右节点 然后左节点  出栈的时候 才会显示左 然后右
    if(cur.right!=null) {
      arr.push(cur.right)
    }
    if(cur.left!=null) {
      arr.push(cur.left)
    }
  }
}

function inOrder(node) {
  if(node!== null) {
    inOrder(node.left);
    console.log(node.data);
    inOrder(node.right);
  }
}

function postOrder(node) {
  if(node!== null) {
    postOrder(node.left);
    postOrder(node.right);
    console.log(node.data);
  }
}

// 层次遍历
function levelOrder() {
  const q = [];
  arr.push(this.root);
  while(q.length > 0) {
    const cur = q.shift();
    console.log(cur.data);
    if(cur.left!=null) {
      cur.push(cur.left);
    }
    if(cur.right!=null) {
      cur.push(cur.right);
    }
  }
}
// 获取最小值
function getMin(root) {
  let current = root;
  while(current.left!==null) {
    current = current.left;
  }
  return current.data;
}

// 获取最大值
function getMax() {
  let current = this.root;
  while(current.right!==null) {
    current = current.right;
  }
  return current.data;
}

// 查找给定值
function find(data) {
  let current = this.root;
  while(current!==null) {
    if(current.data = data) {
      return current
    }else if(data < current.data) {
      current = current.left
    }else{
      current = current.right;
    }
  }
  return null;
}

// 删除节点
function remove(data) {
  root = removeNode(this.root, data);
}

function removeNode(node, data) {
  if(node == null) {
    return null;
  }
  
  if(data == node.data) {
    // 没有子节点的节点
    if(node.left == null && node.right == null) {
      return null;
    }
    // 没有左子节点的节点
    if(node.left == null) {
      return node.right;
    }
    // 没有右子节点的节点
    if(node.right == null) {
      return node.left;
    }
    // 有两个子节点的节点 (找到右子树的最小值 互换)
    let tempNode = getMin(node.right);
    node.data = tempNode.data;
    node.right = removeNode(node.right, tempNode.data);
    return node;
  }else if(data < node.data) {
    node.left = removeNode(node.left, data);
    return node;
  }else {
    node.right = removeNode(node.right, data);
    return node;
  }
}
```

[唯一摩尔斯密码词](https://leetcode-cn.com/problems/unique-morse-code-words/)

