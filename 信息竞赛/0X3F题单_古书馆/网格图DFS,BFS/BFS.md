###### BFS基本流程：
1.给你一个 `m x n` 的迷宫矩阵 `maze` （**下标从 0 开始**），矩阵中有空格子（用 `'.'` 表示）和墙（用 `'+'` 表示）。同时给你迷宫的入口 `entrance` ，用 `entrance = [entrancerow, entrancecol]` 表示你一开始所在格子的行和列。
每一步操作，你可以往 **上**，**下**，**左** 或者 **右** 移动一个格子。你不能进入墙所在的格子，你也不能离开迷宫。你的目标是找到离 `entrance` **最近** 的出口。**出口** 的含义是 `maze` **边界** 上的 **空格子**。`entrance` 格子 **不算** 出口。
请你返回从 `entrance` 到最近出口的最短路径的 **步数** ，如果不存在这样的路径，请你返回 `-1` 。
- `maze.length == m`
- `maze[i].length == n`
- `1 <= m, n <= 100`
- `maze[i][j]` 要么是 `'.'` ，要么是 `'+'` 。
- `entrance.length == 2`
- `0 <= entrancerow < m`
- `0 <= entrancecol < n`
- `entrance` 一定是空格子。
	<font color="yellow">BFS由于是队列实现，故扩散的过程是分步的，其余与DFS相似</font>
```c++
    int nearestExit(vector<vector<char>>& maze, vector<int>& entrance)
    {
        int n = maze.size(), m = maze[0].size();
        queue<tuple<int,int,int>> q;  //记录x,y,步数
        q.emplace(entrance[0],entrance[1],0);  //放置到起点，记得把起点设置为墙
        maze[entrance[0]][entrance[1]] = '+';
        //准备一个x，y的移动数组
        int dx[4] = {0,0,1,-1};
        int dy[4] = {1,-1,0,0};
        //开始填充
        while(!q.empty())
        {
            //把点拿出来
            auto [X,Y,D] = q.front();
            q.pop();
            //向四周扩散并判断
            for(int k=0; k<4; k++)
            {
                int x = X+dx[k], y = Y+dy[k];
                //是否能过去，过去了就要扩展这个点，并标记来过
                if(x >= 0 && x < n && y >= 0 && y < m && maze[x][y] == '.')
                {
                    //若发现出口，因为是逐步走的，这一次定为最少步数
                    if(x == 0 || x == n-1 || y == 0 || y == m-1)
                        return D+1;
                    q.emplace(x,y,D+1);
                    maze[x][y] = '+';
                }
            }
        }
        return -1;
    }
```

2.给你一个大小为 `m x n` 的整数矩阵 `isWater` ，它代表了一个由 **陆地** 和 **水域** 单元格组成的地图。
- 如果 `isWater[i][j] == 0` ，格子 `(i, j)` 是一个 **陆地** 格子。
- 如果 `isWater[i][j] == 1` ，格子 `(i, j)` 是一个 **水域** 格子。
你需要按照如下规则给每个单元格安排高度：
- 每个格子的高度都必须是非负的。
- 如果一个格子是 **水域** ，那么它的高度必须为 `0` 。
- 任意相邻的格子高度差 **至多** 为 `1` 。当两个格子在正东、南、西、北方向上相互紧挨着，就称它们为相邻的格子。（也就是说它们有一条公共边）
找到一种安排高度的方案，使得矩阵中的最高高度值 **最大** 。
请你返回一个大小为 `m x n` 的整数矩阵 `height` ，其中 `height[i][j]` 是格子 `(i, j)` 的高度。如果有多种解法，请返回 **任意一个**
- `m == isWater.length`
- `n == isWater[i].length`
- `1 <= m, n <= 1000`
- `isWater[i][j]` 要么是 `0` ，要么是 `1` 。
- 至少有 **1** 个水域格子。
	<font color="yellow">看看二维vector的初始化</font>
```c++
    int dirs[4][2] = {{1,0},{-1,0},{0,1},{0,-1}};
    queue<tuple<int,int,int>> q;
    //注意全是水或者是陆地的情况，题目说了至少一个水
    vector<vector<int>> highestPeak(vector<vector<int>>& isWater) {
        int n = isWater.size(), m = isWater[0].size();
		 //把水的位置入队
        for(int i=0; i<n; i++)
            for(int j=0; j<m; j++)
                if(isWater[i][j] == 1)
                {
                    q.emplace(i,j,0);
                    isWater[i][j] = -1;
                }
		//初始化答案数组，并开始BFS
		//二维vector的初始化
        vector<vector<int>> ans(n,vector<int>(m,0));
        while(!q.empty())
        {
		    //取点
            auto [x,y,D] = q.front();
            q.pop();
            //扩展并判断
            for(auto& [X,Y] : dirs)
            {
                int nx = x+X, ny = y+Y;
                if(nx < 0 || nx == n || ny < 0 || ny == m || isWater[nx][ny] == -1)
                    continue;
                q.emplace(nx,ny,D+1);
                isWater[nx][ny] = -1;
                ans[nx][ny] = D+1;
            }
        }
        return ans;
    }
```


###### 多源BFS

![[Pasted image 20241214121507.png]]
![[Pasted image 20241214121529.png]]
1.<font color="yellow">我们把陆地都入队，分步扩散，最后一个海洋被扩散的就是离所有陆地最远的（即某海洋离最近的陆地最远）</font>
<font color="yellow">为何不入队所有的海洋呢？由于BFS具有最短路的特性，比如说下面这个图，把0都入队，扩散时里面的0直接被中止扩散了，因为这一步有别的点位可以用更短的路到达（也可以说被vis标记了），扩散到的陆地都是某个海洋到这个陆地的最短距离，而没有最长的特性（信息）。故入队陆地时，BFS扩散到海洋的意义为某陆地到这个海洋的最短路</font>
![[Pasted image 20241214123428.png]]
你现在手里有一份大小为 `n x n` 的 网格 `grid`，上面的每个 单元格 都用 `0` 和 `1` 标记好了。其中 `0` 代表海洋，`1` 代表陆地。
请你找出一个海洋单元格，这个海洋单元格到离它最近的陆地单元格的距离是最大的，并返回该距离。如果网格上只有陆地或者海洋，请返回 `-1`。
我们这里说的距离是「曼哈顿距离」（ Manhattan Distance）：`(x0, y0)` 和 `(x1, y1)` 这两个单元格之间的距离是 `|x0 - x1| + |y0 - y1|` 。
<font color="yellow">此题入队的时候不用改vis，因为只有是海洋才扩散，陆地不用改</font>
```c++
    int maxDistance(vector<vector<int>>& grid) {
        //拿到长度
        int n = grid.size();
        //创建队列，并多源入队，不用标记
        queue<tuple<int,int,int>> q;
        for(int i=0; i<n; i++)
            for(int j=0; j<n; j++)
                if(grid[i][j] == 1)
                {
                    q.emplace(i,j,0);
                }
        //创建方向数组
        int dx[4] = {0,0,1,-1};
        int dy[4] = {1,-1,0,0};
        //染色,并创建答案变量
        int ans = 0;
        while(!q.empty())
        {
            //拿点，并划掉
            auto [X,Y,D] = q.front();
            q.pop();
            //最后一个染为陆地的海洋就是曼哈顿距离最远的
            //因为超级源点，且队列
            ans = max(ans, D);
            //扩散
            for(int k=0; k<4; k++)
            {
                int x = X+dx[k], y = Y+dy[k];
                //判断是否为海，是则入队并标记
                if(x >= 0 && x < n && y >= 0 && y < n && grid[x][y] == 0)
                {
                    q.emplace(x, y, D+1);
                    grid[x][y] = 1;
                }
            }
        }
        //海洋到陆地距离为零，就是只有陆地或海洋
        return ans == 0 ? -1 : ans;
    }
```

2.给定一个由 `0` 和 `1` 组成的矩阵 `mat` ，请输出一个大小相同的矩阵，其中每一个格子是 `mat` 中对应位置元素到最近的 `0` 的距离。
两个相邻元素间的距离为 `1` 。
- `m == mat.length`
- `n == mat[i].length`
- `1 <= m, n <= 104`
- `1 <= m * n <= 104`
- `mat[i][j] is either 0 or 1.`
- `mat` 中至少有一个 `0`
```c++
    //图的多源bfs，这里把0入队，碰到一个1，就是某个零到1的最短路
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int n = mat.size(), m = mat[0].size();
        //bfs,dfs必备的方向数组
        int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
        //准备一个队列，并入队，标记
        queue<tuple<int,int,int>> q;
        for(int i=0; i<n; i++)
            for(int j=0; j<m; j++)
            {
                if(mat[i][j] == 0)
                {
                    q.emplace(i,j,0);
                    mat[i][j] = -1;
                }
                
            }
        //准备一个ans数组,不能直接用数组存，牢vector，等出去了换成数组
        vector<vector<int>> ans(n,vector<int>(m,0));
        while(!q.empty())
        {
            //取点
            auto [x,y,D] = q.front();
            q.pop();
            //扩散
            for(auto& [X,Y] : dirs)
            {
                int nx = x+X, ny = y+Y;
                //判断能否过去
                if(nx < 0 || nx == n || ny < 0 || ny == m || mat[nx][ny] == -1)
                    continue;
                //能过去，放置并标记
                if(mat[nx][ny] == 1)  //找到一个最短路（这个1到某个0的最短路）
                    ans[nx][ny] = D+1; 
                q.emplace(nx,ny,D+1);
                mat[nx][ny] = -1;
            }
        }
        return ans;
    }
```


###### BFS的分层处理
给你一个下标从 **0** 开始的二维整数数组 `grid` ，它的大小为 `m x n` ，表示一个商店中物品的分布图。数组中的整数含义为：

- `0` 表示无法穿越的一堵墙。
- `1` 表示可以自由通过的一个空格子。
- 所有其他正整数表示该格子内的一样物品的价格。你可以自由经过这些格子。

从一个格子走到上下左右相邻格子花费 `1` 步。

同时给你一个整数数组 `pricing` 和 `start` ，其中 `pricing = [low, high]` 且 `start = [row, col]` ，表示你开始位置为 `(row, col)` ，同时你只对物品价格在 **闭区间** `[low, high]` 之内的物品感兴趣。同时给你一个整数 `k` 。

你想知道给定范围 **内** 且 **排名最高** 的 `k` 件物品的 **位置** 。排名按照优先级从高到低的以下规则制定：

1. 距离：定义为从 `start` 到一件物品的最短路径需要的步数（**较近** 距离的排名更高）。
2. 价格：**较低** 价格的物品有更高优先级，但只考虑在给定范围之内的价格。
3. 行坐标：**较小** 行坐标的有更高优先级。
4. 列坐标：**较小** 列坐标的有更高优先级。

请你返回给定价格内排名最高的 `k` 件物品的坐标，将它们按照排名排序后返回。如果给定价格内少于 `k` 件物品，那么请将它们的坐标 **全部** 返回

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 105`
- `1 <= m * n <= 105`
- `0 <= grid[i][j] <= 105`
- `pricing.length == 2`
- `2 <= low <= high <= 105`
- `start.length == 2`
- `0 <= row <= m - 1`
- `0 <= col <= n - 1`
- `grid[row][col] > 0`
- `1 <= k <= m * n`

<font color="yellow">展示牛马sort，详见代码</font>
<font color="yellow">这里队列用的pair，没存步数，因为不需要知道具体是多少步了，只要逐层取东西，取完筛选一次就可（先把第N层的size拿到（必须用len先取出来，因为后面q长度是动态的），只取q中size个元素，然后扩散到N+1层，q正常运行且把有效信息存入临时数组qq里（定义在外面记得完了清空，里面定义的不用管），再对qq筛选正确答案放到ans里）</font>

```c++
    //方向数组
    int dirs[4][2] = {{1,0},{-1,0},{0,1},{0,-1}};
    
    vector<vector<int>> highestRankedKItems(vector<vector<int>>& grid, vector<int>& pricing, vector<int>& start, int k) {
        //n为行数，m为列数
        int n = grid.size(), m = grid[0].size();
        //标记数组
        bool vis[n][m];
        memset(vis,false,sizeof(vis));
        //bfs队列，初始化并标记
        queue<pair<int,int>> q;
        q.emplace(start[0],start[1]);
        vis[start[0]][start[1]] = true;

        //ans答案数组
        vector<vector<int>> ans;
        //逐步bfs，把东西都先取出来用qq来存
        vector<pair<int,int>> qq;
        while(!q.empty())
        {
            //遍历这一层
            int len = q.size();
            for(int _=0; _<len; _++)
            {
                auto [x,y] = q.front();
                q.pop();
                if(grid[x][y] > 1)
                    qq.emplace_back(x,y);
                for(auto& [X,Y] : dirs)
                {
                    int nx = x+X, ny = y+Y;
                    if(nx < 0 || nx == n || ny < 0 || ny == m || grid[nx][ny] == 0 || vis[nx][ny] == true)
                        continue;
                    q.emplace(nx,ny);
                    vis[nx][ny] = true;
                }
            }
            /*
                选数要在后面选，前面选的话，第一层就选不到了
            */
            //根据要求的2,3,4排序
            sort(qq.begin(), qq.end(), [&](auto& a, auto& b){
                int va = grid[a.first][a.second], vb = grid[b.first][b.second];
                return va < vb || va == vb && a.first < b.first || va == vb && a.first == b.first && a.second < b.second;
            });
            //把需要的装进ans数组，若数量够了就返回
            for(auto [x,y] : qq)
            {
                if(ans.size() == k)
                    return ans;
                //必须满足的基本条件 low <= x <= high
                if(pricing[0] <= grid[x][y] && grid[x][y] <= pricing[1])
                    ans.push_back({x,y});
            }
            //每层遍历完就把这层的清理掉
            qq.clear();
        }
        return ans;
    }
```


###### 多状态BFS
给你一个 `m * n` 的网格，其中每个单元格不是 `0`（空）就是 `1`（障碍物）。每一步，您都可以在空白单元格中上、下、左、右移动。
如果您 **最多** 可以消除 `k` 个障碍物，请找出从左上角 `(0, 0)` 到右下角 `(m-1, n-1)` 的最短路径，并返回通过该路径所需的步数。如果找不到这样的路径，则返回 `-1` 。
- `grid.length == m`
- `grid[0].length == n`
- `1 <= m, n <= 40`
- `1 <= k <= m*n`
- `grid[i][j]` 是 `0` 或 `1`
- `grid[0][0] == grid[m-1][n-1] == 0`
<font color="yellow">最多可以消除k个障碍物，即经过的障碍最多有k个</font>

<font color="yellow">因为起点与终点没有障碍，一固定路线从边界走过去最多经历m+n-3个障碍</font>
<font color="yellow">故rest多于这么多时，必定不是最优路线</font>
<font color="yellow">而数据范围k小于m*n，故可以进行优化剪枝，时间复杂度由O(NMK) -> O( NM * min(k,M+N-3) )</font>
<font color="yellow">即原来一个点位可能会被经历K次，现在最多经历M+N-3次</font>

<font color="yellow">与上面的BFS相比，我们在XYD上还要增加一个REST（剩余破坏墙的个数）的状态，表示一个格点上剩余REST不同，结果也会不同</font>
```c++
    queue<tuple<int,int,int,int>> q;
    int dirs[4][2] = {{1,0},{-1,0},{0,-1},{0,1}};
    //bool vis[40][40][1600];
    bool vis[40][40][78];
    
    int shortestPath(vector<vector<int>>& grid, int k)
    {
        int n = grid.size(), m = grid[0].size();
        memset(vis,false,sizeof(vis));
        //优化，但大小为1*1时(m+n-3 == -1)会出现-1，故特判一下
        k = min(k,m+n-3);
        if(n == 1 && m == 1) //数据范围m,n>=1
            return 0;
        //初始点初始化
        vis[0][0][k] = true;
        q.emplace(0,0,0,k);

        while(!q.empty()) //不需打分层，因为无需分层处理数据
        {
            auto [x,y,D,rest] = q.front();
            q.pop();
            //到终点了
            if(x == n-1 && y == m-1)
                return D;
            for(auto& [X,Y] : dirs)
            {
                int nx = x+X, ny = y+Y;
                //标记有三个状态，故要分类判断，放后面
                if(nx < 0 || nx == n || ny < 0 || ny == m)
                    continue;
                //不越界，以是否为墙分情况分配状态，还要注意标记
                if(grid[nx][ny] == 0 && !vis[nx][ny][rest])
                {
                    vis[nx][ny][rest] = true;
                    q.emplace(nx,ny,D+1,rest);
                }
                else if(grid[nx][ny] == 1 && rest && !vis[nx][ny][rest-1])
                {
                    vis[nx][ny][rest-1] = true;
                    q.emplace(nx,ny,D+1,rest-1);
                }
            }
        }
        return -1;
    }
```

