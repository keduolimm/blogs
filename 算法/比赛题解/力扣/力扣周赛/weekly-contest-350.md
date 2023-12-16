# 第 350 场周赛 解题报告 | 珂学家 | 经典状压DP + 二维背包


---

[TOC]

---

# 前言

![image.png](https://pic.leetcode.cn/1687063463-TiTBoO-image.png)

我一直相信着的，不，我现在也相信的，今后也一样，你是我的英雄，无论何时都会来拯救我。


---

# 整体评价

T3很典，T4真心挺难的，总想着2分，但是毫无思路，好不容易发现有背包的线索，但时间复杂度让人后怕。

也许Java真的是力扣的宠儿，$O(n^3)$苟过，当然也欢迎 **Hack**

---

## [T1. 总行驶距离](https://leetcode.cn/contest/weekly-contest-350/problems/total-distance-traveled/)

这题可以借助，divmod来加速收敛， 当然也可以直接模拟(稳妥)

1. 每次计算 mainTank 可以换取多少副油箱的机会（mainTank / 5）
2. mainTank = min(mainTank / 5, additionalTank) + (mainTank % 5)

循环重复1,2，直到$mainTank<5$

这样的话，就算mainTank值很大，也很快速的收敛求解出来

```java
public class Solution {
    public int distanceTraveled(int mainTank, int additionalTank) {
        int res = 0;
        while (mainTank >= 5) {
            int t1 = mainTank / 5;
            mainTank = mainTank % 5;
            
            int d = Math.min(additionalTank, t1);
            additionalTank -= d;
            mainTank += d;
            
            res += t1 * 5 * 10;
        }
        return res + mainTank * 10;
    }
}
```


---

## T2. [找出分区值](https://leetcode.cn/problems/find-the-value-of-the-partition/)

把数组分成两个非空子数组，求$|max(arr1) - min(arr2)|$最小

这种很像对顶堆的形态， 所以直接猜测
> 对数组进行排序, 然后枚举相邻元素的差绝对值最小值

事实也是如此，可以用反证法，即 $a \in arr1 <= b \in arr2$ 不成立

那么交换这两个元素，会让 $|max(arr1) - min(arr2)|$ 更小

```java
class Solution {
    public int findValueOfPartition(int[] nums) {
        Arrays.sort(nums);
        return IntStream.range(0, nums.length - 1)
            .map(x -> nums[x + 1] - nums[x]).min().getAsInt();
    }
}
```


---

## [T3. 特别的排列](https://leetcode.cn/problems/special-permutations/)

经典状压题，这题真的太常见了

令 $opt[s][t]$, s为二进制表示的集合，t表示最后一个元素的索引

转移状态为

> ${opt[s][t] = \sum opt[s - (1 << t)][m], (m, t) \in s}$


```java
class Solution {
    
    public int specialPerm(int[] nums) {

        long mod = 10_0000_0007l;
        
        int n = nums.length;
        int range = 1 << n;
        // 状压
        long[][] opt = new long[range][n];
        
        for (int i = 0; i < n; i++) {
            opt[1 << i][i] = 1;
        }
        
        for (int i = 1; i < range; i++) {
            for (int k = 0; k < n; k++) {
                if ((i & (1 << k)) == 0) continue; // 末尾的元素需要包含在该集合S中
                if (opt[i][k] == 0) continue; // 额外加这个, 表示无意义的状态
                for (int j = 0; j < n; j++) {
                    // 如果这边的for要优化，可以借助预处理，提前构建乘除列表
                    if ((i & (1 << j)) == 0) {
                        if (nums[k] % nums[j] == 0 || nums[j] % nums[k] == 0) {
                            opt[i | (1 << j)][j] = (opt[i | (1 << j)][j] + opt[i][k]) % mod;
                        }
                    }
                }
            }
        }
        
        return (int)(Arrays.stream(opt[range - 1]).sum() % mod);

    }
    
}
```

---

## [T4. 给墙壁刷油漆](https://leetcode.cn/contest/weekly-contest-350/problems/painting-the-walls/)

这是一个有争议的解法，因为时间复杂度为$O(n^3)=(500^3=10^8)$, 不过中间有些常数级的优化。

令$opt[i][j][t], i为数组的前i个元素, j为选择了付费墙数量, t则是时间， 值为代价最小值$

那状态转移满足 **二维背包**

${opt[i][j][t] = min(opt[i - 1][j][t], opt[i - 1][j - 1][t - c_j] + cost[j])}$

由于是背包模型，所以可以对空间上进行降维处理，即$逆序遍历优化$

所以最终压缩为二维 $opt[j][t]$

最后的答案为 ${ans = min ({opt[j][t]}), 需要满足t \geq n - k的条件下}$

**优化点**

- 压缩时间技巧，就是把$t>=n$的时间点缩点为$n$, 这是最大的优化点
- 三重循环中，有效枚举j和t，尽量过滤掉无效状态

时间复杂度为$O(n^3)$

空间复杂度为$O(n^2)$

### 方法一: 二维背包

```java
class Solution {


    public int paintWalls(int[] cost, int[] time) {

        long inf = Long.MAX_VALUE / 10;

        int n = cost.length;
        int mz = Arrays.stream(time).sum();

        // 这个优化很关键，直接压缩了状态数量
        mz = Math.min(n, mz); 

        long ans = Arrays.stream(cost).sum();

        // 初始化状态
        long[][] opt = new long[n + 1][mz + 1];
        for (int i = 0; i <= n; i++) {
            Arrays.fill(opt[i], inf);
        }
        opt[0][0] = 0;

        // 这个acc也属于优化，常数级
        int acc = 0;
        for (int i = 0; i < n; i++) {
            int c = cost[i];
            int t = time[i];

            // 从i开始，也是一个常数级优化
            for (int k = i; k >= 0; k--) {
                for (int j = acc; j >= 0; j--) {
                    int nv = Math.min(j + t, mz);
                    if (opt[k + 1][nv] > opt[k][j] + c) {
                        opt[k + 1][nv] = opt[k][j] + c;
                    }
                }
            }

            // 保持在mz以内(500)
            acc += t;
            acc = Math.min(mz, acc);
        }

        for (int k = 0; k <= n; k++) {
            for (int j = 0; j <= mz; j++) {
                if (j >= n - k) {
                    ans = Math.min(ans, opt[k][j]);
                }
            }
        }

        return (int)ans;

    }

}
```

---
### 方法二: 特别的0-1背包

对于一面墙，可以选择付费，或者免费

这种2选1的情况，可以抽象为特别的0-1背包

令 $opt[i][t], i为前i个元素，t为累计时间值，值为代价最小$

${opt[i][t - 1] = min(opt[i][t - 1],\ opt[i - 1][t])}$, 选择作为免费使用
${opt[i][t + times[i]] = min(opt[i][t+times[i]],\ opt[i - 1][t] + cost[i])}$, 作为付费使用

当然这边需要注意
>  t可以为负值，而有效的是$t\geq0$的情况

为了实现这个，可以使用一个偏移量技巧，就是统一$+n$, 这样就有$[0, 2n]$共$2n+1$个有效状态

当然这个就不能在使用逆序降空间的技巧，不过滚动数组的优化依旧可以


```java
class Solution {

    public int paintWalls(int[] cost, int[] time) {

        // 这边要留有余地
        long inf = Long.MAX_VALUE / 10;

        int n = cost.length;

        int prev = 0, next = 1;
        long[][] opt = new long[2][2 * n + 1];

        // 初始状态
        Arrays.fill(opt[prev], inf);
        opt[prev][n] = 0;

        for (int i = 0; i < n; i++) {
            Arrays.fill(opt[next], inf);

            int c = cost[i], t = time[i];
            for (int j = 0; j <= 2 * n; j++) {
                int add = Math.min(2 * n, j + t);
                opt[next][add] = Math.min(opt[next][add], opt[prev][j] + c);

                int sub = Math.max(0, j - 1);
                opt[next][sub] = Math.min(opt[next][sub], opt[prev][j]);
            }
            prev = 1 - prev;
            next = 1 - next;
        }

        long ans = Long.MAX_VALUE;

        for (int i = n; i <= 2 * n; i++) {
            ans = Math.min(ans, opt[prev][i]);
        }

        return (int)ans;

    }

}
```

---

### 方法三: 限制最小容量的0-1背包

可以如下来定义

就是选择付费的墙，构建的序列

$T=[t_{a1},t_{a2}, ...t_{ak}]$, $a_i \in S$

其满足如下条件, 成本最小

> ${t_{a1} + t_{a2} + ... + t_{ak} >= n - |S|}$
> $|S|为集合\{a_i\}的大小k$

把 |S| 拆分为k个1

可以移项变形

> ${(t_{a1} + 1) + (t_{a2} + 1) + ... + (t_{ak} + 1) >= n}$

然后定义 ${t'_{ai} = t_{ai} + 1}$

那这题就演变成了

> ${(t'_1, cost_1), (t'_2, cost_2),...,(t'_n, cost_n)}$, 保证时间和不少于n的前提下，总代价最小.

然后用经典的0-1背包解法，来求解。


```java
class Solution {

    public int paintWalls(int[] cost, int[] time) {
        
        int n = cost.length;
        int inf = Integer.MAX_VALUE - 100_0000;
        
        int[] opt = new int[n + 1];
        Arrays.fill(opt, inf);
        opt[0] = 0;
        
        for (int i = 0; i < n; i++) {
            int t = time[i], c = cost[i];
            for (int j = n; j >= 0; j--) {
                int mz = Math.min(n, j + t + 1);
                if (opt[mz] > opt[j] + c) {
                    opt[mz] = opt[j] + c;
                }
            }
        }
        
        return opt[n];

    }

}
```

--- 

# 写在最后

人生不仅仅是为了自己而不停向前冲刺，把某人的幸福当成自己的幸福，还有这样的生活方式。

![image.png](https://pic.leetcode.cn/1687066543-jOdRzF-image.png)









