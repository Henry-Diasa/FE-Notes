#### 排序算法

> 让数据有序。排序算法中蕴含着重要的算法设计思想

#### 选择排序法

> 先把最小的拿出来，剩下的，再把最小的拿出来。每次**选择**还没处理的元素里**最小的元素**

基本思路

- 首先把数组中的第一个元素当做最小的元素，然后扫描后续的元素
- 如果后续的元素比第一个小，两个元素交换位置
- 重复以上两步骤

arr[i,n)未排序 arr[0,i) 已排序  （**循环不变量**）

arr[i,n)中的最小值要放到arr[i]的位置

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



