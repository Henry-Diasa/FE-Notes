#### 二叉堆

> 二叉堆是一颗完全二叉树（**把元素顺序排列成树的形状**）
>
> 每个**父节点**的值都**大于**左右**子节点**的值 （大堆）
>
> 每个**父节点**的值都**小于**左右**子节点**的值 （小堆）
>
> 同时我们可以使用一个数组来表示一个堆
>
> 位置**i**的节点的父节点**i/2**,左节点为**2 * i**,右节点为**2* i + 1** (堆的第一个节点为数组的第1项)
>
> 位置**i**的节点的父节点**(i-1)/2**,左节点为**2 * i + 1 **,右节点为**2* i + 2** (堆的第一个节点为数组的第0项)

```js
function swap(arr,i,j) {
  let t = arr[i];
  arr[i] = arr[j];
  arr[j] = t;
}
class MaxHeap {
	constructor(capacity) {
		this.data = new Array(capacity)
	}
  size() {
    return this.data.length;
  }
  isEmpty() {
    return this.data.length === 0;
  }
  parent(i) {
    if(i == 0) {
      return new Error();
    }
    return (i - 1) / 2
  }
  leftChild(i) {
    return i * 2 + 1;
  }
  rightChild(i) {
    return i * 2 + 2;
  }
  add(e) { // O(logn)
    this.data.push(e);
    // 添加完元素 要进行上浮 来保持最大堆的性质
    siftUp(this.data.lenth - 1);
  }
  siftUp(k) {
    // 父节点的值小于当前节点
    while(k > 0 && data[this.parent(k)] < data[k]) {
      swap(this.data, k,this.parent(k));
      k = this.parent(k);
    }
  }
  findMax() {
    if(this.data.length === 0) {
      return new Error();
    }
    return this.data[0]
  }
  extractMax() { // O(logn)
    let ret = findMax();
    swap(this.data,0,this.data.length - 1);
    this.data.pop();
    // 下沉 需要找到子元素中的最大值进行互换
    siftDown(0);
    return ret;
  }
  siftDown(k) {
    while(this.leftChild(k) < this.data.length) {
      let j = this.leftChild(k);
      // 右孩子存在 并且右孩子的值大于左孩子的值
      if(j + 1 < this.data.length && this.data[j + 1] > this.data[j]) {
        j++;
      }
      if(this.data[k] >= this.data[j]) {
        break;
      }
      swap(this.data,k,j);
      k = j;
    }
  }
  replace(e) {
    // 取出堆中最大的值，并且替换成e
    let ret = this.findMax();
    this.data[0] = e;
    this.siftDown(0);
    return ret;
  }
  heapify(arr) {
    // 将任意数组 转化成堆 
    // 可以从最后一个非叶子节点开始向上逐个下沉
    for(let i = this.parent(arr.length - 1);i>=0;i--) {
      this.siftDown(i);
    }
  }
}
```

[剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

[数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

