
# 牛客周赛 Round 27 解题报告 | 珂学家 | 组合数学 + 滑窗

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20240108/446702330_1704677434139/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

牛客周赛好像变了，变成核心代码编写模式了。

T3是经典滑窗题，T4是道有趣的数学题。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 小红的二进制删数字](https://ac.nowcoder.com/acm/contest/73056/A)

本质就是统计1的个数m

然后答案为：m-1

```python []
from collections import Counter

class Solution:
    def minCnt(self , s: str) -> int:
        cnt = Counter(s)
        return cnt['1'] - 1
```

---

## [B. 嘤嘤的新平衡树](https://ac.nowcoder.com/acm/contest/73056/B)

思路：贪心

对于每个节点

其数量为 max(子节点的数) * 2 + 1

dfs一下即可

```python []
class Solution:
    def getTreeSum(self , tree: TreeNode) -> int:
        # write code here
        def getRoot(tree: TreeNode) -> int:
            if tree == None:
                return 0
            mz = max(getRoot(tree.left), getRoot(tree.right))
            return mz * 2 + 1
        
        v = getRoot(tree)
        mod = 10 ** 9 + 7
        return v % mod
```

---

## [C. 连续子数组数量](https://ac.nowcoder.com/acm/contest/73056/C)

对2,5因子进行前缀和的预处理

然后用经典的滑窗技巧，即可

```python []
from itertools import accumulate

class Solution:
    def getSubarrayNum(self , a: List[int], x: int) -> int:
        n = len(a)
        cnt2 = [0] * n
        cnt5 = [0] * n
        
        for i in range(n):
            ca = a[i]
            while ca % 5 == 0:
                ca /= 5
                cnt5[i] += 1
            while ca % 2 == 0:
                ca /= 2
                cnt2[i] += 1
        
        pre2 = list(accumulate(cnt2, initial=0))
        pre5 = list(accumulate(cnt5, initial=0))
        
        ans = 0
        j = 0
        for i in range(n):
            while min(pre2[i + 1] - pre2[j], pre5[i + 1] - pre5[j]) >= x:
                j += 1
            ans += j
        
        mod = 10 ** 9 + 7
        return ans % mod
```


---
## [D. 好矩阵](https://ac.nowcoder.com/acm/contest/73056/D)

思路: 组合数学+思维

$$先思维，再组合数学$$

2x2格子总合是偶数，其实主要关注0-1奇偶就行，不需要关注其具体是什么值

---

在这个基础上，我们可以找到如下规律

就是第一行，每个位子可以随意选择，即0-1奇偶都行

然后从第二行开始，除了第一列，可以0-1奇偶选择，其他列的值都是固定的

因此其总方案数为: 2^m * 2^(n - 1) == 2^(n+m-1)

---

然后我们在回过来头考虑在[1，x]具体的选择值

由于x为偶数，所以根据乘法原理，其附带选择数位 (x/2)^(m*n)

---

最终的结果为： 2^(n+m-1) * (x/2)^(m*n)

题外话，如果x为奇数，感觉这题没法解了。


```python []
class Solution:
    def numsOfGoodMatrix(self , n: int, m: int, x: int) -> int:
        # 只考虑奇偶就行, 然后代号入座就行
        mod = 10 ** 9 + 7
        return pow(2, n + m - 1, mod) * pow(x//2, n * m, mod) % mod
```

---

# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20240108/446702330_1704677402932/D2B5CA33BD970F64A6301FA75AE2EB22)
