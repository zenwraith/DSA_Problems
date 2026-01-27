leetcode link : https://leetcode.com/problems/basic-calculator-ii

```python

class Solution:
    def calculate(self, s: str) -> int:

        stack = []
        num = 0
        last_op = '+'

        for i, char in enumerate(s):
            if char.isdigit():
                num = num * 10 + int(char)
            
            if char in '+-*/' or i == len(s) -1:

                if last_op == '+':
                    stack.append(num)
                elif last_op == '-':
                    stack.append(-num)
                elif last_op == '*':
                    stack.append( stack.pop() * num)
                elif last_op =='/':
                    top = stack.pop()
                    stack.append( int(top / num))

                # update operator and reset current number
                last_op = char
                num = 0
        return sum(stack)

        
```
