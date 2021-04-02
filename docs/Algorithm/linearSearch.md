#### 什么是线性查找法

也就是在线性的数据结构中查找元素的方法。例如在一个**数组**中查找**指定元素**的**索引**

代码演示

```javascript
const LinearSearch = (arr, target) => {
  for(let i = 0;i < arr.length;i++) {
    if(arr[i] === target) {
      return i;
    }
  }
  return -1;
}
```

#### 循环不变量

根据上面的例子来说

```javascript
const LinearSearch = (arr, target) => {
  for(let i = 0;i < arr.length;i++) {
    /* 
    	循环不变量
    	确认arr[i]是否是目标
    	arr[0...i-1]中没有找到目标
    */
    
    // 循环体：维持循环不变量
    if(arr[i] === target) {
      return i;
    }
  }
  /*
  	确认arr[i]不是目标
  	arr[0...i]中没有找到目标
  */
  return -1;
}
```

循环不变量就是每次循环，都会满足上面的条件

写出正确的代码

- 定义清楚循环不变量

- 维护循环不变量

- 定义清楚函数的功能

  > LinearSearch
  >
  > 输入：数组和目标元素
  >
  > 输出：目标元素所在的索引；若不存在，返回-1

#### 复杂度分析

> 表示算法的性能

- 线性查找法  O(n)

- 一个数组中的元素可以组成哪些数据对 O($n^2$) 

  ```javascript
  for(let i = 0;i<data.length;i++) {
  	for(let j = i+1;j<data.length;j++) {
  		// 获取一个数据对
  	}
  }
  ```

- 数字n的二进制位数 O(logn)

  ```javascript
  while(n){
  	n % 2 // n的二进制中的一位
  	n/=2;
  }
  ```

- 数字n的所有约数

  O(n)

  ```javascript
  
  for(let i = 1;i<=n;i++) {
  	if(n % i == 0) {
  		// i是n的一个约数
  	}
  }
  
  ```

  O($\sqrt{n}$)

  ```javascript
  for(let i = 1;i*i<=n;i++) {
  	if(n % i == 0) {
  		// i和n/i是n的两个约数
  	}
  }
  ```

- 长度为n的二进制数字 O($2^n$)

- 长度为n的数组的所有排列 O(n!)

- 判断数字n是否是偶数 O(1)

> O(1) < O(logn) < O($\sqrt{n}$) < O(n) < O(nlogn) < O($n^2$) < O($2^n$) < O(n!)

