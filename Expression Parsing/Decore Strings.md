
# **Decode String**

**LeetCode Link:** [https://leetcode.com/problems/decode-string/](https://leetcode.com/problems/decode-string/)

## **Solution**

```python
class Solution:
    def decodeString(self, s: str) -> str:

        num_stack = []
        str_stack = []

        current_str = ""
        current_num = 0

        for char in s:
            if char.isdigit():
                current_num = current_num * 10 + int(char)
            elif char == '[':
                num_stack.append(current_num)
                str_stack.append(current_str)
                current_num = 0
                current_str = ""
            elif char == ']':
                multiplier = num_stack.pop()
                prev_str = str_stack.pop()
                current_str = prev_str + (multiplier * current_str)
            else:
                current_str += char

        return current_str
```

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