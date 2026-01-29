
leetcode link : https://leetcode.com/problems/minimum-falling-path-sum/description/


 The Strategy: Three Potential OriginsIn Minimum Path Sum, you could only come from the Top or the Left.In Minimum Falling Path Sum, to reach cell $(r, c)$, you can come from three cells in the row above:Directly above: $(r-1, c)$Diagonally left: $(r-1, c-1)$Diagonally right: $(r-1, c+1)$


 The Logic
n = number of rows and m = number of columns.

The first row is your base case (the cost is just the value in the cell).

For every other row, each cell chooses the minimum of its three possible parents and adds its own value.





```python

class Solution:
    def minFallingPathSum(self, matrix: List[List[int]]) -> int:
        n = len(matrix)
        m = len(matrix[0])


        prev_row = matrix[0][:]

        for r in range(1,n):
            curr_row = [0]*n

            for c in range(m):
                # 1. Directly above
                mid = prev_row[c]
                # 2. Top-left (check boundary)
                left = prev_row[c-1] if c > 0 else float('inf')
                # 3. Top-right (check boundary)
                right = prev_row[c+1] if c<m-1 else float('inf')
                # Choose the best parent and add current cell value
                curr_row[c] = matrix[r][c] + min(mid , left , right)

            prev_row = curr_row

        return min(prev_row)

        

```

 Pro-Tip: Modifying the InputIf the interviewer says you can use $O(1)$ extra space, you can modify the matrix directly:


```python

 for r in range(1, n):
    for c in range(m):
        matrix[r][c] += min(matrix[r-1][max(0, c-1)], 
                            matrix[r-1][c], 
                            matrix[r-1][min(m-1, c+1)])
return min(matrix[-1])
```
