
leetcode link : https://leetcode.com/problems/basic-calculator/

```python

class Solution:
    def calculate(self, s: str) -> int:

        stack = [] 
        res = 0
        num = 0

        sign = 1 # 1 for positive and -1 for negative 

        for char in s:
            if char.isdigit():
                num = num * 10 + int(char)
            elif char in '+-':

                # addd the finished number to result
                res += sign * num
                # reset number
                num = 0
                # update sign
                sign = 1 if char == '+' else -1

            elif char == '(':

                # push current state to hnadle nested logic
                stack.append(res)
                stack.append(sign)
                # reset for inside parens
                res = 0
                sign = 1
            elif char == ')':
                
                # push current state to handle nested logic
                res += sign * num
                num = 0
                # apply sign from before the parens
                res*= stack.pop() # this is the sign
                res += stack.pop() # this is the old res

        # add the final number left in 'num'
        return res + (sign * num)

        
```