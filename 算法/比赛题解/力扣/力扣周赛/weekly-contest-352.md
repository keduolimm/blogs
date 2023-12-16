# 第 352 场周赛 解题报告 | 珂学家 | 滑窗 + 数组上并查集


---
# 前言

![image.png](https://pic.leetcode.cn/1688275161-gXWkSo-image.png)


觉得自己是正确的，就要大声地说出来坚决地去行动——这是我一直以来都贯彻的人生理念。

---

# 整体评价

这场周赛，感觉是滑窗计数专场，当然不同的题目，不同的数据规模，给予不同的算法可行性.

T3是经典滑窗题， T4虽然我更愿意称呼它为数组上的并查集，但实际上全文不见并查集，但其独立集合的思想永存。

最后，恭喜式酱老师，获得Rank 1.

## [T1. 最长奇偶子数组](https://leetcode.cn/contest/weekly-contest-352/problems/longest-even-odd-subarray-with-threshold/)

因为数据范围小，枚举左右端点即可， 这样大概是$O(n^3)$

当然也可以固定左端点，然后右侧滑窗，这样大概是$O(n^2)$

```java
class Solution {
    public int longestAlternatingSubarray(int[] nums, int threshold) {
        
        int ans = 0;

        for (int i = 0; i < nums.length; i++) {
            if (nums[i] % 2 != 0 || nums[i] > threshold) continue;
            
            ans = Math.max(ans, 1);
            
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[j] % 2 != nums[j - 1] % 2 && nums[j] <= threshold) {
                    ans = Math.max(ans, j - i + 1);
                } else {
                    break;
                }
            }
        }
        return ans;
    }
}
```

确实有$O(n)$的方法，因为它呈现的是一段一段区间，中间没有交叉

```java
class Solution {
    public int longestAlternatingSubarray(int[] nums, int threshold) {
        int ans = 0;
        int i = 0;
        while (i < nums.length) {
            int v = nums[i];
            if (v > threshold || v % 2 == 1) {
                i++;
                continue;
            }
            
            int j = i + 1;
            while (j < nums.length && nums[j] % 2 != nums[j - 1] % 2 && nums[j] <= threshold) {
                j++;
            }
            
            ans = Math.max(ans, j - i);
            
            i = j;
        }
        return ans;
    }
}
```

---

## [T2. 和等于目标值的质数对](https://leetcode.cn/problems/prime-pairs-with-target-sum/)

这边有个优化的技巧，就是预处理（静态初始化)，生成质数表

然后遍历枚举x，寻找y=n-x, 是否为质数

```java
class Solution {
    
    static final int MAX_SIZE = 100_0000;
    static boolean ok = false;
    static boolean[] isPrimes = new boolean[MAX_SIZE + 1];
    static List<Integer> primes = new ArrayList<>();
    
    public static void initialize() {
        if (ok) return;
        
        Arrays.fill(isPrimes, true);
        isPrimes[0] = isPrimes[1] = false; 

        for (int i = 2; i <= MAX_SIZE; i++) {
            if (isPrimes[i]) {
                primes.add(i);
                for (long j = (long)i * i; j <= MAX_SIZE; j+=i) {
                    isPrimes[(int)j] = false;
                }
            } 
        }
        ok = true;
    }
     
    public List<List<Integer>> findPrimePairs(int n) {
        initialize();
        List<List<Integer>> result = new ArrayList<>();
        for (int v: primes) {
            if (v >= n || n - v < v) break;
            int v2 = n - v;
            if (isPrimes[v2]) {
                result.add(List.of(v, v2));
            }
        }
        return result;
    }
    
}
```
---

## T3. 不间断子数组

经典滑窗题，因为是连续子数组，可以遍历右端点，固定右端点，然后统计满足要求个数

双指针解法

当然这边的窗口判定，有两种思路

- 在区间中 寻找是否有$>nums[j] + 2$ 或者${<nums[j] - 2}$的数存在
- 在区间中，统计满足要求的$nums[j]-2<=x<=nums[j]+2$的数量s，然后补集$T=区间长度 - 满足个数$

采用补集思路

```java
class Solution {

    // 固定右端点
    public long continuousSubarrays(int[] nums) {

        // 滑窗
        Map<Integer, Integer> hash = new HashMap<>();

        long ans = 0;

        int j = 0;
        for (int i = 0; i < nums.length; i++) {
            hash.merge(nums[i], 1, Integer::sum);

            int cnt = i - j + 1;
            int tmp = 0;
            for (int x = nums[i] - 2; x <= nums[i] + 2; x++) {
                tmp += hash.getOrDefault(x, 0);
            }

            while (cnt - tmp > 0) {
                hash.merge(nums[j], -1, Integer::sum);
                if (Math.abs(nums[j] - nums[i]) <= 2) tmp--;
                cnt--;
                j++;
            }

            ans += (i - j + 1);
        }


        return ans;

    }

}
```

---

非补集的话，可借助TreeSet的lower,high接口来实现，也非常的方便

```java
class Solution {

    // 固定右端点
    public long continuousSubarrays(int[] nums) {

        // 滑窗
        TreeMap<Integer, Integer> hash = new TreeMap<>();

        long ans = 0;

        int j = 0;
        for (int i = 0; i < nums.length; i++) {
            hash.merge(nums[i], 1, Integer::sum);
            while (j < i && hash.lowerKey(nums[i] - 2) != null || hash.higherKey(nums[i] + 2) != null) {
                hash.computeIfPresent(nums[j], (a, b) -> b > 1 ? b - 1 : null);
                j++;
            }            
            ans += (i - j + 1);
        }


        return ans;

    }

}
```

---

## [T4. 所有子数组中不平衡数字之和](https://leetcode.cn/contest/weekly-contest-352/problems/sum-of-imbalance-numbers-of-all-subarrays/)

滑窗应用题的，也是先固定左端点，然后右侧滑窗。

那这边如何维护一个数据结构，能够方便地统计不平衡值的数。

其实可以观察发现

> 中间插入的数，已存在，不会改变当前的任何不平衡数量
> 中间插入一个数，会破坏原先其左侧，右侧的不平衡值
> 中间插入一个数，又会和左侧，右侧建立新的不平衡值(只要不相邻)

而不平衡度数，最多因这个值的插入，有增量变动。

所以这个是增量维护的过程，通过寻找左右侧，则是TreeSet的擅长领域。


```java
class Solution {

        
    public int sumImbalanceNumbers(int[] nums) {

        // 1000^2 * log(1000)
        int n = nums.length;

        int ans = 0;
        for (int i = 0; i < nums.length; i++) {

            int diff = 0;

            TreeSet<Integer> ts = new TreeSet<>();
            ts.add(nums[i]);

            for (int j = i + 1; j < nums.length; j++) {
                
                if (ts.contains(nums[j])) {
                    // 如果存在了，不会造成变化
                } else {   
                    // 寻找左右邻
                    Integer key1 = ts.lower(nums[j]);
                    Integer key2 = ts.higher(nums[j]);

                    if (key1 != null && key2 !=null) {
                        diff--;
                    }
                    if (key1 != null && nums[j] > key1 + 1) {
                        diff++;
                    }
                    if (key2 != null && key2 - nums[j] > 1) {
                        diff++;
                    }
                }

                ts.add(nums[j]);
                ans += diff;
            }

        }

        return ans;

    }

}
```

但是这个解法是$O(n^2logn)$, 如果OJ严格一点，会被卡掉。

那如何优化呢？

可能需要继续的挖掘，我们把相邻的元素，合并为同一个集合的，

因为在数组上，相当于满足不同集合，其距离差$s>1$

如果集合数为$m$,那满足的不平衡数，一定为$m -1$

那集合数，不就是并查集施展才华的地方吗？

但实际上，这题，可以绕过并查集，简单计数即可，但它的思想是指导这题的关键。

最终的时间复杂度为$O(n^2)$

```java
class Solution {

    public int sumImbalanceNumbers(int[] nums) {

        int n = nums.length;

        int ans = 0;
        for (int i = 0; i < nums.length; i++) {

            boolean[] hash = new boolean[n + 2];
            
            int grp = 1;
            hash[nums[i]] = true;

            for (int j = i + 1; j < nums.length; j++) {
                
                if (hash[nums[j]]) {
                    
                } else {
                    hash[nums[j]] = true;

                    if (hash[nums[j] - 1] && hash[nums[j] + 1]) {
                        grp--;
                    } else if (hash[nums[j] - 1]) {
                    } else if (hash[nums[j] + 1]) {
                    } else {
                        grp++;
                    }
                }
                ans += (grp - 1);

            }
        }

        return ans;
    }

}
```
---

附带并查集版本

```java
class Solution {
    
    static class Dsu {
        int n;
        int[] arr;
        
        Set<Integer> groups = new TreeSet<>();
        
        public Dsu(int n) {
            this.n = n;
            this.arr = new int[n + 1];
            Arrays.fill(arr, -2); // -2 表示不存在的点， -1表示存在的值
        }
        
        int find(int u) {
            if (arr[u] == -1) return u;
            return arr[u] = find(arr[u]);
        }
        
        void union(int u, int v) {
            int ai = find(u);
            int bi = find(v);
            if (ai != bi) {
                arr[ai] = bi;
                groups.remove(ai);
            }
        }
        
        void enable(int u) {
            if (arr[u] == -2) {
                // 还没被激活过
                arr[u] = -1;
                groups.add(u);
                
                if (arr[u - 1] != -2) union(u - 1, u);
                if (arr[u + 1] != -2) union(u + 1, u);
            }
        }
        
        int groupNum() {
            return groups.size();
        }
        
    }
    
    
    public int sumImbalanceNumbers(int[] nums) {
        int ans = 0;
        
        int n = nums.length;
        for (int i = 0; i < n; i++) {
            Dsu dsu = new Dsu(n + 1);
            dsu.enable(nums[i]);
            for (int j = i + 1; j < n; j++) {
                dsu.enable(nums[j]);
                ans += dsu.groupNum() - 1; 
            }
        }
        return ans;
    }
    
}
```

---


# 写在最后

没关系，继续前进吧。这种程度的雪，算不了什么。

![image.png](https://pic.leetcode.cn/1688275215-xBwkBN-image.png)




