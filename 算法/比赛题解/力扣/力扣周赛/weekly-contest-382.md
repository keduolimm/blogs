
# 第 382 场周赛 解题报告 | 珂学家 | 贪心构造

# 前言


---
# 整体评价

前三题还是蛮简单的，但是T4真的难，难在思维

---

## [T1. 按键变更的次数](https://leetcode.cn/contest/weekly-contest-382/problems/number-of-changing-keys/)

思路: 模拟

可以先归一化，即全部小写

```java []
class Solution {
    public int countKeyChanges(String s) {
        char[] str = s.toLowerCase().toCharArray();
        int res = 0;
        for (int i = 1; i < str.length; i++) {
            if (str[i] != str[i - 1]) res++;
        }
        return res;
    }
}
```

---
## [T2. 子集中元素的最大数量](https://leetcode.cn/contest/weekly-contest-382/problems/find-the-maximum-number-of-elements-in-subset/)

思路: 记忆化搜索

需要特判1，比较特殊

```java []
class Solution {
    Map<Long, Integer> hash = new HashMap<>();
    Map<Long, Integer> memo = new HashMap<>();

    int dfs(long v) {            
        if (memo.containsKey(v)) {
            return memo.get(v);
        }

        int cnt = hash.getOrDefault(v, 0);
        if (cnt == 0) {
            memo.put(v, 0);
            return 0;
        }
        if (cnt == 1) {
            memo.put(v, 1);
            return 1;
        }
        int r = dfs(v * v) + 1;
        memo.put(v, r);

        return r;
    }

    public int maximumLength(int[] nums) {
        for (int v: nums) {
            hash.merge((long)v, 1, Integer::sum);
        }
        int res = 0;
        for (int v: nums) {
            if (v != 1) {
                int tmp = dfs(v);
                res = Math.max(res, tmp * 2 - 1);
            } else {
                int tmp = hash.getOrDefault(1l, 0);
                if (tmp % 2 == 0) tmp--;

                res = Math.max(res, tmp);

            }
        }
        return res;
    }
}
```

---

## [T3. Alice 和 Bob 玩鲜花游戏](https://leetcode.cn/contest/weekly-contest-382/problems/alice-and-bob-playing-flower-game/)

思路: 找规律

因为每次只能取1个，所以只要 

$$x+y为奇数, 则必赢$$

$$n//2 * (m + 1) // 2 + (n+1)//2 * m // 2$$

```java []
class Solution {
    public long flowerGame(int n, int m) {
        long n1 = n, n2 = m;
        return (n1/2) * ((n2 + 1)/2) + ((n1+1)/2) * (n2/2);
    }
}
```

---

## [T4. 给定操作次数内使剩余元素的或值最小](https://leetcode.cn/contest/weekly-contest-382/problems/minimize-or-of-remaining-elements-using-operations/)

思路: 贪心构造

很怕这类题，就但是思维构造题

就是从构造出发，贪心消去更多的高位，其次消去更多的1

这边有个技巧，叫做试填法

中间的那个分段构造贪心，也是最难的点，往往很容易想到从高位到低位的构造。

整体时间复杂度为$O(30n)$

```java []
class Solution {
    
    public int minOrAfterOperations(int[] nums, int k) {
        int mask = (1 << 30) - 1;
        int ans = mask;
        for (int i = 29; i >= 0; i--) {
            int cur = ans - (1 << i);
            int s = mask;
            
            int cnt = nums.length;
            for (int v: nums) {
                s &= (v & ~cur);
                if (s == 0) {
                    cnt--;
                    s = mask;
                }
            }
            if (cnt <= k) {
                ans = cur;
            }
        }
        return ans;
    }
    
}
```
---
# 写在最后

