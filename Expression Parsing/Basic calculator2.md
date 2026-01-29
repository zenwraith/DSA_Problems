# **Basic Calculator II**

**LeetCode Link:** [https://leetcode.com/problems/basic-calculator-ii/](https://leetcode.com/problems/basic-calculator-ii/)

---

## **Problem Statement**

Given a string `s` which represents an expression, evaluate this expression and return its value.

The integer division should truncate toward zero.

**Note:** You are **not** allowed to use any built-in function which evaluates strings as mathematical expressions, such as `eval()`.

**Constraints:**
- `s` consists of integers and operators `'+'`, `'-'`, `'*'`, `'/'`
- **No parentheses** in this version
- All intermediate results will be in the range `[-2^31, 2^31 - 1]`

**Example:**
```
Input: s = "3+2*2"
Output: 7

Input: s = " 3/2 "
Output: 1

Input: s = " 3+5 / 2 "
Output: 5
```

---

## **Strategy: Stack with Operator Precedence**

**Key Challenge:** Multiplication and division have **higher precedence** than addition and subtraction.

**Core Idea:**
- **For `+` and `-`**: Push the number to the stack (with sign)
- **For `*` and `/`**: Immediately pop the last number, compute the result, and push back
- **At the end**: Sum all numbers in the stack

### **Why This Works**

**Expression:** `3 + 2 * 5 - 4`

**Evaluation with Stack:**
```
1. See '3', last_op='+' → push 3        stack: [3]
2. See '2', last_op='+' → push 2        stack: [3, 2]
3. See '5', last_op='*' → pop 2, push 2*5=10   stack: [3, 10]
4. See '4', last_op='-' → push -4       stack: [3, 10, -4]
5. Sum stack: 3 + 10 - 4 = 9
```

By handling `*` and `/` immediately, we respect precedence without needing to scan ahead.

---

## **Solution**

```python
class Solution:
    def calculate(self, s: str) -> int:
        stack = []
        num = 0
        last_op = '+'  # Initialize with '+' for first number
        
        for i, char in enumerate(s):
            if char.isdigit():
                # Build multi-digit numbers
                num = num * 10 + int(char)
            
            # Process when we hit an operator OR reach end of string
            # (char in '+-*/' and char != ' ') handles operators
            # i == len(s) - 1 handles the last number
            if char in '+-*/' or i == len(s) - 1:
                
                # Apply the PREVIOUS operator to current number
                if last_op == '+':
                    stack.append(num)
                elif last_op == '-':
                    stack.append(-num)
                elif last_op == '*':
                    stack.append(stack.pop() * num)
                elif last_op == '/':
                    top = stack.pop()
                    # Truncate toward zero (Python's // truncates toward -inf)
                    stack.append(int(top / num))
                
                # Update operator for next iteration and reset number
                last_op = char
                num = 0
        
        return sum(stack)
```

---

## **Example Walkthrough**

**Input:** `s = "3 + 2 * 2"`

| Step | i | char | num | last_op | Action | stack |
|------|---|------|-----|---------|--------|-------|
| 1 | 0 | `3` | 3 | `+` | Build number | [] |
| 2 | 1 | ` ` | 3 | `+` | Skip space | [] |
| 3 | 2 | `+` | 3 | `+` | Apply '+': push 3, update op | [3] |
| 4 | 3 | ` ` | 0 | `+` | Skip space | [3] |
| 5 | 4 | `2` | 2 | `+` | Build number | [3] |
| 6 | 5 | ` ` | 2 | `+` | Skip space | [3] |
| 7 | 6 | `*` | 2 | `+` | Apply '+': push 2, update op | [3, 2] |
| 8 | 7 | ` ` | 0 | `*` | Skip space | [3, 2] |
| 9 | 8 | `2` | 2 | `*` | Build number | [3, 2] |
| 10 | 8 | END | 2 | `*` | Apply '*': pop 2, push 2*2=4 | [3, 4] |

**Sum:** `3 + 4 = 7`

---

## **Why Check `i == len(s) - 1`?**

**Problem:** The last number in the expression doesn't have a trailing operator.

**Example:** `"3 + 2"`

Without the end check:
```
When we process '2', we build num=2
But there's no operator after '2' to trigger processing!
The '2' would never get added to the stack!
```

With the end check:
```python
if char in '+-*/' or i == len(s) - 1:
```
When we reach the last character, we process the pending number.

---

## **Why `int(top / num)` Instead of `top // num`?**

**Python's Division Behavior:**
```python
-3 // 2 = -2   # Floor division (toward -∞)
-3 / 2 = -1.5
int(-3 / 2) = -1  # Truncate toward zero
```

**LeetCode Requirement:** "Integer division should truncate toward zero."

**Solution:** Use `int(top / num)` which truncates toward zero, not floor division `//`.

---

## **Why Not Use Two Passes?**

**Alternative Approach:**
1. First pass: Handle all `*` and `/`
2. Second pass: Handle all `+` and `-`

**Our Approach:** Single pass with a stack is more elegant and efficient.

---

## **Comparison: Calculator I vs II**

| Feature | Basic Calculator I | Basic Calculator II |
|---------|-------------------|---------------------|
| **Operators** | `+`, `-` only | `+`, `-`, `*`, `/` |
| **Parentheses** | Yes (nested) | No |
| **Precedence** | Equal (left-to-right) | `*`/`/` before `+`/`-` |
| **Stack Usage** | Save state for `()` | Handle high-precedence ops |
| **Key Pattern** | Push/pop on `(` and `)` | Immediate eval for `*`/`/` |

---

## **Common Edge Cases**

### **1. Spaces**
```
Input: " 3 + 2 "
Spaces are skipped (neither digit nor operator)
```

### **2. Division by Zero**
```
Per problem constraints, this won't occur
```

### **3. Negative Division**
```
Input: "5 / -2" is invalid (no parentheses)
Input: "14 - 3 * 2" is valid
```

### **4. Leading Operator**
```
Initializing last_op = '+' handles first number correctly
The first number is treated as '+num'
```

---

## **Complexity Analysis**

**Time Complexity:** $O(n)$  
We iterate through the string once, performing constant-time operations for each character.

**Space Complexity:** $O(n)$  
The stack can hold up to $n/2$ numbers in the worst case (e.g., `"1+2+3+4+5"`).

        
```
