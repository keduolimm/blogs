
#

---

# 前言

![image.png](https://pic.leetcode.cn/1703346814-kfAfNc-image.png)

忘名可以再记，回忆永不再来

---

# 整体评价

好像有一段时间没写周赛题解了，^_^.

感觉今天手感特别好，下午的几场比赛，包括传智杯都能打出超神战绩。

T3这题属于前后缀拆解，然后单调栈上二分(可以引入哨兵机制)，感觉单调栈不太严谨，写起来有点变扭。

T4难道是传说中Dsu On Tree? 感觉有些像。

---

## [T1. 统计移除递增子数组的数目 I](https://leetcode.cn/contest/biweekly-contest-120/problems/count-the-number-of-incremovable-subarrays-i/)

和T3一起讲

---
## [T2. 找到最大周长的多边形](https://leetcode.cn/contest/biweekly-contest-120/problems/find-polygon-with-the-largest-perimeter/)

思路：贪心

猜了一个结论

$$ \sum_{j=0}^{j=i} arr[j] > arr[i+1]， 满足此条件的最大i $$

先对$arr$排序，逆序找到第一个$i$即可


```java
class Solution {

    public long largestPerimeter(int[] nums) {
        // 思维题
        long sum = 0;
        Arrays.sort(nums);
        for (int i = 0; i < nums.length; i++) {
            sum += nums[i];
        }

        // 逆序
        for (int i = nums.length - 1; i >= 2; i--) {
            sum -= nums[i];
            if (sum > nums[i]) {
                return sum + nums[i];
            }
        }
        return -1;
    }

}
```


---
## [T3. 统计移除递增子数组的数目 II](https://leetcode.cn/contest/biweekly-contest-120/problems/count-the-number-of-incremovable-subarrays-ii/)

思路： 前后缀拆解 + 单调栈上二分

因为题目要求最左侧和最右侧都严格递增，所以需要预处理前后缀，保证严格递增

从左往右枚举每个点v

- check后缀是递增的
- 寻找前缀构建的单调栈，且结尾小于v的点，累加数量
- 如果当前值前缀是递增的，则加入单调栈

```java
class Solution {

    public long incremovableSubarrayCount(int[] nums) {

        int n = nums.length;
        boolean[] pre = new boolean[n];
        boolean[] suf = new boolean[n];
        pre[0] = suf[n - 1] = true;
        for (int i = 1; i < n; i++) {
            pre[i] = pre[i - 1] && nums[i] > nums[i - 1];
        }
        for (int i = n - 2; i >= 0; i--) {
            suf[i] = suf[i + 1] && nums[i] < nums[i + 1];
        }

        long res = 0;
        // java可以用treemap来偷鸡单调栈monostack
        TreeMap<Integer, Integer> range = new TreeMap<>();
        range.put(-1, 1); // 哨兵
        for (int i = 0; i < n; i++) {
            int v = nums[i];
            if (suf[i]) {
                var ent = range.lowerEntry(v);
                if (ent != null) {
                    // 删除的子数组必要有1个元素，所以要分类讨论
                    if (ent.getValue() + 1 == i + 2) {
                        res += ent.getValue() - 1;
                    } else {
                        res += ent.getValue();
                    }
                }
            }
            if (pre[i]) {
                // 为啥要+2, 主要是为了统计方便
                range.put(v, i + 2);
            }
        }

        // 处理尾巴
        {
            var ent = range.lowerEntry(Integer.MAX_VALUE);
            if (ent != null) {
                if (ent.getValue() + 1 == n + 2) {
                    res += ent.getValue() - 1;
                } else {
                    res += ent.getValue();
                }
            }
        }

        return res;
    }

}
```

---

## [T4. 树中每个节点放置的金币数目](https://leetcode.cn/contest/biweekly-contest-120/problems/find-number-of-coins-to-place-in-tree-nodes/)

思路: 启发式合并

有一个结论：

$$如果一个序列arr,  arr[0], arr[1], ..., , arr[n - 2], arr[n - 1], 抽取其中3个数使其乘积最大$$

$$ 取arr的最小3个数， a_1, a_2, a_3 (a_1 \le a_2 \le a_3) $$

$$最大的3个数, b_1, b_2, b_3 (b_1 \le b_2 \le b_3) $$

$$ 乘积最大 = max(a_1 * a_2 * a_3, a_1 * a_2 * b_3, a_1 * b_2 * b_3, b_1 * b_2 * b_3) $$


有了这个结论后，剩下的就好办了

每个子节点再往上传的时候，只需要保留3个最小数，3个最大数即可。

而这点，就扣合本题的思路

$$启发式合并$$


```java
class Solution {
    int n;
    List<Integer>[]g;

    int[] cost;
    long[] res;

    List<Integer> []mins;
    List<Integer> []maxs;

    void dfs(int u, int fa) {

        List<Integer> tmpMin = new ArrayList<>();
        List<Integer> tmpMax = new ArrayList<>();
        tmpMin.add(cost[u]);
        tmpMax.add(cost[u]);
        for (int v: g[u]) {
            if (v == fa) continue;
            dfs(v, u);
            for (int tv: mins[v]) {
                tmpMin.add(tv);
            }
            for (int tv: maxs[v]) {
                tmpMax.add(tv);
            }
        }


        if (tmpMin.size() < 3) {
            res[u] = 1;
        } else {
            Collections.sort(tmpMin);
            Collections.sort(tmpMax);

            // 核心逻辑
            long ans = Long.MIN_VALUE / 10;
            long a1 = tmpMin.get(0), a2 = tmpMin.get(1), a3 = tmpMin.get(2);
            int nz = tmpMax.size();
            long a4 = tmpMax.get(nz - 3), a5 = tmpMax.get(nz - 2), a6 = tmpMax.get(nz - 1);

            ans = Math.max(ans, a1 * a2 * a3);
            ans = Math.max(ans, a1 * a2 * a6);
            ans = Math.max(ans, a1 * a5 * a6);
            ans = Math.max(ans, a4 * a5 * a6);

            if (ans < 0) {
                res[u] = 0;
            } else {
                res[u] = ans;
            }
        }

        // 保留3位，往上传
        for (int i = 0; i < 3 && i < tmpMin.size(); i++) {
            mins[u].add(tmpMin.get(i));
        }
        for (int i = tmpMax.size() - 1; i >= 0 && tmpMax.size() - i <= 3; i--) {
            maxs[u].add(tmpMax.get(i));
        }
    }


    public long[] placedCoins(int[][] edges, int[] cost) {
        n = cost.length;
        g = new List[n];
        this.cost = cost;
        Arrays.setAll(g, x->new ArrayList<>());
        for (int[] e: edges) {
            g[e[0]].add(e[1]);
            g[e[1]].add(e[0]);
        }

        mins = new List[n];
        Arrays.setAll(mins, x->new ArrayList<>());
        maxs = new List[n];
        Arrays.setAll(maxs, x->new ArrayList<>());


        res = new long[n];
        dfs(0, -1);

        return res;
    }

}
```

---

# 写在最后

即使是希望、即使是梦想，都是需要被守护的。

![image.png](https://pic.leetcode.cn/1703346895-DMphNR-image.png)




