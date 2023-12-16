# 第 353 场周赛 解题报告 | 珂学家 | DP + 贪心差分验证 + 补一个LIS


---

# 前言
![image.png](https://pic.leetcode.cn/1688874343-SSpUJJ-image.png)

心有所向, 日复一日, 必有精进。

---

# 整体评价

还是DP场吧，T4是一个贪心驱动的差分验证题，T3其实很有趣，看成非连续的LIS, 泪眼汪汪止不住。连续的话，其实是求最大子数组和的DP解法。

其实还是T3，非连续的LIS有趣，这边提供一个解法，也不知道正确否?

---

## T1. 找出最大的可达成数字

也许是除”两数之和“后，最简单的周赛题了

```java
public class Solution {
    public int theMaximumAchievableX(int num, int t) { 
        return num + 2 * t;
    }
}
```

---

## [T2. 达到末尾下标所需的最大跳跃次数](https://leetcode.cn/problems/maximum-number-of-jumps-to-reach-the-last-index/)

DP 入门题，未来青蛙跳河系列，又多了一题。

因为n=1000, 所以$O(n^2)$可以接受。

```java
public class Solution {
    public int maximumJumps(int[] nums, int target) {
        int n = nums.length;
        int[] opt = new int[n];
        Arrays.fill(opt, Integer.MIN_VALUE / 10);
        
        opt[0] = 0;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (Math.abs(nums[j] - nums[i]) <= target) {
                    opt[j] = Math.max(opt[j], opt[i] + 1);
                }
            }
        }
        
        return opt[n - 1] > 0 ? opt[n - 1] : -1;   
    }
}
```

---

这边添加一个基于拓扑排序的DP解法

因为这个经典的有向无环图，最小最大问题都是可以**拓扑结构**这个框架来实现

```java
class Solution {

    public int maximumJumps(int[] nums, int target) {

        // 构建一个DAG (有向无环图)
        // 这题本质上就是, 基于拓扑排序的DP, 经典DP题

        int n = nums.length;
        List<Integer> []g = new List[n]; 
        Arrays.setAll(g, x -> new ArrayList<>());

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (Math.abs(nums[j] - nums[i]) <= target) {
                    g[i].add(j);
                }
            }
        }

        // 定义入度
        int[] deg = new int[n];
        // 定义可达性, 因为拓扑排序有多个入口（入度为0的点），但是这题要求必须从0节点出发
        boolean[] vis = new boolean[n];

        // 这个过程有点像筛表
        vis[0] = true;
        for (int i = 0; i < n; i++) {
            if (!vis[i]) continue;
            for (int v: g[i]) {
                deg[v]++;
                vis[v] = true;
            }
        }

        // 拓扑排序+DP
        int inf = Integer.MIN_VALUE;
        int[] opt = new int[n];
        Arrays.fill(opt, inf);

        Deque<Integer> deq = new LinkedList<>();
        deq.offer(0);
        opt[0] = 0;

        while (!deq.isEmpty()) {
            int u = deq.poll();
            for (int v: g[u]) {
                opt[v] = Math.max(opt[v], opt[u] + 1);
                if (--deg[v] == 0) {
                    deq.offer(v);
                }
            }
        }

        return opt[n - 1] > 0 ? opt[n - 1] : -1;

    }

}
```


---

## T3. 构造最长非递减子数组

2选1，从nums1,nums2数组中，然后构建**最长的连续的非递减子数组**

这种2选1，可以先做一个归一化，就是

> $nums1[i] = min(nums1[i], nums2[i])$
> $nums2[i] = max(nums1[i], nums2[i])$

这样做，可以方便很多

然后定义状态$opt[i][s]$, $i为前i元素，s\in (0,1)$

状态转移为

${opt[i][0] = max(1,\ opt[i - 1][0] + 1\ if\ nums1[i] \ge nums1[i - 1],  opt[i - 1][1] + 1\ if\ nums1[i] \ge nums2[i - 1])}$

${opt[i][1] = max(1,\ opt[i - 1][0] + 1\ if\ nums2[i] \ge nums1[i - 1],  opt[i - 1][1] + 1\ if\ nums2[i] \ge nums2[i - 1])}$

最终的结果为，dp求解过程中，维护最大的值

$ans = max(opt[i][0], opt[i][1])$

```java
class Solution {
    
    public int maxNonDecreasingLength(int[] nums1, int[] nums2) {

        int n = nums1.length;
        for (int i = 0; i < n; i++) {
            if (nums1[i] > nums2[i]) {
                int t = nums1[i];
                nums1[i] = nums2[i];
                nums2[i] = t;
            }
        }
        
        int[][] opt = new int[n][2];
        
        int ans = 1;
        opt[0][0] = 1;
        opt[0][1] = 1;
        
        for (int i = 1; i < n; i++) {
            opt[i][0] = opt[i][1] = 1;
            if (nums1[i] >= nums1[i - 1]) {
                opt[i][0] = Math.max(opt[i][0], opt[i - 1][0] + 1);
            }   
            if (nums1[i] >= nums2[i - 1]) {
                opt[i][0] = Math.max(opt[i][0], opt[i - 1][1] + 1);
            }
            
            if (nums2[i] >= nums1[i - 1]) {
                opt[i][1] = Math.max(opt[i][1], opt[i - 1][0] + 1);
            }   
            if (nums2[i] >= nums2[i - 1]) {
                opt[i][1] = Math.max(opt[i][1], opt[i - 1][1] + 1);
            }
            
            ans = Math.max(ans, Math.max(opt[i][0], opt[i][1]));
        }
        
        return ans;
    
    }
    
}
```

---

假如说，这题是非连续子数组，求最长串

看起来非常的像LIS, 但是呢，有没有那么裸，这个可以如何求解呢？

这里面也有技巧，就是归一化

因为是最长非递增串，这里面需要在分类讨论下

- 归一化, 使得 $nums1 \le nums2$
- 构建一个LIS的模板架子
- 顺序遍历数组(nums1, nums2)
    1. $if\ nums1[i] \lt nums2[i]$ 先添加 nums2[i], 在添加nums1[i] 到LIS数组中
    2. $if\ nums1[i] == nums2[i]$ 只添加 nums2[i]到LIS数组中

这里的顺序 和 相等情况下，很重要

```java
class Solution {
    
    void lis(List<Integer> arr, int v) {
        int l = 0, r = arr.size() - 1;
        while (l <= r) {
            int m = l + (r - l) / 2;
            int u = arr.get(m);
            if (u <= v) {
                l = m + 1;
            } else {
                r = m - 1;
            }
        }
        if (l >= arr.size()) {
            arr.add(v);
        } else {
            arr.set(l, v);
        }
    }

    public int maxNonDecreasingLength(int[] nums1, int[] nums2) {

        int n = nums1.length;
        for (int i = 0; i < n; i++) {
            int t0 = Math.min(nums1[i], nums2[i]);
            int t1 = Math.max(nums1[i], nums2[i]);
            nums1[i] = t0;
            nums2[i] = t1;
        }
        
        List<Integer> arr = new ArrayList<>();
        
        for (int i = 0; i < n; i++) {
            if (nums1[i] == nums2[i]) {
                // 只加一次，因为非递增
                lis(arr, nums1[i]);
            } else {
                // 先加大的，再加小的，可以想想为什么
                lis(arr, nums2[i]);
                lis(arr, nums1[i]);
            }
        }
        
        return arr.size();
        
    }

}
```

---

## [T4. 使数组中的所有元素都等于零](https://leetcode.cn/contest/weekly-contest-353/problems/apply-operations-to-make-all-array-elements-equal-to-zero/)

经典的贪心，然后差分验证

删除操作，可以从左端贪心实现，然后维护一个差分即可。

> 最后的k-1元素，其实是用于验证用的，需要保证 $nums[i]+diff[i] 恒等于 0$, $i \ge k$

```java
class Solution {

    public boolean checkArray(int[] nums, int k) {

        int n = nums.length;
        long[] diff = new long[n + 1];

        long acc = 0;
        for (int i = 0; i <= n - k; i++) {
            // 这是当前值
            long now = acc + diff[i] + nums[i];
            if (now < 0) return false;

            // 差分构建
            diff[i] -= now;
            diff[i + k] += now;

            // 维护差分的前缀和
            acc += diff[i];
        }

        // 验证部分
        for (int i = n - k + 1; i < n; i++) {
            acc += diff[i];
            if (acc + nums[i] != 0) return false;
        }
        return true;
    }

}

```

---

加一个树状数组版本，主要这种写法比差分数组的方式更直观一点

```java
class Solution {
        
    public boolean checkArray(int[] nums, int k) {
        
        int n = nums.length;
        BIT bit = new BIT(n);
        for (int i = 0; i <= n - k; i++) {
            long d = bit.query(i + 1); // 从1开始
            long now = d + nums[i];
            if (now < 0) return false;
            if (now > 0) {
                bit.update(i + 1, -now);
                bit.update(i + 1 + k, now);
            }
        }
        
        for (int i = n - k + 1; i < n; i++) {
            long d = bit.query(i + 1); // 从1开始
            long now = d + nums[i];
            if (now != 0) return false;
        }
        return true;
    }
    
    static class BIT {
        int n;
        long[] arr;
        public BIT(int n) {
            this.n = n;
            this.arr = new long[n + 1];
        }
        long query(int p) {
            long res = 0;
            while (p > 0) {
                res += arr[p];
                p -= lowbit(p);
            }
            return res;
        }
        
        void update(int p, long d) {
            while (p <= n) {
                arr[p] += d;
                p += lowbit(p);
            }
        }
        
        int lowbit(int x) {
            return x & -x;
        }        
    }
    
}
```


---

# 写在最后

剑光如我，斩尽芜杂！

剑出，影随!

![image.png](https://pic.leetcode.cn/1688874366-PtEKvD-image.png)
