#### 数据结构

> 数据结构研究的是数据如何在计算机中进行组织和存储，使得我们可以高效的获取数据或者修改数据。在内存世界的**增删改查**

#### 栈 Stack

> 栈也是一种线性结构
>
> 相比数组，栈对应的操作是数组的子集
>
> 只能从一端添加元素，也只能从一端取出元素
>
> 这一端称为栈顶
>
> 栈是一种后进先出的数据结构 （Last In First Out）

#### 栈的应用

- 无处不在的Undo操作（撤销）- 编辑器
- 程序调用的系统栈 - 操作系统
- 括号匹配 - 编译器

#### 栈的实现

```javascript
class ArrayStack {
  constructor() {
    this.data = [];
  },
  push(e) {
    this.data.push(e);
  },
  pop() {
		this.data.pop();
  },
  peek() {
    return this.data[this.data.length - 1];
  },
  size() {
    return this.data.length;
  },
  isEmpty() {
    return this.data.length === 0;
  },
  toString() {
    let result =  '';
    for(let i = 0; i < this.data.length; i++) {

      let sign = i === this.data.length-1 ? '' : ',';

      result += this.data[i] + sign 
    }
    return result;
  }
  
}
```

#### 栈的例题分析

**例1：判断字符串括号是否合法**

> 【题目】字符串中只有字符'('和')'。合法字符串需要括号可以配对。比如：
>
> 输入："()"
>
> 输出：true
>
> 解释：()，()()，(())是合法的。)(，()(，(()是非法的。
>
> 请你实现一个函数，来判断给定的字符串是否合法

首先，分析题目的时候，要特别注意以下 4 点，归纳为“四步分析法”。

- 模拟：模拟题目的运行。

- 规律：尝试总结出题目的一般规律和特点。

- 匹配：找到符合这些特点的数据结构与算法。

- 边界：考虑特殊情况。

通过模拟可以得出如下规律

- 每个左括号'('或者右括号')'都完成配对，才是合法的
- 配对可以通过**消除法**来消掉合法的括号，如果最后没有任何字符了，那么就是合法字符串
- 奇数长度的字符串总是非法的。
- 同时可以看出每次消除的时候都是把最近的匹配的一项消除掉。也就是满足栈的特点

最后还需要考虑一些边界情况

- 字符串为空
- 字符串只有 1 个或者奇数个
- 字符串是"(((())))"嵌套很多层的是否可以处理

最后可以写出如下代码

```java
boolean isValid(String s) {

  // 当字符串本来就是空的时候，我们可以快速返回true

  if (s == null || s.length() == 0) {

    return true;

  }

  // 当字符串长度为奇数的时候，不可能是一个有效的合法字符串

  if (s.length() % 2 == 1) {

    return false;

  }

  // 消除法的主要核心逻辑: 

  Stack<Character> t = new Stack<Character>();

  for (int i = 0; i < s.length(); i++) {

    // 取出字符

    char c = s.charAt(i);

    if (c == '(') {

      // 如果是'('，那么压栈

      t.push(c);

    } else if (c == ')') {

      // 如果是')'，那么就尝试弹栈

      if (t.empty()) {

        // 如果弹栈失败，那么返回false

        return false;

      }

      t.pop();

    }



  return t.empty();

}

```

**复杂度分析**：每个字符只入栈一次，出栈一次，所以时间复杂度为 O(N)，而空间复杂度为 O(N)，因为最差情况下可能会把整个字符串都入栈。

**1. 深度扩展**

如果仔细观察，你会发现，栈中存放的元素是一样的。全部都是左括号'('，除此之外，再也没有别的元素，优化方法如下。

**栈中元素都相同时，实际上没有必要使用栈，只需要记录栈中元素个数**

```java
public boolean isValid(String s) {

  // 当字符串本来就是空的时候，我们可以快速返回true

  if (s == null || s.length() == 0) {

    return true;

  }

  // 当字符串长度为奇数的时候，不可能是一个有效的合法字符串

  if (s.length() % 2 == 1) {

    return false;

  }

  // 消除法的主要核心逻辑:

  int leftBraceNumber = 0;

  for (int i = 0; i < s.length(); i++) {

    // 取出字符

    char c = s.charAt(i);

    if (c == '(') {

      // 如果是'('，那么压栈

      leftBraceNumber++;

    } else if (c == ')') {

      // 如果是')'，那么就尝试弹栈

      if (leftBraceNumber <= 0) {

        // 如果弹栈失败，那么返回false

        return false;

      }

      --leftBraceNumber;

    }

  }

  return leftBraceNumber == 0;

}

```

**复杂度分析**：每个字符只入栈一次，出栈一次，所以时间复杂度为 O(N)，而空间复杂度为 O(1)，因为我们已经只用一个变量来记录栈中的内容。

【**小结**】经过前面的分析，现在我们可以将题目的特点做一下小结：

![](https://s0.lgstatic.com/i/image6/M00/0B/77/Cgp9HWA4ny2ASkpXAABGeRYQOyU298.png)

**2. 广度扩展**

接下来再来看看如何进行广度扩展。观察题目可以发现，栈中只存放了一个维度的信息：左括号'('和右括号')'。如果**栈中的内容变得更加丰富**一点，就可以得到下面这道扩展题。

【**题目扩展**】给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合
2. 左括号必须以正确的顺序闭合
3. 注意空字符串可被认为是有效字符串

```java
/*
 * 测试平台链接：https://leetcode-cn.com/problems/valid-parentheses/
 * [20] 有效的括号
 */

class Solution {
    public boolean isValid(String s) {
        if (s == null || s.length() == 0) {
            return true;
        }
        if (s.length() % 2 == 1) {
            return false;
        }
        Stack<Character> t = new Stack<Character>();

        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c == '{' || c == '(' || c == '[') {
                t.push(c);
            } else if (c == '}') {
                if (t.empty() || t.peek() != '{') {
                    return false;
                }
                t.pop();
            } else if (c == ']') {
                if (t.empty() || t.peek() != '[') {
                    return false;
                }
                t.pop();
            } else if (c == ')') {
                if (t.empty() || t.peek() != '(') {
                    return false;
                }
                t.pop();
            } else {
                return false;
            }
        }

        return t.empty();
    }
}


```

**【小结**】接下来，我们对拓展题目进行总结，希望你从中**提炼出经验**，以后再遇到相似的题目能够轻松应对。

对于栈的使用，除了知道“后进先出”这个规律，我们还可以帮它长出一些叶子来，如下图所示

![](https://s0.lgstatic.com/i/image6/M00/0B/74/CioPOWA4nzyAJYfYAABDA_sAa3Q037.png)

因此，以后你在看到题目中类似**配对**、**消除**之类的动作时，可以采用**栈**来操作

**例 2：大鱼吃小鱼**

【**题目**】在水中有许多鱼，可以认为这些鱼停放在 x 轴上。再给定两个数组 Size，Dir，Size[i] 表示第 i 条鱼的大小，Dir[i] 表示鱼的方向 （0 表示向左游，1 表示向右游）。这两个数组分别表示鱼的大小和游动的方向，并且两个数组的长度相等。鱼的行为符合以下几个条件:

1. 所有的鱼都同时开始游动，每次按照鱼的方向，都游动一个单位距离；
2. 当方向相对时，大鱼会吃掉小鱼；
3. 鱼的大小都不一样。

输入：Size = [4, 2, 5, 3, 1], Dir = [1, 1, 0, 0, 0]

输出：3

请完成以下接口来计算还剩下几条鱼？

题目的示意图如下所示：

![](https://s0.lgstatic.com/i/image6/M00/0B/74/CioPOWA4n3uAM9nhAACmI5boRa0503.gif)

【**分析**】对于这道题而言，大鱼吃掉小鱼的时候，可以认为是一种**消除**行为。只不过与括号匹配时的行为不一样：

- 括号匹配是会**同时**把左括号与右括号消除掉；
- 大鱼吃小鱼，**只会把小鱼**消除掉。

**1. 模拟**

首先我们以如下示例进行演示：

复制代码

```
Size = [4, 3, 2, 1 5], Dir = [0, 1, 0, 0, 0]
```

![5.gif](https://s0.lgstatic.com/i/image6/M01/0B/74/CioPOWA4n5yAIhjtAAXAzjrqmCE807.gif)

*注意：当鱼的游动方向相同，或者相反时，并不会相遇，此时大鱼不能吃掉小鱼。*

**2. 规律**

通过模拟，可以发现如下规律:

- 如果两条鱼**相对而游时，那么较小的鱼会被吃掉；**
- **其他情况**没有鱼被吃掉。

**3. 匹配**

我们发现，下面活下来的鱼的行为（上图红框部分）就是一个**栈**。每当有新的鱼要进来的时候，就会与栈顶的鱼进行比较。那么我们匹配到的算法就是栈了。

**4. 边界**

在正式开始求解之前，我们还是想一想两种边界：

- 所有的鱼都朝着一个方向游；
- 一条鱼吃掉了其他的所有鱼。

```java
// https://app.codility.com/programmers/lessons/7-stacks_and_queues/fish/start/
int solution(int[] fishSize, int[] fishDirection) {

  // 首先拿到鱼的数量

  // 如果鱼的数量小于等于1，那么直接返回鱼的数量

  final int fishNumber = fishSize.length;

  if (fishNumber <= 1) {

    return fishNumber;

  }

  // 0表示鱼向左游

  final int left = 0;

  // 1表示鱼向右游

  final int right = 1;

  Stack<Integer> t = new Stack();

  for (int i = 0; i < fishNumber; i++) {

    // 当前鱼的情况：1，游动的方向；2，大小

    final int curFishDirection = fishDirection[i];

    final int curFishSize = fishSize[i];

    // 当前的鱼是否被栈中的鱼吃掉了

    boolean hasEat = false;

    // 如果栈中还有鱼，并且栈中鱼向右，当前的鱼向左游，那么就会有相遇的可能性

    while (!t.empty() && fishDirection[t.peek()] == right &&

           curFishDirection == left) {

      // 如果栈顶的鱼比较大，那么把新来的吃掉

      if (fishSize[t.peek()] > curFishSize) {

        hasEat = true;

        break;

      }

      // 如果栈中的鱼较小，那么会把栈中的鱼吃掉，栈中的鱼被消除，所以需要弹栈。

      t.pop();

    }

    // 如果新来的鱼，没有被吃掉，那么压入栈中。

    if (!hasEat) {

      t.push(i);

    }

  }

  return t.size();

}

```

**复杂度分析**：每只鱼只入栈一次，出栈一次，所以时间复杂度 为 O(N)，而空间复杂度为 O(N)，因为最差情况下可能把所有的鱼都入栈。

【**小结**】接下来我们一起对这道题做一下归纳。可以发现，与例 1 相比，它们的消除行为有所不同：

- 在例 1 中，消除行为表现为配对的**两者都会消除；**
- 在例 2 中，消除行为表现为配对的两者中**有一个会被消除**。

此外，在与 例 1 的比较中，可以发现，栈中的内容也有所不同：

- 在例 1 中，栈中的**存放的就是内容本**身；
- 在例 2 中，栈中存**放的只是内容的索引**，可以通过索引得到内容。

再者，我们也发现，在弹栈的时候，不再像以前那样，每次只弹一个元素，而是采用了 while 循环，要一直弹到满足某个条件为止。所以我们总结出，弹栈的时候有两种情况：

- **弹一个元素**就可以；
- 用 while 语句**一直弹，直到满足某个条件**为止。

因此，这道题的考点我们也挖掘出来了：

- 是否会**用栈来存放索引**？
- 是否想到在弹栈的时候一定要**满足某个条件才停止弹栈**？

![](https://s0.lgstatic.com/i/image6/M01/0B/77/Cgp9HWA4n9WAA59XAACgLfhWcGY098.png)

**单调栈的解题技巧**

首先我们看一下**单调栈的定义**：单调栈就是指栈中的元素**必须**是按照**升序**排列的栈，或者是**降序**排列的栈。对于这两种排序方式的栈，还给它们各自取了小名。

升序排列的栈称为**递增栈**，如下图所示：

![8.gif](https://s0.lgstatic.com/i/image6/M01/0B/7B/CioPOWA4qWKAWwxXAAClLMMoPFk436.gif)

降序排列的栈称为**递减栈**，如下图所示：

![9.gif](https://s0.lgstatic.com/i/image6/M01/0B/7F/Cgp9HWA4qXCAM4-PAADnGTexjMk160.gif)

*注意：示意图所展示的这两种栈是横向排列的。栈中元素的值，分别用不同高度的矩形来表示，值越大，矩形越高。*

接下来我们介绍一下递增栈的有序性，一句话：“**任何时候都需要保证栈的有序性**”。

递增栈的特性可以演示如下（上方数组是要依次入栈的元素）：

![13.gif](https://s0.lgstatic.com/i/image6/M00/0B/7B/CioPOWA4qXmAUt2VAANQuRNAR14194.gif)

递减栈的特性可以演示如下：

![14.gif](https://s0.lgstatic.com/i/image6/M00/0B/7B/CioPOWA4qYCABi8aAAUfrNnOGUY452.gif)

通过这两个动图，我们可以总结出单调栈的特点，如下图所示：

![Drawing 29.png](https://s0.lgstatic.com/i/image6/M01/0B/7C/CioPOWA4qiiAEfpbAABn_-GStTI565.png)

**例 3：找出数组中右边比我小的元素**

【**题目**】一个整数数组 A，找到每个元素：右边第一个比我小的下标位置，没有则用 -1 表示。

输入：[5, 2]

输出：[1, -1]

**解释**：因为元素 5 的右边离我最近且比我小的位置应该是 A[1]，最后一个元素 2 右边没有比 2 小的元素，所以应该输出 -1。

复制代码

```
接口：int[] findRightSmall(int[] A);
```

【**分析**】每次开始分析题意时，记得要拿出我们的“**四步分析法”**，通过一步步分析找到题目相应的解法。

**1. 模拟**

在正式开始上手之后，我们先拿两个例子演示一下，看看能不能发现题目中隐藏的一些有趣规律，动图如下所示：

![15.gif](https://s0.lgstatic.com/i/image6/M00/0B/7F/Cgp9HWA4qYqASCuDAArtP3-ZB0A448.gif)

**2. 规律**

这里我们是照着题意去寻找一个右边比它小的数的下标。可以发现，A[4] = 4 及 A[5] = 0，这两个数字多次被用到。并且：

- A[4] 发现有左边 A[3]，A[3] 就匹配成功；
- 结合 A[5] = 0 的例子，我们发现它会把比它大的数都进行**匹配成功**，但是 A[3] 除外；
- A[3] 可以认为是匹配成功之后，被 A[4]**消除**了。

**这时可以总结出：一个数总是想与左边比它大的数进行匹配，匹配到了之后，小的数会消除掉大的数**。

**3. 匹配**

当你发现要解决的题目有两个特点：

- 小的数要与大的数**配对**
- 小的数会**消除**大的数

你的脑海里应该联想到关于**单调栈**的特性。下面我们看看如何利用单调栈解决这道题目。

【**画图**】在这里，依然需要画一个图来描述一下我们的思路及想法，如下图所示：（红色部分表示栈，我们只将**下标绿色值**放到栈中，为了看图方便，把下标对应的值也标在了相应位置。）

![16.gif](https://s0.lgstatic.com/i/image6/M00/0B/7F/Cgp9HWA4qkaALlpRAHsvPijzTIg101.gif)

Step 1. 首先将 A[0] = 1 的下标 **0** 入栈。

Step 2. 将 A[1] = 2 的下标 1 入栈。满足单调栈。

Step 3. 将 A[2] = 4 的下标 2 入栈。满足单调栈。

Step 4. 将 A[3] = 9 的下标 3 入栈。满足单调栈。

Step 5. 将 A[4] = 4 的下标 4 入栈时，不满足单调性，需要将 A[3] = 9 从栈中弹出去。下标 4 将栈中下标 3 弹出栈，记录 A[3] 右边更小的是 index = 4。

Step 6. 将 A[5] = 0 的下标 5 入栈时，不满足单调性，需要将 A[4] = 4 从栈中弹出去。下标 5 将下标 4 弹出栈，记录 A[4] 右边更小的是 index = 5。A[5] = 0 会将栈中的下标 0, 1, 2 都弹出栈，因此也需要记录相应下标右边比其小的下标为 5，再将 A[5] = 0 的下标 5 放入栈中。

Step 7. 将 A[6] = 5 的下标 6 放入栈中。满足单调性。

Step 8. 此时，再也没有元素要入栈了，那么栈中的元素右边没有比其更小的元素。因此设置为 -1.

```java
public static int[] findRightSmall(int[] A) {

  // 结果数组

  int[] ans = new int[A.length];

  // 注意，栈中的元素记录的是下标

  Stack<Integer> t = new Stack();

  for (int i = 0; i < A.length; i++) {

    final int x = A[i];

    // 每个元素都向左遍历栈中的元素完成消除动作

    while (!t.empty() && A[t.peek()] > x) {

      // 消除的时候，记录一下被谁消除了

      ans[t.peek()] = i;

      // 消除时候，值更大的需要从栈中消失

      t.pop();

    }

    // 剩下的入栈

    t.push(i);

  }

  // 栈中剩下的元素，由于没有人能消除他们，因此，只能将结果设置为-1。

  while (!t.empty()) {

    ans[t.peek()] = -1;

    t.pop();

  }

  return ans;

}

```

**复杂度分析**：每个元素只入栈一次，出栈一次，所以时间复杂度为 O(N)，而空间复杂度为 O(N)，因为最差情况可能会把所有的元素都入栈。

【**小结**】到这里我们可以得到一个有趣且非常有用的结论：数组中右边**第一个比我小**的元素的位置，求解用**递增栈**。

![](https://s0.lgstatic.com/i/image6/M01/0B/7F/Cgp9HWA4qrWAR4cuAADTLTA3i8c099.png)

**例 4：字典序最小的 k 个数的子序列**

【**题目**】给定一个正整数数组和 k，要求依次取出 k 个数，输出其中数组的一个子序列，需要满足：1. **长度为 k**；2.**字典序最小**。

输入：nums = [3,5,2,6], k = 2
输出：[2,6]

**解释**：在所有可能的解：{[3,5], [3,2], [3,6], [5,2], [5,6], [2,6]} 中，[2,6] 字典序最小。

所谓字典序就是，给定两个数组：x = [x1,x2,x3,x4]，y = [y1,y2,y3,y4]，如果 0 ≤ p < i，xp == yp 且 xi < yi，那么我们认为 x 的字典序小于 y。

复制代码

```
接口：int[] findSmallSeq(int[] A, int k);
```

【**分析**】根据“四步分析法”，我们一步一步拆解题目。

**1. 模拟**

首先应该拿例子来模拟一下题目所述的过程，动图如下所示：

![12.gif](https://s0.lgstatic.com/i/image6/M01/0B/80/Cgp9HWA4qsyASrO1AAMU43HNuI4415.gif)

**2. 规律**

通过模拟，我们发现**一个特点：一旦发现更小的数时，就可以把前面已经放好的数扔掉，然后把这个最小的数放在最前面**。

如果机智一点，就会发现这里与**例 2 的**“**大鱼吃小鱼**”结果很像。区别在于消除的过程中，大鱼吃小鱼是大鱼留下来了，而这里较小的数和较大的数相遇时，是**较小的数**留下来了。

**3. 匹配**

到这里，我们已经发现了题目的特点——**较小数消除掉较大数**。根据**例 3** 总结出来的规律，此时就可以用上单调栈。并且，由于是较小的数消除掉较大的数，所以应该使用**递增栈**。

**4. 边界**

不过我们还是需要小心题目的边界。

**Case 1**：假设数组右边有一个最小的数，这个最小的数会把左边的数全部都消掉，然后递增栈里面就只剩下这 1 个数了。这跟题意有点不符合，题意需要的是找到 k = 2 个出来。

![10.gif](https://s0.lgstatic.com/i/image6/M01/0B/80/Cgp9HWA4quuAMBDzAALrKCGW33s184.gif)

**解决办法**：不过你可以想一想，是不是可以控制一下消去的数目。当剩下的数字个数与栈中的元素刚好能凑够 k 个数时，就不能再消除了，代码如下 :

复制代码

```
rightLeftNumber + stack.size() == k
```

此时，如果还要进行消除，就不能凑够 k 个数了。这样操作可以保证我们取的序列是最小的 k 个数。

**Case 2**：如果数组是一个升序的数组，那么此时所有的元素都会被压栈。栈中的数目有可能远远超出 k 个。

![11.gif](https://s0.lgstatic.com/i/image6/M00/0B/80/Cgp9HWA4qxqAFbVgAAH8B7oHgJo512.gif)

**解决办法**：只需要把栈中的多出来的数字弹出来即可。

【**画图**】假定输入为[9, 2, 4, 5, 1, 2, 3, 0], k = 3.输出能构成的最小的序列。

![17.gif](https://s0.lgstatic.com/i/image6/M01/0B/81/Cgp9HWA4qzaAHEjwAGccHgMTVOU699.gif)

Step 1. 首先将 9 加入栈中。

Step 2. 当 2 要入栈时，不满足单调栈，需要将数字 9 出栈。由于后面还有足够多的元素，可以把 9 弹栈，再将 2 入栈。

Step 3. 将 4 入栈，满足单调性。

Step 4. 再将元素 5 入栈，满足单调性。

Step 5. 将要入栈的元素 1，会弹出栈中所有元素。

Step 6. 将元素 1 入栈。

Step 7. 将元素 2 入栈，满足单调性。

Step 8. 将元素 3 入栈，满足单调性。

Step 9. 将 0 入栈时，需要将栈顶元素 3 弹出。

Step 10. 将 0 入栈，不满足单调性。这是因为，如果 0 将前面的元素再弹栈，余下的元素个数就小于 k = 3 个了。所以不能再利用单调性来弹出栈中元素了。

```java
public int[] findSmallSeq(int[] nums, int k) {

  int[] ans = new int[k];

  Stack<Integer> s = new Stack();

  // 这里生成单调栈

  for (int i = 0; i < nums.length; i++) {

    final int x = nums[i];

    final int left = nums.length - i;

    // 注意我们想要提取出k个数，所以注意控制扔掉的数的个数

    while (!s.empty() && (s.size() + left > k) && s.peek() > x) {

      s.pop();

    }

    s.push(x);

  }

  // 如果递增栈里面的数太多，那么我们只需要取出前k个就可以了。

  // 多余的栈中的元素需要扔掉。

  while (s.size() > k) {

    s.pop();

  }

  // 把k个元素取出来，注意这里取的顺序!

  for (int i = k - 1; i >= 0; i--) {

    ans[i] = s.peek();

    s.pop();

  }

  return ans;

}

```

**复杂度分析**：每个元素只入栈一次，出栈一次，所以时间复杂度为 O(N)，而空间复杂度为 O(N)，因为最差情况可能会把所有元素都入栈。

【**小结**】写完代码之后，我们需要对代码和题目做一个小结：

- 较小的数**消除**掉较大的数的时候，使用**递增栈；**
- 要注意控制剩下的元素的个数；

如果更进一步推而广之，会发现**从简单栈到单调栈，层层推进的过程中，不停变化就是入栈与出栈的时机**。

那么，到这里，这个题目的考点也就非常明了了：

- 递增栈
- 个数控制，我们只需要取 k 个数出来。

![](https://s0.lgstatic.com/i/image6/M01/0B/7F/CioPOWA4q6qASB-UAADhj7uzOwg933.png)

思考题：（未做）

[柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

#### 队列 Queue

> 队列也是一种线性结构
>
> 相比数组，队列对应的操作是数组的子集
>
> 只能有一端（队尾）添加元素，只能从一端（队首）取出元素
>
> 队列是一种先进先出的数据结构（First In First Out FIFO）

#### 队列实现

```javascript
class ArrayQueue {
	constructor() {
		this.data = [];
	},
	size() {
    return this.data.length;
  },
  isEmpty() {
    return this.data.length === 0;
  },
  enqueue(e) {
    this.data.push(e);
  },
  dequeue() {
    return this.data.shift();
  },
  getFront() {
    return this.data[0];
  }
}
```

#### 循环队列

```javascript
function loopQueue(len){
  // 需要多一个空间 来进行判断队列满
  let _arr = new Array(len+1);
  // 队头指针
  let front = 0;
  // 队尾指针
  let rear = 0;
	// 队列大小
  let size = 0;
  this.enQueue = function(item){
    if(this.isFull()){
      console.log('队列已满!');
      return false;
    }    
    _arr[rear] = item;
    rear = (rear+1)%_arr.length;
    size++;
  }

  this.deQueue = function(){
    if(this.isEmpty()){
      console.log('队列为空!');
      return false;
    }
    let item = _arr[front];
    _arr[front] = null;
    front = (front+1)%_arr.length;
    size--;
    return item;
  }

  this.isEmpty = function(){
    return front === rear;
  }

  this.isFull = function(){
    // 预留一个空间
    return (rear+1) % _arr.length === front;
  }
}
```

**例1：二叉树的层次遍历（两种方法）**

【**题目**】从上到下按层打印二叉树，同一层结点按从左到右的顺序打印，每一层打印到一行。

输入：

![Drawing 4.png](https://s0.lgstatic.com/i/image6/M00/11/08/CioPOWA_RwyAWm07AACF5LV9ej0062.png)

输出：[[3], [9, 8], [6, 7]]

复制代码

```javascript
// 二叉树结点的定义

public class TreeNode {

  // 树结点中的元素值

  int val = 0;

  // 二叉树结点的左子结点

  TreeNode left = null;

  // 二叉树结点的右子结点

  TreeNode right = null;

}
```

**1. 模拟**
首先我们在这棵树上进行模拟，动图演示效果如下所示：

![2.gif](https://s0.lgstatic.com/i/image6/M00/11/08/CioPOWA_RyiAQ0IkAAbQTq2M1V8935.gif)

**2. 规律**
通过运行的模拟，可以总结出以下两个特点。

**（1）广度遍历**（**层次遍历**）：由于二叉树的特点，当我们拿到第 N 层的结点 A 之后，可以通过 A 的 left 和 right 指针拿到下一层的结点。

**（2）顺序输出**：每层输出时，排在**左边的结点**，它的**子结点同样**排在**下一层最左边**。

**3. 匹配**

当你发现题目具备**广度遍历**（**分层遍历**）**和顺序输出的特点，\**就应该想到用\**FIFO 队列**来试一试。

**4.. 边界**

关于二叉树的边界，需要考虑一种空二叉树的情况。当遇到一棵空的二叉树，有两种解决办法。

（1）**特殊判断**：如果发现是一棵空二叉树，就直接返回空结果。

（2）**制定一个规则**：不要让空指针进入到 FIFO 队列。

【**画图**】当我们拿到一道题，脑海中已经关联了相应的数据结构：FIFO 队列，下面就可以利用它来画图了。

不过，二叉树的层次遍历与标准的 FIFO 队列不太一样，需要在每一层开始处理之前，记录一下 Queue Size（当前层里面结点的个数），演示如下图所示：

![3.gif](https://s0.lgstatic.com/i/image6/M01/11/0C/Cgp9HWA_R4WADJ8eACXiUG8cfgY721.gif)

Step1. 在一开始首先将根结点 3 加入队列中。

Step 2. 开始**新一层遍历**，记录下当前队列长度 QSize=1，初始化当前层存放结果的[]。

Step 3. 将结点 3 出队，然后将其放到当前层中。

Step 4. 再将结点 3 的左右子结点分别入队。QSize = 1 的这一层已经处理完毕。

Step 5. **开始新一层的遍历**。记录下新一层的 QSize = 2，初始化新的当前层存放当前层结果的[]。

Step 6. 从队列中取出 9，放到当前层结果中。结点 9 没有左右子结点，不需要继续处理左右子结点。

Step 7. 从队列中取出 8，放到当前层结果中。

Step 8. 将结点 8 的左右子结点分别入队。此时，QSize = 2 的部分已经全部处理完成。

Step 9.**开始新一层的遍历**，记录下当前队列中的结点数 QSize = 2，并且生成存放当前层结果的 list[]。

Step 10. 将队首结点 6 出队放到当前层结果中。结点 6 没有左右子结点，没有元素要入队。

Step 11. 将队首结点 7 出队，放到当前层结果中。结点 7 没有左右子结点，没有元素要入队。

结束，返回我们层次遍历的结果。

```java
class Solution {

    public List<List<Integer>> levelOrder(TreeNode root) {

        // 生成FIFO队列

        Queue<TreeNode> Q = new LinkedList<>();

        // 如果结点不为空，那么加入FIFO队列

        if (root != null) {

            Q.offer(root);

        }

        // ans用于保存层次遍历的结果

        List<List<Integer>> ans = new LinkedList<>();

        // 开始利用FIFO队列进行层次遍历

        while (Q.size() > 0) {

            // 取出当前层里面元素的个数

            final int qSize = Q.size();

            // 当前层的结果存放于tmp链表中

            List<Integer> tmp = new LinkedList<>();

            // 遍历当前层的每个结点

            for (int i = 0; i < qSize; i++) {

                // 当前层前面的结点先出队

                TreeNode cur = Q.poll();

                // 把结果存放当于当前层中

                tmp.add(cur.val);

                // 把下一层的结点入队，注意入队时需要非空才可以入队。

                if (cur.left != null) {

                    Q.offer(cur.left);

                }

                if (cur.right != null) {

                    Q.offer(cur.right);

                }

            }

            // 把当前层的结果放到返回值里面。

            ans.add(tmp);

        }

        return ans;

    }

}

```

**复杂度分析**：由于二叉树的每个结点，我们都只访问了一遍，所以时间复杂度为 O(n)。如果不算返回的数组，那么空间复杂度为 O(k)，这里的 k 表示二叉树横向最宽的那一层的结点数目。

【**小结**】写完代码之后，对 FIFO 队列进行一轮总结。现在除了知道先进先出的特点之外，还可以进一步细化知识点，如下图所示：

![](https://s0.lgstatic.com/i/image6/M01/11/09/CioPOWA_R6aAdoJvAACnbi7IL-c504.png)

在依靠 FIFO 队列的解法中，我们**利用 QSize 得到当前层的元素个数，然后再开始执行 FIFO 是处理分层遍历的关键**

【**解法二**】再来回顾一下题目的特点：

- 分层遍历
- 顺序遍历

那么我们是不是可以用 List 来表示每一层，把下一层的结点统一放到一个新生成的 List 里面。示意图如下：

![4.gif](https://s0.lgstatic.com/i/image6/M00/11/09/CioPOWA_R9eAb3DqAA5cp3pt5r8391.gif)

Step 1. 首先将结点 3 加入 cur,，形成 cur=[3]。

Step 2. 开始依次遍历当前层 cur, 这里 cur 只有结点 3，依次把结点 3 的左子结点和右子结点加入 next，形成 [9, 8]。

Step 3. 将 cur 指向 next，并且 next 设置为 []

Step 4. 依次遍历 cur，并将每个结点的左右子结点放到 next 中。

Step 5. 将 cur 指向 next。并依次遍历。由于这是最后一层，所以不会再生成 next。

Step 6. 最后得到层次遍历的结果。

```java
class Solution {

    public List<List<Integer>> levelOrder(TreeNode root) {

        List<List<Integer>> ans = new ArrayList<>();

        // 初始化当前层结点

        List<TreeNode> curLevel = new ArrayList<>();

        // 注意：需要root不空的时候才加到里面。

        if (root != null) {

            curLevel.add(root);

        }

        while (curLevel.size() > 0) {

            // 准备用来存放下一层的结点

            List<TreeNode> nextLevel = new ArrayList<>();

            // 用来存放当前层的结果

            List<Integer> curResult = new ArrayList<>();

            // 遍历当前层的每个结点

            for (TreeNode cur: curLevel) {

                // 把当前层的值存放到当前结果里面

                curResult.add(cur.val);

                // 生成下一层

                if (cur.left != null) {

                    nextLevel.add(cur.left);

                }

                if (cur.right != null) {

                    nextLevel.add(cur.right);

                }

            }

            // 注意这里的更迭!滚动前进

            curLevel = nextLevel;

            // 把当前层的值放到结果里面

            ans.add(curResult);

        }

        return ans;

    }

}

```

通过这个有趣的解法，我们知道，FIFO 队列不仅可以用 Queue 表示，还可以用两层 ArrayList 来表示，均可达到同样的效果。再把思路扩展一下，思考**是否还有其他的形式可以表达 FIFO 队列呢**？

**思考题**

#### [填充每个节点的下一个右侧节点指针 II](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii/)

![](https://s0.lgstatic.com/i/image6/M00/11/0C/Cgp9HWA_SC2AdwWAAADBBGybQP0811.png)

**二叉树的层次遍历**的解题技巧

> [相关题目](https://github.com/lagoueduCol/Algorithm-Dryad/blob/main/02.Queue/README.md)

![](https://s0.lgstatic.com/i/image6/M01/11/0C/Cgp9HWA_SEGALU-UAADmDhvBE6M451.png)

**例 2：循环队列**

【**题目**】设计一个可以容纳 k 个元素的循环队列。需要实现以下接口：

复制代码

```java
class MyCircularQueue {

    // 参数k表示这个循环队列最多只能容纳k个元素

    public MyCircularQueue(int k);

    // 将value放到队列中, 成功返回true

    public boolean enQueue(int value);

    // 删除队首元素，成功返回true

    public boolean deQueue();

    // 得到队首元素，如果为空，返回-1

    public int Front();

    // 得到队尾元素，如果队列为空，返回-1

    public int Rear();

    // 看一下循环队列是否为空

    public boolean isEmpty();

    // 看一下循环队列是否已放满k个元素

    public boolean isFull();

}
```

循环队列的重点在于**循环使用固定空间**，难点在于**控制好 front/rear 两个首尾指示器**

【**方法** 1】只使用 k 个元素的空间，三个变量 front, rear, used 来控制循环队列的使用。分别标记 k = 6 时，循环队列的三种情况，如下图所示：

![Drawing 36.png](https://s0.lgstatic.com/i/image6/M01/11/0C/Cgp9HWA_SF2AEV3pAADK0cYKmv8794.png)

由图可知，在一般情况下，front 和 rear 都是不相等的。但是，如果仔细观察，你会发现在空队列与满队列的时候，front 和 rear 是相等的。那此时该怎么处理呢？

通过上述分析，可以知道只用 front 和 rear 两个变量，还不足以区分是空队列还是满队列，因此我们还需要用到额外的变量做进一步区分。一种比较简单的办法就是**采用 used 变量**，标记已经放了多少个元素在循环队列里面。

- 如图（a）所示，当队列为空的时候，used == 0；
- 如图（b）所示，当队列满的时候，used == k。

虽然从图片来看这是一个循环数组，但是面试官要求只能使用一个普通的数组来实现。在下标的移动上，要特别注意不要越界。下标只能在 [0, k-1] 范围里面移动。以下 3 点需要你格外注意，正常情况下：

1. index = i 的后一个是 i + 1，前一个是 i - 1
2. index = k-1 的后一个就是 index = 0
3. index = 0 的前一个是 index = k-1

实际上，这三个式子都可以利用**取模的技巧**来统一处理：

- index = i 的后一个 (i + 1) % capacity
- index = i 的前一个(i - 1 + capacity) % capacity

***注意：所有的循环数组下标的处理都需要按照这个取模方法来。***

```java
class MyCircularQueue {

    // 已经使用的元素个数

    private int used = 0;

    // 第一个元素所在位置

    private int front = 0;

    // rear是enQueue可在存放的位置

    // 注意开闭原则

    // [front, rear)

    private int rear = 0;

    // 循环队列最多可以存放的元素个数

    private int capacity = 0;

    // 循环队列的存储空间

    private int[] a = null;

    public MyCircularQueue(int k) {

        // 初始化循环队列

        capacity = k;

        a = new int[capacity];

    }

    public boolean enQueue(int value) {

        // 如果已经放满了

        if (isFull()) {

            return false;

        }

        // 如果没有放满，那么a[rear]用来存放新进来的元素

        a[rear] = value;

        // rear注意取模

        rear = (rear + 1) % capacity;

        // 已经使用的空间

        used++;

        // 存放成功!

        return true;

    }

    public boolean deQueue() {

        // 如果是一个空队列，当然不能出队

        if (isEmpty()) {

            return false;

        }

        // 第一个元素取出

        int ret = a[front];

        // 注意取模

        front = (front + 1) % capacity;

        // 已经存放的元素减减

        used--;

        // 取出元素成功

        return true;

    }

    public int Front() {

        // 如果为空，不能取出队首元素

        if (isEmpty()) {

            return -1;

        }

        // 取出队首元素

        return a[front];

    }

    public int Rear() {

        // 如果为空，不能取出队尾元素

        if (isEmpty()) {

            return -1;

        }

        // 注意：这里不能使用rear - 1

        // 需要取模

        int tail = (rear - 1 + capacity) % capacity;

        return a[tail];

    }

    // 队列是否为空

    public boolean isEmpty() {

        return used == 0;

    }

    // 队列是否满了

    public boolean isFull() {

        return used == capacity;

    }

}

```

**复杂度分析**：入队操作与出队操作都是 O(1)。

【**方法 2**】方法 1 利用 used 变量对满队列和空队列进行了区分。实际上，这种区分方式还有另外一种办法，使用 k+1 个元素的空间，两个变量 front, rear 来控制循环队列的使用。具体如下：

- 在申请数组空间的时候，申请 k + 1 个空间；
- 在放满循环队列的时候，必须要保证 rear 与 front 之间有空隙。

**如下图**（**此时 k = 5**）**所示**：

![Drawing 38.png](https://s0.lgstatic.com/i/image6/M01/11/09/CioPOWA_SHeAP85DAADKwUx6Fio771.png)

此时，可以发现，循环队列实际上是浪费了一个元素的空间。这个浪费的元素必须卡在 front 与 rear 之间。判断队列空或者满可以：

- front == rear 此时队列为空；
- (rear + 1) % capacity == front，此时队列为满。

***注意：由于浪费了一个元素的空间，在申请数组的时候，要申请的空间大小为 k + 1, 并且 capacity 也必须为 k + 1。***

除此之后，由于是循环数组，下标的活动范围是[0, k]（capacity 为 k+1，所以最大只能取到k）。下标的移动仍然需要利用**取模的技巧**。

```java
class MyCircularQueue {

    // 队列的头部元素所在位置

    private int front = 0;

    // 队列的尾巴

    // 注意我们采用的是前开后闭原则

    // [front, rear)

    private int rear = 0;

    private int[] a = null;

    private int capacity = 0;

    public MyCircularQueue(int k) {

        // 初始化队列，注意此时队列中元素个数为

        // k + 1

        capacity = k + 1;

        a = new int[k + 1];

    }

    public boolean enQueue(int value) {

        // 如果已经满了，无法入队

        if (isFull()) {

            return false;

        }

        // 把元素放到rear位置

        a[rear] = value;

        // rear向后移动

        rear = (rear + 1) % capacity;

        return true;

    }

    public boolean deQueue() {

        // 如果为空，无法出队

        if (isEmpty()) {

            return false;

        }

        // 出队之后，front要向前移

        front = (front + 1) % capacity;

        return true;

    }

    public int Front() {

        // 如果能取出第一个元素，取a[front];

        return isEmpty() ? -1 : a[front];

    }

    public int Rear() {

        // 由于我们使用的是前开后闭原则

        // [front, rear)

        // 所以在取最后一个元素时，应该是

        // (rear - 1 + capacity) % capacity;

        int tail = (rear - 1 + capacity) % capacity;

        return isEmpty() ? -1 : a[tail];

    }

    public boolean isEmpty() {

        // 队列是否为空

        return front == rear;

    }

    public boolean isFull() {

        // rear与front之间至少有一个空格

        // 当rear指向这个最后的一个空格时，

        // 队列就已经放满了!

        return (rear + 1) % capacity == front;

    }

}

```

**复杂度分析**：入队与出队操作都是 O(1)。

我们介绍了两种方法来实现循环队列，下面我分别从**相似点、差别，以及适用范围**对这两种方法进行总结。

**1. 相似点**

两种方法都是利用了**取模**的技巧，强调一下，在取模的时候，如果需要向前移动，不要写成 (i - 1) % capacity，注意一定要加上 capacity 之后再取模，否则在 i = 0 的时候就出错了。

复制代码

```
pre = (i - 1 + capacity) % capacity
```

**2. 差别**

这两种方法的唯一区别在于**区分空队列与满队列**时，方法不一样：

- 方法 1 引入了另外一个变量 used 进行区分
- 方法 2 采用了浪费一个存储空间的办法进行区分

**3. 适用范围**

你可能认为方法 2 在队列元素较大时，存在浪费的情况，实际上这两种办法都有不同的适用范围。

方法 1 的缺点在于控制变量较多，达到 3 个。而方法 2 虽然浪费了一个存储空间，但是控制变量较少，只有 2 个。

**在多线程编程里面，控制变量越少，越容易实现无锁编程，因此，在无锁队列里面，利用方法 2 较容易实现无锁队列**。

![](https://s0.lgstatic.com/i/image6/M01/11/09/CioPOWA_SIqAG7qhAADoVzWnab0092.png)

#### 单调队列

### 单调队列

单调队列属于**双端队列**的一种。双端队列与 FIFO 队列的区别在于：

- FIFO 队列只能从尾部添加元素，首部弹出元素；
- 双端队列可以从首尾两端 push/pop 元素。

为了让你更直观地看出两者的区别，我分别绘制了 FIFO 队列和双端队列的图片，如下图所示：

![Drawing 42.png](https://s0.lgstatic.com/i/image6/M01/11/09/CioPOWA_SJeAYvJTAAB3ffWmPoY742.png)

FIFO 队列

![Drawing 44.png](https://s0.lgstatic.com/i/image6/M01/11/09/CioPOWA_SKmAJflUAACNz8oT0A8471.png)

双端队列

虽然双端队列经常用于工程实践中，但在面试中出现得较多的往往是**单调队列**

**什么是单调队列**

首先来看一下单调队列的定义：要求队列中的元素必须满足单调性，比如**单调递增，或者单调递减**。那么在入栈与出栈的时候，就与普通的队列不一样了。

接下来我将以单调递减为例，详细讲解单调队列的特性，希望你可以自己推导单调递增的情况。如果有什么疑问可以在评论区留言，我会定期给大家解答。

**单调队列在入队的时候，需要满足 2 点**：

1. 入队前队列已经满足单调性；
2. 入队后队列仍然满足单调性。

这里以单调递减队列为例，具体操作如下图所示（为了更直观地展示，我将不同大小的数值绘制为不同的高度）：

![5.gif](https://s0.lgstatic.com/i/image6/M01/11/0D/Cgp9HWA_SLyAEHB2AEJPbY2MLoE581.gif)

Step 1. 已有单调队列满足单调递减。

Step 2. 元素 5 要从尾部加入队列中。

Step 3. 元素 5 与尾部元素 3 比较，3 < 5，因此扔掉 3。

Step 4. 元素 5 与尾部元素 4 比较，4 < 5，因此扔掉 4。

Step 5. 元素 5 与尾部元素 6 比较，6 > 5，因此将 5 入队。

可以发现，每次入队的时候，为了保证队列的单调性，还要**剔除掉尾部的元素**。**直到尾部的元素大于等于入队元素（因为是单调递减队列）**。

```java
    private void push(int val) {

        while (!Q.isEmpty() && Q.getLast() < val) {

            Q.removeLast();

        }

        // 将元素入队

        Q.addLast(val);

    }

```

**单调队列在出队时**，也与普通的队列出队方式**不一样**。出队时，需要给出一个 value，如果 value 与队首相等，才能将这个数出队，代码如下所示：

复制代码

```
    // 出队的时候，要相等的时候才会出队

    private void pop(int val) {

        if (!Q.isEmpty() && Q.getFirst() == val) {

            Q.removeFirst();

        }

    }
```

那么，采用这种比较特殊的入队与出队的方式有什么巧妙的地方呢？

- 入队之后，队首元素 Q.getFirst() 就是队列中的最大值。这个比较容易想到，因为我们这里是以单调递减队列为例，所以队首元素就是最大值。
- 出队时，如果一个元素已经被其他元素**剔除**出去了，就不会再出队。但如果一个元素是当前队列中的最大值，那么就会再出队。

关于这一点，你可以参考下面的动图演示（为了更清晰地演示此过程，被叉掉的元素还保留在原处，但实际上已经不在队列中了）

![](https://s0.lgstatic.com/i/image6/M00/11/0A/CioPOWA_SQOAZRMRABqVn-_iVoo720.gif)

Step 1. 将元素 5 入队，元素 3,4 会被剔除掉。区间 [8,6,4,3,5] 最大值为队首元素 8。

Step 2. 将元素 8 出队，元素相等，直接出队。区间 [6,4,3,5] 最大值为队首元素 6。

Step 3. 将元素 6 出队，元素相等，直接出队。区间 [4,3,5] 最大值为队首元素 5。

Step 4. 将元素 4 出队，此时元素不等，队列不变。区间 [3,5] 最大值为队首元素 5。

Step 5. 将元素 3 出队，此时元素不等，队列不变。区间 [5] 最大值为队首元素 5。

Step 6. 将元素 5 出队，此时元素相等，直接出队。

可以发现，单调递减队列最重要的特性是：**入队与出队的组合，可以在 O(1) 时间得到某个区间上的最大值**。

前面说了利用单调队列可以得到某个区间上的最大值。可是这个区间是什么？怎么定量地描述这个区间？与队列中的元素个数有什么关系？

针对以上 3 个疑问，可以分两种情况展开讨论：

1. 只有入队的情况
2. 有出队与入队的情况

为了更直观地展示，我分别制作了两种情况对应的动图演示。先来看只有入队的情况，如下图所示：


![7.gif](https://s0.lgstatic.com/i/image6/M01/11/0D/Cgp9HWA_STiAcHJnAAmEZ9koVKA128.gif)

Step 1. 元素 3 入队，此时队首元素为 3，表示着区间[3]最大值为 3。

Step 2. 元素 2 入队，此时队列首元素为 3，表示区间[3,2]最大值为 3。

Step 3. 元素 5 入队，此时队首元素为 5，此时队列覆盖范围长度为 3，可以得到**区间 [3,2,5] 最大值为 5。**

继续执行入队，想必你也能得出结论了：在没有出队的情况下，黄色覆盖范围会一直增加，队首元素就表示这个黄色覆盖范围的最大值。

下面我们再来看出队与入队混合的情况。在上图 Step3 的基础上，如果再把 A[3] = 6 入队，这个时候，队列的覆盖范围长度为 4，假设我们想控制这个覆盖范围长度为 3，应该怎么办？

此时，我们只需要将 A[0] 出队即可。如下图所示：


![8.gif](https://s0.lgstatic.com/i/image6/M01/11/0E/Cgp9HWA_SX6ADn1yAAlfQannP2I331.gif)
Step 4. 将元素 A[0] = 3 出队，由于此时 3 != Q.getFirst()，所以什么也不做。队列覆盖范围为 [2, 5]，长度为 2。

Step 5. 将 A[3] = 6 入队，此时队首元素为 6，覆盖范围为[2,5,6]，覆盖长度为 3，可以得到**区间 [2,5,6] 最大值为 6**。

Step 6. 将 A[1] = 2 出队，此时 2 != Q.getFirst()，所以什么也不做。此时队列覆盖范围为 [5, 6]，长度为 2。

Step 7. 将 A[4] = 4 入队，此时覆盖范围为 [5, 6, 4]，覆盖长度为 3，**区间 [5,6,4] 最大值为 6**。

从上图中可以发现以下几个重点：

- 入队，扩张单调队列的覆盖范围；
- 出队，控制单调递减队列的覆盖范围；
- 队首元素就是覆盖范围的最大值；
- 队列中的元素个数**小于**覆盖范围元素的个数。

**例 3：滑动窗口的最大值**

【**题目**】给定一个数组和滑动窗口的大小，请找出所有滑动窗口里的最大值。

输入：nums = [1,3,-1,-3,5,3], k = 3

输出：[3,3,5,5]

**解释**：

![Drawing 68.png](https://s0.lgstatic.com/i/image6/M00/11/12/Cgp9HWA_S1aAJXv9AABKF_TFCN8607.png)

【**分析**】这是一道来自 **eBay** 的面试题。拿到时题目之后，可以发现，题目要求还是比较赤裸裸的，不妨先模拟一下，看看能不能想到比较好的解决办法。

**1. 模拟**

首先我们发现窗口在滑动的时候，有元素不停地进出。因此，可以采用**队列**来试一下。由于窗口长度为 3，所以将队列的长度固定为 3。
![9.gif](https://s0.lgstatic.com/i/image6/M00/11/0C/CioPOWA_ScmAQ8ZYAAoV9uo-AJQ439.gif)

Step 1. 首先将元素 1 入队。

Step 2. 再将元素 3 入队。

Step 3. 再将 -1 入队，此时队列长度为 3，可以从 [1, 3, -1] 中得到最大值 3。

Step 4. 将 1 出队，然后将 3 入队，可以得到 [3,-1,3] 的最大值为3。

Step 5. 将 3 出队，然后再将 5 入队，可以得到 [-1, 3, 5] 的最大值为 5。

Step 6. 将 -1 队出，然后再将 3 入队，可以得到 [3,5,3] 的最大值为 5。

**2. 规律**

我们发现两点：

（1）不停地有元素出队入队

（2）需要拿**到队列中的最大值**

如果能够在 O(1) 时间内拿到队列中的最大值，那么就可以在 O(N) 时间解决掉这个问题。

**3. 匹配**

到这里为止，已经匹配到了你学过的数据结构了——**单调递减队列**！

**4. 边界**

接下来，你可能准备开始写代码了，不过我还需要和你讨论一些细节与边界。

- 滑动窗口的大小与队列的大小。
- 哪种单调递减？为什么？

首先我们看**窗口的大小**。当使用 Q.getFirst() 时，得到的是整个队列的最大值，因此队列的大小，必须与滑动窗口的大小一样。也就是说，当 A[i] 入队的时候，A[i-k] 必须要出队！这样才能保证队列中的元素最多有 k 个。

虽然我们已经知道要使用单调递减队列求解这道题目了，但单调递减有两种：

1. 严格单调递减（队列中没有重复元素）
2. 单调递减

那么，应该用哪种呢？首先我们看一下**严格单调递减**是否可以工作，如下图所示：

![Drawing 76.png](https://s0.lgstatic.com/i/image6/M00/11/13/Cgp9HWA_TDqAU-urAADUKwAZCHk961.png)

假设执行到 A[2] = 3 时，采用**严格单调递减（队列中相等的元素也会被踢出去）**，入队时，A[2] 将会把所有的元素都踢出队列，队列变成 [3]，那么可以得到 [3,2,3] 的最大值为 3。

但是由于窗口滑动的时候，接着需要把 A[0] = 3 出队，出队之后，队列为空。然后再将 A[3] = 1 入队得到。

![Drawing 78.png](https://s0.lgstatic.com/i/image6/M00/11/13/Cgp9HWA_TEyAfSBRAADZkvnyEtw271.png)

此时得到 [2,3,1] 的最大值为 1，这就出错了！所以我们**不能使用严格单调递减队列求解**。

```java
class Solution {

    // 单调队列使用双端队列来实现

    private ArrayDeque<Integer> Q = new ArrayDeque<Integer>();

    // 入队的时候，last方向入队，但是入队的时候

    // 需要保证整个队列的数值是单调的

    // (在这个题里面我们需要是递减的)

    // 并且需要注意，这里是Q.getLast() < val

    // 如果写成Q.getLast() <= val就变成了严格单调递增

    private void push(int val) {

        while (!Q.isEmpty() && Q.getLast() < val) {

            Q.removeLast();

        }

        // 将元素入队

        Q.addLast(val);

    }

    // 出队的时候，要相等的时候才会出队

    private void pop(int val) {

        if (!Q.isEmpty() && Q.getFirst() == val) {

            Q.removeFirst();

        }

    }

    public int[] maxSlidingWindow(int[] nums, int k) {

        List<Integer> ans = new ArrayList<>();

        for (int i = 0; i < nums.length; i++) {

            push(nums[i]);

            // 如果队列中的元素还少于k个

            // 那么这个时候，还不能去取最大值

            if (i < k - 1) {

                continue;

            }

            // 队首元素就是最大值

            ans.add(Q.getFirst());

            // 尝试去移除元素

            pop(nums[i - k + 1]);

        }

        // 将ans转换成为数组返回!

        return ans.stream().mapToInt(Integer::valueOf).toArray();

    }

}

```

**复杂度分析**：每个元素都只入队一次，出队一次，每次入队与出队都是 O(1) 的复杂度，因此整个算法的复杂度为 O(n)。

【**小结**】至此，我们已经学习了利用单调队列来解决滑动窗口的最大值。下面还可以扩展一下，比如：如何解决滑动窗口的最大值与最小值？具体你可以参考“[题目](https://www.luogu.com.cn/problem/P1886)与[代码](https://github.com/lagoueduCol/Algorithm-Dryad/blob/main/02.Queue/P1886.java)”。

我们再对单调队列的特性做一下总结：

- 入队，扩展单调队列的覆盖范围
- 出队，缩小单调队列的覆盖范围
- 队首元素，是覆盖范围的最大值/最小值
- 范围内的最大值，需要用单调递减队列
- 范围内的最小值，需要用单调递增队列

单调队列在解决滑动窗口的最大值的时候，由于这个**滑动窗口的大小是固定的**。因此，单调队列的大小也是固定的。那么，你能不能用循环队列来模拟单调队列，求解滑动窗口最大值的题目呢？

> 代码：[Java](https://github.com/lagoueduCol/Algorithm-Dryad/blob/main/02.Queue/239.滑动窗口最大值.循环队列.java)/[C++](https://github.com/lagoueduCol/Algorithm-Dryad/blob/main/02.Queue/239.滑动窗口最大值.循环队列.cpp)/[Python](https://github.com/lagoueduCol/Algorithm-Dryad/blob/main/02.Queue/239.滑动窗口最大值.循环队列.py)

**思考题**

#### [跳跃游戏 VI](https://leetcode-cn.com/problems/jump-game-vi/)

![](https://s0.lgstatic.com/i/image6/M00/11/10/CioPOWA_TLCATeR6AAFTfMBlaiw858.png)

#### leetcode的一些题目

[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

[用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

[用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

[ 最小栈](https://leetcode-cn.com/problems/min-stack/)

[设计一个支持增量操作的栈](https://leetcode-cn.com/problems/design-a-stack-with-increment-operation/)

[用栈操作构建数组](https://leetcode-cn.com/problems/build-an-array-with-stack-operations/)

[设计循环队列](https://leetcode-cn.com/problems/design-circular-queue/)

