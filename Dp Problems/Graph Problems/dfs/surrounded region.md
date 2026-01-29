



```python

class Solution:
    def solve(self, board: List[List[str]]) -> None:
        """
        Do not return anything, modify board in-place instead.
        """

        if not board:
            return 
        
        n,m = len(board), len(board[0])

        def dfs(r,c):

            if r < 0 or r >= n or c < 0 or c>=m or board[r][c] != 'O':
                return
            
            board[r][c] = 'S'

            dfs(r+1,c)
            dfs(r-1,c)
            dfs(r,c+1)
            dfs(r,c-1)

        for r in range(n):
            dfs(r,0)
            dfs(r, m-1)
        for c in range(m):
            dfs(0,c)
            dfs(n-1, c)

        for r in range(n):
            for c in range(m):

                if board[r][c] == 'O':
                    board[r][c] = 'X'
                elif board[r][c] == 'S':
                    board[r][c] = 'O'
        
```