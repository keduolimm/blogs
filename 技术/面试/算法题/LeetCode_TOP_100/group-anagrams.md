

[49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/description/)

```python []
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        func = lambda x: ''.join(sorted(x))
        mp = defaultdict(list)
        for w in strs:
            mp[func(w)].append(w)
        return list(mp.values())
```

python对字符串友好


