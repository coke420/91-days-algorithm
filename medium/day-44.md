# 547.朋友圈

https://leetcode-cn.com/problems/friend-circles/

-   [并查集](#并查集)
-   [DFS](#DFS)
-   [BFS](#BFS)

## 题目描述

```
班上有 N 名学生。其中有些人是朋友，有些则不是。他们的友谊具有是传递性。如果已知 A 是 B 的朋友，B 是 C 的朋友，那么我们可以认为 A 也是 C 的朋友。所谓的朋友圈，是指所有朋友的集合。

给定一个 N * N 的矩阵 M，表示班级中学生之间的朋友关系。如果M[i][j] = 1，表示已知第 i 个和 j 个学生互为朋友关系，否则为不知道。你必须输出所有学生中的已知的朋友圈总数。

示例 1:

输入:
[[1,1,0],
 [1,1,0],
 [0,0,1]]
输出: 2
说明：已知学生0和学生1互为朋友，他们在一个朋友圈。
第2个学生自己在一个朋友圈。所以返回2。
示例 2:

输入:
[[1,1,0],
 [1,1,1],
 [0,1,1]]
输出: 1
说明：已知学生0和学生1互为朋友，学生1和学生2互为朋友，所以学生0和学生2也是朋友，所以他们三个在一个朋友圈，返回1。
注意：

N 在[1,200]的范围内。
对于所有学生，有M[i][i] = 1。
如果有M[i][j] = 1，则有M[j][i] = 1。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/friend-circles
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

## 并查集

### 思路

**前置知识**

-   并查集

**入门学习资料**

-   [Using Disjoint Set (Union-Find) To Build A Maze Generator](https://coderscat.com/using-disjoint-set-union-find-to-create-maze)
-   [Disjoint Set Data Structure](https://www.topcoder.com/community/competitive-programming/tutorials/disjoint-set-data-structures/)
-   [个人笔记](https://github.com/suukii/Articles/blob/master/articles/dsa_union_find.md)

### 复杂度分析

-   时间复杂度：$O(α(n))$。
-   空间复杂度：$O(N)$，N 为学生人数。

### 代码

TypeScript Code

```ts
class UnionFind {
    private parents: Array<number>
    private rank: Array<number>
    private numOfSets: number

    constructor(size: number) {
        this.parents = Array(size)
            .fill(0)
            .map((_, i) => i)
        this.rank = Array(size).fill(0)
        this.numOfSets = size
    }

    size(): number {
        return this.numOfSets
    }

    findSet(x: number): number {
        if (x !== this.parents[x]) {
            this.parents[x] = this.findSet(this.parents[x])
        }
        return this.parents[x]
    }

    unionSet(x: number, y: number): void {
        const px: number = this.findSet(x)
        const py: number = this.findSet(y)
        if (px === py) return
        if (this.rank[px] > this.rank[py]) {
            this.parents[py] = px
        } else {
            this.parents[px] = py
            this.rank[px] === this.rank[py] && ++this.rank[py]
        }
        this.numOfSets--
    }
}

function findCircleNum(M: number[][]): number {
    const len: number = M.length
    const uf: UnionFind = new UnionFind(len)
    for (let i = 0; i < len; i++) {
        for (let j = i + 1; j < len; j++) {
            M[i][j] === 1 && uf.unionSet(i, j)
        }
    }
    return uf.size()
}
```

## DFS

### 思路

1. 先选定一个学生，从他开始，找到他的朋友们，再继续分别找出他朋友的朋友们，这就形成了一个朋友圈；
2. 接着找到一个不在这个朋友圈中的学生，再继续步骤 1。那我们怎么知道这个学生不在这个朋友圈中呢？很简单，在步骤 1 的寻找过程中，我们把找到的学生记录在一个 `visited` 数组中就行。

### 复杂度分析

-   时间复杂度：$O(N)$，N 是矩阵 M 的长度，也就是学生数量。
-   空间复杂度：$O(N)$，N 是学生数量，记录访问状态的 `visited` 数组空间是 $O(N)$，递归栈的空间也是 $O(N)$。

### 代码

TypeScript Code

```ts
function findCircleNum(M: number[][]): number {
    const visited: Array<boolean> = Array(M.length)
    let groups: number = 0

    const bfs = (M: number[][], cur: number, visited: Array<boolean>): void => {
        for (let i = 0; i < M[cur].length; i++) {
            // If they are not friends or the ith person has been visited, skip him.
            if (M[cur][i] === 0 || visited[i]) continue

            // Mark as visited because they are all in the same friend circle with the cur person.
            // And continue the search.
            visited[i] = true
            bfs(M, i, visited)
        }
    }

    for (let i = 0; i < M.length; i++) {
        // If he's not been visited, then there is a new friend circle.
        // Start searching for all his friends.
        if (!visited[i]) {
            groups++
            visited[i] = true
            bfs(M, i, visited)
        }
    }
    return groups
}
```

## BFS

### 复杂度分析

-   时间复杂度：$O(N)$，N 是学生数量。
-   空间复杂度：$O(N)$，N 是学生数量。

TypeScript Code

```ts
function findCircleNum(M: number[][]): number {
    let ans: number = 0
    const visited: Array<boolean> = Array(M.length)
    const stack: Array<number> = []

    for (let i = 0; i < M.length; i++) {
        if (visited[i]) continue

        stack.push(i)
        ans++
        while (stack.length) {
            const cur: number = stack.pop() as number

            visited[cur] = true
            const friends: Array<number> = M[cur]
            for (let j = 0; j < friends.length; j++) {
                if (visited[j] || friends[j] === 0) continue
                stack.push(j)
            }
        }
    }

    return ans
}
```

**官方题解**
