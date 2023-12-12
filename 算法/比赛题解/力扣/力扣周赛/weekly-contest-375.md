
---

# 前言

---

# 整体评价

难得的手速场，这几题都比较套路，确实区间合并很久没考察到了。

不过T4有多种解，栈模拟/差分/链式并查集，都可以的。

---

## [T1. 统计已测试设备](https://leetcode.cn/contest/weekly-contest-375/problems/count-tested-devices-after-test-operations/)

思路: 差分思维

```java
class Solution {
    public int countTestedDevices(int[] batteryPercentages) {
        // 采用类似差分的思想
        int ans = 0;
        for (int v: batteryPercentages) {
            if (v - ans > 0) {
                ans++;
            }
        }
        return ans;
    }
}
```

---
## [T2. 双模幂运算](https://leetcode.cn/contest/weekly-contest-375/problems/double-modular-exponentiation/)

思路：快速幂(带模数)

这题还是板子题

```java
class Solution {
    
    // 快速幂板子
    int ksm(int base, int v, int mod) {
        int r = 1;
        while (v > 0) {
            if (v % 2 == 1) {
                r = r * base % mod;
            }
            v /= 2;
            base = base * base % mod;
        }
        return r;
    }
    
    public List<Integer> getGoodIndices(int[][] variables, int target) {
        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < variables.length; i++) {
            int[] v = variables[i];
            int a = v[0], b = v[1], c = v[2], m = v[3];
            if (ksm(ksm(a, b, 10), c, m) == target) {
                res.add(i);
            }
        }
        return res;
    }
    
}
```

---
## [T3. 统计最大元素出现至少 K 次的子数组](https://leetcode.cn/contest/weekly-contest-375/problems/count-subarrays-where-max-element-appears-at-least-k-times/)

思路: 双指针

这题的关键是: **整个数组的最大元素**

也是比较套路的一道题

窗口内部维护不符合要求的区间，区间外都是符合的。

```java
class Solution {
    public long countSubarrays(int[] nums, int k) {
        long res = 0;
        
        int maxValue = Arrays.stream(nums).max().getAsInt();
        
        int j = 0;
        int nk = 0;
        // 枚举右端点
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == maxValue) {
                nk++;
            }
            
            // 滑动左侧端点
            while (j <= i && nk >= k) {
                if (nums[j] == maxValue) {
                    nk--;
                }
                j++;
            }
            
            // 这边都是有效的值
            res += j;
        }
        
        return res;
    }
}
```

---
## [T4. 统计好分割方案的数目](https://leetcode.cn/contest/weekly-contest-375/problems/count-the-number-of-good-partitions/)

区间合并的裸题

不过这里有好几种思路

- 排序 + 栈模拟合并
  - 时间复杂度为$O(nlogn)$
- 差分模拟
  - 时间复杂度为$O(n)$
- 链式并查集
  - 时间复杂度为$O(n)$

### 1. 排序 + 栈模拟合并
```java
class Solution {

    // 快速幂
    long ksm(long base, long v, long mod) {
        long r = 1;
        while (v > 0) {
            if (v % 2 == 1) {
                r = r * base % mod;
            }
            v /= 2;
            base = base * base % mod;
        }
        return r;
    }

    public int numberOfGoodPartitions(int[] nums) {

        // 区间构造
        Map<Integer, int[]> hash = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int v = nums[i];
            if (hash.containsKey(v)) {
                hash.get(v)[1] = i;
            } else {
                hash.put(v, new int[] {i, i});
            }
        }

        // 区间按左端点排序
        List<int[]> intervals = hash.values().stream().sorted(Comparator.comparing(x -> x[0])).collect(Collectors.toList());

        // 利用堆栈进行合并操作
        Deque<int[]> stack = new ArrayDeque<>();
        for (int[] xy: intervals) {
            int s = xy[0], e = xy[1];
            while (!stack.isEmpty() && stack.peek()[1] >= s) {
                int[] tmp = stack.pop();
                e = Math.max(tmp[1], e);
            }
            stack.push(new int[] {s, e});
        }

        // 独立的区间数
        int m = stack.size();
        long mod = (long)1e9 + 7;
        return (int)ksm(2, m - 1, mod);
    }

}
```
---
### 2. 差分模拟

这边的差分模拟，终点有点特别，它是明确统计一个区间的最后的端点来计数的

```java
class Solution {

    // 快速幂
    long ksm(long base, long v, long mod) {
        long r = 1;
        while (v > 0) {
            if (v % 2 == 1) {
                r = r * base % mod;
            }
            v /= 2;
            base = base * base % mod;
        }
        return r;
    }

    public int numberOfGoodPartitions(int[] nums) {

        // 区间构造
        Map<Integer, int[]> hash = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int v = nums[i];
            if (hash.containsKey(v)) {
                hash.get(v)[1] = i;
            } else {
                hash.put(v, new int[] {i, i});
            }
        }

        int n = nums.length;
        int[] diff = new int[n + 1];
        for (var kv: hash.entrySet()) {
            int[] xy = kv.getValue();
            diff[xy[0]]++;
            diff[xy[1]]--; // 注意没有偏移+1
        }
        
        // 独立的区间数
        // 差分还原
        int m = 0;
        int acc = 0;
        for (int i = 0; i < n; i++) {
            acc += diff[i];
            if (acc == 0) m++;
        }
        
        long mod = (long)1e9 + 7;
        return (int)ksm(2, m - 1, mod);
    }

}
```

### 3. 链式并查集

它也可以做到O(n)的时间复杂度

---
# 写在最后

![](./images/375b.jpg)

