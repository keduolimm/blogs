

---

# 前言

---

# 整体评价

又是一场手速赛，是换出题人，还是快元旦了，happy year!

T4是二进制枚举集合+floyd最短路板子，T3是滑窗，但是T2挺特别的，算是一个思维题。

---

## [T1. 找到两个数组中的公共元素](https://leetcode.cn/contest/biweekly-contest-119/problems/find-common-elements-between-two-arrays/)

写个java stream玩玩，属于简单的模拟签到题

```java
class Solution {
    public int[] findIntersectionValues(int[] nums1, int[] nums2) {
        Set<Integer> h1 = Arrays.stream(nums1).mapToObj(Integer::valueOf).collect(Collectors.toSet());
        Set<Integer> h2 = Arrays.stream(nums2).mapToObj(Integer::valueOf).collect(Collectors.toSet());
        
        return new int[] {
            (int)Arrays.stream(nums1).filter(x-> h2.contains(x)).count(),
            (int)Arrays.stream(nums2).filter(x-> h1.contains(x)).count()
        };
    }
}
```

```python []
class Solution:
    def findIntersectionValues(self, nums1: List[int], nums2: List[int]) -> List[int]:
        s1 = set(nums1)
        s2 = set(nums2)
        
        a = sum([1 if v in s2 else 0 for v in nums1])
        b = sum([1 if v in s1 else 0 for v in nums2])
        return [a, b]
```

---

## [T2. 消除相邻近似相等字符](https://leetcode.cn/contest/biweekly-contest-119/problems/remove-adjacent-almost-equal-characters/)

状态机DP

令opt[i][j], 前i项以j结尾的最小代价(j为字母集合大小26)

$$opt[i][j] = min(opt[i - 1][j - 1], opt[i - 1][i] , opt[i - 1][j + 1]) + cost(str[i], j)$$

$$代价函数 cost(str[i], j) = (str[i] == j) ? 0 : 1$$

这样时间复杂度为$O(26^2 * n)$

```java
class Solution {
    
    public int removeAlmostEqualCharacters(String word) {
        char[] str = word.toCharArray();
        int n = str.length;
        
        int inf = Integer.MAX_VALUE / 10;
        int[][] dp = new int[n][26];
        for (int i = 0; i < 26; i++) {
            dp[0][i] = (i == str[0] - 'a' ? 0 : 1);
        }
        
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < 26; j++) {
                dp[i][j] = inf;
                for (int t = 0; t < 26; t++) {
                    if (Math.abs(j - t) > 1) {
                        dp[i][j] = Math.min(dp[i][j], dp[i - 1][t] + (j == str[i] - 'a' ? 0 : 1));
                    }
                }
            }
        }
        
        return Arrays.stream(dp[n - 1]).min().getAsInt();
    }
    
}
```

```python []
class Solution:
    def removeAlmostEqualCharacters(self, word: str) -> int:
        n = len(word)
        dp0 = [inf] * n
        dp1 = [inf] * n
        
        dp0[0] = 0
        dp1[0] = 1
        
        for i in range(1, n):
            if abs(ord(word[i]) - ord(word[i - 1])) > 1:
                dp0[i] = min(dp0[i - 1], dp1[i - 1])
                dp1[i] = min(dp0[i - 1], dp1[i - 1]) + 1
            else:
                dp0[i] = dp1[i - 1]
                dp1[i] = min(dp0[i - 1], dp1[i - 1]) + 1
                
        return min(dp0[n - 1], dp1[n - 1])
```

当然这里也有额外的技巧，可以把时间复杂度降到$O(n)$

---

## [T3. 最多 K 个重复元素的最长子数组](https://leetcode.cn/contest/biweekly-contest-119/problems/length-of-longest-subarray-with-at-most-k-frequency/)

思路: 双指针

经典的双指针解法，求最长的满足要求的子数组

```java
class Solution {

    public int maxSubarrayLength(int[] nums, int k) {

        int n = nums.length;
        int res = 0;
        
        Map<Integer, Integer> hash = new HashMap<>();
        int j = 0;
        for (int i = 0; i < n ;i++) {
            hash.merge(nums[i], 1, Integer::sum);
            // 窗口内是满足需求的
            while (j < i && hash.getOrDefault(nums[i], 0) > k) {
                hash.put(nums[j], hash.getOrDefault(nums[j], 0) - 1);
                j++;
            }
            res = Math.max(res, i - j + 1);
        }
        
        return res;
    }
    
}
```

```python []
class Solution:
    def maxSubarrayLength(self, nums: List[int], k: int) -> int:
        c = Counter()
        j = 0
        
        res = 0
        for i in range(len(nums)):
            c[nums[i]] += 1
            while j <= i and c[nums[i]] > k:
                c[nums[j]] -= 1
                j += 1
            res = max(res, i - j + 1)
            
        return res
```

---

## [T4. 关闭分部的可行集合数目](https://leetcode.cn/contest/biweekly-contest-119/problems/number-of-possible-sets-of-closing-branches/)

这题的关键是n=10

所以一种核心的思维是，二进制枚举集合

然后使用floyd计算所有点之间的最短路径

这样总的时间复杂度为 $O(2^n * n^3)$

如果n很大，感觉真的无从下手，挺难的

```java
class Solution {
    
    public int numberOfSets(int n, int maxDistance, int[][] roads) {
        
        int res = 0;
        
        int inf = Integer.MAX_VALUE / 10;
        int range = 1 << n;
        for (int i = 0; i < range; i++) {
            int[][] path = new int[n][n];
            for (int j = 0; j < n; j++) {
                Arrays.fill(path[j], inf);
                path[j][j] = 0;
            }
            for (int[] e: roads) {
                // 过滤掉不在范围内的边
                if ((i & (1 << e[0])) == 0 || (i & (1 << e[1])) == 0) continue;
                // 存在重边
                path[e[0]][e[1]] = Math.min(path[e[0]][e[1]], e[2]);
                path[e[1]][e[0]] = Math.min(path[e[1]][e[0]], e[2]);
            }
            
            for (int k = 0; k < n; k++) {
                for (int s = 0; s < n; s++) {
                    for (int e = 0; e < n; e++) {
                        if (path[s][e] > path[s][k] + path[k][e]) {
                            path[s][e] = path[s][k] + path[k][e];
                        }
                    }
                }
            }
                    
            boolean ok = true;
            for (int s = 0; s < n; s++) {
                for (int e = 0; e < n; e++) {
                    // 过滤掉不在范围内的边
                    if ((i & (1 << s)) == 0 || (i & (1 << e)) == 0) continue;
                    if (path[s][e] > maxDistance) ok = false;
                }
            }
            
            if (ok) res++;
        }
        
        return res;
    }
    
}
```

```python []
class Solution:
    def numberOfSets(self, n: int, maxDistance: int, roads: List[List[int]]) -> int:
        mask = 1 << n
        res = 0
        for s in range(0, mask):
            path = [[inf] * n for _ in range(n)]
            for j in range(n):
                path[j][j] = 0
            for u, v, c in roads:
                if ((1 << u) & s) != 0 and ((1 << v) & s) != 0:
                    path[u][v] = min(path[u][v], c)
                    path[v][u] = min(path[v][u], c)
                
            for k in range(n):
                for i in range(n):
                    for j in range(n):
                        if path[i][j] > path[i][k] + path[k][j]:
                            path[i][j] = path[i][k] + path[k][j]
            
            ok = True
            for i in range(n):
                if ((1 << i) & s) == 0:
                    continue
                for j in range(n):
                    if i == j or ((1 << j) & s) == 0:
                        continue
                    if path[i][j] > maxDistance:
                        ok = False
            if ok:
                res += 1
                
            # print (s, ok)
                
        return res
```

---

# 写在最后

![](./images/119r.jpeg)