#### 数据结构脑图

[数据结构](https://naotu.baidu.com/file/b832f043e2ead159d584cca4efb19703?token=7a6a56eb2630548c)

#### 算法脑图

[算法](https://naotu.baidu.com/file/0a53d3a5343bd86375f348b2831d3610?token=5ab1de1c90d5f3ec)

#### map

[1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

[3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

#### 栈的题目

**判断字符串括号是否合法**

>
>
>【题目】字符串中只有字符'('和')'。合法字符串需要括号可以配对。比如：
>
>输入："()"
>
>输出：true
>
>解释：()，()()，(())是合法的。)(，()(，(()是非法的。
>
>请你实现一个函数，来判断给定的字符串是否合法。
>
>boolean isValid(String s);

```js
// 时间空间复杂度都为 O(n)
function isValid(s) {
	if(!s)return true
  if(s.length % 2 == 1) return false
  const stack = []
  for(let i = 0;i<s.length;i++) {
    const curr = s[i]
    if(curr == '(') {
      stack.push(curr)
    }else if(curr == ')') {
      if(stack.length == 0) {
        return false
      }
      stack.pop();
    }
  }
  return stack.length == 0
}

// 优化方式 使用变量替换stack  空间复杂度降为O(1)
function isValid(s) {
	if(!s)return true
  if(s.length % 2 == 1) return false
  let number = 0
  for(let i = 0;i<s.length;i++) {
    const curr = s[i]
    if(curr == '(') {
      number++
    }else if(curr == ')') {
      if(number <= 0) {
        return false
      }
     --number
    }
  }
  return number == 0
}

```

[大鱼吃小鱼](https://app.codility.com/c/run/trainingJ2N3G9-P27/)

> 【题目】在水中有许多鱼，可以认为这些鱼停放在 x 轴上。再给定两个数组 Size，Dir，Size[i] 表示第 i 条鱼的大小，Dir[i] 表示鱼的方向 （0 表示向左游，1 表示向右游）。这两个数组分别表示鱼的大小和游动的方向，并且两个数组的长度相等。鱼的行为符合以下几个条件:
>
> 所有的鱼都同时开始游动，每次按照鱼的方向，都游动一个单位距离；
>
> 当方向相对时，大鱼会吃掉小鱼；
>
> 鱼的大小都不一样。
>
> 输入：Size = [4, 2, 5, 3, 1], Dir = [1, 1, 0, 0, 0]
>
> 输出：3

```js
function solution(fishSize, fishDirection) {
  const fishNumber = fishSize.length
  if(fishNumber <=1) {
    return fishNumber
  }
  const left = 0
  const right = 1
  const stack = []
  for(let i = 0;i<fishNumber;i++) {
  	const curFishDirection = fishDirection[i]
  	const curFishSize = fishSize[i]
  	let hasEat = false
  	while(stack.length > 0 && fishDirection[stack[stack.length - 1]] == right && curFishDirection == left) {
  	  if(fishSize[stack[stack.length - 1]] > curFishSize) {
  	    hasEat = true
  	    break
  	  }
  	  stack.pop()
  	}
  	if(!hasEat) {
  	  stack.push(i)
  	}
  }
  
  return stack.length
}
```

[20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

[155. 最小栈](https://leetcode-cn.com/problems/min-stack/)

**找出数组中右边比我小的元素**

> 【题目】一个整数数组 A，找到每个元素：右边第一个比我小的下标位置，没有则用 -1 表示。
>
> 输入：[5, 2]
>
> 输出：[1, -1]
>
> 解释：因为元素 5 的右边离我最近且比我小的位置应该是 A[1]，最后一个元素 2 右边没有比 2 小的元素，所以应该输出 -1。

```js
// 递增栈
function findRightSmall(A) {
  // 结果数组
  const ans = []
  // 注意，栈中的元素记录的是下标
  const stack = []
  for(let i = 0;i<A.length;i++) {
    let x = A[i]
    // 每个元素都向左遍历栈中的元素完成消除动作
    while(stack.length && A[stack[stack.length-1]] > x) {
      // 小数 消除 大数  消除的时候，记录一下被谁消除了
      ans[stack[stack.length-1]] = i
      // 消除时候，值更大的需要从栈中消失
      stack.pop()
    }
    // 剩下的入栈
    stack.push(i)
  }
   // 栈中剩下的元素，由于没有人能消除他们，因此，只能将结果设置为-1。
  while(stack.length) {
    ans[stack[stack.length-1]] = -1
    stack.pop()
  }
  return ans
}

```

**字典序最小的k个数的子序列**

> 【**题目**】给定一个正整数数组和 k，要求依次取出 k 个数，输出其中数组的一个子序列，需要满足：1. **长度为 k**；2.**字典序最小**。
>
> 输入：nums = [3,5,2,6], k = 2
> 输出：[2,6]
>
> **解释**：在所有可能的解：{[3,5], [3,2], [3,6], [5,2], [5,6], [2,6]} 中，[2,6] 字典序最小。
>
> 所谓字典序就是，给定两个数组：x = [x1,x2,x3,x4]，y = [y1,y2,y3,y4]，如果 0 ≤ p < i，xp == yp 且 xi < yi，那么我们认为 x 的字典序小于 y。

```js
function findSmallSeq(nums, k) {
  const ans = []
  const stack = []
  for(let i = 0;i<nums.length;i++) {
    const x = nums[i]
    const left = nums.length - i
    // 注意我们想要提取出k个数，所以注意控制扔掉的数的个数
    while(stack.length && (stack.length + left > k) && stack[stack.length - 1] > x) {
         stack.pop() 
    }
    stack.push(x)
  }
  
  // 如果递增栈里面的数太多，那么我们只需要取出前k个就可以了。
  // 多余的栈中的元素需要扔掉。
  while(stack.length > k) {
    stack.pop()
  }
  
  // 把k个元素取出来，注意这里取的顺序!
  for(let i = k -1;i>=0;i--) {
    ans[i] = stack[stack.length-1]
    stack.pop()
  }
  return ans
}
```

[84. 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

![](https://s0.lgstatic.com/i/image6/M01/0B/7F/CioPOWA4q6qASB-UAADhj7uzOwg933.png)



#### 队列

[933. 最近的请求次数](https://leetcode-cn.com/problems/number-of-recent-calls/)

[637. 二叉树的层平均值](https://leetcode-cn.com/problems/average-of-levels-in-binary-tree/)

[429. N 叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

[1302. 层数最深叶子节点的和](https://leetcode-cn.com/problems/deepest-leaves-sum/)

[662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/)

[103. 二叉树的锯齿形层序遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

[107. 二叉树的层序遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

[559. N 叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/)

[622. 设计循环队列](https://leetcode-cn.com/problems/design-circular-queue/)

#### 双指针

[125. 验证回文串](https://leetcode-cn.com/problems/valid-palindrome/)

[283. 移动零](https://leetcode-cn.com/problems/move-zeroes/)

[11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

[15. 三数之和](https://leetcode-cn.com/problems/3sum/)

[26. 删除有序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

[76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)


#### 链表

[146. LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)

[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

[24. 两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

[141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

[142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

[21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

[237. 删除链表中的节点](https://leetcode-cn.com/problems/delete-node-in-a-linked-list/)

[2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

[83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

#### 集合

[349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

#### 动态规划



#### 其他

[70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

[66. 加一](https://leetcode-cn.com/problems/plus-one/)

[189. 旋转数组](https://leetcode-cn.com/problems/rotate-array/)

