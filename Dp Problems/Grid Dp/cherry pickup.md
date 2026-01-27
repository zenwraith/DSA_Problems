# **Cherry Pickup**

**LeetCode Link:** [https://leetcode.com/problems/cherry-pickup/](https://leetcode.com/problems/cherry-pickup/)

## **Strategy**

### **Two People, One Way**

The **"Return" trip is the trap.** If you find the best path forward and then the best path back, it doesn't work. Why? Because picking cherries on the first trip changes the grid for the second.

**The Key Insight:** Instead of one person going there and back, imagine **two people starting at $(0, 0)$ and walking to $(n-1, n-1)$ simultaneously.**

- If they land on **different cells**, they both **collect the cherries**.
- If they land on the **same cell**, only **one of them collects the cherry** (since it's gone once picked).

### **Reducing the Dimensions**

Normally, two people in a 2D grid would need 4 variables: $(r1, c1)$ and $(r2, c2)$. That's $O(n^4)$.

However, because they move at the same time, their total steps taken ($t$) is always the same:

$$r1 + c1 = t \text{ and } r2 + c2 = t$$

This means if we know $t, r1,$ and $r2$, we can calculate the columns:

$$c1 = t - r1$$
$$c2 = t - r2$$

Now we only need **3 variables:** `dp[t][r1][r2]`. This reduces complexity to $O(n^3)$.

### **Movement Transitions**

At each step $t$, each person can move **Down** or **Right**. This gives us **4 combinations** for the next step:

1. Both move Right
2. Both move Down
3. Person A moves Right, Person B moves Down
4. Person A moves Down, Person B moves Right

### **Why This Works**

By moving them together, we solve the **"cherry gone" problem.** At any point where $r1 == r2$, we know they are on the same cell and we only count the cherry once. Because they are moving towards the destination together, we never have to "revisit" a cell.

## **Solution**

```python
class Solution:
    def cherryPickup(self, grid: List[List[int]]) -> int:
        n = len(grid)
        # memo[t][r1][r2] stores max cherries at step t with person 1 at r1 and person 2 at r2
        memo = {}

        def dp(t, r1, r2):
            c1, c2 = t - r1, t - r2

            # Out of bounds or hit a thorn (-1)
            if (r1 == n or c1 == n or r2 == n or c2 == n or
                grid[r1][c1] == -1 or grid[r2][c2] == -1):
                return float('-inf')

            # Reached destination
            if t == 2 * n - 2:
                return grid[r1][c1]

            if (t, r1, r2) in memo:
                return memo[(t, r1, r2)]

            # Collect cherries at current position
            if r1 == r2:
                cherries = grid[r1][c1]  # Same cell
            else:
                cherries = grid[r1][c1] + grid[r2][c2]  # Different cells

            # Try all 4 combinations of moves
            # (Down, Down), (Down, Right), (Right, Down), (Right, Right)
            res = max(
                dp(t + 1, r1 + 1, r2 + 1),  # Both down
                dp(t + 1, r1 + 1, r2),      # A down, B right
                dp(t + 1, r1, r2 + 1),      # A right, B down
                dp(t + 1, r1, r2)           # Both right
            )

            memo[(t, r1, r2)] = cherries + res
            return memo[(t, r1, r2)]

        result = dp(0, 0, 0)
        return max(0, result)
```
