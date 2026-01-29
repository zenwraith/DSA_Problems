# Longest Valid Parentheses

**LeetCode:** [Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/)

---

## Problem Statement

Given a string containing just the characters `'('` and `')'`, return the length of the longest valid (well-formed) parentheses substring.

**Example:**
- `s = "(()"` → Output: 2 (The longest valid parentheses substring is "()")
- `s = ")()())"` → Output: 4 (The longest valid parentheses substring is "()()")
- `s = ""` → Output: 0

---

## DP Solution

### Strategy

We use `dp[i]` to represent the length of the longest valid parentheses ending at index `i`.

**Key Observations:**
1. Valid parentheses can only end with `')'`, so `dp[i] = 0` for all `'('`.
2. When we encounter `')'` at position `i`, we check two cases:
   - **Case 1:** Simple pair `"()"`  - if `s[i-1] == '('`
   - **Case 2:** Nested pair `"))"` - if `s[i-1] == ')'` and there's a matching `'('` before the inner valid sequence

### Recurrence Relations

| Case | Pattern | Formula |
|------|---------|---------|
| Simple pair `()` | `s[i-1] == '('` | `dp[i] = dp[i-2] + 2` |
| Nested pair `))` | `s[i-1] == ')'` and `s[i - dp[i-1] - 1] == '('` | `dp[i] = dp[i-1] + 2 + dp[i - dp[i-1] - 2]` |

**Explanation of nested case:**
- `dp[i-1]` is the length of the inner valid sequence
- We check if there's a `'('` before this inner sequence: `s[i - dp[i-1] - 1]`
- If yes, we add: inner length + 2 (for the outer pair) + any valid sequence before the outer `'('`

---

## DP Solution Code

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        n = len(s)
        # dp[i] = length of longest valid parentheses ending at index i
        dp = [0] * n
        ans = 0
        
        for i in range(1, n):
            if s[i] == ')':
                # Case 1: Simple pair "()"
                if s[i-1] == '(':
                    # Add 2 for current pair + any valid sequence before it
                    dp[i] = (dp[i-2] if i-2 >= 0 else 0) + 2
                
                # Case 2: Nested pair "))" - check if there's a matching '(' before inner sequence
                elif i - dp[i-1] > 0 and s[i - dp[i-1] - 1] == '(':
                    inner_length = dp[i-1]  # Length of inner valid sequence
                    # Length of valid sequence before the outer '('
                    length_before_inner_start = dp[i - dp[i-1] - 2] if i - dp[i-1] - 2 >= 0 else 0
                    # Total = inner + outer pair + before outer
                    dp[i] = inner_length + 2 + length_before_inner_start
                
                # All other cases remain 0 (invalid)
            
            ans = max(ans, dp[i])
        
        return ans
```

---

## Why do we use `range(1, n)` and check `i-2 >= 0`?

- We start from `i = 1` because the first character (index 0) cannot form a valid pair by itself.
- We check `i-2 >= 0` before accessing `dp[i-2]` to avoid negative indices (boundary handling).
- The condition `i - dp[i-1] > 0` ensures we don't go out of bounds when looking for the matching `'('`.

**Example walkthrough:**

For `s = "()()"`:
- `i=1`: `s[1]=')'`, `s[0]='('` → Simple pair → `dp[1] = 0 + 2 = 2`
- `i=2`: `s[2]='('` → Skip (only `)` can end valid sequence)
- `i=3`: `s[3]=')'`, `s[2]='('` → Simple pair → `dp[3] = dp[1] + 2 = 4`
- Answer: 4

---

## Stack Solution

### Strategy

Use a stack to track indices of unmatched parentheses. The stack always maintains a "base" for calculating lengths.

**Algorithm:**
1. Initialize stack with `-1` as the base
2. For each `'('`: push its index
3. For each `')'`:
   - Pop from stack
   - If stack becomes empty: this `)` is unmatched, push its index as new base
   - Otherwise: calculate length as `i - stack[-1]`

---

## Stack Solution Code 

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        n = len(s)
        # Stack stores indices; -1 is the base for length calculation
        stack = [-1]
        ans = 0
        
        for i, char in enumerate(s):
            if char == '(':
                # Push index of '('
                stack.append(i)
            else:  # char == ')'
                # Pop the matching '(' or the previous base
                stack.pop()
                
                if not stack:
                    # No matching '(' - this ')' becomes the new base
                    stack.append(i)
                else:
                    # Valid pair found - calculate length from current base
                    curr = i - stack[-1]
                    ans = max(ans, curr)
        
        return ans
```

---

## Why does the stack solution work?

The stack maintains the index of the last unmatched character (either `'('` or `')'`), which serves as a base for calculating valid lengths.

**Key Points:**
- When we find a valid pair, the distance from the current index to the base gives us the length of the valid substring ending here.
- If a `)` has no match, it becomes the new base (because nothing before it can extend to after it).

**Example walkthrough:**

For `s = ")()())"`:
- `i=0, ')'`: pop(-1), stack empty → push 0 (new base)
- `i=1, '('`: push 1
- `i=2, ')'`: pop(1), length = 2 - 0 = 2, ans = 2
- `i=3, '('`: push 3
- `i=4, ')'`: pop(3), length = 4 - 0 = 4, ans = 4
- `i=5, ')'`: pop(0), stack empty → push 5
- Answer: 4

---

## Complexity Analysis

### DP Solution
- **Time Complexity:** $O(n)$ - single pass through the string
- **Space Complexity:** $O(n)$ - dp array

### Stack Solution
- **Time Complexity:** $O(n)$ - single pass through the string
- **Space Complexity:** $O(n)$ - stack in worst case