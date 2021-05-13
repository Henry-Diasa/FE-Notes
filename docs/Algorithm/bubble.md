#### 冒泡排序

> 冒泡排序就是每次将值最大的元素排序到数组的最后。

```js
function bubble(arr) {
	for(let i = 0;i<arr.length - 1;i++) {
    // arr[n -i, n] 已排好序
    // 通过冒泡在arr[n-i-1]位置放上合适的元素
    for(let j = 0;j<arr.length - 1 - i;j++) {
      if(arr[j] > arr[j+1]) {
        swap(arr, j, j+1);
      }
    }
  }
}
```

**优化1**

> 如果在某一轮循环之后已经是排好序的了，接下来的排序就不需要了
>
> 通过一个变量控制。初始值为false，如果产生交换就置为true。如果最后变量的值为false，直接跳出循环

```js
function bubble(arr) {
	for(let i = 0;i<arr.length - 1;i++) {
    // arr[n -i, n] 已排好序
    // 通过冒泡在arr[n-i-1]位置放上合适的元素
    let isSwapped = false;
    for(let j = 0;j<arr.length - 1 - i;j++) {
      if(arr[j] > arr[j+1]) {
        swap(arr, j, j+1);
        isSwapped = true;
      }
    }
    if(!isSwapped) break;
  }
}
```

**优化2**

> 如果数组的一部分已经是排好序的，可以记住当前交换的最后索引，然后减少循环的轮数

```js
function bubble(arr) {
	for(let i = 0;i<arr.length - 1) {
    // arr[n -i, n] 已排好序
    // 通过冒泡在arr[n-i-1]位置放上合适的元素
    let lastSwappedIndex = 0;
    for(let j = 0;j<arr.length - 1 - i;j++) {
      if(arr[j] > arr[j+1]) {
        swap(arr, j, j+1);
        lastSwappedIndex = j + 1;
      }
    }
    // 已经有几个元素排好序了
    i = arr.length - lastSwappedIndex;
  }
}
```

#### 希尔排序

> 基本思想：让数组越来越有序
>
> 不能只处理相邻的逆序对
>
> 对元素间距为n/2的所有数组做插入排序
>
> 对元素间距为n/4的所有数组做插入排序
>
> 对元素间距为n/8的所有数组做插入排序
>
> ...
>
> 对元素间距为1的所有数组做插入排序

```js
function shellSort(data) {
  let h = data.length / 2;
  while(h>=1) {
    // 相当于一组
    for(let start = 0;start < h;start++) {
      for(let i= start+h;i<data.length;i+=h) {
        let t = data[i];
        let j;
        for(j = i;j-h>=h && t < data[j-h];j-=h) {
          data[j] = data[j-h];
        }
        data[j] = t;
      }
    }
    h/=2;
  }
}
```

