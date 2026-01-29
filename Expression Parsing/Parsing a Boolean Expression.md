
# **Parsing a Boolean Expression**

**LeetCode Link:** [https://leetcode.com/problems/parsing-a-boolean-expression/](https://leetcode.com/problems/parsing-a-boolean-expression/)

---

## **Problem Statement**

Return the result of evaluating a given boolean expression, represented as a string.

An expression can either be:
- `"t"`, evaluating to True
- `"f"`, evaluating to False
- `"!(expr)"`, NOT operation
- `"&(expr1,expr2,...)"`, AND operation  
- `"|(expr1,expr2,...)"`, OR operation

**Example:**
```
Input: expression = "|(f,t)"
Output: true

Input: expression = "&(t,f)"
Output: false

Input: expression = "!(f)"
Output: true

Input: expression = "|(&(t,f,t),!(t))"
Output: false
```

---

## **Strategy: Stack-Based Expression Evaluation**

**Key Insight:** Nested parentheses naturally map to a stack structure. When we encounter a closing parenthesis, we know we have all the operands needed to evaluate that sub-expression.

### **The Algorithm:**

1. **Push every character** onto the stack (except commas) until you hit a closing parenthesis `)`
2. **When you see `)`**: Pop all operands (t's and f's) until you find the opening parenthesis `(`
3. **Pop the operator** (before the `(`) — this is `!`, `&`, or `|`
4. **Evaluate** the sub-expression and push the result (`t` or `f`) back onto the stack
5. **Continue** until the entire expression is processed

### **Why Use a Set for Operands?**

```python
operands = set()
while stack[-1] != '(':
    operands.add(stack.pop())
```

Using a **set** makes evaluation trivial:
- **AND (`&`)**: Result is `f` if ANY operand is `f` → check `'f' in operands`
- **OR (`|`)**: Result is `t` if ANY operand is `t` → check `'t' in operands`
- **NOT (`!`)**: Result is opposite of the single operand → check `'f' in operands`

---

## **Solution**

```python
class Solution:
    def parseBoolExpr(self, expression: str) -> bool:
        stack = []
        
        for char in expression:
            # Skip commas (they're just separators)
            if char == ',':
                continue
            
            # Push everything except closing parenthesis
            if char != ')':
                stack.append(char)  # Including '(', '!', '&', '|', 't', 'f'
            else:
                # Collect all operands for this sub-expression
                operands = set()
                while stack[-1] != '(':
                    operands.add(stack.pop())
                
                stack.pop()  # Remove '('
                operator = stack.pop()  # Get the operator '!', '&', or '|'
                
                # Evaluate based on operator
                if operator == '!':
                    # NOT: flip the value
                    result = 't' if 'f' in operands else 'f'
                elif operator == '&':
                    # AND: false if any operand is false
                    result = 'f' if 'f' in operands else 't'
                elif operator == '|':
                    # OR: true if any operand is true
                    result = 't' if 't' in operands else 'f'
                
                # Push result back for further evaluation
                stack.append(result)
        
        # Final result
        return stack[0] == 't'
```

---

## **Example Walkthrough**

**Input:** `expression = "|(&(t,f,t),!(t))"`

### **Step-by-Step Execution**

| Step | Char | Stack | Action |
|------|------|-------|--------|
| 1 | `\|` | `['\|']` | Push operator |
| 2 | `(` | `['\|', '(']` | Push |
| 3 | `&` | `['\|', '(', '&']` | Push operator |
| 4 | `(` | `['\|', '(', '&', '(']` | Push |
| 5 | `t` | `['\|', '(', '&', '(', 't']` | Push operand |
| 6 | `,` | - | Skip |
| 7 | `f` | `['\|', '(', '&', '(', 't', 'f']` | Push operand |
| 8 | `,` | - | Skip |
| 9 | `t` | `['\|', '(', '&', '(', 't', 'f', 't']` | Push operand |
| 10 | `)` | `['\|', '(', 'f']` | Pop {t,f,t}, evaluate `&` → `f` |
| 11 | `,` | - | Skip |
| 12 | `!` | `['\|', '(', 'f', '!']` | Push operator |
| 13 | `(` | `['\|', '(', 'f', '!', '(']` | Push |
| 14 | `t` | `['\|', '(', 'f', '!', '(', 't']` | Push operand |
| 15 | `)` | `['\|', '(', 'f', 'f']` | Pop {t}, evaluate `!` → `f` |
| 16 | `)` | `['f']` | Pop {f,f}, evaluate `\|` → `f` |

**Result:** `False`

---


**Result:** `False`

---

## **Expression Parsing Patterns Overview**

### **1. The "Calculator" Pattern (Basic Calculator I, II, III)**

These are the most common. You are given a string like `"3 + (2 * 4) - 5"`.

**Key Skill:** Handling **Operator Precedence** (multiplication before addition) and **Parentheses**.

**The "Magic" Trick:** Use a **Stack**.

- When you see a `+` or `-`, you push the current number to the stack (using the sign).
- When you see `*` or `/`, you pop the last number, multiply/divide it by the current number, and push it back.
- When you see `(`, you use **Recursion** to solve the inside as a "mini-problem."

---

### **2. The "Decoding" Pattern (Decode String)**

**LeetCode 394:** Given `3[a2[c]]`, return `"accaccacc"`.

This is important because it teaches you how to handle **nested multipliers**.

**The Strategy:** Use **two stacks** — one for numbers (the multipliers) and one for strings (the content being built).

**The Logic:**
- When you see a **number**, build it up (e.g., `1` then `12`).
- When you see `[`, push the current number and current string to their stacks and "reset."
- When you see `]`, pop the number and the previous string, repeat your current string $N$ times, and attach it to the previous string.

---

### **3. The "True" Interval DP Parsing (Boolean Parenthesization)**

As we discussed earlier, this is the "hard" version where parentheses are missing.

**Problem:** Given `T | F & T`, how many ways can you add parentheses so the result is True?

**Why it's Interval DP?** Because you have to **choose where to put the brackets**. You try splitting at every operator.

**The State:** `dp[i][j][True]` = number of ways the range $i$ to $j$ can be True.

**The Transition:** If the operator at split $k$ is `&`, then:
$$\text{TotalTrue} = \text{LeftTrue} \times \text{RightTrue}$$

---

## **Complexity Analysis**

**Time Complexity:** $O(n)$  
We iterate through the expression once, and each character is pushed/popped from the stack at most once.

**Space Complexity:** $O(n)$  
In the worst case (deeply nested expressions), the stack can grow to size $n$.