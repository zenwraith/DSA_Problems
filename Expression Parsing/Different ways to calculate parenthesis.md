
# **Different Ways to Add Parentheses**

**LeetCode Link:** [https://leetcode.com/problems/different-ways-to-add-parentheses/](https://leetcode.com/problems/different-ways-to-add-parentheses/)

---

## **Problem Statement**

Given a string `expression` of numbers and operators, return all possible results from computing all the different possible ways to group numbers and operators.

You may return the answer in any order.

**Example:**
```
Input: expression = "2-1-1"
Output: [0, 2]
Explanation:
((2-1)-1) = 0
(2-(1-1)) = 2

Input: expression = "2*3-4*5"
Output: [-34, -14, -10, -10, 10]
Explanation:
(2*(3-(4*5))) = -34
((2*3)-(4*5)) = -14
((2*(3-4))*5) = -10
(2*((3-4)*5)) = -10
(((2*3)-4)*5) = 10
```

---

## **Strategy: Divide and Conquer with Memoization**

**Key Insight:** For each operator in the expression, we can treat it as the "last operation" to perform. This splits the expression into two independent subproblems (left and right parts).

### **The Algorithm:**

1. **Scan** the expression for operators (`+`, `-`, `*`)
2. **For each operator at position `i`**:
   - Split into left part: `expression[:i]`
   - Split into right part: `expression[i+1:]`
   - Recursively compute all possible results for both parts
3. **Combine** results from left and right using the operator
4. **Base case:** If no operators found, the expression is just a number → return `[int(expression)]`

### **Why Memoization?**

Sub-expressions like `"2-1"` may appear multiple times when we recursively split. Memoizing avoids recomputing the same subproblem.

---

## **Solution**

```python
class Solution:
    def diffWaysToCompute(self, expression: str) -> List[int]:
        memo = {}
        
        def solve(expr):
            # Check if already computed
            if expr in memo:
                return memo[expr]
            
            res = []
            
            # Try each operator as the "last operation"
            for i, char in enumerate(expr):
                if char in '+-*':
                    # Split into two sub-problems
                    left_results = solve(expr[:i])
                    right_results = solve(expr[i+1:])
                    
                    # Combine results from both sides
                    for l in left_results:
                        for r in right_results:
                            if char == '+':
                                res.append(l + r)
                            elif char == '-':
                                res.append(l - r)
                            elif char == '*':
                                res.append(l * r)
            
            # Base case: no operators found, must be a single number
            if not res:
                res.append(int(expr))
            
            memo[expr] = res
            return res
        
        return solve(expression)
```

---

## **Example Walkthrough**

**Input:** `expression = "2-1-1"`

### **Recursive Tree**

```
solve("2-1-1")
├─ Split at first '-' (index 1)
│  ├─ Left: solve("2") → [2]
│  └─ Right: solve("1-1")
│     ├─ Split at '-' (index 1)
│     │  ├─ Left: solve("1") → [1]
│     │  └─ Right: solve("1") → [1]
│     │  └─ Result: [1 - 1] = [0]
│     └─ Return [0]
│  └─ Combine: [2 - 0] = [2]
│
└─ Split at second '-' (index 3)
   ├─ Left: solve("2-1")
   │  ├─ Split at '-' (index 1)
   │  │  ├─ Left: solve("2") → [2]
   │  │  └─ Right: solve("1") → [1]
   │  │  └─ Result: [2 - 1] = [1]
   │  └─ Return [1]
   └─ Right: solve("1") → [1]
   └─ Combine: [1 - 1] = [0]

Final Results: [2, 0]
```

**Parenthesization Mapping:**
- `2 - 0` → `2 - (1 - 1)` = `2`
- `1 - 1` → `(2 - 1) - 1` = `0`

---

## **Why This Works: The "Last Operation" Principle**

In any fully parenthesized expression, there's always a **last operation** that combines two sub-expressions:

```
((2 * 3) - (4 * 5))
           ^
    This '-' is the last operation
```

By trying **every operator** as the potential "last operation," we enumerate all possible parenthesizations.

---

## **Memoization Key Insight**

Without memoization:
```
For "2-1-1-1":
solve("1-1") is computed multiple times in different branches
```

With memoization:
```python
memo = {
    "1": [1],
    "1-1": [0],
    "2": [2],
    "2-1": [1],
    ...
}
```

Each unique sub-expression is computed only once.

---

## **Comparison: Divide & Conquer vs Interval DP**

| Aspect | This Problem (D&C) | Burst Balloons (Interval DP) |
|--------|-------------------|------------------------------|
| **Subproblem** | Left and Right parts | Left and Right ranges |
| **Memoization** | String → List[int] | 2D table dp[i][j] |
| **Split Logic** | Each operator position | Each balloon as "last burst" |
| **Base Case** | Single number | No balloons in range |
| **Combination** | Apply operator | Add coins |

---

## **Complexity Analysis**

**Time Complexity:** $O(n \times 2^n)$  
- There are exponentially many ways to parenthesize (Catalan number: $C_n \approx \frac{4^n}{n^{3/2}}$)
- Each way involves $O(n)$ operations
- With memoization, we avoid recomputing identical subproblems

**Space Complexity:** $O(n \times 2^n)$  
- Memoization table stores results for all unique sub-expressions
- Each result is a list that can contain multiple values
- Recursion depth is $O(n)$