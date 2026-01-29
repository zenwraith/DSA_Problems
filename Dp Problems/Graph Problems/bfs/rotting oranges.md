


```python

class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:

        rows, cols = len(grid) , len(grid[0])
        queue = deque()
        freash_count = 0
        minutes = 0

        for r in range(rows):
            for c in range(cols):

                if grid[r][c] == 2:
                    queue.append((r,c))
                elif grid[r][c] == 1:
                    freash_count +=1
        
        if freash_count == 0:
            return 0
        
        directions = [ (0,1) , (0,-1) , (1,0) , (-1,0)]

        while queue and freash_count > 0:
            minutes +=1 

            for _ in range(len(queue)):

                r,c = queue.popleft()

                for dr , dc in directions:

                    nr , nc = r+dr , c+dc

                    if 0 <= nr < rows and 0<= nc < cols and grid[nr][nc] ==1:
                        grid[nr][nc] = 2
                        freash_count -=1
                        queue.append((nr,nc))

        return minutes if freash_count == 0 else -1

```