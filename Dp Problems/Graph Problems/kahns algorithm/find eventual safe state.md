

```python

class Solution:
    def eventualSafeNodes(self, graph: List[List[int]]) -> List[int]:

        n = len(graph)
        # 1. out_degree tracks how many edges leave a node in the ORIGINAL graph
        out_degree = [0]*n

        # 2. reversed_adj stores edges in the opposite direction
        reversed_adj = [ [ ] for i in range(n)]

        for i in range(n):
            for neighbor in graph[i]:

                reversed_adj[neighbor].append(i)
                out_degree[i] +=1

        # 3. Start with Terminal Nodes (out-degree 0 in original graph)
        queue = deque([ i for i in range(n ) if out_degree[i] == 0])

        safe = [ False] *n

        while queue:
            node = queue.popleft()
            safe[node] = True

            # 4. For every node that pointed to this 'safe' node
            for prev in reversed_adj[node]:
                out_degree[prev]-=1
                # If all paths from 'prev' now lead to safe nodes
                if out_degree[prev] ==0:
                    queue.append(prev)

        # Result must be in ascending order
        return [i for i in range(n) if safe[i]]

```