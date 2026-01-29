
# **Decode String**

**LeetCode Link:** [https://leetcode.com/problems/decode-string/](https://leetcode.com/problems/decode-string/)

---

## **Problem Statement**

Given an encoded string, return its decoded string.

The encoding rule is: `k[encoded_string]`, where the `encoded_string` inside the square brackets is being repeated exactly `k` times.

**Note:** You may assume that the input string is always valid; there are no extra spaces, brackets are well-formed, etc.

**Example:**
```
Input: s = "3[a]2[bc]"
Output: "aaabcbc"

Input: s = "3[a2[c]]"
Output: "accaccacc"

Input: s = "2[abc]3[cd]ef"
Output: "abcabccdcdcdef"
```

---

## **Strategy: Two-Stack Pattern**

**Key Insight:** Nested brackets create "contexts" where:
- Each `[` starts a new sub-problem
- Each `]` completes a sub-problem and merges it with the previous context

### **Why Two Stacks?**

We need to track two pieces of information when entering a new context:
1. **The multiplier** (`k`) for the current bracket level
2. **The string built so far** (before entering the brackets)

### **The Algorithm:**

1. **If digit**: Build the multiplier (handle multi-digit like `"12"`) → `current_num`
2. **If `[`**: Save current state (push `current_num` and `current_str` to stacks), reset for sub-expression
3. **If `]`**: Pop saved state, multiply current string, concatenate with previous string
4. **If letter**: Append to current string

---

## **Solution**

```python
class Solution:
    def decodeString(self, s: str) -> str:
        num_stack = []    # Stack to store multipliers
        str_stack = []    # Stack to store previous strings
        
        current_str = ""  # String being built at current level
        current_num = 0   # Multiplier being built
        
        for char in s:
            if char.isdigit():
                # Build multi-digit numbers (e.g., "12" → 12)
                current_num = current_num * 10 + int(char)
                
            elif char == '[':
                # Save current state and start new sub-expression
                num_stack.append(current_num)
                str_stack.append(current_str)
                # Reset for the new context
                current_num = 0
                current_str = ""
                
            elif char == ']':
                # Complete current sub-expression
                multiplier = num_stack.pop()
                prev_str = str_stack.pop()
                # Repeat current string and concatenate with previous
                current_str = prev_str + (multiplier * current_str)
                
            else:
                # Regular character: append to current string
                current_str += char
        
        return current_str
```

---

## **Example Walkthrough**

**Input:** `s = "3[a2[c]]"`

### **Step-by-Step Execution**

| Step | Char | Action | num_stack | str_stack | current_num | current_str |
|------|------|--------|-----------|-----------|-------------|-------------|
| 1 | `3` | Build number | [] | [] | 3 | "" |
| 2 | `[` | Push & reset | [3] | [""] | 0 | "" |
| 3 | `a` | Append char | [3] | [""] | 0 | "a" |
| 4 | `2` | Build number | [3] | [""] | 2 | "a" |
| 5 | `[` | Push & reset | [3, 2] | ["", "a"] | 0 | "" |
| 6 | `c` | Append char | [3, 2] | ["", "a"] | 0 | "c" |
| 7 | `]` | Pop: "a" + 2*"c" | [3] | [""] | 0 | "acc" |
| 8 | `]` | Pop: "" + 3*"acc" | [] | [] | 0 | "accaccacc" |

**Result:** `"accaccacc"`

---

## **Detailed State Transitions**

### **When We See `[`**

```python
num_stack.append(current_num)    # Save multiplier
str_stack.append(current_str)    # Save context string
current_num = 0                   # Reset for nested number
current_str = ""                  # Reset for nested string
```

**Why?** We're entering a new "sub-world" inside brackets. We need to remember:
- How many times to repeat what we build inside (the number before `[`)
- What string came before the `[` (to concatenate later)

### **When We See `]`**

```python
multiplier = num_stack.pop()     # Get the repeat count
prev_str = str_stack.pop()       # Get the previous context
current_str = prev_str + (multiplier * current_str)
```

**Why?** We're exiting the "sub-world." We need to:
1. Repeat the string we built inside the brackets
2. Concatenate it with the string from before the brackets

---

## **Visual Example: Nested Brackets**

**Input:** `"2[a3[b]c]"`

```
Level 0: (outer)                  → final: "abbbcabccc"
   ↓
Level 1: 2[...c]                  → builds: "abbbc" (repeated 2x)
      ↓
   Level 2: 3[b]                  → builds: "bbb" (repeated 3x)
```

**Execution:**
```
See '2['  → Save state, enter level 1
See 'a'   → current_str = "a"
See '3['  → Save state, enter level 2
See 'b'   → current_str = "b"
See ']'   → Exit level 2: "a" + 3*"b" = "abbb"
See 'c'   → current_str = "abbbc"
See ']'   → Exit level 1: "" + 2*"abbbc" = "abbbcabbbc"
```

---

## **Why Not Use Recursion?**

**Recursive Approach:**
```python
def decode(s, i):
    if s[i] == '[':
        return decode(s, i+1)  # Recurse into bracket
    # ... more logic
```

**Iterative with Stacks:**
- Cleaner code (no index management)
- Easier to understand state transitions
- No risk of stack overflow on deeply nested inputs

---

## **Common Edge Cases**

### **1. Multi-Digit Numbers**
```
Input: "12[a]"
Output: "aaaaaaaaaaaa" (12 a's)
Building: current_num = 1 → 12
```

### **2. No Brackets**
```
Input: "abc"
Output: "abc"
Stacks remain empty, just build string
```

### **3. Adjacent Brackets**
```
Input: "2[2[y]]"
Output: "yyyy"
Inner: 2*"y" = "yy"
Outer: 2*"yy" = "yyyy"
```

### **4. Trailing Characters**
```
Input: "2[abc]def"
Output: "abcabcdef"
After popping from stacks, continue building
```

---

## **Complexity Analysis**

**Time Complexity:** $O(\text{maxK} \times n)$  
Where $\text{maxK}$ is the maximum value of `k` and $n$ is the length of the string. In the worst case, we might need to repeat a long string multiple times (e.g., `"1000[a]"`). However, for typical inputs with balanced nesting, it's closer to $O(n)$.

**Space Complexity:** $O(m + n)$  
Where $m$ is the maximum depth of nested brackets (stack depth) and $n$ is the length of the output string.

## **Stack Trace Example**

**Input:** `3[a2[c]]`

**Step-by-step execution:**

| Character | Action | num_stack | str_stack | current_num | current_str |
|-----------|--------|-----------|-----------|-------------|-------------|
| 3 | Read digit | [] | [] | 3 | "" |
| [ | Push to stacks | [3] | [""] | 0 | "" |
| a | Append to string | [3] | [""] | 0 | "a" |
| 2 | Read digit | [3] | [""] | 2 | "a" |
| [ | Push to stacks | [3, 2] | ["", "a"] | 0 | "" |
| c | Append to string | [3, 2] | ["", "a"] | 0 | "c" |
| ] | Pop and multiply | [3] | [""] | — | "a" + (2 × "c") = "acc" |
| ] | Pop and multiply | [] | [] | — | "" + (3 × "acc") = "accaccacc" |

**Final Result:** `accaccacc`