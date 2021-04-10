#### 排序算法

> 让数据有序。排序算法中蕴含着重要的算法设计思想

#### 选择排序法

> 先把最小的拿出来，剩下的，再把最小的拿出来。每次**选择**还没处理的元素里**最小的元素**

基本思路

- 首先把数组中的第一个元素当做最小的元素，然后扫描后续的元素
- 如果后续的元素比第一个小，两个元素交换位置
- 重复以上两步骤

arr[i,n)未排序 arr[0,i) 已排序  （**循环不变量**）

arr[i,n) 中的最小值要放到arr[i]的位置

```javascript
const selectSort = (arr) => {
  // arr[0,i)是有序的，arr[i,n)是无序的
  for(let i = 0;i<arr.length;i++) {
    // 选择arr[i,n)中的最小值的索引
    let minIndex = i;
    for(let j = i；j<arr.length;j++) {
      if(arr[j]<arr[minIndex]) {
        minIndex = j;
      }
    }
    swap(arr, i, minIndex)
  }
}

const swap = (arr, i, j) => {
  let temp = arr[i];
  arr[i] = arr[j];
  arr[j] = temp;
}
```

#### 选择排序法的复杂度分析

> 1 + 2 + 3 + ...+ n
>
> = (1+n)*n / 2
>
> = 1/2 * n^2 + 1/2 *n
>
> O(n^2)

#### 插入排序法

> 例如扑克牌，每次拿到牌处理一张牌，把这张牌插入到前面已经排好序的牌中

基本思路

- 遍历数组定义索引i,然后在定义一个索引j=i
- 从j往前遍历比较大小，找到合适的位置放入

arr[0,i)已排好序；arr[i,n)未排序

把arr[i]放到合适的位置

**选择排序和插入排序**

- 选择排序经过几轮排序之后，已经排序好的元素已经是整个数组的顺序了
- 插入排序只是局部当中排好序的

```javascript
const insertSort = (arr) => {
  for(let i = 0;i<arr.length;i++) {
    // 将arr[i]插入到合适的位置
    for(let j = i;j-1>=0 && arr[j]<arr[j-1];j--) {
      swap(arr, j, j-1);
    }
  }
}

const swap = (arr, i, j) => {
  let temp = arr[i];
  arr[i] = arr[j];
  arr[j] = temp;
}
```

**小优化**

> 上面的代码每次需要与相邻的元素进行交换，实际放入元素的位置的后面的元素只是平移，所以我们有接下来的优化

```javascript
const insertSort2 = (arr) => {
	for(let i = 0;i<arr.length;i++) {
    let t = arr[i];
    let j;
    for(j = i;j-1>=0 && t<arr[j-1];j--) {
      arr[j] = arr[j-1];
    }
    arr[j] = t;
  }
}
```

**重要特性**

对于有序数组，插入排序的复杂度是O(n)。而选择排序没有这个特点。所以对于有序的或者部分有序的，使用插入排序的效率更高

