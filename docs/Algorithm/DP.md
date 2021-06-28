### 动态规划模板

> 重叠子问题、最优子结构、状态转移方程就是动态规划三要素

```js
// 明确base case -> 明确状态 -> 明确选择 -> 定义dp数组 / 函数的含义
// 初始化 base case
dp[0][0][...] = base
// 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```

### 例题

#### 斐波那契数列

**暴力递归**

```javascript
function fib(n) {
	if(n ==1 || n == 2) return 1;
  return fib(n-1) + fib(n-2);
}
```

缺点：时间复杂度高，二叉树的节点个数大概为O(2^n), 每个子问题是`f(n - 1) + f(n - 2)` 一个加法操作，时间为 O(1)。导致低效的原因是存在很多重复计算的问题（重叠子问题）

**带备忘录（缓存）的递归解法**

```js
function fib(n) {
	if(n<1) return 0;
  let memo = new Array(n + 1).fill(0)
  return helper(memo, n)
}

function helper(memo, n) {
  // base case
  if(n == 1 || n == 2) return 1;
  // 缓存
  if(memo[n]!=0) return memo[n];
  memo[n] = helper(memo, n-1) + helper(memo, n-2)
  return memo[n]
}
```

由于本算法不存在冗余计算，子问题就是 `f(1)`, `f(2)`, `f(3)` ... `f(20)`，数量和输入规模 n = 20 成正比，所以子问题个数为 O(n)

**dp数组的迭代解法**

```
function fib(n) {

}
```



