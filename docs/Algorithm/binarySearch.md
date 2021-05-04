#### 二分查找法

> 针对有序的数组来说，每次查找中间的位置的元素，如果找到了目标值，直接返回，如果比目标值小的话，则从中间位置左侧的数组来查找，否则去中间位置右侧的数组来查找

**递归方式**

```js
function binarySearch(arr,l,r,target) {
  if(l>r) return -1;
  let mid = l + (r - l) / 2;
  if(arr[mid] == target) {
    return mid;
  }else if(arr[mid] < target){
    return binarySearch(arr, mid+1,r,target);
  }else {
    return binarySearch(arr,l, mid-1, target);
  }
}
```

**非递归方式**

```js
function binarySearch(arr, target) {
	let l = 0, r = data.length -1;
  while(l<=r) {
    let mid = l + (r - l) / 2;
    if(data[mid] == target) {
      return mid;
    }
    if(data[mid] < target) {
      l = mid + 1;
    }else{
      r = mid - 1;
    }
  }
  return -1;
}
```

[ 爱吃香蕉的珂珂](https://leetcode-cn.com/problems/koko-eating-bananas/)

[在 D 天内送达包裹的能力](https://leetcode-cn.com/problems/capacity-to-ship-packages-within-d-days/)

