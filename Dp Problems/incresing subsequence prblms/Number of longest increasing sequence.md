
# **Number of Longest Increasing Subsequence**

**LeetCode Link:** [https://leetcode.com/problems/number-of-longest-increasing-subsequence/](https://leetcode.com/problems/number-of-longest-increasing-subsequence/)

---

## **Problem Statement**

Given an integer array `nums`, return the **number of longest increasing subsequences**.

**Notice** that the sequence has to be **strictly increasing**.

**Example:**
```
Input: nums = [1,3,5,4,7]
Output: 2
Explanation: The two longest increasing subsequences are [1, 3, 4, 7] and [1, 3, 5, 7].

Input: nums = [2,2,2,2,2]
Output: 5
Explanation: The length of the longest increasing subsequence is 1, and there are 5 increasing subsequences of length 1.
```

---

## **Strategy: DP with Two Arrays**

**Key Insight:** We need to track not only the **length** of the LIS ending at each position but also **how many ways** we can achieve that length.

### **DP State Definition**

```
lengths[i] = length of longest increasing subsequence ending at index i
counts[i] = number of LIS of that length ending at index i
```

### **DP Transitions**

For each position `i`, look back at all previous positions `j < i`:

**If `nums[i] > nums[j]`:**

**Scenario A: Found a longer path**
```python
if lengths[j] + 1 > lengths[i]:
    lengths[i] = lengths[j] + 1
    counts[i] = counts[j]  # Inherit the count
```
We found a **new best** length for position `i`. Reset count to match.

**Scenario B: Found an alternative path of the same length**
```python
elif lengths[j] + 1 == lengths[i]:
    counts[i] += counts[j]  # Add the counts together
```
We found **another way** to reach the current maximum length. Accumulate counts.

**Scenario C: Shorter path**
```python
if lengths[j] + 1 < lengths[i]:
    # Ignore it; not contributing to LIS
```

---

## **Solution**

```python
class Solution:
    def findNumberOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        n = len(nums)
        # lengths[i] = length of LIS ending at index i
        lengths = [1] * n
        # counts[i] = number of LIS of that length ending at i
        counts = [1] * n
        
        for i in range(n):
            for j in range(i):
                if nums[i] > nums[j]:
                    if lengths[j] + 1 > lengths[i]:
                        # Found a longer path
                        lengths[i] = lengths[j] + 1
                        counts[i] = counts[j]
                    elif lengths[j] + 1 == lengths[i]:
                        # Found an alternative path of same length
                        counts[i] += counts[j]
        
        # Find the maximum length
        max_len = max(lengths)
        
        # Sum counts for all positions with max length
        return sum(c for l, c in zip(lengths, counts) if l == max_len)
```

---

## **Example Walkthrough**

**Input:** `nums = [1, 3, 5, 4, 7]`

### **DP Computation**

**Initial State:**
```
lengths = [1, 1, 1, 1, 1]
counts = [1, 1, 1, 1, 1]
```

**Process i=1 (nums[1]=3):**
```
Check j=0: nums[3] > nums[1] ✓
  lengths[0] + 1 = 2 > lengths[1] = 1 → Found longer path
  lengths[1] = 2
  counts[1] = counts[0] = 1

State:
lengths = [1, 2, 1, 1, 1]
counts = [1, 1, 1, 1, 1]
```

**Process i=2 (nums[2]=5):**
```
Check j=0: nums[5] > nums[1] ✓
  lengths[0] + 1 = 2 > lengths[2] = 1 → Found longer path
  lengths[2] = 2
  counts[2] = counts[0] = 1

Check j=1: nums[5] > nums[3] ✓
  lengths[1] + 1 = 3 > lengths[2] = 2 → Found even longer path
  lengths[2] = 3
  counts[2] = counts[1] = 1

State:
lengths = [1, 2, 3, 1, 1]
counts = [1, 1, 1, 1, 1]
```

**Process i=3 (nums[3]=4):**
```
Check j=0: nums[4] > nums[1] ✓
  lengths[0] + 1 = 2 > lengths[3] = 1
  lengths[3] = 2
  counts[3] = 1

Check j=1: nums[4] > nums[3] ✓
  lengths[1] + 1 = 3 > lengths[3] = 2
  lengths[3] = 3
  counts[3] = counts[1] = 1

Check j=2: nums[4] < nums[5] ✗ Skip

State:
lengths = [1, 2, 3, 3, 1]
counts = [1, 1, 1, 1, 1]
```

**Process i=4 (nums[4]=7):**
```
Check j=0: nums[7] > nums[1] ✓
  lengths[0] + 1 = 2 > lengths[4] = 1
  lengths[4] = 2, counts[4] = 1

Check j=1: nums[7] > nums[3] ✓
  lengths[1] + 1 = 3 > lengths[4] = 2
  lengths[4] = 3, counts[4] = 1

Check j=2: nums[7] > nums[5] ✓
  lengths[2] + 1 = 4 > lengths[4] = 3
  lengths[4] = 4, counts[4] = counts[2] = 1

Check j=3: nums[7] > nums[4] ✓
  lengths[3] + 1 = 4 == lengths[4] = 4 → Alternative path!
  counts[4] += counts[3] = 1 + 1 = 2

State:
lengths = [1, 2, 3, 3, 4]
counts = [1, 1, 1, 1, 2]
```

### **Final Calculation**
```
max_len = max(lengths) = 4

Sum counts where length == 4:
  lengths[4] = 4, counts[4] = 2

Answer: 2
```

**The Two LIS:**
1. `[1, 3, 5, 7]` (via j=2)
2. `[1, 3, 4, 7]` (via j=3)

---

## **The Logic**

As we look back at $j < i$ where $\text{nums}[i] > \text{nums}[j]$, we have three scenarios:

### **Scenario A: We found a longer path**
If `lengths[j] + 1 > lengths[i]`, it means we just found a new "best" length for $i$. We update `lengths[i]` and **inherit** the count: `counts[i] = counts[j]`.

### **Scenario B: We found an alternative path of the same "best" length**
If `lengths[j] + 1 == lengths[i]`, it means we found **another way** to reach the current maximum length. We **add** the counts together: `counts[i] += counts[j]`.

### **Scenario C: Shorter path**
If `lengths[j] + 1 < lengths[i]`, we ignore it; it's not contributing to the longest subsequence.

---

## **Why Add Counts in Scenario B?**

**Question:** Why do we add counts instead of taking max?

**Answer:** Because we want to count **all different ways** to achieve the same length!

**Example:**
```
nums = [1, 2, 4, 3, 5]

At position 4 (value 5):
  Can extend from position 2 (value 4): [1,2,4,5] - 1 way
  Can extend from position 3 (value 3): [1,2,3,5] - 1 way
  Both paths have length 4!
  
  Total ways = 1 + 1 = 2
```

We're **counting combinations**, not picking the best!

---

## **Edge Cases**

### **1. All Equal Elements**
```
Input: [2, 2, 2, 2]
Output: 4

Each element is a LIS of length 1
lengths = [1, 1, 1, 1]
counts = [1, 1, 1, 1]
max_len = 1, sum = 4
```

### **2. Strictly Decreasing**
```
Input: [5, 4, 3, 2, 1]
Output: 5

No element can extend any previous
Each element forms its own LIS of length 1
```

### **3. Strictly Increasing**
```
Input: [1, 2, 3, 4]
Output: 1

Only one LIS: [1, 2, 3, 4]
lengths = [1, 2, 3, 4]
counts = [1, 1, 1, 1]
max_len = 4, sum = 1
```

---

## **Comparison with Standard LIS**

| Aspect | Standard LIS | Number of LIS |
|--------|--------------|---------------|
| **DP Arrays** | 1 (lengths only) | 2 (lengths + counts) |
| **Transition** | `dp[i] = max(dp[i], dp[j] + 1)` | Check if longer/equal, update counts |
| **Return Value** | `max(dp)` | `sum(counts where length == max)` |
| **Complexity** | O(n²) | O(n²) |
| **Tracks** | Length only | Length + number of ways |

---

## **Complexity Analysis**

**Time Complexity:** $O(n^2)$  
Two nested loops, each iterating up to $n$ times.

**Space Complexity:** $O(n)$  
Two arrays of size $n$ (lengths and counts).