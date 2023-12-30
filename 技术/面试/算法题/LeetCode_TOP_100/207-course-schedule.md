

[207. 课程表](https://leetcode.cn/problems/course-schedule/description)

```python []
class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:

        g = [[] for _ in range(numCourses)]
        deg = [0] * numCourses
        for e in prerequisites:
            g[e[1]].append(e[0])
            deg[e[0]] += 1

        deq = deque()
        for i in range(numCourses):
            if deg[i] == 0:
                deq.append(i)

        res = []
        while len(deq) > 0:
            u = deq.popleft()
            res.append(u)
            for v in g[u]:
                deg[v] -= 1
                if deg[v] == 0:
                    deq.append(v)
        
        return len(res) == numCourses

```