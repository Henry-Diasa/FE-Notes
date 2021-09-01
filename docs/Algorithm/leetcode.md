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

[117. 填充每个节点的下一个右侧节点指针 II](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii/)

[other](https://github.com/lagoueduCol/Algorithm-Dryad/blob/main/02.Queue/README.md)


[637. 二叉树的层平均值](https://leetcode-cn.com/problems/average-of-levels-in-binary-tree/)

[429. N 叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

[1302. 层数最深叶子节点的和](https://leetcode-cn.com/problems/deepest-leaves-sum/)

[662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/)

[103. 二叉树的锯齿形层序遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

[107. 二叉树的层序遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

[559. N 叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/)

[622. 设计循环队列](https://leetcode-cn.com/problems/design-circular-queue/)

[1696. 跳跃游戏 VI](https://leetcode-cn.com/problems/jump-game-vi/)

![](https://s0.lgstatic.com/i/image6/M00/11/10/CioPOWA_TLCATeR6AAFTfMBlaiw858.png)

#### 双指针

[125. 验证回文串](https://leetcode-cn.com/problems/valid-palindrome/)

[283. 移动零](https://leetcode-cn.com/problems/move-zeroes/)

[11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

[15. 三数之和](https://leetcode-cn.com/problems/3sum/)

[26. 删除有序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

[76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

[3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

[713. 乘积小于K的子数组](https://leetcode-cn.com/problems/subarray-product-less-than-k/)

[567. 字符串的排列](https://leetcode-cn.com/problems/permutation-in-string/)

[76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

[392. 判断子序列](https://leetcode-cn.com/problems/is-subsequence/)

[443. 压缩字符串](https://leetcode-cn.com/problems/string-compression/)

[42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

#### 回溯

[17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

[78. 子集](https://leetcode-cn.com/problems/subsets/)

[46. 全排列](https://leetcode-cn.com/problems/permutations/)

[90. 子集 II](https://leetcode-cn.com/problems/subsets-ii/)

[47. 全排列 II](https://leetcode-cn.com/problems/permutations-ii/)


#### 链表

[146. LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)

[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

[203. 移除链表元素](https://leetcode-cn.com/problems/remove-linked-list-elements/)

[24. 两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

[141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

[142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

[相关链接](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=685#/detail/pc?id=6694)

[21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

[237. 删除链表中的节点](https://leetcode-cn.com/problems/delete-node-in-a-linked-list/)

[2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

[83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

[82. 删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

[23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

[25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

[逆向反转k](https://github.com/lagoueduCol/Algorithm-Dryad/blob/main/05.LinkedList/25.%E9%80%86%E5%90%91k%E4%B8%AA%E4%B8%80%E7%BB%84%E7%BF%BB%E8%BD%AC%E9%93%BE%E8%A1%A8.java)

[19. 删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

**拆分链表**

> 【题目】给定一个链表，需要把链表从中间拆分成长度相等的两半（如果链表长度为奇数，那么拆分之后，前半部分长度更长一点）。
>
> 输入：[1->2->3->4->5]
>
> 输出：[1->2->3, 4->5]

```java
class Solution {

    private ListNode findMiddleNode(ListNode head) {

        // 注意这里转化为带假头的链表，免去了空链表的判断

        ListNode dummy = new ListNode();

        dummy.next = head;

        // 注意，假头并不算是链表的一部分，所以这里是从head开始走

        ListNode s2 = head;

        ListNode s1 = head;

        // dummy就是head的前驱，所以pre要指向dummy.

        ListNode pre = dummy;

        // 两个指针开始同时走

        // 因为s2指针每次都要走两步，所以判空需要这样判断。

        while (s2 != null && s2.next != null) {

            pre = s1;

            s1 = s1.next;

            s2 = s2.next.next;

        }

        // 当有偶数个结点的时候，s2是空指针，

        // 此时，s1位于后半部分指针的头部，因此需要返回s1的前驱。

        // 当有奇数个结点的时候，s2是最后一个结点，

        // 此时s1指针位于前半部分的最后，直接返回s1即可。

        return s2 != null ? s1 : pre;

    }

    public ListNode[] split(ListNode head) {

        // 这里获取了链表的中间结点

        ListNode mid = findMiddleNode(head);

        // 拿到链表的中间结点之后，可以得到链表的后半部分的开头

        ListNode back = mid.next;

        // 把链表拆分为两半

        mid.next = null;

        // 返回两个链表的头部

        return new ListNode[]{head, back};

    }

}

```

[143. 重排链表](https://leetcode-cn.com/problems/reorder-list/)

![](https://s0.lgstatic.com/i/image6/M01/3E/39/CioPOWCYmIWAFXX4AASqbS526bc322.png)

#### 集合

[349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

#### 动态规划

[152. 乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)

[64. 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)



####  堆

[剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

[347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

[692. 前K个高频单词](https://leetcode-cn.com/problems/top-k-frequent-words/)

[973. 最接近原点的 K 个点](https://leetcode-cn.com/problems/k-closest-points-to-origin/)

[23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

[373. 查找和最小的K对数字](https://leetcode-cn.com/problems/find-k-pairs-with-smallest-sums/)

[1642. 可以到达的最远建筑](https://leetcode-cn.com/problems/furthest-building-you-can-reach/)

[1705. 吃苹果的最大数目](https://leetcode-cn.com/problems/maximum-number-of-eaten-apples/)

[871. 最低加油次数](https://leetcode-cn.com/problems/minimum-number-of-refueling-stops/)

[743. 网络延迟时间](https://leetcode-cn.com/problems/network-delay-time/)

![](https://s0.lgstatic.com/i/image6/M00/13/4A/Cgp9HWBB-QCAcvk-AADd5wNIZG0008.png)

#### 树

[144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

[98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

[100. 相同的树](https://leetcode-cn.com/problems/same-tree/)

[572. 另一棵树的子树](https://leetcode-cn.com/problems/subtree-of-another-tree/)

[113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

[94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

[783. 二叉搜索树节点最小距离](https://leetcode-cn.com/problems/minimum-distance-between-bst-nodes/)

[99. 恢复二叉搜索树](https://leetcode-cn.com/problems/recover-binary-search-tree/)

[450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

[701. 二叉搜索树中的插入操作](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)

[700. 二叉搜索树中的搜索](https://leetcode-cn.com/problems/search-in-a-binary-search-tree/)

[145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

[236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

[105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

[148. 排序链表](https://leetcode-cn.com/problems/sort-list/)

[104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

[543. 二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/)

#### 排序

[剑指 Offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

[315. 计算右侧小于当前元素的个数](https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self/)

[4. 寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

[88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

[75. 颜色分类](https://leetcode-cn.com/problems/sort-colors/)

[136. 只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

[240. 搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)

**第K小的数**

> 【**题目**】给定一个数组，请找出第 k 小的数（最小的数为第 1 小）。
>
> 输入：A = [2, 4, 1, 5, 3], k = 3
>
> 输出：3

```java
void swap(int[] A, int i, int j) {

  int t = A[i];

  A[i] = A[j];

  A[j] = t;

}

// 注意这里区间为[b, e), k也是从0开始算的

int kth(int[] A, int b, int e, int k) {

  // 如果为空

  if (b >= e) {

    return 0;

  }

  // 如果只有一个元素

  if (b + 1 >= e) {

    return A[b];

  }

  // 进行三路切分

  final int x = A[b + ((e - b) >> 1)];

  int i = b;

  int l = b;

  int r = e - 1;

  while (i <= r) {

    if (A[i] < x)

      swap(A, l++, i++);

    else if (A[i] == x)

      i++;

    else

      swap(A, r--, i);

  }

  // 分别拿到三段的长度

  final int lcnt = l - b;

  final int mcnt = i - l;

  // 如果第k个数落在左区间

  if (k < lcnt)

    return kth(A, b, l, k);

  // 如果第k个数落在右区间

  if (k >= (lcnt + mcnt))

    return kth(A, i, e, k - lcnt - mcnt);

  // 如果第k个数落在中间，那么直接返回x

  return x;

}

int kthNumber(int[] A, int n, int k) { 

  return kth(A, 0, n, k - 1);

}

```

![](https://s0.lgstatic.com/i/image6/M00/26/50/Cgp9HWBa166AD9hdAAKp0wZR38g493.png)

**二分排序**

```
function binarySearch(A, target) {
  if(!A || A.length==0) {
    return false
  }
  // 首先设定初始区间，这里我们采用开闭原则[l, r)
  let l = 0, r = A.length
  // 循环结束的判断条件是整个区间为空区间，那么运行结束。
  // 我们使用的是开闭原则来表示一个区间，所以当l < r的时候
  // 我们要查找的区间还不是一个空区间。
  while(l<r){
    let m = l + (r - l) / 2
    if(A[m] == target) {
     return true
    }
    
    if(A[m] < target) {
     l = m + 1
    }else{
     r = m
    }
  }
  
  return false
}
```

**有序数组中最左边的元素**

> 【**题目**】给定一个有序数组，返回指定元素在数组的最左边的位置
>
> 输入：A = [1, 2, 2, 2, 2, 3, 3], target = 2
>
> 输出：1
>
> 解释：第一个出现的 2 位于下标 1，是从左往右看时，第一个出现 2 的位置。

```
int lowerBound(long[] A, int n, long target) {

  int l = 0, r = n;

  while (l < r) {

    final int m = l + ((r - l) >> 1);

    if (A[m] < target) {

      l = m + 1;

    } else {

      r = m;

    }

  }

  return l;

}

```

**写一个函数 upperBound 寻找数组中给定元素的上界**

> 上界是刚好比 target 大的那个元素的位置。比如 A = [1, 1, 100, 100]，target = 1，那么 upperBound 应该返回下标 2

```
int upperBound(long[] A, int n, long target) {

  int l = 0, r = n;

  while (l < r) {

    final int m = l + ((r - l) >> 1);

    if (A[m] <= target) {

      l = m + 1;

    } else {

      r = m;

    }

  }

  return l;

}

```

[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

[35. 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

[852. 山脉数组的峰顶索引](https://leetcode-cn.com/problems/peak-index-in-a-mountain-array/)

**给定一个有序数组，找出数组中下标与值相等的那些数。** [链接](https://hub.fastgit.org/lagoueduCol/Algorithm-Dryad/blob/main/09.BinarySearch/69.%E6%95%B0%E7%BB%84%E4%B8%AD%E6%95%B0%E5%80%BC%E5%92%8C%E4%B8%8B%E6%A0%87%E7%9B%B8%E7%AD%89%E7%9A%84%E5%85%83%E7%B4%A0.java?fileGuid=xxQTRXtVcqtHK6j8)

**给定一个有序数组，找出数组中下标与值相等的数的范围。**[链接](https://hub.fastgit.org/lagoueduCol/Algorithm-Dryad/blob/main/09.BinarySearch/69.%E6%95%B0%E7%BB%84%E4%B8%AD%E6%95%B0%E5%80%BC%E5%92%8C%E4%B8%8B%E6%A0%87%E7%9B%B8%E7%AD%89%E7%9A%84%E5%85%83%E7%B4%A0.2.java?fileGuid=xxQTRXtVcqtHK6j8)

[209. 长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

[76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

[33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)



**冒泡排序**

```
function swap(A, i, j) {
  let temp = A[i]
  A[i] = A[j]
  A[j] = temp
}
function bubbleSort(arr) {
  // n - 1轮循环
  for(let i = 0;i<arr.length-1；i++){
  	for(let j = 0;j<arr.length - 1 - i;j++) {
  	  // 保证j+1有意义 所有循环到length - 1
  	  // - i 是因为每循环一次 就可以排除掉最大的那一个
  	  if(arr[j] > arr[j+1]) {
  	  	swap(arr, j, j+1)
  	  }
  	}
  }
}
```

**选择排序**

```
function selectSort(arr) {
	// n - 1 轮
	for(let i = 0;i<arr.length-1;i++) {
		let min = arr[i]
		let minIndex = i
		// 要保证可以选择到最后一个元素
		for(let j = i;j<arr.length;j++) {
			if(arr[j]<min) {
			 	min = arr[j]
			 	minIndex = j
			}
		}
		swap(arr,i,minIndex)
	}
}
```

**插入排序**

```
function insertSort(arr) {
	// 插入排序 从第二项开始插入
	for(let i = 1;i<arr.length;i++) {
		let insertVal = arr[i]
		let j = i-1
		while(j>=0 && arr[j] > insertVal) {
		  // 插入的值 小于当前的值 所有元素向后移动
			arr[j+1] = arr[j]
			j--
		}
		arr[j+1] = insertVal
	}
}
```

**快速排序**

```
function quickSort(arr, start, end) {
	let low = start
	let high = end
	let temp = arr[low]
	
	while(low<high) {
		while(low<high && arr[high]>=temp) {
			high--
		}
		arr[low] = arr[high]
		
		while(low<high && arr[low] <= temp) {
			low++
		}
		arr[high] = arr[low]
	}
	arr[high] = temp
	
	if(start<low -1) {
		quickSort(arr, start, low-1)
	}
	if(low+1<end) {
		quickSort(arr, low+1, end)
	}
	return
}
```

**归并排序**

```
function mergeSort(arr) {
 let temp = []
 // 分
 sort(arr, 0, arr.length - 1, temp)
}

function sort(arr, left, right, temp) {
	let mid = (left + right) / 2
	if(left < right) {
		sort(arr, left, mid, temp)
		sort(arr, mid+1, right, temp)
		merge(arr,left,right,temp)
	}
}

function merge(arr, left, right, temp) {
	let mid = (left + right) / 2
	let l = left
	let r = mid+1
	let index = 0
	while(l<=mid && r<=right) {
		if(arr[l]<arr[r]) {
			temp[index++] = arr[l++]
		}else{
			temp[index++] = arr[r++]
		}
	}
	
	while(l<=mid){
		temp[index++] = arr[l++]
	}
	
	while(r<=right) {
		temp[index++] = arr[r++]
	}
	
	index = 0
	while(left<=right) {
		arr[left++] = temp[index++]
	}
}
```



#### 其他

[70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

[66. 加一](https://leetcode-cn.com/problems/plus-one/)

[189. 旋转数组](https://leetcode-cn.com/problems/rotate-array/)

