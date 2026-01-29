
# **Largest Divisible Subset**

**LeetCode Link:** [https://leetcode.com/problems/largest-divisible-subset/](https://leetcode.com/problems/largest-divisible-subset/)

---

## **Problem Statement**

Given a set of **distinct** positive integers `nums`, return the largest subset `answer` such that every pair `(answer[i], answer[j])` of elements in this subset satisfies:
- `answer[i] % answer[j] == 0`, or
- `answer[j] % answer[i] == 0`

If there are multiple solutions, return any of them.

**Example:**
```
Input: nums = [1,2,3]
Output: [1,2] or [1,3]

Input: nums = [1,2,4,8]
Output: [1,2,4,8]
```

---

## **Strategy: LIS with Path Reconstruction**

**Key Insight:** After sorting, if `a` divides `b` and `b` divides `c`, then `a` divides `c` (transitivity).

This means we can use an **LIS-style DP approach** where:
- Sort the array
- For each number, check if it's divisible by any previous number
- Track the "parent" to reconstruct the path

### **Why Sorting Works**

**Property:** If `nums[i] % nums[j] == 0` and `i > j` after sorting, then `nums[i] ≥ nums[j]`.

**Chain Property:** In a sorted array, if we have a chain `[a, b, c]` where `a | b` and `b | c`, then:
```
a | b means b = k1 * a
b | c means c = k2 * b = k2 * k1 * a
Therefore: a | c (transitivity)
```

This means we only need to check if the **current number** is divisible by the **last number in the chain**, not all previous numbers!

---

## **Solution**

```python
class Solution:
    def largestDivisibleSubset(self, nums: List[int]) -> List[int]:
        if not nums:
            return []
        
        nums.sort()
        n = len(nums)
        
        # dp[i] = length of longest divisible subset ending at index i
        dp = [1] * n
        # parent[i] = previous index in the chain ending at i
        parent = [-1] * n
        
        max_size = 1
        best_idx = 0
        
        for i in range(n):
            for j in range(i):
                # Check if nums[i] is divisible by nums[j]
                if nums[i] % nums[j] == 0:
                    # Can extend the chain ending at j
                    if dp[j] + 1 > dp[i]:
                        dp[i] = dp[j] + 1
                        parent[i] = j
            
            # Track the best ending position
            if dp[i] > max_size:
                max_size = dp[i]
                best_idx = i
        
        # Reconstruct the subset using parent pointers
        result = []
        while best_idx != -1:
            result.append(nums[best_idx])
            best_idx = parent[best_idx]
        
        return result[::-1]  # Reverse to get ascending order
```

---

## **Example Walkthrough**

**Input:** `nums = [1, 2, 4, 8]`

### **After Sorting**
```
nums = [1, 2, 4, 8] (already sorted)
```

### **DP Computation**

| i | nums[i] | Check j | Divisible? | dp[i] | parent[i] |
|---|---------|---------|------------|-------|-----------|
| 0 | 1 | - | - | 1 | -1 |
| 1 | 2 | j=0: 2%1=0 | ✓ | 2 | 0 |
| 2 | 4 | j=0: 4%1=0 | ✓ | 2 | 0 |
|   |     | j=1: 4%2=0 | ✓ | 3 | 1 |
| 3 | 8 | j=0: 8%1=0 | ✓ | 2 | 0 |
|   |     | j=1: 8%2=0 | ✓ | 3 | 1 |
|   |     | j=2: 8%4=0 | ✓ | 4 | 2 |

### **Final State**
```
dp = [1, 2, 3, 4]
parent = [-1, 0, 1, 2]
max_size = 4, best_idx = 3
```

### **Path Reconstruction**
```
Start at index 3: nums[3] = 8
parent[3] = 2 → nums[2] = 4
parent[2] = 1 → nums[1] = 2
parent[1] = 0 → nums[0] = 1
parent[0] = -1 → STOP

reverse([8, 4, 2, 1]) = [1, 2, 4, 8]
```

**Result:** `[1, 2, 4, 8]`

---

## **Why We Need Parent Array**

**Problem:** DP only gives us the **length** of the longest subset, not the actual elements.

**Solution:** The `parent` array tracks which previous element each current element extends.

**Reconstruction Process:**
```python
while best_idx != -1:
    result.append(nums[best_idx])
    best_idx = parent[best_idx]  # Jump to previous element in chain
```

This walks backward through the chain, collecting all elements.

---

## **Why Check Only `nums[i] % nums[j]` and Not Vice Versa?**

**Question:** Do we need to check both `nums[i] % nums[j]` and `nums[j] % nums[i]`?

**Answer:** No! After sorting, `nums[i] > nums[j]` when `i > j`, so:
- `nums[i] % nums[j]` could be 0 (i is multiple of j)
- `nums[j] % nums[i]` can never be 0 (smaller can't divide larger)

**Example:**
```
nums = [1, 2, 4]
i=2, j=1:
  nums[2] % nums[1] = 4 % 2 = 0 ✓ Check this
  nums[1] % nums[2] = 2 % 4 = 2 ✗ Never check this
```

---

## **Mathematical Property: Transitivity**

**Theorem:** If `a | b` and `b | c`, then `a | c`.

**Proof:**
```
a | b means b = k1 × a for some integer k1
b | c means c = k2 × b for some integer k2

Substituting:
c = k2 × b = k2 × (k1 × a) = (k2 × k1) × a

Therefore: a | c
```

**Why This Matters:**
We don't need to verify all pairs in the subset. If we build the subset incrementally (checking only the last element), transitivity guarantees all pairs will satisfy the divisibility condition.

---

## **Edge Cases**

### **1. Array with 1**
```
Input: [1, 2, 3]
Output: [1, 2] or [1, 3]

1 divides everything, so it can be in any subset
```

### **2. All Prime Numbers**
```
Input: [2, 3, 5, 7]
Output: [2] (or any single element)

No prime divides another prime, so max subset size = 1
```

### **3. Powers of Same Number**
```
Input: [3, 9, 27, 81]
Output: [3, 9, 27, 81]

Each is divisible by the previous: 9%3=0, 27%9=0, 81%27=0
```

---

## **Comparison with Standard LIS**

| Aspect | Standard LIS | Largest Divisible Subset |
|--------|--------------|---------------------------|
| **Condition** | `nums[i] > nums[j]` | `nums[i] % nums[j] == 0` |
| **Need Sorting?** | No (order matters) | Yes (enables transitivity) |
| **DP Value** | Length | Length |
| **Return Type** | Length only | Actual subset |
| **Need Parent Array?** | No (unless reconstructing) | Yes (to reconstruct path) |

---

## **Complexity Analysis**

**Time Complexity:** $O(n^2)$  
- Sorting: $O(n \log n)$
- DP: Two nested loops: $O(n^2)$
- Path reconstruction: $O(n)$
- **Total:** $O(n^2)$

**Space Complexity:** $O(n)$  
- DP array: $O(n)$
- Parent array: $O(n)$
- Result array: $O(n)$