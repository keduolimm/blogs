# 第 372 场周赛 解题报告 | 珂学家 | 贪心构造 + 单调栈上二分


---

# 前言

![image.png](https://pic.leetcode.cn/1700371875-hudlhx-image.png)


---

# 整体评价

被这个T3难哭了，一度在求一元二次方的最值问题，后来想了下是否可以直接构造解，让左右两侧尽量相等，那么其值也最大。

T4好像又是一个典题，感觉解法很多，这边用了离线+单调栈上二分，可以借助treemap来模拟代替。


---

## [T1. 使三个字符串相等](https://leetcode.cn/contest/weekly-contest-372/problems/make-three-strings-equal/)

就是求s1, s2, s3的公共前缀 ps

- $如果 ps = 0, 则无解$

- $ps > 0, 则 s1 + s2 + s3 - 3 * ps$

```java
class Solution {
    
    // 公共前缀
    int commonPrefix(String a, String b, String c) {
        int mn = Math.min(Math.min(a.length(), b.length()), c.length());
        for (int i = 0; i < mn; i++) {
            char c1 = a.charAt(i), c2 = b.charAt(i), c3 = c.charAt(i);
            if (c1 != c2 || c1 != c3) return i; 
        }
        return mn;
    }
    
    public int findMinimumOperations(String s1, String s2, String s3) {
        int r = commonPrefix(s1, s2, s3);
        if (r == 0) return -1;
        return s1.length() + s2.length() + s3.length() - 3 * r;
    }
    
}
```

```python [] 
class Solution:
    def findMinimumOperations(self, s1: str, s2: str, s3: str) -> int:
        mp = defaultdict(int)
        for i in range(1, len(s1) + 1):
            mp[s1[:i]] += 1
          
        for i in range(1, len(s2) + 1):
            mp[s2[:i]] += 1
            
        for i in range(1, len(s3) + 1):
            mp[s3[:i]] += 1
            
        
        res = 10 ** 18
        for k, v in mp.items():
            if v == 3:
                res = min(res, len(s1) + len(s2) + len(s3) - 3 * len(k))
        
        return -1 if res == 10 ** 18 else res
```

---

## [T2. 区分黑球与白球](https://leetcode.cn/contest/weekly-contest-372/problems/separate-black-and-white-balls/)

交换题，不过这题很眼熟，感觉做好好几次了。

不需要模拟，也不需要真的去分治求逆序对

简单的统计，每个0前面的1的个数，然后累计即可，纯思维题。

因为0前1的个数，就是逆序数

```java
class Solution {
    
    public long minimumSteps(String s) {
        long res = 0;
        int n1 = 0;
        for (char c: s.toCharArray()) {
            if (c == '0') {
                res += n1;
            } else {
                n1++;
            }
        }    
        return res;
    }
    
}
```

```python [] 
class Solution:
    def minimumSteps(self, s: str) -> int:
        black = 0
        res = 0
        for i in range(len(s)):
            if s[i] == '0':
                res += black
            else:
                black += 1
                
        return res
```

---
## [T3. 最大异或乘积](https://leetcode.cn/contest/weekly-contest-372/problems/maximum-xor-product/)

如果位运算的题有姓名，一定是薛定谔家族

这边x有限定, ($0\le x \lt 2^n$)

$$ 求最大 (a \oplus x ) * (b \oplus x) $$

如果对于i位

- $a_i == b_i, 则可以构造x，是的left，right同时为1$

- $a_i \ne b_i, 没法同时使得left，right为1，但是保证必然一个1,一个0$

所以问题的核心是

$$a_i \ne b_i时，这个2^i该赋予给left，还是right？$$

可以猜个结论

$$左右两侧越接近，则(a \oplus x ) * (b \oplus x)最大。$$

这个结论，可以从构造一元二次方程求最值解去验证

贪心思路

- 从49位到0位遍历
- 如果在mask($2^n- 1$)范围外，各取a，b高位
- 如果在mask($2^n- 1$)范围内

    - $a_i==b_i$, 左右侧各取1
    - $a_i\ne b_i$, 左右侧小的那个取1


所以这题是基于贪心的构造解

```java []
class Solution {
    // 构造
    public int maximumXorProduct(long a, long b, int n) {
        long mod = 10_0000_0007l;
        
        long left = 0, right = 0;
        for (int i = 49; i >= 0; i--) {
            long t1 = a & (1l << i);
            long t2 = b & (1l << i);
            if (i >= n) {
                left |= t1;
                right |= t2;
            } else {
                if (t1 == t2) {
                    left |= (1l << i);
                    right |= (1l << i);
                } else {
                    if (left <= right) {
                        left |= (1l << i);
                    } else {
                        right |= (1l << i);
                    }
                }
            }
        }
        return (int)(((left % mod) * (right % mod)) % mod);
    }

}
```

```python
class Solution:
    def maximumXorProduct(self, a: int, b: int, n: int) -> int:
        
        c, d = 0, 0
        for i in range(50, -1, -1):
            if i >= n:
                c |= (a & (1 << i))
                d |= (b & (1 << i))
            else:
                t1 = (a >> i) & 1
                t2 = (b >> i) & 1
                if t1 == t2:
                    c |= (1 << i)
                    d |= (1 << i)
                else:
                    if c > d:
                        d |= (1 << i)
                    else:
                        c |= (1 << i)
        
        return c * d % (10 ** 9 + 7)
```

---

## [T4. 找到 Alice 和 Bob 可以相遇的建筑](https://leetcode.cn/contest/weekly-contest-372/problems/find-building-where-alice-and-bob-can-meet/)

经典题

多查询的题，可以尝试离线思路

离线往往和排序结合，这边按右侧节点从大到小排序

然后问题核心是，如何找到最左边的位置x

$$height[x] > max(height[i], height[j]),  x>i,x>j$$

可以维护一个严格递减单调栈，然后二分找最小的x

java可以借助treemap来模拟单调栈，主要是二分方便

```java
class Solution {

    public int[] leftmostBuildingQueries(int[] heights, int[][] queries) {

        int n = queries.length;
        int[] res = new int[n];

        List<int[]> arr = new ArrayList<>();
        for (int i = 0; i < heights.length; i++) {
            arr.add(new int[] {0, i, heights[i]});
        }
        for (int i = 0; i < n; i++) {            
            if (queries[i][0] == queries[i][1]) {
                res[i] = queries[i][1];
            } else {
                // swap
                if (queries[i][0] > queries[i][1]) {
                    int tmp = queries[i][0];
                    queries[i][0] = queries[i][1];
                    queries[i][1] = tmp;
                }
                if (heights[queries[i][1]] > heights[queries[i][0]]) {
                    res[i] = queries[i][1];
                } else { 
                    arr.add(new int[] {1, queries[i][1], i});
                }
            }
        }

        Collections.sort(arr, (a, b)-> {
            int ret = -Integer.compare(a[1], b[1]);
            if (ret != 0) return ret;
            return Integer.compare(a[0], b[0]);
        });

        // 使用treemap构建一个单调栈
        TreeMap<Integer, Integer> mono = new TreeMap<>();
        for (int[] op: arr) {
            int t = op[0];
            if (t == 0) {
                int id = op[1];
                int v = op[2];
                
                // 这边的while, 保证右侧<=都被剔除掉
                Map.Entry<Integer, Integer> ent = null;
                while ((ent = mono.floorEntry(v)) != null) {
                    mono.remove(ent.getKey());
                }
                mono.put(v, id);
            } else {
                // 查询
                int idx = op[2];
                int mh = Math.max(heights[queries[idx][0]], heights[queries[idx][1]]) + 1;
                var kv = mono.ceilingEntry(mh);
                if (kv == null) {
                    res[idx] = -1;
                } else {
                    res[idx] = kv.getValue();
                }
            }
        }

        return res;
    }

}
```


---

# 写在最后

![image.png](https://pic.leetcode.cn/1700371731-EWhNgC-image.png)






