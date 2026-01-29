
# **Basic Calculator**

**LeetCode Link:** [https://leetcode.com/problems/basic-calculator/](https://leetcode.com/problems/basic-calculator/)

---

## **Problem Statement**

Given a string `s` representing a valid expression, implement a basic calculator to evaluate it, and return the result of the evaluation.

**Note:** You are **not** allowed to use any built-in function which evaluates strings as mathematical expressions, such as `eval()`.

**Constraints:**
- `s` consists of digits, `'+'`, `'-'`, `'('`, `')'`, and `' '` (spaces)
- No multiplication or division (only addition and subtraction)
- Parentheses can be nested

**Example:**
```
Input: s = "(1+(4+5+2)-3)+(6+8)"
Output: 23

Input: s = " 2-1 + 2 "
Output: 3
```

---

## **Strategy: Stack for Nested Parentheses**

**Key Insight:** Parentheses create "sub-problems" that need to be evaluated independently. We use a stack to save the current state when entering parentheses and restore it when exiting.

### **The Core Idea**

1. **Build numbers digit by digit** (handle multi-digit numbers like "123")
2. **Apply the sign** when we see `+`, `-`, or hit a closing `)` or end of string
3. **Save state on `(`**: Push current result and sign to stack, reset for sub-expression
4. **Restore state on `)`**: Multiply sub-result by saved sign, add to saved result

### **Why We Need a Stack**

Consider: `5 - (3 + (2 - 1))`

```
Outer level: 5 - (...)
Middle level: 3 + (...)
Inner level: 2 - 1
```

Each `(` creates a new "context" with its own running result. The stack lets us temporarily save the outer context and return to it later.

---

## **Solution**

```python
class Solution:
    def calculate(self, s: str) -> int:
        stack = [] 
        res = 0          # Running result at current level
        num = 0          # Current number being built
        sign = 1         # 1 for positive, -1 for negative
        
        for char in s:
            if char.isdigit():
                # Build multi-digit numbers (e.g., "123")
                num = num * 10 + int(char)
                
            elif char in '+-':
                # Add the completed number to result with its sign
                res += sign * num
                # Reset number for next value
                num = 0
                # Update sign for next number
                sign = 1 if char == '+' else -1
                
            elif char == '(':
                # Save current state before entering sub-expression
                stack.append(res)    # Save current result
                stack.append(sign)   # Save sign before parenthesis
                # Reset for sub-expression
                res = 0
                sign = 1
                
            elif char == ')':
                # Complete current sub-expression
                res += sign * num
                num = 0
                # Pop saved state and combine
                res *= stack.pop()   # Apply sign from before '('
                res += stack.pop()   # Add to previous result
        
        # Add the final number (handles cases with no trailing operator)
        return res + (sign * num)
```

---

## **Example Walkthrough**

**Input:** `s = "5 - (3 + 2)"`

| Step | Char | Action | stack | res | num | sign |
|------|------|--------|-------|-----|-----|------|
| 1 | `5` | Build number | [] | 0 | 5 | 1 |
| 2 | `-` | Add 5 to res, set sign | [] | 5 | 0 | -1 |
| 3 | `(` | Push state, reset | [5, -1] | 0 | 0 | 1 |
| 4 | `3` | Build number | [5, -1] | 0 | 3 | 1 |
| 5 | `+` | Add 3 to res | [5, -1] | 3 | 0 | 1 |
| 6 | `2` | Build number | [5, -1] | 3 | 2 | 1 |
| 7 | `)` | Complete: res=3+2=5, pop: 5*(-1)+5=0 | [] | 0 | 0 | — |
| End | — | Add final num | [] | 0 | 0 | — |

**Result:** `0`

**Verification:** `5 - (3 + 2) = 5 - 5 = 0` ✓

---

## **Why This Order on Stack?**

When we see `(`, we push:
```python
stack.append(res)   # Push result FIRST
stack.append(sign)  # Push sign SECOND
```

When we see `)`, we pop in **reverse order**:
```python
res *= stack.pop()  # Pop sign FIRST
res += stack.pop()  # Pop result SECOND
```

This follows **LIFO (Last In, First Out)** stack behavior.

---

## **Common Edge Cases**

### **1. Multi-Digit Numbers**
```
Input: "12 + 34"
num builds: 1 → 12, then 3 → 34
```

### **2. Leading/Trailing Spaces**
```
Input: "  2 + 3  "
Spaces are ignored automatically
```

### **3. Nested Parentheses**
```
Input: "1 + (2 - (3 + 4))"
Stack saves multiple levels: [1, 1, 2, -1]
```

### **4. Negative Sub-Expression**
```
Input: "5 - (3 + 2)"
Sign before '(' is -1, so result of (3+2) is multiplied by -1
```

---

## **Why No Multiplication/Division?**

Basic Calculator (this problem) only has `+` and `-` because:
- Both have **equal precedence** and are **left-associative**
- We can evaluate left-to-right without worrying about order of operations
- No need to defer operations or handle precedence

**Basic Calculator II** adds `*` and `/`, requiring operator precedence handling.

---

## **Complexity Analysis**

**Time Complexity:** $O(n)$  
We iterate through the string once, performing constant-time operations for each character.

**Space Complexity:** $O(n)$  
In the worst case (deeply nested parentheses like `"((((5))))"`), the stack can grow to $O(n)$ size.