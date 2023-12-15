
# 第 373 场周赛 解题报告 | 珂学家 | 分组重排 + hash套hash(前缀和&同余)


----
# 前言

![image.png](https://pic.leetcode.cn/1700970919-DihcVg-image.png)

旅途太长，人生太短，清汤一盏敬过往。

---
# 整体评价

T3是分组重排的题，操作起来有点绕。 T4是道好题，能联想到-1,+1前缀和的转化，但是k的倍数，如何处理呢？ 关键点就在这，想明白了，就容易了。

---

## T1. 循环移位后的矩阵相似检查

模拟题

分奇偶讨论下即可

```java
class Solution {
    public boolean areSimilar(int[][] mat, int k) {
        int n = mat.length, m = mat[0].length;
        for (int i = 0; i < n; i++) {
            if (i % 2 == 0) {
                for (int j = 0; j < m; j++) {
                    if (mat[i][j] != mat[i][((j - k) % m + m) % m]) return false;
                }
            } else {
                for (int j = 0; j < m; j++) {
                    if (mat[i][j] != mat[i][(j + k) % m]) return false;
                }
            }
        }
        return true;
    }
}
```

---

## T2. 统计美丽子字符串 I

和T4，一起讲

---

## T3. 交换得到字典序最小的数组

如果 $|A - B| \le limit$，那A, B可以交换

假如 $|B - C| \le limit$，那B, C可以交换

进而推导A，C也可以交换，就算 $|A - C| \le limit 不成立$

这题的核心：

$$交换是可以传导的$$

可以把这些可交换的，归为一类，称为分组

如何让数组最小呢？ 就是分组后，把同组的最小值放在同组索引最小的。

因为这种关系，基于大小的，这边可以排序后，分组来简化。

如果这种关系和大小无关，那可能需要为引入并查集来处理。


```java
class Solution {

    public int[] lexicographicallySmallestArray(int[] nums, int limit) {

        // 分组 + 重排
        int n = nums.length;
        int[][] arr = new int[n][2];
        for (int i = 0; i < n; i++) {
            arr[i][0] = nums[i];
            arr[i][1] = i;
        }
        Arrays.sort(arr, Comparator.comparingInt(x -> x[0]));

        int[] res = new int[n];

        int j = 0;
        while (j < n) {
            List<Integer> index = new ArrayList<>();
            List<Integer> values = new ArrayList<>();

            // 进行分组提取
            values.add(arr[j][0]);
            index.add(arr[j][1]);
            int t = j + 1;
            while (t < n && arr[t][0] - arr[t - 1][0] <= limit) {
                values.add(arr[t][0]);
                index.add(arr[t][1]);
                t++;
            }
            
            // 分离后，重排
            Collections.sort(values);
            Collections.sort(index);
            for (int s = 0; s < values.size(); s++) {
                res[index.get(s)] = values.get(s);
            }
            j = t;
        }

        return res;
    }

}
```

---

## T4. 统计美丽子字符串 II

如果没有倍数k的限制

那这题是一道很典的题，即把元音字母视为+1, 非元音字母视为-1

这样 $vowels == consonants$，既可以转化为前缀和Hash模型

那现在的问题是

$$(vowels * consonants) \% k == 0$$

因为$vowels == consonants$两者相等，所以求的区间距离最小的$y=2x$, 使得$x^2 \% k == 0$

有了这个最小距离y后，可以在原先一维map中，再嵌套一个map，用于存储前缀长度对y的取模数

---

简单而言

$$元组 (前缀和，前缀长度取余) 为Key，相等key的value累加即为解$$


```java
class Solution {

    boolean isVowel(char c) {
        return "aeiou".indexOf(c) != -1;
    }

    public long beautifulSubstrings(String s, int k) {
        // -1, +1  =>
        char[] str = s.toCharArray();
        int n = str.length;

        // 最最小的基数x, 是的x^2 % k = 0,
        // offset需要乘以2， 因为它是x+x
        int offset = 0;
        for (int i = 1; i <= k; i++) {
            if ((i * i) % k == 0) {
                offset = i * 2;
                break;
            }
        }

        long res = 0;

        // hashmap嵌套hashmap
        Map<Integer, Map<Integer, Integer>> hash = new HashMap<>();
        hash.put(0, new HashMap<>());
        hash.get(0).put(0, 1);
        int acc = 0;
        for (int i = 0; i < n; i++) {
            if (isVowel(str[i])) {
                acc++;
            } else {
                acc--;
            }

            int key = (i + 1) % offset;
            if (hash.containsKey(acc)) {
                var hash2 = hash.get(acc);
                res += hash2.getOrDefault(key, 0);
            }

            hash.computeIfAbsent(acc, x -> new HashMap<>()).merge(key, 1, Integer::sum);
        }

        return res;
    }

}
```

---

# 写在最后

我对这个人根本就一无所知，仅仅是一起旅行了10年而已。

![image.png](https://pic.leetcode.cn/1700971276-XUlQLD-image.png)

