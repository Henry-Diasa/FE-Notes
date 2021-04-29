#### 快速排序（O(nlogn)）

> 快速排序就是每次寻找一个基准点，经过一次排序后基准点左侧的元素都是小于基准点，基准点的右侧是大于基准点的

第一种方式

> 定义两个数组left和right，遍历数组小于基准点的值放入left，大于基准点的值放入right，最后在对left和right进行快速排序

```js
var quickSort = function(arr) {
    if (arr.length <= 1) { return arr; }
    var pivotIndex = Math.floor(arr.length / 2);   //基准位置（理论上可任意选取）
    var pivot = arr.splice(pivotIndex, 1)[0];  //基准数
    var left = [];
    var right = [];
    for (var i = 0; i < arr.length; i++){
        if (arr[i] < pivot) {
            left.push(arr[i]);
        } else {
            right.push(arr[i]);
        }
    }
    return quickSort(left).concat([pivot], quickSort(right));  //链接左数组、基准数构成的数组、右数组
};

```

**缺点**

- 需要额外的空间，不是原地排序

第二种方式

> 原地排序，需要确定partition的位置

```js
function swap(arr, i, j) {
  let temp = arr[i];
  arr[i] = arr[j];
  arr[j] = temp;
}
function partition(arr, l, r) {
	// arr[l+1, j] < v ; arr[j+1,i] >= v
  let j = l;
  for(let i = j + 1; i<=r; i++) {
    if(arr[i] < arr[j]) {
      j++;
      swap(arr, i, j);
    }
  }
  swap(arr, l, j);
  return j;
}

function quicksort(arr, l, r) {
  if(l>=r) return;
  let p = partition(arr, l, r);
  quicksort(arr, l, p-1);
 	quicksort(arr, p+1, r); 
}
```

虽然第二种方式已经很好了，但是仍然存在一个问题，那就是如果**数组是有序**的（例如从小到大），每次partition之后，左侧的数组会是一个空数组，而右侧是n-1的元素所组成的数组，这就导致最终的复杂度退化为O(n^2)的复杂度。解决方式是引入**随机数**的概念

代码如下

```js
function swap(arr, i, j) {
  let temp = arr[i];
  arr[i] = arr[j];
  arr[j] = temp;
}
function partition(arr, l, r) {
  // 取[l,r]之间的随机数，然后和l的值进行交换
  let p = Math.floor(Math.randow() * [r - l + 1]) + l;
  swap(arr,l,p);
	// arr[l+1, j] < v ; arr[j+1,i] >= v
  let j = l;
  for(let i = j + 1; i<=r; i++) {
    if(arr[i] < arr[j]) {
      j++;
      swap(arr, i, j);
    }
  }
  swap(arr, l, j);
  return j;
}

function quicksort(arr, l, r) {
  if(l>=r) return;
  let p = partition(arr, l, r);
  quicksort(arr, l, p-1);
 	quicksort(arr, p+1, r); 
}
```

为什么加入随机数就可以解决这个问题呢？

- 为什么不每次取[l,r]的中间值呢？
- 随机数的时候也可能每次都取到最小的值啊

首先，如果每次取[l,r]中间值，那么必然会有一种特殊的示例来导致每次取得值都是最小值，从而导致复杂度还是O(n^2)

如果随机数每次都取到最小值，我们来算一下概率，第一次的概率是1/n,第二次的概率是1/(n-1),第三次的概率为1/(n-2)...如此算下去 最终的概率是 1/n! ,很明显这个概率是非常低的。

所以我们取随机值就可以肯定不能够有特定的实例来导致最终的复杂度退化。

但是，还有一个问题，那就是**数组元素都相同**的情况呢？还是会退化复杂度。解决方式就是**双路快排**

#### 双路快排

> partition的定义，arr[l+1,i-1] < v    arr[j+1,r] > v