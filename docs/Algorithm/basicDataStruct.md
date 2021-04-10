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

#### leetcode的一些题目

[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

[用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

[用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

[ 最小栈](https://leetcode-cn.com/problems/min-stack/)

[设计一个支持增量操作的栈](https://leetcode-cn.com/problems/design-a-stack-with-increment-operation/)

[用栈操作构建数组](https://leetcode-cn.com/problems/build-an-array-with-stack-operations/)

[设计循环队列](https://leetcode-cn.com/problems/design-circular-queue/)

