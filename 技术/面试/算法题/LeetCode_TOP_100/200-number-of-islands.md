


[200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/description)

如何申明数组

```c++ 
int dirs[4][2] = {
    {1, 0}, {-1, 0}, {0, -1}, {0, 1}
};
```

bfs 借助 deque

```c++
deque<pair<int, int>> deq;
deq.push_back({y, x});
vis[y][x] = true;
while (!deq.empty()) {
    auto [ty, tx] = deq.front();
    deq.pop_front();
    // DO ....
}
```

---

```c++ []
class Solution {
public:

    // dirs数组定义
    int dirs[4][2] = {
        {1, 0}, {-1, 0}, {0, -1}, {0, 1}
    };

    int numIslands(vector<vector<char>>& grid) {
        int h = grid.size(), w = grid[0].size();
        vector<vector<bool>> vis(h, vector<bool>(w, false));

        // lambda函数
        auto bfs = [&](int y, int x) -> void {
            deque<pair<int, int>> deq;
            deq.push_back({y, x});
            vis[y][x] = true;
            while (!deq.empty()) {
                auto [ty, tx] = deq.front();
                deq.pop_front();
                for (int j = 0; j < 4; j++) {
                    int ny = ty + dirs[j][0];
                    int nx = tx + dirs[j][1];
                    if (ny >= 0 && ny < h && nx >= 0 && nx < w) {
                        if (grid[ny][nx] == '1' && !vis[ny][nx]) {
                            vis[ny][nx] = true;
                            deq.push_back({ny, nx});
                        }
                    }
                }
            }
        };

        int res = 0;
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (grid[i][j] == '1' && !vis[i][j]) {
                    bfs(i, j);
                    res++;
                }
            }
        }
        return res;
    }

};
```