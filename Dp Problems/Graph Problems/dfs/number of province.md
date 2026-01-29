

leetcode link :https://leetcode.com/problems/number-of-provinces/description/

```python
class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:

        n = len(isConnected)
        visited = set()
        provinces = 0

        def dfs(city):
            for neighbor, connected in enumerate(isConnected[city]):
                if connected == 1 and neighbor not in visited:
                    visited.add(neighbor)
                    dfs(neighbor)
        
        for i in range(n):
            if i not in visited:

                provinces += 1
                visited.add(i)
                dfs(i)

        return provinces

```

```python

class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:

        n = len(isConnected)
        parent = [ i for i in range(n)]
        self.provinces = n

        def find(i):
            if parent[i] == i:
                return i

            return find(parent[i])

        def union(i,j):
            root_i = find(i)
            root_j = find(j)

            if root_i != root_j:
                parent[root_i] = root_j
                self.provinces -=1

        for i in range(n):
            for j in range(i+1,n):
                if isConnected[i][j] == 1:
                    union(i,j)

        return self.provinces 

```

```python
from collections import deque

class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:
        n = len(isConnected)
        visited = set()
        provinces = 0
        
        for i in range(n):
            if i not in visited:
                # We found a new province!
                provinces += 1
                # Start BFS to mark all cities in this province
                queue = deque([i])
                visited.add(i)
                
                while queue:
                    curr_city = queue.popleft()
                    # Check all possible neighbors for curr_city
                    for neighbor, connected in enumerate(isConnected[curr_city]):
                        if connected == 1 and neighbor not in visited:
                            visited.add(neighbor)
                            queue.append(neighbor)
                            
        return provinces
```