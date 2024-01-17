
# KMP 模板

该模板来源自

https://oi-wiki.org/string/kmp/


```java []
// 来源自 https://oi-wiki.org/string/kmp/
class KMP {

    // *) kmp
    // 在text中，找到pattern出现的位置列表
    public static List<Integer> findOccurrences(String text, String pattern) {
        String cur = pattern + '#' + text;
        int sz1 = text.length(), sz2 = pattern.length();
        List<Integer> v = new ArrayList<>();
        int[] lps = prefixFunction(cur);
        for (int i = sz2 + 1; i <= sz1 + sz2; i++) {
            if (lps[i] == sz2) v.add(i - 2 * sz2);
        }
        return v;
    }

    private static int[] prefixFunction(String ss) {
        char[] s = ss.toCharArray();
        int n = s.length;
        int[] pi = new int[n];
        for (int i = 1; i < n; i++) {
            int j = pi[i - 1];
            while (j > 0 && s[i] != s[j]) j = pi[j - 1];
            if (s[i] == s[j]) j++;
            pi[i] = j;
        }
        return pi;
    }
    
}
```

# 题单

[3008. 找出数组中的美丽下标 II](https://leetcode.cn/problems/find-beautiful-indices-in-the-given-array-ii/description/)

```java []
class Solution {
    public List<Integer> beautifulIndices(String s, String a, String b, int k) {
        List<Integer> arr = KMP.findOccurrences(s, a);
        List<Integer> brr = KMP.findOccurrences(s, b);
        
        List<Integer> res = new ArrayList<>();
        int j = 0;
        for (int i = 0; i < arr.size(); i++) {
            while (j < brr.size() && brr.get(j) - arr.get(i) < -k) {
                j++;
            }
            if (j < brr.size() && Math.abs(arr.get(i) - brr.get(j)) <= k) {
                res.add(arr.get(i));
            }
        }
        return res;
    }
    
}

class KMP {

    // *) 
    public static List<Integer> findOccurrences(String text, String pattern) {
        String cur = pattern + '#' + text;
        int sz1 = text.length(), sz2 = pattern.length();
        List<Integer> v = new ArrayList<>();
        int[] lps = prefixFunction(cur);
        for (int i = sz2 + 1; i <= sz1 + sz2; i++) {
            if (lps[i] == sz2) v.add(i - 2 * sz2);
        }
        return v;
    }

    private static int[] prefixFunction(String ss) {
        char[] s = ss.toCharArray();
        int n = s.length;
        int[] pi = new int[n];
        for (int i = 1; i < n; i++) {
            int j = pi[i - 1];
            while (j > 0 && s[i] != s[j]) j = pi[j - 1];
            if (s[i] == s[j]) j++;
            pi[i] = j;
        }
        return pi;
    }
}
```
