


# **Longest Increasing Subsequence (LIS)**

**LeetCode Link:** [https://leetcode.com/problems/longest-increasing-subsequence/](https://leetcode.com/problems/longest-increasing-subsequence/description/)

---

## **Problem Statement**

Given an integer array `nums`, return the length of the longest strictly increasing subsequence.

A **subsequence** is a sequence that can be derived from an array by deleting some or no elements without changing the order of the remaining elements.

**Example:**
```
Input: nums = [10,9,2,5,3,7,101,18]
Output: 4
Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4.

Input: nums = [0,1,0,3,2,3]
Output: 4
```

---

## **Strategy Overview**

There are two main approaches:
1. **Dynamic Programming O(n²)**: Classic DP approach
2. **Binary Search O(n log n)**: Optimized approach using binary search

---

## **Approach 1: Dynamic Programming O(n²)**

### **DP State Definition**

```
dp[i] = length of longest increasing subsequence ending at index i
```

### **DP Recurrence**

For each position `i`, look back at all previous positions `j < i`:
```python
if nums[i] > nums[j]:
    dp[i] = max(dp[i], dp[j] + 1)
```

**Interpretation:** If `nums[i]` can extend the subsequence ending at `j`, update `dp[i]` to be the maximum.

### **Solution 1: O(n²) DP**

```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        n = len(nums)
        # dp[i] = length of LIS ending at index i
        dp = [1] * n  # Every element is a subsequence of length 1
        
        for i in range(n):
            for j in range(i):
                # If nums[i] can extend the subsequence ending at j
                if nums[i] > nums[j]:
                    dp[i] = max(dp[i], dp[j] + 1)
        
        return max(dp)
```

---

## **Approach 2: Binary Search O(n log n)**

### **Key Insight: Patience Sorting**

**Core Idea:** Maintain a `tails` array where `tails[i]` is the **smallest tail element** of all increasing subsequences of length `i+1`.

**Why This Works:**
- If we want to build long subsequences, we should use **small numbers** as early as possible
- By keeping the smallest possible tail for each length, we maximize future extension opportunities

### **The Algorithm**

For each number `x` in the array:
1. **Binary search** for the position where `x` should be in `tails`
2. **If `x` is larger than all elements:** Append it (extends the longest subsequence)
3. **Otherwise:** Replace the first element ≥ `x` (makes that length's tail smaller)

### **Solution 2: O(n log n) Binary Search**

```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        tails = []
        
        for x in nums:
            # Binary search for the leftmost position where x fits
            idx = bisect.bisect_left(tails, x)
            
            if idx == len(tails):
                # x is larger than all elements, extend the subsequence
                tails.append(x)
            else:
                # Replace the element at idx with x (make tail smaller)
                tails[idx] = x
        
        return len(tails)
```

---

## **Example Walkthrough: Binary Search Approach**

**Input:** `nums = [10, 9, 2, 5, 3, 7, 101, 18]`

### **Step-by-Step Execution**

**A Walkthrough: [10, 9, 2, 5, 3, 7, 101, 18]**

| Num | Action | tails state | Reason |
|-----|--------|-------------|--------|
| 10 | Append | [10] | First element. |
| 9 | Replace | [9] | 9 is better than 10 (easier to build on). |
| 2 | Replace | [2] | 2 is better than 9. |
| 5 | Append | [2, 5] | 5 is larger than 2. |
| 3 | Replace | [2, 3] | 3 is better than 5 (idx 1). |
| 7 | Append | [2, 3, 7] | 7 is larger than 3. |
| 101 | Append | [2, 3, 7, 101] | 101 is larger than 7. |
| 18 | Replace | [2, 3, 7, 18] | 18 is better than 101. |

**Final Answer:** `len(tails) = 4`

**Note:** The `tails` array doesn't necessarily represent the actual LIS. It's a **tool** to compute the length. The actual LIS could be `[2, 3, 7, 18]` or `[2, 3, 7, 101]`, both have length 4.

---

## **Why Replace Instead of Insert?**

**Question:** Why do we replace `tails[idx]` instead of inserting?

**Answer:** Because we're only tracking **lengths**, not the actual subsequence. Replacing keeps the array size equal to the maximum length found so far.

**Example:**
```
tails = [2, 5, 7]
New element: 3

Option 1 (Insert): [2, 3, 5, 7] ← Wrong! Length would be 4, but we haven't found a subsequence of length 4
Option 2 (Replace): [2, 3, 7] ← Correct! Still length 3, but now position 1 has a better (smaller) tail
```

---

## **Why `bisect_left` Instead of `bisect_right`?**

**`bisect_left(tails, x)`**: Finds the **first position** where `x` could be inserted to keep the array sorted.

**Why we need the leftmost position:**
- We want to replace the **smallest element ≥ x**
- This makes the tail as small as possible for that length
- Using `bisect_right` would place `x` after equal elements, missing the replacement opportunity

**Example:**
```
tails = [2, 5, 5, 7]
New element: 5

bisect_left([2,5,5,7], 5) = 1 ← Replace first 5
bisect_right([2,5,5,7], 5) = 3 ← Would try to replace 7 (wrong!)
```

---

## **Comparison: O(n²) vs O(n log n)**

| Aspect | O(n²) DP | O(n log n) Binary Search |
|--------|----------|---------------------------|
| **Time** | O(n²) | O(n log n) |
| **Space** | O(n) | O(n) |
| **Can reconstruct LIS?** | Yes (with parent pointers) | No (only gives length) |
| **Code Complexity** | Simple | Requires understanding binary search |
| **Best for** | n ≤ 1000 | n ≤ 100,000 |

---

## **Complexity Analysis**

### **Approach 1: O(n²) DP**
**Time Complexity:** $O(n^2)$  
Two nested loops, each iterating up to $n$ times.

**Space Complexity:** $O(n)$  
DP array of size $n$.

### **Approach 2: O(n log n) Binary Search**
**Time Complexity:** $O(n \log n)$  
For each of $n$ elements, we perform binary search on the `tails` array, which is $O(\log n)$.

**Space Complexity:** $O(n)$  
The `tails` array can grow up to size $n$ in the worst case (when the entire array is strictly increasing).

```python

class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:

        if not nums:
            return 0

        dp = [1] * len(nums)

        for i in range(len(nums)):
            for j in range(i):

                if nums[i] > nums[j]:

                    dp[i] = max(dp[i] , dp[j] +1)

        return max(dp)
        
```

```python

class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:

        if not nums:
            return 0

        tails = []

        for x in nums:
            # Manual binary search for leftmost position
            left, right = 0, len(tails)
            
            while left < right:
                mid = (left + right) // 2
                if tails[mid] < x:
                    left = mid + 1
                else:
                    right = mid
            
            idx = left

            if idx == len(tails):
                tails.append(x)
            else:
                tails[idx] = x
                
        return len(tails)
```