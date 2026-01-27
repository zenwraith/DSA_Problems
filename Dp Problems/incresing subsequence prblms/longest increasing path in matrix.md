

leetcode link: https://leetcode.com/problems/longest-increasing-path-in-a-matrix/

```python

class Solution:
    def longestIncreasingPath(self, matrix: List[List[int]]) -> int:

        if not matrix or not matrix[0]:return 0


        rows ,cols = len(matrix) , len(matrix[0])

        memo = {}

        def dfs(r,c):

            if (r,c) in memo:
                return memo[(r,c)]

            longest = 1

            for dr,dc in [ (0,1) , (0,-1) , (1,0) , (-1, 0)]:
                nr , nc = r+ dr , c +dc 

                if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:

                    longest = max(longest, 1+ dfs(nr,nc))

            memo[(r,c)] = longest
            return longest

        max_path = 0
        for r in range(rows):
            for c in range(cols):
                max_path = max(max_path, dfs(r,c))

        return max_path


        

```


In most grid-based DFS problems (like finding the number of islands), you need a visited set to avoid spinning in circles. But here, the "Strictly Increasing" rule acts as a natural one-way street.

Since you can only move from a value of 5 to something > 5, you can never circle back to that 5. This means the path is guaranteed to be a Directed Acyclic Graph (DAG).

Why it works (And why it doesn't loop forever)You might worry about infinite recursion. However, because we only move to a strictly greater value, we can never return to a cell we've already visited in the current path. This effectively turns the matrix into a Directed Acyclic Graph (DAG).4. Complexity BreakdownTime: $O(M \times N)$. Even though we call dfs for every cell, the memoization ensures that we only actually compute the logic for each cell once.Space: $O(M \times N)$ for the memoization table and the recursion stack.



A Cool Alternative: Topological Sort (Kahn's Algorithm)
Because this problem is technically finding the longest path in a DAG, you could actually solve it without DFS.

Calculate the out-degree of every cell (how many neighbors are larger).

Put all cells with out-degree == 0 (the "peaks") into a queue.

Perform a BFS level-by-level, "peeling" the matrix like an onion from the largest values down to the smallest.

the number of levels you peel is your answer.