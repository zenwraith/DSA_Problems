
# **Partition Array for Maximum Sum**

**LeetCode Link:** [https://leetcode.com/problems/partition-array-for-maximum-sum/](https://leetcode.com/problems/partition-array-for-maximum-sum/)

## **Strategy: Linear DP with "Look Back" Window**

We define `dp[i]` as the maximum sum we can get for the prefix `arr[0...i-1]`.

To calculate `dp[i]`, we look at the last partition. This last partition could have a size anywhere from 1 to $k$.

### **Recurrence Relation**

| Last Partition Length | Calculation |
|---|---|
| Length 1 | `dp[i] = dp[i-1] + (arr[i-1] × 1)` |
| Length 2 | `dp[i] = dp[i-2] + (max(arr[i-2...i-1]) × 2)` |
| Length $len$ | `dp[i] = dp[i-len] + (max(arr[i-len...i-1]) × len)` |

We try all possible lengths from 1 to $k$ and pick the maximum.





## **Solution**


```python
class Solution:
    def maxSumAfterPartitioning(self, arr: List[int], k: int) -> int:

        n = len(arr)
        # dp[i] is the max sum for the first i elements
        dp = [0] * (n+1)

        for i in range(1,n+1):

            current_max = 0
            # Try every possible length for the LAST subarray (up to k)
            # We ensure i - j >= 0 so we don't go out of bounds

            for j in range(1, min(k,i) +1):
                # Update the max element in the current window [i-j ... i-1]
                current_max = max(current_max , arr[i-j])
                # Recurrence: best sum of previous part + (max of current part * length)
                dp[i] = max(dp[i] , dp[i-j] + current_max * j)

        return dp[n]
```




---

## Why do we use `range(1, n+1)` for `i` and `range(1, min(k, i)+1)` for `j`?

- `dp[i]` represents the best sum for the first `i` elements (i.e., `arr[0]` to `arr[i-1]`).
- We want to fill `dp[1]` (first element) up to `dp[n]` (all elements), so `i` goes from 1 to n (inclusive), which is `range(1, n+1)` in Python.
- `j` is the length of the last partition ending at position `i-1`. We can only look back up to `k` elements, but not more than `i` (can't go before start), so `j` goes from 1 to `min(k, i)` (inclusive).

**This indexing is a common DP trick to simplify boundary conditions and avoid negative indices.**

---

## Why do we use `arr[i-j]` and not `arr[j]`?

- In the inner loop, `j` represents the length of the last partition, not an index in the array.
- `arr[i-j]` is the starting index of the last partition ending at position `i-1`.
- If you use `arr[j]`, you would be accessing the wrong element (it would not correspond to the current window you are evaluating).

**Example:**

Suppose `i = 5`, `j = 2` (partition length 2):
- The last partition is `arr[3]` and `arr[4]` (i.e., `arr[i-j]` to `arr[i-1]`).
- `arr[i-j]` gives you the leftmost element of the current window.
- `arr[j]` would just give you the element at index 2, which is unrelated to the current window.

**Summary:**
- You must use `arr[i-j]` to correctly reference the start of the current partition window. Using `arr[j]` would break the logic and produce incorrect results.

leetcode linke : https://leetcode.com/problems/partition-array-for-maximum-sum/description/


