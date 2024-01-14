
# Z函数

```java []
String s = b + "#" + a;
````

返回的是 S和S[i:n] 的最长公共前缀长度

```java []

// z函数模板, https://oi-wiki.org/string/z-func/
static int[] zFunction(String a) {
    char[] s = a.toCharArray();
    int n = s.length;
    int[] z = new int[n];
    z[0] = n;
    for (int i = 1, l = 0, r = 0; i < n; ++i) {
        if (i <= r && z[i - l] < r - i + 1) {
            z[i] = z[i - l];
        } else {
            z[i] = Math.max(0, r - i + 1);
            while (i + z[i] < n && s[z[i]] == s[i + z[i]]) ++z[i];
        }
        if (i + z[i] - 1 > r) {
            l = i;
            r = i + z[i] - 1;
        }
    }
    return z;
}

```