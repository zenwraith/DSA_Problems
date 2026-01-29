
# **Palindrome Partitioning IV**

**LeetCode Link:** [https://leetcode.com/problems/palindrome-partitioning-iv/](https://leetcode.com/problems/palindrome-partitioning-iv/)

---

## **Problem Statement**

Given a string `s`, return `true` if it is possible to split the string `s` into three non-empty palindromic substrings. Otherwise, return `false`.

A string is said to be palindrome if it reverses to itself.

**Example:**
```
Input: s = "abcbdd"
Output: true
Explanation: "abcbdd" = "a" + "bcb" + "dd", and all three substrings are palindromes.
```

---

## **Strategy: Interval DP + Exhaustive Search**

### **Step 1: Build Palindrome Table**

First, we use **Interval DP** to precompute which substrings are palindromes. This is the same pattern as all interval DP problems.

```
isPal[i][j] = True if s[i:j+1] is a palindrome
```

### **Step 2: Try All Possible Partitions**

We need to find two cut positions that divide the string into three palindromic parts:
- **Part 1:** `s[0:i+1]` (must be palindrome)
- **Part 2:** `s[i+1:j+1]` (must be palindrome)
- **Part 3:** `s[j+1:n]` (must be palindrome)

We try all valid combinations of `i` and `j` where:
- `0 ≤ i < n-2` (leaves room for parts 2 and 3)
- `i+1 ≤ j < n-1` (leaves room for part 3)

---

## **Solution**

```python
class Solution:
    def checkPartitioning(self, s: str) -> bool:
        n = len(s)
        
        # --- Step 1: Interval DP (Build palindrome table) ---
        isPal = [[False] * n for i in range(n)]
        
        # Try all possible substring lengths
        for length in range(1, n + 1):
            # Try all starting positions for this length
            for i in range(n - length + 1):
                j = i + length - 1
                
                # Check if s[i:j+1] is palindrome
                if s[i] == s[j]:
                    # Single char or two chars: automatically palindrome
                    # Longer string: inner substring must also be palindrome
                    if length <= 2 or isPal[i+1][j-1]:
                        isPal[i][j] = True
        
        # --- Step 2: Try every possible pair of cuts ---
        # i is the end of the first part (Part 1: 0 to i)
        # j is the end of the second part (Part 2: i+1 to j)
        # Third part is automatic: j+1 to n-1
        for i in range(n - 2):
            for j in range(i + 1, n - 1):
                # Check if all three parts are palindromes
                if isPal[0][i] and isPal[i+1][j] and isPal[j+1][n-1]:
                    return True
        
        return False
```

---

## **Why Use Interval DP for Palindrome Check?**

**Naive Approach:** Check each substring individually with a helper function:
```python
def is_palindrome(s, i, j):
    while i < j:
        if s[i] != s[j]:
            return False
        i += 1
        j -= 1
    return True
```

**Time Complexity:** $O(n^3)$ - for each of $O(n^2)$ partition attempts, we check 3 palindromes, each taking $O(n)$ time in the worst case.

**Interval DP Approach:** Precompute all palindromic substrings once:
```python
isPal[i][j] = (s[i] == s[j]) and (length <= 2 or isPal[i+1][j-1])
```

**Time Complexity:** $O(n^2)$ - build palindrome table once, then $O(n^2)$ to try all partitions.

---

## **Interval DP Formula Breakdown**

For a substring from index `i` to `j`:

```python
if s[i] == s[j]:
    if length <= 2:
        isPal[i][j] = True  # "a" or "aa" or "aba"
    else:
        isPal[i][j] = isPal[i+1][j-1]  # Depends on inner substring
```

**Key Insight:** A string is a palindrome if:
1. First and last characters match: `s[i] == s[j]`
2. The inner substring is also a palindrome: `isPal[i+1][j-1]`

**Base Cases:**
- Single character (`length = 1`): Always palindrome
- Two characters (`length = 2`): Palindrome if both characters match

---

## **Example Walkthrough**

**Input:** `s = "abcbdd"`

### **Step 1: Build Palindrome Table**

```
    a  b  c  b  d  d
a [ T  F  F  F  F  F ]
b [ -  T  F  F  F  F ]
c [ -  -  T  F  F  F ]
b [ -  -  -  T  F  F ]
d [ -  -  -  -  T  T ]
d [ -  -  -  -  -  T ]
```

After processing length = 3:
```
isPal[1][3] = True  # "bcb"
```

After processing length = 2:
```
isPal[4][5] = True  # "dd"
```

### **Step 2: Try All Partitions**

Try `i = 0, j = 3`:
```
Part 1: s[0:1] = "a"      → isPal[0][0] = True ✓
Part 2: s[1:4] = "bcb"    → isPal[1][3] = True ✓
Part 3: s[4:6] = "dd"     → isPal[4][5] = True ✓
```

**All three parts are palindromes!** Return `True`.

---

## **Why `range(n - 2)` and `range(i + 1, n - 1)`?**

**Constraint:** We need three **non-empty** parts.

**For `i` (first cut):**
- Must leave room for at least 2 more characters (parts 2 and 3)
- Maximum value: `n - 3` (e.g., if `n = 6`, `i` can be at most `3`, leaving positions `4` and `5` for parts 2 and 3)
- Range: `0` to `n - 2` (exclusive upper bound in Python)

**For `j` (second cut):**
- Must be after `i`: `j ≥ i + 1`
- Must leave room for at least 1 character for part 3
- Maximum value: `n - 2` (e.g., if `n = 6`, `j` can be at most `4`, leaving position `5` for part 3)
- Range: `i + 1` to `n - 1` (exclusive upper bound)

---

## **Complexity Analysis**

**Time Complexity:** $O(n^2)$  
- Interval DP: $O(n^2)$ to build palindrome table
- Partition search: $O(n^2)$ to try all cut positions

**Space Complexity:** $O(n^2)$  
We use a 2D array `isPal` of size $n \times n$ to store palindrome information.